# virtual network

## 1. Virtual network switches

 virtual network switch là một phần mềm hoạt động trên một máy chủ vật lý giúp cho các máy ảo(guests)kết nối.

## 2. Bridged Mode

Khi sử dụng chế độ bridged mode toàn bộ các máy ảo sẽ ở trong cùng một subnet như một máy chủ vật lý(các máy tính trong mạng sẽ coi máy ảo như một máy chủ vật lý). Tất cả các máy chủ vật lý khác có thể  biết được sự xuất hiện của các máy ảo đó trong mạng và có thể  truy cập vào các máy ảo đó. Chế độ này họat động trên layer2 trong mô hình OSI. Sơ đồ chế độ này như hình sau:![https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/vn-Bridged-Mode-Diagram.png](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/vn-Bridged-Mode-Diagram.png)

Các kịch bản sử dụng bridged mode:
- Triển khai các máy ảo trong một mạng với các máy chủ vật lý làm cho sự khác biệt giữa máy vật lý và máy ảo là trong suốt với người dùng.
- Triển khai các máy ảo mà không muốn thực hiện bất kỳ thay đổi cấu hình nào với các máy vật lý hiện có.
- Triển khai các máy ảo có thể dễ dàng truy cập vào các máy chú vật lý hiện có.

## 3. NAT mode

Đây là chế độ mặc định cho virtual network switches. Chế độ này sử dụng ip ảo(IP masquerading). Chế độ này cho phép máy ảo sử dụng địa chỉ IP của máy chủ vật lý để giao tiếp với bất kỳ mạng bên ngoài nào. Mặc định máy tính nào ở bên ngoài máy chủ vật lý sẽ không thể  `khởi tạo` giao tiếp với máy ảo trong chế độ NAT mode. Sơ đồ chế độ này như hình sau:

![https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/vn-04-hostwithnatswitch.png](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/vn-04-hostwithnatswitch.png)

### 3.1 DNS và DHCP

Địa chỉ IP có thể được gán cho máy ảo thông qua DHCP. Một tập các địa chỉ IP có thể được gán cho các virtual network switches. Libvirt sử dụng chương trình `dnsmasq` để thực hiện việc này. Chương trình này sẽ tự động cấu hình cho mỗi virtual network switch khi cần.

![https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/vn-05-switchwithdnsmasq.jpg](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/vn-05-switchwithdnsmasq.jpg)

## 4. Routed mode

Trong chế độ này máy chủ vật lý họat động như một router cho phép các máy bên ngoài giao tiếp với máy ảo thông qua địa chỉ IP. Virtual switch có thể xem xét toàn bộ packet đi qua và sử dụng địa chỉ IP trong các packet để đưa ra quyết định định tuyến. Trong chế độ này các máy ảo đều thuộc cùng một mạng con và được định tuyến thông qua một virtual switch. Chế độ này không phải lúc nào cũng là lý tưởng vì các máy chủ vật lý khác trong cùng một mạng vật lý sẽ không thể biết về các máy ảo nếu không được cấu hình router bằng tay và do đó không thể giao tiếp với máy ảo. Chế độ routed hoạt động ở trên layer3 trong mô hình OSI. Sơ đồ chế độ này như trong hình sau:

![https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/vn-06-routed-switch.png](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/vn-06-routed-switch.png)

## 5. Isolated mode

Khi sử dụng chế  độ isolated máy ảo có thể giao tiếp với nhau và giao tiếp với máy chủ vật lý nhưng chúng không thể giao tiếp với các máy chủ bên ngoài máy chủ vật lý(cả vào và ra đều không thể). Sử dụng `dnsmasq` là bắt buộc cho những chức năng như DHCP. Tuy nhiên ngay cả khi bị cô lập với tất cả các mạng vật lý các tên DNS vẫn được giải quyết(resolved). Do đó một tình huống xảy ra khi các tên DNS được giải quyết nhưng các yêu cầu ICMP (ping) lại không thể thực hiện được.

![https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/vn-07-isolated-switch.png](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/vn-07-isolated-switch.png)
# Tạo 2 máy ảo với cấu hình:

- RAM: 1GB
- Ổ cứng: 20GB
- CPU: 2 core
- Mỗi máy có 2 card mạng như hình sau:

![https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/vm_diagram.png](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/vm_diagram.png)

Các bước tiến hành:

Bước 1: Tạo máy ảo với card mạng mặc định(NAT mode)

![https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/may_ao_1.png](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/may_ao_1.png)

![https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/may_ao_2.png](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/may_ao_2.png)

![https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/may_ao_3.png](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/may_ao_3.png)

![https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/may_ao_4.png](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/may_ao_4.png)

