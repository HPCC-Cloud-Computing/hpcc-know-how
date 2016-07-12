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
apt-get install software-properties-common -y
add-apt-repository cloud-archive:mitaka -y
```

Sau đó cập nhật lại:
```sh
apt-get update && apt-get dist-upgrade
```

Sau đó ta cài đặt OpenStack client:
```sh
apt-get install python-openstackclient
```

Khởi động lại máy, đăng nhập vào quyền ```root``` và thực hiện các bước tiếp theo
####Cài đặt hệ quản trị cơ sở dữ liệu SQL

Sau khi cài đặt OpenStack client, chúng ta cần cài đặt cơ sở dữ liệu lên controller node, vì các dịch vụ của OpenStack sử dụng SQL để lưu trữ thông tin.
Ta cài đặt gói mariaDb:
```sh
apt-get install mariadb-server python-pymysql
```

Thiết lập mật khẩu: bkcloud16

Tạo file ``` /etc/mysql/conf.d/mysqld_openstack.cnf ```với nội dung sau:
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
service mysql restart
```
####Cài đặt dịch vụ Network Time Protocol
Network Time Protocol là dịch vụ cho phép đồng bộ hóa giữa các máy tính trong mạng thông qua sử dụng NTP để đồng bộ thời gian giữa các máy.

Để cài đặt dịch vụ NTP, ta cài đặt the packages chrony:
```sh
apt-get install chrony
```
Tiến hành chỉnh sửa file cấu hình ```/etc/chrony/chrony.conf```:
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
service chrony restart
```
####Cài đặt dịch vụ Message queue
Dịch vụ Message queue giúp các services của hệ thống trao đổi các thông điệp với  nhau. Ở bản cài đặt này ta sử dụng dịch vụ RabbitMQ.

Ta cài đặt gói rabbitmq-server lên controller node:
```sh
apt-get -y install rabbitmq-server
```

Cấu hình RabbitMQ, tạo user openstack với mật khẩu là ```bkcloud16```:
```sh
rabbitmqctl add_user openstack bkcloud16
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
Dùng nano  sửa ``` file /etc/memcached.conf ```, thay dòng 
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
echo "compute" > /etc/hostname
hostname -F /etc/hostname
```

Khởi động lại máy.

Sau đó thiết lập địa chỉ IP cho eth0 và eth1:
Thiết lập địa chỉ IP, chỉnh sửa  file ``` /etc/network/interfaces ``` với nội dung sau:
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
apt-get install software-properties-common
add-apt-repository cloud-archive:mitaka
```
Sau đó cập nhật lại:
```sh
apt-get update && apt-get dist-upgrade
```
Sau đó ta cài đặt OpenStack client:
```sh
apt-get install python-openstackclient
```
###Cài đặt và cấu hình NTP trên Compute node
Ta cài đặt NTP Client
```sh
apt-get -y install chrony
```
Chỉnh sửa file ```/etc/chrony/chrony.conf```.Thay các dòng dưới
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

<h3><a name="install_config">3. Cấu hình và cài đặt Keystone</a></h3>
- Trước tiên, cần phải tạo ra một database cho dịch vụ keystone bằng các câu lệnh sau:
```sh
mysql -u root –pbkcloud16
CREATE DATABASE keystone;

GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
IDENTIFIED BY 'bkcloud16';

GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
IDENTIFIED BY 'bkcloud16';
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
  admin_token = bkcloud16
  ```
 - Trong phần database, cấu hình truy cập đến database:
  ```sh
  [database]
  ...
  connection = mysql+pymysql://keystone:bkcloud16@controller/keystone
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
export OS_TOKEN=bkcloud16
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
  identity admin http://controller:35357/v3
 ```
<h3>Tạo domain, user, project và role</h3>
- Tạo domain

	```sh
	openstack domain create --description "Default Domain" default
	```

- Tạo `admin` project

	```sh
	openstack project create --domain default  --description "Admin Project" admin
	```

- Tạo user `admin`
	```sh
	openstack user create admin --domain default --password bkcloud16
	```

- Tạo role `admin`
	```sh
	openstack role create admin
	```

- Gán user `admin` vào role `admin` thuộc project `admin`
	```sh
	openstack role add --project admin --user admin admin
	```

- Tạo project có tên là `service` để chứa các user service của openstack
	```sh
	openstack project create --domain default --description "Service Project" service
	```

- Tạo project tên là `demo`
	```sh
	openstack project create --domain default --description "Demo Project" demo
	```

