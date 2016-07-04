#OpenStack Compute
#Mục lục
<h4><a href="#overview">5.1. Tóm tắt về dịch vụ OpenStack Compute</a></h4>
<h4><a href="#service">5.2. Các service của OpenStack Compute</a></h4>
<ul>
<li><h5><a href="#api">5.2.1.  API</a></h5></li>
<li><h5><a href="#compute_core">5.2.2. Compute core</a></h5></li>
<li><h5><a href="#network">5.2.3. Networking for VMs</a></h5></li>
<li><h5><a href="#console">5.2.4. Console interface </a></h5></li>
<li><h5><a href="#command_line">5.2.5. Command-line clients and other interfaces</a></h5></li>
<li><h5><a href="#other_component">5.2.6. Other components</a></h5></li>
</ul>
<h4><a href="#request">5.3. Xử lý khi nhận được request</a></h4>
<h4><a href="#install_nova_controller">5.4. Cài đặt và cấu hình nova trên node Controller</a></h4>
<ul>
<li><h5><a href="#create_db_enpoint">5.4.1. Tạo database và endpoint cho nova</a></h5></li>
<li><h5><a href="#end">5.4.2. Kết thúc bước cài đặt và cấu hình nova</a></h5></li>
</ul>
<h4><a href="#install_nova_compute">5.4. Cài đặt và cấu hình nova trên node Compute</a></h4>

---

