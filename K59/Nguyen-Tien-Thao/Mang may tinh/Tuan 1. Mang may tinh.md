# Mạng Máy Tính

## Các Khái niệm

### Mạng máy Tính

Mạng máy tính là sự kết hợp các máy tính với nhau thông qua các thiết bị kết nối và phương tiện truyền thông theo một giao thức nào đó giúp các máy tính có thể chia sẻ tài nguyên với nhau. Các máy tính(nodes) kết nối với nhau thông qua dây cáp hoặc qua sóng. Mạng máy tính có thể có nhiều kích thước, hình dạng và cách thức khác nhau. Một mạng máy tính phổ biến nhất hiện nay là mạng internet.

### Tầng

Tầng là cách tổ chức chương trình thành các thành phần chức năng riêng biệt theo một cách nhất định và theo thứ bậc. Mỗi layer thường có một giao diện duy nhất cung cấp cho layer bên trên và sử dụng giao thức do layer bên dưới cung cấp còn lại sẽ độc lập với các layer khác.

### Giao thức

Giao thức là tập hợp các quy tắc quy định khuôn dạng, ngữ nghĩa, thứ tự các thông điệp được gửi và nhận bởi các nút mạng và các hành vi khi trao đổi thông điệp đó. Các giao thức xác định cách tương tác giữa các thực thể trong giao tiếp. Ví dụ giao thức TCP/IP

## Mô hình OSI

## Tổng quan

Mô hình OSI là mô hình cho phép tất cả các hệ thống có thể truyền thông với nhau mà không quan tâm kiến trúc bên dưới của chúng.
Mô hình OSI hồm bảy tầng riêng biệt nhưng vẫn có mối liên hệ với nhau, mỗi tầng định nghĩa một quá trình truyền tin trên mạng.
| Tầng | Chức năng |
| ----------- | ------------------------------------------- |
| Tần vật lý | Chuyển dữ liệu **thành tín hiệu và truyền** |
| Tầng liên kết dữ liệu | Truyền thông giữa **hai nút mạng kế tiếp với nhau. Sử dụng địa chỉ MAC** |
| Tầng mạng | **Chọn đường (định tuyến) và chuyển tiếp** gói tin từ nguồn đến đích. Sử dụng đỉa chỉ IP |
| Tầng giao vận | Xử lý việc **truyền-nhận dữ liệu cho các ứng dụng** chạy trên các thiết bị đầu cuối. |

## Chi tiết

### Tầng liên kết dữ liệu

#### Nhiệm vụ

Giao thức tầng liên kết dữ liệu được sử dụng để truyền gói tin trên một môi trường vật lý. Giao thức ở tầng này định nghĩa khuôn dạng dữ liệu truyền giữa các nút ở đầu của mỗi đường truyền cũng như công việc mà các nút thực hiện khi nhận và gửi đơn vị dữ liệu này.

Các chức năng chính:

    - Đóng gói
        - Đơn vị dữ liệu là frame
        - Bên gửi: thêm header, trailer cho gói tin nhận được từ tầng mạng
        - Bên nhận: bỏ header và trailer chuyển lên tầng mạng.
    - Địa chỉ hóa: Sử dụng địa chỉ MAC
    - Điều khiển truy cập đường truyền
    - Kiểm soát luồng: đảm bảo bên nhận không bị quá tải
    - Kiểm soát lỗi
    - Chế độ truyền: simplex, half-duplex,full-duplex.

#### Mạng cục bộ LAN

Mạng LAN là mạng máy tính giới hạn trong một khu vực địa lý. Các máy tính trong mạng LAN có thể chia sẻ tài nguyên với nhau. Thông thường khi truy cập internet từ cơ quan hay trường học mọi người thường truy cập qua LAN. Khi đó máy tính của người dùng là một nút trong mạng LAN và mạng LAN cung cấp khả năng truy cập internet thông qua router.

Các hình trạng cơ bản của mạng LAN:

- Tất cả các nút sử dụng chung đường truyền-trục.
- Hình Sao
- Ring


Định dạng gói tin của LAN:
![Định dạng gói tin LAN](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/LAN.PNG)


