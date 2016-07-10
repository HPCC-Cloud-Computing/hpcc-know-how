##Giới thiệu về dịch vụ mạng trong OpenStack
Một trong những yêu cầu quan trọng khi thiết lập một hệ thống đám mây là phải cung cấp cho các thành phần trong đám mây khả năng kết nối với nhau và kết nối ra bên ngoài, tức là cần thiết lập được một hệ thống mạng trong đám mây. Với đối tượng phục vụ kết nối mạng chính trong các đám mây là hệ thống máy ảo, đồng thời cùng với yêu cầu phải đảm bảo các tính chất của một đám mây- một hệ thống phân tán, OpenStack đã giải quyết bằng cách sử dụng một gói dịch vụ cho phép thiết lập một hệ thống mạng ảo trên đám mây. Gói dịch vụ cung cấp các dịch vụ mạng cho hệ thống OpenStack có tên là neutron.

Nhiệm vụ chính của dịch vụ Neutron là:

•	Tạo ra và quản lỷ các đối tượng và các thiết bị mạng ảo: router, switch, bridge, ip, subnet, network….

•	Sử dụng các thiết bị mạng ảo để xây dựng nên hệ thống mạng ảo, cung cấp dịch vụ mạng cho các dịch vụ và các đối tượng khác trong OpenStack.

•	Cung cấp các API cho phép người dùng thiết lập môi trường mạng cho và đánh địa chỉ cho môi trường mạng người đó thiết lập.

•	Cung cấp các dịch vụ liên quan tới mạng như cấp phát địa chỉ (DHCP), định tuyến (routing), DNS, cân bằng tải (Load-banlance)…

Chúng ta sẽ xem xét mô hình mạng ảo gồm những thành phần nào và được triển khai trên môi trường thực tế như thế nào.
##2 Mô hình hệ thống mạng ảo (virtual networking) trong OpenStack
###2.1 Các thành phần của hệ thống mạng ảo 
Để xây dựng nên hệ thống mạng ảo, chúng ta cần tạo ra các thiết bị mạng ảo tương ứng với các thiết bị mạng vật lý. Các thiết bị được ảo hóa là: Card mạng (Network Card Interfae –NIC), bridge, switch, router, DHCP, … Vai trò của từng thiết bị trong mạng cụ thể như sau:

•	NIC: Cổng kết nối internet, cung cấp kết nối mạng cho các máy ảo.

•	Bridge, switch: Kết nối các thiết bị  mạng và các máy ảo với nhau ở lớp Layer2 tạo thành các mạng cục bộ.

•	Router: Kết nối Các Switch với nhau để kết nối các mạng cục bộ với nhau và cung cấp kết nối internet cho các mạng cục bộ.

•	DHCP: Cấp phát địa chỉ IP cho các máy trong mạng.

Sau khi đã xác định được các thiết bị đã có trong mạng ảo, chúng ta sẽ sử dụng các thành phần này để triển khai trên các node tạo ra hạ tầng mạng. Có rất nhiều cách triển khai, ví dụ như sơ đồ sau là một cách triển khai hạ tầng trên network node:

![virtual-device-example.png](./img/virtual-device-example.png)

Sau khi triển khai xong hạ tầng mạng ảo, các user có thể sử dụng các dịch vụ mà neutron cung cấp để xây dựng nên các mạng ảo cho project của user đó và cung cấp kết nối mạng cho các máy ảo trong project. 
###2.2 Ví dụ về một hệ thống mạng trong OpenStack cloud
Để hiểu rõ hơn về cách triển khai một hệ thống mạng trên OpenStack cloud, chúng ta cùng xem xét 1 ví dụ. Dưới đây là một hệ thống mạng self-service  được triển khai trên cấu hình vật lý gồm 1 node controller và một node compute.Ở đây chúng ta sẽ triển khai một hệ thống mạng self service sử dụng linux-bridge làm L2 plugin, hệ thống này cho phép người dùng có thể tạo ra các mạng ảo riêng cho các project (private self-service network). Sơ đồ triển khai hệ thống mạng như sau:

![self-service-architect1.png](./img/self-service-architect1.png)

Triển khai chi tiết trong compte node và controller node được thể hiện như sau:

Tại compute node:

![self-service-architect-compute-node.png](./img/self-service-architect-compute-node.png)

Tại controller node:

![self-service-architect-controller-node.png](./img/self-service-architect-controller-node.png)

Ở đây, chúng ta có thể thấy, một hệ thống mạng ảo được xây dựng dựa trên các thành phần thiết bị mạng ảo như Switch-Bridge, các card mạng (eth), router, dhcp,... Sau đó các thành phần thiết bị trên được triển khai trên các node tương ứng để cấu thành hệ thống mạng.

Trong mô hình mạng self-service, các thành phần ảo của hệ thống mạng được triển khai trên các node tương ứng. Ví dụ, trên compute-node, các bridge sẽ được triển khai để kết nối các máy ảo với hệ thống mạng, trên compute node, các bridge ảo sẽ được tạo ra, sau đó, hệ thống sẽ tạo ra các thiết bị dhcp ảo và router ảo rồi kết nối chúng tới các bridge. Sau khi đã tạo ra các thiết bị ảo, chúng ta sẽ kết nối các thiết bị ảo này tới các thiết bị vật lý. Ở đây, ta có thể thấy các thiết bị mạng ảo và các máy ảo kết nối tới các bridge ảo, các bridge ảo lại kết nối tới các card mạng vật lý trên các node. Các card mạng vật lý này lại được kết nối với nhau thông qua hệ thống mạng vật lý. Do đó sau khi các thành phần được kết nối với nhau, thì kết nối giữa các thiết bị trên hệ thống mạng ảo của chúng ta sẽ được thông suốt.

Như vậy, bằng cách tạo ra và kết nối các thiết bị mạng ảo với nhau, chúng ta có thể thiết lập được một hệ thống mạng ảo phục vụ cho cloud. Vấn đề đặt ra ở đây là các thiết bị mạng ảo này sẽ hoạt động như thế nào để phục vụ các chức năng về mạng mà 1 cloud yêu cầu, vấn đề này sẽ được làm rõ trong các phần tiếp theo.
