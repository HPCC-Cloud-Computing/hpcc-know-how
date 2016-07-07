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

#3.Dịch vụ Keystone
<h2><a name="tong_quan">3.1	Giới thiệu tổng quan</a></h2>
&emsp;Keystone là dịch vụ xác thực của OpenStack – cung cấp quyền truy cập đảm bảo đã được kiểm soát đến tài nguyên của một cloud. Trong môi trường OpenStack, Keystone thực hiện nhiều chức năng quan trọng, như xác thực người dùng và xác định những tài liệu nào người dùng được quyền truy cập đến.</br>
&emsp;Để hiểu rõ cách Keystone cung cấp truy cập an toàn và được kiểm soát đến các tài nguyên, chúng ta cần xem xét các tính năng cơ bản của Keystone:</br>
<b>Indentity</b></br>
&emsp;Identity là định danh của người đang truy cập các tài nguyên cloud. Trong OpenStack keystone, identity thường đại diện cho một người dùng (user). Trong triển khai đơn giản nhất, identity của user có thể được lưu trữ trong database của Keystone. Nhưng trong môi trường sản xuất hay trong môi trường doanh nghiệp, một Identity Provider bên ngoài thường được sử dụng. Ví dụ như Manager Identity Federated Tivoli của IBM. Keystone có thể lấy ra thông tin định danh của người dùng từ các Provider Identity bên ngoài này.</br>
<b>Authentication</b></br>
&emsp;Authentication là quá trình của việc xác minh một thông tin định danh của người dùng. Một định danh người dùng đầu tiên sẽ được xác thực thông qua một password, sau đó các thông tin này sẽ được sử dụng để tạo ra một token để sử dụng các lần xác thực tiếp theo. Việc này sẽ làm giảm số lượng lần hiển thị và tiếp xúc với password (password được ẩn giấu và bảo vệ các nhiều càng tốt). Token cũng có giá trị trong một thời gian giới hạn (Việc này làm giảm khả năng sử dụng của nó trong trường hợp bị trộm). Trong OpenStack, Keystone là dịch vụ duy nhất có thể tạo ra token. </br>
&emsp;Hiện nay, Keystone sử dụng một dạng token được gọi là bearer token. Cụ thể, bất kỳ ai nắm quyền sở hữu token, cho dù là đúng hay là sai (vd: kẻ trộm,…) thì đều có khả năng sử dụng thẻ token để xác thực và truy cập đến các tài nguyên. Do đó, khi sử dung Keystone thì việc bảo vệ token là rất quan trọng.</br>
<b>Access Management (Authorication)</b></br>
&emsp;Access Management hay Authorication là quá trình xác định những tài nguyên nào một người dùng có thể truy cập.</br>
<b>các tính năng của Keystone</b></br>
&emsp;Trong khi Keystone tập trung hầu hết vào cung cấp các dịch vụ Identity, Authentication và Access Management, nó cũng hỗ trợ một số tính năng khác cho môi trường OpenStack:</br>
<ul>
<li>Xác thực đơn và cấp quyền cho các dịch vụ khác của OpenStack</li>
<li>Xử lý các hệ thống xác thực ngoài và cô lập tất cả các dịch vụ trong OpenStack</li>
<li>Keystone cung cấp các Project giúp các dịch vụ khác của OpenStack có thể sử dụng để phân biệt các tài nguyên (ví dụ như các máy chủ, các image,…).</li>
<li>Keystone cung cấp các Domain được sử dụng để định nghĩa name-space của các user, group và project để cho phép phân biệt giữa các người dùng</li>
<li>Cung cấp việc đăng ký các Roles được sử dụng cho việc cấp phép giữa Keystone và file policy của mỗi dịch vụ OpenStack</li>
<li>Cung cấp một catalog lưu trữ các dịch vụ OpenStack, endpoint và region, cho phép client có thể tìm kiếm các dịch vụ hoặc các endpoint mà họ cần.</li>
</ul>
<h3><a name="tq_khaiNiem">3.1.1	Các khái niệm </a></h3></br>
Để có thể tìm hiểu Keystone, trước tiên cần phải hiểu các khái niệm được sử dụng trong Keystone.</br>
<h3><a name="khaiNiem_project">3.1.1.1	 Project </a></h3></br>
&emsp;Trong Keystone, Project là khái niệm trừu tượng được sử dụng bởi các dịch vụ OpenStack để nhóm và phân biệt các tài nguyên (vd: các server, các image,…)</br>
&emsp;Trước đây, các Project Keystone sử dụng khái niệm “tenant”, nhưng do trực quan hơn nên đã đổi sang “Project”. Khái niệm này phù hợp hơn để nói về mục tiêu cơ bản nhất của Keystone là cung cấp các project và trình bày ai là được phép truy cập đến các project này.</br>
&emsp;Người dùng và nhóm người dùng nhận quyền truy cập đến một project sử dụng một khái niệm là Role Assignment (phân công). Một Role được gán cho một user hoặc user group trong một Project cho biết user hoặc user group có một số quyền truy cập đến các tài nguyên trong Project.</br>
&emsp;Ví dụ:</br>
-	Một người có thể thuộc nhiều project và có các quyền khác nhau.</br>
-	“Alice” có quyền admin trong project “abc” nhưng chỉ có quyền user trong project “xyz”</br>
<h3><a name="khaiNiem_domain">3.1.1.2	 Domain</a></h3></br>
&emsp; Trong giao đoạn đầu của OpenStack, không có cơ chế để hạn chế tầm nhìn của các project trong các tổ chức người dùng khác nhau. Điều này dẫn đến nhưng va chạm về tên Project của các tổ chức khác nhau. Tên user cũng có tầm nhìn tổng thể nên cũng có thể dẫn đến những va chạm không mong muốn khi 2 tổ chức đều có chung một tên user. Keystone đã đưa ra khái niệm là Domain.</br>
&emsp; Domain cung cấp khả năng để cô lập tầm nhìn của một tập các Project và user (và group) trong một tổ chức cụ thể. Một Domain được định nghĩa như là một tập hợp bao gồm các user, các group và các project.</br>
<h3><a name="khaiNiem_user_group">3.1.1.3	 User và User Group (Actor)</a></h3></br>
&emsp;Trong Keystone, user và user group là các thực thể được cấp quyền truy cập đến các tài nguyên trong các project. Group là một tập hợp của nhiều user. User là đại diện cho một người, hay một dịch vụ sử dụng các dịch vụ OpenStack và được chứa trong một Domain cụ thể.</br>
&emsp;Mối quan hệ của Domain, project, user và group được thể hiện trong hình sau:</br>
<img src="img/relationship.png"/><br><br>
&emsp;Các user, group và project luôn luôn nằm trong phạm vi của Domain. Điều này dẫn đến, một tên của user hay groups hay project có thể được sử dụng trên các Domain khác nhau. Ví dụ như user “Alice” hoặc groups “Adminstrator” đều có ở cả IBM và Acme Inc.</br>
<h3><a name="khaiNiem_role">3.1.1.4	 Role</a></h3></br>
&emsp;Role được sử dụng trong Keystone để xác định những quyền mà môt user hay group có thể thực hiện với một tài nguyên OpenStack.</br>
&emsp;Ví dụ, Role admin được gán cho user “Alice” đối với project “Development”. Khi đó, user “Alice” sẽ thực hiện được các hành động của quyền admin đối với các tài nguyên của project “Development”</br>
<h3><a name="khaiNiem_token">3.1.1.5	 Token</a></h3></br>
&emsp;Khi người dùng gọi đến bất kỳ một API OpenStack thì họ phải chứng minh được rằng họ là ai và họ phải được cho phép để gọi đến các API. Thực hiện việc này bằng cách truyền một token OpenStack vào trong lời gọi API. Keystone là dịch vụ duy nhất chịu trách nhiệm tạo ra token. Người dùng nhận một token khi xác thực thành công với Keystone. Token chứa quyền của một user trong cloud. Nó bao gồm một ID và một payload. ID đảm bảo tính duy nhất của token trên mỗi cloud và payload chứa dữ liệu về người dùng.</br>
&emsp;Ví dụ: một token sẽ chứa nhiều trường thể hiện các thuộc tính Identity và Authorization về một user trong một project.</br>
 <img src="img/exTokenV3.png"/><br><br>
&emsp;Token trên chứa những thông tin sau: Khi nào token được tạo ra; Khi nào token hết hạn, user nào đã được xác thực- do đó, được phép sử dụng token này; xác định project nào token này có giá trị; Và cuối cùng là Catalog chứa các dịch vụ trong OpenStack.</br>
<h3><a name="khaiNiem_catalog">3.1.1.6	 Catalog</a></h3></br>
&emsp;Catalog là danh mục chứa các URL và các endpoint của các dịch vụ OpenStack khác nhau. Nó rất cần thiết cho OpenStack, thiếu catalog thì các user, và các ứng dụng sẽ không biết địa chỉ để gửi request tạo một máy ảo hoặc là lưu trữ một đối tượng.</br>
&emsp;Catalog được chứa các endpoint và mỗi endpoints chứa một admin URL, enternal URL và public URL.</br>
&emsp;Ví dụ về một Catalog chỉ chứa hai dịch vụ Object Storage và Identity</br>
  <img src="img/exCatolog.png"/><br><br>
&emsp;Trên đây là các khái niệm trong OpenStack, tiếp theo sẽ đi vào chi tiết các tính năng của Keystone.</br>
<h3><a name="tq_identity">3.1.2	Identity</a></h3></br>
&emsp;Dịch vụ Identity trong Keystone cung cấp các Actor (user hay group). Identity trong Openstack có thể đến từ các vị trí khác nhau như SQL, LDAP hay Federated Identity Provider.</br>
<h3><a name="iden_sql">3.1.2.1 SQL</a></h3></br>
&emsp;Keystone hỗ trợ các database để lưu trữ các actor như MySQL, PostgreSQL hay DB2. Keystone sẽ lưu trữ các thông tin của người dùng như tên, password và mô tả. </br>
&emsp;Về cơ bản, Keystone hoạt động như một Identity Provider như chắc chắn rằng sẽ không phải là một giải pháp tốt ưu trong một số trường hợp như trong môi trường doanh nghiệp.</br>
&emsp;Các ưu, nhược điểm khi sử dụng SQL:</br>
<ul>
    <li>Ưu điểm:
        <ul>
            <li>Dễ dàng cài đặt</li>
            <li>Quản lý user và group thông qua các API OpenStack</li>
        </ul>
    </li>
    <li>Nhược điểm:
        <ul>
            <li>Keystone không phải là một Identity Provider</li>
            <li>Hỗ trợ password yếu: Không luân chuyển password và không khôi phục password</li>
            <li>Hầu hết các doanh nghiệp sử dụng LDAP</li>
        </ul>
    </li>
