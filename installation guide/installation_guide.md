#Giới thiệu về OpenStack
OpenStack là một hệ thống cung cấp khả năng triển khai đám mây trên một nền tảng hệ thống máy chủ vật lý. Sử dụng OpenStack, người dùng có thể tạo ra, sử dụng và quản lý  một đám mây với các tài nguyên điện toán, lưu trữ và mạng thông qua  nhiều phương tiện khác nhau như giao diện dòng lệnh (CLI) hoặc thông qua giao diện web.

Nói một cách dễ hiểu, OpenStack là một gói các dịch vụ cho phép thiết lập một đám mây trên 1 nền tảng vật lý. Điều kiện cần để triển khai OpenStack là chúng ta có một hệ thống máy chủ vật lý được kết nối với nhau. Sau đó trên từng đơn vị máy thành viên trong hệ thống sẽ được triển khai các dịch vụ của đám mây như: Xác thực(identity) , điện toán (compute), mạng(network), lưu trữ (storage), giao diện web… để tạo thành một đám mây hoàn chỉnh. Tùy thuộc vào dịch vụ được triển khai trên các máy vật lý, các tài nguyên vật lý sẽ được ánh xạ lên đám mây tạo ra các tài nguyên trên đám mây như các máy ảo, hệ thống lưu trữ và mạng.

Các dịch vụ chính trong OpenStack :

***
Dịch vụ | Tên Project | Mô tả
------------ | ------------- | -------------
Dashboard | Horizon | Cung cấp cổng truy cập trên nền web cho phép tương tác với các dịch vụ OpenStack phía dưới, như chạy 1 thực thể, gán địa chỉ IP và sửa đổi các điều khiển truy cập 
Compute | Nova | Quản lý các thực thể trong môi trường OpenStack, bao gồm sinh ra, lập lịch và ngừng hoạt động các thực thể (chính là các máy ảo) dựa trên nhu cầu
Networking | Neutron | Kích hoạt dịch vụ mạng cho các dịch vụ OpenStack khác, ví dụ như cho dịch vụ Compute. Dịch vụ Networking cung cấp cho người dùng các API quản lý các mạng ảo, ví dụ như định nghĩa 1 mạng mới và gắn các địa chỉ cho các máy ảo, dịch vụ này có 1 kiến trúc mở, hỗ trợ rất nhiều các nhà cung cấp và các công nghệ mạng khác nhau
Object Storage | Swift | Lưu trữ và lấy các đối tượng dữ liệu không cấu trúc thông qua RESTful và HTTP API . Có tính chịu lỗi cao với sao lưu dữ liệu và kiến trúc mở. Dịch vụ này ghi lại file và các đối tượng lên các drive và đảm bảo dữ liệu được sao lưu qua các cụm server.
Block Storage | Cinder | Cung cấp block lưu trữ cố định để chạy thực thể.
Identity service | Keystone | Cung cấp dịch vụ xác thực và phân quyền cho các services khác, cung cấp danh mục endpoint cho các dịch vụ khác 
Image service | Glance | Lưu trữ và cung cấp disk image cho các máy ảo. Các máy ảo sử dụng disk image trong quá trình khởi tạo.
Telemetry | Ceilometer | Kiểm soát và đo đạc đám mây OpenStack, sử dụng khi benckmark, thu phí, mở rộng.
Orchestration | Heat | Điều phối các ứng dụng trên môi trường OpenStack bằng cách sử dụng các định dạng mẫu.

Tùy thuộc vào nhu cầu sử dụng và khả năng phần cứng, có rất nhiều cấu hình triển khai hệ thống OpenStack. Bài hướng dẫn này sử dụng cấu hình đơn giản nhất của OpenStack:
![Physic-deploy.png](./img/physic-deploy.png)

Như ở sơ đồ trên, Controller node sẽ triển khai các dịch vụ liên quan tới quản lý hệ thống, còn compute node sẽ triển khai các dịch vụ để thiết lập máy ảo dựa trên tài nguyên vật lý.
2 node kết nối với nhau thông qua mạng quản lý Management network, mạng External network phục vụ cho việc truy cập internet và kết nối tới máy ảo.

Sau khi tìm hiểu xong về phần cấu hình cài đặt, chúng ta đi vào phần cài đặt chi tiết các dịch vụ vào hệ thống.
#Chuẩn bị môi trường cài đặt trên các node.
##2.1 Thiết lập môi trường phần cứng
Để thiết lập hệ thống, đầu tiên ta cần chuẩn bị môi trường.
Môi trường triển khai hệ thống là VMWare workstation 12 trên windows 10.
Sau khi cài đặt VMWare workstation, ta thiết lập cấu hình mạng như sau:

Thiết lập dải mạng cho external network:
 
![external-physic-network.png](./img/external-physic-network.png)

Thiết lập dải mạng cho management network:
![external-physic-network.png](./img/management-physic-network.png) 

Sau khi thiết lập cấu hình mạng, ta tiến hành tạo máy ảo controller và máy ảo compute với cấu hình như bên dưới, với 2 card mạng nối tới 2 mạng ta đã thiết lập bên trên:

![hardware-setting-vmware.png](./img/hardware-setting-vmware.png) 

##2.2 Chuẩn bị môi trường cho controller node
###2.2.1 Thiết lập địa chỉ mạng
Ta kiểm tra và chỉnh sửa sao cho eth0 nằm ở mạng Vmnet2(internal) và eth1 nằm ở mạng Vmnet8(external):
![hardware-setting-vmware.png](./img/setting-network-controller.png) 

Sau đó ta thiết lập hostname và địa chỉ tĩnh cho các card mạng:
Thiết lập hostname với tên là controller

```sh
	echo "controller" > /etc/hostname
	hostname -F /etc/hostname
```

Khởi động lại máy, sau đó thiết lập địa chỉ IP tĩnh cho eth0 và eth1. Chỉnh sửa file /etc/network/interfaces với nội dung sau:

