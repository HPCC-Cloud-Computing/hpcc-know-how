Introduction to networking in OpenStack
=========

#1. Basic networking


###Ethernet
Ethernet là một trong những công nghệ phổ biến nhất để tạo ra mạng cục bộ (LAN). 

Trong mô hình OSI, Ethernet là giao thức ở tầng **datalink** mô tả cách thức để các thiết bị mạng có thể định dạng dữ liệu và truyền chúng tới các thiết bị khác trên cùng một mạng.

 Mỗi thiết bị trên mạng Ethernet được định danh duy nhất  bởi địa chỉ MAC. Nhờ địa chỉ MAC, hai thiết bị giao tiếp được với nhau bằng cách gửi đi các *frame* đã đóng gói địa chỉ MAC nguồn và địa chỉ MAC đích và chúng được chuyển tiếp nhờ thiết bị mạng được gọi là switch.

###VLANs
VLAN ( virtual LAN) là một kĩ thuật mạng cho phép chia một miền quảng bá vật lí ra thành nhiều mạng cục bộ độc lập nhau.  Mỗi mạng cục bộ được đặc trưng bởi một định danh, đó là VLAN ID. Cụ thể, xét hình bên dưới, khi sử dụng VLAN, nếu các máy A,B,C cùng kết nối tới một switch nhưng được định nghĩa ở 2 mạng VLAN khác nhau thì khi máy A gửi gói tin quảng bá(broadcast) thì chỉ những máy thuộc cùng mạng VLAN với A (là máy B) mới có thể nhận được gói tin.

