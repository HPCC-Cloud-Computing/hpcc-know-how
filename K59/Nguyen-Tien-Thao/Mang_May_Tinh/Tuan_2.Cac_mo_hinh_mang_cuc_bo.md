# Các mô hình mạng cục bộ

## Flat

Mạng flat là một mạng không cung cấp bất kỳ một tùy chọn thêm nào. Nó là mạng truyền thống được sử dụng ở Layer2. Tất cả các thiết bị trong một mạng flat có cùng một miền quảng bá.


## VLAN(virtual LAN)

Khác với Flat, VLAN là một kỹ thuật cho phép tạo các mạng LAN độc lập một cách logic trên cơ sở cùng một kiến trúc hạ tầng vật lý. Việc sử dụng VLAN giúp tạo ra nhiều miền quảng bá độc lập.

![VLAN1](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/VLAN1.png)

Có 3 Loại VLAN:

- VLAN dựa trên cổng: Mỗi cổng được gắn với một VLAN xác định. Có nghĩa là thiết bị nào kết nối tới cổng đó sẽ thuộc một VLAN xác định. Đây là loại VLAN đơn giản và phổ biến nhất.
- VLAN dựa trên địa chỉ MAC: Mỗi địa chỉ MAC được gắn vào một VLAN nhất định cách này khó khăn trong việc quản lý.
- VLAN dựa trên giao thức: Tương tự như dựa trên địa chỉ MAC nhưng sử dụng địa chỉ IP thay cho MAC. Loại này không được thông dụng.

Vậy tại sao đã có mạng flat lại cần thêm mạng VLAN. Mạng VLAN có ưu nhược điểm là gì?

- Tiết kiệm băng thông mạng: Do VLAN chia mạng LAN thành các mạng quảng bá nhỏ nên khi một gói tin quảng bá được gửi đi nó chi được gửi trong một mạng VLAN duy nhất, không truyền đến các VLAN khác nên tiết kiệm được băng thông đường truyền.
- Tăng tính bảo mật: Các VLAN có các mạng quảng bá khác nhau không truy cập được vào nhau trừ khi có khai báo định tuyến.
- Dễ dàng thêm bớt các máy vào VLAN việc thêm bớt các máy có thể được thực hiện bằng phần mềm.

Sự khác nhau LAN và VLAN

Ta thấy VLAN cho phép tạo ra các miền quảng bá khác nhau trên cùng một hạ tầng vật lý vậy làm sao để phân biệt được các máy tính nào thuộc mạng VLAN nào hẳn phải có một thông tin nào đó? Và đó là VLAN ID. Với VLAN khung Ethenet được gán thên VLAN ID. Giá trị VLAN ID được gán vởi switch.

Giao thức thường dùng để cấu hình VLAN ngày này là giao thức IEEE 802.1Q. Theo giao thức này số lượng VLAN tối đa trọng một Ethernet là 4094(dùng 12 bit cho trường VLAN ID trong đó có hai địa chỉ không sử dụng là 0x0000 và 0xFFFF).

Cấu trúc gói tin 802.1Q

![VLANFrame](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/VLANframe.png)

So với frame ethernet frame theo chuẩn 802.1Q chèn thêm 32 bit(802.1Q header) vào giữa hai trường: MAC address và EtherType. Cụ thể cấu trúc của 32 bit đó là :

