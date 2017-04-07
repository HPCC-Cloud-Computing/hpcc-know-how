Ở bài viết thứ 3 trong loạt bài này, chúng ta sẽ tìm hiểu cách thức triển khai các mạng nội bộ bằng open vSwitch plugin
#Open vSwitch và triển khai của các loại mạng sử dụng Open vSwitch plugin trong OpenStack
##1 Giới thiệu về Open vSwitch
Open vSwitch là một service cho phép tạo ra và quản lý các đối tượng switch ảo trên môi trường Linux. Tuy cũng sử dụng các switch ảo kết nối với các máy ảo và hệ thống mạng vật lý để thiết lập hệ thống mạng ảo, tuy nhiên cách triển khai mạng ảo bằng Open vSwitch plugin có sự khác biệt so với sử dụng Linux Bridge plugin để triển khai mạng ảo. Chúng ta sẽ thấy sự khác biệt này khi nhìn vào cách sơ đồ chi tiết của các loại mạng khi sử dụng Open vSwitch để triển khai.
Chi tiết về Open vSwitch có thể được tìm thấy ở ```http://openvswitch.org/```
##2 Triển khai các loại mạng cục bộ với Open vSwitch plugin
Khi triển khai các loại mạng cục bộ với Open vSwitch plugin, mô hình chung của các mạng này sẽ là: Ở mỗi một node sẽ có 1 open vswitch ảo đóng vai trò switch trung tâm trong node đó, được đặt tên là br-int. Để kết nối với hệ thống mạng bên ngoài, mỗi một card vật lý sẽ được kết nối với một open vswitch, các switch  này lại nối với switch trung tâm. Node trung tâm kết nối với các máy ảo trong node thông qua một switch ảo khác thuộc loại linux bridge. Như vậy, chúng ta cần hiểu, khi sử dụng Open vSwitch plugin, trong hệ thống mạng ảo sẽ sử dụng 2 loại switch ảo là linux bridge và Open vSwitch. Ở đây, linux  bridge được sử dụng để triển khai các tường lửa cho các máy ảo trong node. Tuy cách thiết kế các mạng ảo đều theo mô hình như trên, nhưng cấu hình các switch sẽ khác nhau tùy thuộc vào loại mạng triển khai. 

Để tìm hiểu xem các switch ảo trong hệ thống mạng ảo được cấu hình như thế nào, chúng ta cần hiểu được một số vấn đề sau:
####Các openvswitch ảo có trong 1 node và các port của một openvswitch ảo
Để kiểm tra được mô hình mạng ảo trong 1 node, bước đầu tiên chúng ta cần kiểm tra được những switch nào đang được triển khai trên node đó.
Ta xét một mô hình mạng trên 1 node, node này có 3 card mạng eth0, eth1, eth2. Trên node triển khai 5 máy ảo, mỗi máy ảo kết nối với switch trung tâm qua 1 linux bridge. Khi đó ta có mô hình của node như sau
![Ovs-3Physic.png](./img/Ovs-3Physic.png)
Kiểm tra các switch ảo, ta thấy được các switch kết nối với các thiết bị khác qua các cổng nào:
```sh
root@compute:/home/cong# ovs-vsctl show
    Bridge br-tun
        fail_mode: secure
        Port br-tun
            Interface br-tun
                type: internal
        Port patch-int
            Interface patch-int
                type: patch
                options: {peer=patch-tun}
        Port "vxlan-0a0a0a0a"
            Interface "vxlan-0a0a0a0a"
                type: vxlan
                options: {df_default="true", in_key=flow, local_ip="10.10.10.11", out_key=flow, remote_ip="10.10.10.10"}
    Bridge br-vlan
        Port "eth2"
            Interface "eth2"
        Port br-vlan
            Interface br-vlan
                type: internal
        Port phy-br-vlan
            Interface phy-br-vlan
                type: patch
                options: {peer=int-br-vlan}
    Bridge br-ex
        Port br-ex
            Interface br-ex
                type: internal
        Port phy-br-ex
            Interface phy-br-ex
                type: patch
                options: {peer=int-br-ex}
        Port "eth1"
            Interface "eth1"
    Bridge br-int
        fail_mode: secure
        Port "qvo1fed3e27-84"
            tag: 3
            Interface "qvo1fed3e27-84"
        Port int-br-provider
            Interface int-br-provider
                type: patch
                options: {peer=phy-br-provider}
        Port "qvo0fbe5cba-9e"
            tag: 4
            Interface "qvo0fbe5cba-9e"
        Port br-int
            Interface br-int
                type: internal
        Port "qvo7be3c33f-f4"
            tag: 2
            Interface "qvo7be3c33f-f4"
        Port int-br-ex
            Interface int-br-ex
                type: patch
                options: {peer=phy-br-ex}
        Port "qvo28447704-dc"
            tag: 3
            Interface "qvo28447704-dc"
        Port patch-tun
            Interface patch-tun
                type: patch
                options: {peer=patch-int}
        Port "qvob362bedf-c1"
            tag: 1
            Interface "qvob362bedf-c1"
        Port int-br-vlan
            Interface int-br-vlan
                type: patch
                options: {peer=phy-br-vlan}
    ovs_version: "2.5.0"
root@compute:/home/cong# brctl show
bridge name			bridge id		STP enabled		interfaces
qbr0fbe5cba-9e		8000.fe163e333534	no			qvb0fbe5cba-9e
														tap0fbe5cba-9e
qbr1fed3e27-84		8000.1207cb57696a	no			qvb1fed3e27-84
														tap1fed3e27-84
qbr28447704-dc		8000.9a6f234eba4f	no			qvb28447704-dc
														tap28447704-dc
qbr7be3c33f-f4		8000.62d8f048c2e5	no			qvb7be3c33f-f4
														tap7be3c33f-f4
qbrb362bedf-c1		8000.9686eedeba00	no			qvbb362bedf-c1
														tapb362bedf-c1

```
Như đã nói ở trên, mỗi card vật lý được triển khai mạng ảo sẽ được triển khai một openvswitch, do đó khi chúng ta sử dụng lệnh ```ovs-vsctl show``` để kiểm tra, chúng ta có thể thấy trên node sẽ có 4 open vswitch, bao gồm 3 switch liên kết với 3 node vật lý và 1 openvswitch trung tâm. Sử dụng lệnh ```brctl show```, chúng ta có thể thấy trên hệ thống có 5 Linux Bridge, mỗi bridge phục vụ cho 1 máy ảo.
1 switch có nhiều cổng để kết nối tới các máy khách, các card vật lý hoặc các switch khác. Khi kiểm tra xem một switch có các cổng nào, chúng ta đồng thời cũng có thể biết được switch đó đang kết nối với các thiết bị nảo qua cổng nào. Ví dụ như khi chúng ta sử dụng các câu lệnh trên để kiểm tra trên 1 node, chúng ta có thể thấy
```sh
 Bridge br-vlan
        Port "eth2"
            Interface "eth2"
        Port br-vlan
            Interface br-vlan
                type: internal
        Port phy-br-vlan
            Interface phy-br-vlan
                type: patch
                options: {peer=int-br-vlan}
```
openvswitch này có 3 port, trong đó có một port là internal, 2 port còn lại, port ```br-vlan``` kết nối trực tiếp tới card vật lý eth2, port ```phy-br-vlan```
nối với openvswitch br-int. Hoặc chúng ta xét openvswitch trung tâm
```sh
    Bridge br-int
        fail_mode: secure
        Port "qvo1fed3e27-84"
            tag: 3
            Interface "qvo1fed3e27-84"
        Port int-br-provider
            Interface int-br-provider
                type: patch
                options: {peer=phy-br-provider}
        Port "qvo0fbe5cba-9e"
            tag: 4
            Interface "qvo0fbe5cba-9e"
        Port br-int
            Interface br-int
                type: internal
        Port "qvo7be3c33f-f4"
            tag: 2
            Interface "qvo7be3c33f-f4"
        Port int-br-ex
            Interface int-br-ex
                type: patch
                options: {peer=phy-br-ex}
        Port "qvo28447704-dc"
            tag: 3
            Interface "qvo28447704-dc"
        Port patch-tun
            Interface patch-tun
                type: patch
                options: {peer=patch-int}
        Port "qvob362bedf-c1"
            tag: 1
            Interface "qvob362bedf-c1"
        Port int-br-vlan
            Interface int-br-vlan
                type: patch
                options: {peer=phy-br-vlan}
```
chúng ta có thể thấy openvswitch này kết nối tới openvswitch br-vlan qua port ```int-br-vlan``` , kết nối với openvswitch br-tun qua port ```patch-tun```, kết nối với openvswitch br-ex qua port ```int-br-ex```. Đồng thời openvswitch trung tâm kết nối với các linux bridge thông qua các port ```qvoxxxx```, tương ứng với chúng là các linux bridge có tên là ```qbrxxxx```

