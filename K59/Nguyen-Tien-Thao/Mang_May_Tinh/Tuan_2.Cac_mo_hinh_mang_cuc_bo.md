# Các mô hình mạng cục bộ

Báo cáo sẽ đi tìm hiểu chi tiết về bốn loại mạng cục bộ được sử dụng trong thực tế là:

1. Flat
1. VLAN
1. VXLAN
1. GRE

## Flat

Mạng flat là một mạng không cung cấp bất kỳ một tùy chọn thêm nào. Nó là mạng truyền thống được sử dụng ở Layer2. Mạng flat phân tách miền quảng bá bằng cách sử dụng router và mội máy kết nối đến cùng một router đều thuộc cùng một miền quản bá. Nói chung là tất cả các thiết bị trong một mạng flat có cùng một miền quảng bá.


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

Giải pháp đó có một vấn đề nếu một switch có nhiều mạng VLAN kết nối sẽ tốn nhiều cổng cho và sẽ không hiệu quả. Một giải pháp khác là sử dụng một cổng đặc biệt trên mỗi switch gọi là trunk port để kết nối giữa hai switch. Trunk port không thuộc một mạng VLAN nào mà thuộc về tất cả các VLANs và do đó gửi từ  bất kỳ VLAN nào sẽ được chuyển qua trunk port để đến switch khác. Để có thể xác đinh VLAN nào sẽ được gửi tới sẽ thêm VLAN-tag theo giao thức 802.1Q đã trình bày ở trên.

![TRunk](http://www.technologuepro.com/reseaux/Configuration-d-un-switch/images/Configuration-des-VLAN-1.gif)

## VXLAN

## Tài liệu tham khảo

[https://en.wikipedia.org/wiki/Virtual_LAN](https://en.wikipedia.org/wiki/Virtual_LAN)