<h2><a name="overview">5.1. Tóm tắt về dịch vụ OpenStack Compute</a></h2>
Sử dụng OpenStack Compute để lưu trữ và quản lý hệ thống điện toán đám mây. OpenStack compute là 1 phần quan trọng của hệ thống cung cấp dịch vụ hạ tầng điện toán đám mây IaaS (Infrestructure-as-a-Service). Các module chính được viết bằng Python.
</br></br>
<img src="https://github.com/cloudcomputinghust/openstack-manual/blob/master/img_Glance/os%20.png"/>
</br>
OS Compute tương tác với OS Identity (Keystone) để xác thực người dùng, tương tác với dịch vụ OS Image (Glance & Cinder) để lấy đĩa cài hệ điều hành, và OS Dashboard  để cung cấp dịch vụ tương tác cho người sử dụng và giao diện quản trị. Truy cập vào Image bị giới hạn bởi các project và các người dùng, và qouta cung cấp bị giới hạn cho project. OpenStack Compute có thể scale lên phần cứng chuẩn (mở rộng phần cứng vật lý), và tải xuống image để chạy thực thể.
</br>
<img src="https://github.com/cloudcomputinghust/openstack-manual/blob/master/img_Glance/nova%20architecture%202.png"/>
</br></br>
Một số thành phần chính của OpenStack Compute:
</br>
<b>+ Cloud controller:</b> biểu diễn trạng thái tổng thể và tương tác với các thành phần khác. Các máy chủ API đóng vai trò như các web service front-end cho Cloud Controller.
</br>
Compute controller cung cấp các máy chủ tính toán có chứa luôn cả dịch vụ Compute.
</br>
<b>+ Object storage:</b> là 1 thành phần cung cấp các dịch vụ lưu trữ, bạn có thể sử dụng OpenStack Object Storage để thay thế.
</br>
<b>+ Auth manager:</b> cung cấp dịch vụ xác thực và ủy quền khi sử dụng hệ thống Compute. Bạn cũng có thể sử dụng OpenStack Identity như 1 dịch vụ xác thực riêng biệt để thay thế.
</br>
<b>+ Volume controller:</b> cung cấp các khối lưu trữ 1 cách nhanh chóng và thường xuyên cho các máy chủ compute.
</br>
<b>+ Network controller:</b> cung cấp các mạng ảo để cho phép các máy chủ compute tương tác với nhau và với mạng công cộng. Bạn cũng có thể sử dụng OpenStack Networking để thay thế.
</br>
<b>+ Scheduler:</b> được sử dụng để lựa chọn máy compute phù hợp nhất để triển khai 1 instance.
</br></br>
Các thành phần của nova được liên kết với nhau bằng Queue Server.Ở đây, Queue Server dùng rabbitmq, một gói phần mềm chuyên làm nhiệm vụ chuyển request đến đích tương ứng. Do các thành phần của nova đều hoàn toàn độc lập với nhau, có thể chạy trên các máy khác nhau, và số lượng mỗi thành phần là không hạn chế, trong trường hợp có 1 máy bị hỏng, rabbitmq sẽ chọn ra 1 máy khác có cùng dịch vụ để gởi request. Trạng thái của toàn bộ hệ thống sẽ được lưu trong 1 csdl. Cloud controller giao tiếp với các internal object sử dụng HTTP, nhưng nó giao tiếp với các dịch vụ scheduler, network controller và volume controller sử dụng AMQP (Advance Message Queuing Protocol). Để tránh bị block 1 thành phần trong khi đợi 1 response thì Compute sử dụng các cuộc gọi không đồng bộ, với 1 callback sẽ được kích hoạt khi nhận được 1 response. Một hệ điều hành ảo có thể được format và mount 1 thiết bị lưu trữ như vậy.
</br></br>
<b>Ảo hóa (Hypervisor)</b>
</br>
Compute điểu khiển ảo hóa thông qua 1 máy chủ API. Để chọn hypervisor tốt nhất để sử dụng khá khó khăn, bạn phải lấy tài nguyên, các tính năng được hỗ trợ, và yêu cầu thông số kỹ thuật cần thiết trong tài khoản. Tuy nhiên, phần lớn các phát triển của OpenStack sử dụng KVM và Xen-based.
</br></br>
<b>Projects, users và roles</b> </br>
Hệ thống Compute được thiết kế để phục vụ nhiều người dùng khác nhau trong các project trên 1 hệ thống được chia sẻ, và dựa trên các vai trò truy cấp. Các role kiểm soát các hoạt động mà 1 user được phép thực hiện.
</br>
Các project bao gồm một VLAN riêng, và các volume, instance, images, keys, và các user. Một user có thể chỉ định project bằng cách gắn thêm project_id vào khóa truy cập (access key) của họ. Nếu không có project nào được chỉ định trong API request, Compute sẽ chọn 1 project có ID tương tự như user.
</br>
Đối với các project, bạn có thể sử dụng quota control để hạn chế:
<ul>
<li>Số lượng volume có thể được triển khai </li>
<li>Số nhân xử lý và dung lượng Ram có thể được phân bổ</li>
<li>Glián địa chỉ IP cho bất kỳ instance nào khi nó được chạy. Điều này cho phép các instance có thể có cùng 1 địa chỉ IP được public</li>
<li>Cố định các địa chỉ IP được giao cho các instance giống nhau khi nó chạy. Điều này cho phép các instance có thể có địa chỉ IP public hay private giống nhau.</li>
</ul>
Các role kiểm soát các hành động của 1 user được phép thực hiện. Theo mặc định, hầu hết các hành động không yêu cầu 1 role đặc biệt, nhưng bạn có thể cấu hình chúng bằng cách sửa file policy.json để thêm role cho user. Ví dụ, 1 luật có thể được định nghĩa để 1 user phải có quyền admin để có thể phân bổ 1 địa chỉ IP public.
</br> 
Một project hạn chế người dùng truy cập đến các image cụ thể. Mỗi user được gán 1 tên sử dụng và mật khẩu. 1  keypair cấp quyền truy nhập đến 1 instance được kích hoạt cho mỗi user, nhưng quota được thiết lập để cho mỗi project có thể kiểm soát sự tiêu thụ tài nguyên trên phần cứng vật lý có sẵn.
</br></br>
<b>Block storage</b>
</br>
OpenStack cung cấp 2 lớp của block storage: ephemerak storage (lưu trữ ngắn hạn) và persistent volume (volume dài hạn).
</br></br>
<b>Ephemeral storage:</b> bao gồm 1 root ephemeral storage và 1 additional ephemeral volume.
Các đĩa gốc được liên kết với 1 instance, và chỉ tồn tại khi instance này vẫn còn “sống”. Nói chung là nó được sử dụng để lưu trữ các hệ thống tập tin gốc của 1 instance, nó khong bị mất đi khi reboot lại hệ điều hành, nhưng sẽ bị mất khi 1 instance bị xóa. Tổng số root ephemeral volume được xác định bơi flavor của 1 instance.
</br>
Ngoài ephemeral root volume, tất các các kiểu mặc định của flavor, trừ m1.tiny có kích thước nhỏ nhất, thì đều cung cấp thêm 1 additional ephemeral volume có kích thước từ 20-160 GB (giá trị này có thể cấu hình để phù hợp với môi trường). Nó được biểu diễn như 1 block device gốc không có các bảng partition hay các file system. 
</br></br>
<b>Persistent volume:</b> được biểu diễn thông qua 1 persistent virtualized block device của bất kỳ 1 instance riêng biệt nào, và nó được cung cấp bởi OpenStack Block Storage.
Chỉ có 1 cấu hình instance đơn lẻ mới truy cập được vào 1 persistent volume. Multiple instance không thể truy cập vào persistent. Kiểu cấu hình này yêu cầu phải có 1 hệ thống mạng gốc các file systemt  để cho phép multiple instance có thể truy cập vào persistent volume. Những hệ thống này có thể được xây dựng trong 1 OpenStack cluster, hoặc được cung cấp từ bên ngoài, nhưng phần mềm OpenStack không hỗ trợ tính năng này.
</br></br>