```sh
	# NIC loopback
	auto lo
	iface lo inet loopback
	
	# NIC MGNG
	auto eth0
	iface eth0 inet static
	address 10.10.10.10
	netmask 255.255.255.0
	
	# NIC EXTERNAL
	auto eth1
	iface eth1 inet static
	address 192.168.2.10
	netmask 255.255.255.0
	gateway 192.168.2.1
	dns-nameservers 8.8.8.8
```





 
Chỉnh sửa file  /etc/hosts để phân giải IP cho các node:
```sh
	127.0.0.1   controller localhost
	10.10.10.10    controller
	10.10.10.11    compute
```
	
Khởi động lại máy tính.

###2.2.2 Cài đặt dịch vụ MySQL, message queue, Network Time Protocol, message queue, memcached và OpenStack Client
####Cài đặt OpenStack Client
OpenStack Client là services cho phép người dùng tương tác với hệ thống OpenStack thông qua các câu lệnh. Hiện tại OpenStack Client hỗ trợ các dịch vụ : keystone, image, object storage và compute

Sau khi khởi động lại, ta kích hoạt repository Openstack:
```sh
	# apt-get install software-properties-common
	# add-apt-repository cloud-archive:liberty
```

Sau đó cập nhật lại:
```sh
	# apt-get update && apt-get dist-upgrade
```

Sau đó ta cài đặt OpenStack client:
```sh
	# apt-get install python-openstackclient
```
####Cài đặt hệ quản trị cơ sở dữ liệu SQL

Sau khi cài đặt OpenStack client, chúng ta cần cài đặt cơ sở dữ liệu lên controller node, vì các dịch vụ của OpenStack sử dụng SQL để lưu trữ thông tin.
Ta cài đặt gói mariaDb:
```sh
	# apt-get install mariadb-server python-pymysql
```

Thiết lập mật khẩu: 1111

Tạo file /etc/mysql/conf.d/mysqld_openstack.cnf với nội dung sau:
```sh
	[mysqld]
	bind-address = 10.10.10.10
	
	[mysqld]
	default-storage-engine = innodb
	innodb_file_per_table
	collation-server = utf8_general_ci
	init-connect = 'SET NAMES utf8'
	character-set-server = utf8
```


khởi động lại service mysql:
```sh
	# service mysql restart
```
####Cài đặt dịch vụ Network Time Protocol
Network Time Protocol là dịch vụ cho phép đồng bộ hóa giữa các máy tính trong mạng thông qua sử dụng NTP để đồng bộ thời gian giữa các máy.

Để cài đặt dịch vụ NTP, ta cài đặt the packages chrony:
```sh
	# apt-get install chrony
```
Tiến hành chỉnh sửa file cấu hình ```sh/etc/chrony/chrony.conf```:
Thay các dòng dưới
```sh
	server 0.debian.pool.ntp.org offline minpoll 8
	server 1.debian.pool.ntp.org offline minpoll 8
	server 2.debian.pool.ntp.org offline minpoll 8
	server 3.debian.pool.ntp.org offline minpoll 8
```
bằng dòng
```sh
	server 1.vn.pool.ntp.org iburst
	server 0.asia.pool.ntp.org iburst
	server 3.asia.pool.ntp.org iburst
```
Khởi động lại dịch vụ :
```sh
	# service chrony restart
```
####Cài đặt dịch vụ Message queue
Dịch vụ Message queue giúp các services của hệ thống trao đổi các thông điệp với  nhau. Ở bản cài đặt này ta sử dụng dịch vụ RabbitMQ.

Ta cài đặt gói rabbitmq-server lên controller node:
```sh
	apt-get -y install rabbitmq-server
```

Cấu hình RabbitMQ, tạo user openstack với mật khẩu là 1111:
```sh
	rabbitmqctl add_user openstack 1111
```

Gán quyền read, write cho tài khoản openstack trong RabbitMQ
```sh
	rabbitmqctl set_permissions openstack ".*" ".*" ".*"
```

####Cài đặt dịch vụ Memcached
Dịch vụ xác thực sử dụng Memcached để làm bộ đệm lưu trữ các token, giúp cải thiện hiệu năng của hệ thống.

Ta cài đặt các gói cần thiết cho memcached
```sh
	apt-get -y install memcached python-memcache
```
Dùng nano  sửa ```sh file /etc/memcached.conf ```, thay dòng 
```sh -l 127.0.0.1 ``` 
bằng dòng dưới: 
```sh -l 10.10.10.10 ```
Trong đó 10.10.10.10 là địa chỉ nic management của controller node

Khởi động lại memcache
```sh 
	service memcached restart
``` 
##2.3 Chuẩn bị môi trường cho compute node
###2.3.1 Thiết lập địa chỉ mạng
Ta kiểm tra và chỉnh sửa sao cho eth0 nằm ở mạng Vmnet2(internal) và eth1 nằm ở mạng Vmnet8(external):
![setting-network-compute.png](./img/setting-network-compute.png) 

Sau đó ta thiết lập hostname và địa chỉ tĩnh cho các card mạng:

Thiết lập hostname với tên là compute
```sh
	echo "copmute" > /etc/hostname
	hostname -F /etc/hostname
```

Khởi động lại máy.

Sau đó thiết lập địa chỉ IP tĩnh cho eth0 và eth1:
Thiết lập địa chỉ IP, chỉnh sửa ```sh file /etc/network/interfaces ``` với nội dung sau:
```sh 
	# NIC loopback
	auto lo
	iface lo inet loopback
	
	# NIC MGNG
	auto eth0
	iface eth0 inet static
	address 10.10.10.11
	netmask 255.255.255.0
	
	# NIC EXTERNAL
	auto eth1
	iface eth1 inet static
	address 192.168.2.11
	netmask 255.255.255.0
	gateway 192.168.2.1
	dns-nameservers 8.8.8.8
```
 
Chỉnh sửa file  /etc/hosts để phân giải IP cho các node:
```sh 
	127.0.0.1   compute localhost
	10.10.10.10    controller
	10.10.10.11    compute
```
	
Khởi động lại máy tính.

