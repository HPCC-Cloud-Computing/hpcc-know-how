[TOC]

# Networking

## Tổng quan

- Tập hợp các máy tính kết nối với nhau dựa trên một kiến trúc nào đó để trao đổi dữ liệu
  - Máy tính: máy trạm, máy chủ, bộ định tuyến
  - Kết nối bằng một phương tiện truyền
  - Theo một kiến trúc mạng
- Kiến trúc mạng
  - Hình trạng mạng- Topology
    - Topology vât lý: hình trạng dựa trên cáp kết nối
      - Point-to-point: dạng topology đơn giản nhất, kết nối trực tiếp giữa hai nút mạng
      - Bus: mỗi máy tính kết nối với một dây cáp duy nhất, tín hiệu sẽ được truyền tới tất cả các kết nối trên mạng tới khi tìm được địa chỉ MAC hoặc IP phù hợp. Bus dễ triển khai, chi phí thấp do chỉ có 1 dây, nhưng khó quản lý, khi cáp mạng bị ngắt toàn bộ các nút sẽ không truy cập được do chỉ có một dây cáp
      - Star: mỗi máy chủ kết nối với hub trung tâm theo kết nối point-to-point, hub đóng vai trò như bộ tăng cường tín hiệu hoặc lặp lại. Hiệu suất cao với ít nút mạng và lưu lượng mạng thấp; dễ thêm nút mạng; dễ quản lý; chi phí cao, khi hub trung tâm lỗi toàn mạng sẽ không thể truy cập
      - Ring: hai nút kế tiếp kết nối với nhau, nút cuối kết nối với nút đầu tiêng tạo dạng vòng, tín hiệu di chuyển theo một chiều duy nhất; dễ cài đặt, khó bảo trì, một nút lỗi sẽ ảnh hưởng đến toàn bộ mạng
      - Mesh: topo mạng mà mỗi nút đều kết nối điểm điểm với các nút còn lại; kết nối mạnh, dễ sửa lỗi, tính bảo mật và riêng tư cao; không linh hoạt, cài đặt và câu hình khó, chí phí lớn
      - Tree: topo mạng hình thành hệ thống cấp bậc với một nút gốc, thường có 3 level nút; sử dụng trong mạng WAN, mở rộng của bus và star; dễ dàng mở rộng, quản lý, bảo trì và sửa lỗi; chi phí cao, nút gốc lỗi mạng sẽ không hoạt động được
      - Hyprid: kết hợp các dạng topo sao cho phù hợp; tính hiệu quả cao, dễ mở rộng và sửa lỗi; thiết kế phức tạp và chi phí cao
- Chuyển tiếp dữ liệu qua các kết nối
  - Chuyển mạch kênh: cấp phát tài nguyên đường truyền dành riêng cho từng kết nối logic giữa 2 nút mạng
  - Chuyển mạch gói: dữ liệu được chia thành các gói tin gồm header và payload, thiết bị chuyển mạch chuyển tiếp gói tin dựa trên tiêu đề
    - Unicast: chuyển tiếp gói tin tới một nút mạng
    - Multicast: chuyển tiếp gói tin tới một nhóm các nút mạng
    - Broadcast: chuyển tiếp gói tin tới tất cả các nút trong mạng, địa chỉ ff:ff:ff:ff:ff:ff được sử dụng, giao thức chung được sử dụng là ARP và DHCP