![https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/may_ao_5.png](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/may_ao_5.png)


Bước 2: Tạo virtual network(isolated mode)
![https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/vn_1.png](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/vn_1.png)

![https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/vn_2.png](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/vn_2.png)

![https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/vn_3.png](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/vn_3.png)

![https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/vn_4.png](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/vn_4%5C.png)

![https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/vn_5.png](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/vn_5.png)

![https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/vn_6.png](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/vn_6.png)

Bước 3: Add thêm card mạng isolated mode cho máy ảo
![https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/ad_1.png](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/ad_1.png)

![https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/ad_2.png](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/ad_2.png)
Bước 4: Tạo IPv4 tĩnh cho từng máy ảo

- Chỉnh sửa file /etc/network/interfaces
 `sudo vim /etc/network/interfaces`
  với nội dung như sau

  ```ssh
  auto lo
  iface lo inet loopback
  auto ens3  # ens3 tên của card mạng cần cấu hình
  iface ens3 inet static # static cho biết đây là địa chỉ IP tĩnh được cấu hình với các tham số như ở dưới
  	address 192.168.10.10 # Địa chỉ IP của máy ảo
  	netmask 255.255.255.0 # Mặt nạ mạng của subnet chứa máy ảo
  	gateway 192.168.10.1  # Cổng mặc định sử dụng khi chuyển gói tin ra ngoài subnet
    dns-nameservers 8.8.8.8 8.8.4.4 # Địa chỉ máy chủ phân giải tên miền
    broadcast 192.168.10.255 # Địa chỉ IP để quảng bá gói tin đến toàn bộ các máy trong cùng subnet

```

- Sau khi đã cấu hình xong tiến hành restart network bằng câu lệnh: `sudo systemctl status networking.service`.

## Các câu lệnh và các file liên quan đến cấu hình mạng trong ubuntu

- Hostname
  - Câu lệnh thay đổi tên của host
  `hostnamectl set-hostname <tên hostname>`
  - File chứa tên của host `/etc/hostname`
- Quản lý network interfaces
  - Liệt kê tất cả các card mạng trong máy tính
  `ip addr`
  - Quản lý trạng thái của từng card mạng
    - Romove địa chỉ IP đã được gán cho card mạng và không cho phép card kết nối tới networks
    `ip link set <tên card mạng> down`
    - Back up lại một card mạng
    `ip link set <tên card mạng> up`

- Gán địa chỉ IP tĩnh
  - Chỉnh sửa file /etc/network/interfaces
   `sudo vim /etc/network/interfaces`
    với nội dung như sau

    ```ssh
    auto lo
    iface lo inet loopback
    auto ens3  # ens3 tên của card mạng cần cấu hình
    iface ens3 inet static # static cho biết đây là địa chỉ IP tĩnh được cấu hình với các tham số như ở dưới
    	address 192.168.10.10 # Địa chỉ IP của máy ảo
    	netmask 255.255.255.0 # Mặt nạ mạng của subnet chứa máy ảo
    	gateway 192.168.10.1  # Cổng mặc định sử dụng khi chuyển gói tin ra ngoài subnet
      dns-nameservers 8.8.8.8 8.8.4.4 # Địa chỉ máy chủ phân giải tên miền
      broadcast 192.168.10.255 # Địa chỉ IP để quảng bá gói tin đến toàn bộ các máy trong cùng subnet

  ```
  - Sau khi đã cấu hình xong tiến hành restart network bằng câu lệnh: `sudo systemctl status networking.service`.

## OpenSSH

OpenSSH cho phép mở shell trên một server linux khác, cho phép bạn thực hiện các lệch như thể bạn đang thực hiện trực tiếp trên máy đó.

Để có thể kết sử dụng ta cần phải cài các package ở cả phía client(máy dùng để  nhập câu lệnh và gửi tới máy server) và máy server(máy nhận câu lệnh từ client và thực hiện trả kết quả lại cho client)
- Phía server: `# apt-get install openssh-server `
- Phía client: `# apt-get install openssh-client`

Để kết nối tới máy chủ bằng ssh ta có thể sử dụng tên hoặc địa chỉ IP của máy chủ như sau:

Cách 1:` ssh < địa chỉ IP>` Mặc định tên người sử dụng được dùng sẽ là tên đang được logged.
Cách 2: `ssh <username>@<địa chỉ IP>` Ngoài ra có thể tùy chỉnh tên người dùng.
Cách 3: `ssh -p <số hiệu cổng> <username>@<địa chỉ IP>` Theo mặc định ssh sẽ sử dụng cổng 22 nhưng t cũng có thể tùy chỉnh cổng này.