</ul>
<h3><a name="iden_ldap">3.1.2.2	 LDAP</a></h3></br>	
&emsp;Keystone cũng có một tùy chọn để lưu trữ và lấy dữ liệu actor trong Lightweight Directory Access Protocol (LDAP). Keystone sẽ truy cập đến LDAP như các ứng dụng khác sử dụng LDAP (như Email, Web application,…). Thiết lập kết nối đến LDAP trong file cấu hình Keystone. </br>	
&emsp;LDAP thường chỉ thực hiện quyền đọc, chẳng hạn như tìm kiếm user (thông qua search) và xác thực (thông qua bind).</br>	
&emsp;Ưu, nhược điểm khi sử dụng LDAP:</br>	
<ul>
    <li>Ưu điểm:
        <ul>
            <li>Không duy trì bản sao của tài khoản user</li>
            <li>Keystone hoạt động như Identity Provider (nhà cung cấp nhận dạng).</li>
        </ul>
    </li>
    <li>Nhược điểm:
        <ul>
            <li>Các tài khoản của các dịch vụ khác trong OpenStack cũng cần được lưu trữ ở đâu đó, và admin LDAP không muốn lưu các tài khoản này trong LDAP</li>
            <li>Keystone vẫn thấy được password của user, khi password nằm trong một yêu cầu xác thực. Keystone đơn giản chỉ chuyển các yêu cầu, vì vậy không cần thiết phải biết password của user</li>
        </ul>
    </li>
</ul>
<h3><a name="iden_backends">3.1.2.3 Multiple Backend</a></h3></br>
&emsp;Keystone hỗ trợ sử dụng multiple backend, nghĩa là mỗi domain có thể sử dụng một backend</br>
&emsp;Ví dụ: Trong ví dụ sau, Identity sử dụng LDAP cho Domain A và Domain B, một SQL để lưu trữ các service account và các Assignment</br>
<img src="img/multiple_backend.png"/><br><br>
&emsp;Ưu, nhược điểm khi sử dụng multiple backend:</br>
<ul>
    <li>Ưu điểm:
        <ul>
            <li>Hỗ trợ đồng thời nhiều backend</li>
            <li>Có thể sử dụng lại các LDAP đã có</li>
        </ul>
    </li>
    <li>Nhược điểm:
        <ul>
            <li>Cài đặt khá phức tạp</li>
            <li>Xác thực người dùng bắt buộc phải nằm trong phạm vi Domain</li>
        </ul>
    </li>
</ul>
<h3><a name="iden_provider">3.1.2.4 Identity Provider</a></h3></br>
&emsp;Từ bản IceHouse, keystone hỗ trợ federate authentication thông qua Apache Module với nhiều Identity Provider. User không lưu trữ ở keystone. Identity Provider là các nguồn lưu trữ thông tin về Identity được lưu bởi các loại backend như Ldap, AD, MongoDB hoặc Social Login (FB, Twitter).</br>
&emsp;Ưu, nhược điểm khi sử dụng identity provider:</br>
<ul>
    <li>Ưu điểm:
        <ul>
            <li>Có thể tích hợp với các hệ thống có sẵn để xác thực người dùng và lấy các thông tin người dùng sẵn có</li>
            <li>Có sự tách biệt giữa keystone và xử lý thông tin định danh.</li>
            <li>Mở ra khả năng single sign-on và hibrid cloud.</li>
            <li>Keystone không thể xem được password của user (an toàn).</li>
            <li>Identity Provider xử lý tất cả các công việc xác thực, vì vậy, việc sử dụng kiểu xác thực nào (password, cert, two-factor,…) không liên quan đến keystone</li>
        </ul>
    </li>
    <li>Nhược điểm:
        <ul>
            <li>Phải cài đặt phức tạp</li>
        </ul>
    </li>
</ul>
<h3><a name="iden_usecase_backend">3.1.2.5	Các trường hợp sử dụng các Identity backend</a></h3></br>
&emsp;Tùy từng trường hợp cụ thể, có thể lựa chọn sử dụng các identity backend phù hợp.
<ul>
    <li>SQL:
        <ul>
            <li>Thử nghiệm hoặc phát triển OpenStack</li>
            <li>Với một tập hợp ít user</li>
            <li>Có các service account.</li>
        </ul>
    </li>
    <li>LDAP:
        <ul>
            <li>Trong môi trường doanh nghiệp</li>
            <li>Chỉ sử dụng nếu có khả năng tạo service account cần thiết trong LDAP</li>
        </ul>
    </li>
     <li>Multiple Backend:
        <ul>
            <li>Hầu hết được sử dụng trong các doanh nghiệp</li>
            <li>Sử dụng trong trường hợp LDAP không cho phép service account</li>
        </ul>
    </li>
     <li>Identity Provider:
        <ul>
            <li>Muốn tận dụng cơ chế mới Federated Identity</li>
            <li>Keystone không thể truy cập được LDAP</li>
        </ul>
    </li>
</ul>
<h3><a name="tq_Authentication">3.1.3	Authentication </a></h3></br>
&emsp;Keystone sử dụng hai cách để xác thực là cung cấp password hoặc sử dụng một token.</br>
<h3><a name="authen_pass">3.1.3.1	Password</a></h3></br>
&emsp;Cách phổ biến là các user hay service có thể xác thực là cung cấp một password. Ví dụ dưới đây là payload request đến keystone. Nó thể hiện tất cả thông tin cần thiết cho việc xác thực bao gồm thông tin user và scope của user đó</br>
  <img src="img/password.png"/><br><br>
&emsp;Phần scope là không bắt buộc,nhưng nó thường xuyên được dùng. Bởi vì, thiếu scope user sẽ không lấy được một catalog các service. Scope được sử dụng để chỉ ra những project mà người dùng muốn làm việc. Nếu người dùng không có role trong project này, thì yêu cầu bị hủy bỏ. Cũng giống như phần user, phần scope cũng phải có đầy đủ thông tin về project mà user muốn tìm. Việc này để tránh trương hợp va chạm tên Project.</br>
<h3><a name="authen_token">3.1.3.2	Token</a></h3></br>
&emsp;Tương tự như password, user yêu cầu một token mới bằng cách cung cấp token hiện tại. Dưới đây là một ví dụ về một request token mới</br> 
 <img src="img/exToken.png"/><br><br>
 <h3><a name="tq_author">3.1.4	Access Management và Authorization </a></h3></br>
&emsp;KeyStone tạo ra một chính sách Role-Based Access Control (RBAC), thực thi tại mỗi public API endpoint. Các chính sách được lưu trong file policy.json, nó bao gồm mục tiêu và quy tắc. Mỗi luật bắt đầu với " identity:" </br>
<img src="img/password_diagram.png"/><br><br>
<h3><a name="tq_format">3.1.5	Các định dạng Token </a></h3></br>
<h3><a name="format_uuid">3.1.5.1	UUID </a></h3></br>
&emsp;Trong những ngày đầu tiên, định dạng token của Keystone là UUID format. UUID format đơn giản chỉ lại một chuỗi 32 ký tự được sinh ngẫu nhiên. Token này được cung cấp và được xác thực bởi dịch vụ identity. </br>
&emsp;Lợi ích của Token format này là token nhỏ (ngắn) và dễ sử dụng, đủ đơn giản để có thể thêm vào trong các lệnh cURL. </br>
&emsp;Tuy nhiên nhược điểm của nó là không mang đủ những thông tin để có thể xác thực một cách trực tiếp các dữ liệu và request. Các dịch vụ của openstack cứ liên tục phải gửi lại token về phía keystone server để xác thực các request đến các service đó. Dẫn đến bất kì hành động nào trong Openstack đều phải thông qua keystone server.</br>
<b>UUID Token Generation Workflow</b></br>
<img src="img/uuid-token_generation.png"/><br><br>
&emsp;Khi user gửi yều cầu tạo token đến Keystone với các thông tin Username, password và Project Name, Keystone sẽ thực hiện các bước sau để tạo ra UUID Token</br>
<ol>
    <li>Xác nhận user và lấy user ID</li>
    <li>Xác nhận project và lấy project ID và domain ID</li>
    <li>Lấy các Role của user trên project hoặc domain. Trả về failure, nếu user không có quyền nào.</li>
    <li>Lấy các service và endpoint của tất các các service trong OpenStack</li>
    <li>Đóng gói các thông tin Identity, Resource, Assignment và Catalog vào payload token và tạo Token ID</li>
&emsp;Lưu giữ các thông tin Token ID, Expiration, Valid, User ID, Extra vào backend.</br>
</ol>
<b>UUID Token Validation Workflow</b></br>
<img src="img/uuid-token_validation.png"/><br><br>
&emsp;Quá trình xác minh UUID Token như sau:</br>
<ol>
    <li>Sử dụng câu lệnh “GET v3/auth/tokens” để gửi token đến Keystone </li>
    <li>Lấy payload Token từ backend và kiểm tra các giá trị có đúng hay không? Nếu sai “Token Not Found”, Nếu đúng thực hiện bước tiếp theo.</li>
    <li>Phân tích token và lấy metadata của token đó (bao gồm User ID, Project ID, Audit  ID, và Token Expiry).</li>
    <li>Kiểm tra xem token đã hết hạn hay chưa (sử dung thời gian hiện tại được tính toàn trong UTC)? Nếu đã quá hạn (thời gian hiện tại lớn hơn thời gian hết hạn) thì báo “Token Not Found”, nếu đúng thực hiện bước tiếp theo.</li>
    <li>Kiểm tra xem Token đã bị thu hồi hay chưa? Nếu đã bị thu hồi, thì trả về “Token Not Found”. Nếu đúng, trả về “HTTP/1.1 200 OK” (có nghĩa là token sử dụng được)</li>