<b>Building blocks</b></br>
Trong OpenStack, hệ điều hành cơ sở thường được copy từ 1 image được lưu trữ trong dịch vụ OpenStack Image. Đây là trường hợp phổ biến nhất và kết quả trong 1 ephemeral instance bắt đầu từ 1 trạng thái mẫu được biết đến và mất tất cả các trạng thái trước đó khi máy ảo bị xóa. Nó cũng có thể đặt 1 hệ điều hành trên 1 persistent volume trong hệ thông OpenStack Block Storage volume. Để xem danh sách các image sẵn có trên hệ thống của bạn, bạn chạy lệnh :
```sh
root@controller:/home/hamanhdong# nova image-list
+--------------------------------------+-----------------------------+--------+---------+
| ID                                   | Name                        | Status | Server  |
+--------------------------------------+-----------------------------+--------+---------+
| aee1d242-730f-431f-88c1-87630c0f07ba | Ubuntu 14.04 cloudimg amd64 | ACTIVE |         |
| 0b27baa1-0ca6-49a7-b3f4-48388e440245 | Ubuntu 14.10 cloudimg amd64 | ACTIVE |         |
| df8d56fc-9cea-4dfd-a8d3-28764de3cb08 | jenkins                     | ACTIVE |         |
+--------------------------------------+-----------------------------+--------+---------+
```
Trong đó: </br>
<b>ID: </b>Được tự động tạo ra cho image qua UUID</br>
<b>Name: </b>Không cần theo mẫu cụ thể, có thể là cả tên người đặt cho image cũng được.</br>
<b>Status: </b>trạng thía của image, nếu là active nghĩa là có sẵn để sử dụng.</br>
<b>Server: </b>Đối với các image được tạo ra từ snapshot thì đây là UUID của instance được dùng để snapshot. Còn đối với các image tải lên thì phần này để trống.
</br></br>
Các mẫu phần cứng ảo được gọi là flavor. Theo cài đặt mặc định sẽ có 5 flavor được cung cấp. Đây là những cấu hình bởi admin user, tuy nhiên các hành vi có thể được thay đổi bằng cách xác định lại các điều khiển truy cập cho compute_extension: flavormanage trong file /etc/nova/policy.json trên máy chủ compute-api.
</br>
Dưới đây là danh sách 5 flavor sẵn có trên hệ thống:
</br>
```sh
root@controller:/home/hamanhdong# nova flavor-list
+-----+-----------+-----------+------+-----------+------+-------+-------------+-----------+
| ID  | Name      | Memory_MB | Disk | Ephemeral | Swap | VCPUs | RXTX_Factor | Is_Public |
+-----+-----------+-----------+------+-----------+------+-------+-------------+-----------+
| 1   | m1.tiny   | 512       | 1    | 0         |      | 1     | 1.0         | True      |
| 2   | m1.small  | 2048      | 20   | 0         |      | 1     | 1.0         | True      |
| 3   | m1.medium | 4096      | 40   | 0         |      | 2     | 1.0         | True      |
| 4   | m1.large  | 8192      | 80   | 0         |      | 4     | 1.0         | True      |
| 5   | m1.xlarge | 16384     | 160  | 0         |      | 8     | 1.0         | True      |
+-----+-----------+-----------+------+-----------+------+-------+-------------+-----------+
```