Để có thể xem rõ hơn về các port của một openvswitch, chúng ta có thể dùng lệnh ```ovs-ofctl show <tên openvSwitch>```, ví dụ
```sh
root@compute:/home/cong# ovs-ofctl show br-int
OFPT_FEATURES_REPLY (xid=0x2): dpid:00008abe90f05b41
n_tables:254, n_buffers:256
capabilities: FLOW_STATS TABLE_STATS PORT_STATS QUEUE_STATS ARP_MATCH_IP
actions: output enqueue set_vlan_vid set_vlan_pcp strip_vlan mod_dl_src mod_dl_dst mod_nw_src mod_nw_dst mod_nw_tos mod_tp_src mod_tp_dst
 1(int-br-provider): addr:12:78:40:51:11:b1
     config:     0
     state:      0
     speed: 0 Mbps now, 0 Mbps max
 2(int-br-vlan): addr:6a:22:c4:28:8d:d1
     config:     0
     state:      0
     speed: 0 Mbps now, 0 Mbps max
 3(int-br-ex): addr:86:0d:b4:1c:e7:83
     config:     0
     state:      0
     speed: 0 Mbps now, 0 Mbps max
 4(patch-tun): addr:c2:dd:86:2f:7d:1c
     config:     0
     state:      0
     speed: 0 Mbps now, 0 Mbps max
 5(qvob362bedf-c1): addr:22:f3:c9:17:20:10
     config:     0
     state:      0
     current:    10GB-FD COPPER
     speed: 10000 Mbps now, 0 Mbps max
 7(qvo7be3c33f-f4): addr:ca:c9:72:09:87:e6
     config:     0
     state:      0
     current:    10GB-FD COPPER
     speed: 10000 Mbps now, 0 Mbps max
 8(qvo1fed3e27-84): addr:5a:75:f5:84:81:30
     config:     0
     state:      0
     current:    10GB-FD COPPER
     speed: 10000 Mbps now, 0 Mbps max
 9(qvo0fbe5cba-9e): addr:4a:41:8f:03:d7:2a
     config:     0
     state:      0
     current:    10GB-FD COPPER
     speed: 10000 Mbps now, 0 Mbps max
 10(qvo28447704-dc): addr:46:9a:8c:4e:f8:48
     config:     0
     state:      0
     current:    10GB-FD COPPER
     speed: 10000 Mbps now, 0 Mbps max
 LOCAL(br-int): addr:8a:be:90:f0:5b:41
     config:     PORT_DOWN
     state:      LINK_DOWN
     speed: 0 Mbps now, 0 Mbps max
OFPT_GET_CONFIG_REPLY (xid=0x4): frags=normal miss_send_len=0

```

Như vậy, chúng ta có thể nhìn thấy được các switch ảo có trong một node và sự kết nối giữa chúng với các thiết bị khác qua các port như thế nào. Điều tiếp theo chúng ta cần xem xét là các switch ảo trên xử lý các gói tin vào ra bằng cách nào với từng loại mạng cục bộ. Để giải quyết việc này, các openvswitch sử dụng một cơ chế để xử lý các gói dữ liệu tương ứng với các thông tin định danh của gói dữ liệu đó, được gọi là ```flow-rules```. Do cơ chế hoạt động của openVswitch, cách xử lý thông tin định danh của open vSwitch có sự khác biệt so với hoạt động của các switch thông thường. Chúng ta sẽ làm rõ điều này hơn khi đi vào flow rules của từng loại mạng cụ thể. Ở đây chúng ta thử xem xét xem flow rules của một open vSwitch có dạng như thế nào. Sử dụng câu lệnh ```ovs-ofctl dump-flows <bridge>``` để xem flow rule của một bridge nào đó. Ở đây chúng ta thử xem flow rule của br-int

```sh
ovs-ofctl dump-flows --rsort  br-int
 cookie=0xb595b9f297d967e0, duration=476.373s, table=0, n_packets=78, n_bytes=8679, priority=3,in_port=2,dl_vlan=4 actions=mod_vlan_vid:1,NORMAL
 cookie=0xb595b9f297d967e0, duration=472.277s, table=0, n_packets=83, n_bytes=9209, priority=3,in_port=2,dl_vlan=104 actions=mod_vlan_vid:2,NORMAL
 cookie=0xb595b9f297d967e0, duration=471.938s, table=0, n_packets=61, n_bytes=6960, priority=3,in_port=3,dl_vlan=101 actions=mod_vlan_vid:3,NORMAL
 cookie=0xb595b9f297d967e0, duration=532.100s, table=0, n_packets=33, n_bytes=4782, priority=2,in_port=2 actions=drop
 cookie=0xb595b9f297d967e0, duration=529.854s, table=0, n_packets=610, n_bytes=42964, priority=2,in_port=3 actions=drop

```
Lưu ý là khi dữ liệu đi vào br-int sẽ được xử lý theo mức độ ưu tiên, nghĩa là dữ liệu nếu hợp với nhiều rule, thì rule có mức ưu tiên (priority) cao nhất sẽ được áp dụng để xử lý dữ liệu, còn các rule khác sẽ không được áp dụng

Một điểm quan trọng mà chúng ta cần lưu ý khi tìm hiểu hệ thống mạng ảo sử dụng open vSwitch, đó là vì trên một node có thể tồn tại rất nhiều máy ảo thuộc nhiều mạng nội bộ khác nhau, nên hệ thống các open vSwitch ảo cần có cơ chế quản lý các máy này, tức là cần phải có cơ chế để xác định máy ảo nào thuộc mạng nào. Để làm được điều này, bridge trung tâm sẽ thay thế mỗi một mạng  được triển khai trên hệ thống bằng một mạng VLAN nội bộ dùng riêng trong node đó. Ví dụ nếu trên 1 node triển khai 2 mạng VXLAN, 3 mạng VLAN, 1 mạng Flat thì sẽ có 6 mạng VLAN nội bộ trên node đó. Việc thay thế các gói tin đi từ bên ngoài vào thành các gói tin thuộc các mạng nội bộ bên trong  node hoặc chuyển các gói tin thuộc các mạng nội bộ bên trong node thành gói tin thuộc các mạng bên ngoài sẽ được thực hiện bằng các thao tác thay thế thẻ định danh. Chúng ta sẽ tìm hiểu các quy tắc thay thế thẻ định danh khi xem xét từng loại mạng ở phần dưới đây. 

Sau khi nắm được các khái niệm cơ bản trong một hệ thống mạng ảo sử dụng openvswitch plugin, chúng ta sẽ xem xem các khái niệm này được thể hiện như thế nào trong các loại mạng cục bộ.

###2.1 Mạng VLAN
Loại mạng đầu tiên chúng ta tìm hiểu là mạng VLAN. Để đơn giản hóa, chúng ta sẽ sử dụng một cấu hình đơn giản trên 1 node gồm 1 card vật lý và 2 mạng VLAN ảo, mỗi mạng VLAN ảo triển khai 1 máy ảo. Chúng ta cùng khảo sát xem luồng dữ liệu di chuyển trong hệ thống như thế nào.
Đầu tiên là sơ đồ khối trong hệ thống
![ovs-vlan.png](./img/ovs-vlan.png)
Kiểm tra các port có trên br-int
```sh
root@compute:/home/cong# ovs-ofctl show br-int
OFPT_FEATURES_REPLY (xid=0x2): dpid:00008abe90f05b41
n_tables:254, n_buffers:256
capabilities: FLOW_STATS TABLE_STATS PORT_STATS QUEUE_STATS ARP_MATCH_IP
actions: output enqueue set_vlan_vid set_vlan_pcp strip_vlan mod_dl_src mod_dl_dst mod_nw_src mod_nw_dst mod_nw_tos mod_tp_src mod_tp_dst
 2(int-br-vlan): addr:26:20:af:8a:70:59
     config:     0
     state:      0
     speed: 0 Mbps now, 0 Mbps max
 10(qvoa1335f68-ff): addr:76:64:36:3e:57:4a
     config:     0
     state:      0
     current:    10GB-FD COPPER
     speed: 10000 Mbps now, 0 Mbps max
 11(qvo035618f7-c2): addr:e6:7f:b1:c5:8f:30
     config:     0
     state:      0
     current:    10GB-FD COPPER
     speed: 10000 Mbps now, 0 Mbps max
 LOCAL(br-int): addr:8a:be:90:f0:5b:41
     config:     PORT_DOWN
     state:      LINK_DOWN
     speed: 0 Mbps now, 0 Mbps max

```
như vậy ta thấy 2 instance gắn với br-int qua port 10 và 11, switch vlan gắn với br-int qua port 2