</ol>
<b>UUID Token Revocation Workflow</b></br> 
<img src="img/uuid - token_revocation.png"/><br><br>
&emsp;Quá trình thu hồi một token thực hiện như sau:</br>
<ol>
    <li>Gửi request “DELETE v3/auth/tokens” đến Keystone để yêu cầu thu hồi token. Trước khi thu hồi token, thì phải thực hiện xác minh token (thực hiện các bước như trên)</li>
    <li>Kiểm tra xem có Audit ID không? Nếu không chuyển sang bước 3. Nếu có thì chuyển sang bước 6</li>
    <li>Token được thu hồi khi hết hạn.</li>
    <li>Tạo event thu hồi với các thông tin sau: User ID, Project ID, Revoke At, Issued Before và Token Expiry</li>
    <li>Chuyển đến bước 8.</li>
    <li>Thu hồi bởi Audit ID.</li>
    <li>Tạo event thu hồi với các thông tin: Audit ID, Revoke At, Issued Before</li>
    <li>Lọc các event revoke đang tồn tại dựa trên revoke at</li>
    <li>Cài đặt giá trị false cho token kvs</li>
</ol>
<h3><a name="format_pki">3.1.5.2	PKI – PKIZ</a></h3></br>
&emsp;Fomat token thứ 2 mà Keystone hỗ trợ là PKI format. Trong format này, token sẽ chứa toàn bộ thông tin xác minh chẳng hạn như: Khi nào token được cung cấp, khi nào token hết hạn, định danh người dùng, project, domain, role và service catalog. Tất cả những thông tin này được đóng gói trong một payload.</br>
&emsp;Muốn gửi token qua HTTP, JSON token payload phải được mã hóa base64 với 1 số chỉnh sửa nhỏ. Cụ thể, Format=CMS+[zlib] + base64. Ban đầu JSON payload phải được ký sử dụng một khóa bất đối xứng(private key), sau đó được đóng gói trong CMS (cryptographic message syntax - cú pháp thông điệp mật mã). Với PKIz format, sau khi đóng dấu, payload được nén lại sử dụng trình nén zlib. Tiếp đó PKI token được mã hóa base64 và tạo ra một URL an toàn để gửi token đi.</br>
&emsp;Các OpenStack services cache lại token này để đưa ra quyết định ủy quyền mà không phải liên hệ lại keystone mỗi lần có yêu cầu ủy quyền dịch vụ cho user.</br>
&emsp;Kích thước của 1 token cơ bản với single endpoint trong catalog lên tới 1700 bytes. Với các hệ thống triển khai lớn nhiều endpoint và dịch vụ, kích thước của PKI token có thể vượt quá kích thước giới hạn cho phép của HTTP header trên hầu hết các webserver(8KB). Thực tế khi sử dụng chuẩn token PKIz đã nén lại nhưng kích thước giảm không đáng kể (khoảng 10%).</br>
&emsp;PKI và PKIz tokens tuy rằng có thể cached nhưng chúng có nhiều hạn chế</br>
<ul>
    <li>Khó cấu hình để sử dụng</li>
    <li>Kích thước quá lớn làm giảm hiệu suất web</li>
    <li>Khó khăn khi sử dụng trong cURL command.</li>
    <li>Keystone phải lưu các token với rất nhiều thông tin trong backend database với nhiều mục đích, chẳng hạn như tạo danh sách các token đã bị thu hồi. Hệ quả là người dùng phải lo về việc phải flush Keystone token database định kì tránh ảnh hưởng hiệu suất.</li>
</ul>
<b>Token PKI/PKIZ Generation Workflow</b></br>
<img src="img/pki-token_generation.png"/><br><br>
<ol>
    <li>User gửi request đến Keystone.</li>
    <li>Xác minh identity, resource (project và domain), assignment</li>
    <li>Tạo JSON Token Payload</li>
    <li>“Ký" lên JSON payload với Signing Key và Signing Certificate , sau đó được đóng gói lại dưới định dang CMS (cryptographic message syntax - cú pháp thông điệp mật mã)</li>
    <li>Bước tiếp theo, nếu muốn đóng gói token định dạng PKI thì convert payload sang UTF-8, convert token sang một URL định dạng an toàn. Nếu muốn token đóng gói dưới định dang PKIz, thì phải nén token sử dụng zlib, tiến hành mã hóa base64 token tạo ra URL an toàn, convert sang UTF-8 và chèn thêm tiếp đầu ngữ "PKIZ"</li>
    <li>Lưu vào Backend</li>
</ol>
<b>Token PKI/PKIZ Validation Workflow</b></br>
&emsp;Tương tự như UUID, chỉ khác ở bước gửi yêu cầu, Keystone sẽ băm lại PKI Token với thuật toán băm đã cấu hình trước. Các bước tiếp theo tương tự UUID Token</br>
<b>Token PKI/PKIZ Revocation Workflow</b></br>
&emsp;Tương tự như UUID.</br>
<h3><a name="format_fernet">3.1.5.3	Fernet</a></h3></br>
&emsp;Format mới nhất mà Keystone hỗ trợ là Fernet format. Fernet token chứa tối đa 255 ký tự, lớn hơn UUID và bé hơn rất nhiều so với PKI. Token này chỉ chứa đầy đủ thông tin của một token để không cần phải lưu trong database.</br>
&emsp;Fernet token cũng được “Ký” sử dụng khóa đối xứng để ngăn chặn giả mạo. Và các khóa này cần phải được phân bổ đến các OpenStack region khác nhau.</br>
<b>Fernet key</b></br>
&emsp;Một key reponsitory được yêu cầu bởi Keystone để lưu trữ các Key dùng để mã hóa và giải mã các thông tin của token. Mỗi Key trong reponsitory có 3 trạng thái:</br>
<ul>
    <li>Primary key:  Một key reponsitory chỉ có duy nhất một primary key. Sử dụng để mã hóa và giải mã token. Key này luôn đặt tên là index cao nhất trong key reponsitory</li>
    <li>Secondary key: là một thời điểm của primary key, nhưng nó đã bị giảm vị trí của một primary key khác. Key này chỉ được dùng để giải mã token. Keystone sữ dụng để giải mã các token được mã hóa bởi các primary key cũ.</li>
    <li>Staged key: Là một Key đặc biệt, có một số điểm tương đồng với Secondary key. Một key reponsitory chỉ có duy nhất một primary key. Nó giống với Secondary key là chỉ có khả năng giải mã token. Staged key sẽ là primary key trong lên chuyển khóa tiếp theo. Nó được đặt tên là index 0 trong key reponsitory.</li>
</ul>
&emsp;Fernet key có một chu kỳ, được bắt đầu từ Staged key, sau đó được nâng lên Primary key và cuối cùng giảm xuống Secondary key. Fernet key được lưu trữ trong file “/etc/keystone/fernet-keys”</br>
<b>Fernet key rotation</b></br>
<img src="img/fernet_key_rotation.png"/><br><br>
&emsp;Quá trình rotate key cụ thể như sau:</br>
<ul>
    <li>Ban đầu: Key reponsitory sẽ có Staged key và primary key. Không có secondary key.</li>
    <li>Rotate lần đầu tiên: Staged key sẽ trở thành primary key. Primary key chuyển thành secondary key. Một Staged key mới được tạo ra</li>
    <li>Rotate lầu thứ 2: Staged key Staged key sẽ trở thành primary key. Primary key chuyển thành secondary key thứ 2 trong key reponsitory. Một Staged key mới được tạo ra.</li>
    <li>Tương tự như vậy cho các lần tiếp theo.</li>
</ul>
&emsp;Vậy thì key reponsitory sẽ bị quá tải khi số lượng secondary key tăng lên? Không, key reponsitory sẽ có số lượng key cố định, khi vượt mức thì nó sẽ xóa secondary key xuất hiện sớm nhất.</br>
<b>Token Generation Workflow</b></br>
<img src="img/fernet-token_generation.png"/><br><br>
&emsp;Sau khi nhận được yêu cầu tạo token từ user Keystone sẽ thực hiện quá trình tạo token:</br>
&emsp;Như hình trên, token bao gồm các trường sau:</br>
<ol>
    <li>Fernet token version: cho biết phiên bản của định dạng token </li>
    <li>Current Timestamp : nhãn thời gian hiện tại, chỉ ra thời điểm token được tạo ra</li>
    <li>IV – Initialation : sử dụng để mã hóa và giãi mã </li>
    <li>Cipher text: là sự kết hợp của token payload (với các thông tin như version, user ID, Methods, Project ID,…) và Padding message. Sau đó, encrypt sử dụng encryption key được cung cấp</li>
    <li>HMAC: là tổng hợp tất cả các trường trên và “ký” sử dụng signing key mà user cung cấp </li>
</ol>
<b>Token Validation Workflow</b></br>
<img src="img/fernet-token_validation.png"/><br><br>
&emsp;Quá trình validation token cụ thể như sau:</br>
<ol>
    <li>User gửi yêu cầu validation đến keystone với phương thức: GET v3/auth/tokens</li>
    <li>Khôi phục lại Padding, trả về token với padding chính xác</li>
    <li>Giải mã token sử dụng fernet key để lấy token payload</li>
    <li>Xác định version từ token payload. Version cố định bởi keystone (unscope payload: 0; domain scope payload: 1; project scope payload: 2).</li>
    <li>Phân tích payload để xác minh các trường. ví dụ, trong phạm vi một project có các trường: user ID, project ID, Methods, Audit ID, expiry</li>
    <li>Kiểm tra xem token đã hết hạn chưa? Nếu đã hết hạn (thời gian hiện tại lớn hơn thời gian hết hạn), thì trả về “Token Not Found”. Nếu chưa hết hạn, tiếp tục bước sau.</li>
    <li>Kiểm tra token đã bị thu hồi hay chưa? Nếu đã bị thu hồi, trả về “Token not found”. Nếu đúng, trả về “HTTP/1.1 200 OK” token đã được validation</li>