- Tạo user tên là `demo`
	```sh
	openstack user create demo --domain default --password bkcloud16
	```

- Tạo role tên là `user`
	```sh
	openstack role create user
	```

- Gán tài khoản `demo` có role là `user` vào project `demo`
	```sh
	openstack role add --project demo --user demo user
	```

<h3>Kiểm tra hoạt động</h3>
- Vì lý do bảo mật, vô hiệu hóa cơ chế thẻ token tạm thời bằng cách chỉnh sửa trong file `/etc/keystone/keystone-paste.ini`, xóa các dòng `admin_token_auth` từ các phần `[pipeline:public_api]`,`[pipeline:admin_api]` và `[pipeline:api_v3]`
- Gỡ bỏ các biến môi trường đã thiết lập trong quá trình tạo service và endpoint cho dịch vụ Identity:
```sh
unset OS_TOKEN OS_URL
```
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
export OS_PASSWORD=bkcloud16
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

<h3><a name="prerequisites">4.1 Tạo database, dịch vụ xác thực và API endpoints cho Glance</a></h3>
<a name="4.1.1."> </a> 
#### 4.1.1 Tạo database cho `glance`
- Đăng nhập vào mysql
  ```sh
  mysql -uroot -pbkcloud16
  ```

- Tạo database và gán các quyền cho user trong database `glance`
	```sh
	CREATE DATABASE glance;
	GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY 'bkcloud16';
	GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'bkcloud16';
	FLUSH PRIVILEGES;
		
	exit;
	```

<a name="4.1.2."> </a> 
#### 4.1.2. Cấu hình xác thực cho dịch vụ `glance`
- Lấy thông tin xác thực bằng cách sử dụng file `admin-openrc`
  ```sh
  source admin-openrc
  ```
  
- Tạo user `glance`
	```sh
	openstack user create glance --domain default --password bkcloud16
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

<a name="4.1.3."> </a> 
#### 4.1.3. Tạo các endpoints
- Tạo các endpoint cho dịch vụ `glance`
	```sh
	openstack endpoint create --region RegionOne image public http://controller:9292

	openstack endpoint create --region RegionOne image internal http://controller:9292

	openstack endpoint create --region RegionOne image admin http://controller:9292
	```
	
<h3><a name="ins_conf">4.2 Cài đặt và cấu hình các thành phần của Glance</a></h3>
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
 
 - Trong section `[database]` :
 
 - Comment dòng 
	 ```sh
	 #sqlite_db = /var/lib/glance/glance.sqlite
	 ```
 - Thêm dòng dưới 
	 ```sh
	 connection = mysql+pymysql://glance:bkcloud16@controller/glance
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
		password = bkcloud16
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
	 connection = mysql+pymysql://glance:bkcloud16@controller/glance
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
	 password = bkcloud16
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
<h3><a name="verify">4.3 Kiểm chứng lại việc cài đặt Glance</a></h3>
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
##5. Cấu hình và cài đặt Nova
<a name="overview"></a>
##5.1. Các thành phần của OpenStack Nova
- Nova đảm nhiệm chức năng cung cấp và quản lý tài nguyên trong OpenStack để cấp cho các VM. Trong hướng dẫn nãy sẽ sử dụng KVM làm hypervisor. Nova sẽ tác động vào KVM thông qua libvirt.
Nova có các thành phần như sau: 

- API:
    * nova-api
    * nova-api-metadata
- Compute Core:
    * nova-compute
    * nova-schedule
    * nova-conductor
    * nova-cert
- Networking for VMs:
    * nova-network
- Console interfaces:
    * nova-novncproxy
    * nova-spicehtml5proxy
    * nova-xvpvncproxy
    * nova-cert
    * euca2ools
- Command line:
    * nova client
- Other component:
    * The queue
    * SQL database


<a name="install_nova_controller"></a>
##5.2. Cài đặt và cấu hình nova trên node Controller
<a name="create_db_enpoint"></a>
###5.2.1. Tạo database và endpoint cho nova

Đăng nhập vào database với quyền root
</br>

```sh
$ mysql -uroot -pbkcloud16
```
Tạo database
</br>
```sh
CREATE DATABASE nova_api;
CREATE DATABASE nova;
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' \
  IDENTIFIED BY 'bkcloud16';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' \
  IDENTIFIED BY 'bkcloud16';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' \
  IDENTIFIED BY 'bkcloud16';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' \
  IDENTIFIED BY 'bkcloud16';
```
Khai báo biến môi trường
 ```sh
$ . admin.openrc
  ```