###2.2.2 Cài đặt dịch vụ Network Time Protocol và OpenStack Client
####Cài đặt OpenStack Client
Sau khi khởi động lại, ta kích hoạt repository Openstack:
```sh
	# apt-get install software-properties-common
	# add-apt-repository cloud-archive:liberty
```
Sau đó cập nhật lại:
```sh
	#apt-get update && apt-get dist-upgrade
```
Sau đó ta cài đặt OpenStack client:
```sh
	#apt-get install python-openstackclient
```
###Cài đặt và cấu hình NTP trên Compute node
Ta cài đặt NTP Client
```sh
	apt-get -y install chrony
```
Chỉnh sửa file /etc/chrony/chrony.conf.Thay các dòng dưới
```sh
	server 0.debian.pool.ntp.org offline minpoll 8
	server 1.debian.pool.ntp.org offline minpoll 8
	server 2.debian.pool.ntp.org offline minpoll 8
	server 3.debian.pool.ntp.org offline minpoll 8
```
bằng dòng
```sh
	server controller iburst
```
Khởi động lại dịch vụ NTP
```sh
	service chrony restart
```
#Cài đặt Keystone
#3.1 Giới thiệu dịch vụ Keystone
Dịch vụ Keystone là dịch vụ xác thực trong OpenStack (identity service),có vai trò cung cấp các chức năng quản lý xác thực, phân quyền và quản lý danh mục các services cho toàn bộ hệ thống. Bên cạnh đó, dịch vụ này lưu trữ thông tin các người sử dụng trong hệ thống. Tất cả các dữ liệu liên quan tới dịch vụ không được lưu trữ trực tiếp trong hệ thống OpenStack, mà được lưu trữ trong một hệ cơ sở dữ liệu như MySQL.
###3.1.1 Mô hình xác thực của Keystone
Mô hình xác thực của Keystone được xây dựng dựa trên các khái niệm sau:

``` User```: Đối tượng đại diện cho một cá nhân, một dịch vụ hoặc một hệ thống sử dụng các dịch vụ của OpenStack. Dịch vụ xác thực xác minh các yêu cầu xem nó có đúng là do User gắn với yêu cầu đó thực hiện hay không. User cần đăng nhập và sử dụng các thẻ (token) được dịch vụ xác thực cấp sau khi đăng nhập để truy cập tới các tài nguyên.

``` Credentials ```: Thông tin xác thực, dùng để xác nhận danh tính của User. Thông tin xác thực có thể là tên tài khoản và mật khẩu, hoặc tên tài khoản và khóa API (API key) hoặc là thẻ xác thực (token).
``` Authentication```: Quá trình xác nhận danh tính của User. Dịch vụ xác thực xác nhận một yêu cầu là do user nào đó gửi tới bằng cách xác nhận các thông tin xác thực được user cung cấp. 
Khởi đầu, User sẽ gửi thông tin xác thực là tên đăng nhập và mật khẩu. Sau khi xác nhận danh tính của User, dịch vụ xác thực sẽ cấp cho User thẻ xác thực (token) để người dùng sử dụng làm thông tin xác thực ( thay cho tên đăng nhập và mật khẩu) ở các yêu cầu sau ( Để đảm bảo tính bảo mật).
 
``` Token```: Thẻ xác thực, là một chuỗi ký tự được sử dụng để giúp người dùng truy cập vào các dịch vụ của OpenStack và các tài nguyên. Token có thể bị thu hồi ở bất kỳ thời điểm nào và chỉ có giá trị trong một khoảng thời gian hữu hạn. 

``` Project ```: Đại diện cho một nhóm tài nguyên, một nhóm dịch vụ hoặc 1 nhóm người dùng.

``` Service```: Dịch vụ trong OpenStack, như là dịch vụ điện toán (compute), dịch vụ lưu trữ (storage), dịch vụ cung cấp ảnh tệp (image). Các dịch vụ cung cấp các cổng đầu cuối (endpoint), là địa chỉ mà người dùng có thể truy cập vào để sử dụng tài nguyên hoặc thực hiện các thao tác.

``` Endpoint```: Là địa chỉ trên mạng, nơi mà người dùng có thể truy cập tới các dịch vụ, thường là 1 địa chỉ URL.
Role: Quy định một tập các quyền mà người dùng có thể thực hiện trên hệ thống. Một User có thể có nhiều roles.Trong OpenStack, thì thẻ xác thực sẽ chứa luôn thông tin về các quyền mà 1 user có. Các dịch vụ dựa vào các quyền có trong thẻ xác thực để xác định xem người dùng có quyền thực hiện yêu cầu gửi tới dịch vụ đó hay không.

Để xây dựng hệ thống phân quyền và xác thực, 3 khái niệm ``` User```, ```Project``` và ```Role``` có các mối quan hệ với nhau như sau:

Một Project khi được tạo ra sẽ được định nghĩa như một tập hợp các tài nguyên. Nếu một user được thêm vào 1 Project, user đó sẽ có một hoặc nhiều quyền (Role) đối với project đó. Mỗi một quyền sẽ quy định user được thực hiện những thao tác gì hoặc được sử dụng những tài nguyên gì trong project tương ứng. Một user có thể thuộc nhiều Project, và ở mỗi một project khác nhau user đó có thể có các tập quyền khác nhau.

Ví dụ:
 
![user-role-project-relation.png](./img/user-role-project.png) 

Ta có sơ đồ như hình trên. Vì ở project A User A có các quyền A và B nên User A có thể truy cập được tất cả 5 tài nguyên C1, C2, C3, C4, C5, còn User B chỉ có thể truy cập vào được các tài nguyên C4, C5. Còn ở Project B thì User A chỉ có thể truy cập được vào các tài nguyên C1, C2, C3, còn User  B truy cập được vào tài nguyên C4.

Lưu ý: Quyền admin có tác dụng với toàn bộ các service, ví dụ nếu 1 user demo ở project demo có quyền là admin thì cũng có thể thực hiện các lệnh yêu cầu quyền admin (nghĩa là có quyền như user admin chúng ta tạo), mặc dù user đó được cấp quyền admin đối với project demo, hay nói cách khác quyền admin để truy cập vào các service không phụ thuộc vào việc nó được cấp ở project nào cho user nào.