</ol>
<b>Token Validation Workflow</b></br>
&emsp;Tương tự UUID và PKI/ PKIZ </br>
<b>Multiple Data Center</b></br>
<img src="img/fernet-multiple_data_center.jpg"/><br><br>
&emsp;Giả sử triển khai hệ thống cloud với keystone ở cả hai bên US-WEST và US-EAST. Cả hai bên này đều có LDAP và Database luôn luôn được đồng bộ. Hoạt động của keystone khi user muốn thực hiện tạo một máy ảo VM như sau:</br>
<ul>
    <li>User sử dụng Nova ở US-WEST:
        <ul>
            <li>Đầu tiên, user sẽ yêu cầu token đến keystone. Keystone sẽ tạo ra một fernet token cho user (mô tả trong Token Generation Workflow). Trả lại user fernet token</li>
            <li>User gửi yều cầu tạo máy ảo đến Nova và fernet token của mình. </li>
            <li>Nova thực hiện validation token bằng cách gửi yêu cầu validation token đến cho Keystone. Keystone thực hiện validation token (các bước cụ thể mô tả ở trên). Keystone gửi kết quả lại cho Nova </li>
            <li>Nếu xác minh đúng, Nova thực hiện yêu cầu tạo máy ảo và gửi phản hồi cho user</li>
        </ul>
    </li>
    <li>
        <ul>
            <li>User gửi yều cầu tạo máy ảo đến Nova và fernet token của mình (đã được cấp bởi keystone bên US-WEST). </li>
            <li>Nova thực hiện validation token bằng cách gửi yêu cầu validation token đến cho Keystone. Keystone thực hiện validation token (các bước cụ thể mô tả ở trên). Keystone gửi kết quả lại cho Nova </li>
            <li>Nếu xác minh đúng, Nova thực hiện yêu cầu tạo máy ảo và gửi phản hồi cho user</li>
        </ul>
    </li>
</ul>
<h3><a name="tq_activity">3.1.6	Hoạt động của Keystone</a></h3></br>
&emsp;Biểu đồ tuẩn tự dưới đây sẽ mô tả quá trình hoạt động của Keystone trong ví dụ cụ thể là tạo một máy ảo VM.</br>
<img src="img/keystone_process_flow.png"/><br><br>
&emsp;Cụ thể các bước như sau :</br>
<ol>
    <li>User gửi thông tin username và password đến keystone.</li>
    <li>Keystone kiểm tra thông tin. Nếu thông tin đúng (xác thực thành công), thì gửi một token lại cho user</li>
    <li>User gửi token và yêu cầu đến Nova.</li>
    <li>Nova gửi token (của user gửi đến) cho Keystone để yêu cầu xác thực có đúng không? Có những quyền hạn gì? Và Keystone sẽ đáp trả lại cho Nova</li>
    <li>Nếu đúng, Nova token (của nova) và yêu cầu image đến cho Glance.</li>
    <li>Glance gửi token (của Nova gửi đến) cho Keystone để xác thực và kiểm tra quyền đối với file image? Keystone đáp trả lại cho Glance.</li>
    <li>Glance gửi image lại cho Nova.</li>
    <li>Nova gửi token và yêu cầu cung cấp mạng đến Neutron.</li>
    <li>Neutron gửi token (của Nova gửi đến) cho Keystone để xác thực và kiểm tra quyền của Nova. Keystone đáp trả lại cho Neutron.</li>
    <li>Neutron phản hồi lại cho Nova.</li>
    <li>Nova thực hiện tiếp quá trình và gửi phản hồi lại cho user </li>
</ol>
<h3><a name="install_config">3.2 Cấu hình và cài đặt Keystone</a></h3>
- Trước tiên, cần phải tạo ra một database cho dịch vụ keystone bằng các câu lệnh sau:
```sh
mysql -u root –pWelcome123
CREATE DATABASE keystone;

GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
IDENTIFIED BY 'Welcome123';

GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
IDENTIFIED BY 'Welcome123';
flush privileges;
exit;
```
- Cấu hình không cho dịch vụ keystone tự động khởi động:
```sh
echo "manual" > /etc/init/keystone.override
```
- Chạy lệnh sau để cài đặt các gói của keystone:
```sh
apt-get install keystone apache2 libapache2-mod-wsgi
```
- Cấu hinh file `/etc/keystone/keystone.conf` với các yêu cầu sau:
 - Trong phần `[default]`, định nghĩa giá trị của thẻ quản trị ban đầu:
  ```sh
  [DEFAULT]
  ...
  admin_token = Welcome123
  ```
 - Trong phần database, cấu hình truy cập đến database:
  ```sh
  [database]
  ...
  connection = mysql+pymysql://keystone:Welcome123@controller/keystone
  ```
  - Trong phần [token], cấu hình nhà cung cấp thẻ Fernet:
  ```sh  
  [token]
  ...
  provider = fernet
  ```
- Đồng bộ database dịch vụ xác thực:
  ```sh
  su -s /bin/sh -c "keystone-manage db_sync" keystone
  ```
- Thiết lập Fernet key:
  ```sh
  keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
  ```
<h3>cấu hình máy chủ Apache</h3>
- Chỉnh sửa file `/etc/apache2/apache2.conf` và cấu hình tùy chọn Servername để ánh xạ đến node controller:
```sh
ServerName controller
```
- Tạo file `/etc/apache2/sites-available/wsgi-keystone.conf` với nội dung sau:
```sh
Listen 5000
Listen 35357

<VirtualHost *:5000>
    WSGIDaemonProcess keystone-public processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-public
    WSGIScriptAlias / /usr/bin/keystone-wsgi-public
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    ErrorLogFormat "%{cu}t %M"
    ErrorLog /var/log/apache2/keystone.log
    CustomLog /var/log/apache2/keystone_access.log combined

    <Directory /usr/bin>
        Require all granted
    </Directory>
</VirtualHost>

<VirtualHost *:35357>
    WSGIDaemonProcess keystone-admin processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
WSGIProcessGroup keystone-admin
    WSGIScriptAlias / /usr/bin/keystone-wsgi-admin
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    ErrorLogFormat "%{cu}t %M"
    ErrorLog /var/log/apache2/keystone.log
    CustomLog /var/log/apache2/keystone_access.log combined

    <Directory /usr/bin>
        Require all granted
    </Directory>
</VirtualHost>
```
- Cấp phép các máy ảo dịch vụ xác thực:
```sh
ln -s /etc/apache2/sites-available/wsgi-keystone.conf /etc/apache2/sites-enabled
```
- Khởi động lại apache:
```sh
service apache2 restart
```
- Xóa database mặc định của keystone:
```sh
rm -f /var/lib/keystone/keystone.db
```
<h3>Tạo endpoint và các service cho keystone</h3>
- Vì ban đầu, database của Keystone không chứa thông tin xác thực và catalog sevices nên để tạo được các endpoint và các service thì phải có một token để cho phép thực hiện bước này. 
- Truyền vào các biến môi trường để khởi tạo service và các endpoint indentity:
```sh
export OS_TOKEN=ADMIN_TOKEN
```
ADMIN_TOKEN ở đây là giá trị đã được khai báo trong file cấu hình ở bước trước.
- Khai báo URL endpoint và version API identity:
```sh
export OS_URL=http://controller:35357/v3
export OS_IDENTITY_API_VERSION=3
```
- Keystone quản lý một catalog các dịch vụ trong môi trường OpenStack. Các dịch vụ sử dụng catalog này để xác định các dịch vụ khác đang có trong môi trường.
- Tạo service cho dịch vụ identity:
```sh
openstack service create \
  --name keystone --description "OpenStack Identity" identity
```
- Keystone cũng quản lý một danh mục các endpoint API được kết nối với các dịch vụ trong môi trường OpenStack . Các dịch vụ sử dụng catalog này để xác định các giao tiếp với các dịch vụ khác trong môi trường OpenStack.
- Tạo các endpoints API: 
 - Public API endpoint:
 ```sh
 openstack endpoint create --region RegionOne \
 identity public http://controller:5000/v3
 ```
 - Internal API endpoint:
 ```sh
 openstack endpoint create --region RegionOne \
 identity internal http://controller:5000/v3
 ```
 - Admin API endpoint:
 ```sh
 openstack endpoint create --region RegionOne \
  identity public http://controller:5000/v3
 ```
<h3>Tạo domain, user, project và role</h3>
- Tạo domain `default`:
```sh
openstack domain create --description "Default Domain" default
```
- Tạo project `admin`:
```sh
openstack project create --domain default \
  --description "Admin Project" admin
```	
- Tạo user `admin`:
```sh
openstack user create --domain default \
  --password-prompt admin
```
- Và tạo role `admin`:
```sh
openstack role create admin
```	
- Gán role admin cho user `admin` đối với project `admin`:
```sh
openstack role add --project admin --user admin admin
```

<h3>Kiểm tra hoạt động</h3>
- Vì lý do bảo mật, vô hiệu hóa cơ chế thẻ token tạm thời bằng cách chỉnh sửa trong file `/etc/keystone/keystone-paste.ini`, xóa các dòng `admin_token_auth` từ các phần `[pipeline:public_api]`,`[pipeline:admin_api]` và `[pipeline:api_v3]`
- Gỡ bỏ các biến môi trường đã thiết lập trong quá trình tạo service và endpoint cho dịch vụ Identity.
- Kiểm tra hoạt động bằng cách yêu cầu token cho user “admin” đã tạo ở trên:
```sh
  openstack --os-auth-url http://controller:35357/v3 \
   --os-project-domain-name default --os-user-domain-name default \
   --os-project-name admin --os-username admin token issue
  Password:
```
  Nhập password vào và hiển thị kết quả như sau:
```sh
+------------+-----------------------------------------------------------------+
| Field      | Value                                                           |
+------------+-----------------------------------------------------------------+
| expires    | 2016-02-12T20:14:07.056119Z                                     |
| id         | gAAAAABWvi7_B8kKQD9wdXac8MoZiQldmjEO643d-e_j-XXq9AmIegIbA7UHGPv |
|            | atnN21qtOMjCFWX7BReJEQnVOAj3nclRQgAYRsfSU_MrsuWb4EDtnjU7HEpoBb4 |
|            | o6ozsA_NmFWEpLeKy0uNn_WeKbAhYygrsmQGA49dclHVnz-OMVLiyM9ws       |
| project_id | 343d245e850143a096806dfaefa9afdc                                |
| user_id    | ac3377633149401296f6c0d92d79dc16                                |
+------------+-----------------------------------------------------------------+
```
- Ngoài ra, để tăng hiệu quả của hoạt động ở phía client khi thực hiện yêu cầu token thì OpenStack hỗ trợ một kịch bản môi trường client
- Tạo kịch bản môi trường client cho user “admin” bằng cách tạo file `admin-openrc` với nội dung sau:
```sh
export OS_PROJECT_DOMAIN_NAME=default
export OS_USER_DOMAIN_NAME=default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=ADMIN_PASS
export OS_AUTH_URL=http://controller:35357/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```
- Sau khi tạo xong, thực hiện yêu cầu token đối với user “admin” sẽ thực hiện các bước sau:
- Chạy scripts vừa tạo :
```sh
. admin-openrc
```
- Yêu cầu token và thu được kết quả sau:
```sh
openstack token issue
+------------+-----------------------------------------------------------------+
| Field      | Value                                                           |
+------------+-----------------------------------------------------------------+
| expires    | 2016-02-12T20:44:35.659723Z                                     |
| id         | gAAAAABWvjYj-Zjfg8WXFaQnUd1DMYTBVrKw4h3fIagi5NoEmh21U72SrRv2trl |
|            | JWFYhLi2_uPR31Igf6A8mH2Rw9kv_bxNo1jbLNPLGzW_u5FC7InFqx0yYtTwa1e |
|            | eq2b0f6-18KZyQhs7F3teAta143kJEWuNEYET-y7u29y0be1_64KYkM7E       |
| project_id | 343d245e850143a096806dfaefa9afdc                                |
| user_id    | ac3377633149401296f6c0d92d79dc16                                |
+------------+-----------------------------------------------------------------+
```

#Cài đặt Glance
<h2><a name="overview">4.1. Tổng quan về OpenStack Glance</a></h2>
<h3><a name="glossary">4.1.1. Một số thuật ngữ</a></h3>
<ul> 
<li><b>Image:</b> một tập hợp các file chứa một hệ điều hành cụ thể đã được đóng gói sẵn, có thể sử dụng nó để tạo mới hoặc xây dựng lại một máy ảo.
</li>
<li><b>Metadata:</b> là dạng dữ liệu mô tả về dữ liệu. Trong cơ sở dữ liệu, metadata là các dạng biểu diễn khác nhau của các đối tượng trong cơ sở dữ liệu.
</li>
<li><b>VM (virtual machine):</b> là một môi trường phần mềm cho phép một hoặc hơn một HĐH và các ứng dụng của chúng hoạt động song song trên chỉ một máy tính duy nhất.
</li>
<li><b>Middleware:</b> là phần mềm máy tính với nhiệm vụ kết nối các thành phần phần mềm hoặc các ứng dụng với nhau.
</li>
</ul>
<h3><a name="intro">4.1.2 Giới thiệu về Glance</a></h3>
Một lợi thế của việc sử dụng dịch vụ điện toán đám mây thay vì máy chủ vật lí là vì chúng sử dụng dễ dàng hơn. Một khi bạn đã xây dựng được một môi trường máy ảo, bạn có thể sao chép và sử dụng nó bất cứ khi nào bạn muốn. Thậm chí ngay cả khi môi trường máy ảo có sự cố, bạn vẫn có thể build lại nó nếu bạn đã sao lưu máy chủ ảo của bạn. Ở đây, các file dữ liệu được tạo ra để build một máy ảo được gọi là <b>image</b>.
</br></br>
Trong OpenStack, Nova cung cấp các tính năng để tạo ra image từ các máy ảo hiện có và ngược lại. Tuy nhiên, một dịch vụ khác được gọi là Glance có các tính năng để quản lí các images.
<ul> 
<li>OpenStack Glance là một Image service cung cấp việc tìm kiếm, đăng kí, thu thập các images của các máy ảo. Glance cung cấp RESTful API cho phép truy vấn metadata của image máy ảo cũng như thu thập image thực sự.
</li>
<li>Glance đã được thiết kế là một dịch vụ độc lập cho những người cần phải tổ chức tập hợp lớn các hình ảnh đĩa ảo. Tuy nhiên, khi được sử dụng cùng với Nova và Swift, nó cung cấp một giải pháp end-to-end cho quản lý đĩa hình ảnh đám mây.
<br>
<img src="./img/glance+nova+swift.PNG"/>
</li>
<li>Trong Glance, các images được sử dụng để vận hành máy ảo mới. Nó cũng có thể lấy bản snapshots từ các máy ảo đang chạy để thực hiện dự phòng cho các VM và trạng thái các máy ảo đó.
</li>
</ul>
<h3><a name="component">4.1.3 Các thành phần của Glance</a></h3>

Glance bao gồm các thành phần sau:
<ul>
<li><b>glance-api: </b>tiếp nhận lời gọi API để tìm kiếm, thu thập và lưu trữ image</li>
<li><b>glance-registry: </b>thực hiện tác vụ lưu trữ, xử lý và thu thập metadata của images</li>
<li><b>database: </b>cơ sở dữ liệu lưu trữ metadata của image</li>
<li><b>image-store: </b>địa điểm lưu trữ các image</li>
</ul>
<img src="./img/component_glance.png"/><br><br>
Glance tiếp nhận các API request yêu cầu images từ người dùng đầu cuối hoặc các nova component và có thể lưu trữ các file images trong hệ thống object storage Swift hoặc các storage repos khác. Glance hỗ trợ các hệ thống backend lưu trữ sau:
<ul>
<li><b>File system: </b>
<div>lưu trữ, xóa, và nhận được hình ảnh từ một thư mục hệ thống tập tin được quy định trong file cấu hình, hỗ trợ đọc ghi các image files dễ dàng vào hệ thống tập tin.</div>
</li>
<li><b>HTTP: </b>
<div>Glance có thể đọc các images của các máy ảo sẵn sàng trên Internet thông qua HTTP. Hệ thống lưu trữ này chỉ cho phép đọc.</div>
</li>
<li><b>Object Storage: </b>
<div>là hệ thống lưu trữ do OpenStack Swift cung cấp - dịch vụ lưu trữ có tính sẵn sàng cao , lưu trữ các image dưới dạng các object.</div>
</li>
<li><b>BlockStorage: </b>hệ thống lưu trữ có tính sẵn sàng cao do OpenStack Cinder cung cấp, lưu trữ các image dưới dạng khối</li>
<li><b>VMWare</b></li>
<li><b>Amazon S3</b></li>
<li><b>RADOS Block Device(RBD): </b>
<div>lưu trữ các images trong cụm lưu trữ Ceph sử dụng giao diện RBD của Ceph</div>
</li>
<li><b>Sheepdog: </b>
<div>hệ thống lưu trữ phân tán dành cho QEMU/KVM</div>
</li>
<li><b>GridFS: </b>
lưu trữ các image sử dụng MongoDB
</li>
</ul>
<br>
<h2><b>Luồng điều khiển của Glance</b></h2>
<br>
<img src="./img/flow_control.png"/><br><br>
Khi người dùng nhận một image từ Glance, nó yêu cầu Glance database để lấy metadata của image bao gồm cả vị trí lưu trữ image. Khi đó Glance có thể gửi cho người dùng image mà họ muốn.
<h3><a name="arch">4.1.4 Kiến trúc của Glance</a></h3>
Glance có kiến trúc client-server và cung cấp REST API thông qua đó yêu cầu tới server được thực hiện. Yêu cầu từ client được tiếp nhận thông qua REST API và đợi sự xác thực của Keystone. Keystone Domain controller quản lý tất cả các tác vụ vận hành bên trong. Các tác vụ này chia thành các lớp, mỗi lớp triển khai nhiệm vụ vụ riêng của chúng.
<br>
Glance store driver là lớp giao tiếp giữa glane và các hệ thống backend bên ngoài hoặc hệ thống tệp tin cục bộ, cung cấp giao diện chung để truy cập. Glance sử dụng SQL Database làm điểm truy cập cho các thành phần khác trong hệ thống.
<br>
Kiến trúc Glance bao gồm các thành phần sau:
<ul>
<li><b>Client: </b>ứng dụng sử dụng Glance server</li>
<li><b>REST API: </b>gửi yêu cầu tới Glance thông qua REST</li>
<li><b>Database Abstraction Layer (DAL): </b>là một API thống nhất việc giao tiếp giữa Glance và databases</li>
<li><b>Glance Domain Controller: </b>là middleware triển khai các chức năng chính của Glance: ủy quyền, thông báo, các chính sách, kết nối cơ sở dữ liệu</li>
<li><b>Glance Store: </b>tổ chức việc tương tác giữa Glance và các hệ thống lưu trữ dữ liệu</li>
<li><b>Registry Layer: </b>lớp tùy chọn tổ chức việc giao tiếp một cách bảo mật giữa domain và DAL nhờ việc sử dụng một dịch vụ riêng biệt</li>
</ul>
<img src="./img/architecture.png"/><br>
<h3><a name="formats">4.1.5 Các định dạng lưu trữ image của Glance</a></h3>
<h3>Disk Formats</h3>
Là định dạng của các disk image
<table>
<tr>
<td>Disk Format</td>
<td>Notes</td>
</tr>

<tr>
<td>Raw</td>
<td>Định dạng đĩa phi cấu trúc</td>
</tr>

<tr>
<td>VHD</td>
<td>Định dạng chung hỗ trợ bởi nhiều công nghệ ảo hóa trong OpenStack, ngoại trừ KVM</td>
</tr>

