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

![https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/cf_1.png](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/cf_1.png)

![https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/cf_2.png](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/cf_2.png)

![https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/cf_3.png](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/cf_3.png)
