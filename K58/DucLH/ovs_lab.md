# Lab OVS with mininet
## Overview
Sau khi tìm hiểu về Open vSwitch (OVS), bao gồm kiến trúc, các thành phần và các xử lý các gói tin. Với mục đích để cũng cố lý thuyết về OVS và hiểu rõ hơn cách sử dụng OVS, tài liệu này sẽ đi vào triển khai một mô hình mạng đơn giản dựa trên mininet với các virtual switch dưới sự điều khiển của các công cụ trong OVS. Trước khi đọc tài liệu này, nên đọc qua 2 tài liệu về [OVS](https://github.com/huynhducbk95/openstack-manual/blob/master/DucLH/ovs_components.md) và [mininet](https://github.com/huynhducbk95/openstack-manual/blob/master/DucLH/mininet_document.md) để hiểu rõ hơn các câu lệnh được dùng trong quá trình triển khai.
## Mô hình Lab
Dưới đây là mô hình lab được triển khai trong tài liệu này:
![](https://github.com/huynhducbk95/networking_document/blob/master/image/lab_ovs_topology.png?raw=true)
Để tạo lab này, chúng ta cần chuẩn bị môi trường là một virtual machine đã cài đặt openvswitch và mininet. Tất cả các câu lệnh trong tài liệu này đều được thực hiện trên VM này. Sử dụng ssh để ssh từ local machine đến VM.

Trong mô hình trên, chúng ta sẽ có 3 switch kết nối tuyến tính với nhau và mỗi switch kết nối với một host. Để tạo ra mô hình này, chúng ta sẽ sử dụng mininet với câu lệnh sau:
```
	$ sudo mn -topo linear,3 --mac --switch ovsk,datapath=user

```
Câu lệnh này sẽ tạo ra một mạng giả lập 3 switch nối với nhau và nối với một host ( *--topo linear,3*); tùy chọn *--mac* sẽ đánh địa chỉ mac cho các host h1, h2 và h3; tùy chọn *--switch ovsk,datapath=user* sẽ tạo ra các switch thuộc OVS switch và datapath=user sẽ thực hiện mọi xử lý gói tin ở userspace mà không đi xuống kernel module của OVS.

Sau khi thực hiện câu lệnh trên, chúng ta sẽ có được mô hình mạng như hình trên. Có thể dùng các câu lệnh sau để kiểu tra các link hay thông tin cấu hình các switch trong mạng vừa tạo ra:
```
	mininet> net
    mininet> h1 ifconfig

```
Tiếp theo, mở một tab terminal khác, ssh vào VM để thực hiện các câu lệnh để quản lý các switch vừa được tạo ra. Chúng ta sẽ đặt tên **tab_mininet** là tab chạy câu lệnh mininet trên và **tab_ovs** là tab sẽ chạy các câu lệnh Open vSwitch.

Đầu tiên, chúng ta có thể kiểm tra ping giữa các host trên tab_mininet với câu lệnh sau:
```
	mininet> pingall
	*** Ping: testing ping reachability
	h1 -> h2 h3
	h2 -> h1 h3
	h3 -> h1 h2
	*** Results: 0% dropped (6/6 received)

```
kết quả là các host có thể ping được với nhau.

Bây giờ, chúng ta sẽ nói về mục đích của lab này: Chúng ta sẽ quy định các switch này chỉ làm việc với các gói tin ARP và IPv4, các gói tin khác sẽ bị xóa bỏ. Thực hiện tạo các flow để kết nối h1 và h3 đối với các gói tin IPv4 và các gói tin ARP của 2 host này, điều này có nghĩa là host h2 sẽ không thể gửi nhận các gói tin IPv4 hoặc ARP đến h1 và h3. Và tất nhiên, h1 và h3 có thể ping được cho nhau, trong khi không thể ping được cho h2.

Để thực hiện điều này, chúng ta sẽ chuyển qua **tab_ovs**, tạo một script có tên là "flow_script.sh". Sau đây sẽ là các câu lệnh được thực hiện trong script.

Phiên bản hiện tại của mininet hỗ trợ OpenFlow version 1.0 theo mặc định, chúng ta sẽ chuyển sang version 1.3 hiện nay, bằng câu lệnh sau:

    # Configure switch to use OpenFlow 1.3
    echo "Setting Switches to work with OpenFlow 1.3"
    
    for i in s1 s2 s3; do
	    sudo ovs-vsctl set bridge $i protocols=OpenFlow13
	done

Tiếp theo, để các switch có thể làm thực hiện theo các flow mà mình tự định nghĩa thì chúng ta sẽ xóa đi tất các các flow hiện tại đang có trong switch s1, s2 và s3. Thực hiện bằng câu lệnh sau:

    # Clear flow tables
    echo "Clearing Flow tables.."
    
    for i in s1 s2 s3; do
	    sudo ovs-ofctl -O OpenFlow13 del-flows $i
	done
**Important note:** Để hiểu rõ các flow sẽ được mô tả tiếp sau, chúng ta nên xem kỹ topology ở hình trên, để có thể hiểu được vì sao có port vào và port ra như thế. Đồng thời, ở mỗi câu lệnh add-flow, nên quay lại topology để hình dung dễ hơn flow sẽ thực hiện như thế nào.

Bây giờ, chúng ta sẽ tạo các flow để match với các gói tin IPv4 để switch s1 tại cổng s1-eth1 và s1-eth2:

    ###### Rules at S1 ######
    echo "Setting up rules at s1"
    
    ## TABLE 0 ##

	sudo ovs-ofctl -O OpenFlow13 add-flow s1 "table=0,in_port=1,eth_type=0x800,dl_src=00:00:00:00:00:01,dl_dst=00:00:00:00:00:03,actions=output:2"
	
	sudo ovs-ofctl -O OpenFlow13 add-flow s1 "table=0,in_port=2,eth_type=0x800,dl_src=00:00:00:00:00:03,dl_dst=00:00:00:00:00:01,actions=output:1"

Trong 2 câu lệnh add-flow trên chúng ta có thể thấy:

 - Mã hexa của gói tin IPv4 là 0x800.
 - in_port là port mà gói tin đi qua để vào switch.
 - Table 0 là table đầu tiên mà một packet đi vào switch trong chuỗi các flow table trong kỹ thuật pipeline processing của mỗi switch.
 - dl_src, dl_dst là địa chỉ MAC của máy gửi và máy nhận.
 - action=output:2 là hành động sẽ forward gói tin đến port s1-eth2 của switch.

Ví dụ như trong câu lệnh add-flow đầu tiên, ta thấy, để h1 ping được h3 thì h1 cần gửi một gói tin IPv4 đến h3. Do h1 gắn trực tiếp với switch s1 tại port s1-eth1 của s1, nên mọi gói tin được gửi từ h1 sẽ phải đi qua s1-eth1 rồi mới đi vào được switch s1. Trong trường hợp này, h1 gửi đến h3 nên destination MAC là h3. Sau đó, thực hiện hành động forward gói tin ra khỏi switch s1 qua port s1-eth2.

Tiếp theo, chúng ta sẽ cấu hình flow tại switch s2 với các flow sau:

    ###### Rules at s2 ######
	echo "Setting up rules at s2"
	
	sudo ovs-ofctl -O OpenFlow13 add-flow s2 "table=0,in_port=2,eth_type=0X800,dl_src=00:00:00:00:00:01,dl_dst=00:00:00:00:00:03,actions=output:3"
	
	sudo ovs-ofctl -O OpenFlow13 add-flow s2 "table=0,in_port=3,eth_type=0X800,dl_src=00:00:00:00:00:03,dl_dst=00:00:00:00:00:01,actions=output:2"
Các câu lệnh trong switch s2 cũng tương tự như trong s1. Điều duy nhất chúng ta cần quan tâm ở đây là: Trong các câu lệnh trên, chúng ta không thấy sự xuất hiện của port s2-eth1. Như trong topology ở đầu trang, h2 gắn với switch s2 qua port s2-eth1. Điều này cho thấy, h2 sẽ không nhận hay gửi được gói tin nào từ bên ngoài ( đây chính là mục đích của lab).

Tiếp theo, chúng ta sẽ cấu hình flow cho switch s3:

    ###### Rules at s3 ######
	echo "Setting up rules at s3"
	
	sudo ovs-ofctl -O OpenFlow13 add-flow s3 "table=0,in_port=2,eth_type=0X800,dl_src=00:00:00:00:00:01,dl_dst=00:00:00:00:00:03,actions=output:1"
	
	sudo ovs-ofctl -O OpenFlow13 add-flow s3 "table=0,in_port=1,eth_type=0X800,dl_src=00:00:00:00:00:03,dl_dst=00:00:00:00:00:01,actions=output:2"
Như vậy, chúng ta đã cấu hình xong các flow sẽ thực hiện việc chuyển tiếp các gói tin IPv4 giữa h1 và h3. Host h2 sẽ bị cô lập ( mà sau này chúng ta sẽ tạo một script khác để h2 có thể ping được với h1 và h3).

Tiếp theo sẽ là các gói tin ARP, sau đây sẽ là các flow được thêm vào các switch:

    ###### Rules for ARP traffic ######
	echo "Setting up rules for ARP traffic"
	
	sudo ovs-ofctl -O OpenFlow13 add-flow s1 "table=0,in_port=1,eth_type=0X806,dl_src=00:00:00:00:00:01,nw_src=10.0.0.1,nw_dst=10.0.0.3,actions=output:2"
	sudo ovs-ofctl -O OpenFlow13 add-flow s1 "table=0,in_port=2,eth_type=0X806,dl_src=00:00:00:00:00:03,nw_src=10.0.0.3,nw_dst=10.0.0.1,actions=output:1"
	
	sudo ovs-ofctl -O OpenFlow13 add-flow s2 "table=0,in_port=2,eth_type=0X806,dl_src=00:00:00:00:00:01,nw_src=10.0.0.1,nw_dst=10.0.0.3,actions=output:3"
	sudo ovs-ofctl -O OpenFlow13 add-flow s2 "table=0,in_port=3,eth_type=0X806,dl_src=00:00:00:00:00:03,nw_src=10.0.0.3,nw_dst=10.0.0.1,actions=output:2"
	
	sudo ovs-ofctl -O OpenFlow13 add-flow s3 "table=0,in_port=2,eth_type=0X806,dl_src=00:00:00:00:00:01,nw_src=10.0.0.1,nw_dst=10.0.0.3,actions=output:1"
	sudo ovs-ofctl -O OpenFlow13 add-flow s3 "table=0,in_port=1,eth_type=0X806,dl_src=00:00:00:00:00:03,nw_src=10.0.0.3,nw_dst=10.0.0.1,actions=output:2"

Chúng ta sẽ xem xét câu lệnh add-flow đầu tiên và các câu lệnh sau sẽ tương tự. Mục đích của gói tin ARP là xác định địa chỉ MAC của host đích. Do đó, chúng ta sẽ cung cấp flow với các thông tin là MAC source address, IP source address, IP destination address:

 - Mã hecxa của gói tin ARP là 0x806.
 - in_port là port mà gói tin đi qua để vào switch.
 - nw_src, nw_dst là địa chỉ IP nguồn và đích của gói tin.
 - action=output:2 là port ra mà gói tin sẽ được forward đến.

Như vậy là chúng ta đã hoàn thành việc cấu hình các flow trong các switch s1, s2 và s3. Và cuối cùng, file script "flow_script.sh" chúng ta có được là:

    #!/bin/bash
	# Configure switch to use OpenFlow 1.3
	echo "Setting Switches to work with OpenFlow 1.3"
	
	for i in s1 s2 s3; do
	 sudo ovs-vsctl set bridge $i protocols=OpenFlow13
	done
	
	
	# Clear flow tables
	echo "Clearing Flow tables.."
	for i in s1 s2 s3; do
	 sudo ovs-ofctl -O OpenFlow13 del-flows $i
	done
	
	
	###### Rules at S1 ######
	
	echo "Setting up rules at s1"
	
	## TABLE 0 ##
	sudo ovs-ofctl -O OpenFlow13 add-flow s1 "table=0,in_port=1,eth_type=0x800,dl_src=00:00:00:00:00:01,dl_dst=00:00:00:00:00:03,actions=output:2"
	sudo ovs-ofctl -O OpenFlow13 add-flow s1 "table=0,in_port=2,eth_type=0x800,dl_src=00:00:00:00:00:03,dl_dst=00:00:00:00:00:01,actions=output:1"
	
	
	###### Rules at s2 ######
	echo "Setting up rules at s2"
	
	sudo ovs-ofctl -O OpenFlow13 add-flow s2 "table=0,in_port=2,eth_type=0X800,dl_src=00:00:00:00:00:01,dl_dst=00:00:00:00:00:03,actions=output:3"
	sudo ovs-ofctl -O OpenFlow13 add-flow s2 "table=0,in_port=3,eth_type=0X800,dl_src=00:00:00:00:00:03,dl_dst=00:00:00:00:00:01,actions=output:2"
	
	
	###### Rules at s3 ######
	echo "Setting up rules at s3"
	sudo ovs-ofctl -O OpenFlow13 add-flow s3 "table=0,in_port=2,eth_type=0X800,dl_src=00:00:00:00:00:01,dl_dst=00:00:00:00:00:03,actions=output:1"
	sudo ovs-ofctl -O OpenFlow13 add-flow s3 "table=0,in_port=1,eth_type=0X800,dl_src=00:00:00:00:00:03,dl_dst=00:00:00:00:00:01,actions=output:2"
	
	###### Rules for ARP traffic ######
	echo "Setting up rules for ARP traffic"
	
	sudo ovs-ofctl -O OpenFlow13 add-flow s1 "table=0,in_port=1,eth_type=0X806,dl_src=00:00:00:00:00:01,nw_src=10.0.0.1,nw_dst=10.0.0.3,actions=output:2"
	sudo ovs-ofctl -O OpenFlow13 add-flow s1 "table=0,in_port=2,eth_type=0X806,dl_src=00:00:00:00:00:03,nw_src=10.0.0.3,nw_dst=10.0.0.1,actions=output:1"
	
	sudo ovs-ofctl -O OpenFlow13 add-flow s2 "table=0,in_port=2,eth_type=0X806,dl_src=00:00:00:00:00:01,nw_src=10.0.0.1,nw_dst=10.0.0.3,actions=output:3"
	sudo ovs-ofctl -O OpenFlow13 add-flow s2 "table=0,in_port=3,eth_type=0X806,dl_src=00:00:00:00:00:03,nw_src=10.0.0.3,nw_dst=10.0.0.1,actions=output:2"
	
	sudo ovs-ofctl -O OpenFlow13 add-flow s3 "table=0,in_port=2,eth_type=0X806,dl_src=00:00:00:00:00:01,nw_src=10.0.0.1,nw_dst=10.0.0.3,actions=output:1"
	sudo ovs-ofctl -O OpenFlow13 add-flow s3 "table=0,in_port=1,eth_type=0X806,dl_src=00:00:00:00:00:03,nw_src=10.0.0.3,nw_dst=10.0.0.1,actions=output:2"

Bây giờ, lưu file "flow_script.sh" và chạy script này. chú ý cấp quyền thực thi đối với file script với **chmod**:

    chmod 755 flow_script.sh
    ./flow_script.sh

Sau khi chạy xong script, chúng ta sẽ test những gì đã làm bằng câu lệnh **pingall** trong tab **tab_mininet** và kết qủa đạt được như sau:

    mininet> pingall
	*** Ping: testing ping reachability
	h1 -> X h3 
	h2 -> X X 
	h3 -> h1 X 
	*** Results: 66% dropped (2/6 received)
	mininet> 

Như vậy, h1 và h3 đã ping được với nhau trong khi h2 không thể ping được khác host khác và các host khác cũng không ping được h2.

Để có thể nhìn thầy rõ hơn các gói tin nào đi qua các port thì có thể sử dụng công cụ **wireshark**, chọn port ví dụ s2-eth2, và thực hiện ping giữa các host. Wireshark sẽ chỉ hiển thị các gói tin ARP và IPv4 đi qua s2-eth2.


![](https://github.com/huynhducbk95/hpcc-know-how/blob/master/K58/DucLH/image/test_pingall_lab_ovs.png?raw=true)
 
 Trong hình trên cho thấy rõ h2 không thực hiện được việc gửi và nhận các gói tin ARP và IPv4 đến các host khác.
## Cấu hình đưa h2 vào mạng

Trên đây, chúng ta đã thực hiện tạo các flow có thể gửi nhận các gói tin ARP và IPv4 giữa h1 và h3. Host h2 không thể ping được đến các host khác và ngược lại. Trong phần này, mình sẽ tạo một script khác để tạo ra các flow thực hiện việc gửi nhận các gói tin ARP và IPv4 đến các host khác và ngược lại, các host khác gửi đến h2.

Quay lại tab **tab_ovs** và tạo file script "h2_script.sh" với nội dung như sau:

**Important Note**: Vì các flow cũng tương tự như script đã viết ở trên, nên mình sẽ không giải chi tiết. Ở mỗi câu lệnh add-flow nên quay lại topology trên để có thể hiểu rõ hơn flow thực hiện công việc gì. Tư tưởng đơn giản là xác định đường đi cho gói tin trong topology, xác định các port vào, port ra tương ứng trên các switch, xác định các tham số chính xác trong các flow để tạo ra các flow chính xác cho đường đi.

    #!/bin/bash

	# Rule s1
	
	sudo ovs-ofctl -O OpenFlow13 add-flow s1 "table=0,in_port=2,eth_type=0x800,dl_src=00:00:00:00:00:02,dl_dst=00:00:00:00:00:01,actions=output:1"
	sudo ovs-ofctl -O OpenFlow13 add-flow s1 "table=0,in_port=1,eth_type=0x800,dl_src=00:00:00:00:00:01,dl_dst=00:00:00:00:00:02,actions=output:2"
	
	
	
	
	# Rule s2
	
	sudo ovs-ofctl -O OpenFlow13 add-flow s2 "table=0,in_port=1,eth_type=0x800,dl_src=00:00:00:00:00:02,dl_dst=00:00:00:00:00:03,actions=output:3"
	sudo ovs-ofctl -O OpenFlow13 add-flow s2 "table=0,in_port=1,eth_type=0x800,dl_src=00:00:00:00:00:02,dl_dst=00:00:00:00:00:01,actions=output:2"
	sudo ovs-ofctl -O OpenFlow13 add-flow s2 "table=0,in_port=3,eth_type=0x800,dl_src=00:00:00:00:00:03,dl_dst=00:00:00:00:00:02,actions=output:1"
	sudo ovs-ofctl -O OpenFlow13 add-flow s2 "table=0,in_port=2,eth_type=0x800,dl_src=00:00:00:00:00:01,dl_dst=00:00:00:00:00:02,actions=output:1"
	
	# Rule s3
	
	sudo ovs-ofctl -O OpenFlow13 add-flow s3 "table=0,in_port=2,eth_type=0x800,dl_src=00:00:00:00:00:02,dl_dst=00:00:00:00:00:03,actions=output:1"
	sudo ovs-ofctl -O OpenFlow13 add-flow s3 "table=0,in_port=1,eth_type=0x800,dl_src=00:00:00:00:00:03,dl_dst=00:00:00:00:00:02,actions=output:2"
	
	
	# ARP packet
	
	sudo ovs-ofctl -O OpenFlow13 add-flow s1 "table=0,in_port=1,eth_type=0x806,dl_src=00:00:00:00:00:01,nw_src=10.0.0.1,nw_dst=10.0.0.2,actions=output:2"
	sudo ovs-ofctl -O OpenFlow13 add-flow s1 "table=0,in_port=2,eth_type=0x806,dl_src=00:00:00:00:00:02,nw_src=10.0.0.2,nw_dst=10.0.0.1,actions=output:1"
	
	sudo ovs-ofctl -O OpenFlow13 add-flow s2 "table=0,in_port=2,eth_type=0x806,dl_src=00:00:00:00:00:01,nw_src=10.0.0.1,nw_dst=10.0.0.2,actions=output:1"
	sudo ovs-ofctl -O OpenFlow13 add-flow s2 "table=0,in_port=3,eth_type=0x806,dl_src=00:00:00:00:00:03,nw_src=10.0.0.3,nw_dst=10.0.0.2,actions=output:1"
	sudo ovs-ofctl -O OpenFlow13 add-flow s2 "table=0,in_port=1,eth_type=0x806,dl_src=00:00:00:00:00:02,nw_src=10.0.0.2,nw_dst=10.0.0.1,actions=output:2"
	sudo ovs-ofctl -O OpenFlow13 add-flow s2 "table=0,in_port=1,eth_type=0x806,dl_src=00:00:00:00:00:02,nw_src=10.0.0.2,nw_dst=10.0.0.3,actions=output:3"
	
	
	sudo ovs-ofctl -O OpenFlow13 add-flow s3 "table=0,in_port=2,eth_type=0x806,dl_src=00:00:00:00:00:02,nw_src=10.0.0.2,nw_dst=10.0.0.3,actions=output:1"
	sudo ovs-ofctl -O OpenFlow13 add-flow s3 "table=0,in_port=1,eth_type=0x806,dl_src=00:00:00:00:00:03,nw_src=10.0.0.3,nw_dst=10.0.0.2,actions=output:2"

Sau đó, chạy script "h2_script.sh" và thực hiện test với câu lệnh **pingall** trong tab **tab_mininet**. Chúng ta sẽ thu được kết qủa:

    mininet> pingall
	*** Ping: testing ping reachability
	h1 -> h2 h3 
	h2 -> h1 h3 
	h3 -> h1 h2 
	*** Results: 0% dropped (6/6 received)
	mininet> 

Như vậy chúng ta đã có thể ping được giữa các host. Chúng ta cũng có thể **bắt** các gói tin tại các port của các switch bằng **wireshark** tương tự như trên. Ở đây, mình sẽ kiểm tra lại port s2-eth2 như phần trước để xem xét sự khác biệt ở đây.


![](https://github.com/huynhducbk95/hpcc-know-how/blob/master/K58/DucLH/image/test_ping_h2.png?raw=true)


So sánh gửi 2 hình, chúng ta cũng có thể thấy host h2 đã thực hiện được việc gửi và nhận các gói tin ARP và IPv4 từ các host khác.

Cuối cùng, để kiểm tra các flow đang có trong mỗi switch, chúng ta có thể sử dụng câu lệnh sau:

	    root@vm1-VirtualBox:/home/vm1# sudo ovs-ofctl -O openflow13 dump-flows s1
	OFPST_FLOW reply (OF1.3) (xid=0x2):
	 cookie=0x0, duration=424.361s, table=0, n_packets=4, n_bytes=240, arp,in_port=1,dl_src=00:00:00:00:00:01,arp_spa=10.0.0.1,arp_tpa=10.0.0.2 actions=output:2
	 cookie=0x0, duration=2351.786s, table=0, n_packets=7, n_bytes=420, arp,in_port=1,dl_src=00:00:00:00:00:01,arp_spa=10.0.0.1,arp_tpa=10.0.0.3 actions=output:2
	 cookie=0x0, duration=424.353s, table=0, n_packets=4, n_bytes=240, arp,in_port=2,dl_src=00:00:00:00:00:02,arp_spa=10.0.0.2,arp_tpa=10.0.0.1 actions=output:1
	 cookie=0x0, duration=2351.777s, table=0, n_packets=7, n_bytes=420, arp,in_port=2,dl_src=00:00:00:00:00:03,arp_spa=10.0.0.3,arp_tpa=10.0.0.1 actions=output:1
	 cookie=0x0, duration=2351.859s, table=0, n_packets=9, n_bytes=882, ip,in_port=1,dl_src=00:00:00:00:00:01,dl_dst=00:00:00:00:00:03 actions=output:2
	 cookie=0x0, duration=424.457s, table=0, n_packets=4, n_bytes=392, ip,in_port=2,dl_src=00:00:00:00:00:02,dl_dst=00:00:00:00:00:01 actions=output:1
	 cookie=0x0, duration=2351.849s, table=0, n_packets=9, n_bytes=882, ip,in_port=2,dl_src=00:00:00:00:00:03,dl_dst=00:00:00:00:00:01 actions=output:1
	 cookie=0x0, duration=424.446s, table=0, n_packets=4, n_bytes=392, ip,in_port=1,dl_src=00:00:00:00:00:01,dl_dst=00:00:00:00:00:02 actions=output:2
	root@vm1-VirtualBox:/home/vm1# 


# Tổng kết

Trên đây là bài viết về lab mình đã thực hiện để có thể hiểu hơn các câu lệnh với flow trong Open vSwitch switch. Tuy nhiên, bài viết này chỉ thực hiện trên table 0 ( flow table đầu tiên mà các packet đi vào switch), ngoài ra còn có các bảng khác để tương tác với packet như table 1 dùng để thêm VLAN cho packet, table 2 để thực hiện cơ chế tự học MAC+VLAN của switch,... Cách sử dụng của các table này mình sẽ thực hiện ở một bài viết khác