Kiểm tra các port có trên br-vlan
```sh
root@compute:/home/cong# ovs-ofctl show br-vlan
OFPT_FEATURES_REPLY (xid=0x2): dpid:0000000c299ddd17
n_tables:254, n_buffers:256
capabilities: FLOW_STATS TABLE_STATS PORT_STATS QUEUE_STATS ARP_MATCH_IP
actions: output enqueue set_vlan_vid set_vlan_pcp strip_vlan mod_dl_src mod_dl_dst mod_nw_src mod_nw_dst mod_nw_tos mod_tp_src mod_tp_dst
 2(phy-br-vlan): addr:ee:89:e8:e4:a5:73
     config:     0
     state:      0
     speed: 0 Mbps now, 0 Mbps max
 3(eth2): addr:00:0c:29:9d:dd:17
     config:     0
     state:      0
     current:    1GB-FD COPPER AUTO_NEG
     advertised: 10MB-HD 10MB-FD 100MB-HD 100MB-FD 1GB-FD COPPER AUTO_NEG
     supported:  10MB-HD 10MB-FD 100MB-HD 100MB-FD 1GB-FD COPPER AUTO_NEG
     speed: 1000 Mbps now, 1000 Mbps max
 LOCAL(br-vlan): addr:00:0c:29:9d:dd:17
     config:     PORT_DOWN
     state:      LINK_DOWN
     speed: 0 Mbps now, 0 Mbps max
```
như vậy ta thấy br-vlan gắn với br-int qua port 2, gắn với card vật lý qua port 3.

Ở đây trên hệ thống đang triển khai 2 vlan với vlan1 có id là 101, vlan2 có id là 102. Chúng ta khảo sát luồng dữ liệu đi qua br-int
```sh
root@compute:/home/cong# ovs-ofctl dump-flows --rsort  br-int
 cookie=0xb595b9f297d967e0, duration=1786.844s, table=0, n_packets=0, n_bytes=0, priority=10,icmp6,in_port=10,icmp_type=136 actions=resubmit(,24)
 cookie=0xb595b9f297d967e0, duration=1736.716s, table=0, n_packets=0, n_bytes=0, priority=10,icmp6,in_port=11,icmp_type=136 actions=resubmit(,24)
 cookie=0xb595b9f297d967e0, duration=1786.514s, table=0, n_packets=11, n_bytes=462, priority=10,arp,in_port=10 actions=resubmit(,24)
 cookie=0xb595b9f297d967e0, duration=1736.517s, table=0, n_packets=12, n_bytes=504, priority=10,arp,in_port=11 actions=resubmit(,24)
 cookie=0xb595b9f297d967e0, duration=1787.096s, table=0, n_packets=586, n_bytes=102793, priority=9,in_port=10 actions=resubmit(,25)
 cookie=0xb595b9f297d967e0, duration=1736.926s, table=0, n_packets=590, n_bytes=103101, priority=9,in_port=11 actions=resubmit(,25)
 cookie=0xb595b9f297d967e0, duration=1791.575s, table=0, n_packets=75, n_bytes=8430, priority=3,in_port=2,dl_vlan=101 actions=mod_vlan_vid:5,NORMAL
 cookie=0xb595b9f297d967e0, duration=1738.401s, table=0, n_packets=78, n_bytes=8698, priority=3,in_port=2,dl_vlan=102 actions=mod_vlan_vid:6,NORMAL
 cookie=0xb595b9f297d967e0, duration=12839.784s, table=0, n_packets=43, n_bytes=5720, priority=2,in_port=2 actions=drop
 cookie=0xb595b9f297d967e0, duration=1786.961s, table=24, n_packets=0, n_bytes=0,priority=2,icmp6,in_port=10,icmp_type=136,nd_target=fe80::f816:3eff:fe9a:654c actions=NORMAL
 cookie=0xb595b9f297d967e0, duration=1736.815s, table=24, n_packets=0, n_bytes=0,priority=2,icmp6,in_port=11,icmp_type=136,nd_target=fe80::f816:3eff:fe13:70ac actions=NORMAL
 cookie=0xb595b9f297d967e0, duration=1786.734s, table=24, n_packets=11, n_bytes=462, priority=2,arp,in_port=10,arp_spa=10.10.40.3 actions=resubmit(,25)
 cookie=0xb595b9f297d967e0, duration=1736.620s, table=24, n_packets=12, n_bytes=504, priority=2,arp,in_port=11,arp_spa=10.10.30.3 actions=resubmit(,25)
 cookie=0xb595b9f297d967e0, duration=1787.542s, table=25, n_packets=120, n_bytes=11326, priority=2,in_port=10,dl_src=fa:16:3e:9a:65:4c actions=NORMAL
 cookie=0xb595b9f297d967e0, duration=1737.153s, table=25, n_packets=123, n_bytes=11564, priority=2,in_port=11,dl_src=fa:16:3e:13:70:ac actions=NORMAL
 cookie=0xb595b9f297d967e0, duration=12841.451s, table=0, n_packets=336, n_bytes=48047, priority=0 actions=NORMAL
 cookie=0xb595b9f297d967e0, duration=12841.341s, table=23, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0xb595b9f297d967e0, duration=12841.224s, table=24, n_packets=0, n_bytes=0, priority=0 actions=drop
```
và luồng dữ liệu đi qua br-vlan
```
root@compute:/home/cong# ovs-ofctl dump-flows --rsort  br-vlan
 cookie=0xa82fee3418e6cbad, duration=687.647s, table=0, n_packets=120, n_bytes=11326, priority=4,in_port=2,dl_vlan=5 actions=mod_vlan_vid:101,NORMAL
 cookie=0xa82fee3418e6cbad, duration=634.468s, table=0, n_packets=119, n_bytes=11284, priority=4,in_port=2,dl_vlan=6 actions=mod_vlan_vid:102,NORMAL
 cookie=0xa82fee3418e6cbad, duration=11735.531s, table=0, n_packets=170, n_bytes=30575, priority=2,in_port=2 actions=drop
 cookie=0xa82fee3418e6cbad, duration=11736.502s, table=0, n_packets=359, n_bytes=41687, priority=0 actions=NORMAL
```
Đầu tiên, chúng ta xét luồng dữ liệu đi từ ngoài vào. Dữ liệu từ bên ngoài sẽ đi vào card eth2 đầu tiên, sau đó đi tới br-vlan thông qua port 3. Tại đây chúng ta tra bảng flow-rule của br-vlan. Do in_port của dữ liệu là 3 (đi qua port 3), do đó 3 rule đàue tiên có in_port=2 sẽ không được áp dụng. 
```sh
root@compute:/home/cong# ovs-ofctl dump-flows --rsort  br-vlan
 cookie=0xa82fee3418e6cbad, duration=687.647s, table=0, n_packets=120, n_bytes=11326, priority=4,in_port=2,dl_vlan=5 actions=mod_vlan_vid:101,NORMAL
 cookie=0xa82fee3418e6cbad, duration=634.468s, table=0, n_packets=119, n_bytes=11284, priority=4,in_port=2,dl_vlan=6 actions=mod_vlan_vid:102,NORMAL
 cookie=0xa82fee3418e6cbad, duration=11735.531s, table=0, n_packets=170, n_bytes=30575, priority=2,in_port=2 actions=drop
```
Dữ liệu đi vào sẽ được xử lý bằng rule cuối cùng
```sh
 cookie=0xa82fee3418e6cbad, duration=11736.502s, table=0, n_packets=359, n_bytes=41687, priority=0 actions=NORMAL
```
ta thấy  ```actions=NORMAL```, do đó dữ liệu được chuyển tiếp lên br-int qua port 2
tại br-vlan, ta tìm các rule ứng với ```in_port = 2``` (hoặc không có in_port, tức là chấp nhận mọi luồng dữ liệu). Ở đây các rule tương ứng là:
```sh
 cookie=0xb595b9f297d967e0, duration=1791.575s, table=0, n_packets=75, n_bytes=8430, priority=3,in_port=2,dl_vlan=101 actions=mod_vlan_vid:5,NORMAL
 cookie=0xb595b9f297d967e0, duration=1738.401s, table=0, n_packets=78, n_bytes=8698, priority=3,in_port=2,dl_vlan=102 actions=mod_vlan_vid:6,NORMAL
 cookie=0xb595b9f297d967e0, duration=12839.784s, table=0, n_packets=43, n_bytes=5720, priority=2,in_port=2 actions=drop
```
3 rule này cho thấy dữ liệu đi từ br-vlan vào  được br-int xử lý như thế nào. Với các gói tin của mạng vlan 101 ```dl_vlan=101```, br-int sẽ chuyển định danh gói tin từ vid=101 bên ngoài thành định danh bên trong node, gói tin sẽ là gói tin của mạng vlan nội bộ bên trong node có id=5 ```actions=mod_vlan_vid:5```
tương tự các gói tin từ mạng vlan102 sẽ được chuyển thành các gói tin của mạng vlan nội bộ có id =6. Các gói tin không phải thuộc 2 mạng vlan này sẽ bị hủy bỏ ``drop```. Ở đây chúng ta thấy thao tác ánh xạ gói tin của các mạng bên ngoài thành gói tin của các mạng nội bộ bên trong như đã trình bày ở phần trước.
Sau đó, như ta đã nói, 1 máy ảo trên node sẽ thuộc mạng vlan 101, 1 máy ảo trên node sẽ thuộc mạng vlan 102. Do đó khi được gắn vào br-int, máy ảo ở mạng 101 sẽ được br-int quản lý với thẻ vlan nội bộ là ```id=5``` và máy ảo ở mạng 102 sẽ được br-int quản lý với thẻ vlan nội bộ là ```id=6```. Do đó sau khi các gói tin được chuyển thành các gói tin nội bộ, các gói tin sẽ được forward về đúng các máy đích dựa trên bảng định tuyến cho các gói tin vlan nội bộ trong br-int (có thể ngầm hiểu trước khi đi vào máy ảo, thông tin về mạng nội bộ sẽ được br-int loại bỏ để gói tin chuyển tiếp tới là gói tin nguyên bản.)

Khi dữ liệu đi từ các máy ảo ra bên ngoài, đầu tiên chúng sẽ đi vào br-int thông qua các port 10 và 11. Các gói tin này sau khi đi vào sẽ được br-int gán nhãn vlan nội bộ, do như nói ở phần trước các máy ảo trên mạng được br-int sắp xếp vào các mạng vlan nội bộ để quản lý.
Xét các flow rule tương ứng với 2 port 10 và 11, các rules đầu tiên ứng với các gói tin thuộc các giao thức ICMP và ARP
```sh
 cookie=0xb595b9f297d967e0, duration=1786.844s, table=0, n_packets=0, n_bytes=0, priority=10,icmp6,in_port=10,icmp_type=136 actions=resubmit(,24)
 cookie=0xb595b9f297d967e0, duration=1736.716s, table=0, n_packets=0, n_bytes=0, priority=10,icmp6,in_port=11,icmp_type=136 actions=resubmit(,24)
 cookie=0xb595b9f297d967e0, duration=1786.514s, table=0, n_packets=11, n_bytes=462, priority=10,arp,in_port=10 actions=resubmit(,24)
 cookie=0xb595b9f297d967e0, duration=1736.517s, table=0, n_packets=12, n_bytes=504, priority=10,arp,in_port=11 actions=resubmit(,24)