Ngoài chức năng xác thực, dịch vụ keystone còn cung cấp danh mục các địa chỉ của các dịch vụ. Khi một người dùng hoặc 1 dịch vụ muốn sử dụng một dịch vụ nào đó, chỉ cần truy cập tới dịch vụ keystone để lấy địa chỉ của dịch vụ đó và gửi yêu cầu tới địa chỉ đó. Vì vậy, khi cài đặt một dịch vụ lên hệ thống OpenStack, người cài đặt cần phải tạo endpoint của dịch vụ lên và đưa endpoint vào Keystone.

###3.1.2 Ví dụ minh họa về luồng làm việc của dịch vụ KeyStone (Keystone workflow)
Để hiểu rõ hơn về cách thức làm việc của dịch vụ KeyStone, chúng ta xét một ví dụ minh họa như sau:

Hệ thống đang hoạt động có 3 thành phần chính: Dịch vụ Keystone, dịch vụ A và dịch vụ B. 2 dịch vụ A và B đều đã đăng ký endpoint ở dịch vụ Keystone, cũng như đều có các tài khoản của dịch vụ Keystone. Một khách hàng đã có tài khoản cần sử dụng dịch vụ A để thực hiện một yêu cầu. Quá trình thực hiện yêu cầu của khách hàng bao gồm cac giai đoạn sau:
![keystone-workflow.png](./img/keystone-workflow.png) 

1. User gửi cho Keystone yêu cầu tạo thẻ xác thực, đồng thời gửi tên tài khoản và mật khẩu về cho Keystone.
2. Keystone xác minh thông tin xác thực (tên tài khoản và mật khẩu ) của user.
3. Keystone tạo và gửi cho user thẻ xác thực (User token) có bao gồm thông tin về các quyền của User.

4. User yêu cầu keystone cung cấp endpoint của Service A.
5. Keystone gửi cho user endpoint của Service A.
6. Dựa trên endpoint mà keystone gửi tới, User gửi yêu cầu( kèm theo token) tới endpoint của Service A.
7. Service A yêu cầu Keystone xác minh xem token đi kèm với yêu cầu của user có chứa quyền (role) có khả năng thực hiện yêu cầu của user hay không.
8. Keystone xác minh quyền của token.
9. Keystone gửi cho service A kết quả xác minh token.
10. Service A thực hiện yêu cầu của user.
11. Để thực hiện yêu cầu của user, Service A cần phải thực hiện 1 thao tác ở service B. Service A cung cấp tên đăng nhập và mật khẩu của mình cho Keystone, và yêu cầu keystone cấp thẻ xác thực cho service A.
12. Keystone xác minh thông tin đăng nhập của service A và tạo, gửi thẻ xác thực cho service A.
13. Service A yêu cầu keystone cung cấp endpoint của service B.
14. Keystone gửi cho Service A endpoint của service B.
15. Service A yêu cầu service B thực hiện yêu cầu, cùng với token của service A.
16. Service B yêu cầu Keystone xác minh thẻ xác thực của service A xem service A có quyền thực hiện yêu cầu mà service A gửi tới service B hay không.
17. Keystone xác minh quyền của token của service A.
18. Keystone gửi kết quả xác thực cho service B. 
19. Service B thực hiện yêu cầu của service A.
20. Service B trả kết quả thực hiện cho service A.
21. Service A tiếp tục thực hiện yêu cầu của user.
22. Service A gửi kết quả thực hiện yêu cầu về cho user.

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




#Cài đặt dịch vụ mạng của OpenStack
##6.1 Giới thiệu chung về dịch vụ mạng của OpenStack
Một trong những yêu cầu quan trọng khi thiết lập một hệ thống đám mây là phải cung cấp cho các thành phần trong đám mây khả năng kết nối với nhau và kết nối ra bên ngoài, tức là cần thiết lập được một hệ thống mạng trong đám mây. Với đối tượng phục vụ kết nối mạng chính trong các đám mây là hệ thống máy ảo, đồng thời cùng với yêu cầu phải đảm bảo các tính chất của một đám mây- một hệ thống phân tán, OpenStack đã giải quyết bằng cách sử dụng một gói dịch vụ cho phép thiết lập một hệ thống mạng ảo trên đám mây. Gói dịch vụ cung cấp các dịch vụ mạng cho hệ thống OpenStack có tên là neutron.

Nhiệm vụ chính của dịch vụ Neutron là:

•	Tạo ra và quản lỷ các đối tượng và các thiết bị mạng ảo: router, switch, bridge, ip, subnet, network….

•	Sử dụng các thiết bị mạng ảo để xây dựng nên hệ thống mạng ảo, cung cấp dịch vụ mạng cho các dịch vụ và các đối tượng khác trong OpenStack.

•	Cung cấp các API cho phép người dùng thiết lập môi trường mạng cho và đánh địa chỉ cho môi trường mạng người đó thiết lập.

•	Cung cấp các dịch vụ liên quan tới mạng như cấp phát địa chỉ (DHCP), định tuyến (routing), DNS, cân bằng tải (Load-banlance)…

Chúng ta sẽ xem xét mô hình mạng ảo gồm những thành phần nào và được triển khai trên môi trường thực tế như thế nào.
##6.2 Mô hình hệ thống mạng ảo (virtual networking) trong OpenStack
###6.2.1 Các thành phần của hệ thống mạng ảo 
Để xây dựng nên hệ thống mạng ảo, chúng ta cần tạo ra các thiết bị mạng ảo tương ứng với các thiết bị mạng vật lý. Các thiết bị được ảo hóa là: Card mạng (Network Card Interfae –NIC), bridge, switch, router, DHCP, … Vai trò của từng thiết bị trong mạng cụ thể như sau:

•	NIC: Cổng kết nối internet, cung cấp kết nối mạng cho các máy ảo.

•	Bridge, switch: Kết nối các thiết bị  mạng và các máy ảo với nhau ở lớp Layer2 tạo thành các mạng cục bộ.

•	Router: Kết nối Các Switch với nhau để kết nối các mạng cục bộ với nhau và cung cấp kết nối internet cho các mạng cục bộ.

•	DHCP: Cấp phát địa chỉ IP cho các máy trong mạng.

