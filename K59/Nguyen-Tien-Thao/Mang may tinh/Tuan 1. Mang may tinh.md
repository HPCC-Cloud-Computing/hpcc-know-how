# Mạng máy tính

Bài viết bao gồm các phần sau:

- Phần 1: Các khái niệm cơ bản trong mạng máy tính
- Phần 2: Mô hính OSI trong mạng máy tính

## Phần 1. Các Khái niệm cơ bản trong mạng máy tính

### 1.1 Mạng máy Tính

Mạng máy tính là sự kết hợp các máy tính với nhau thông qua các thiết bị kết nối và phương tiện truyền thông theo một giao thức nào đó giúp các máy tính có thể chia sẻ tài nguyên với nhau. Các máy tính(nodes) kết nối với nhau thông qua dây cáp hoặc qua sóng. Mạng máy tính có thể có nhiều kích thước, hình dạng và cách thức khác nhau. Một mạng máy tính phổ biến nhất hiện nay là mạng internet.

### 1.2 Tầng

Tầng là cách tổ chức chương trình thành các thành phần chức năng riêng biệt theo một cách nhất định và theo thứ bậc. Mỗi layer thường có một giao diện duy nhất cung cấp cho layer bên trên và sử dụng giao thức do layer bên dưới cung cấp còn lại sẽ độc lập với các layer khác.``

### 1.3 Giao thức

Giao thức là tập hợp các quy tắc quy định khuôn dạng, ngữ nghĩa, thứ tự các thông điệp được gửi và nhận bởi các nút mạng và các hành vi khi trao đổi thông điệp đó. Các giao thức xác định cách tương tác giữa các thực thể trong giao tiếp. Ví dụ giao thức TCP/IP

## Phần 2. Mô hình OSI trong mạng máy tính

Trong phần này ta sẽ tìm hiểu xem mô hình OSI là gì, nó dùng để làm gì, cấu tầng của mô hình OSI, chức năng cũng như các thức hoạt động của từng tầng như thế nào ?

## 2.1 Tổng quan về mô hình OSI

Mô hình OSI là mô hình cho phép tất cả các hệ thống có thể truyền thông với nhau mà không quan tâm kiến trúc bên dưới của chúng.
Mô hình OSI chia thành bảy tầng mỗi tầng có chức năng riêng biệt nhưng vẫn có mối liên hệ với nhau. Mỗi tầng định nghĩa một quá trình truyền tin trên mạng.

Cụ thể chức năng của từng tầng trong mô hình OSI:

| Tầng | Chức năng |
| ----------- | ------------------------------------------- |
| Tần vật lý | Chuyển dữ liệu **thành tín hiệu và truyền** |
| Tầng liên kết dữ liệu | Truyền thông giữa **hai nút mạng kế tiếp với nhau. Sử dụng địa chỉ MAC** |
| Tầng mạng | **Chọn đường (định tuyến) và chuyển tiếp** gói tin từ nguồn đến đích. Sử dụng đỉa chỉ IP |
| Tầng giao vận | Xử lý việc **truyền-nhận dữ liệu cho các ứng dụng** chạy trên các thiết bị đầu cuối. |

Sau đây ta sẽ đi làm rõ hơn về ba tầng là tầng liên kết dữ liệu, tầng mạng và tầng giao vận xem nó hoạt động như thế nào ?

### 2.2 Tầng liên kết dữ liệu

#### Nhiệm vụ

Giao thức tầng liên kết dữ liệu được sử dụng để truyền gói tin trên một môi trường vật lý. Giao thức ở tầng này định nghĩa khuôn dạng dữ liệu truyền giữa các nút ở đầu của mỗi đường truyền cũng như công việc mà các nút thực hiện khi nhận và gửi đơn vị dữ liệu này.

#### Mạng cục bộ LAN

Mạng LAN là mạng máy tính giới hạn trong một khu vực địa lý. Các máy tính trong mạng LAN có thể chia sẻ tài nguyên với nhau. Thông thường khi truy cập internet từ cơ quan hay trường học mọi người thường truy cập qua LAN. Khi đó máy tính của người dùng là một nút trong mạng LAN và mạng LAN cung cấp khả năng truy cập internet thông qua router.

Mỗi thiết bị trên mạng LAN được định danh duy nhất bởi địa chỉ MAC. Nhờ địa chỉ MAC đó mà hai thiết bị trong cùng một mạng LAN có thể giao tiếp với nhau thông qua các gói tin đã được đóng gói địa chỉ MAC nguồn và MAC đích. Cụ thể định dạng gói tin LAN như sau:
![Định dạng gói tin LAN](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/LAN.PNG)