```
chúng ta có thể thấy các rule này sẽ forward các gói tin này tới xử lý theo các rule tương ứng với table 24. Các rule tương ứng với các gói tin bình thường 
```sh
 cookie=0xb595b9f297d967e0, duration=1787.096s, table=0, n_packets=586, n_bytes=102793, priority=9,in_port=10 actions=resubmit(,25)
 cookie=0xb595b9f297d967e0, duration=1736.926s, table=0, n_packets=590, n_bytes=103101, priority=9,in_port=11 actions=resubmit(,25)
```
được forward tới xử lý theo các rule tương ứng với table 25. Quan sát các rule tương ứng với table 24 và table 25:
```sh
 cookie=0xb595b9f297d967e0, duration=1786.961s, table=24, n_packets=0, n_bytes=0,priority=2,icmp6,in_port=10,icmp_type=136,nd_target=fe80::f816:3eff:fe9a:654c actions=NORMAL
 cookie=0xb595b9f297d967e0, duration=1736.815s, table=24, n_packets=0, n_bytes=0,priority=2,icmp6,in_port=11,icmp_type=136,nd_target=fe80::f816:3eff:fe13:70ac actions=NORMAL
 cookie=0xb595b9f297d967e0, duration=1786.734s, table=24, n_packets=11, n_bytes=462, priority=2,arp,in_port=10,arp_spa=10.10.40.3 actions=resubmit(,25)
 cookie=0xb595b9f297d967e0, duration=1736.620s, table=24, n_packets=12, n_bytes=504, priority=2,arp,in_port=11,arp_spa=10.10.30.3 actions=resubmit(,25)
 cookie=0xb595b9f297d967e0, duration=1787.542s, table=25, n_packets=120, n_bytes=11326, priority=2,in_port=10,dl_src=fa:16:3e:9a:65:4c actions=NORMAL
 cookie=0xb595b9f297d967e0, duration=1737.153s, table=25, n_packets=123, n_bytes=11564, priority=2,in_port=11,dl_src=fa:16:3e:13:70:ac actions=NORMAL
```
ta thấy các gói tin xuất phát từ port 10 và port 11 đều được xử lý bình thường. Như vậy khi các gói tin đi từ các máy ảo ra bên ngoài thì br-int sẽ làm nhiệm vụ gắn thẻ vlan nội bộ cho các gói tin rồi chuyển chúng tới br-vlan.
Khi các gói tin đi vào br-vlan, chúng đi vào từ port 2, chúng ta xét các rule tương ứng với in_port = 2
```sh
cookie=0xa82fee3418e6cbad, duration=687.647s, table=0, n_packets=120, n_bytes=11326, priority=4,in_port=2,dl_vlan=5 actions=mod_vlan_vid:101,NORMAL
 cookie=0xa82fee3418e6cbad, duration=634.468s, table=0, n_packets=119, n_bytes=11284, priority=4,in_port=2,dl_vlan=6 actions=mod_vlan_vid:102,NORMAL
 cookie=0xa82fee3418e6cbad, duration=11735.531s, table=0, n_packets=170, n_bytes=30575, priority=2,in_port=2 actions=drop