<h2><a name="service" >5.2. Các service của OpenStack Compute</a></h2>
<h3><a name="api">5.2.1.  API</a></h3>
<b>Nova-api service:</b> chấp nhận và phản hồi với người dùng cuối yêu cầu API Compute. Nó hỗ trợ các dịch vụ OpenStack Compute API, Amazon EC2 API, và Admin API đặc biệt để cho người dùng có đặc quyền thực hiện các hành vi chính. Nó thực thi một số chính sách và khởi tạo các hoạt động cùng nhau, ví dụ như chạy 1 instance.
Theo mặc định, nova-api lắng nghe trên cổng 8773 đối với EC2 API và trên cồng 8774 đối với OpenStack API.
</br>
<b>Nova-api-metadata service:</b> chấp nhận các yêu cầu metadata từ các instance. Nova-api-metadata service thường được sử dụng khi bạn chạy ở chế độ đa máy chủ (multi-host) với cài đăth nova-network

<h3><a name="compute_core">5.2.2. Compute core</a></h3>
<img src="https://github.com/cloudcomputinghust/openstack-manual/blob/master/img_Glance/nova%20compute.jpg"/></br>
<b>Nova-compute service:</b> là 1 daemon tạo ra và chấm dứt các thể hiện của máy ảo thông qua các API của công nghệ tạo máy ảo đó. Ví dụ
<ul>
<li>XenAPI cho XenServer/XCP</li>
	<li>Libvirt cho KVM hay QEMU</li>
	<li>VMwareAPI cho VMware</li>
	</ul>
Tiến trình này khá phức tạp, về cơ bản, các daemon chấp nhận các hoạt động từ hàng đợi và thực hiện một loạt các lệnh hệ thống như chạy 1 máy ảo KVM và cập nhật trạng thái của nó trong database.
</br>
<b>Nova-scheduler service:</b> lấy một yêu cầu tạo máy ảo từ hàng đợi và xác định máy chủ compute mà nó chạy.
</br>
<b>Nova-conductor module:</b> trung gian tương tác giữa nova-compute service và database. Nó giúp loại bỏ các truy cập trực tiếp từ các nova-compute service đến database. Nova-conductor module chia theo chiều ngang. Tuy nhiên, không triển khai module này trên các nút có dịch vụ nova-compute chạy.
</br>
<b>Nova-cert module:</b> Một máy chủ daemon phục vụ dịch vụ Nova Cert cho chuẩn X509. Được sử dụng để tạo ra các giấy chứng nhận cho euca-bundle-image. Nó chỉ cần thiết cho EC2 API.

<h3><a name="network">5.2.3. Networking for VMs</a></h3>
<b>Nova-network worker daemon:</b> tương tự như nova-compute, nó chấp nận nhiệm vụ kết nối mạng từ hàng đợi và điều khiển mạng. Nhiệm vụ thực hiện là thiet lập các giao diện cầu nối hoặc thay đổi các luật IPtables.
</br>

<h3><a name="console">5.2.4. Console interface </a></h3>
<b>Nova-consoleauth daemon: </b>dịch vụ này phải chạy trong giao diện proxy để hoạt động. Bạn có thể chạy các proxy của 1 trong 2 loại đối với 1 dịch vụ nova-consoleauth trong 1 cấu hình cluster duy nhất.
</br>
<b>Nova-novncproxy daemon:</b> cung cấp 1 proxy để truy cập chạy các instance thông qua 1 kết nối VNC. Hỗ trợ các client trên nền web với novnc.
</br>
<b>Nova-spicehtml5proxy daemon:</b> cung cấp 1 proxy để truy cập chạy các instance thông qua kết nối SPICE. Hỗ trợ các client trên nền web với HTML5
</br>
<b>Nova-xvpvncproxy daemon:</b> cung cấp 1 proxy để truy cập chạy các instance thông qua kết nối VNC. Hỗ trợ client Java OpenStack cụ thể.
</br>
<b>Nova-cert daemon:</b> chứng nhận X509 (trong nova-cert module)
</br>

<h3><a name="command_line">5.2.5. Command-line clients and other interfaces</a></h3>
<b>Nova client:</b> cho phép người dùng thực thi các lệnh như 1 quản trị viên được ủy quyền hay người dùng cuối.
</br>