Các gói tin trong cùng một mạng LAN có thể chuyển giữa các thiết bị bằng địa chỉ MAC nhờmột thiết bị gọi là **switch**. Vậy **swicth** có đặc điểm gì và cơ chế hoạt động của nó cụ thể ra sao.

Switch là một thiết bị tầng liên kết dữ liệu dùng để kết nối hai hoặc nhiều mạng LANs.

Cơ chế hoạt động của switch: Khi một gói tin được chuyển đến **switch**. Switch sẽ tách trường địa chỉ(địa chỉ MAC) trong gói tin và tìm kiếm trong bảng của switch để biết gói tin sẽ được truyền đến đâu.

### 2.3 Tầng mạng

Như ở phần trước ta đã thấy hai máy tính hoàn toàn có thể giao tiếp dữ liệu với nhau thông qua tầng liên kết dữ liệu và địa chỉ MAC vậy tại sao lại cần đến tầng mạng và địa chỉ IP. Nội dung phần này sẽ đi làm rõ sự cần thiết của tầng mạng và địa chỉ IP.

Trước tiên cần chỉ rõ sự khác biệt giữa địa chỉ MAC và địa chỉ IP:

- Địa chỉ MAC là địa chỉ gắn cố định trên card mạng do nhà sản xuất cung cấp gắn. Trên thực tế một nhà sản xuất ra một card mạng sẽ không thể biết card mạng của mình sẽ được bán đi đâu, cho ai. Do vậy địa chỉ MAC mặc dù có thể giúp phân biệt hai thiết bị mạng nhưng lại không có ý nghĩa thực tế trong việc tìm ra thiết bị có địa chỉ đó trong thực tế. Giả sử trên thế giới có năm triệu máy nằm rải rác trên toàn thế giới. Một thiết bị mạng có địa chỉ MAC _00:0A:95:9D:68:16_ cần gửi một gói tin đến thiết bị khác có địa chỉ MAC _00:A0:C9:14:C8:29_. Nó sẽ phải đi hỏi lần lượt các máy trong năm triệu máy đó xem máy nào có địa chỉ MAC là _00:A0:C9:14:C8:29_ và điều này là hoàn toàn không khả thi trong thực tế. Do vậy cần một loại địa chỉ mới thay thế cho địa chỉ MAC trong thực tế để xác định một thiết bị mạng trên thực tế. Đó là địa chỉ IP. Vậy cụ thể địa chỉ IP là gì, có cấu trúc như thế nào tại sao nó lại có thể giúp tìm kiếm thiết bị mạng khác trong thực tế.

Địa chỉ IP là một địa chỉ được gán cho mỗi thiết bị **khi truy cập vào internet**. Cấu trúc của địa chỉ IP gồm hai phần cụ thể như sau:

- HostID – Phần địa chỉ máy trạm
- NetworkID – Phần địa chỉ mạng

ví dụ một địa chỉ IP:
![Cấu trúc địa chỉ  IP](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/IP.PNG)

Địa chỉ IP có các dạng là :

- Network Address: Định danh cho một mạng . Tất cả bit phân HostID là 0
- Broadcast Address: Địa chỉ để gửi dữ liệu cho tất cả các máy trạm trong mạng. Tất cả các bit phần HostID là 1.
- Unicast Address: Gán cho một cổng mạng.
- Multicast address: Định danh cho một nhóm

Vậy sự khác biệt giữa địa chỉ IP và địa chỉ MAC cụ thể như thế nào: Địa chỉ IP có tính phân cấp(gồm các phần khác nhau) và được phân theo vùng địa lý giúp cho việc tìm kiếm trên thực tế nhanh hơn.
Ví dụ khi một thiết bị muốn gửi một gói tin đến địa chỉ IP **202.191.57.25** nó biết được máy tính đó ở Việt Nam và sẽ gửi đến Việt Nam. Cơ chế xác định đó được thực hiện thông qua router.

Vậy router là gì và chức năng chính làm gì:
Router là một thiết bị tầng mạng dùng để chuyển gói tin qua một liên mạng đến các thiết bị đầu cuối thông qua việc định tuyến và chuyển tiếp:

- Định tuyến: Tìm đường đi qua các nút trung gian để gửi dữ liệu từ nguồn đến đích. Giao thức định tuyến xác định đường đi ngắn nhất giữa hai điểm truyền tin.
- Chuyển tiếp: Chuyển gói tin trên cổng vào tới cổng ra theo tuyến đường. Bảng chuyển tiếp xác định cổng ra để chuyển tiếp dữ liệu.