- Mô hình OSI và TCP/IP
  - Mô hình OSI
    - Kiến trúc
      - Physical Layer (level 1): chuyển dữ liệu bit thành tín hiệu và truyền
      - Data Link Layer (level 2): truyền dữ liệu trên các liên kết vật lý giữa các nút mạng liên kết trực tiếp với nhau
      - Network Layer (level 3): vận chuyển gói tin từ một mạng đến một mạng khác (giao thức IP)
      - Transport Layer (level 4): truyền dữ liệu theo trình tự từ nguồn tới đích (TCP, UDP)
      - Session Layer (level 5): khởi tạo, quản lý và kết thúc các kết nối 
      - Presentation Layer (level 6): khởi tạo nội dung giữa các thực thể tầng ứng dụng (mã hóa, nén, chuyển đổi)
      - Application Layer (level 7): cung cấp ứng dụng cho người dùng cuối
    - Ý nghĩa: là mô hình tham chiếu chức năng, có ý nghĩa lớn về mặt cơ sở lý thuyết, các mô hình trên thực tế tham chiếu từ mô hình OSI
  - Mô hình TCP/IP 
    - ![Chồng giao thức TCP/IP](https://github.com/vuducmanh11/HPCC_Lab/blob/master/report/week3/images/tcpip.png)
    - sử dụng duy nhất IP là giao thức liên mạng, tách rời phát triển ứng dụng ở tầng trên với công nghệ truyền dẫn tầng thấp
    - khó nâng cấp giao thức IP (IPv4 sang IPv6)
    - định danh
      - tên miền DNS: định danh sử dụng trên tầng ứng dụng
      - số hiệu cổng: định danh sử dụng trên tầng giao vận độ dài 16 bit
      - địa chỉ IP: định danh sử dụng trên tầng mạng
      - địa chỉ MAC
- Thiết bị mạng
  - Gateway
    - thiết bị đặt tại một nút mạng để giao tiếp với nút mạng trong mạng khác sử dụng phương thức khác nhau
  - Repeater
    - khuếch đại hoặc tái tạo tín hiệu nhận được để truyền tín hiệu xa hơn
  - Hub
    - được coi là repeater có nhiều cổng, thiết bị trong mạng làm việc tại tầng vật lý, kết nối các thiết bị mạng vật lý với nhau, có thể phát hiện các lỗi mạng cơ bản, dễ gây tắc nghẽn
  - Switch
    - là thiết bị với nhiều cổng chuyển tiếp các gói tin Ethernet từ một mạng đến một mạng khác
    - khi khung tin đầu tiên qua switch, switch chưa biết địa chỉ MAC nào được kết hợp với cổng nào để chuyển tiếp gói tin; nếu gói tin đặt cho địa chỉ MAC không xác định, switch sẽ chuyển gói tin đến tất cả các cổng 
    - qua quá trình chuyển tiếp gói tin trên mạng, nó sẽ biết địa chỉ MAC nào kết hợp với cổng nào để thực hiện chuyển tiếp gói tin.
  - Router
    - thiết bị được thiết kế để nhận, xử lý và chuyển tiếp gói tin tới mạng khác; khi nhận dữ liệu nó xác định địa chỉ đích bằng cách đọc header; sau đó tìm kiếm trên routing table để chuyển tiếp gói tin. Router không chuyển tiếp gói tin broadcast 
  - Bridge
    - chia mạng thành nhiều segment, mỗi đoạn sẽ có vùng đụng độ riêng làm giảm đụng độ trên mạng; bridge hoạt động ở tầng 2 mô hình OSI Data link
    - bridge kiểm tra lưu lượng mạng và sẽ quyết định chuyển tiếp gói tin hay loại bỏ nó; khi nhận gói tin nó kiểm tra địa chỉ MAC nếu địa chỉ nằm trên 1 đoạn khác của mạng, nó sẽ chuyển tiếp gói tin tới đoạn đó còn không sẽ bị loại bỏ
  - Sự khác nhau giữa Hub, Switch và Router
    - Hub chỉ quan tâm đến việc chuyển tiếp gói tin nó được nhận mà không rõ nó đang chuyển tiếp cái gì
    - Switch xử lý gói tin nhận được sau đó xác định đích đến qua địa chỉ MAC 
    - Router có chức năng chính là định tuyến gói dữ liệu tới mạng khác  thông qua địa chỉ IP không chỉ là trên các máy cục bộ, hỗ trợ các tính năng như firewall, QoS, VPN tăng tính bảo mật và riêng tư cá nhân

## Nội dung chính

------

### 1.Tầng ứng dụng

Cung cấp dịch vụ trên mạng cho các hệ thống đầu cuối

#### 1.1. Tên miền và dịch vụ DNS

- Giới thiệu
  - Tên miền là định danh trên tầng ứng dụng cho các nút mạng, được quản lý tập trung trên mạng Internet
  - DNS(Domain Name System) chứa không gian thông tin về tên miền, gồm các máy chủ quản lý tên miền và cung cấp dịch vụ phân giải tên miền
  - Người dùng sử dụng tên miền để truy cập mạng còn các thiết bị mạng sử dụng địa chỉ IP khi trao đổi dữ liệu
- Quy tắc đặt tên miền
  - Độ dài tối đa 255 ký tự, độ dài tối đa của label là 63 ký tự
  - Label phải bắt đầu bằng số hoặc chữ; chỉ chứa số, chữ, "-", "."
- Hệ thống máy chủ DNS
  - Theo cấu trúc hình cây
  - Máy chủ tên miền cấp 1: quản lý tên miền cấp 1(Top level domain-TLD)
  - Máy chủ được ủy quyền (authoritative DNS severs): quản lý tên miền cấp dưới
  - Máy chủ của các tổ chức, ISP: không nằm trong phân cấp DNS
  - Máy chủ cục bộ: dành cho mạng nội bộ của cơ quan tổ chức, không nằm trong phân cấp DNS
- Phân giải tên miền
  - Tự phân giải
    - file HOST: /etc/hosts (ubuntu) với window C:\WINDOWS\system32\drivers\etc\ 
    - Cache
  - Dịch vụ phân giải tên miền DNS: client/server
    - UDP, Port 53
  - Phân giải đệ quy: người dùng bắt đầu hỏi địa chỉ của tên miền cần đến Local server, không thấy máy chủ cục bộ sẽ hỏi đến root server, root server hỏi TLD server, TLD server hỏi Authoritative DNS server
  - Phân giải tương tác: người dùng bắt đầu hỏi địa chỉ của tên miền cần đến Local server, không thấy máy chủ cục bộ sẽ lần lượt hỏi root server, TLD server, Authoritative DNS server

#### 1.2. HTTP và WWW

**WWW**: world wide web trao đổi dữ liệu siêu văn bản HTML

**HTTP**

- HyperText Transfer Protocol, mô hình Client/Server trao đổi với nhau thông qua một kết nối TCP
- Trao đổi thông điệp HTTTP
  - HTTP Request: với các phương thức GET, POST, HEAD; PUT; DELETE
  - HTTP Response: các mã trả lời 200, 404, 500,..
- HTTP Cookie
  - Cookie: dữ liệu do Web server tạo chưa thông tin trạng thái của phiên
  - Sau khi xử lý yêu cầu từ phía Client, Server gửi thông điệp HTTP Response với cookie đính kèm chưa thông tin phiên làm việc của Client
  - Trình duyệt lưu Cookie, sử dụng để đính kèm trong các HTTP Request tiếp theo
- Caching
  - Sử dụng bộ nhớ đệm của máy chủ proxy, nếu đối tượng web còn giữ trên máy chủ proxy nó sẽ trả lại cho người dùng giảm thời gian yêu cầu từ máy chủ
  - Sử dụng local cache lưu trên máy cục bộ

#### 1.3. Proxy

##### 1.3.1. Tổng quát

- Máy chủ proxy là một máy tính chuyên dụng hoặc phần mềm hệ thống chạy trên một máy tính hoạt động trung gian giữa thiết bị điểm cuối như một máy tính và máy chủ khác mà từ đó người dùng hay client gửi request dịch vụ. Máy chủ proxy có thể cùng tồn tại trong máy tính như một tường lửa hoặc nó nằm trên một máy chủ riêng chuyển tiếp các yêu cầu thông qua tường lửa.
- Một lợi ích của server proxy là bộ nhớ cache có thể phục vụ tất cả người dùng. Nếu một hoặc nhiều trang web thường được truy cập, nó giống như nằm trong bộ nhớ cache proxy, việc này sẽ cải thiện thời gian người dùng phải chờ. Một proxy cũng có thể ghi lại tương tác của nó, điều này giúp hữu ích trong việc khắc phục sự cố

##### 1.3.2. Forward và reverse proxy servers

- **Forward proxies**
  - gửi các request của người dùng tới web server. Những người dùng truy cập proxy này bằng cách truy cập trực tiếp địa chỉ web proxy. Forward proxy cho phép vượt tường lửa tăng sự riêng tư và bảo mật cho người dùng nhưng đôi khi nó giúp tải xuống các tài liệu bất hợp pháp như tài liệu bản quyền hay nội dung 18+.
- **Reverse proxies** 
  - xử lý minh bạch tất cả các yêu cầu về tài nguyên trên server đích mà không yêu cầu bất kỳ hành động nào của người yêu cầu
  - thường được sử dụng
    - cho phép truy cập gián tiếp khi website từ chối kết nối trực tiếp như 1 sự đảm bảo an toàn
    - cho phép cân bằng tải giữa các server
    - truyền nội dung nội bộ tới người dùng Internet
    - vô hiệu hóa truy cập trang web, như khi ISP hay chính phủ yêu cầu chặn website
  - trang web có thể bị chặn vì nhiều lý do, Reverse proxy có thể được sử dụng để ngăn cản truy cập vào nội dung vô đạo, bất hợp pháp hoặc nội dung có bản quyền. Đôi khi việc này ngăn cản người dùng truy cập trang web tin tức nơi thông tin bị rò rỉ, trang tiết lộ thông tin về hành động của chính phủ gây mất tự do ngôn luận

#### 1.4. Firewall

##### 1.4.1. Khái niệm

- **Firewall** là phần mềm hoặc chương trình cơ sở thực thi bộ quy tắc về các gói dữ liệu sẽ được cho phép ra vào mang.
- Tường lửa (firewall) được tích hợp vào một loạt các thiết bị được kết nối mạng, kiểm soát truy cập và giảm thiểu rủi ro các gói tin độc hại được thông qua mạng công cộng ảnh hưởng tới an toàn của mạng người dùng.
- Tường lửa có thể mua dạng ứng dụng độc lập
- Tường lửa là một hàng rào ảo được đặt bao quanh mạng để hạn chế thiệt hại từ một cuộc tấn công mạng bên nguoaif hoặc nội bộ

##### 1.4.2. Phân loại

- Tường lửa được chia làm hai loại chính 
  - host-based
    - cài đặt riêng trên server, giám sát mọi tín hiệu đến và đi
  - network-based
    - có thể được xây dựng tích hợp cơ sở hạ tầng cloud hoặc có thể là một dịch vụ tường lửa ảo

##### 1.4.3. Iptables

- **Iptables** được cấu hình và hoạt động trên console, tăng tính bảo mật cho hệ thống linux	
  - lọc package dựa vào MAC và một số cờ hiệu TCP Header
  - cung cấp tùy chọn ghi nhận sự kiện hệ thống
  - cung cấp kỹ thuật NAT
  - có khả năng chặn một số cơ chế tấn công DoS
- show list rule: `sudo iptables -L`
  - TARGET: hành động được áp dụng cho quy tắc
    - Accept: gói dữ liệu được phép chuyển tiếp để xử lý 
    - Drop: chặn gói tin và loại bỏ
    - Reject: chặn gói tin, loại bỏ và gửi thông báo lỗi đến người dùng
  - PROT (protocol): quy định giao thức được áp dụng để thực hiện quy tắc gồm 
    - all
    - TCP
    - UDP
  - SOURCE, DESTINATION: địa chỉ của lượt truy cập được phép áp dụng quy tắc

------

### 2. Tầng giao vận

- Cung cấp các phương tiện truyền giữa các ứng dụng cuối, chỉ được cài đặt trên các hệ thống cuối
- Truyền nhận dữ liệu
  - Bên gửi nhận dữ liệu từ ứng dụng đóng gói vào các gói tin chuyển cho tầng mạng, dữ liệu sẽ chia thành nhiều phần nếu quá lớn để gói vào các đoạn tin
  - Bên nhận nhận các đoạn tin từ tầng mạng, tách dữ liệu và tập hợp chuyển lên cho tầng ứng dụng
  - Hai dạng dịch vụ giao vận
    - Tin cậy, hướng liên kết TCP, đơn vị truyền segment
    - Không tin cậy, không liên kết UDP, đơn vị truyền datagram

#### 2.1. UDP

**User Datagram Protocol** giao thức hướng không kết nối

- Truyền tin "best-effort" chỉ gửi một lần, không phát lại, mỗi tiến trình sử dụng một socket duy nhất
- Đặc điểm
  - Không thiết lập liên kết giảm độ trễ
  - Đơn giản không cần lưu trạng thái liên kết ở bên gửi và bên nhận
  - Không quản lý tắc nghẽn: UDP cứ gửi dữ liệu nhanh nhất, nhiều nhất best-effort
- Khuôn dạng bức tin
  - Đơn vị dữ liệu: datagram
  - Header dài 8 bytes gồm: source port, dest port, length (độ dài bức tin theo byte), checksum (kiểm tra lỗi để hủy gói tin)
- Sử dụng: ứng dụng cần chuyển dữ liệu nhanh có khả năng chịu lỗi cao; được dùng trong các giao thức DNS, DHCP, SNMP

#### 2.2. TCP

**Transmission Control Protocol** giao thức hướng liên kết, thiết lập liên kết qua bắt tay ba bước

- Đặc điểm
  - Giao thức truyền dữ liệu theo dòng byte, tin cậy
  - Kiểm soát lỗi, mất gói tin để phát lại 
  - truyền theo kiểu pipeline (gửi liên tục một lượng hữu hạn các gói tin) tăng hiệu suất truyền
  - Kiểm soát luồng và tắc nghẽn: không làm quá tải bên nhận cũng như làm tắc nghẽn mạng
- Khuôn dạng bức tin
  - Đơn vị dữ liệu: segment
  - Header dài 24 bytes gồm: source port, dest port, sequence number, acknowledgement number,data offset, reserved, control bit, window, checksum, urgent pointer, options, padding
- Sử dụng: phù hợp ứng dụng cần độ tin cậy cao, không quan trọng thời gian truyền nhận tin; sử dụng trong các giao thức HTTP, SMTP, FTP

------

### 3. Tầng Mạng

- Điều khiển truyền dữ liệu giữa các nút mạng qua môi trường liên mạng
- Truyền dữ liệu từ host-host, được cài đặt trên mọi hệ thống cuối và bộ định tuyến
- Chức năng: định tuyến, chuyển tiếp, định địa chỉ, đóng gói dữ liệu, đảm bảo chất lượng dịch vụ (QoS)
- Các giao thức
  - Giao thức định tuyến
  - Giao thức ICMP
  - Giao thức IP

#### 3.1. Giao thức IP

- Chức năng
  - Định danh: địa chỉ IP
  - Đóng gói
  - Chuyển tiếp: theo địa chỉ IP 
  - QoS
- Đặc điểm
  - Là giao thức cơ sở của tầng mạng, hướng không liên kết, không tin cậy/nhanh, không có cơ chế phục hồi nếu có lỗi

#### 3.2. Địa chỉ IPv4

- Dài 32 bit định danh cổng giao tiếp mạng trên nút đầu cuối (PC, server, smart phone), router
- Cấp phát cố định hoặc cấp phát động DHCP
- Địa chỉ IP chia làm hai phần, phần đầu m bit NetworkID, phần sau (32 -m) bit HostID, m là giá trị mặt nạ mạng
- Phân loại địa chỉ IP
  - Địa chỉ mạng: tất cả các bit HostID là 0
  - Địa chỉ quảng bá: tất cả các bit HostID là 1
  - Địa chỉ máy trạm: gán cho một cổng mạng
  - Địa chỉ nhóm: định danh cho nhóm
- Các địa chỉ IP đặc biệt
  - Private address dùng làm địa chỉ riêng trên các riêng không kết nối với Internet
    - 10.0.0.0/8
    - 172.16.0.0/16 --> 172.31.0.0/16
    - 192.168.0.0/24 --> 192.168.255.0/24
  - Loopback address: địa chỉ localhost
    - 127.0.0.0/8
  - Multicast address
    - 224.0.0.0 --> 239.255.255.255
    - được chia thành nhiều lớp, đặc biệt hai lớp:
      - 224.0.0.0 --> 224.0.0.255: địa chỉ local subnetwork, router không chuyển tiếp các gói tin ra ngoài
      - 224.0.1.0 --> 224.0.1.255: địa chỉ Internet control các gói tin qua địa chỉ này phải được định tuyến qua public Internet

#### 3.3. Dynamic Host Configuration Protocol (DHCP)

##### 3.3.1. Khái niệm và thành phần

**Dynamic host configuration protocol**: tự động cấp phát địa chỉ IP động cho khách khi tham gia mạng

- DHCP server: quản lý việc cấu hình và cấp phát địa chỉ IP cho Client
- DHCP client: máy nhận thông tin cấu hình địa chỉ IP từ DHCP server
- Scope: dải địa chỉ IP được dùng để cấp phát cho Client
- Exclusing Scope: địa chỉ trong Scope không được cấp phát động cho Client

##### 3.3.2. Các thông điệp

- DHCP Discover: Một DHCP Client khi mới tham gia vào hệ thống mạng, nó sẽ yêu cầu thông tin địa chỉ IP từ DHCP Server bằng cách broadcast một gói DHCP Discover. Địa chỉ IP nguồn trong gói là 0.0.0.0 bởi vì client chưa có địa chỉ IP 
- DHCP Offer: Khi DHCP server nhận được gói DHCP Discover từ client, nó sẽ gửi lại một gói DHCP Offer chứa địa chỉ IP, Subnet Mask, Gateway,... Có thể nhiều DHCP server gửi lại với gói DHCP Offer nhưng Client sẽ chấp nhận gói DHCP Offer đầu tiên nó nhận được 
- DHCP Request: Khi DHCP client nhận được một gói DHCP Offer, nó đáp lại bằng việc broadcast gói DHCP Request để xác nhận hoặc để kiểm tra lại các thông tin mà DHCP server vừa gửi 
- DHCP Acknowledge: Server kiểm tra và xác nhận lại sự chấp nhận trên bằng gói tin DHCP Acknowledge, Client có thể tham gia trên mạng TCP/IP và hoàn thành hệ thống khởi động 
- DHCP Nak: Nếu một địa chỉ IP đã hết hạn hoặc đã được đặt một máy tính khác, DHCP Server gửi gói DHCP Nak cho Client, như vậy Client phải bắt đầu tiến trình thuê bao lại 
- DHCP Decline: Nếu DHCP Client nhận được bản tin trả về không đủ thông tin hoặc hết hạn, nó sẽ gửi gói DHCP Decline đến các Server và Client phải bắt đầu tiến trình thuê bao lại 
- DHCP Realease: Client gửi bản tin này đến server để ngừng thuê IP. Khi nhận được bản tin này, server sẽ thu hồi lại IP đã cấp cho Client, khi đó client sẽ không được cấu hình IP

##### 3.3.3. Cơ chế hoạt động

- Client gửi một bản tin Discover - có chứa thông tin client (MAC, Tên máy tính,...) với dạng Broadcast để "truy tìm" một DHCP Server để xin cấp phát địa chỉ IP 
- Khi DHCP Server nhận được một bản tin Discover sẽ gửi lại cho Client một bản tin Offer - chứa những thông tin IP,thời hạn cho thuê, GW, DNS Server,... dưới dạng Unicast 
- Client gửi lại một bản tin Request cho Server, để xác nhận lại các thông tin 
- Server trả về bản tin ACK để hoàn tất quá trình 

#### 3.4. Quality of service (QoS)

##### 3.4.1. Khái quát

- Quality of service đề cập tới mọi công nghệ quản lý lưu lượng dữ liệu để giảm mất gói tin, độ trễ hay jitter (độ lệch về thông tin, thời gian) trên mạng. QoS điều khiển và quản lý tài nguyên mạng bằng cách set ưu tiên cho loại dữ liệu cụ thể trên mạng.
- Mạng doanh nghiệp cần cung cấp dịch vụ có thể dự đoán và đo lường như ứng dụng - như dữ liệu thoại, video, dữ liệu nhạy cảm về độ trễ đi qua mạng. Tổ chức sử dụng QoS để đáp ứng yêu cầu của ứng dụng nhạy cảm, như chat thoại và video thời gian thực, để ngăn chặn sự xuống cấp chất lượng gây ra do mất gói tin, delay và jitter.
- Tổ chức có thể hoàn thành QoS bằng cách sử công cụ và kỹ thuật nhất định như jitter buffer và traffic shaping. VỚi nhiều tổ chức, QoS bao gồm thỏa thuận mức dịch vụ với nhà cung cấp mạng để đảm bảo một hiệu suất nhất định

##### 3.4.2. QoS parameter

- Tổ chức có thể đo lường định lượng QoS qua một số tham số
  - Packet loss
  - Jitter
  - Latency
  - Bandwidth
  - Mean opinion score

##### 3.4.3. Thực hiện QoS

- Ba mô hình hiện có để thực hiện QoS: Best Effort, Integrated Services và Differentiated Services
  - Best Effort: là một mô hình QoS nơi tất cả các gói nhận được có cùng độ ưu tiên và không có sự đảm bảo việc truyền gói. Best Effort được áp dụng khi không có chính sách QoS nào hoặc khi kiến trúc hạ tầng không hỗ trợ QoS.
  - Integrated Services (IntServ): là một mô hình QoS dự trữ băng thông theo một đường dẫn cụ thể trên mạng. Ứng dụng yêu cầu mạng để đặt trước tài nguyên và thiết bị mạng giám sát luồng dữ liệu để đảm bảo tài nguyên mạng có thể chấp nhận gói. Thực hiện IntServ yêu cầu khả năng IntServ của router và sự dụng Resource Reservation Protocol (RSVP) cho việc yêu cầu tài nguyên mạng. IntSerc có khả năng mở rộng hạn chế và tiêu thụ tài nguyên mạng cao.
  - Differeniated Services (DiffServ) : là một mô hình QoS nơi các phần tử mạng cũng như router hay switch, được cấu hình với nhiều sự ưu tiên cho các truy cập khác nhau. Lưu lượng mạng phải được chia thành các lớp dựa trên yêu cầu của công ty.
- **Cơ chế QoS**
  - Một số cơ chế QoS nhất định có thể quản lý chất lượng luồng dữ liệu và duy trì các yêu cầu QoS được chỉ định trong SLAs (service-level agreement). Cơ chế QoS thuộc các danh mục cụ thể phụ thuộc vào vai trò của chúng trong việc quản lý mạng
  - Classification and marking: công cụ phân biệt giữa ứng dụng và loại gói dẫn tới loại lưu lượng khác nhau. Marking sẽ đánh dấu mỗi gói như một thành viên của lớp mạng. Phân lớp và đánh dấu được thực hiện trên các thiết bị mạng như router, switch và điểm truy cập.
  - Congestion management: công cụ sử dụng phân lớp gói và đánh dấu để xác định hàng đợi nơi chứa các gói, trong Congestion management tool bao gồm các loại hàng đợi: theo độ ưu tiên i, first-in, first-out và hàng đợi độ trễ thấp.
  - Congestion avoidance: công cụ giám sát lưu lượng mạng với việc tắc nghẽn và sẽ gỡ bỏ gói độ ưu tiên thấp khi tắc nghẽn xảy ra. Công cụ tránh tắc nghẽn bao gồm phát hiện sớm ngẫu nhiên có trọng số và phát hiện sớm ngẫu nhiên
  - Shaping: công cụ thao tác lưu lượng truy cập vào mạng và ưu tiên ứng dụng thời gian thực hơn ứng dụng ít nhạy cảm với thời gian như email, messaging. Công cụ bao gồm buffer, Generic Traffic Shaping, Frame Relay Traffic Shaping. Giống như việc định hình, công cụ chính sách lưu lượng tập trung vào điều chỉnh lưu lượng truy cập vượt quá và gỡ bỏ gói.
  - Link efficiency: công cụ tối đa hóa việc sử dụng băng thông và giảm độ trễ truy cập gói mạng. Link effciency thường được kết hợp với các cơ chế QoS khác. Link effciency bao gồm nén tiêu đề Real-Time Transport Protocol, nén tiêu đề Transmission Control Protocol và nén link.

#### 3.5. Network Address Translation (NAT)

##### 3.5.1. Tổng quát

- Network Address Translation (NAT) kỹ thuật cho phép thay đổi thông tin địa chỉ IP trong gói tin được truyền qua thiết bị định tuyến, từ địa chỉ IP cục bộ thành địa chỉ IP công cụ để giao tiếp với Internet. NAT giải quyết vấn đề cạn kiệt địa chỉ IPv4, cho phép nhiều máy chủ trong mạng cục bộ có thể kết nối với Internet thông qua một địa chỉ IP công cộng duy nhất

##### 3.5.2. Phân loại NAT

- Static NAT: 
  - chuyển dịch một địa chỉ IP cục bộ thành một địa chỉ IP công cộng
  - NAT router chưa một bảng liên kết mỗi địa chỉ IP cục bộ với một địa chỉ IP công cộng; tất cả các thiết bị sẽ cần được đăng ký với một địa chỉ IP công cộng để kết nối Internet
- Dynamic NAT
  - NAT router chứa một danh sách địa chỉ IP công cộng, khi máy cục bộ truy cập Internet router sẽ ánh xạ nó tới một trong những địa chỉ IP chưa được sử dụng
- Overloading NAT
  - NAT router chỉ chứa một địa chỉ IP công cộng
  - NAT router map mỗi máy cục bộ khi cần giao tiếp với internet với địa chỉ IP kết hợp với cổng; cặp giá trị này sẽ được lưu vào bảng, khi nhận gói tin từ Internet dựa vào bảng này có thể xác định được máy cục bộ nào là đích chuyển gói tin

#### 3.6. Internet Control Message Protocol (ICMP)

- Là giao thức được sử dụng ở tầng mạng để trao báo lỗi, gửi phản hồi cho sender về việc dữ liệu gửi đi

- Gói tin ICMP

  - Nằm trong gói tin IP
  - Chứa 3 trường cơ bản
    - Type: dạng gói tin ICMP
    - Code: nguyên nhân gây lỗi
    - Checksum

- Một số dạng gói tin ICMP

  - Thông báo lỗi

    | Type | Description             |
    | ---- | ----------------------- |
    | 3    | Destination Unreachable |
    | 4    | Source quench           |
    | 5    | Redirection             |
    | 11   | Tim exceeded            |
    | 12   | Parameter problem       |

  - Truy vấn

    | Type     | Description                          |
    | -------- | ------------------------------------ |
    | 8 or 0   | Echo reply or request                |
    | 13 or 14 | Time stamp request or reply          |
    | 17 or 18 | Address mask request or reply        |
    | 9 or 10  | Router advertisement or solicitation |

  - Sử dụng ICMP qua các công cụ debug như ping, traceroute

#### 3.8. Routing

##### 3.8.1. Các khái niệm cơ bản

- Định nghĩa
  - Định tuyến là tìm đường đi tốt nhất để chuyển tiếp dữ liệu tới nút đích
  - Trong mạng IP chuyển mạch gói, các gói tin được chuyển tiếp độc lập, mỗi nút không chỉ ra toàn bộ các chặng trên đường tới đích
- Các thành phần của định tuyến
  - Thông tin: Thông số của đường truyền được sử dụng là độ đo tính toán chi phí đường đi
  - Bảng định tuyến: Lưu thông tin đường đi đã tìm tới các mạng đích
  - Giải thuật, giao thức định tuyến: cách thức tìm đường đi và trao đổi thông tin định tuyến giữa các nút mạng
- Router
  - Thiết bị chuyển tiếp các gói tin giữa các mạng
    - là một máy tính với phần cứng chuyên dụng
    - kết nối nhiều mạng với nhau
    - chuyển tiếp gói tin dựa trên bảng định tuyến
  - Nhiều cổng kết nối mạng
  - Phù hợp với nhiều dạng lưu lượng và phạm vi mạng

##### 3.8.2. Bảng định tuyến

| Destination | Outgoing Port | Next hop | Cost |
| :---------: | :-----------: | :------: | :--: |
|             |               |          |      |

- Destination: địa chỉ mạng đích
  - Định tuyến classless: sử dụng địa chỉ không phân lớp
  - Định tuyến classfull: sử dụng địa chỉ phân lớp
- Outgoing Port: cổng ra cho gói tin để tới mạng đích
- Next hop: địa chỉ cổng nhận gói tin của nút kết tiếp
- Cost: chi phí gửi gói tin từ nút xét tới nút đích

------

### 4. Tầng liên kết dữ liệu

**Chức năng**

- Logic Link Control sublayer
  - Kiểm soát luồng
  - Dồn kênh, phân kênh các giao thức
- Media Access Control sublayer
  - Đóng gói dữ liệu
  - Định địa chỉ vật lý
  - Phát hiện và sửa lỗi
  - Điều khiển truy nhập đường truyền

#### 4.1. Địa chỉ MAC

- Địa chỉ MAC (Media access control) của máy tính là một định danh duy nhất được gán cho một card giao tiếp mạng NIC (Network Interface Card) cho truyền thông tại tầng liên kết dữ liệu. Một nút mạng có thể có nhiều NIC, mỗi NIC phải có một địa chỉ MAC duy nhất.
- Độ dài 48 bit chia thành 6 octet, 3 octet đầu là ID của nhà sản xuất OUI(Organizationally Unique Identifer), 3 octet tiếp theo là giá trị định danh của NIC
- Ở byte đầu tiên của địa chỉ MAC
  - bit thứ 7
    - 0: globally unique (địa chỉ gán bởi nhà sản xuất)
    - 1: locally administered (địa chỉ được gán bởi admin, được sử dụng khi tạo máy ảo hoặc card mạng ảo)
  - bit thứ 8
    - 0: unicast
    - 1: multicast
- DHCP dựa vào địa chỉ MAC để gán địa chỉ IP cho thiết bị mạng

#### 4.2.  Address Resolution Protocol (ARP)

- Sử dụng để tìm địa chỉ MAC khi đã biết địa chỉ IP, do tầng mạng dùng địa chỉ IP, tầng liên kết dữ liệu dùng địa chỉ MAC
- Hoạt động
  - Mỗi nút trong mạng LAN sử dụng bảng ARP Table gồm các ánh xạ
    - <Địa chỉ IP, Địa chỉ MAC, TTL>
    - TTL: thời gian giữ ánh xạ trong bảng
  - Khi cần biết địa chỉ MAC tương ứng với địa chỉ IP không có trong ARP Table, nút mạng quảng bá gói tin ARP Request lên trên mạng để hỏi
  - Nút mạng mang địa chỉ IP tương ứng sẽ gửi gói tin ARP Reply để trả lời

------

### 5. Flat network

#### 5.1. Khái niệm

- Flat là một thiết kế mạng tiếp cận với mục tiêu giảm chi phí dễ bảo trì và quản trị. Flat network được thiết kế để giảm số lượng router và switch trên mạng máy tính bằng cách kết nối các thiết bị tới cùng một switch thay vì các switch riêng biệt.

#### 5.2. Sử dụng

- Flat network thường được sử dụng trong hộ gia đình hoặc doanh nghiệp nhỏ nới yêu cầu về mạng thấp, không yêu cầu bảo mật chuyên sâu hoặc sự tách biệt bởi mạng được cung cấp cho nhiều thiết bị truy cập. Trong trường hợp này việc cấu hình mạng phức tạp với nhiều switch là không cần thiết.
- Flat network cũng dễ để quản lý bảo trì vì ít switch và router được sử dụng, việc này cũng làm giảm chi phí cấu hình mạng.

#### 5.3. Hạn chế

- Kém an toàn: vì lưu lượng chỉ di chuyển qua một switch, nó không thể phân đoạn mạng và ngăn cản người dùng truy cập một số phần mạng. Các hacker có thể dễ dàng chặn và ăn cắp dữ liệu trên mạng.
- Không dự phòng: vì chỉ có 1 switch hay là ít thiết bị, việc hỏng hóc sẽ gây ra không có đường dẫn thay thế, mạng sẽ không thể truy cập dẫn đến máy tính kết nối thất bại.
- Khả năng mở rộng và tốc độ: kết nối tất cả thiết bị tới một switch trung tâm trực tiếp hoặc qua các hub làm tăng khả năng đụng độ, giảm tốc độ do dữ liệu cần được xử lý và chuyển tiếp ở switch trung tâm. Khả năng mở rộng tồi và tăng khả năng lỗi mạng nếu sử dụng nhiều hub và không đủ switch để điều khiển luồng mạng

### 6. VLAN

#### 6.1. Khái quát

- Yêu cầu thực tế
  - Chia sẻ tài nguyên (file, máy in,..) giữa các máy trạm xa nhau
  - Bảo mật thông tin nội bộ trong một phòng ban
- Giải pháp VLAN: mạng LAN ảo
  - Nhóm các trạm thành một mạng LAN logic không bị ràng buộc về mặt địa lý của các trạm
  - VLAN độc lập với các ứng dụng mạng
  - Một VLAN là một broadcast domain được tạo ra trên một hoặc nhiều switch
  - Một switch có thể chứa một hoặc nhiều VLAN

#### 6.2. Phân loại VLAN

Các phương pháp chia VLAN

- Chia theo cổng trên switch - VLAN tĩnh (Static VLAN): tất cả các thiết bị gắn với cổng đó phải cùng VLAN
- Chia theo địa chỉ MAC - Dynamic VLAN: linh hoạt
- Chia theo giao thức tầng 3 (địa chỉ IP): phụ thuộc vào giao thức tầng trên

#### 6.3. Đặc điểm

- VLANs thường được kết hợp với địa chỉ IP mạng con, tạo VLAN cần set các tham số

  - VLAN number
  - VLAN name
  - VLAN type
  - VLAN state (active or suspended)
  - Maximum transmission unit (MTU) của VLAN
  - Security Association Identifer (SAID)

- Các liên kết trong VLAN

  - Trunk link
    - Khi nhu cầu lớn hơn, các nhân viên trong các phòng ban khác nhau muốn giao tiếp với nhau. Khi đó, giữa các switch phải hình thành một truk link. Trunk link là một đường kết nối mà mỗi đầu được cấu hình port kiểu trunk. Trunk link cho phép vận chuyển frame có vlan id khác nhau
  - Trunk port
    - Được dùng để cho phép kết nối với một switch và hình thành trunk link. Frame trước khi đi qua trunk port sẽ được gán (tag) một vlan id vào frame để switch ở đầu bên kia biết cần đẩy frame này đến các port thuộc vlan nào 

  ![trunk port image](https://github.com/vuducmanh11/HPCC_Lab/blob/master/report/week3/images/trunk-port.JPG)

  - Access port
    - Access port chỉ thuộc về một vlan duy nhât. Access port thường dùng để nối với các thiết bị đầu cuối của người dùng hoặc các switch không hỗ trợ vlan. Trước khi đẩy frame đến một access port, vlan id trên frame sẽ bị gỡ bỏ

- Quá trình chuyển frame

  - ![image](https://github.com/vuducmanh11/HPCC_Lab/blob/master/report/week3/images/forward%20frame%20vlan.JPG)
  - Mỗi switch có một vlan database: định nghĩa tất cả vlan mà switch có; một nhóm port sẽ nằm trong một vlan, mỗi port chỉ được định nghĩa nằm trong một vlan duy nhất ngoại trừ trunk port
  - Khi máy tính trong vlan2 gửi frame (broadcast hoặc unicast frame tùy vào thông tin trong bảng CAM của switch ), frame sẽ được send đến trunk port. Tại đây, frame được tag thêm một vlan id để nhận diện. Sau khi đi xuyên qua trunk link, frame được switch đầu bên kia gỡ tag vlan id trước khi đẩy vào các access port thuộc vlan 2

### 7. Overlay Network

#### 7.1. Tổng quát

- Một overlay network là một mạng ảo chứa các nút mạng liên kết logic với nhau được xây dựng trên nền tảng mạng vật lý có sẵn, nhằm thực hiện các dịch vụ mạng không có trên mạng đã tồn tại.
- Internet là một overlay network với mục đích kết nối các mạng cục bộ xây dựng dựa trên local area network, phone lines

#### 7.2. Đặc điểm

- Ưu điểm và nhược điểm
  - không phải triển khai thêm thiết bị mới trên mạng hay chỉnh sửa phần mềm giao thức đã có
  - không cần triển khai trên mọi nút mạng do không phải nút mạng nào cũng cần dịch vụ tại mọi thời điểm
  - Làm tăng chi phí truyền tải do phải bổ sung header, tăng thời gian xử lý
  - Làm phức tạp mạng
- Overlay networking được sử dụng để phát triển mộ số giao thức IP, VXLAN, VPNs, IP multicast.

### 8. Virtual Extensible LAN (VXLAN)

#### Khái niệm

- VXLAN cung cấp dịch vụ để kết nối Ethernet tới các thiết bị cuối như VLAN nhưng mở rộng hơn về quy mô và khả năng triển khai; tạo mạng vật lý layer 2 dựa trên lớp mạng IP

#### Packet VXLAN

- VXLAN sử dụng MAC Address-in-User Datagram Protocol (MAC-in-UDP) đóng gói để mở rộng layer 2 segment để đi qua data center
- VXLAN encapsulate MAC-in-UDP từ khung tin layer2 và thêm VXLAN header, sau đó đặt trong gói tin UDP-IP
- ![packet_vxlan_image](https://github.com/vuducmanh11/HPCC_Lab/blob/master/report/week3/images/VXLANpacketformat.png)
- VXLAN header: 8 byte bao gồm các trường
  - Flags: 8bit, trong đó bit thứ 5 (I flag) được thiết lập 1 để chỉ ra đó là frame có VNI có giá trị; 7bit còn lại dùng dự trữ được thiết lập 0
  - VNI: 24bit, cung cấp định danh duy nhất cho VXLAN segment
- Outer UDP Header: port nguồn của Outer UDP được gán tự động sinh ra bởi VTEP và port đích 
- Outer IP Header: cung cấp địa chỉ IP nguồn của VTEP và địa chỉ đích của VTEP nhận frame
- Outer Ethernet Header: cung cấp địa chỉ MAC nguồn của VTEP chứa khung frame ban đầu. Địa chỉ MAC đích là địa chỉ của hop tiếp theo được định tuyến với VTEP

#### VTEP-Virtual Tunnel Endpoints

- ![vtep_image](https://github.com/vuducmanh11/HPCC_Lab/blob/master/report/week3/images/white-paper-c11-729383_2.jpg)
- VXLAN sử dụng thiết bị VTEP để map thiết bị cuối với đoạn tin VXLAN để encapsulation và de-encapsulation các gói tin vận chuyển và ánh xạ các máy trong VXLAN. VTEP cung cấp hai interface một là switch interace trong mạng LAN để hỗ trợ điểm cuối giao tiếp thông qua cầu nối, giao diện thứ hai là IP interface với địa chỉ IP duy nhất xác đinh thiết bị VTEP. Địa chỉ IP của VTEP để đúng gói trong khung tin Ethernet và chuyển gói tin đã được đóng gói thông qua IP interface

#### VXLAN Packet Forwarding Flow

- VXLAN sử dụng stateless tunnel giữa các VTEP để chuyển tiếp gói tin trên mạng 

  ![forwarding](https://github.com/vuducmanh11/HPCC_Lab/blob/master/report/week3/images/vxlan_forward.jpg)

  - Host A tạo frame 1(L2 Frame) với các trường IP và MAC
  - frame 1 được chuyển đến VTEP-1, tại VTEP-1 dựa vào bảng ánh xạ cho biết host B tại VTEP-2. VTEP-1 đóng gói thêm các trường VXLAN, UDP, outer IP header vào L2 Frame. VTEP-1 sau đó tra cứu địa chỉ VTEP-2 (lấy ở outer IP header) để chọn đường tiếp theo cho gói tin, sau đó sử dụng địa chỉ MAC của nút kế tiếp đóng gói vào gói tin và gửi tới nút kế tiếp
  - Gói tin được định tuyến theo địa chỉ IP chuyển tới VTEP-2. VTEP-2 nhận được gói tin loại bỏ outer IP header, UDP, VXLAN header, đọc thông tin và chuyển gói tin tới máy B theo địa chỉ MAC đích trong L2 frame

### 9. Generic routing encapsulation (GRE)

- Khái niệm

  - GRE là giao thức để thiết lập kết nối trực tiếp point-to-point giữa các node trong mạng. Là giao thức đơn giản và hiệu quả để vận chuyển dữ liệu thông qua mạng công cộng như Internet.
  - GRE đóng gói dữ liệu và chuyển chúng trực tiếp tới thiết bị tách gói tin và định tuyến chúng tới đích cuối cùng

- Lợi ích

  - Sử dụng nhiều giao thức thông qua giao thức chính (IP)
  - Cung cấp cách giải quyết cho mạng giới hạn hop (số lần dịch chuyển của gói tin giữa các máy trạm)
  - Kết nối các mạng con một cách gián tiếp
  - Sử dụng ít tài nguyên hơn các giản pháp thay thế (IPsec VPN)

- GRE Tunneling

  - Dữ liệu được định tuyến bởi hệ thống tới GRE endpoint dựa vào thông tin route table. Khi điểm cuối đường hầm nhận được gói tin, nó sẽ tách gói và định tuyến đến địa chỉ đích
  - GRE tunnel là stateless, điểm cuối đường hầm không chứa trạng thái của remote tunnel endpoint. Do đó switch hoạt động như router nguồn đường hầm không thể thay đổi trạng thái GRE tunnel nếu không liên kết được remote endpoint

- Encapsulation và De-Encapsulation trên switch

  - Encapsulation
    - Switch họa động như tunnel source router đóng gói và chuyển tiếp gói tin GRE 
      - Khi switch nhận được gói tin cần chuyển qua đường hầm, nó sẽ chuyển gói tin ra tunnel interface
      - Tunnel interface đóng gói dữ liệu trong GRE packet và thêm outer IP header
      - Gói tin IP được chuyển đến địa chỉ đích outer IP header (địa chỉ IP tunnel interface của bên nhận)
  - De-Encapsulation
    - Switch hoạt động như tunnel remote router xử lý gói tin GRE
      - Khi switch đích nhận được gói tin IP từ tunnel interface, nó loại bỏ outer IP header và GRE header
      - Gói tin sau bước trên được định tuyến tới địa chỉ IP đích trong header

- GRE frame format

  - GRE thêm tối thiểu 24 byte vào gói tin gồm

    - 20 byte IP header mới
    - 4 byte GRE header
    - Các tùy chọn mở rộng: checksum, key chứng thực, sequence number có thể được thêm vào GRE header

  - Cấu trúc GRE header

    - 2 byte đầu tiên là các bit trường flags

      - | GRE bit | Option          | Description                                                  |
        | ------- | --------------- | ------------------------------------------------------------ |
        | 0       | checksum        | nếu bit 0 set 1, thêm 4 byte checksum sau Protocol type      |
        | 2       | key             | nếu bit 2 set 1, thêm key mã hóa dài 4 byte sau trường checksum |
        | 3       | sequence number | nếu bit 3 set 1, thêm sequence number 4 byte vào GRE header  |
        | 13-15   | version number  | phải chứa giá trị 0                                          |

      - Protocol Type: 2 octets

        - chứa kiểu giao thức của gói payload, nếu chứa giao thức không trong danh sách RFC1700 hoặc [ETYPES]( https://tools.ietf.org/html/rfc2784#ref-ETYPES) thì nên loại bỏ gói

- Phân loại GRE

  - GRE có thể đóng gói bất kỳ gói tin nào của lớp network, cung cấp khả năng định tuyến giữa private network thông qua môi trường internet bằng cách sử dụng địa chỉ IP đã được định tuyến

  - GRE gồm 2 loại

    - Point-to-point: 

      - kiểu truyền thống truyền gói tin đến một địa chỉ nhất địch 

        ![point-to-point-image](https://github.com/vuducmanh11/HPCC_Lab/blob/master/report/week3/images/point_to_point_GRE.png)

    - Point-to-multipoint: 

      - truyền gói tin từ một địa chỉ đến địa chỉ multicast
      - Vì chưa xác định được điểm cuối, nó cần xác định qua giao thức trung gian để ánh xạ địa chỉ tunnel sang cổng vật lý Next Hop Resolution Protocol (NHRP)
      - ![point-to-multipoint-image](https://github.com/vuducmanh11/HPCC_Lab/blob/master/report/week3/images/point_to_multipoint_GRE.png)