Sau khi đã xác định được các thiết bị đã có trong mạng ảo, chúng ta sẽ sử dụng các thành phần này để triển khai trên các node tạo ra hạ tầng mạng. Có rất nhiều cách triển khai, ví dụ như sơ đồ sau là một cách triển khai hạ tầng trên network node:

![virtual-device-example.png](./img/virtual-device-example.png)

Sau khi triển khai xong hạ tầng mạng ảo, các user có thể sử dụng các dịch vụ mà neutron cung cấp để xây dựng nên các mạng ảo cho project của user đó và cung cấp kết nối mạng cho các máy ảo trong project.

Để hiểu rõ hơn về cách triển khai hạ tầng mạng ảo, chúng ta sẽ xây dựng nên một hạ tầng mạng ảo với một kiến trúc đơn giản trong số rất nhiều các kiến trúc có thể triển khai. Ở đây chúng ta sẽ triển khai một hệ thống mạng self service sử dụng linux-bridge làm L2 plugin, hệ thống này cho phép người dùng có thể tạo ra các mạng ảo riêng cho các project (private self-service network).
###6.2.2 Mô hình hệ thống mạng Self-service sử dụng linux-bridge
Hệ thống mạng self-service này được triển khai trên cấu hình vật lý đã giới thiệu ở đầu bài, gồm 1 node controller và một node compute. Sơ đồ triển khai hệ thống mạng như sau:

![self-service-architect1.png](./img/self-service-architect1.png)

Triển khai chi tiết trong compte node và controller node được thể hiện như sau:

Tại compute node:

![self-service-architect-compute-node.png](./img/self-service-architect-compute-node.png)

Tại controller node:

![self-service-architect-controller-node.png](./img/self-service-architect-controller-node.png)

Ở đây, chúng ta có thể thấy, một hệ thống mạng ảo được xây dựng dựa trên các thành phần thiết bị mạng ảo như Switch-Bridge, các card mạng (eth), router, dhcp,... Sau đó các thành phần thiết bị trên được triển khai trên các node tương ứng để cấu thành hệ thống mạng.

Ở trong mô hình hệ thống mạng self service, mỗi một mạng ảo (private network) do một user nào đó tạo ra đều có các thành phần được triển khai trên cả 2 node. Ví dụ, user tạo ra mạng khi mạng ảo net01,  1 bridge ảo được tạo ra trên controller node và 1 bridge ảo được tạo ra trên controller node. Bridge ảo trên controller node  và compute node sẽ được kết nối tới các card mạng vật lý, sao cho các card mạng vật lý này kết nối với nhau trong cùng một mạng. Như ở trong hình vẽ, thì cả card eth1 trong compute node và card eth1 trong controller node đều thuộc mạng vật lý private network. Nhờ đó, 2 bridge ảo được tạo ra sẽ kết nối với nhau tạo thành bộ khung cho mạng ảo chúng ta muốn tạo. Các bridge này kết nối với nhau bằng cách đánh địa chỉ VLAN, theo đó, thì mỗi 1 mạng ảo sẽ được triển khai trên 1 mạng VLAN, nhờ đó mà các mạng ảo này sẽ độc lập với nhau. Sau khi kết nối thành công các bridge được tạo ra trên các node với nhau, thì để cung cấp dịch vụ định tuyến và đánh địa chỉ cho mạng, một router ảo và một DHCP ảo được tạo ra trên controller node và kết nối với bridge ảo trên controller node. Tiếp đó, router sẽ đặt gateway lên mạng mạng provider network thông qua card mạng eth0 trên controller node. Mạng provider network kết nối với mạng internet, đo đó mạng ảo được tạo ra sẽ có thể kết nối được với internet thông qua router này.

Khi một máy ảo được tạo ra, người dùng sẽ phải xác định máy ảo đó sẽ thuộc mạng nào. Ví dụ ta muốn tạo ra một máy ảo VM01 thuộc mạng net 01 như trên hình vẽ. Khi đó service nova-compute sẽ tạo ra 1 card mạng ảo eth1 trên máy ảo VM01, card mạng eth1 này sau đó được kết nối với bridge brq trên compute node. Khi đó máy ảo VM01 sẽ kết nối được tới mạng ảo private net01.

Sau khi đã hiểu được mô hình triển khai của mạng self-service sử dụng linux bridge, chúng ta tiến hành cài đặt mô hình này trên hệ thống máy tính vật lý. Lưu ý là chúng ta còn rất nhiều cấu hình triển khai khác nữa, ví dụ như có thể triển khai riêng hệ thống neutron lên 1 network node, hoặc có thể sử dụng plugin OpenVSwitch thay cho linux-bridge, vv… Các bạn có thể tham khảo các cấu hình này trong các tài liệu tham khảo. 

##6.3 Cài đặt dịch vụ Neutron
###6.3.1 Các thành phần của dịch vụ Neutron
Dịch vụ Neutron bao gồm các thành phần sau:

-	neutron-server: Tiếp nhận các  API request từ user và chuyển các request tới các plugin để xử lý

-	neutron-plugin-ml2 and neutron-*-agent : Quản lý layer 2 (switch và bridge ảo) trong hệ thống mạng ảo.

-	neutron-l3-agent : Tạo ra và quản lý các router ảo.

-	neutron-dhcp-agent: Tạo và quản lý các thiết bị DHCP cho các mạng ảo.

-	neutron-metadata-agent: cung cấp kết nối giữa các máy ảo và dịch vụ nova-metadata service. Dịch vụ nova-metadata service có vai trò cung cấp các thông tin về máy ảo(instance metadata) cho máy ảo (Ví dụ như lúc máy ảo bắt đầu khởi động cần thông tin về số lõi, dung lượng RAM, địa chỉ IP,…. lúc này máy ảo sẽ yêu cầu dịch vụ nova-metadata cung cấp thông tin về máy ảo cho mình để cấu hình).

Khi triển khai cài đặt trên các máy vật lý, các dịch vụ sẽ được phân bố như sau:

-	Trên node controller:  neutron-server, neutron-plugin-ml2, neutron-linuxbridge-agent, neutron-l3-agent, neutron-dhcp-agent, neutron-metadata-agent.

-	Trên node compute: neutron-linuxbridge-agent
Ngoài các thành phần trên, Neutron còn có các thành phần khác như cơ sở dữ liệu, các endpoint và user trong Keystone Identity.

###6.3.2 Chuẩn bị cấu hình trên controller node và tải về các thành phần của neutron trên controller node

Để bắt đầu cài đặt trên controller node, ta cần tạo cơ sở dữ liệu cho Neutron:

####Tạo database, user và endpoint cho neutron.

- Đăng nhập vào MySQL
```sh
	mysql -uroot -p1111
```
- Tạo database và phân quyền
```sh
	CREATE DATABASE neutron;
	GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY '1111';
	GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY '1111';
		
	FLUSH PRIVILEGES;
	exit;
```
- Khai báo biến môi trường
```sh
source admin-openrc
```
- Tạo tài khoản tên ```sh neutron```, thêm tài khoản ```sh neutron``` vào project ```sh service``` với quyền của tài khoản ```sh neutron``` đối với project ```sh service``` là ```sh admin```
```sh
	openstack user create neutron --domain default --password 1111
	openstack role add --project service --user neutron admin
```
- Tạo dịch vụ tên là neutron
```sh
	openstack service create --name neutron --description "OpenStack Networking" network
```
- Tạo các endpoint cho dịch vụ neutron trong keystone
```sh	
	openstack endpoint create --region RegionOne network public http://controller:9696
	openstack endpoint create --region RegionOne network internal http://controller:9696	
	openstack endpoint create --region RegionOne network admin http://controller:9696
```
####Tải về các dịch vụ của neutron
Ta tiến hành tải về các dịch vụ của neutron trên controller node:
```sh
	apt-get -y install neutron-server neutron-plugin-ml2 \
	neutron-linuxbridge-agent neutron-l3-agent neutron-dhcp-agent \ neutron-metadata-agent
```
Tiếp theo, ta cấu hình các dịch vụ của neutron
###6.3.3 Cấu hình cài đặt neutron trên controller node
Đầu tiên, ta cấu hình file /etc/neutron/neutron.conf:
####Cấu hình để neutron sử dụng database
Chỉnh sửa section [database] để neutron có thể sử dụng database neutron mà chúng ta vừa tạo ở phần trước:
```sh 
	connection = mysql+pymysql://neutron:1111@controller/neutron
```
Lưu ý: xóa cơ sở dữ liệu mặc định của neutron, comment dòng này ở section [database]
```sh
	#connection = sqlite:////var/lib/neutron/neutron.sqlite
```
Cấu hình để nova kích hoạt ml2 plugin, router services và ovelaping  ip address:
```sh
	[DEFAULT]
	...
	core_plugin = ml2
	service_plugins = router
	allow_overlapping_ips = True
```
####Cấu hình để neutron sử dụng messaging service
Neutron liên lạc với các dịch vụ khác thông qua messaging service. Cập nhật section [DEFAULT] và section [oslo_messaging_rabbit] để cấu hình giúp neutron sử dụng messaging service:
```sh
	[DEFAULT]
	...
	rpc_backend = rabbit
```
Phần xác thực cho rabbit_mq phải khớp với các thông tin ta thiết lập khi cài đặt messaging service ở phần trước đó:
```sh
	[oslo_messaging_rabbit]
	...
	rabbit_host = controller
	rabbit_userid = openstack
	rabbit_password = 1111
```
####Cấu hình để neutron sử dụng dịch vụ xác thực Keystone
Để hệ thống mạng neutron hoạt động, cần cấp quyền admin cho dịch vụ neutron để neutron có thể sử dụng được các dịch vụ khác khi hoạt động. 

Chỉnh sửa section [DEFAULT] để thiết lập keystone là phương thức xác thực cho neutron:
```sh
	[DEFAULT]
	...
	auth_strategy = keystone
```

Cập nhật section [keystone_authtoken] để gán user neutron mà ta mới tạo ở phần trước cho neutron services, neutron service sẽ sử dụng user này khi xác thực với keystone:
```sh
	[keystone_authtoken]
	...
	auth_uri = http://controller:5000
	auth_url = http://controller:35357
	auth_plugin = password
	project_domain_id = default
	user_domain_id = default
	project_name = service
	username = neutron
	password = 1111
```

####Cấu hình neutron để thông báo các sự kiện cho nova

Neutron cần thông báo cho Nova khi cấu hình mạng (network topology) thay đổi. Cập nhật các section [DEFAULT] và [nova] 
```sh
	[DEFAULT]
	...
	notify_nova_on_port_status_changes = True
	notify_nova_on_port_data_changes = True
	[nova]
	...
	auth_url = http://controller:35357
	auth_type = password
	project_domain_name = default
	user_domain_name = default
	region_name = RegionOne
	project_name = service
	username = nova
	password = 1111

```

####Note: Viết 1 bài viết về ML2 và các plugin (Linux-bridge, OpenVswitch) của nó.

####Cấu hình Modular Layer 2 (ML2) plug-in
Trong bài hướng dẫn này, chúng ta sử dụng ML2 plugin kết hợp với Linux-bridge-agent để xây dựng nên Layer2 cho mạng ảo. Để cấu hình ML2 plug-in, tiến hành cập nhật file ```/etc/neutron/plugins/ml2/ml2_conf.ini```
- Cập nhật section [ml2], option ``` type_driver``` cho phép chọn các loại mạng có thể được tạo ra và hoạt động bởi mechanism driver. Vì chúng ta sử dụng linux-bridge, chúng ta chọn các mạng flat, lan, vxlan
```sh
[ml2]
	...
	type_drivers = flat,vlan,vxlan
```

option ```mechanism_drivers``` cho phép thiết lập các plugin kết hợp với ml2 plugin. Ở đây ta thiết lập giá trị cho option này là linuxbridge và l2population
```sh
	[ml2]
	...
	mechanism_drivers = linuxbridge,l2population
```