```
Chúng ta có thể thấy các gói tin đi từ br-int vào br-vlan sẽ được thay thế các thẻ vlan nội bộ 5 và 6 bằng các thẻ vlan của các mạng bên ngoài là 101 và 102. Sau khi thay thế thẻ vlan xong,các gói tin cùng thông tin định danh vlan của các mạng bên ngoài sẽ được br-vlan chuyển ra hệ thống mạng vật lý để tới đích thông qua card mạng eth2.

Các chức năng khác của br-int như làm thế nào để xác định gói tin đi tới máy nào trong một mạng vlan nội bộ, vvv... chúng ta sẽ tìm hiểu trong 1 bài viết khác. Ở đây chúng ta chủ yếu tìm hiểu quá trình dữ liệu vào và ra các switch sẽ được gắn thẻ định danh như thế nào.
###2.2 Mạng Flat
Mạng Flat là mạng không sử dụng thêm bất kỳ thông tin gì để định danh gói tin khi truyền gói tin đi trong hệ thống mạng. Chính vì vậy mà một card mạng vật lý chỉ có thể triển khai nhiều nhất một mạng Flat. Tuy nhiên, sau khi triển khai mạng Flat, chúng ta vẫn có thể triển khai thêm nhiều mạng VLAN trên card mạng Flat đó. Mặt khác, như ở phần trước, chúng ta đã biết các gói tin ở mạng bên ngoài sẽ được ánh xạ vào thành các gói tin của các mạng vlan nội bộ bên trong 1 node. Quy trình xử lý các gói tin của mạng Flat cũng như vậy.
Ta xét một hệ thống mạng triển khai  1 mạng Flat và 2 mạng VLAN trên 1 card mạng vật lý. Vẫn với sơ đồ các switch như trên, hệ thống xử lý thêm các luồng dữ liệu của mạng flat
![ovs-vlan.png](./img/ovs-vlan.png)

Các port trên br-int:
```sh
root@compute:/home/cong# ovs-ofctl show br-int
OFPT_FEATURES_REPLY (xid=0x2): dpid:00008abe90f05b41
n_tables:254, n_buffers:256
capabilities: FLOW_STATS TABLE_STATS PORT_STATS QUEUE_STATS ARP_MATCH_IP
actions: output enqueue set_vlan_vid set_vlan_pcp strip_vlan mod_dl_src mod_dl_dst mod_nw_src mod_nw_dst mod_nw_tos mod_tp_src mod_tp_dst
 2(int-br-vlan): addr:26:20:af:8a:70:59
     config:     0
     state:      0
     speed: 0 Mbps now, 0 Mbps max

 12(qvo23d0f2c7-c2): addr:b2:cf:8c:2d:1b:f2
     config:     0
     state:      0
     current:    10GB-FD COPPER
     speed: 10000 Mbps now, 0 Mbps max
 13(qvoe9feecdf-5d): addr:22:dc:d3:f6:ac:72
     config:     0
     state:      0
     current:    10GB-FD COPPER
     speed: 10000 Mbps now, 0 Mbps max
 14(qvodbe633c6-33): addr:6a:10:94:e7:3b:ac
     config:     0
     state:      0
     current:    10GB-FD COPPER
     speed: 10000 Mbps now, 0 Mbps max
 LOCAL(br-int): addr:8a:be:90:f0:5b:41
     config:     PORT_DOWN
     state:      LINK_DOWN
     speed: 0 Mbps now, 0 Mbps max
OFPT_GET_CONFIG_REPLY (xid=0x4): frags=normal miss_send_len=0
```
Ta có port 12 kết nối tới máy ảo của mạng flat, port 13 kết nối tới máy ảo của mạng vlan 101, port 14 kết nối tới máy ảo của mạng vlan102
Các port trên br-vlan:
```sh
root@compute:/home/cong# ovs-ofctl show br-vlan
OFPT_FEATURES_REPLY (xid=0x2): dpid:0000000c299ddd17
n_tables:254, n_buffers:256
capabilities: FLOW_STATS TABLE_STATS PORT_STATS QUEUE_STATS ARP_MATCH_IP
actions: output enqueue set_vlan_vid set_vlan_pcp strip_vlan mod_dl_src mod_dl_dst mod_nw_src mod_nw_dst mod_nw_tos mod_tp_src mod_tp_dst
 2(phy-br-vlan): addr:ee:89:e8:e4:a5:73
     config:     0
     state:      0
     speed: 0 Mbps now, 0 Mbps max
 3(eth2): addr:00:0c:29:9d:dd:17
     config:     0
     state:      0
     current:    1GB-FD COPPER AUTO_NEG
     advertised: 10MB-HD 10MB-FD 100MB-HD 100MB-FD 1GB-FD COPPER AUTO_NEG
     supported:  10MB-HD 10MB-FD 100MB-HD 100MB-FD 1GB-FD COPPER AUTO_NEG
     speed: 1000 Mbps now, 1000 Mbps max
 LOCAL(br-vlan): addr:00:0c:29:9d:dd:17
     config:     PORT_DOWN
     state:      LINK_DOWN
     speed: 0 Mbps now, 0 Mbps max
OFPT_GET_CONFIG_REPLY (xid=0x4): frags=normal miss_send_len=0
```
flow rules trên br-int
```sh
root@compute:/home/cong# ovs-ofctl dump-flows --rsort  br-int
 cookie=0xb595b9f297d967e0, duration=341.623s, table=0, n_packets=0, n_bytes=0, priority=10,icmp6,in_port=12,icmp_type=136 actions=resubmit(,24)
 cookie=0xb595b9f297d967e0, duration=256.519s, table=0, n_packets=0, n_bytes=0, priority=10,icmp6,in_port=13,icmp_type=136 actions=resubmit(,24)
 cookie=0xb595b9f297d967e0, duration=240.286s, table=0, n_packets=0, n_bytes=0, priority=10,icmp6,in_port=14,icmp_type=136 actions=resubmit(,24)
 cookie=0xb595b9f297d967e0, duration=341.384s, table=0, n_packets=11, n_bytes=462, priority=10,arp,in_port=12 actions=resubmit(,24)
 cookie=0xb595b9f297d967e0, duration=256.285s, table=0, n_packets=10, n_bytes=420, priority=10,arp,in_port=13 actions=resubmit(,24)
 cookie=0xb595b9f297d967e0, duration=240.014s, table=0, n_packets=11, n_bytes=462, priority=10,arp,in_port=14 actions=resubmit(,24)
 cookie=0xb595b9f297d967e0, duration=341.851s, table=0, n_packets=209, n_bytes=30069, priority=9,in_port=12 actions=resubmit(,25)
 cookie=0xb595b9f297d967e0, duration=256.849s, table=0, n_packets=211, n_bytes=30679, priority=9,in_port=13 actions=resubmit(,25)
 cookie=0xb595b9f297d967e0, duration=240.582s, table=0, n_packets=210, n_bytes=30715, priority=9,in_port=14 actions=resubmit(,25)
 cookie=0xb595b9f297d967e0, duration=344.561s, table=0, n_packets=76, n_bytes=8243, priority=3,in_port=2,vlan_tci=0x0000 actions=mod_vlan_vid:7,NORMAL
 cookie=0xb595b9f297d967e0, duration=258.775s, table=0, n_packets=74, n_bytes=8372, priority=3,in_port=2,dl_vlan=101 actions=mod_vlan_vid:8,NORMAL
 cookie=0xb595b9f297d967e0, duration=243.046s, table=0, n_packets=75, n_bytes=8436, priority=3,in_port=2,dl_vlan=102 actions=mod_vlan_vid:9,NORMAL
 cookie=0xb595b9f297d967e0, duration=17537.651s, table=0, n_packets=49, n_bytes=6193, priority=2,in_port=2 actions=drop
 cookie=0xb595b9f297d967e0, duration=17535.405s, table=0, n_packets=8286, n_bytes=504116, priority=2,in_port=3 actions=drop
 cookie=0xb595b9f297d967e0, duration=341.743s, table=24, n_packets=0, n_bytes=0, priority=2,icmp6,in_port=12,icmp_type=136,nd_target=fe80::f816:3eff:fe80:dbc8 actions=NORMAL
 cookie=0xb595b9f297d967e0, duration=256.694s, table=24, n_packets=0, n_bytes=0, priority=2,icmp6,in_port=13,icmp_type=136,nd_target=fe80::f816:3eff:fe1d:43ed actions=NORMAL
 cookie=0xb595b9f297d967e0, duration=240.433s, table=24, n_packets=0, n_bytes=0, priority=2,icmp6,in_port=14,icmp_type=136,nd_target=fe80::f816:3eff:fe93:1d4f actions=NORMAL
 cookie=0xb595b9f297d967e0, duration=341.505s, table=24, n_packets=11, n_bytes=462, priority=2,arp,in_port=12,arp_spa=10.10.25.3 actions=resubmit(,25)
 cookie=0xb595b9f297d967e0, duration=256.384s, table=24, n_packets=10, n_bytes=420, priority=2,arp,in_port=13,arp_spa=10.10.30.3 actions=resubmit(,25)
 cookie=0xb595b9f297d967e0, duration=240.152s, table=24, n_packets=11, n_bytes=462, priority=2,arp,in_port=14,arp_spa=10.10.40.3 actions=resubmit(,25)
 cookie=0xb595b9f297d967e0, duration=342.073s, table=25, n_packets=120, n_bytes=11326, priority=2,in_port=12,dl_src=fa:16:3e:80:db:c8 actions=NORMAL
 cookie=0xb595b9f297d967e0, duration=257.121s, table=25, n_packets=119, n_bytes=11284, priority=2,in_port=13,dl_src=fa:16:3e:1d:43:ed actions=NORMAL
 cookie=0xb595b9f297d967e0, duration=240.840s, table=25, n_packets=120, n_bytes=11326, priority=2,in_port=14,dl_src=fa:16:3e:93:1d:4f actions=NORMAL
 cookie=0xb595b9f297d967e0, duration=17539.318s, table=0, n_packets=391, n_bytes=58664, priority=0 actions=NORMAL
 cookie=0xb595b9f297d967e0, duration=17539.208s, table=23, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0xb595b9f297d967e0, duration=17539.091s, table=24, n_packets=0, n_bytes=0, priority=0 actions=drop