<h3><a name="other_component">5.2.6. Other components</a></h3>
<b>Hàng đợi:</b> một trung tâm để truyền các tin nhắn giữa các daemon. Thường được thực hiện với RabbitMQ, nhưng về mặt lý thuyết có thể được thực hiện với 1 hàng đợi tin nhắn AMQP chẳng hạn như Zero MQ.
Nova tạo ra một số loại hàng đợi thông điệp để tạo điều kiện giao tiếp giữa các daemon khác nhau. Chúng bao gồm: topics queue, fanout queue, và host queue. Topics queue sẽ cấp số cho các tin nhắn truyền đi vào các class riêng ứng với topic tương ứng của các worker daemon. Ví dụ, Nova sử dụng chúng để truyền các tin nhắn tới tất cả (hoặc một số) các volume daemon hay compute. Điều này cho phép Nova sử dụng worker đầu tiên có sẵn để xử lý tin nhắn. Host queue cho phép Nova gửi tin nhắn tới các dịch vụ riêng biệt trên các máy chủ cụ thể nào đó. Fanout queues hiện đang được sử dụng cho các thông báo của các dịch vụ phục vụ cho nova-scheduler worker.
Để xem các topic trong RabbitMQ, bạn sử dụng lệnh:
<img src="https://github.com/cloudcomputinghust/openstack-manual/blob/master/img_Glance/topic%20rabbitmq.jpg"/>
</br>
Để xem các hàng đợi được tạo ra trong RabbitMQ cho 1 node được cài đặt đơn giản, các bạn có thể dùng lệnh:
</br>
```sh
$ sudo rabbitmqctl list_queues
```
</br>
Kết quả:
</br>
<img src="https://github.com/cloudcomputinghust/openstack-manual/blob/master/img_Glance/list%20queues.jpg"/>
</br>
<b>Cơ sở dữ liệu SQL: </b>lưu trữ tất cả các trạng thái build-time và run-time cho 1 cơ sở hạ thầng điện toán đám mây, bao gồm:
	<ul>
	<li>Các loại instance sẵn có</li>
	<li>Các instance trong sử dụng</li>
	<li>Các project</li>
	</ul>
Về mặt lý thuyết, OpenStack Compute có thể hỗ trợ bất kỳ cơ sở dữ liệu nào mà SQL-Alchemy hỗ trợ. Csdl phổ biến là SQLite3 để thử nghiệm và phát triển công việc, MySQL và PostgreSQL.
</br></br>

<h2><a name="request">5.3. Xử lý khi nhận được request</a></h2>
</br>
<img src="https://ilearnstack.files.wordpress.com/2013/04/request-flow1.png"/>
</br>
link: https://ilearnstack.com/2013/04/26/request-flow-for-provisioning-instance-in-openstack/</br>
1. Từ Dashboard hoặc CLI, user nhập các thông tin chứng thực (ví dụ như user name và password) và thực hiện lời gọi REST tới Keystone để xác thực.</br>
2. Keystone sẽ xác thực thông tin người dùng và tạo ra 1 token, gửi trở lại cho user để user có thể sử dụng nó để xác thực sử dụng các dịch vụ khác thông qua REST</br>
2b. Nova API tạo 1 entry trong nova db</br>
3. Dashboard/CLI chuyển yêu cầu tạo máy ảo tới nova-api thông qua REST API request</br>
4. Nova-api nhận yêu cầu và hỏi lại keystone xem auth-token mang theo yêu cầu tạo máy ảo của user có hợp lệ không và quyền hạn truy cập của user đó khi auth-token hợp lệ</br>
5. Keystone xác nhận token và update lại trong header xác thực với roles và quyền hạn truy cập dịch vụ lại cho nova-api </br>
6. Nova-api tương tác với nova-database để yêu cầu tạo ra 1 entry lưu các thông tin về máy cảo mới cần tạo </br>
7. Database tạo entry lưu thông ti máy ảo mới </br>
8. Nova-api gửi rpc.call request tới nova-scheduler để cập nhật entry của máy ảo mới với giá trị hostID (ID của máy compute mà máy ảo sẽ được triển khai trên đó). </br>
9. Nova-scheduler lấy yêu cầu từ hàng đợi </br>
10. Nova-scheduler tương tác với database để tìm host compute phù hợp thông qua các yêu cầu về cấu hình của máy ảo </br>
11. Nova-database cập nhật lại entry của máy ảo với host ID phù hợp tìm được sau khi lọc </br>
12. Nova-scheduler gửi rpc.cast request tới nova-compute chứa yêu cầu tạo máy ảo mới với host phù hợp.</br>
13. Nova-compute lấy message từ hàng đợi </br>
14. Nova-compute gửi rpc.call request tới nova-conductor để lấy thông tin như host ID và flavor (các thông tim về RAM, CPU, disk) </br>
15. Nova-conductor lấy message từ hàng đợi </br>
16. Nova-conductor tương tác với database để lấy thông tin mà nova-compute yêu cầu</br>
17. Nova-database trẻ lại thông tin của máy ảo mới cho nova-conductor và nova-conductor gửi thông tin này lên hàng đợi cho nova-compute </br>
18. Nova-compute lấy mesage chứa thông tin về máy ảo từ hàng đợi </br>
19. Nova-compute thực hiện lời gọi REST bằng việc gửi token xác thực tới glance-api để lấy Image URI, Image ID và upload image từ image store </br>
20. Glance-api xác thực auth-token với keystone </br>
21. Nova-compute lấy metadata của image (bao gồm image type, size, etc...) </br>
22. Nova-compute thực hiện REST-call bằng việc gửi token xác thực tới Netword APU để xin cấp phát IP và cấu hình mạng cho máy ảo </br>
23. Neutron server (tên gọi mới của quantum) xác thực token với keystone </br>
24. Nova-compute lấy các thông tin về network </br>
25. Nova-compute thực hiện REST-call bằng việc gửi token tới Volume API để yêu cầu volumes cho máy ảo </br>
26. Cinder-api xác thực token với Keystone </br>
27. Nova-compute lấy thông tin bloock storage cấp cho máy ảo </br>
28. Nova-compute tạo ra dữ liệu cho hypervisor drriver và thực thi yêu cầu tạo máy ảo trên Hypervisor
</br></br>