+ Tạo user, phân quyền và tạo endpoint cho dịch vụ nova
</br>
Tạo user có tên là nova
 ```sh
openstack user create nova --domain default  --password bkcloud16
  ```
Phân quyền cho tài khoản nova
 ```sh
$ openstack role add --project service --user nova admin
  ```
  Tạo service có tên là nova
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
  Cài đặt các gói và cấu hình:
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
connection = mysql+pymysql://nova:bkcloud16@controller/nova_api
```
- Khai báo trong section [database] dòng dưới. Do section [database] chưa có nên ta khai báo thêm.
```sh
	[database]
	connection = mysql+pymysql://nova:bkcloud16@controller/nova
```
- Trong section [DEFAULT] :

	* Thay dòng: 
	```sh
		logdir=/var/log/nova
	```
	Bằng dòng: 
	```sh
		log-dir=/var/log/nova>
	```
	* Thay dòng:
	```sh
		enabled_apis=ec2,osapi_compute,metadata
	```
	Bằng dòng:
	```sh
		enabled_apis=osapi_compute,metadata
	```
	* Bỏ dòng:
	```sh
		verbose = True
	```
	* Khai báo thêm các dòng sau:
	```sh
		rpc_backend = rabbit
		auth__strategy = keystone
		rootwrap_config = /etc/nova/rootwrap.conf
		#IP MGNT cua node Controller
		my_ip = 10.10.10.10
		
		use_neutron = True
		firewall_driver = nova.virt.firewall.NoopFirewallDriver
	```
	
- Khai báo trong section [oslo_messaging_rabbit] các dòng dưới. Do section[oslo_messaging_rabbit] chưa có nên ta khai báo thêm.

```sh
[oslo_messaging_rabbit]
rabbit_host = controller
rabbit_userid = openstack
rabbit_password = bkcloud16
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
password = bkcloud16
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
password = bkcloud16


service_metadata_proxy = True
metadata_proxy_shred_secret = bkcloud16
```
  
  Tạo database cho nova:
  
```sh
# su -s /bin/sh -c "nova-manage api_db sync" nova
# su -s /bin/sh -c "nova-manage db sync" nova
```
  
<a name="end"></a>
###5.2.2. Kết thúc bước cài đặt và cấu hình nova

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
##5.3. Cài đặt và cấu hình nova trên node Compute

Cài đặt gói nova-compute </br>
```sh
apt-get -y install nova-compute
```
<b>Cấu hình nova-comupte</b> </br>
- Sao lưu file /etc/nova/nova.conf 
```sh
	cp /etc/nova/nova.conf /etc/nova/nova.conf.orig
```
- Trong section [DEFAULT] khai báo các dòng sau: 
```sh
pc_backend = rabbit
auth_strategy = keystone
my_ip = 10.10.10.11
	
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver
```
- Khai báo thêm section [oslo_messaging_rabbit] và các dòng dưới:
```sh
[oslo_messaging_rabbit]
rabbit_host = controller
rabbit_userid = openstack
rabbit_password = bkcloud16
```

- Khai báo thêm section [keystone_authtoken] và các dòng dưới:
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
password = bkcloud16
```
- Khai báo thêm section [vnc] và các dòng dưới:
```sh
	[vnc]
	enabled = True
	vncserver_listen = 0.0.0.0
	vncserver_proxyclient_address = $my_ip
	novncproxy_base_url = http://192.168.2.10:6080/vnc_auto.html
```
- Khai báo thêm section [glance] và các dòng dưới: 
```sh
[glance]
api_servers = http://controller:9292
```
- Khai báo thêm section [oslo_concurrency] và các dòng dưới: 
```sh
[oslo_concurrency]
lock_path = /var/lib/nova/tmp
```
- Khai báo thêm section [neutron] và các dòng dưới:
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
password = bkcloud16
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
export OS_PASSWORD=bkcloud16
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
#6 Cài đặt dịch vụ OpenStack Networking - Neutron
##6.1 Các thành phần của dịch vụ Neutron
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

##6.2 Chuẩn bị cấu hình trên controller node và tải về các thành phần của neutron trên controller node

Để bắt đầu cài đặt trên controller node, ta cần tạo cơ sở dữ liệu cho Neutron:

###6.2.1 Tạo database, user và endpoint cho neutron.