```
flow rules trên br-vlan
```sh
root@compute:/home/cong# ovs-ofctl dump-flows --rsort  br-vlan
 cookie=0xa82fee3418e6cbad, duration=382.007s, table=0, n_packets=120, n_bytes=11326, priority=4,in_port=2,dl_vlan=7 actions=strip_vlan,NORMAL
 cookie=0xa82fee3418e6cbad, duration=296.239s, table=0, n_packets=119, n_bytes=11284, priority=4,in_port=2,dl_vlan=8 actions=mod_vlan_vid:101,NORMAL
 cookie=0xa82fee3418e6cbad, duration=280.538s, table=0, n_packets=120, n_bytes=11326, priority=4,in_port=2,dl_vlan=9 actions=mod_vlan_vid:102,NORMAL
 cookie=0xa82fee3418e6cbad, duration=17574.771s, table=0, n_packets=225, n_bytes=41192, priority=2,in_port=2 actions=drop
 cookie=0xa82fee3418e6cbad, duration=17575.742s, table=0, n_packets=663, n_bytes=75439, priority=0 actions=NORMAL
```
Ở đây chúng ta quan tâm chủ yếu tới luồng dữ liệu của mạng flat.
Đầu tiên là luồng dữ liệu của mạng flat đi vào node. Luồng dữ liệu này sẽ đi qua card mạng vật lý eth2 vào br-vlan qua port 3. Chúng ta có thể thấy tương tự như vlan, luồng dữ liệu này được br-vlan xử lý với action là NORMAL và được chuyển tiếp lên br-int thông qua port 2. 

Tại br-int luồng dữ liệu này được xử lý với in_port=2.Xét các flow rules liên quan tới in_port=2
```sh
 cookie=0xb595b9f297d967e0, duration=344.561s, table=0, n_packets=76, n_bytes=8243, priority=3,in_port=2,vlan_tci=0x0000 actions=mod_vlan_vid:7,NORMAL
 cookie=0xb595b9f297d967e0, duration=258.775s, table=0, n_packets=74, n_bytes=8372, priority=3,in_port=2,dl_vlan=101 actions=mod_vlan_vid:8,NORMAL
 cookie=0xb595b9f297d967e0, duration=243.046s, table=0, n_packets=75, n_bytes=8436, priority=3,in_port=2,dl_vlan=102 actions=mod_vlan_vid:9,NORMAL
```
tại đây rule đầu tiên chính là rule để xử lý luồng dữ liệu của mạng flat đi vào node (do có vlan_tci=0x0000). Chúng ta có thể thấy br-int ánh xạ gói tin của mạng flat thành gói tin vlan của mạng nội bộ với id là 7. Cách xử lý này phản ánh quy ước của open vswitch là ánh xạ mọi gói tin thuộc các mạng bên ngoài thành gói tinc của các mạng nội bộ trong node.

Các máy thuộc mạng flat này sẽ được quản lý bởi br-int với góc nhìn của mạng vlan nội bộ với id =7, do đó khi sau khi chuyển đổi gói tin thành gói tin nội bộ với id =7, gói tin sẽ được chuyển tới đúng địa chỉ máy ảo cần đến trên mạng nội bộ có id =7. (có thể ngầm hiểu trước khi đi vào máy ảo, thông tin về mạng nội bộ sẽ được br-int loại bỏ để gói tin chuyển tiếp tới là gói tin nguyên bản.)

Xét luồng dữ liệu của máy thuộc mạng flat đi từ máy ảo trong node ra bên ngoài. máy ảo nối với br-int thông qua port 12. Chúng ta xét các rule tương ứng với in_port=12:
```sh
cookie=0xb595b9f297d967e0, duration=341.623s, table=0, n_packets=0, n_bytes=0, priority=10,icmp6,in_port=12,icmp_type=136 actions=resubmit(,24)
 cookie=0xb595b9f297d967e0, duration=341.384s, table=0, n_packets=11, n_bytes=462, priority=10,arp,in_port=12 actions=resubmit(,24)
 cookie=0xb595b9f297d967e0, duration=341.851s, table=0, n_packets=209, n_bytes=30069, priority=9,in_port=12 actions=resubmit(,25)
cookie=0xb595b9f297d967e0, duration=341.743s, table=24, n_packets=0, n_bytes=0, priority=2,icmp6,in_port=12,icmp_type=136,nd_target=fe80::f816:3eff:fe80:dbc8 actions=NORMAL
 cookie=0xb595b9f297d967e0, duration=341.505s, table=24, n_packets=11, n_bytes=462, priority=2,arp,in_port=12,arp_spa=10.10.25.3 actions=resubmit(,25)
 cookie=0xb595b9f297d967e0, duration=342.073s, table=25, n_packets=120, n_bytes=11326, priority=2,in_port=12,dl_src=fa:16:3e:80:db:c8 actions=NORMAL
```
tất cả các rule này đều xử lý gói tin theo chế độ NORMAL, tức là khi gói tin của máy ảo thuộc mạng flat đi vào br-int qua port 12 sẽ được gắn thông tin định danh nội bộ  id = 7 rồi chuyển tiếp tới đích dựa trên bảng định tuyến của mạng nội bộ id=7. Do gói tin đang đi ra bên ngoài nên gói tin sẽ được chuyển tiếp tới br-vlan qua in_port=2. Xét các flowrule liên quan tới in_port=2 trên br-vlan:
```
 cookie=0xa82fee3418e6cbad, duration=382.007s, table=0, n_packets=120, n_bytes=11326, priority=4,in_port=2,dl_vlan=7 actions=strip_vlan,NORMAL
 cookie=0xa82fee3418e6cbad, duration=296.239s, table=0, n_packets=119, n_bytes=11284, priority=4,in_port=2,dl_vlan=8 actions=mod_vlan_vid:101,NORMAL
 cookie=0xa82fee3418e6cbad, duration=280.538s, table=0, n_packets=120, n_bytes=11326, priority=4,in_port=2,dl_vlan=9 actions=mod_vlan_vid:102,NORMAL
```
ta thấy gói tin của mạng flat sẽ được đánh số định danh nội bộ là vlan=7, do đó rule đầu tiên được áp dụng vào gói tin. Rule này thực hiện việc gỡ bỏ thông tin định danh nội bộ của gói tin (action =strip_vlan). Do đó khi đi ra ngoài hệ thống mạng vật lý, gói tin không còn thông tin định danh vlan nội bộ trong node nữa mà trở thành một gói tin l2 nguyên bản như lúc bắt đầu gửi từ máy ảo.

###2.3 Mạng VXLAN
Chúng ta xem xét mạng VXLAN dưới một cấu hình đơn giản, gồm 1 card mạng eth0, một br-tun kết nối với card mạng eth0 và một máy ảo triển khai trên 1 mạng VXLAN có id = 101

Các cổng của bridge br-tun
```sh
root@compute:/home/cong# ovs-ofctl show br-tun
OFPT_FEATURES_REPLY (xid=0x2): dpid:0000be960c3d6448
n_tables:254, n_buffers:256
capabilities: FLOW_STATS TABLE_STATS PORT_STATS QUEUE_STATS ARP_MATCH_IP
actions: output enqueue set_vlan_vid set_vlan_pcp strip_vlan mod_dl_src mod_dl_dst mod_nw_src mod_nw_dst mod_nw_tos mod_tp_src mod_tp_dst
 1(patch-int): addr:3e:66:32:9c:e5:26
     config:     0
     state:      0
     speed: 0 Mbps now, 0 Mbps max
 2(vxlan-0a0a0a0a): addr:3e:b7:60:4d:3e:65
     config:     0
     state:      0
     speed: 0 Mbps now, 0 Mbps max
 LOCAL(br-tun): addr:be:96:0c:3d:64:48
     config:     PORT_DOWN
     state:      LINK_DOWN
     speed: 0 Mbps now, 0 Mbps max