<h2><a name="install_nova_controller">5.4. Cài đặt và cấu hình nova trên node Controller</a></h2>
<h3><a name="create_db_enpoint">5.4.1. Tạo database và endpoint cho nova</a></h3>
Đăng nhập vào database với quyền root
</br>

```sh
$ mysql -u root -p
```
Tạo database
</br>
```sh
CREATE DATABASE nova_api;
CREATE DATABASE nova;
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' \
  IDENTIFIED BY 'NOVA_DBPASS';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' \
  IDENTIFIED BY 'NOVA_DBPASS';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' \
  IDENTIFIED BY 'NOVA_DBPASS';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' \
  IDENTIFIED BY 'NOVA_DBPASS';
```
Khai báo biến môi trường
 ```sh
$ . admin.sh
  ```
+ Tạo user, phân quyền và tạo endpoint cho dịch vụ nova
</br>
Tạo user có tên là nova
 ```sh
openstack user create nova --domain default  --password Welcome123
  ```
Phân quyền cho tài khoản nova
 ```sh
$ openstack role add --project service --user nova admin
  ```
  Tạo service có tên là nova
  ```sh
$ openstack service create --name nova \
  --description "OpenStack Compute" compute
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Compute                |
| enabled     | True                             |
| id          | 060d59eac51b4594815603d75a00aba2 |
| name        | nova                             |
| type        | compute                          |
+-------------+----------------------------------+
  ```
Tạo endpoint
 ```sh
$ openstack endpoint create --region RegionOne \
  compute public http://controller:8774/v2.1/%\(tenant_id\)s
+--------------+-------------------------------------------+
| Field        | Value                                     |
+--------------+-------------------------------------------+
| enabled      | True                                      |
| id           | 3c1caa473bfe4390a11e7177894bcc7b          |
| interface    | public                                    |
| region       | RegionOne                                 |
| region_id    | RegionOne                                 |
| service_id   | e702f6f497ed42e6a8ae3ba2e5871c78          |
| service_name | nova                                      |
| service_type | compute                                   |
| url          | http://controller:8774/v2.1/%(tenant_id)s |
+--------------+-------------------------------------------+

$ openstack endpoint create --region RegionOne \
  compute internal http://controller:8774/v2.1/%\(tenant_id\)s
+--------------+-------------------------------------------+
| Field        | Value                                     |
+--------------+-------------------------------------------+
| enabled      | True                                      |
| id           | e3c918de680746a586eac1f2d9bc10ab          |
| interface    | internal                                  |
| region       | RegionOne                                 |
| region_id    | RegionOne                                 |
| service_id   | e702f6f497ed42e6a8ae3ba2e5871c78          |
| service_name | nova                                      |
| service_type | compute                                   |
| url          | http://controller:8774/v2.1/%(tenant_id)s |
+--------------+-------------------------------------------+

$ openstack endpoint create --region RegionOne \
  compute admin http://controller:8774/v2.1/%\(tenant_id\)s
+--------------+-------------------------------------------+
| Field        | Value                                     |
+--------------+-------------------------------------------+
| enabled      | True                                      |
| id           | 38f7af91666a47cfb97b4dc790b94424          |
| interface    | admin                                     |
| region       | RegionOne                                 |
| region_id    | RegionOne                                 |
| service_id   | e702f6f497ed42e6a8ae3ba2e5871c78          |
| service_name | nova                                      |
| service_type | compute                                   |
| url          | http://controller:8774/v2.1/%(tenant_id)s |
+--------------+-------------------------------------------+
  ```
  Cài đặt xác gói và cấu hình:
```sh
# apt-get install nova-api nova-conductor nova-consoleauth \
  nova-novncproxy nova-scheduler
  ```  
  Sao lưu file /etc/nova/nova.conf trước khi cấu hình
  ```sh
# cp /etc/nova/nova.conf /etc/nova/nova.conf.orig
  ```
  Chỉnh sửa file /etc/nova/nova.conf như dưới: </br>
<b>Lưu ý:</b> Trong trường hợp nếu có dòng khai bao trước đó thì tìm và thay thế, chưa có thì khai báo mới hoàn toàn.
Khai báo trong section [api_database] dòng dưới, do section [api_database] chưa có nên ta khai báo thêm
 ```sh
[api_database]
connection = mysql+pymysql://nova:NOVA_DBPASS@controller/nova_api
  ```
  Khai báo trong section [database] dòng dưới. Do section [database] chưa có nên ta khai báo thêm.
   ```sh
[database]
connection = mysql+pymysql://nova:NOVA_DBPASS@controller/nova
  ``` 
  Trong section [DEFAULT] :
  <ul>
  <li>Thay dòng:</br> logdir=/var/log/nova
  </br>Bằng dòng:</br> log-dir=/var/log/nova</li>
  <li>Thay dòng:</br> enabled_apis=ec2,osapi_compute,metadata
  </br>Bằng dòng:</br> enabled_apis=osapi_compute,metadata</li>
  <li>Bỏ dòng:</br> verbose = True</li>
  <li>Khai báo thêm các dòng sau:</br>
   ```sh
rpc_backend = rabbit
auth__strategy = keystone
rootwrap_config = /etc/nova/rootwrap.conf
#IP MGNT cua node Controller
my_ip = 10.10.10.40
</br>
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver
  ```
  </li>
  </ul>
Khai báo trong section [oslo_messaging_rabbit] các dòng dưới. Do section[oslo_messaging_rabbit] chưa có nên ta khai báo thêm.
```sh
[oslo_messaging_rabbit]
rabbit_host = controller
rabbit_userid = openstack
rabbit_password = Welcome123
  ```
Trong section [keystone_authtoken] khai báo các dòng dưới. Do section[keystone_authtoken] chưa có nên ta khai báo thêm.
 ```sh
[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = Welcome123
  ```
  Trong section [vnc] khai báo các dòng dưới để cấu hình VNC điều khiển các máy ảo trên web. Do section [vnc] chưa có nên ta khai báo thêm.
  ```sh
[vnc]
vncserver_listen = $my_ip
vncserver_proxyclient_address = $my_ip
  ``` 
Trong section [glance] khai báo dòng để nova kết nối tới API của glance. Do section [glance] chưa có nên ta khai báo thêm.
 ```sh
[glance]
api_servers = http://controller:9292
  ```
  Trong section [oslo_concurrency] khai báo dòng dưới. Do section [oslo_concurrency]chưa có nên ta khai báo thêm.
   ```sh
[oslo_concurrency]
lock_path = /var/lib/nova/tmp
  ```
  Khai báo thêm section mới [neutron] để nova làm việc với neutron
   ```sh
[neutron]
url = http://controller:9696
auth_url = http://controller:35357
auth_type = password
project_domian_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = Welcome123
</br>
service_metadata_proxy = True
metadata_proxy_shred_secret = Welcome123
  ```
  Tạo database cho nova:
   ```sh
# su -s /bin/sh -c "nova-manage api_db sync" nova
# su -s /bin/sh -c "nova-manage db sync" nova
  ```
  
  <h3><a name="end">5.4.2. Kết thúc bước cài đặt và cấu hình nova</a></h3>
  Khởi động lại các dịch vụ của nova sau khi cài đặt & cấu hình nova
   ```sh
# service nova-api restart
# service nova-consoleauth restart
# service nova-scheduler restart
# service nova-conductor restart
# service nova-novncproxy restart
  ```
Xóa database mặc định của nova
   ```sh
# rm –f /var/lib/nova/nova.sqlite
  ```
  Kiểm tra các service của nova hoạt động hay chưa bằng lệnh dưới
