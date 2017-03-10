Linux networking - Macvlan, Macvtap
===

#1. Macvlan

Macvlan driver là một Linux kernel driver tách biệt được Macvtap sử dụng. Macvlan cho phép bạn cấu hình các sub-interface (hay còn gọi là **slave devices**) trên một card mạng vật lý (còn gọi là **upper device** hay **lower device**). Chúng ta có thể hiểu đơn giản là tạo ra các card mạng ảo "gắn" trực tiếp lên card mạng vật lí, mỗi sub-interfaces này có địa chỉ MAC và địa chỉ IP riêng. Các ứng dụng, VMs và các containers(docker) có thể kết nối với một sub-interface nhất định để kết nối trực tiếp với mạng vật lý, sử dụng địa chỉ MAC và địa chỉ IP riêng của chúng. 

Các sub-interfaces này không có khả năng giao tiếp trực tiếp tới parent interface (card vật lý tạo ra các sub-interfaces),. Có nghĩa là các máy ảo không thể nói chuyện trực tiếp được với host. Nếu chúng ta muốn máy ảo - host có thể giao tiếp được với nhau, ta cần tạo thêm một macvlan sub-interfaces khác và gán chúng tới host. Tham khảo thêm tại https://www.furorteutonicus.eu/2013/08/04/enabling-host-guest-networking-with-kvm-macvlan-and-macvtap/