OFPT_GET_CONFIG_REPLY (xid=0x4): frags=normal miss_send_len=0
```
br-tun kết nối với br-int qua port1, kết nối với card mạng eth0 qua port2.

Các cổng của br-int
```sh
root@compute:/home/cong# ovs-ofctl show br-int
OFPT_FEATURES_REPLY (xid=0x2): dpid:00008abe90f05b41
n_tables:254, n_buffers:256
capabilities: FLOW_STATS TABLE_STATS PORT_STATS QUEUE_STATS ARP_MATCH_IP
actions: output enqueue set_vlan_vid set_vlan_pcp strip_vlan mod_dl_src mod_dl_dst mod_nw_src mod_nw_dst mod_nw_tos mod_tp_src mod_tp_dst
 4(patch-tun): addr:9e:a7:19:ae:e8:17
     config:     0
     state:      0
     speed: 0 Mbps now, 0 Mbps max
 15(qvo098a9ccb-81): addr:ee:b5:4d:7a:57:6b
     config:     0
     state:      0
     current:    10GB-FD COPPER
     speed: 10000 Mbps now, 0 Mbps max
 LOCAL(br-int): addr:8a:be:90:f0:5b:41
     config:     PORT_DOWN
     state:      LINK_DOWN
     speed: 0 Mbps now, 0 Mbps max
OFPT_GET_CONFIG_REPLY (xid=0x4): frags=normal miss_send_len=0
```
br-int kết nối với br-tun qua port 4, kết nối với máy ảo qua port 15(qvoxxx)

Chúng ta khảo sát bảng flow rule của br-int và br-tun
```sh
root@compute:/home/cong# ovs-ofctl dump-flows --rsort  br-int
 cookie=0xb595b9f297d967e0, duration=54.226s, table=0, n_packets=0, n_bytes=0, priority=10,icmp6,in_port=15,icmp_type=136 actions=resubmit(,24)
 cookie=0xb595b9f297d967e0, duration=54.052s, table=0, n_packets=11, n_bytes=462, priority=10,arp,in_port=15 actions=resubmit(,24)
 cookie=0xb595b9f297d967e0, duration=54.418s, table=0, n_packets=143, n_bytes=17478, priority=9,in_port=15 actions=resubmit(,25)
 cookie=0xb595b9f297d967e0, duration=20784.131s, table=0, n_packets=50, n_bytes=6300, priority=2,in_port=2 actions=drop
 cookie=0xb595b9f297d967e0, duration=20781.885s, table=0, n_packets=9869, n_bytes=599170, priority=2,in_port=3 actions=drop
 cookie=0xb595b9f297d967e0, duration=54.315s, table=24, n_packets=0, n_bytes=0, priority=2,icmp6,in_port=15,icmp_type=136,nd_target=fe80::f816:3eff:fe30:9f8a actions=NORMAL
 cookie=0xb595b9f297d967e0, duration=54.142s, table=24, n_packets=11, n_bytes=462, priority=2,arp,in_port=15,arp_spa=10.10.60.3 actions=resubmit(,25)
 cookie=0xb595b9f297d967e0, duration=54.609s, table=25, n_packets=120, n_bytes=11326, priority=2,in_port=15,dl_src=fa:16:3e:30:9f:8a actions=NORMAL
 cookie=0xb595b9f297d967e0, duration=20785.798s, table=0, n_packets=483, n_bytes=70044, priority=0 actions=NORMAL
 cookie=0xb595b9f297d967e0, duration=20785.688s, table=23, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0xb595b9f297d967e0, duration=20785.571s, table=24, n_packets=0, n_bytes=0, priority=0 actions=drop
```
```sh
root@compute:/home/cong# ovs-ofctl dump-flows --rsort  br-tun
 cookie=0x97a54ecbc6c45d52, duration=2995.526s, table=22, n_packets=24, n_bytes=1816, dl_vlan=10 actions=strip_vlan,set_tunnel:0x65,output:2
 cookie=0x97a54ecbc6c45d52, duration=2995.677s, table=20, n_packets=118, n_bytes=10900, priority=2,dl_vlan=10,dl_dst=fa:16:3e:04:48:cc actions=strip_vlan,set_tunnel:0x65,output:2
 cookie=0x97a54ecbc6c45d52, duration=904.120s, table=20, n_packets=3, n_bytes=238, priority=2,dl_vlan=10,dl_dst=fa:16:3e:be:e2:f4 actions=strip_vlan,set_tunnel:0x65,output:2
 cookie=0x97a54ecbc6c45d52, duration=23722.584s, table=0, n_packets=774, n_bytes=93310, priority=1,in_port=1 actions=resubmit(,2)
 cookie=0x97a54ecbc6c45d52, duration=2995.677s, table=0, n_packets=97, n_bytes=9593, priority=1,in_port=2 actions=resubmit(,4)
 cookie=0x97a54ecbc6c45d52, duration=2998.835s, table=4, n_packets=97, n_bytes=9593, priority=1,tun_id=0x65 actions=mod_vlan_vid:10,resubmit(,10)
 cookie=0x97a54ecbc6c45d52, duration=23722.581s, table=10, n_packets=285, n_bytes=29295, priority=1 actions=learn(table=20,hard_timeout=300,priority=1,cookie=0x97a54ecbc6c45d52,NXM_OF_VLAN_TCI[0..11],NXM_OF_ETH_DST[]=NXM_OF_ETH_SRC[],load:0->NXM_OF_VLAN_TCI[],load:NXM_NX_TUN_ID[]->NXM_NX_TUN_ID[],output:NXM_OF_IN_PORT[]),output:1
 cookie=0x97a54ecbc6c45d52, duration=23722.583s, table=0, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x97a54ecbc6c45d52, duration=23722.583s, table=2, n_packets=369, n_bytes=34634, priority=0,dl_dst=00:00:00:00:00:00/01:00:00:00:00:00 actions=resubmit(,20)
 cookie=0x97a54ecbc6c45d52, duration=23722.582s, table=2, n_packets=405, n_bytes=58676, priority=0,dl_dst=01:00:00:00:00:00/01:00:00:00:00:00 actions=resubmit(,22)
 cookie=0x97a54ecbc6c45d52, duration=23722.582s, table=3, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x97a54ecbc6c45d52, duration=23722.582s, table=4, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x97a54ecbc6c45d52, duration=23722.581s, table=6, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x97a54ecbc6c45d52, duration=23722.581s, table=20, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,22)
 cookie=0x97a54ecbc6c45d52, duration=23722.480s, table=22, n_packets=363, n_bytes=54500, priority=0 actions=drop

```
Khảo sát dữ liệu đi vào. Dữ liệu đi vào thông qua card vật lý vào qua port 2, chúng ta khảo sát các rule xử lý in_port=2:
```sh
cookie=0x97a54ecbc6c45d52, duration=87.965s, table=0, n_packets=75, n_bytes=8091, priority=1,in_port=2 actions=resubmit(,4)
 cookie=0x97a54ecbc6c45d52, duration=91.123s, table=4, n_packets=75, n_bytes=8091, priority=1,tun_id=0x65 actions=mod_vlan_vid:10,resubmit(,10)
 cookie=0x97a54ecbc6c45d52, duration=20814.869s, table=10, n_packets=263, n_bytes=27793, priority=1 actions=learn(table=20,hard_timeout=300,priority=1,cookie=0x97a54ecbc6c45d52,NXM_OF_VLAN_TCI[0..11],NXM_OF_ETH_DST[]=NXM_OF_ETH_SRC[],load:0->NXM_OF_VLAN_TCI[],load:NXM_NX_TUN_ID[]->NXM_NX_TUN_ID[],output:NXM_OF_IN_PORT[]),output:1
 cookie=0x97a54ecbc6c45d52, duration=14.282s, table=20, n_packets=0, n_bytes=0, hard_timeout=300, priority=1,vlan_tci=0x000a/0x0fff,dl_dst=fa:16:3e:be:e2:f4 actions=load:0->NXM_OF_VLAN_TCI[],load:0x65->NXM_NX_TUN_ID[],output:2
 cookie=0x97a54ecbc6c45d52, duration=6.919s, table=20, n_packets=0, n_bytes=0, hard_timeout=300, priority=1,vlan_tci=0x000a/0x0fff,dl_dst=fa:16:3e:04:48:cc actions=load:0->NXM_OF_VLAN_TCI[],load:0x65->NXM_NX_TUN_ID[],output:2


