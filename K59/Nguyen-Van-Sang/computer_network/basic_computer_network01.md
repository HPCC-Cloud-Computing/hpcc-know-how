# Mạng máy tính
<!-- TOC -->

- [Mạng máy tính](#m-ng-m-y-t-nh)
- [Chương 1: Giới thiệu cơ bản về mạng máy tính](#ch-ng-1-gi-i-thi-u-c-b-n-v-m-ng-m-y-t-nh)
    - [1.1 Các khái niệm cơ bản](#1-1-c-c-kh-i-ni-m-c-b-n)
        - [Mạng máy tính là gì?](#m-ng-m-y-t-nh-l-g)
        - [Topology](#topology)
        - [Protocal](#protocal)
    - [1.2 Tầng](#1-2-t-ng)
        - [1.2.1  Mô hình OSI(7 tầng)](#1-2-1-m-h-nh-osi-7-t-ng)
        - [1.2.2 Mô hình TCP/IP](#1-2-2-m-h-nh-tcp-ip)
- [Chương 2: LAN](#ch-ng-2-lan)
    - [2.1 Nhiệm vụ của LAN](#2-1-nhi-m-v-c-a-lan)
    - [2.2 Đình dạng gói tin trong LAN](#2-2-nh-d-ng-g-i-tin-trong-lan)
    - [2.3 Phân biệt giữa Hub, Switch](#2-3-ph-n-bi-t-gi-a-hub-switch)
        - [Hub](#hub)
        - [Switch](#switch)
            - [Bài toán chuyển gói tin trong 1 mạng Lan](#b-i-to-n-chuy-n-g-i-tin-trong-1-m-ng-lan)
- [Chương 3: Internet Layer](#ch-ng-3-internet-layer)
    - [3.1 Internet Protocol](#3-1-internet-protocol)
    - [3.2 Địa chỉ IP](#3-2-a-ch-ip)
    - [3.3 Giao thức ARP](#3-3-giao-th-c-arp)
        - [Cơ chế hoạt động](#c-ch-ho-t-ng)
    - [3.4 Router](#3-4-router)
        - [Bài toán chuyển gói tin trong 2 mạng Lan khác nhau](#b-i-to-n-chuy-n-g-i-tin-trong-2-m-ng-lan-kh-c-nhau)
- [Chương 4: TCP,UDP](#ch-ng-4-tcp-udp)
    - [4.1 Phân biệt TCP và UDP](#4-1-ph-n-bi-t-tcp-v-udp)
        - [TCP](#tcp)
        - [UDP](#udp)
        - [Điểm giống nhau](#i-m-gi-ng-nhau)
        - [Điểm khác nhau](#i-m-kh-c-nhau)
    - [4.2 NAT](#4-2-nat)
        - [Tại sao lại cần NAT?](#t-i-sao-l-i-c-n-nat)
        - [NAT là gì?](#nat-l-g)
        - [Nhiệm vụ:](#nhi-m-v)
        - [Các loại NAT](#c-c-lo-i-nat)
            - [Bài toán chuyển gói tin sử dụng Overloading NAT](#b-i-to-n-chuy-n-g-i-tin-s-d-ng-overloading-nat)

<!-- /TOC -->
# Chương 1: Giới thiệu cơ bản về mạng máy tính
## 1.1 Các khái niệm cơ bản
###  Mạng máy tính là gì?
Tập hợp các máy tính kết nối với nhau bằng một đường truyền vật lí theo một kiến trúc nào đó để có thể trao đổi dữ liệu cho nhau.
+ Máy tính: máy chủ (chia sẻ tài nguyên của nó cho các máy tính khác trong mạng), các máy trạm (các máy tính cá nhân kết nối với máy chủ và sử dụng tài nguyên được chia sẻ từ máy chủ),thiết bị ngoại vi(máy in, ổ đĩa cứng ...), card mạng (điều khiển việc truyền chính xác dữ liệu tới các nút mạng, và chuyển đổi dữ liệu sang dạng điện hay quang),..
+ Kết nối bằng 1 phương tiện truyền (như cáp mạng,..) -> tạo ra đường truyền vật lí để liên kết các nút mạng, truyền dẫn các tín hiệu điện hay quang
+ Theo 1 kiến trúc mạng (topology + protocol)
###  Topology 
Là cẩu trúc hình học không gian của mạng (thực chất là cách bố trí các phần tử của mạng cũng như cách nối giữa chúng với nhau)

Có 3 dạng topology(hình trạng) cơ bản:
+ Mạng hình sao(star topology)
+ Mạng dạng tuyến(bus topology)
+ Mạng hình vòng(ring topology)

Tuy nhiên trên thực tế luôn có sự kết hợp giữa các hình trạng mạng để tạo thành mạng phức tạp hơn như StarBus, StarRing ...
### Protocal 
Khi 2 máy khác nhau muốn giao tiếp với nhau thì chúng cần một protocol(giao thức).

Chúng ta hiểu nôm na protocol là sự thống nhất về cách thức trao đổi thông tin giữa các máy.

Để chuẩn hơn ta định nghĩa như sau:
+ Protocol là quy tắc, quy ước truyền thông ao gồm việc gửi và nhận các thông điệp bao gồm: **Gửi** một yêu cầu hoặc thông tin và  **Nhận** một thông tin hoặc yêu cầu hành động
+ Protocol là việc xác định khuôn dạng dữ liệu, thông điệp; thứ tự truyền nhận thông điệp giữa các thực thể trên mạng; cũng như các hành động tương ứng khi nhận được thông điệp
+ Ví dụ một vài giao thức mạng: TCP, UDP, IP, HTTP, Telnet, Ethernet, ...

## 1.2 Tầng
**Tâng là gì?**
Tầng(layer) là cách tổ chức chương trình thành các thành phần chức năng riêng biệt theo một cách nhất định và theo thứ bậc. Mỗi tầng thường có một giao diện duy nhất cung cấp cho tầng bên trên và sử dụng giao thức do tầng bên dưới cung cấp còn lại sẽ độc lập với các tầng khác

>Vậy tại sao cần phải phân tầng?
>+ Áp dụng nguyên lý chia để trị đối với các hệ thống phức tạp
>+ Cho phép xác định rõ nhiệm vụ của mỗi bộ phận và quan hệ giữa chúng
>+ Cho phép dễ dàng bảo trì và nâng cấp hệ thống
### 1.2.1  Mô hình OSI(7 tầng)

**Tầng 1: Tầng vật lý (Physical Layer)**
Điều khiển việc truyền tải thật sự các bit trên đường truyền vật lý. Nó định nghĩa các tín hiệu điện, trạng thái đường truyền, phương pháp mã hóa dữ liệu, các loại đầu nối được sử dụng ...

**Tầng 2: Tầng liên kết dữ liệu (Data-Link Layer)**
Tầng này đảm bảo truyền tải các khung dữ liệu (Frame) giữa hai máy tính có đường truyền vật lý nối trực tiếp với nhau. Nó hỗ trợ cơ chế phát hiện và xử lý lỗi dữ liệu.

**Tầng 3: Tầng mạng (Network Layer)**
Tầng này đảm bảo cho việc truyền các gói tin dữ liệu (Packet) giữa 2 máy tính bất kỳ trong mạng máy tính(có thể có hoặc không có kết nối đường truyền vật lí trực tiếp). Nói cách khác, tầng mạng nhận nhiệm vụ tìm đường đi cho dữ liệu đến các đích khác nhau trong mạng.

**Tầng 4: Tầng vận chuyển (Transport Layer)**
Tầng này đảm bảo truyền tải dữ liệu giữa các quá trình. Dữ liệu gởi đi được đảm bảo không có lỗi, theo đúng trình tự, không bị mất mát, trùng lắp. Đối với các gói tin có kích thước lớn, tầng này sẽ phân chia chúng thành các phần nhỏ trước khi gởi đi, cũng như tập hợp lại chúng khi nhận được.

**Tầng 5: Tầng phiên (Session Layer)**
Quản lý các phiên làm việc giữa các người sử dụng. Tầng phiên cung cấp cơ chế và chức năng bảo mật thông tin khi truyền qua mạng máy tính.

**Tầng 6: Tầng trình bày (Presentation Layer)**
Tầng này đảm bảo các máy tính có kiểu định dạng dữ liệu khác nhau vẫn có thể trao đổi  thông tin cho nhau. Thông thường các mày tính sẽ thống nhất với nhau về một kiểu định dạng dữ liệu trung gian để trao đổi thông tin giữa các máy tính. Một dữ liệu cần gửi đi sẽ được tầng trình bày chuyển sang định dạng trung gian trước khi nó được truyền lên mạng. Ngược lại, khi nhận dữ liệu từ mạng, tầng trình bày sẽ chuyển dữ liệu sang định dạng riêng của nó.

**Tầng 7: Tầng ứng dụng (Application Layer)**
Đây là tầng trên cùng, cung cấp các ứng dụng truy xuất đến các dịch vụ mạng. Nó bao gồm các ứng dụng của người dùng, ví dụ như Web Browser,Mail User Agent ... hoặc các chương trình làm server cung cấp các dịch vụ mạng như Web Server, FTP Server, Mailserver... Người dùng mạng giao tiếp trực tiếp với tầng này.


### 1.2.2 Mô hình TCP/IP

**Tầng 1: Tầng vật lý** 

**Tầng 2: Tầng liên kết dữ liệu**

**Tầng 3: Tầng mạng - sử dụng giao thức IP**

**Tầng 4: Tầng giao vận - sử dụng giao thức TCP, UDP**

**Tầng 5: Tầng ứng dụng - bao gồm chức năng của cả 3 tâng 5, 6, 7 trong mô hình OSI**


# Chương 2: LAN
## 2.1 Nhiệm vụ của LAN

**LAN là gì?**

 LAN(Local Address Network) là một hệ thống máy tính được kết nối với nhau để truyền tải dữ liệu và chia sẻ tập tin cho nhau.
## 2.2 Đình dạng gói tin trong LAN

![H0](http://i.imgur.com/quUyZPA.png)

## 2.3 Phân biệt giữa Hub, Switch 

### Hub

**Hub** thường là thiết bị được dùng để nối mạng, thông qua những đầu cắm của nó người ta liên kết với các máy tính dưới dạng hình sao. Vai trò  của Hub là đại tín hiệu vật lí ở đầu vào và cung cấp năng lượng cho tín hiệu ở đầu ra.

>Câu hỏi: Tại sao lại sinh ra hub? -Do giới hạn của cáp mạng, sự suy hao trên đường ⇒ hub ra đời để truyền tín hiệu đi xa hơn và đảm bảo độ ổn định của tín hiệu

>Note: Khi một máy truyền tín hiệu đến Hub thì Hub sẽ chuyển tín hiệu đó cho tất cả các máy khác có kết nối với nó, hiểu nôm na là nếu có gói tin được truyền tới hub thì hub sẽ gửi nó cho tất cả các máy khác có nối với nó


### Switch

Switch là 1 thiết bị mạng Lan nhiều cổng, các máy trạm nối với switch thông qua các cổng của chúng (các switch cũng có thể nối với các switch khác để tạo thành mạng Lan lớn hơn) qua đó hình thành các các liên kết giữa các máy với nhau:
+ Switch sẽ quan sát các gói tin trên mạng đồng thời cũng học thông tin của mạng qua các gói tin mà nó nhận được, sử dụng thông tin này để xây dựng **bảng đ/c MAC** (Đ/c MAC máy trạm, số hiệu cổng, TTL) cho biết máy nào ở cổng nào để từ đó chuyển tiếp gói tin đến đúng đich.
+ Điểm nổi bật của switch là **cơ chế tự học**: nó tự nhận biết được địa chỉ MAC của các máy nối vào.


#### Bài toán chuyển gói tin trong 1 mạng Lan

Khi gói tin đi qua 1 port của Switch, tại đây Switch sẽ mở nó ra, đọc địa chỉ MAC nguồn và lưu vào đ/c MAC của nó với số port và MAC nguồn kết nối trực tiếp với port đó của switch; Tiếp đó đọc địa chỉ MAC đích của gói tin, tiến hành tìm kiếm trong bảng đ/c MAC của nó:
+ Nếu MAC đích tồn tại trong bảng đ/c MAC, switch sẽ gửi gói tin qua port tương ứng và máy đích sẽ nhận được
+ Nếu MAC đích không tồn tại trong bảng đ/c MAC, hoặc nó là 1 địa chỉ broadcast thì switch sẽ gửi gói tin đến tất cả các port trừ cổng nhận vào, gói tin được truyền đến tất cả các máy, được bóc ra và so sánh địa chỉ MAC đích trên gói tin, nếu máy nào trùng với ddiacj chỉ MAC của mình thì gói tin được giữ lại, nếu không gói tin sẽ bị hủy
+ Nếu MAC đích trùng MAC nguồn thì gói tin sẽ bị drop


Ví dụ: Máy 1 gửi gói tin tới Máy 4

![H1](http://i.imgur.com/dS4QJXc.png)

# Chương 3: Internet Layer

## 3.1 Internet Protocol 

Là một giao thức tầng mạng.
Có 2 chức năng cơ bản:
+ Chọn đường(Routing): Xác định đường đi của gói tin từ nguồn tới đích
+ Chuyển tiếp(Forwarding): Chuyển tiếp dữ liệu từ đầu vào tới đầu ra của bộ định tuyến


Đặc điểm của giao thức IP:
+ Là giao thức truyền tin nhanh: truyền dữ liệu theo phương thức "best offort"
+ Là giao thức không tin cậy: không có cơ chế phục hồi lỗi, khi cần sẽ sử dụng dịch vụ tầng trên để đảm bảo độ tin cậy
+ Là giao thức không liên kết: các gói tin được xử lý độc lập

## 3.2 Địa chỉ IP
**Địa chỉ IP là gì?**
Để gửi gói tin ta cần phải biết địa chỉ của máy đích, luôn phải có một loại địa chỉ để xác định vị trí, từ đó trao đổi thông ti chính xác từ máy nguồn tới máy đích. Vậy nên trong Internet địa chỉ IP là duy nhất.

**Cấu trúc địa chỉ IP**
IP là một giải nhị phân dài 32 bit, gồm Network ID dùng để xác định mạng mà thiết bị kết nối vào và phần Host ID để xác định thiết bị của mạng đó.

![H3](http://i.imgur.com/aIQnEDq.png)

Để cho đơn giản người ta thường viế lại địa chỉ IP dưới dạng 4 số thập phân được cách nhau bởi dấu chấm.
>Ví dụ: 11001011 10110010 10001111 01100100 sẽ có dạng thập phân là  203.178.143.10/14 (/14 là mặt nạ mạng cho biết số bit thuộc phần Network ID) 

Các loại địa chỉ IP:

+ Địa chỉ Unicast: khi bạn muốn gửi gói tin đến một máy tính cụ thể, khi đó địa chỉ để bạn gửi tới sẽ là một địa chỉ unicast. Đây đơn giản là địa chỉ IP của một thiết bị nào đó trong cùng mạng cục bộ hoặc khác mạng cục bộ.
+ Địa chỉ Multicast: trong trường hợp bạn muốn gửi gói tin đến nhiều máy tính, ta sẽ gửi một địa chỉ multicast, địa chỉ này đại diện cho một nhóm các thiết bị
+ Địa chỉ Broadcast: dùng để gửi thông điệp đến tất cả các máy trong mạng nội bộ, địa chỉ Broadcast có toàn bộ các bits phần Host IP bằng 1 và dại diện cho toàn bộ các thiết bị trong mạng
+ Địa chỉ mạng: dùng để xác định chính xác mạng đó. Địa chỉ mạng địa chỉ có tất cả các bits phần Host đều bằng 0
+ Default Gateway: trước tiên là một địa chỉ (còn được gọi là cổng mặc định).Địa chỉ này được cấu hình cho máy tính và khi một gói tin được gửi đến một địa chỉ không cùng mạng, hoặc đơn giản là không biết gửi đi đâu thì gói tin sẽ được gửi đến địa chỉ này để tiếp tục đi đến nơi khác

## 3.3 Giao thức ARP

Là giao thức phân giải địa chỉ: chuyển đổi từ địa chỉ IP sang địa chỉ MAC.

### Cơ chế hoạt động

Trong 1 mạng lan, máy A muốn gửi 1 gói tin cho máy B nhưng chỉ biết được IP của B, để làm được điều đó A cần phải biết được địa chỉ MAC của B, để A gắn địa chỉ này vào gói tin giúp gói tin chuyển đi đến đúng được B.

Các bước để A xác định MAC của B:

+ A sẽ kiểm tra cache của mình(APR table: <IP address, MAC address, TTL>),nếu tìm thấy MAC của B thì sẽ tiến hành thêm MAC đích vào gói tin rồi truyền đi
+ Nếu không tìm thấy, A sẽ gửi 1 gói tin broadcast (APR Request)đến các máy khác trong mạng( trong đó có MAC nguồn, IP nguồn của A, IP đích của B và MAC đích mặc định là: ff:ff:ff:ff:ff:ff)
+ Các máy còn lại trong mạng sẽ so sánh IP của mình với IP đích,B biết được máy A cần tìm là nó, khi đó B sẽ tạo gói tin APR Replay (chứa MAC của B) rồi gửi lại cho A, đồng thời nhập MAC, IP của A vào APR Table của mình.
+ Khi A nhận được gói tin do B gửi tới, nó sẽ cập nhật MAC, IP của B vào ARP Table (lần dùng sau nó sẽ không phải request nữa)

## 3.4 Router

Router là một thiết bị hoạt động trên tầng mạng, nó có thể tìm được đường đi tốt nhất cho các gói tin qua nhiều kết nối để đi từ trạm gửi thuộc mạng đầu đến trạm nhận thuộc mạng cuối. Router có thể được sử dụng trong việc nối nhiều mạng với nhau và cho phép các gói tin có thể đi theo nhiều đường khác nhau để tới đích.

Router có địa chỉ riêng biệt và nó chỉ tiếp nhận và xử lý các gói tin gửi đến nó mà thôi. Khi một trạm muốn gửi gói tin qua Router thì nó phải gửi gói tin với địa chỉ trực tiếp của Router (Trong gói tin đó phải chứa các thông tin khác về đích đến) và khi gói tin đến Router thì Router mới xử lý và gửi tiếp.

Khi xử lý một gói tin Router phải tìm được đường đi của gói tin qua mạng. Để làm được điều đó Router phải tìm được đường đi tốt nhất trong mạng dựa trên các thông tin nó có về mạng, thông thường trên mỗi Router có một bảng chỉ đường (Router table). Dựa trên dữ liệu về Router gần đó và các mạng trong liên mạng, Router tính được bảng chỉ đường (Router table) tối ưu dựa trên một thuật toán xác định trước.

![H4](http://i.imgur.com/yLXj0yz.png)

### Bài toán chuyển gói tin trong 2 mạng Lan khác nhau

Trường hợp gói tin gửi đi mà máy nhận nằm khác mạng với máy gửi,thì gói tin tiếp tục được chuyến đến router để xử lý:
+ Router gỡ bỏ lớp Header của DataLink(gồm MAC nguồn, MAC cuối),sau đó đọc thông tin lớp Network(gồm IP nguồn, IP đích)
+ Router lấy IP đích, so sánh với IP trong Routing Table:

     *Nếu không tìm được đường đi ứng với IP đích hoặc TTL(trong IP Header) = 0(trường hợp lặp vô tận ) ⇒ gói tin bị drop và router gửi thông báo không tìm thấy máy đích về cho máy gửi gói tin*

     *Nếu tìm được đường đi ứng với IP đích thì router sẽ thêm lại header chứa: MAC nguồn mới - là địa chỉ MAC của router này, MAC đích mới - là địa chỉ MAC của router tiếp theo. Quá trình này lặplại cho đến khi router phát hiện ra IP đích nằm chung mạn với interface của router ⇒ Router sẽ dùng giao thức ARP để xác thực MAC của máy đích, MAC này sẽ được sử dụng làm MAC đích để máy gửi gói tin đến máy đích*



Ví dụ: Máy 1 ở mạng LAN 1 muốn chuyển một gói tin cho máy Máy 4 ở mạng LAN 2.

![H2](http://i.imgur.com/QNoTTr1.png)

Quá trình 2 máy trên hình vẽ gửi gói tin cho nhau:
>Bước 1: Máy 1 tạo gói tin IP để chuẩn bị gửi cho Máy 4, IP Packet có dạng:

![H2.1](http://i.imgur.com/3O8J9tK.png)

>Bước 2: Máy 1 kiểm tra và biết Máy 4 không cùng mạng LAN với nó, khi đó Máy 1 tiến hành đưa gói tin IP xuống tầng liên kết dữ liệu, đóng gói lại thành Frame và gửi tới cho Router, lúc này Frame có dạng:

![H2.2](http://i.imgur.com/4CjoFm1.png)

>Bước 3: Router nhận được Frame, nó mở ra và bỏ Header của tầng liên kết giữ liệu(gồm MAC nguồn, MAC đích) thu được IP Packet sau:

![H2.1](http://i.imgur.com/3O8J9tK.png)

>tiếp đó router sẽ kiểm tra địa chỉ IP đích, sau đó gắn thêm Header chứa  MAC nguồn là MAC của nó và MAC đích là MAC của Máy 4(hoặc là nút kế tiếp trong đường đi từ Máy 1 đến Máy 4). Lúc này Frame mới tới Máy 4 có dạng:

![H2.3](http://i.imgur.com/HJJggH3.png)

>Bước 4: Máy 4 nhận được Frame từ Router, nó mở ra rồi bỏ MAC Header, kiểm tra IP đích trùng với IP của nó và tiếp tục bóc bỏ IP Header để lấy dữ liệu từ máy A gửi tới 
# Chương 4: TCP,UDP

## 4.1 Phân biệt TCP và UDP


### TCP
TCP (Transmission Control Protocol - "Giao thức điều khiển truyền vận") là một trong các giao thức cốt lõi của bộ giao thức TCP/IP. Sử dụng TCP, các ứng dụng trên các máy chủ được nối mạng có thể tạo các "kết nối" với nhau, mà qua đó chúng có thể trao đổi dữ liệu hoặc các gói tin. Giao thức này đảm bảo chuyển giao dữ liệu tới nơi nhận một cách đáng tin cậy và đúng thứ tự. TCP còn phân biệt giữa dữ liệu của nhiều ứng dụng (chẳng hạn, dịch vụ Web và dịch vụ thư điện tử) đồng thời chạy trên cùng một máy chủ.

### UDP
UDP (User Datagram Protocol) là một trong những giao thức cốt lõi của giao thức TCP/IP. Dùng UDP, chương trình trên mạng máy tính có thể gởi những dữ liệu ngắn được gọi là datagram tới máy khác. UDP không cung cấp sự tin cậy và thứ tự truyền nhận mà TCP làm; các gói dữ liệu có thể đến không đúng thứ tự hoặc bị mất mà không có thông báo. Tuy nhiên UDP nhanh và hiệu quả hơn đối với các mục tiêu như kích thước nhỏ và yêu cầu khắt khe về thời gian. Do bản chất không trạng thái của nó nên nó hữu dụng đối với việc trả lời các truy vấn nhỏ với số lượng lớn người yêu cầu.

### Điểm giống nhau
Đều là các giao thức mạng TCP/IP, đều có chức năng kết nối các máy lại với nhau, và có thể gửi dữ liệu cho nhau....

### Điểm khác nhau

Header của TCP và UDP khác nhau ở kích thước (20 và 8 byte) nguyên nhân chủ yếu là do TCP phải hộ trợ nhiều chức năng hữu ích hơn(như khả năng khôi phục lỗi). UDP dùng ít byte hơn cho phần header và yêu cầu xử lý từ host ít hơn

TCP :
+ Dùng cho mạng WAN 
+ Không cho phép mất gói tin 
+ Đảm bảo việc truyền dữ liệu 
+ Tốc độ truyền thấp hơn UDP

UDP: 
+ Dùng cho mạng LAN 
+ Cho phép mất dữ liệu 
+ Không đảm bảo.
+ Tốc độ truyền cao, VolP truyền tốt qua UDP

## 4.2 NAT

### Tại sao lại cần NAT?

Số lượng địa chỉ IP là rất lớn, nhưng không phải là vô hạn. Vì vậy để bảo tồn địa chỉ IP, người ta chia địa chỉ IP ra làm 2 loại là địa chỉ public và địa chỉ private.

IP Public là các địa chỉ độc nhất, sử dụng được trong môi trường Internet.

IP Private chỉ sử dụng được trong mạng cục bộ, có thể tái sử dụng lại ở mạng cục bộ khác, nhưng trong một mạng thì vẫn phải mang giá trị duy nhất.

Khi các thiết bị sử dụng địa chỉ IP private trong mạng cục bộ muốn truy cập được Internet – môi trường không sử dụng địa chỉ private, công nghệ NAT (Network Address Translation) được cài đặt trên các thiết bị router(đã được gán 1 địa chỉ IP Public) được sử dụng để chuyển IP private thành IP public và ngược lại, giúp cho các thiết bị trong mạng cục bộ vẫn có thể truy cập được Internet.

### NAT là gì?

+ Là một kỹ thuật để kết nối với Internet. Nó cho phép 1 (hay nhiều) địa chỉ IP nội miền được ánh xạ với 1 (hay nhiều) IP ngoại miền
+ Cho phép 1 thiết bị như Router hoạt động trung gian giữa Internet(Public Network) và Local(Private Network)

### Nhiệm vụ:

+ NAT duy trì 1 bảng thông tin về gói tin được gửi qua.
+ Khi 1 máy tính kết nối đến 1 website, địa chỉ IP nguồn của máy được NAT ánh xạ sang 1 địa chỉ IP Public đã được cấu hình sẵn trên NAT Server.
+ Sau khi có gói tin trở về, NAT dựa vào bảng record mà nó đã lưu về gói tin thay đổi địa chỉ đích thành địa chỉ của máy tính trong mạng và chuyển tiếp 

### Các loại NAT

+ **Static NAT** (NAT tĩnh) là phương thức 1-1. Một địa chỉ IP Private sẽ được Router map với 1 địa chỉ IP Public. Được sử dụng khi thiết bị cần truy cập từ bên ngoài mạng.
+ **Dynamic NAT**: Một địa chỉ IP Private sẽ được Router map với 1 địa chỉ IP Public trong nhóm địa chỉ IP Public.
+ **Overloading NAT**: là 1 dạng thức của Dynamic NAT, nhiều địa chỉ IP Private sẽ được Router map đến 1 địa chỉ IP Public qua các cổng port khác nhau
+ **Overlapping NAT**: khi 1 địa chỉ IP trong mạng nội bộ là IP Public đang sử dụng trên một hệ thống mạng khác, Router phải duy trì 1 bảng tìm kiếm các địa chỉ này để ngăn và thay thế bằng 1 IP Public duy nhất

#### Bài toán chuyển gói tin sử dụng Overloading NAT

Ví dụ : Máy 1 có địa chỉ IP là 137.96.7.14/24, gửi gói dữ liệu X đến Máy 4 có IP là 128.45.18.12/24. Hãy trình bày các bước để gửi gói tin X, biết port của ứng dụng gửi trên Máy 1 là 80, port của ứng dụng nhận của Máy 4 là 500.

![H5](http://i.imgur.com/j2BQejV.png)

+ Dữ liệu X được ứng dụng trên Máy 1 chuyển xuống tầng transport được gắn thêm header transport -> gói segment có dạng | 80 | 500 | Data |
+ Segment chuyển xuống tầng network tiếp tục được gắn thêm IP Header -> IP Packet có dạng | 137.96.7.14/24 | 128.45.18.12/24 | 80 | 500 | Data |
+ IP Packet sẽ chuyển xuống data link, physical và chuyển sang router
+ Router tiến hành bỏ header của data link. Tiếp đó, NAT router dựa trên IP nguồn(137.96.7.14/24) và số hiệu cổng nguồn(80) ánh xạ tới IP Public của router(213.12.12.13/24) với port ánh xạ là 102. Khi đó gói tin gửi từ router có dạng | 213.12.12.13/24 | 128.45.18.12/24 | 102 | 500 | Data |, được router định tuyến và chuyển tiếp tới Máy 4
+ Gói tin tới Máy 4, qua tầng network để bỏ IP header, qua transport để bỏ số hiệu cổng nguồn và đích, đồng thời dựa vào số hiệu cổng đích(port 500) để chuyển đúng X tới ứng dụng nhận của Máy 4














