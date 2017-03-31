Trong bài viết này chúng ta sẽ tìm hiểu  xem luồng dữ liệu khi đi qua các loại mạng cục bộ được triển khai trên Openstack sử dụng Linux bridge hoạt động như thế nào.

# 1. Giới thiệu Linux bridge

Một linux bridge là một switch ảo cho phép kết nối nhiều network interface. Khi sử dụng Neutron, một linux bridge thường kết nối một physical interface(card mạng vật lí) tới một hoặc nhiều virtual interface(card mạng ảo). Một physical interface có thể là Ethernet interface như **eth1**, hoặc bonded interface như **bond0**. Một virtual interface có thể là VLAN interface như **eth1.100** hoặc **tap** interface được tạo bởi KVM. Ngoài ra bạn có thể kết nối nhiều card mạng vật lí hoặc ảo tới Linux bridge.

![Imgur](http://i.imgur.com/sjhdDTh.png)

Trong hình trên, Linux bridge (**brqXXXX**) được kết nối tới một physical interface (**eth1**) và 3 virtual interface (tap0, tap1, tap2). Các tap này được tạo ra bởi trình ảo hóa, cụ thể trong ví dụ này là KVM. Thay vì phải đưa frames tới hoặc nhận frame từ card Ethernet vật lý, các frames này được đọc và ghi bởi chương trình thuộc user space. Kernel sẽ cho phép kích hoạt các tap interface thông qua file **/dev/tapN**, trong đó N là số hiệu của card mạng. Luồng dữ liệu từ eth0 trên máy ảo có thể đi qua thông suốt trên tap interface tương ứng cũng như bridge interface và physical interface.

# 2. Luồng dữ liệu (network traffic flow) khi sử dụng Linux bridge

Khi một Ethernet frame được di chuyển từ máy ảo tới mạng vật lí sử dụng Linux bridge, chúng có thể đi qua các interface hoặc device sau tùy thuộc vào loại mạng được sử dụng (Flat, VLAN, VXLAN)
    
   •  **Tap interface** (tapN): Nó là một virtual interface dưới dạng phần mềm được tạo và sử dụng bởi hypervisor ví dụ như QEMU/KVM. Một tap interface tương ứng với một card mạng trên VM.
   
   • **Linux bridge** (brqXXXX)
   
   • **VLAN interface** (ethX.<vlan>): Linux hỗ trợ 802.1q VLAN tagging thông qua sử dụng virtual VLAN interface. Một VLAN interface có thể được tạo ra bằng cách sử dụng iproute2 command và chúng thường được đặt tên là ethX.<vlan> với ethX là card mạng vật lí liên kết tới nó và <vlan> là VLANID.
   
   • **VXLAN interface** (vxlan-z): Là một virtual interface giúp truyền gói tin L2 frame bằng cách sử dụng giao thức MAC-in-UDP (MAC Address-in-User Datagram Protocol). Tức là phương thức để truyền gói tin L2 frame trong mạng là sử dụng IP và UDP để truyền dẫn.  
   
   • **Physical interface** (ethX)
    
Để hiểu rõ hơn cách Neutron sử dụng Linux bridge và network traffic flow như thế nào, chúng ta sẽ xem xét một vài ví dụ sử dụng Linux bridge để triển khai mạng cục bộ trên Openstack dưới đây.
    
## 2.1 Flat

Khi triển khai mạng Flat, sẽ có một bridge ảo được tạo ra trên các node và nối trực tiếp tới card mạng vật lý. Các máy ảo sử dụng mạng Flat đó, mỗi máy sẽ nối 1 card mạng ảo vào bridge ảo này để kết nối với mạng Flat.  Điều này có nghĩa là một card mạng vật lí chỉ triển khai được một mạng Flat. Trong trường hợp không có kết nối giữa bridge và card mạng vật lí thì mạng Flat trở thành mạng **local**, các máy ảo chỉ có thể giao tiếp được với nhau nhưng thể kết nối tới external network.
   
Hình dưới đây biểu diễn một card mạng vật lí kết nối tới Linux bridge trong kịch bản sử dụng mạng Flat. Trong kịch bản trên, bridge **brqXXXX** được kết nối trực tiếp tới eth1 và nối với 3 máy ảo thông qua 3 tap interface **tapX**.

![Imgur](http://i.imgur.com/8GNvFtJ.png)
    
Khi nhiều hơn một mạng Flat được tạo, mỗi mạng Flat phải được kết nối tới một card vật lí riêng biệt. Hình dưới đây biểu diễn kịch bản sử dụng 2 mạng Flat với 2 card mạng vật lí.

![Imgur](http://i.imgur.com/t8xmi3e.png)
    
## 2.1 VLAN

Chúng ta sẽ xem xét ví dụ gồm một mạng duy nhất sử dụng VLAN để kết nối tới các VMs như hình dưới đây.
   
![Imgur](http://i.imgur.com/PmHMIBc.png)

Trong hình trên, 3 VMs được kết nối tới Linux bridge **brqXXXX** thông qua các tap interface tương ứng. Khi VM đầu tiên được tạo ra trên mạng VLAN thuộc mạng VLAN100 thì một virtual interface tương ứng eth1.100 sẽ được tạo ra và kết nối tới Linux bridge. Card mạng **eth1.100** này cũng sẽ gắn tới card mạng vật lí eth1. Trong ví dụ này chỉ có một node mạng VLAN100, nên việc kết nối giữa các VMs diễn ra bình thường thông qua Linux bridge.
    
Để kiểm tra thông tin về các bridge cũng như các interface được gắn trên bridge đó, ta sử dụng command line sau:
    
    #brctl show
    bridge name		bridge id			STP enabled		interfaces
	brqXXXX	        8000.000c299ddd0d		no			eth1.100
														tap0
														tap1
														tap2

Trong trường hợp cần nhiều hơn một mạng VLAN,  ứng với mỗi một mạng VLAN sẽ tạo ra một Linux bridge và một card mạng VLAN ảo. Các máy ảo thuộc cùng một mạng VLAN trên cùng một node sẽ kết nối card mạng ảo của máy ảo đó với LinuxBridge quản lý VLAN đó
    
![Imgur](http://i.imgur.com/PfSW5KE.png)

 Thông tin về bridge trong hình trên được thể hiện như sau:
     
     # brctl show
	 bridge name	bridge id		    STP enabled		interfaces
	 brqXXXX	    8000.000c299ddd0d		no			eth1.100
														tap0
														tap1
														
	 bridge name	bridge id		    STP enabled		interfaces
	 brqYYYY	    8000.000a34dcdd30		no			eth1.101
														tap2
																											
														
## 2.3 VXLAN
Khi triển khai mạng VXLAN bằng Linux Bridge, các linux bridge sẽ kết hợp với VXLAN interface để thực hiện các chức năng của một thiết bị VTEP. VXLAN  được đặt tên tương ứng với ID của mạng VXLAN mà nó được triển khai lên.

	#brctl show
    bridge name		bridge id			STP enabled		interfaces
	brqXXXX	        8000.45fdd400934		no			tap0
														vxlan-10
    

Còn tiếp ........