![enter image description here](https://github.com/vanduc95/OpenStack_Network/blob/master/img/macvlan_overview.png)

Macvlan thường được sử dụng trong công nghệ ảo hóa container (ví dụ như Linux Container - LXC hoặc Docker). Ngoài ra nó cũng được sử dụng trong trường hợp cần sử dụng các địa chỉ MAC ảo ví dụ như [VRRP](https://en.wikipedia.org/wiki/Virtual_Router_Redundancy_Protocol) (Virtual Router Redundancy Protocol)

Macvlan là giải pháp tốt cho phép các VMs và các container kết nối tới mạng vật lí, nhưng nó có một vài hạn chế :
• Switch mà host kết nối tới có thể có policy hạn chế số lượng địa chỉ MAC trên một port vật lý.
• Nhiều card mạng có sự hạn chế về số lượng địa chỉ MAC hỗ trợ trên phần cứng. Vượt quá số lượng cho phép sẽ làm ảnh hưởng đến hiệu năng của hệ thống.
• Chuẩn mạng không dây IEEE 802.11 không cho phép nhiều địa chỉ MAC trên một client. Các macvlan sub-interface sẽ bị block bởi wireless interfaces driver. **Do vậy chúng ta không thể tạo ra macvtap trên wlan (mạng cục bộ không dây)**

##Macvlan mode
Một macvlan interface có thể được cấu hình theo 4 chế độ (mode) sau: **VEPA**, **Bridge**, **Private** và **Passthru**. Chúng ta sẽ đi tìm hiểu cụ thể từng mode này.

###VEPA
VEPA (Virtual Ethernet Port Aggregator) là chế mặc định. Trong chế độ này, dữ liệu giữa các endpoints (các VM trên máy vật lý) trên cùng một card mạng vật lý (card mạng tạo ra các macvtap interface) được gửi thông qua card này tới switch vật lý. Chế độ này yêu cầu switch ngoài phải hỗ trợ "Reflective Relay" hay "hairpin mode", nghĩa là switch có thể gửi trả lại một frame trên chính port mà nhận frame đó. Tuy nhiên, hầu hết các switch ngày nay đều không hỗ trợ chế độ này.

![enter image description here](https://github.com/vanduc95/OpenStack_Network/blob/master/img/VEPA_mode.png)

###Bridge
Đây là chế độ được sử dụng phổ biến nhất . Ở chế độ này, các endpoints (VMs, container...) có thể giao tiếp trực tiếp với nhau mà không phải gửi dữ liệu thông qua lower device (card vật lý tạo ra các macvtap interface), do vậy các endpoints có thể nói chuyện với nhau nhanh hơn chế độ VEPA. Việc sử dụng chế độ này không yêu cầu switch vật lý phải hỗ trợ "Reflective Relay". Chế độ này hữu ích khi muốn thiết lập một switch cơ bản và khi hiệu suất truyền thông nội bộ giữa các VM cần được đảm bảo. 

![enter image description here](https://github.com/vanduc95/OpenStack_Network/blob/master/img/bridge_mode.png)

###Private
Ở chế độ này, các sub-interface trên cùng một parent-interface không thể giao tiếp được với nhau. Các frames từ sub-interface đươc chuyển tiếp ra ngoài switch vật lí thông qua parent-interface. Ngay cả khi switch gửi lại frame tới một sub-interface khác thì thì cũng sẽ bị **drop**.

![enter image description here](https://github.com/vanduc95/OpenStack_Network/blob/master/img/private_mode.png)

###Passthrough

Chế độ này chỉ cho phép một tạo một sub-interface trên một paren-interf 

Chế độ này cho phép một single VM kết nối trực tiếp với switch vật lí. Khi sử dụng chế độ này thì parent-interface sẽ không thể **ping** ra ngoài được nữa mà chỉ sub-interface mới **ping** được ra ngoài internet mà thôi hay có thể hiểu là máy thật sẽ không thể kết nối internet mà chỉ có máy ảo mới kết nối được. Một điều cần lưu ý nữa là chế độ Passthrough này chỉ cho phép tạo một sub-interface trên một parent-interface. Khi sử dụng KVM tạo 2 máy ảo sử dụng chế độ này thì hệ thống sẽ báo lỗi.

![enter image description here](https://github.com/vanduc95/OpenStack_Network/blob/master/img/Passthru_mode.png)

#2. Tap
**Tap** interface là một card mạng dưới dạng phần mềm. Thay vì phải đưa frames tới hoặc nhận frame từ card Ethernet vật lý, các frames này được đọc và ghi bởi chương trình thuộc user space. Kernel sẽ cho phép kích hoạt các tap interface thông qua file `/dev/tapN`, trong đó N là số hiệu của card mạng. 

Tap interface thường đi kèm với hai công nghệ switch ảo trong Linux là **linux bridge** và **openvswitch**. Trong đó tap interfaces chính là các cổng tạo trên các switch ảo để các máy ảo gắn vào(ví dụ trên linux bridge thường được đặt tên là vnetN - với N là số hiệu cổng). Tap interface làm việc với các Ethernet frames (khác với Tun interfaces làm việc với các IP frames). 

#3. Macvtap
**Macvtap** interfaces kết hợp thuộc tính của hai công nghệ **macvlan** và **tap**, nó là một card mạng ảo tương tự như tap và tạo trên một card mạng vật lý.  

Ta có thể tạo ra một macvtap như sau:

    $ sudo ip link add link eth0 name macvtap0 type macvtap

Để kiểm tra macvtap vừa tạo, sử dụng command:

    $ ip link
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN mode DEFAULT
     link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT qlen 1000
     link/ether 00:1f:d0:15:7b:e6 brd ff:ff:ff:ff:ff:ff
    3: macvtap0@eth0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT qlen 500
     link/ether 42:96:80:ee:2d:23 brd ff:ff:ff:ff:ff:ff

Macvtap interface này sẽ được chỉ định trong file xml /dev/tap3:

    ls -l /dev | grep tap
    # ket qua se tuong tu nhu sau
    crw-------  1 root root    247,   1 Aug  9 20:32 tap3
Để xóa macvtap interface này ta dùng câu lệnh sau:

    $ sudo ip link del macvtap3


Một chương trình thuộc user space có thể mở file thiết bị này và sử dụng nó để gửi và nhận các Ethernet frames thông qua nó. Khi kernel truyền một frame qua interface macvtap0, thay vì gửi nó tới card mạng vật lý, nó sẽ được đọc từ file này bởi một chương trình thuộc userspace. Tương tự như vậy, khi một chương trình thuộc user space ghi nội dung lên một Ethernet frame tới file /dev/tap3, các đoạn mã networking của kernel sẽ thấy được frame bởi vì nó được nhận thông qua thiết bị macvtap0. 

**Some note about macvtap:**
• Do macvtap dựa trên công nghệ macvlan nên macvtap có thể sử dụng ở các mode của macvlan: VEPA, bridge, private and passthru
• Tương tự macvlan, các VMs sử dụng macvtap interface không thể kết nối trực tiếp với card mạng vật lí. 

#4. Tài liệu tham khảo
[1]https://github.com/hocchudong/networking-team/blob/master/ThaiPH/Linux%20Networking/ThaiPH_linux_networking_macvtap.md

[2] http://hicu.be/bridge-vs-macvlan

[3] https://seravo.fi/2012/virtualized-bridged-networking-with-macvtap

[4] http://backreference.org/2014/03/20/some-notes-on-macvlanmacvtap/

[5] http://hicu.be/macvlan-vs-ipvlan