- Đăng nhập vào MySQL
```sh
	mysql -uroot -pbkcloud16
```
- Tạo database và phân quyền
```sh
	CREATE DATABASE neutron;
	GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'bkcloud16';
	GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'bkcloud16';
		
	FLUSH PRIVILEGES;
	exit;
```
- Khai báo biến môi trường
```sh
source admin.sh
```
- Tạo tài khoản tên ```sh neutron```, thêm tài khoản ```sh neutron``` vào project ```sh service``` với quyền của tài khoản ```sh neutron``` đối với project ```sh service``` là ```sh admin```
```sh
	openstack user create neutron --domain default --password bkcloud16
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
	apt-get update
	apt-get -y install neutron-server neutron-plugin-ml2 \
	neutron-linuxbridge-agent neutron-l3-agent neutron-dhcp-agent \
	neutron-metadata-agent
```
Tiếp theo, ta cấu hình các dịch vụ của neutron
###6.2.2 Cấu hình cài đặt neutron trên controller node
Đầu tiên, ta cấu hình file /etc/neutron/neutron.conf:
####Cấu hình để neutron sử dụng database
Chỉnh sửa section [database] để neutron có thể sử dụng database neutron mà chúng ta vừa tạo ở phần trước:
```sh 
connection = mysql+pymysql://neutron:bkcloud16@controller/neutron
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
rabbit_password = bkcloud16
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
password = bkcloud16
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
password = bkcloud16

```
####Cấu hình Modular Layer 2 (ML2) plug-in
Trong bài hướng dẫn này, chúng ta sử dụng ML2 plugin kết hợp với Linux-bridge-agent để xây dựng nên Layer2 cho mạng ảo. Để cấu hình ML2 plug-in, tiến hành cập nhật file ```/etc/neutron/plugins/ml2/ml2_conf.ini```
- Cập nhật section [ml2], option ``` type_driver``` cho phép chọn các loại mạng có thể được tạo ra và hoạt động bởi mechanism driver. Vì chúng ta sử dụng linux-bridge, chúng ta chọn các mạng flat, vxlan
```sh
[ml2]
...
type_drivers = flat,vxlan
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
metadata_proxy_shared_secret = bkcloud16
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
password = bkcloud16
	
service_metadata_proxy = True
metadata_proxy_shared_secret = bkcloud16	
```
###6.2.3 Kết thúc cài đặt trên controller node
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
##6.3 Cài đặt neutron lên compute node
###6.3.1 Chuẩn bị các thành phần của neutron trên compute node
Trên compute node, ta sẽ triển khai thành phần neutron-linuxbridge-agent. Tải về neutron-linuxbridge-agent:
```sh
apt-get install neutron-linuxbridge-agent
```

###6.3.2 Cấu hình neutron trên compute node
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
rabbit_password = bkcloud16
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
password = bkcloud16
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
local_ip = 10.10.10.11
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
password = bkcloud16
```
###6.3.3 Kết thúc cài đặt trên compute node
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

##7. Cài đặt HORIZON (dashboad)
HORIZON hay còn gọi là dashboad dùng để cung cấp giao diện trên web để người dùng có thể sử dụng OpenStack 1 cách dễ dàng hơn.

###7.1. Cài đặt và cấu hình Horizon 

Cài đặt các thành phần cho dashboad:

```sh
	apt-get -y install openstack-dashboard
```

Sao lưu lại file cấu hình cho dashboad

```sh
	cp /etc/openstack-dashboard/local_settings.py /etc/openstack-dashboard/local_settings.py.orig
```

Tìm các dòng sau trong file /etc/openstack-dashboard/local_settings.py và chỉnh sửa như bên dưới

```sh
	OPENSTACK_HOST = "controller"
```

```sh
	CACHES = {
    		'default': {
         		'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
         		'LOCATION': 'controller:11211',
    		}
	}
```

```sh
	OPENSTACK_KEYSTONE_URL = "http://%s:5000/v3" % OPENSTACK_HOST
```

```sh
	OPENSTACK_API_VERSIONS = {
    		"identity": 3,
    		"image": 2,
    		"volume": 2,
	}
```

```sh
	OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = "default"
```

```sh
	OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"
```

```sh
	TIME_ZONE = "Asia/Ho_Chi_Minh"
```

Xóa theme mặc định của ubuntu

```sh
	apt-get -y remove --auto-remove openstack-dashboard-ubuntu-theme
```

Khởi động lại apache

```sh
	service apache2 restart
```

Mở web với địa chỉ http://192.168.2.10/horizon để vào dashboad