option ```tenant_network_types``` cho phép thiết lập các loại mạng mà các user có thể tạo trong các project. Ở đây ta thiết lập loại mạng mà các user có thể tạo là ```vxlan```
```sh
	[ml2]
	...
	tenant_network_types = vxlan
```
kích hoạt port_security:
```sh
	[ml2]
	...
	extension_drivers = port_security
```
- Cập nhật section [ml2_type_flat], xác định interfaces vật lý hỗ trợ thiết lập mạng ảo flat. Ở đây chúng ta thiết lập giá chị cho option ```flat_networks``` là một ```provider label```. ```provider label``` là một nhãn logic được gán vào 1 interface vật lý. nhãn logic này sẽ được xác định sẽ gán vào interface vật lý nào khi chúng ta cấu hình linux-bridge. Ở đây chúng ta sử dụng tên nhãn logic là ```provider```. 
```sh
	[ml2_type_flat]
	...
	flat_networks = provider
```
- Cập nhật section [ml2_type_vxlan], xác định khoảng giá trị có thể dùng để gán ID cho các VXLAN. Như đã cấu hình ở option ```tenant_network_type```, các private network do user tạo ra sẽ có kiểu là vxlan. Để phân biệt được các máy thuộc các mạng ảo( tức các VXLAN ) khác nhau, thì mỗi VXLAN sẽ có 1 id riêng. Ở đây ta thiết lập giá trị cho option ```vni_ranges``` là ```1:1000```, tức là các VXLAN được tạo ra trong hệ thống mạng ảo sẽ nhận ID có giá trị từ 1 tới 1000.
```sh
	[ml2_type_vxlan]
	...
	vni_ranges = 1:1000
```
- Cập nhật section [securitygroup], cải thiện hiệu quả hoạt động của tường lủa bằng cách enable option ```enable_ipset```
```sh
	[securitygroup]
	...
	enable_ipset = True
```
####Cấu hình Linux bridge agent
Để cấu hình Linux bridge agent, chúng ta chỉnh sửa file ```/etc/neutron/plugins/ml2/linuxbridge_agent.ini```
- Cập nhật section [linux_bridge], xác định ánh xạ giữa interface vật lý với tên nhãn logic:
```sh
	[linux_bridge]
	physical_interface_mappings = provider:eth1
```
- Ở đây, chúng ta ánh xạ nhãn logic ```provider``` với card vật lý eth1. Cấu hình này có liên quan tới việc chúng ta thiết lập mạng flat ở phần cấu hình trước trên ml2 plugin. Khi chúng ta thiết lập cấu hình này, thông qua nhãn logic provider, chúng ta có thể triển khai một mạng flat network ảo trên các card mạng vật lý eth1. Ở các phần sau, mạng flat network này sẽ là provider external network, cung cấp kết nối internet cho các private network thông qua các router. Chúng ta triển khai mạng flat network này trên các card mạng eth1, vì các card eth1 này kết nối tới mạng internet bên ngoài.
- Cập nhật section [vxlan], kích hoạt option enable_vxlan để linux_bridge hỗ trợ VXLAN, kích hoạt l2_population option và ánh xạ ```local_ip``` sang địa chỉ ip của card mạng sẽ triển khai vxlan network. Ở cấu hình đang cài đặt, chúng ta sẽ triển khai các VXLAN (các private network của các user) thông qua mạng vật lý management network. Do vậy, ở từng node chúng ta sẽ cấu hình giá trị của option này tương ứng với các ip của mạng này trên các node. Ở trên controller node, management network có 1 ip 10.10.10.10 trên card eth0. Do đó chúng ta sẽ sử dụng giá trị này gán cho option ```local_ip```
```sh
	[vxlan]
	enable_vxlan = True
	local_ip = 10.10.10.10
	l2_population = True
```

- Cập nhật section [securitygroup], để kích hoạt firewall trên linux_bridge
####firewall này triển khai trên bridge nào ?
```sh
	[securitygroup]
	...
	enable_security_group = True
	firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```

####Cấu hình Neutron DHCP agent
Dịch vụ ```neutron-dhcp-agent``` có chức năng tạo ra, quản lý, cấu hình và cung cấp các thông tin metadata về ```dnsmasq```, một loại DHCP ảo có chức năng cung cấp dịch vụ DHCP cho mạng ảo.

Để cấu hình dịch vụ ```neutron-dhcp-agent```, chỉnh sửa file ```/etc/neutron/dhcp_agent.ini```, section [DEFAULT], chỉnh sửa option interface-driver sử dụng Linux-bridge, dhcp_drive sử dụng dnsmasq, và kích hoạt option enable_isolated_metadata để dhcp có thể đóng vai trò cung cấp metadata cho instance:
```sh
	[DEFAULT]
	...
	interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver
	dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
	enable_isolated_metadata = True
```
####Cấu hình L3 agent
Dịch vụ ```neutron-l3-agent``` có chức năng tạo ra và quản lý các router ảo trên hệ thống mạng ảo. Cấu hình file ```/etc/neutron/l3_agent.ini```, thiết lập interface_driver của các router sử dụng Linux-bridge và để trống giá trị external_network_bridge.
```sh
	[DEFAULT]
	...
	interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver
	external_network_bridge =
```
####Cấu hình neutron metadata agent
Metadata agent có chức năng cung cấp các dữ liệu cho máy ảo khi máy ảo cần. Cấu hình metadata agent bằng cách chỉnh sửa file: ``` /etc/neutron/metadata_agent.ini ``` , thiết lập địa chỉ của metadata server là controller và mật khẩu để truy cập vào metadata server là 1111 trong section [DEFAULT]
```sh 
[DEFAULT]
...
nova_metadata_ip = controller
metadata_proxy_shared_secret = 1111
```