![](https://github.com/vanduc95/OpenStack_Network/blob/master/img/VLAN_ex1.png)

Để máy C có thể gửi gói tin cho máy A (nằm ở 2 mạng VLAN khác nhau), chúng ta cần một router hoặc thiết bị tầng 3. Máy C sẽ gửi gói tin lên router và router sẽ gửi lại gói tin xuống cho A thông qua switch.

 ![](https://github.com/vanduc95/OpenStack_Network/blob/master/img/VLAN_ex2.png)

Tham khảo chi tiết [tại đây](https://github.com/cloudcomputinghust/openstack-manual/blob/master/Introduction-to-OpenStack-networking/OpenStack-networking-Layer2-Introduction.md)

###ARP 

Trong hệ thông mạng có 2 loại địa chỉ được gán cho một máy tính là địa chỉ IP và địa chỉ MAC.  Trong thực tế, các card mạng chỉ có thể hiểu và liên lạc với nhau bằng địa chỉ MAC. Vì vậy, để các máy có thể hiểu và liên lạc với nhau trong môi trường mạng, cần có một cơ chế diễn giải địa chỉ giữa IP và MAC. Đó là giao thức **Address Resolution Protocol** viết tắt là  **ARP**

**Cơ chế của ARP như sau:** 

Giả sử máy A có địa chỉ IP là **192.168.1.5/24** và muốn gửi một packet tới máy B với địa chỉ IP là **192.168.1.7**.

 - Đầu tiên, máy A sẽ gửi một ARP request bao gồm địa chỉ MAC của máy A
   và địa chỉ IP của máy B.
 - Vì là gói tin broadcast nên các máy trên mạng sẽ nhận được gói tin và
   xử lí. Máy B nhận được request  đúng là IP của nó thì sẽ gửi lại cho
   máy A một response chứa địa chỉ MAC của máy B.
 - Lúc này máy A đã có địa chỉ MAC của máy B và máy A có thể bắt đầu
   truyền gói tin cho máy B.
   
**Sau đây ta sẽ xem một ví dụ về việc chuyển tin giữa hai gói như thế nào?**

Giả sử A biết địa chỉ IP của B

![enter image description here](https://github.com/vanduc95/OpenStack_Network/blob/master/img/foward.png)

 - A tạo một gói tin IP, địa chỉ nguồn A, địa chỉ đích B.
 - A dùng ARP để lấy địa chỉ MAC của router 111.111.111.110
 - A tạo một frame với địa chỉ đích là router và đặt gói tin vào.
 - A chuyển frame tới R
 - Tại R: nhận frame và đọc địa chỉ IP của B từ trong khung tin
 - R dùng ARP để tìm địa chỉ MAC của B
 - R tạo một frame, đặt gói tin vào và chuyển đến B.

###DHCP

DHCP (Dynamic Host Configuration Protocol) là giao thức cho phép cấp phát địa chỉ IP một cách tự động cho clients. Mục đích quan trọng nhất là tránh trường hợp hai máy tính khác nhau lại có cùng địa chỉ IP.

DHCP gồm 2 thành phần chính là **DHCP client** và **DHCP server**. Trong đó DHCP client là một thiết bị mạng yêu cầu địa chỉ IP còn DHCP server có nhiệm vụ lắng nghe và cấp phát IP cho client.

Quá trình cấp phát IP được mô tả như hình dưới đây 

![](https://github.com/vanduc95/OpenStack_Network/blob/master/img/DHCP.png)


 - DHCP client gửi một discover ("Tôi là client có địa chỉ MAC
   **08:00:27:b9:88:74**, tôi cần một địa chỉ IP").
 - DHCP server gửi một offer ("OK **08:00:27:b9:88:74**, tôi sẽ gửi cho
   bạn địa chỉ IP **10.10.0.112**").
 - DHCP client gửi một request ("Tôi đồng ý lấy địa chỉ này").
 - DHCP server gửi một acknowledgement ("OK, IP  **10.10.0.112 ** là của
   bạn").

Trong OpenStack, phần mềm thứ 3 có tên là [dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html)  được dùng để implement dịch vụ DHCP server.

#2. Network devices
Khi nói đến network, có 2 thiết bị mạng cơ bản và  quan trọng cho phép nhiều thiết bị có thể kết nối được với nhau đó là switch và router. 

###Switch

Switch thông thường được biết đến như là một "thiết bị chuyển mạch". Nó là thiết bị mạng thuộc tầng 2 trong mô hình OSI (Data Link Layer). Nó có thể coi là một Bridge có nhiều cổng. Switch chuyển tiếp các frame dựa trên địa chỉ MAC. Switch tập trung các kết nối và quyết định chọn đường dẫn để truyền dữ liệu hiệu quả sử dụng cơ chế tự học tính toán bảng MAC Table. Dữ liệu được đóng gói trong frame được chuyển mạch từ cổng input đến cổng output và đến được node đích như mong muốn.

![enter image description here](https://github.com/vanduc95/OpenStack_Network/blob/master/img/switch.png)

###Router
Router là thiết bị mạng thuộc tầng 3 trong mô hình OSI (Network layer). Nó còn được gọi là "thiết bị định tuyến hoặc bộ định tuyến" có chức năng chuyển các gói dữ liệu (packet) qua một liên mạng đến các đầu cuối dựa trên địa chỉ IP thông qua một tiến trình gọi là định tuyến.

![enter image description here](http://vnreview.vn/image/14/69/73/1469738.jpg?t=1448523262236)

Chúng ta có thể hiểu rõ hơn về sự khác nhau về switch và router qua video [này](https://www.youtube.com/watch?v=Ofjsh_E4HFY) 

# Network address translation (NAT)

Trong IPv4, có 2 loại địa chỉ IP là IP public và IP private.

 IP public là địa chỉ được cung cấp bởi ISP (Internet Service Provider) khi bạn kết nối vào Internet, nó là địa chỉ dùng để xác định máy tính của bạn trên Internet là duy nhất. Khác với IP public, IP private là địa chỉ dùng để xác định máy tính của bạn trên một mạng riêng nào đó như mạng nội bộ... và được quyền gán bất kì địa chỉ IP nào tùy thích miễn là nằm trong dải IP được quy định sẵn cho mạng private. Các địa chỉ nằm trong dải đó là:  

> Lớp A:  10.0.0.0 đến 10.255.255.255
> 
> Lớp B: 172.16.0.0 đến 172.31.255.255
> 
> Lớp C: 192.168.0.0 đến 192.168.255.255

Máy tính chỉ ra ngoài internet được khi nó có địa chỉ public vì địa chỉ public là duy nhất, do vậy cần phải có một kĩ thuật để chuyển đồi các IP private trong mạng cục bộ thành IP public để ra ngoài internet và ngược lại để các máy ngoài internet có thể gửi trả dữ liệu cho các máy trong mạng cục bộ. **NAT được tạo ra để giải quyết vấn đề này**

Kỹ thuật NAT dùng để chuyển tiếp các gói tin giữa những lớp mạng khác nhau trên một mạng lớn. NAT dịch hay thay đổi một hoặc cả hai địa chỉ bên trong một gói tin khi gói đó đi qua router, hay một số thiết bị khác.

Sau đây ta sẽ đề cập đến một số loại NAT cơ bản sau:

###Static NAT (NAT tĩnh)
Static NAT (NAT tĩnh) là phương thức NAT một đôi một. Một địa chỉ IP Private sẽ được map với một địa chỉ IP Public.

![enter image description here](https://github.com/vanduc95/OpenStack_Network/blob/master/img/static_NAT.png)
 
Trong Static NAT (NAT tĩnh), địa chỉ IP của máy tính là 192.168.32.10 luôn được Router biên dịch đến địa chỉ IP 213.18.123.110.

###Dynamic NAT (NAT động)
Một địa chỉ IP Private sẽ được map với một địa chỉ IP Public trong nhóm địa chỉ IP Public.

![enter image description here](https://github.com/vanduc95/OpenStack_Network/blob/master/img/dynamic_NAT.png)

Trong Dynamic NAT (NAT động), máy tính có địa chỉ IP 192.168.32.10 luôn được Router biên dịch đến địa chỉ đầu tiên 213.18.123.100 trong dãy địa chỉ IP từ 213.18.123.100  đến 213.18.123.150.
###Overloading NAT
NAT Overloading là một dạng thức của NAT động (Dynamic Overload). Nhiều địa chỉ IP Private sẽ được map với một địa chỉ IP Public qua các Port (cổng) khác nhau.

![enter image description here](https://github.com/vanduc95/OpenStack_Network/blob/master/img/overload_NAT.png)

Trong Overloading NAT, mỗi máy tính trong mạng nội bộ (Private Network) được Router biên dịch đến cùng một địa chỉ IP 213.18.123.100 nhưng trên các cổng giao tiếp khác nhau.

Tham khảo chi tiết [tại đây](https://github.com/hocchudong/networking-team/blob/master/LinhLT/Iptables/kienthuccanco.md)