Bây giờ một máy tính khi tham gia vào mạng tồn tại hai địa chỉ một là địa chỉ IP một là đỉa chỉ MAC cần có một cơ chế để xác định địa chỉ MAC của một máy tính trọng mạng khi biết địa chỉ IP của nó. Giao thức ARP giúp giải quyết vấn đề đó. Vậy giao thức ARP hoạt động như thế nào?

 Khi một thiết bị mạng muốn biết địa chỉ MAC của một thiết bị nào đó mà nó đã biết địa chỉ IP nó sẽ gửi một ARP request bao gồm địa chỉ MAC của nó và địa chỉ IP của máy nó cần xác định địa chỉ MAC trên toàn bộ một miền broadcast. Mỗi thiết bị trên mạng broadcast sẽ so khớp địa chỉ IP của gói tin đó với địa chỉ IP của mình nếu trùng khớp nó sẽ gửi lại cho máy gửi ARP request một gói tin trong đó chứa địa chỉ MAC của mình. Sau đó hai thiết bị bắt đầu trao đổi thông tin với nhau.

Ví dụ về một máy tính A muốn gửi một gói tin đến một máy B trong cùng một mạng LAN. Trong trường hợp A biết địa chỉ IP của B

Giả sử:

- A có địa chỉ MAC là 00:A0:C9:14:C8:29
- B có địa chỉ MAC là 00:0A:95:9D:68:16 và địa chỉ IP là :202.191.57.25

![2 máy cùng LAN](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/2m.png)

Quá trình gửi được thực hiện qua các bước:

Bước 1: A gửi gói tin ARQ broadcast đến switch sau đó switch gửi gói tin đó tới toàn bộ các máy tính trong mạng LAN(trong trường hợp này B,C,D,E,F).
Bước 2: Máy B nhận được gói tin ARP broadcast tách lấy trường địa chỉ IP(202.191.57.25) và thấy là trùng với IP của mình.
Bước 3: B gửi cho A một gói tin trong đó chứa địa chỉ MAC của B(00:0A:95:9D:68:16) cho A.
Bước 4: A sử dụng địa chỉ MAC của B gửi gói tin cần gửi cho B gửi tới switch và switch chuyển đến B thông qua bảng của switch đã trình bày ở trên.

Ví dụ trên ta xét trường hợp hai máy tính cùng mạng LAN bây giờ ta xét trường hợp Host 1 thuộc mạng LAN CS muốn gửi một gói tin tới Host 4(192.32.63.8) thuộc mạng LAN EE. Địa chỉ của các host được cho trong hình dưới:

![Chuyển gói tin giữu hai mạng LAN](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/ARP.PNG)

Quá trình thực hiện qua các bước:

Bước 1: Host 1 gửi gói tin ARP broadcast đến switch sau đó switch gửi gói tin đó đến toàn bộ máy trong mạng CS và không nhận được phản hồi từ bất kỳ máy nào(không có máy nào trong mạng CS có địa chỉ MAC là E6).
Bước 2: Switch gửi gói tin tới router(default gateway). Để thuận tiện default gateway thường là địa chỉ nhỏ nhất trong mạng(198.31.65.1).
Bước 3: Router nhận được gói tin tách lấy trường địa chỉ(192.32.63.8) trong gói tin và dựa vào mặt nạ mạng của địa chỉ để biết gói tin đó cần gửi tới mạng EE.
Bước 4: Switch trong mạng EE nhận được gói tin và tiến hành quảng bá trong mạng EE để tìm địa chỉ MAC(E6) có địa chỉ IP là 192.32.63.8 để truyền gói tin. Tìm hiểu thêm tại “Computer network của tanenbaun trang 469”

Hai quá trình gửi giữ liệu trên ta đều đang xét gói dữ liệu nhỏ có thể truyền trực tiếp trên đường truyền. Nếu gói tin có kích thươc lớn (1GB) thì quá trình gửi sẽ như thế nào ?

Khi tầng mạng nhận được một gói tin lớn (1GB) từ tầng giao vận chuyển xuống. Tầng mạng dựa vào giá trị MTU (kích thươc đơn vị dữ liệu tối đa) của đường truyền để chia gói tin lớn thành các gói tin nhỏ. Gói tin lớn được phân mảnh và chuyển đi sau đó được ghép tại máy đích.

### 2.4 Tầng giao vận

Trong phần trên đã giúp ta hiểu quá trình gói tin truyền từ hai thiết bị mạng có thể truyền gói tin cho nhau nhưng trong mỗi máy tính có rất nhiều ứng dụng làm sao để biết gói tin cần chuyển cho ứng dụng nào trong máy tính. Đó là nhiệm vụ của tầng giao vận.