Tầng | Thiết Bị | Đặc điểm
---------|----------|---------
 Vật lý |Repeater | Làm việc với các tín hiệu trên cáp. Các tín hiệu trên cáp có thể được làm sạch, khuếch đại và chuyển vào một cáp khác. Repeater không nhận biết frame, packet or header. Nó chỉ làm việc với tín hiệu
 vật lý | Hub | Tương tự như repeater nhưng có nhiều cổng, tín hiệu đến một cổng sẽ được gửi đến tất cả các cổng còn lại và hub không có tính năng khuếch đại.
 Liên kết dữ liệu | Bridge | Dùng để kết nối hai hoặc nhiều mạng LANs. Khác với hub khi một frame đến bridge tách trường địa chỉ trong gói tin và tìm trong bảng để biết nơi sẽ gửi gói tin đến. Bridge sẽ chỉ gửi gói tin đến cổng mà dữ liệu cần tới và có thể chuyển tiếp nhiều frame trong cùng một thời điểm. Bridge thường có hiệu năng cao hơn hub. Khác với hub là các luồng dữ liệu vào có cùng tốc độ, bridge có thể xử lý dữ liệu với các tốc độ khác nhau do vậy cần có buffer để lưu trữ. Bridge có thể sử dụng trên nhiều loại LAN khác nhau nhưng trên thực tế do vấn đề về bảo mật hay chất lượng của dịch vụ nên bridge thường chỉ dùng cho cùng một loại mạng và router thường được dùng để kết nối các mạng khác nhau. Cụ thể hơn tại “Conputer Networks của Tanenbaum Chapter 4 trang 341”
 Liên kết dữ liệu | Switch | Tương tự như bridge. Switch có nhiều cổng hơn bridge và thường được sử dụng trong thực tế hơn. Lọc frame dựa vào địa chỉ vật lý và tự động xây dựng bảng lọc khi có frame đi qua.
 Tầng mạng | Router | Là một thiết bị mạng dùng để chuyển các gói dữ liệu qua một liên mạng và đến các thiết bị đầu cuối thông qua tiến trình gọi là định tuyến. Khi một packet tới router trường header và trailer sẽ được tách ra và dùng để định tuyến để chọn ra đường ra cho gói tin.

### Tầng mạng

Chọn đường (định tuyến) và chuyển tiếp gói tin từ nguồn đến đích

- Định tuyến: Tìm đường đi qua các nút trung gian để gửi dữ liệu từ nguồn đến đích. Giao thức định tuyến xác định đường đi ngắn nhất giữa hai điểm truyền tin.
- Chuyển tiếp: Chuyển gói tin trên cổng vào tới cổng ra theo tuyến đường. Bảng chuyển tiếp xác định cổng ra để chuyển tiếp dữ liệu.

Cấu trúc địa chỉ IP:

- HostID – Phần địa chỉ máy trạm
- NetworkID – Phần địa chỉ mạng