```
khi dữ liệu đi vào in_port=2, dữ liệu được forward sang rule tương ứng với table =4 để xử lý ```actions=resubmit(,4)```. Xét rule xử lý table=4, ta thấy rule này xử lý các gói dữ liệu của mạng vxlan có id=0x65 =101, dữ liệu được xử lý theo hướng chuyển từ gói tin thuộc mạng vxlan có id=101 sang gói tin vlan nội bộ có id = 10, đồng thời xử lý gói tin theo rule có table =10. 

xét rule xử lý table = 10, rule xử lý gói tin theo hướng học địa chỉ mac của thiết bị gửi gói tin (do hướng của gói tin là từ bên ngoài vào trong node) rồi lưu vào các entry thuộc table 20, sau đó thì chuyển tiếp gói tin qua port 1 (output:1). port 1 là port nối br-tun với br-int. do đó gói tin sau khi được gán nhãn vlan id nội bộ và được học địa chỉ MAC gửi gói tin thì được chuyển tiếp vào br-int để xử lý.
Các địa chỉ sau khi được học trở thành các entry ở table 20. Ở đây ta thấy table 20 tương ứng với 2 entry, với dl_dst là ```fa:16:3e:be:e2:f4 ``` và ```fa:16:3e:04:48:cc```, 2 giá trị này chính là 2 giá trị MAC của router và dhcp device trên network node cùng nằm trên mạng vxlan 101. Các entry này cho phép định tuyến gói tin ra cổng nào để đi ra ngoài.

Quay trở lại với luồng dữ liệu, khi gói tin đi vào br-int sẽ được xử lý theo các rule tương ứng với in_port=4 (do br-tun kết nối với br-int tại port 4 của br-int). Ta thấy các entry không gắn in_port sẽ xử lý các gói tin với in_port=4
```sh
cookie=0xb595b9f297d967e0, duration=20785.798s, table=0, n_packets=483, n_bytes=70044, priority=0 actions=NORMAL
 cookie=0xb595b9f297d967e0, duration=20785.688s, table=23, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0xb595b9f297d967e0, duration=20785.571s, table=24, n_packets=0, n_bytes=0, priority=0 actions=drop
``` 
ta thấy các gói tin này được xử lý theo action NORMAL, như vậy các gói tin này sẽ được xử lý như các gói tin vlan nội bộ, tức là sẽ được định tuyến đến cổng tương ứng với MAC đích của gói tin trong bảng định tuyến của từng mạng nội bộ, sau đó trước khi đi vào máy ảo sẽ được gỡ bỏ thông tin vlan id nội bộ.

Bây giờ chúng ta xét đến luồng dữ liệu đi từ máy ảo ra bên ngoài. Luồng dữ liệu này đi từ máy ảo vào br-int thông qua port 15. Xét các rule tương ứng với port 15 trên br-int:
```sh
cookie=0xb595b9f297d967e0, duration=54.226s, table=0, n_packets=0, n_bytes=0, priority=10,icmp6,in_port=15,icmp_type=136 actions=resubmit(,24)
 cookie=0xb595b9f297d967e0, duration=54.052s, table=0, n_packets=11, n_bytes=462, priority=10,arp,in_port=15 actions=resubmit(,24)
 cookie=0xb595b9f297d967e0, duration=54.418s, table=0, n_packets=143, n_bytes=17478, priority=9,in_port=15 actions=resubmit(,25)
cookie=0xb595b9f297d967e0, duration=54.315s, table=24, n_packets=0, n_bytes=0, priority=2,icmp6,in_port=15,icmp_type=136,nd_target=fe80::f816:3eff:fe30:9f8a actions=NORMAL
 cookie=0xb595b9f297d967e0, duration=54.142s, table=24, n_packets=11, n_bytes=462, priority=2,arp,in_port=15,arp_spa=10.10.60.3 actions=resubmit(,25)
 cookie=0xb595b9f297d967e0, duration=54.609s, table=25, n_packets=120, n_bytes=11326, priority=2,in_port=15,dl_src=fa:16:3e:30:9f:8a actions=NORMAL

```
các rule này đều forward xử lý gói tin theo rule 25. Trên rule 25, các gói tin có địa chỉ nguồn là ```dl_src=fa:16:3e:30:9f:8a``` sẽ được gắn thẻ vlan nội bộ vid=10 rồi chuyển tiếp qua br-tun. Địa chỉ nguồn này chính là MAC của máy ảo trên mạng VXLAN.

Khi đi tới br-tun qua port 1 (patch-int) gói tin được xử lý theo các rule tương ứng với in_port=1. Khảo sát các rule tương ứng với in_port=1:
```sh
 cookie=0x97a54ecbc6c45d52, duration=23722.584s, table=0, n_packets=774, n_bytes=93310, priority=1,in_port=1 actions=resubmit(,2)
cookie=0x97a54ecbc6c45d52, duration=23722.583s, table=2, n_packets=369, n_bytes=34634, priority=0,dl_dst=00:00:00:00:00:00/01:00:00:00:00:00 actions=resubmit(,20)
 cookie=0x97a54ecbc6c45d52, duration=23722.582s, table=2, n_packets=405, n_bytes=58676, priority=0,dl_dst=01:00:00:00:00:00/01:00:00:00:00:00 actions=resubmit(,22)
```
rule ở in_port=1 foward các gói tin tới rule ứng với table=2 để xử lý. rule tương ứng với table=2 forward gói tin tới các rule 20 và 22 để xử lý.
Các rule 20 và 22 xử lý các gói tin thuộc mạng nội bộ vlan=10:
```sh
cookie=0x97a54ecbc6c45d52, duration=2995.526s, table=22, n_packets=24, n_bytes=1816, dl_vlan=10 actions=strip_vlan,set_tunnel:0x65,output:2
 cookie=0x97a54ecbc6c45d52, duration=2995.677s, table=20, n_packets=118, n_bytes=10900, priority=2,dl_vlan=10,dl_dst=fa:16:3e:04:48:cc actions=strip_vlan,set_tunnel:0x65,output:2
 cookie=0x97a54ecbc6c45d52, duration=904.120s, table=20, n_packets=3, n_bytes=238, priority=2,dl_vlan=10,dl_dst=fa:16:3e:be:e2:f4 actions=strip_vlan,set_tunnel:0x65,output:2
```
Chúng ta có thể thấy các gói tin gửi ra ngoài không rõ địa chỉ đích hoặc địa chỉ đích là một trong các địa chỉ đã học đều được xử lý theo hướng bỏ thẻ định danh vlan nội bộ và gắn thẻ định danh của mạng VXLAN với id=0x65 =101 rồi đóng vào gói tin UDP +IP trước khi chuyển tiếp qua card vật lý eth0 qua port2.
Ở giai đoạn này, sau khi gắn thẻ định danh của mạng VXLAN, br-tun đồng thời xác định địa chỉ IP đích của node mà chứa địa chỉ MAC đích để xác định địa chỉ đích của gói tin IP sử dụng để chuyển gói tin L2 frame. Sau đó gói tin IP này được chuyển qua cho card mạng eth2 để gửi ra hệ thống mạng vật lý bên ngoài.
#Kết luận
3 bài viết này là một số kiến thức về các loại mạng nội bộ và các triển khai của các loại mạng nội bộ này trên môi trường OpenStack. Tuy đã cố gắng, tuy nhiên bài viết vẫn còn rất nhiều thiếu sót. Hy vọng những thiếu sót này sẽ được bổ sung trong tương lai với sự đào sâu tìm hiểu hơn về networking trong OpenStack
#Tài liệu tham khảo
- Learning OpenStack Networking (Neutron), 2nd Edition,  James Denton 

- https://wiki.openstack.org/wiki/Ovs-flow-logic

- http://www.opencloudblog.com

- OpenStack Networking Guide 

- http://docs.openstack.org/mitaka/networking-guide/deploy.html

- http://www.cisco.com/c/en/us/products/collateral/switches/nexus-9000-series-switches/white-paper-c11-729383.html

- https://disqus.com/home/discussion/packetpushers/tcpip_over_vxlan_bandwidth_overheads/- 

- http://docs.openstack.org/mitaka/install-guide-ubuntu/

- https://assafmuller.com/2013/10/14/gre-tunnels-in-openstack-neutron/

- https://tools.ietf.org/html/rfc2784
- 
- http://docs.ocselected.org/openstack-manuals/kilo/networking-guide/content/section_networking-scenarios.html

Và còn rất nhiều bài blog, bài viết về networking trong OpenStack trên internet...