<tr>
<td>VMDK</td>
<td>Định dạng hỗ trợ bởi VMWare</td>
</tr>

<tr>
<td>qcow2</td>
<td>Định dạng đĩa QEMU, định dạng mặc định hỗ trợ bởi KVM vfa QEMU, hỗ trợ các chức năng nâng cao</td>
</tr>

<tr>
<td>VDI</td>
<td>Định dạng ảnh đĩa ảo hỗ trợ bởi VirtualBox</td>
</tr>

<tr>
<td>ISO</td>
<td>Định dạng lưu trữ cho đĩa quang</td>
</tr>

<tr>
<td>AMI, ARI, AKI</td>
<td>Định dạng ảnh Amazon machine, ramdisk, kernel</td>
</tr>
</table>

<h3>Container Formats</h3>
Container Formats mô tả định dạng files và chứa các thông tin metadata về máy ảo thực sự. Các định dạng container hỗ trợ bởi Glance
<table>
<tr>
<td>Container Formats</td>
<td>Notes</td>
</tr>

<tr>
<td>bare</td>
<td>Định dạng xác định không có container hoặc meradate đóng gói cho image</td>
</tr>

<tr>
<td>ovf</td>
<td>Định dạng container OVF</td>
</tr>

<tr>
<td>aki</td>
<td>Xác định lưu trữ trong Glance là Amazon kernel image</td>
</tr>

<tr>
<td>ari</td>
<td>Xác định lưu trữ trong Glance là Amazon ramdisk image </td>
</tr>

<tr>
<td>ami</td>
<td>Xác định lưu trữ trong Glance là Amazon machine image</td>
</tr>

<tr>
<td>ova</td>
<td>Xác định lưu trữ trong Glance là file lưu trữ OVA</td>
</tr>

<tr>
<td>docker</td>
<td>Xác định lưu trữ trong Glance và file lưu trữ Docker</td>
</tr>
</table>

<h3><a name="flow">4.1.6 Luồng trạng thái của Glance</a></h3>
Các trạng thái của image:
<ul>
<li><b>queued</b><br>
<div>Định danh của image được bảo vệ trong Glance registry. Không có dữ liệu nào của image được tải lên Glance và kích thước của image không được thiết lập rõ ràng sẽ được thiết lập về zero khi khởi tạo.</div>
</li>
<li><b>saving</b><br>
<div>Trạng thái này biểu thị rằng dữ liệu thô của image đang upload lên Glance. Khi image được đăng ký với lời gọi POST /images và có một header đại diện x-image-meta-location, image đó sẽ không bao giờ được đưa và trạng thái "saving" (bởi vì dữ liệu của image đã có sẵn ở một nơi nào đó)</div>
</li>
<li><b>active</b><br>
<div>Biểu thị rằng một image đã sẵn sàng trong Glance. Trạng thái này được thiết lập khi dữ liệu của image được tải lên hoàn toàn.</div>
</li>
<li><b>deactivated</b><br>
<div>Trạng thái biểu thị việc không được phép truy cập vào dữ liệu của image với tài khoản không phải admin. Khi image ở trạng thái này, ta không thể tải xuống cũng như export hay clone image.</div>
</li>
<li><b>killed</b><br>
<div>Trạng thái biểu thị rằng có vấn đề xảy ra trong quá trình tải dữ liệu của image lên và image đó không thể đọc được</div>
</li>
<li><b>deleted</b><br>
<div>Trạng thái này biểu thị việc Glance vẫn giữ thông tin về image nhưng nó không còn sẵn sàng để sử dụng nữa. Image ở trạng thái này sẽ tự động bị gỡ bỏ vào ngày hôm sau.</div>
</li>
<li><b>pending_delete: </b><br>
Tương tự như trạng thái <b>deleted</b>, tuy nhiên Glance chưa gỡ bỏ dữ liệu của image ngay. Một image khi đã rơi vào trạng thái này sẽ không có khả năng khôi phục.
</li>
</ul>
<b>Deactivating & Reactivating một image</b><br>
Deactivating một image nhằm mục đích cơ bản là hạn chế không cho bất cứ instance nào được built từ image đó. Hoặc trong khi thực hiện việc cập nhật các image, admin có thể muốn deactivating nó tới tất cả người dùng, sau đó khi cập nhật được hoàn tất, admin có thể reactivating các image để users có thể khởi động máy ảo từ nó.
<br> 
Một image chỉ có thể deactivated  khi nó thực sự đã active. Image ở các trạng thái khác không thể deactivate được.
<br><br>
Luồng trạng thái của Glance cho biết trạng thái của image trong quá trình tải lên. Luồng trạng thái của flow được mô tả theo hình sau: <img src="./img/status_flow.png"/>
<br>
<ul>
<li>Khi tạo một image, bước đầu tiên là <b>Queing</b>, image được đưa vào hàng đợi trong một khoảng thời gian ngắn, được bảo vệ và sẵn sàng để tải lên.
</li>
<li>Sau khi queuing, image chuyển sang trạng thái <b>Saving</b> nghĩa là quá trình tải lên chưa hoàn thành.
</li>
<li>Một khi image được tải lên hoàn toàn, trạng thái image chuyển sang <b>Active</b>.
</li>
<li>Khi quá trình tải lên thất bại nó sẽ chuyển sang trạng thái bị <b>killed</b> hoặc <b>deleted</b>.
</li>
</ul>