- TPID(tag protocol identifier): 16 bit có giá trị 0x8100 đẻ cho biết đây là một IEEE 802.1Q-tagged frame.
- TCI(tag control information):
  - PCP(prority code point): 3 bit: Mức độ ưu tiên của frame. Giá trị 1(background),2( 0 (best effort, default), 2 (excellent effort), 3 (critical application), 4 (video), 5 (voice), 6 (internetwork control), 7 (network control).
  - DEI(drop eligible indicator): 1 bit Có thể được sử dụng riêng lẻ hoặc cùng với PCP để xác định frame thích hợp để bỏ khi xẩy ra tắc nghẽn.
  - VLAN ID(VLAN identifier): 12 bit dùng để đặc tả VLAN mà frame đó thuộc vào(có thể nhận 4094 giá trị).

Bây giờ nếu hai máy tính muốn gửi tin cho nhau chúng sẽ thực hiện như thế nào. Ta sẽ xét hai trường hợp: Thuộc cùng một VLAN và khác VLAN.

- Khác VLAN: Giả sử máy A thuộc VLAN20  muốn gửi gói tin cho máy B thuộc VLAN30. MẶc dù hai máy này cùng thuộc một switch nhưng do khác mạng VLAN nếu gói tin từ máy A sẽ đi tới switch sau đó tới router và sau đó lại về switch tới mạng VLAN30 và tới máy B. Điều này có vẻ là bất hợp ký nhưng có một số thiết bị bao gồm một VLAN switch và router nên việc này có thể thực hiện trên một thiết bị.

![http://www.cisco.com/c/dam/en/us/td/i/000001-100000/45501-50000/46501-47000/46647.ps/_jcr_content/renditions/46647.jpg](http://www.cisco.com/c/dam/en/us/td/i/000001-100000/45501-50000/46501-47000/46647.ps/_jcr_content/renditions/46647.jpg)

- Đó và với hai máy trên hai VLAN vậy nếu cùng một VLAN thì vấn đề như thế nào: Ta cũng cần xét hai trường hợp: Thứ nhất là cùng một switch, thứ hai là thuộc hai switch khác nhau.

  - Trường hợp thuộc cùng một switch mọi vấn đề đơn giản.
  - Trường hợp thuộc hai switch vấn đề làm sao để có thể kết nối hai switch đó với nhau. Giải pháp thứ nhất là trên mỗi switch để hai cổng để có thể kết nối đến switch kia một để gửi và một để nhận như trong hình.

![https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/2Switch.pnghttps://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/2Switch.png](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/2Switch.png)

Giải pháp đó có một vấn đề nếu một switch có nhiều mạng VLAN kết nối sẽ tốn nhiều cổng cho và sẽ không hiệu quả. Một giải pháp khác là sử dụng một cổng đặc biệt trên mỗi switch gọi là trunk port để kết nối giữa hai switch hoặc switch với router. Trunk port không thuộc một mạng VLAN nào mà thuộc về tất cả các VLANs và do đó gửi từ  bất kỳ VLAN nào sẽ được chuyển qua trunk port để đến switch khác hoặc đến router. Để có thể xác đinh VLAN nào sẽ được gửi tới sẽ thêm VLAN-tag theo giao thức 802.1Q đã trình bày ở trên.

![https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/trunk.png](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/trunk.png)

## VXLAN

Ta có thể thấy mạng VLAN có một số nhược điêm như sau:

- Giới hạn 4094 VLANs là không đủ.
- Hạn chế về khoảng cách và triển khai.

Ta thấy mạng VLAN giúp tạo ra các mạng miền quảng bá độc lập trong cùng một mạng LAN nhưng VLAN có một số nhược điểm như chỉ có thể có 4094 VLANs trong một mạng LAN, bảng địa chỉ MAC của switch là quá hơn. Một giải pháp được sử dụng cải thiện những nhược điểm trên là VXLAN.

Mạng VXLAN cung cấp một cơ chế để kết hợp hai hay nhiều layer3 network domain và làm cho chúng giống như một layer2 domain. Điều này cho phép hai máy tính trên hai mạng khác nhau nếu chúng cùng ở trong một layer2 subnet.
![https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/VXLAN1.png](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/VXLAN1.png)

### VXLAN đóng gói và packet format

VXLAN sử dụng MAC Address-in-User Datagram Protocol (MAC-in-UDP) đóng gói để mở rộng layer2 segment. VXLAN đinh nghĩ MAC-in-UDP được đóng gói bảo gồm gói tin gốc của layer2+ VXLAN header sau đó thêm vào UDP-IP Packet.


Định dạng packet VXLAN:

![VXLANPACketformat](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/VXLANpacketformat.png)

VXLAN sử dụng 8 byte cho trường VXLAN header trong đó  bit để đánh địa chỉ VNID. Với 24 bit này VXLAN có thể hỗ trợ 16 triệu LAN.

### VXLAN tunnel Endpoint(VTEP)

VTEP là một thiết bị được VXLAN sử dụng để encapsulation and de-encapsulation các gói tin vận chuyển và ánh xạ các máy trong VXLAN. VTEP cung cấp hai interface một là switch interface trong mạng LAN hai là IP interface dùng trong IP network.

![VTEP](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/VTEP.png)

- Tương ứng với hai interface VTEP gồm hai thành phần. Thành phần thứ nhất là switch cung cấp switch interface. Thành phần thứ hai cung cấp IP interface. Thành phần này có một địa chỉ IP duy nhất cho phép xác định VTEP trong mạng (IP tầng 3). Địa chỉ IP của VTEP này được sử dụng để đóng gói thêm vào Ethernet frame và chuyển gói tin đã được đóng gói đó lên qua mạng thông qua IP interface.

### Cách để chuyển gói tin giưã hai máy trong VXLAN

![VXLAN](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/VXLAN.png)

Trong hình giả sử máy A muốn gửi gói tin cho máy B gói tin từ máy A đến máy B sẽ thay đổi và được đóng gói và bóc tách cụ thể là:

- Máy A tạo một L2 Frame với các trường IP và MAC như trong hình vẽ.
- L2 Frame được gửi đến VTEP-1. Tại VTEP-1 dựa vào bảng ánh xạ cho biết máy B tại VTEP-2. VTEP-1 đóng gói thêm các trường VXLAN,UDP, outer IP header vào L2 Frame. Cụ thể trong trường outer IP header, IP nguồn là IP của VTEP-1, IP đích là IP của VTEP-2. VTEP-1 sau đó thực hiện tra cứu địa chỉ IP của VTEP-2 để chọn đường tiếp theo cho gói tin. Sau đó sử dụng địa chỉ MAC của nút kế tiếp đóng gói vào gói tin và gửi tới nút kế tiếp.
- Gói tin được định tuyến và chuyển tới VTEP-2 dựa vào địa chỉ VTEP-2 trong outer IP header. Sau khi VTEP-2 nhận được gói tin nó tiếp hành loại bỏ các trường outer IP header, UDP, VXLAN và tiến hành chuyển tới máy B dựa trên địa chỉ MAC đích trong L2 frame.


## GRE

Giao thức này sẽ đóng gói một số kiểu gói tin vào bên trong các IP tunnels để tạo thành các kết nối điểm-điểm ảo.

Ví dụ: Có thể đóng gói IPX packet vào trong gói tin IP.

Gói tin sau khi đóng gói được truyền qua mạng và sử dụng GRE header để định tuyến.

GRE thêm ít nhất 24 bytes vào gói tin trong đó gồm 20 byte IP header và 4 byte là GRE header. GRE cũng có thể thêm vào 12 byte để cung cấp các tính năng như: checksum, key chứng thực, sequence number.
![GRE](https://sirpremier.files.wordpress.com/2012/05/a.png)

Cấu trúc của GRE header:

- bit thứ nhất : Checksum. Bit này cho biết có sử dụng checksum hay không. Nếu giá trị là 1, 4 byte checksum sẽ được thêm vào GRE header sau trường Protocol type.
- bit 13-15: GRE version
- 2 byte còn lại sử dụng cho trường giao thức. 16 bit này xác định kiểu gói tin sẽ được mang trong GRE tunnel.

![GRE](https://sirpremier.files.wordpress.com/2012/05/c.png)



## Tài liệu tham khảo

- [https://en.wikipedia.org/wiki/Virtual_LAN](https://en.wikipedia.org/wiki/Virtual_LAN)
- [https://github.com/NTT-TNN/Basic_knowledge/blob/master/Mang_may_tinh/white-paper-c11-729383.pdf](https://github.com/NTT-TNN/Basic_knowledge/blob/master/Mang_may_tinh/white-paper-c11-729383.pdf)