```sh
# openstack compute service list
+----+--------------------+------------+----------+---------+-------+----------------------------+
| Id | Binary             | Host       | Zone     | Status  | State | Updated At                 |
+----+--------------------+------------+----------+---------+-------+----------------------------+
|  1 | nova-consoleauth   | controller | internal | enabled | up    | 2016-03-27T16:19:42.000000 |
|  2 | nova-scheduler     | controller | internal | enabled | up    | 2016-03-27T16:19:42.000000 |
|  3 | nova-conductor     | controller | internal | enabled | up    | 2016-03-27T16:19:42.000000 |
|  5 | nova-cert          | controller | internal | enabled | up    | 2016-03-27T16:19:42.000000 |
|  6 | nova-osapi_compute | 0.0.0.0    | internal | enabled | down  | None                       |
|  7 | nova-metadata      | 0.0.0.0    | internal | enabled | down  | None                       |
+----+--------------------+------------+----------+---------+-------+----------------------------+
  ```
<h2><a name="install_nova_compute">5.4. Cài đặt và cấu hình nova trên node Compute</a></h2> </br>
Cài đặt gói nova-compute </br>
```sh
apt-get -y install nova-compute
 ```
<b>Cấu hình nova-comupte</b> </br>
+ Sao lưu file /etc/nova/nova.conf </br>
	```sh
	cp /etc/nova/nova.conf /etc/nova/nova.conf.orig
	 ```
+ Trong section [DEFAULT] khai báo các dòng sau: </br>
  ```sh
pc_backend = rabbit
auth_strategy = keystone
my_ip = 10.10.10.41
	
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver
 ```
+ Khai báo thêm section [oslo_messaging_rabbit] và các dòng dưới: </br>
  ```sh
[oslo_messaging_rabbit]
rabbit_host = controller
rabbit_userid = openstack
rabbit_password = Welcome123
 ```

+ Khai báo thêm section [keystone_authtoken] và các dòng dưới: </br>
 ```sh
[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = Welcome123
 ```
+ Khai báo thêm section [vnc] và các dòng dưới: </br>
 ```sh
[vnc]
enabled = True
vncserver_listen = 0.0.0.0
vncserver_proxyclient_address = $my_ip
novncproxy_base_url = http://172.16.69.40:6080/vnc_auto.html
 ```
+ Khai báo thêm section [glance] và các dòng dưới: </br>
 ```sh
[glance]
api_servers = http://controller:9292
 ```
+ Khai báo thêm section [oslo_concurrency] và các dòng dưới: </br>
 ```sh
[oslo_concurrency]
lock_path = /var/lib/nova/tmp
 ```
+ Khai báo thêm section [neutron] và các dòng dưới: </br>
 ```sh
[neutron]
url = http://controller:9696
auth_url = http://controller:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = Welcome123
 ```

Khởi động lại dịch vụ nova-compute
```sh
service nova-compute restart
 ```
Xóa database mặc định của hệ thống tạo ra
```sh
rm -f /var/lib/nova/nova.sqlite
 ```
Dùng lệnh nano để thêm file admin-openrc chứa nội dung dưới:
```sh
export OS_PROJECT_DOMAIN_NAME=default
export OS_USER_DOMAIN_NAME=default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=Welcome123
export OS_AUTH_URL=http://controller:35357/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
 ```
Thực thi file admin-openrc
```sh
source admin-openrc
 ```
Kiểm tra lại các dịch vụ của nova đã được cài đặt thành công hay chưa:
```sh
root@compute1:~# openstack compute service list
+----+------------------+------------+----------+---------+-------+----------------------------+
| Id | Binary           | Host       | Zone     | Status  | State | Updated At                 |
+----+------------------+------------+----------+---------+-------+----------------------------+
|  3 | nova-cert        | controller | internal | enabled | up    | 2016-04-15T15:10:30.000000 |
|  4 | nova-consoleauth | controller | internal | enabled | up    | 2016-04-15T15:10:30.000000 |
|  5 | nova-scheduler   | controller | internal | enabled | up    | 2016-04-15T15:10:31.000000 |
|  6 | nova-conductor   | controller | internal | enabled | up    | 2016-04-15T15:10:32.000000 |
|  7 | nova-compute     | compute1   | nova     | enabled | up    | 2016-04-15T15:10:25.000000 |
+----+------------------+------------+----------+---------+-------+----------------------------+
 ```