<h3><a name="img">4.1.7 Image and Instance</a></h3>
Virual machine images hay còn gọi là Image chứa một đĩa ảo có khả năng khởi động hệ điều hành trên đó. Dịch vụ Image kiểm soát việc lưu trữ và quản lí các image.
<br>
Instances là các máy ảo riêng biệt chạy trên node compute. User có thể vận hành bao nhiêu máy ảo tùy ý với cùng một image. Mỗi máy ảo đã được vận hành được tạo nên bởi một bản sao của image gốc, bởi vậy bất kỳ chỉnh sửa nào trên instance cũng không ảnh hưởng tới image gốc. Ta có thể tạo bản snapshot của các máy ảo đang chạy nhằm mục đích dự phòng hoặc vận hành một máy ảo khác. 
<br>
Khi ta vận hành một máy ảo, ta cần phải chỉ ra flavor của máy ảo đó. Flavor đại diện cho tài nguyên ảo hóa cung cấp cho máy ảo, định nghĩa số lượng CPU ảo, tổng dung lượng RAM cấp cho máy ảo và kích thước ổ đĩa không bền vững cấp cho máy ảo. OpenStack cung cấp một số flavors đã định nghĩa sẵn như hình dưới, ta cũng có thể tạo và chỉnh sửa các flavors theo ý mình.
<br>
<img src="./img/flavor.png"/>
<br>
Sơ đồ dưới đây chỉ ra trạng thái của hệ thống trước khi vận hành máy ảo. Trong đó image store chỉ số lượng các images đã được định nghĩa trước, compute node chứa các vcpu có sẵn, tài nguyên bộ nhớ và tài nguyên đĩa cục bộ, cinder-volume chứa số lượng volumes đã định nghĩa trước đó.
<br>
<img src="./img/state_no_runing.png"/>
<br>
Để chạy một máy ảo, ta phải chọn một image, flavor và các thuộc tính tùy chọn. Lựa chọn flavor nào cung cấp root volume, có nhãn là vda và một ổ lưu trữ tùy chọn được đánh nhãn vdb (ephemeral - không bền vững, và cinder-volumen được map với ổ đĩa ảo thứ ba, có thể gọi tên là vdc
<br>
<img src="./img/state_runing.png"/>
<br>
Theo mô tả trên hình, image gốc được copy vào ổ lưu trữ cục bộ từ image store. Ổ vda là ổ đầu tiên mà máy ảo truy cập. Ổ vdb là ổ tạm thời (không bền vững - ephemeral) và rỗng, được tạo nên cùng với máy ảo, nó sẽ bị xóa khi ngắt hoạt động của máy ảo. Ổ vdc kết nối với cinder-volume sử dụng giao thức iSCSI. Sau khi compute node dự phòng vCPU và tài nguyên bộ nhớ, máy ảo sẽ boot từ root volume là vda. Máy ảo chạy và thay đổi dữ liệu trên các ổ đĩa. Nếu volume store được đặt trên hệ thống mạng khác, tùy chọn "my_block_storage_ip" phải được dặc tả chính xác trong tệp cấu hình storage node chỉ ra lưu lượng image đi tới compute node. 
<br>
Khi máy ảo bị xóa, ephemeral storage (khối lưu trữ không bền vững) bị xóa; tài nguyên vCPU và bộ nhớ được giải phóng. Image không bị thay đổi sau tiến trình này.
<br>
<img src="./img/state_end_running.png"/>
<br>
<h2><a name="install">4.2. Cài đặt dịch vụ Glance</a></h2>
<b>Lưu ý: </b>Password thống nhất cho tất cả các dịch vụ là Welcome123
<h3><a name="prerequisites">4.2.1 Tạo database, dịch vụ xác thực và API endpoints cho Glance</a></h3>
<a name="4.2.1.1."> </a> 
#### 4.2.1.1 Tạo database cho `glance`
- Đăng nhập vào mysql
  ```sh
  mysql -uroot -pWelcome123
  ```

- Tạo database và gán các quyền cho user trong database `glance`
	```sh
	CREATE DATABASE glance;
	GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY 'Welcome123';
	GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'Welcome123';
	FLUSH PRIVILEGES;
		
	exit;
	```

<a name="4.2.1.2."> </a> 
#### 4.2.1.2. Cấu hình xác thực cho dịch vụ `glance`
- Lấy thông tin xác thực bằng cách sử dụng file `admin-openrc`
  ```sh
  source admin-openrc
  ```
  
- Tạo user `glance`
	```sh
	openstack user create glance --domain default --password Welcome123
	```

- Gán quyền `admin` cho user `glance` và project `service` 
	```sh
	openstack role add --project service --user glance admin
	```
  Các bước phía trên giúp tạo một user tên là glance được cấp quyền admin trong project service, từ đó dịch vụ glance có thể sử dụng user này để thực hiện các request tới các dịch vụ khác khi cần thiết.
  
- Kiểm tra lại xem user `glance` có role là gì

  ```sh
  openstack role list --user glance --project service
  ```
	
- Tạo dịch vụ có tên là `glance`
	```sh
	openstack service create --name glance --description "OpenStack Image service" image
	```

<a name="4.2.1.3."> </a> 
#### 4.2.1.3. Tạo các endpoints
- Tạo các endpoint cho dịch vụ `glance`
	```sh
	openstack endpoint create --region RegionOne image public http://controller:9292

	openstack endpoint create --region RegionOne image internal http://controller:9292

	openstack endpoint create --region RegionOne image admin http://controller:9292
	```
	
<h3><a name="ins_conf">4.2.2 Cài đặt và cấu hình các thành phần của Glance</a></h3>
- Cài đặt gói `glance`
	```sh
	apt-get -y install glance
	```

- Sao lưu các file `/etc/glance/glance-api.conf` và `/etc/glance/glance-registry.conf` trước khi cấu hình
	```sh
	cp /etc/glance/glance-api.conf /etc/glance/glance-api.conf.orig
	cp /etc/glance/glance-registry.conf /etc/glance/glance-registry.conf.orig
	```

- Sửa các mục dưới đây trong hai file `/etc/glance/glance-api.conf`

 - Trong section `[DEFAULT]`  thêm hoặc tìm và thay thế dòng cũ bằng dòng dưới để cho phép chế độ ghi log với `glance`
	 ```sh
	 verbose = true
	 ```
 
 - Trong section `[database]` :
 
 - Comment dòng 
	 ```sh
	 #sqlite_db = /var/lib/glance/glance.sqlite
	 ```
 - Thêm dòng dưới 
	 ```sh
	 connection = mysql+pymysql://glance:Welcome123@controller/glance
	 ```
 
 - Trong section `[keystone_authtoken]` sửa các dòng cũ thành dòng dưới để cấu hình dịch vụ xác thực với Keystone khi có yêu cầu từ user hoặc nova component
		```sh
		auth_uri = http://controller:5000
		auth_url = http://controller:35357
		memcached_servers = controller:11211
		auth_type = password
		project_domain_name = default
		user_domain_name = default
		project_name = service
		username = glance
		password = Welcome123
		```
    <b>auth_uri:</b> chỉ đến dịch vụ Keystone. Thông tin này được sử dụng bởi các middleware để truy vấn Keystone về tính hợp lệ của thẻ xác thực.
    
 - Trong section ` [paste_deploy]` khai báo dòng dưới
		```sh
		flavor = keystone
		```
 - Khai báo trong section `[glance_store]` nơi lưu trữ file image
 
     ```sh
     stores = file,http
     default_store = file
     filesystem_store_datadir = /var/lib/glance/images/
     ```
    Trong cấu hình trên, ta cho phép hai hệ thống backend lưu trữ image là `file` và `http`, trong đó sử dụng hệ thống backend lưu trữ mặc định là `file`. Cấu hình thư mục lưu trữ các file images khi tải lên glance nằm trong thư mục `/var/lib/glance/images/`
    
- Sửa các mục dưới đây trong hai file `/etc/glance/glance-registry.conf`
 - Trong section `[database]` :
 
 - Comment dòng 
	 ```sh
	 #sqlite_db = /var/lib/glance/glance.sqlite
	 ```
 - Thêm dòng dưới 
	 ```sh
	 connection = mysql+pymysql://glance:Welcome123@controller/glance
	 ```

 - Trong section `[keystone_authtoken]` sửa các dòng cũ thành dòng dưới
	 ```sh
	 auth_uri = http://controller:5000
	 auth_url = http://controller:35357
	 memcached_servers = controller:11211
	 auth_type = password
	 project_domain_name = default
	 user_domain_name = default
	 project_name = service
	 username = glance
	 password = Welcome123
	 ```

 - Trong section ` [paste_deploy]` khai báo dòng dưới
	 ```sh
	 flavor = keystone
	 ```
	
- Đồng bộ database cho glance
	```sh
	su -s /bin/sh -c "glance-manage db_sync" glance
	```

- Khởi động lại dịch vụ `Glance`
	```sh
	service glance-registry restart
	service glance-api restart
	```

- Xóa file database mặc định trong `glance`
	```sh
	rm -f /var/lib/glance/glance.sqlite
	```
<h3><a name="verify">4.2.3 Kiểm chứng lại việc cài đặt Glance</a></h3>
- Khai báo biến môi trường cho dịch vụ `glance`
	```sh
	echo "export OS_IMAGE_API_VERSION=2" | tee -a admin-openrc demo-openrc

	source admin-openrc
	```

- Tải file image cho `glance`. Ở đây ta tải image <b>Cirros</b>, chúng có kích thước bé được thiết kế để test trên clound cũng như trên OpenStack Compute. 
	```sh
	wget http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img
	```
  Ngoài ra ta có thể sử dung một số image khác như CentOS, Debian, Fedora… trên trang web <a>http://docs.openstack.org/image-guide/obtain-images.html</a>

- Upload file image vừa tải về
	```sh
	openstack image create "cirros" \
	 --file cirros-0.3.4-x86_64-disk.img \
	 --disk-format qcow2 --container-format bare \
	 --public
	```

- Kiểm tra lại image đã có hay chưa
	```sh
	openstack image list
	```
	
- Nếu kết quả lệnh trên hiển thị như bên dưới thì dịch vụ `glance` đã cài đặt thành công.
	```sh
	root@controller:~# openstack image list
	+--------------------------------------+--------+--------+
	| ID                                   | Name   | Status |
	+--------------------------------------+--------+--------+
	| 19d53e24-2985-4f75-bd63-7568a5f2f10f | cirros | active |
	+--------------------------------------+--------+--------+
	root@controller:~#

	```
##5. OpenStack Compute và cấu hình cài đặt Nova
<a name="overview"></a>
##5.1. Tóm tắt về dịch vụ OpenStack Compute
Sử dụng OpenStack Compute để lưu trữ và quản lý hệ thống điện toán đám mây. OpenStack compute là 1 phần quan trọng của hệ thống cung cấp dịch vụ hạ tầng điện toán đám mây IaaS (Infrestructure-as-a-Service). Các module chính được viết bằng Python.

![](./img/os .png)

OS Compute tương tác với OS Identity (Keystone) để xác thực người dùng, tương tác với dịch vụ OS Image (Glance & Cinder) để lấy đĩa cài hệ điều hành, và OS Dashboard  để cung cấp dịch vụ tương tác cho người sử dụng và giao diện quản trị. Truy cập vào Image bị giới hạn bởi các project và các người dùng, và qouta cung cấp bị giới hạn cho project. OpenStack Compute có thể scale lên phần cứng chuẩn (mở rộng phần cứng vật lý), và tải xuống image để chạy thực thể.

![](./img/nova architecture 2.png)

Một số thành phần chính của OpenStack Compute:
<b>+ Cloud controller:</b> biểu diễn trạng thái tổng thể và tương tác với các thành phần khác. Các máy chủ API đóng vai trò như các web service front-end cho Cloud Controller.
Compute controller cung cấp các máy chủ tính toán có chứa luôn cả dịch vụ Compute.
<b>+ Object storage:</b> là 1 thành phần cung cấp các dịch vụ lưu trữ, bạn có thể sử dụng OpenStack Object Storage để thay thế.
<b>+ Auth manager:</b> cung cấp dịch vụ xác thực và ủy quền khi sử dụng hệ thống Compute. Bạn cũng có thể sử dụng OpenStack Identity như 1 dịch vụ xác thực riêng biệt để thay thế.
<b>+ Volume controller:</b> cung cấp các khối lưu trữ 1 cách nhanh chóng và thường xuyên cho các máy chủ compute.

<b>+ Network controller:</b> cung cấp các mạng ảo để cho phép các máy chủ compute tương tác với nhau và với mạng công cộng. Bạn cũng có thể sử dụng OpenStack Networking để thay thế.

<b>+ Scheduler:</b> được sử dụng để lựa chọn máy compute phù hợp nhất để triển khai 1 instance.


Các thành phần của nova được liên kết với nhau bằng Queue Server.Ở đây, Queue Server dùng rabbitmq, một gói phần mềm chuyên làm nhiệm vụ chuyển request đến đích tương ứng. Do các thành phần của nova đều hoàn toàn độc lập với nhau, có thể chạy trên các máy khác nhau, và số lượng mỗi thành phần là không hạn chế, trong trường hợp có 1 máy bị hỏng, rabbitmq sẽ chọn ra 1 máy khác có cùng dịch vụ để gởi request. Trạng thái của toàn bộ hệ thống sẽ được lưu trong 1 csdl. Cloud controller giao tiếp với các internal object sử dụng HTTP, nhưng nó giao tiếp với các dịch vụ scheduler, network controller và volume controller sử dụng AMQP (Advance Message Queuing Protocol). Để tránh bị block 1 thành phần trong khi đợi 1 response thì Compute sử dụng các cuộc gọi không đồng bộ, với 1 callback sẽ được kích hoạt khi nhận được 1 response. Một hệ điều hành ảo có thể được format và mount 1 thiết bị lưu trữ như vậy.

<b>Ảo hóa (Hypervisor)</b>

Compute điểu khiển ảo hóa thông qua 1 máy chủ API. Để chọn hypervisor tốt nhất để sử dụng khá khó khăn, bạn phải lấy tài nguyên, các tính năng được hỗ trợ, và yêu cầu thông số kỹ thuật cần thiết trong tài khoản. Tuy nhiên, phần lớn các phát triển của OpenStack sử dụng KVM và Xen-based.

<b>Projects, users và roles</b>
Hệ thống Compute được thiết kế để phục vụ nhiều người dùng khác nhau trong các project trên 1 hệ thống được chia sẻ, và dựa trên các vai trò truy cấp. Các role kiểm soát các hoạt động mà 1 user được phép thực hiện.
</br>
Các project bao gồm một VLAN riêng, và các volume, instance, images, keys, và các user. Một user có thể chỉ định project bằng cách gắn thêm project_id vào khóa truy cập (access key) của họ. Nếu không có project nào được chỉ định trong API request, Compute sẽ chọn 1 project có ID tương tự như user.
</br>
Đối với các project, bạn có thể sử dụng quota control để hạn chế:

- Số lượng volume có thể được triển khai 
- Số nhân xử lý và dung lượng Ram có thể được phân bổ
- Glián địa chỉ IP cho bất kỳ instance nào khi nó được chạy. Điều này cho phép các instance có thể có cùng 1 địa chỉ IP được public
- Cố định các địa chỉ IP được giao cho các instance giống nhau khi nó chạy. Điều này cho phép các instance có thể có địa chỉ IP public hay private giống nhau.

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

<a name="service" ></a>
##5.2. Các service của OpenStack Compute
<a name="api"></a>
###5.2.1.  API
<b>Nova-api service:</b> chấp nhận và phản hồi với người dùng cuối yêu cầu API Compute. Nó hỗ trợ các dịch vụ OpenStack Compute API, Amazon EC2 API, và Admin API đặc biệt để cho người dùng có đặc quyền thực hiện các hành vi chính. Nó thực thi một số chính sách và khởi tạo các hoạt động cùng nhau, ví dụ như chạy 1 instance.
Theo mặc định, nova-api lắng nghe trên cổng 8773 đối với EC2 API và trên cồng 8774 đối với OpenStack API.

<b>Nova-api-metadata service:</b> chấp nhận các yêu cầu metadata từ các instance. Nova-api-metadata service thường được sử dụng khi bạn chạy ở chế độ đa máy chủ (multi-host) với cài đăth nova-network

<a name="compute_core"></a>
###5.2.2. Compute core

![](./img/nova compute.jpg)

<b>Nova-compute service:</b> là 1 daemon tạo ra và chấm dứt các thể hiện của máy ảo thông qua các API của công nghệ tạo máy ảo đó. Ví dụ

- XenAPI cho XenServer/XCP
- Libvirt cho KVM hay QEMU
- VMwareAPI cho VMware

Tiến trình này khá phức tạp, về cơ bản, các daemon chấp nhận các hoạt động từ hàng đợi và thực hiện một loạt các lệnh hệ thống như chạy 1 máy ảo KVM và cập nhật trạng thái của nó trong database.
</br>
<b>Nova-scheduler service:</b> lấy một yêu cầu tạo máy ảo từ hàng đợi và xác định máy chủ compute mà nó chạy.
</br>
<b>Nova-conductor module:</b> trung gian tương tác giữa nova-compute service và database. Nó giúp loại bỏ các truy cập trực tiếp từ các nova-compute service đến database. Nova-conductor module chia theo chiều ngang. Tuy nhiên, không triển khai module này trên các nút có dịch vụ nova-compute chạy.
</br>
<b>Nova-cert module:</b> Một máy chủ daemon phục vụ dịch vụ Nova Cert cho chuẩn X509. Được sử dụng để tạo ra các giấy chứng nhận cho euca-bundle-image. Nó chỉ cần thiết cho EC2 API.

<a name="network"></a>
###5.2.3. Networking for VMs
<b>Nova-network worker daemon:</b> tương tự như nova-compute, nó chấp nận nhiệm vụ kết nối mạng từ hàng đợi và điều khiển mạng. Nhiệm vụ thực hiện là thiet lập các giao diện cầu nối hoặc thay đổi các luật IPtables.
</br>

<a name="console"></a>
###5.2.4. Console interface 
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

<a name="command_line"></a>
###5.2.5. Command-line clients and other interfaces
<b>Nova client:</b> cho phép người dùng thực thi các lệnh như 1 quản trị viên được ủy quyền hay người dùng cuối.
</br>

<a name="other_component"></a>
###5.2.6. Other components
<b>Hàng đợi:</b> một trung tâm để truyền các tin nhắn giữa các daemon. Thường được thực hiện với RabbitMQ, nhưng về mặt lý thuyết có thể được thực hiện với 1 hàng đợi tin nhắn AMQP chẳng hạn như Zero MQ.
Nova tạo ra một số loại hàng đợi thông điệp để tạo điều kiện giao tiếp giữa các daemon khác nhau. Chúng bao gồm: topics queue, fanout queue, và host queue. Topics queue sẽ cấp số cho các tin nhắn truyền đi vào các class riêng ứng với topic tương ứng của các worker daemon. Ví dụ, Nova sử dụng chúng để truyền các tin nhắn tới tất cả (hoặc một số) các volume daemon hay compute. Điều này cho phép Nova sử dụng worker đầu tiên có sẵn để xử lý tin nhắn. Host queue cho phép Nova gửi tin nhắn tới các dịch vụ riêng biệt trên các máy chủ cụ thể nào đó. Fanout queues hiện đang được sử dụng cho các thông báo của các dịch vụ phục vụ cho nova-scheduler worker.
Để xem các topic trong RabbitMQ, bạn sử dụng lệnh:

![](./img/topic rabbitmq.jpg);

</br>
Để xem các hàng đợi được tạo ra trong RabbitMQ cho 1 node được cài đặt đơn giản, các bạn có thể dùng lệnh:
</br>
```sh
$ sudo rabbitmqctl list_queues
```
</br>
Kết quả:
</br>
![](./img/list queues.jpg)

</br>
<b>Cơ sở dữ liệu SQL: </b>lưu trữ tất cả các trạng thái build-time và run-time cho 1 cơ sở hạ thầng điện toán đám mây, bao gồm:

- Các loại instance sẵn có
- Các instance trong sử dụng
- Các project

Về mặt lý thuyết, OpenStack Compute có thể hỗ trợ bất kỳ cơ sở dữ liệu nào mà SQL-Alchemy hỗ trợ. Csdl phổ biến là SQLite3 để thử nghiệm và phát triển công việc, MySQL và PostgreSQL.
</br></br>

<a name="request"></a>
##5.3. Xử lý khi nhận được request
</br>
![](./img/request-flow1.png)

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

<a name="install_nova_controller"></a>
##5.4. Cài đặt và cấu hình nova trên node Controller
<a name="create_db_enpoint"></a>
###5.4.1. Tạo database và endpoint cho nova

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
- Khai báo trong section [api_database] dòng dưới, do section [api_database] chưa có nên ta khai báo thêm
 ```sh
[api_database]
connection = mysql+pymysql://nova:NOVA_DBPASS@controller/nova_api
  ```
- Khai báo trong section [database] dòng dưới. Do section [database] chưa có nên ta khai báo thêm.
   ```sh
[database]
connection = mysql+pymysql://nova:NOVA_DBPASS@controller/nova
  ``` 
- Trong section [DEFAULT] :

	* Thay dòng:</br> logdir=/var/log/nova
	</br>Bằng dòng:</br> log-dir=/var/log/nova>
	* Thay dòng:</br> enabled_apis=ec2,osapi_compute,metadata
	Bằng dòng:</br> enabled_apis=osapi_compute,metadata
	* Bỏ dòng:</br> verbose = True
	* Khai báo thêm các dòng sau:</br>
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
- Khai báo trong section [oslo_messaging_rabbit] các dòng dưới. Do section[oslo_messaging_rabbit] chưa có nên ta khai báo thêm.
```sh
[oslo_messaging_rabbit]
rabbit_host = controller
rabbit_userid = openstack
rabbit_password = Welcome123
  ```
- Trong section [keystone_authtoken] khai báo các dòng dưới. Do section[keystone_authtoken] chưa có nên ta khai báo thêm.
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
- Trong section [vnc] khai báo các dòng dưới để cấu hình VNC điều khiển các máy ảo trên web. Do section [vnc] chưa có nên ta khai báo thêm.
  ```sh
[vnc]
vncserver_listen = $my_ip
vncserver_proxyclient_address = $my_ip
  ``` 
- Trong section [glance] khai báo dòng để nova kết nối tới API của glance. Do section [glance] chưa có nên ta khai báo thêm.
 ```sh
[glance]
api_servers = http://controller:9292
  ```
- Trong section [oslo_concurrency] khai báo dòng dưới. Do section [oslo_concurrency]chưa có nên ta khai báo thêm.
   ```sh
[oslo_concurrency]
lock_path = /var/lib/nova/tmp
  ```
- Khai báo thêm section mới [neutron] để nova làm việc với neutron
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
  
<a name="end"></a>
###5.4.2. Kết thúc bước cài đặt và cấu hình nova

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
<a name="install_nova_compute"></a>
##5.4. Cài đặt và cấu hình nova trên node Compute

Cài đặt gói nova-compute </br>
```sh
apt-get -y install nova-compute
 ```
<b>Cấu hình nova-comupte</b> </br>
- Sao lưu file /etc/nova/nova.conf </br>
	```sh
	cp /etc/nova/nova.conf /etc/nova/nova.conf.orig
	 ```
- Trong section [DEFAULT] khai báo các dòng sau: </br>
  ```sh
pc_backend = rabbit
auth_strategy = keystone
my_ip = 10.10.10.41
	
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver
 ```
- Khai báo thêm section [oslo_messaging_rabbit] và các dòng dưới: </br>
  ```sh
[oslo_messaging_rabbit]
rabbit_host = controller
rabbit_userid = openstack
rabbit_password = Welcome123
 ```

- Khai báo thêm section [keystone_authtoken] và các dòng dưới: </br>
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
- Khai báo thêm section [vnc] và các dòng dưới: </br>
 ```sh
[vnc]
enabled = True
vncserver_listen = 0.0.0.0
vncserver_proxyclient_address = $my_ip
novncproxy_base_url = http://172.16.69.40:6080/vnc_auto.html
 ```
- Khai báo thêm section [glance] và các dòng dưới: </br>
 ```sh
[glance]
api_servers = http://controller:9292
 ```
- Khai báo thêm section [oslo_concurrency] và các dòng dưới: </br>
 ```sh
[oslo_concurrency]
lock_path = /var/lib/nova/tmp
 ```
- Khai báo thêm section [neutron] và các dòng dưới: </br>
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




#6. Cài đặt dịch vụ mạng của OpenStack
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