![Cấu trúc địa chỉ  IP](https://c1.staticflickr.com/3/2911/33217992560_20a96c7cb3_b.jpg)

Các dạng địa chỉ IP là :

- Network Address: Định danh cho một mạng . Tất cả bit phân HostID là 0
- Broadcast Address: Địa chỉ để gửi dữ liệu cho tất cả các máy trạm trong mạng. Tất cả các bit phần HostID là 1.
- Unicast Address: Gán cho một cổng mạng.
- Multicast address: Định danh cho một nhóm

Gateway: Cho phép ghép hai mạng sử dụng hai loại giao thức khác nhau. Ví dụ một mạng sử dụng giao thức IP muốn ghéo nối tới một mạng khác sử dụng giao thức IPX… thì gateway sẻ chuyển đổi từ giao thức này sang giao thức khác.

Giao thức ARP: Dùng để xác định địa chỉ MAC của một máy tính trong mạng khi biết địa chỉ IP của nó. Nguyên tắc làm việc : Khi một thiết bị mạng muốn biết địa chỉ MAC của một thiết bị nào đó mà nó đã biết địa chỉ IP nó sẽ gửi một ARP request bao gồm địa chỉ MAC của nó và địa chỉ IP của máy nó cần xác định địa chỉ MAC trên toàn bộ một miền broadcast. Mỗi thiết bị trên mạng broadcast sẽ so khớp địa chỉ IP của gói tin đó với địa chỉ IP của mình nếu trùng khớp nó sẽ gửi lại cho máy gửi ARP request một gói tin trong đó chứa địa chỉ MAC của mình. Sau đó hai thiết bị bắt đầu trao đổi thông tin với nhau.

Các bước để hai máy cùng một mạng LAN gửi gói tin cho nhau:
    Giả sử máy A muốn gửi một gói tin đến máy B trong cùng một mạng LAN. 
Máy A chỉ biết địa chỉ IP của máy B. Khi đó máy A sẽ gửi một ARP broadcast cho toàn bộ các máy trong mạng LAN để hỏi xem địa chỉ IP (IP của máy B ) ứng với MAC nào. Khi máy B nhận được gói tin này nó sẽ so sánh với IP của mình và nhận thấy đó là gói tin của mình khi đó máy B sẽ gửi một gói tin cho máy A trong đó chứa địa chỉ MAC của B và hai máy bắt đầu trao đổi thông tin với nhau.

Các bước để hai máy trong hai mạng LAN gửi gói tin  cho nhau :


Giả sử Host 1 muốn gửi một packet tới host 4(192.32.63.8) trên mạng EE. Host 1 quảng bá packet trong mạng CS và không thấy địa chỉ IP khớp với địa chỉ IP của host 4. Và nó biết phải gửi gói tin tới router( dafault gateway). Để gửi packet tới router host 1 phải biết địa chỉ MAC của router trên mạng CS. Nó tìm địa chỉ này bằng cách gửi gói tin quảng bá trong mạng CS. Sau đó nó gửi frame tới router. Sau khi router nhận được gói tin dựa vào trường địa chỉ trong gói tin nó biết gói tin sẽ phải gửi tới mạng EE nơi chứa host 4. Nếu router không biết địa chỉ MAC của host 4 nó sẽ sử dụng lại giao thức ARP để xác định đỉa chỉ đó trong mạng EE. Sau đó router gửi gói tin tới host 4.Tìm hiểu thêm tại “Computer network của tanenbaun trang 469”

Các bước để một gói tin lớn (1GB) từ một máy có IP là A gửi tới một máy có IP là B:

Khi tầng mạng nhận được một gói tin lớn (1GB) từ tầng giao vận chuyển xuống. Tầng mạng dựa vào giá trị MTU (kích thươc đơn vị dữ liệu tối đa) của đường truyền để chia gói tin lớn thành các gói tin nhỏ. Gói tin lớn được phân mảnh và chuyển đi sau đó được ghép tại máy đích.

### Tầng giao vận

Cung cấp phương tiện truyền giữa các thiết bị đầu cuối.
Bên gửi:

- Nhận dữ liệu từ tầng ứng dụng
- Đặt dữ liệu vào các gói tin và chuyển cho tầng mạng
- Nếu dữ liệu lớn quá nó sẽ được chia làm các phần và đặt vào các đoạn tin khách nhau.

Bên nhận:

- Nhận dữ liệu tầng mạng.
- Tập hợp dữ liệu và chuyển lên cho tầng trên

#### TCP vs UDP

| TCP | UDP
---------|----------|---------
  | Tin cậy , hướng liên kết | Không tin cậy, không liên kết
 Đơn vị truyền | Segment | datagram
 Trường hợp sử dụng | Các ứng dụng cần dịch vụ với 100% độ tin cậy như mail,web | Các ứng dụng cần chuyển nhanh dữ liệu có khả năng chịu lỗi như Video Streaming,...

 NAT là thiết bị cho phép một (hay nhiều) địa chỉ IP nội miền được ánh xạ với một (hay nhiều) địa chỉ IP ngoại miền. Có ba loại NAT: NAT tĩnh, NAT động và NAT vượt tải(overload).

- NAT tĩnh: Một địa chỉ IP nội miền được ánh xạ với một địa chỉ IP ngoại miền.
- NAT động:  địa chỉ IP nội bộ sẽ được tự động khớp với một bộ địa chỉ IP ngoài. Quá trình vẫn là ánh xạ một-một nhưng được diễn ra tự động.
- NAT overload: Với NAT overload ánh xạ một-một như NAT động và NAT tĩnh không được sử dụng thay vì một địa chỉ ngoài chỉ được ánh xạ với một địa chỉ trong mạng nội bộ thì bây giờ nó có thể gán cho tất cả các máy trong mạng nội bộ dựa trên số cổng. Chỉ khi số lượng các cổng khả dụng sử dụng với địa chỉ IP ngoài cạn kiệt thì một địa chỉ IP ngoài thứ hai mới được sử dụng với phương pháp tương tự.

#### Các bước một gói tin TCP từ một máy tính cục bộ đi tới trang facebook.com.vn (IP là 191.58.58.59, port 433) thông qua giao thức overload NAT

![Chuyển gói tin giữu hai mạng LAN](https://c1.staticflickr.com/4/3929/33472225731_3364d53fa3_b.jpg)
Ban đầu gói tin TCP được gửi từ một máy cục bộ có địa chỉ IP private (giả sử 10.0.0.1) gửi tới NAT. NAT sẽ ánh xạ địa chỉ IP private này vào một địa chỉ IP public ứng với một cổng (giả sử 100.123.4.5 port 123) Sau đó NAT thay thế địa chỉ trường người gửi trong gói tin TCP bằng địa chỉ IP public+ cổng sau đó sử dụng trường địa chỉ đích để gửi gói tin(cụ thể gửi tới facebook.com.vn 191.58.58.59 port 433). Sau đó NAT sử dụng giao thức ARP để gửi gói tin đến địa chỉ Facebook. Tại router nơi chứa mạng con chứa facebook. Router sẽ sử dụng bảng NAT để tìm địa chỉ IP private tương ứng với địa chỉ 191.58.58.59 port 443 nếu có sẽ chuyển tiếp đến máy đó nếu không có gói tin sẽ bị hủy.

Tài liệu tham khảo :

1. Slide Mạng máy tính thầy Bùi Trọng Tùng
2. Slide Mạng máy tính cô Trương diệu Linh
3. Computer Network tanenabun