Có hai giao thức được sử dụng ở tầng giao vận là TCP và UDP. Bảng dưới đây cho thấy sự khác biệt giữa hai giao thức này.

#### TCP vs UDP

. | TCP | UDP
---------|----------|---------
  | Tin cậy , hướng liên kết | Không tin cậy, không liên kết
 Đơn vị truyền | Segment | datagram
 Trường hợp sử dụng | Các ứng dụng cần dịch vụ với 100% độ tin cậy như mail,web | Các ứng dụng cần chuyển nhanh dữ liệu có khả năng chịu lỗi như Video Streaming,...

### Network address translation (NAT)

Trong địa chỉ IPv4, có hai loại đỉa chỉ là IP Pubic và IP Private.

IP Public là địa chỉ IP được cung cấp bởi ISP(Nhà cung cấp dịch vụ mạng) khi truy cập vào internet. Nó dùng để xác định máy tính của bạn trên internet là duy nhất. Khác với IP Public địa chỉ IP Private dùng để xác định định máy tính của bạn trong một mạng cục bộ. Khi các máy tính trong cùng một mạng cục bộ trao đổi với nhau chúng có thể dùng địa chỉ IP Private nhưng khi muốn trao đổi với một máy tính ở ngoài mạng cục bộ nó cần một địa chỉ IP Public. Do vậy cần có một kỹ thuật để chuyển đổi một địa chỉ IP Private trong mạng cục bộ thành địa chỉ IP Public. NAT được sử dụng để giải quyết vấn đề này. Có ba loại NAT: NAT tĩnh, NAT động và NAT vượt tải.

- NAT tĩnh: Một địa chỉ IP nội miền được ánh xạ với một địa chỉ IP ngoại miền.
- NAT động:  địa chỉ IP nội bộ sẽ được tự động khớp với một bộ địa chỉ IP ngoài. Quá trình vẫn là ánh xạ một-một nhưng được diễn ra tự động.
- NAT overload: Với NAT overload ánh xạ một-một như NAT động và NAT tĩnh không được sử dụng thay vì một địa chỉ ngoài chỉ được ánh xạ với một địa chỉ trong mạng nội bộ thì bây giờ nó có thể gán cho tất cả các máy trong mạng nội bộ dựa trên số cổng. Chỉ khi số lượng các cổng khả dụng sử dụng với địa chỉ IP ngoài cạn kiệt thì một địa chỉ IP ngoài thứ hai mới được sử dụng với phương pháp tương tự.

#### Các bước một gói tin TCP từ một máy tính cục bộ đi tới trang facebook.com.vn (IP là 191.58.58.59, port 433) thông qua giao thức overload NAT

![gửi tới facebook](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/Screenshot%20from%202017-03-30%2014-59-42.png)

Quá trình gửi gói tin thực hiện qua các bước:
Bước 1: Gói tin được gửi từ host 1 có địa chỉ IP Private 10.0.0.2 cổng 1000 gửi tới NAT.
Bước 2: NAT ảnh xạ địa chỉ IP Private(10.0.0.2 cổng 1000) thành địa chỉ IP Public 200.31.32.2 cổng 2000.
NAT đóng gói lại gói tin với trường địa chỉ nguồn mới là IP Public + cổng (200.31.32.2 cổng 2000).
Bước 3: NAT gửi gói tin đến facebook(thông qua cơ chế định tuyến và chuyển tiếp của router đã nói đến ở phần trên).


Tài liệu tham khảo :

1. Slide Mạng máy tính thầy Bùi Trọng Tùng
1. Slide Mạng máy tính cô Trương diệu Linh
1. Computer Network tanenabun
1. [https://quantrimang.com/network-address-translation-nat-hoat-dong-nhu-the-nao-phan-1-118495](https://quantrimang.com/network-address-translation-nat-hoat-dong-nhu-the-nao-phan-1-118495)
1. [https://quantrimang.com/tim-hieu-ve-cau-hinh-nat-phan-2-118501](https://quantrimang.com/tim-hieu-ve-cau-hinh-nat-phan-2-118501)
1. [https://quantrimang.com/dynamic-nat-nat-dong-va-overloading-nat-hoat-dong-nhu-the-nao-phan-3-118518](https://quantrimang.com/dynamic-nat-nat-dong-va-overloading-nat-hoat-dong-nhu-the-nao-phan-3-118518)
1. [https://quantrimang.com/tim-hieu-ve-nat-phan-cuoi-118574](https://quantrimang.com/tim-hieu-ve-nat-phan-cuoi-118574)
1. Giáo trình nhập môn mạng máy tính thầy Hồ Đắc Phương