####Cấu hình nova để sử dụng neutron và metadata agent.
Để nova sử dụng neutron services để quản lý mạng cho các máy ảo, cần cấu hình lại dịch vụ nova.Chỉnh sửa file ```	/etc/nova/nova.conf```, cập nhật các section sau để cung cấp cho nova endpoint, thông tin xác thực của neutron services và thông tin về metadata service:
```sh
	[neutron]
	...
	url = http://controller:9696
	auth_url = http://controller:35357
	auth_type = password
	project_domain_name = default
	user_domain_name = default
	region_name = RegionOne
	project_name = service
	username = neutron
	password = NEUTRON_PASS
	
	service_metadata_proxy = True
	metadata_proxy_shared_secret = 1111	
```
###6.3.4 Kết thúc cài đặt trên controller node
- Đồng bộ hóa cơ sở dữ liệu cho neutron
```sh
	su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
	--config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron 
```
- Khởi động lại dịch vụ nova-api và các dịch vụ trong neutron
```sh
	service nova-api restart
	service neutron-server restart
	service neutron-linuxbridge-agent restart
	service neutron-dhcp-agent restart
	service neutron-metadata-agent restart
	service neutron-l3-agent restart
```
###6.3.5 Chuẩn bị các thành phần của neutron trên compute node
Trên compute node, ta sẽ triển khai thành phần neutron-linuxbridge-agent. Tải về neutron-linuxbridge-agent:
```sh
	apt-get install neutron-linuxbridge-agent
```

###6.3.6 Cấu hình neutron trên compute node
Ta cấu hình file /etc/neutron/neutron.conf:
####Cấu hình để neutron sử dụng messaging service
Neutron liên lạc với các dịch vụ khác thông qua messaging service. Cập nhật section [DEFAULT] và section [oslo_messaging_rabbit] để cấu hình giúp neutron sử dụng messaging service:
```sh
	[DEFAULT]
	...
	rpc_backend = rabbit
```
Phần xác thực cho rabbit_mq phải khớp với các thông tin ta thiết lập khi cài đặt messaging service ở phần trước đó:
```sh
	[oslo_messaging_rabbit]
	...
	rabbit_host = controller
	rabbit_userid = openstack
	rabbit_password = 1111
```
####Cấu hình để neutron sử dụng dịch vụ xác thực Keystone
Để hệ thống mạng neutron hoạt động, cần cấp quyền admin cho dịch vụ neutron để neutron có thể sử dụng được các dịch vụ khác khi hoạt động. 

Chỉnh sửa section [DEFAULT] để thiết lập keystone là phương thức xác thực cho neutron:
```sh
	[DEFAULT]
	...
	auth_strategy = keystone
```

Cập nhật section [keystone_authtoken] để gán user neutron mà ta mới tạo ở phần trước cho neutron services, neutron service sẽ sử dụng user này khi xác thực với keystone:
```sh
	[keystone_authtoken]
	...
	auth_uri = http://controller:5000
	auth_url = http://controller:35357
	memcached_servers = controller:11211
	auth_type = password
	project_domain_name = default
	user_domain_name = default
	project_name = service
	username = neutron
	password = 1111
```
####Cấu hình linux-bridge agent
- Cấu hình linux-bridge agent trên compute node để chuẩn bị hạ tầng mạng ảo trên compute node. Chỉnh sửa file ```/etc/neutron/plugins/ml2/linuxbridge_agent.ini ```, cấu hình để mapping - ánh xạ nhãn ```provider``` vào card vật lý eth1. Khi chúng ta muốn các máy ảo trên compute node có khả năng kết nối trực tiếp vào mạng external network, khi đó các máy ảo này sẽ kết nối vào mạng ảo flat này thông qua 1 bridge kết nối tới eth1.
```sh
	[linux_bridge]
	physical_interface_mappings = provider:eth1
```
- Cấu hình cho vxlan tương tự như controller node
```sh
	[vxlan]
	enable_vxlan = True
	local_ip = 10.10.10.10
	l2_population = True
```
ở đây local_ip là 10.10.10.11 là địa chỉ của card mạng vật lý kết nối tới mạng management network, mà chúng ta sẽ triển khai mạng vxlan trên mạng vât lý này.
- Kích hoạt security group trên compute node
```sh
	[securitygroup]
	...
	enable_security_group = True
	firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```
####Cấu hình nova-compute trên computenode để nova-compute sử dụng neutron.
Chỉnh sửa file cấu hình ```/etc/nova/nova.conf``` để nova-compute có thể sử dụng neutron, thêm thông tin xác thực của neutron vào file cấu hình của nova-compute.
```sh
	[neutron]
	...
	url = http://controller:9696
	auth_url = http://controller:35357
	auth_type = password
	project_domain_name = default
	user_domain_name = default
	region_name = RegionOne
	project_name = service
	username = neutron
	password = 1111
```
###6.3.7 Kết thúc cài đặt trên compute node
- Khởi động lại dịch vụ nova-compute
```sh
	service nova-compute restart
```
- Khởi động lại linux-bridge agent
```sh
	service neutron-linuxbridge-agent restart
```
##6.4 Kiểm tra hoạt động của dịch vụ neutron
Trên controller node, nhập file xác thực admin.sh
```sh
	source admin.sh
```
Kiểm tra xem các agent đã được bật đầy đủ hay chưa 
```sh
	neutron agent-list
```
Kết quả nếu như hệ thống hoạt động bình thường
```sh
	+--------------------------------------+--------------------+------------+-------+----------------+---------------------------+
	| id                                   | agent_type         | host       | alive | admin_state_up | binary                    |
	+--------------------------------------+--------------------+------------+-------+----------------+---------------------------+
	| 08905043-5010-4b87-bba5-aedb1956e27a | Linux bridge agent | compute1   | :-)   | True           | neutron-linuxbridge-agent |
	| 27eee952-a748-467b-bf71-941e89846a92 | Linux bridge agent | controller | :-)   | True           | neutron-linuxbridge-agent |
	| 830344ff-dc36-4956-84f4-067af667a0dc | L3 agent           | controller | :-)   | True           | neutron-l3-agent          |
	| dd3644c9-1a3a-435a-9282-eb306b4b0391 | DHCP agent         | controller | :-)   | True           | neutron-dhcp-agent        |
	| f49a4b81-afd6-4b3d-b923-66c8f0517099 | Metadata agent     | controller | :-)   | True           | neutron-metadata-agent    |
	+--------------------------------------+--------------------+------------+-------+----------------+---------------------------+
```
