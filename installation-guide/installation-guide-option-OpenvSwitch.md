#Hướng dẫn cài đặt Open Stack với Open vSwitch
Trong bài viết này mình xin trình bày cách cài đặt Open Stack sử dụng Open vSwitch với một cấu hình đơn giản.

Trong bài viết này thống nhất mật khẩu cho các dịch vụ là 1111
#1 Chuẩn bị môi trường
Cấu hình triển khai gồm 2 node, 2 dải mạng theo sơ đồ sau:
![Openstack-architect.png](./img/Openstack-architect.png).
##1.1 Thiết lập môi trường phần cứng
Để thiết lập hệ thống, đầu tiên ta cần chuẩn bị môi trường.
Môi trường triển khai hệ thống là VMWare workstation 12 trên windows 10.
Sau khi cài đặt VMWare workstation, ta thiết lập cấu hình mạng như sau:

Thiết lập dải mạng cho external network:
 
![external-physic-network.png](./img/external-physic-network.png)

Thiết lập dải mạng cho management network:
![external-physic-network.png](./img/management-physic-network.png) 

Sau khi thiết lập cấu hình mạng, ta tiến hành tạo máy ảo controller và máy ảo compute với cấu hình như bên dưới, với 2 card mạng nối tới 2 mạng ta đã thiết lập bên trên:

![hardware-setting-vmware.png](./img/hardware-setting-vmware.png) 

##1.2 Chuẩn bị môi trường cho controller node
###1.2.1 Thiết lập địa chỉ mạng
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

###1.2.2 Cài đặt dịch vụ MySQL, message queue, Network Time Protocol, message queue, memcached và OpenStack Client
####Cài đặt OpenStack Client
OpenStack Client là services cho phép người dùng tương tác với hệ thống OpenStack thông qua các câu lệnh. Hiện tại OpenStack Client hỗ trợ các dịch vụ : keystone, image, object storage và compute

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
####Cài đặt hệ quản trị cơ sở dữ liệu SQL

Sau khi cài đặt OpenStack client, chúng ta cần cài đặt cơ sở dữ liệu lên controller node, vì các dịch vụ của OpenStack sử dụng SQL để lưu trữ thông tin.
Ta cài đặt gói mariaDb:
```sh
apt-get install mariadb-server python-pymysql
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
Dùng nano  sửa file ``` /etc/memcached.conf ```, thay dòng 
```-l 127.0.0.1 ``` 
bằng dòng dưới: 
```-l 10.10.10.10 ```
Trong đó 10.10.10.10 là địa chỉ nic management của controller node

Khởi động lại memcache
```sh 
service memcached restart
``` 
##1.3 Chuẩn bị môi trường cho compute node
###1.3.1 Thiết lập địa chỉ mạng
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

###1.3.2 Cài đặt dịch vụ Network Time Protocol và OpenStack Client
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
#2 Cài đặt Keystone
##2.1 Chuẩn bị
Thiết lập database cho keystone và gán các role lên database
```sh
mysql -u root -p
CREATE DATABASE keystone;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
  IDENTIFIED BY '1111';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
  IDENTIFIED BY '1111';
```
##2.2 Cài đặt các thành phần
Keystone được cài đặt trên controller node. Tiến hành ngăn keystone tự động bật lên bằng lệnh sau:
```sh
echo "manual" > /etc/init/keystone.override
```
cài đặt các dịch vụ phục vụ cho keystone
```sh
apt-get install keystone apache2 libapache2-mod-wsgi
```
cấu hình keystone ở file ```/etc/keystone/keystone.conf```
```
[DEFAULT]
...
admin_token = 1111
[database]
...
connection = mysql+pymysql://keystone:1111@controller/keystone
[token]
...
provider = fernet
```
đồng bộ hóa dữ liệu cho keystone
```sh
su -s /bin/sh -c "keystone-manage db_sync" keystone
```
tạo khóa fernet
```sh
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
```
cấu hình apache server, chỉnh sửa file ```/etc/apache2/apache2.conf```
```
ServerName controller
```
tạo file ở vị trí sau ```/etc/apache2/sites-available/wsgi-keystone.conf ``` với nội dung như sau
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
kích hoạt dịch vụ xác thực thông qua apache
```sh
ln -s /etc/apache2/sites-available/wsgi-keystone.conf /etc/apache2/sites-enabled
```
khởi động lại apache service
```sh
service apache2 restart
```
xóa bỏ file cơ sở dữ liệu đi kèm keystone
```sh
rm -f /var/lib/keystone/keystone.db
```
##2.3 Tạo service và endpoint cho dịch vụ xác thực
Để lấy được quyền đẻ tạo service và các endpoint, cần xác thực với keystone bằng token đã nạp vào file cấu hình trước đó. Nạp đoạn script sau vào hệ thống để lấy quyền tạm thời và lấy địa chỉ endpoint quản lý của keystone
```sh
export OS_TOKEN=1111
export OS_URL=http://controller:35357/v3
export OS_IDENTITY_API_VERSION=3
```
Sau khi nạp đoạn script trên, tiến hành tạo service cho dịch vụ xác thực và các endpoint cho service này
```sh
openstack service create --name keystone --description "OpenStack Identity" identity
openstack endpoint create --region RegionOne identity public http://controller:5000/v3
openstack endpoint create --region RegionOne identity internal http://controller:5000/v3
openstack endpoint create --region RegionOne identity admin http://controller:35357/v3
```
##2.4 Tạo user, role, project
tạo các project cần thiết, tạo 1 tài khoản có quyền admin đối với hệ thống và 1 tài khoản có quyền user với 1 project mới tạo
```sh
openstack domain create --description "Default Domain" default
openstack project create --domain default  --description "Admin Project" admin
openstack user create admin --domain default --password 1111
openstack role create admin
openstack role add --project admin --user admin admin
openstack project create --domain default --description "Service Project" service
openstack project create --domain default --description "Demo Project" demo
openstack user create demo --domain default --password 1111
openstack role create user
openstack role add --project demo --user demo user
```
##2.5 Kiểm tra cài đặt và tạo script biến môi trường cho client
loại bỏ script lúc trước ta sử dụng
```sh
unset OS_TOKEN OS_URL
```
tạo file admin.sh với nội dung sau
```sh
export OS_PROJECT_DOMAIN_NAME=default
export OS_USER_DOMAIN_NAME=default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=1111
export OS_AUTH_URL=http://controller:35357/v3
export OS_IDENTITY_API_VERSION=3
```
tạo file demo.sh với nội dung sau
```sh
export OS_PROJECT_DOMAIN_NAME=default
export OS_USER_DOMAIN_NAME=default
export OS_PROJECT_NAME=demo
export OS_USERNAME=demo
export OS_PASSWORD=1111
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
```
nạp file admin.sh vào, kiểm tra hoạt động của keystone
```sh
source admin.sh
openstack token issue
```
kết quả nếu keystone hoạt động bình thường
```sh
root@controller:~# openstack token issue
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field      | Value                                                                                                                                                                                   |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| expires    | 2016-03-26T23:10:06.916201Z                                                                                                                                                             |
| id         | gAAAAABW9wi_SwsD6CLTlvIuBUtXbn1eHH8wTtthyWNzNlG2bKO-9e-VK2fRVdRJ4oKnJ1ceU72KyK_C7N9MSWVlyedP3tQucQW2dzxu_fKl5YeslehFjw-Phv2EhXTOWPc-Ga9l017T8SvwzG4pKWqM7BRNe11GP7DeTBDGVHxFOxXuh7OJ3iE |
| project_id | 38b9ce7973634d83a0390f47f028ed32                                                                                                                                                        |
| user_id    | 100f1bf64d184f22bd6babdd5929696c                                                                                                                                                        |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```
#3 Cài đặt Glance
Glance được cài đặt trên controller node. Các cài đặt sau được thực hiện trên controller node
##3.1 Chuẩn bị cơ sở dữ liệu, endpoint, user và service
Thiết lập database cho glance và gán các role lên database
```sh
mysql -u root -p
CREATE DATABASE glance;
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' \
  IDENTIFIED BY '1111';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' \
  IDENTIFIED BY '1111';
```
Tạo glance service, user cho các dịch vụ trong glance sử dụng để xác thực, tạo endpoint
```sh
openstack user create glance --domain default --password 1111
openstack role add --project service --user glance admin
openstack service create --name glance --description "OpenStack Image service" image
openstack endpoint create --region RegionOne image public http://controller:9292
openstack endpoint create --region RegionOne image internal http://controller:9292
openstack endpoint create --region RegionOne image admin http://controller:9292
```
##3.2 Cài đặt và cấu hình glance
Cài đặt glance service
```sh
apt-get -y install glance
```
Cấu hình glance-api ở file ```/etc/glance/glance-api.conf```
```sh
[database]
...
connection = mysql+pymysql://glance:1111@controller/glance
[keystone_authtoken]
...
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = glance
password = 1111

[paste_deploy]
...
flavor = keystone
[glance_store]
...
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/
```
Cấu hình glance-registry ở file ```/etc/glance/glance-registry.conf```
```sh
[database]
...
connection = mysql+pymysql://glance:1111@controller/glance
[keystone_authtoken]
...
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = glance
password = 1111

[paste_deploy]
...
flavor = keystone
```
Đồng bộ hóa cơ sở dữ liệu
```sh
su -s /bin/sh -c "glance-manage db_sync" glance
```
Khởi động lại các dịch vụ của glance
```sh
sudo service glance-registry restart
sudo service glance-api restart
```
##3.3 Kiểm tra hoạt động của glance
Tải file image về máy
```sh
wget http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img
```
Nạp file image này vào hệ thống bằng glance
```sh
openstack image create "cirros" \
  --file cirros-0.3.4-x86_64-disk.img \
  --disk-format qcow2 --container-format bare \
  --public
```
Kiểm tra xem glance đã nạp thành công image vào máy chưa
```sh
openstack image list
+--------------------------------------+--------+--------+
| ID                                   | Name   | Status |
+--------------------------------------+--------+--------+
| 38047887-61a7-41ea-9b49-27987d5e8bb9 | cirros | active |
+--------------------------------------+--------+--------+
```
#4 Cài đặt nova
#4.1 Cài đặt nova trên controller node
###4.1.1 Chuẩn bị cơ sở dữ liệu, endpoint, user và service
Thiết lập database cho glance và gán các role lên database
```sh
mysql -u root -p
CREATE DATABASE nova;
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' \
  IDENTIFIED BY '1111';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' \
  IDENTIFIED BY '1111';
CREATE DATABASE nova_api;
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' \
  IDENTIFIED BY '1111';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' \
  IDENTIFIED BY '1111';
```
Tạo glance service, user cho các dịch vụ trong glance sử dụng để xác thực, tạo endpoint
```sh
openstack user create nova --domain default --password 1111
openstack role add --project service --user nova admin
openstack service create --name nova --description "OpenStack Compute" compute
openstack endpoint create --region RegionOne compute public http://controller:8774/v2.1/%\(tenant_id\)s
openstack endpoint create --region RegionOne compute internal http://controller:8774/v2.1/%\(tenant_id\)s
openstack endpoint create --region RegionOne compute admin http://controller:8774/v2.1/%\(tenant_id\)s
```
###4.1.2 Cài đặt các dịch vụ của nova lên controller node và cấu hình
Cài đặt các package của nova
```
apt-get -y install nova-api nova-cert nova-conductor nova-consoleauth nova-novncproxy nova-scheduler
```
Chỉnh sửa file cấu hình ```/etc/nova/nova.conf```
```sh
[DEFAULT]
...
enabled_apis = osapi_compute,metadata
rpc_backend = rabbit
auth_strategy = keystone
my_ip = 10.10.10.10
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver

[api_database]
...
connection = mysql+pymysql://nova:1111@controller/nova_api

[database]
...
connection = mysql+pymysql://nova:1111@controller/nova

[oslo_messaging_rabbit]
...
rabbit_host = controller
rabbit_userid = openstack
rabbit_password = 1111
[keystone_authtoken]
...
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = 1111
[DEFAULT]
[DEFAULT]
[vnc]
vncserver_listen = $my_ip
vncserver_proxyclient_address = $my_ip
[glance]
api_servers = http://controller:9292
[oslo_concurrency]
...
lock_path = /var/lib/nova/tmp
```
Đồng bộ hóa cơ sở dữ liệu cho nova
```sh
su -s /bin/sh -c "nova-manage api_db sync" nova
su -s /bin/sh -c "nova-manage db sync" nova
```
Khởi động lại các dịch vụ của nova
```sh
sudo service nova-api restart
sudo service nova-consoleauth restart
sudo service nova-scheduler restart
sudo service nova-conductor restart
sudo service nova-novncproxy restart
```
#4.2 Cài đặt nova trên compute node
Ta cài đặt gói nova-compute lên compute node:
```sh 
apt-get install nova-compute
```
Cấu hình nova-compute, cấu hình file ```/etc/nova/nova.conf```
```sh
[DEFAULT]
...
rpc_backend = rabbit
auth_strategy = keystone
my_ip = 10.10.10.11
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver
[oslo_messaging_rabbit]
...
rabbit_host = controller
rabbit_userid = openstack
rabbit_password = 1111


[keystone_authtoken]
...
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = 1111

[vnc]
...
enabled = True
vncserver_listen = 0.0.0.0
vncserver_proxyclient_address = $my_ip
novncproxy_base_url = http://controller:6080/vnc_auto.html
[glance]
...
api_servers = http://controller:9292
[oslo_concurrency]
...
lock_path = /var/lib/nova/tmp
```
Chú ý, nếu máy các bạn không hỗ trợ KVM thì cần chuyển sang chế độ qemu trong file ```/etc/nova/nova-compute.conf```
```sh
[libvirt]
...
virt_type = qemu
```
Khởi động lại service nova-compute
```sh
sudo service nova-compute restart
```
##4.3 Kiểm tra lại hoạt động của các dịch vụ nova
```sh
$ openstack compute service list
+----+--------------------+------------+----------+---------+-------+----------------------------+
| Id | Binary             | Host       | Zone     | Status  | State | Updated At                 |
+----+--------------------+------------+----------+---------+-------+----------------------------+
|  1 | nova-consoleauth   | controller | internal | enabled | up    | 2016-02-09T23:11:15.000000 |
|  2 | nova-scheduler     | controller | internal | enabled | up    | 2016-02-09T23:11:15.000000 |
|  3 | nova-conductor     | controller | internal | enabled | up    | 2016-02-09T23:11:16.000000 |
|  4 | nova-compute       | compute1   | nova     | enabled | up    | 2016-02-09T23:11:20.000000 |
+----+--------------------+------------+----------+---------+-------+----------------------------+
```
#5 Cài đặt neutron
##5.1 Cấu hình cài đặt neutron trên controller node

###5.1.1 Tạo database, user và endpoint cho neutron.

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
source admin.sh
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
###5.1.2 Tải về các dịch vụ của neutron
Ta tiến hành tải về các dịch vụ của neutron trên controller node:
```sh
	source admin.sh
	apt-get update
	apt-get -y install neutron-server neutron-plugin-ml2 neutron-l3-agent neutron-dhcp-agent neutron-metadata-agent neutron-plugin-openvswitch-agent
```


###5.1.3 Cấu hình các dịch vụ neutron
Đầu tiên, ta cấu hình file ```/etc/neutron/neutron.conf```
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
verbose = True
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
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
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
####Cấu hình Modular Layer 2 (ML2) plug-in
Cấu hình file ``` /etc/neutron/plugins/ml2/ml2_conf.ini ```
```sh
[ml2]
type_drivers = flat,vlan,gre,vxlan
tenant_network_types = vxlan
mechanism_drivers = openvswitch,l2population
extension_drivers = port_security

[ml2_type_flat]
flat_networks = external

[ml2_type_vlan]
network_vlan_ranges = external

[ml2_type_gre]
tunnel_id_ranges = 1:1000

[ml2_type_vxlan]
vni_ranges = 1:1000

[securitygroup]
enable_ipset = True
```

Cấu hình file ``` /etc/neutron/plugins/ml2/openvswitch_agent.ini```
```sh
[ovs]
local_ip = 10.10.10.10
bridge_mappings = external:br-ex

[agent]
tunnel_types = gre,vxlan
l2_population = True
prevent_arp_spoofing = True

[securitygroup]
firewall_driver = neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver
enable_security_group = True
```
Cấu hình L3 agent trong file ```/etc/neutron/l3_agent.ini```
```sh
[DEFAULT]
verbose = True
interface_driver = neutron.agent.linux.interface.OVSInterfaceDriver
use_namespaces = True
external_network_bridge =
```
Cấu hình DHCP agent trong file ```/etc/neutron/dhcp_agent.ini ```
```sh
[DEFAULT]
verbose = True
interface_driver = neutron.agent.linux.interface.OVSInterfaceDriver
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = True
```
Cấu hình metadata agent trong file ```/etc/neutron/metadata_agent.ini```
```sh
[DEFAULT]
verbose = True
nova_metadata_ip = controller
metadata_proxy_shared_secret = 1111
```
Tạo  br-ex, kết nối br-ex tới eth1
```sh
sudo /usr/share/openvswitch/scripts/ovs-ctl start
ovs-vsctl add-br br-ex
ovs-vsctl add-port br-ex eth1
```
#### Cấu hình nova để sử dụng neutron và metadata agent.
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
password = 1111
	
service_metadata_proxy = True
metadata_proxy_shared_secret = 1111	
```
Đồng bộ hóa cơ sở dữ liệu:
```sh
su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
	--config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron 
```
Khởi động lại các service:
```sh
sudo service nova-api restart
sudo service neutron-server restart
sudo service neutron-openvswitch-agent restart
sudo service neutron-dhcp-agent restart
sudo service neutron-metadata-agent restart
sudo service neutron-l3-agent restart
```
##5.2 Cấu hình trên compute node
###5.2.1 Chuẩn bị gói cài đặt trên compute node
```sh
sudo apt-get install neutron-plugin-openvswitch-agent
```
###5.2.2 Cấu hình cài đặt neutron trên compute node
Cấu hình file ```/etc/neutron/neutron.conf```
```sh
[DEFAULT]
verbose = True
[DEFAULT]
...
rpc_backend = rabbit

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
####Cấu hình ovs-agent
Cấu hình file ``` /etc/neutron/plugins/ml2/openvswitch_agent.ini```
```sh
[ovs]
local_ip = 10.10.10.11

bridge_mappings = external:br-ex

[agent]
tunnel_types = gre,vxlan
l2_population = True
prevent_arp_spoofing = True

[securitygroup]
firewall_driver = neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver
enable_security_group = True
```
Tạo br-vlan và br-ex, kết nối br-vlan tới eth2 và br-ex tới eth1
```sh
sudo /usr/share/openvswitch/scripts/ovs-ctl start
ovs-vsctl add-br br-ex
ovs-vsctl add-port br-ex eth1
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


###5.2.3 Kết thúc cài đặt trên compute node
- Khởi động lại dịch vụ nova-compute
```sh
sudo service nova-compute restart
```
- Khởi động lại linux-bridge agent
```sh
sudo service neutron-openvswitch-agent restart
```
##5.3 Kiểm tra hoạt động của dịch vụ neutron
Trên controller node, nhập file xác thực admin.sh
```sh
	source admin.sh
```
Kiểm tra xem các agent đã được bật đầy đủ hay chưa 
```sh
	neutron agent-list
```

Tạo thử các mạng ảo:

```sh
neutron net-create --shared --provider:physical_network external  --provider:network_type vlan --provider:segmentation_id 101 provider-vlan1 
neutron subnet-create --name ext-subnet-vlan1 --allocation-pool start=192.168.8.100,end=192.168.8.150 --dns-nameserver 8.8.4.4 --gateway 192.168.8.1 provider-vlan1 192.168.8.0/24
```
#6 Cài đặt Horizon
##6.1 Chuẩn bị
Cài đặt gói horizon
```sh
# apt-get install openstack-dashboard
```
##6.2 Cấu hình horizon
Chỉnh sửa file ```/etc/openstack-dashboard/local_settings.py```
```sh
OPENSTACK_HOST = "controller"
ALLOWED_HOSTS = ['*', ]
CACHES = {
    'default': {
         'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
         'LOCATION': 'controller:11211',
    }
}
OPENSTACK_KEYSTONE_URL = "http://%s:5000/v3" % OPENSTACK_HOST
OPENSTACK_API_VERSIONS = {
    "identity": 3,
    "image": 2,
    "volume": 2,
}
OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = "default"
OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"
```
Xóa theme mặc định của ubuntu
```sh
apt-get -y remove --auto-remove openstack-dashboard-ubuntu-theme
```
Khởi động lại apache2 service
```sh
service apache2 restart
```
Mở trang web ```192.168.2.10/horizon```để kiểm tra horizon service

#7 Cài đặt máy ảo
##7.1 Thiết lập mạng ảo
Để thiết lập hệ thống mạng ảo, đầu tiên ta cần tạo ra 1 mạng external network để cung cấp cho tất cả các project khả năng kết nối internet
- Ta tạo 1 mạng provider network:
```sh
neutron net-create --shared --provider:physical_network external  --provider:network_type flat  provider-net
```
-Cấp subnet cho provider network và cho phép cắm router lên provider network
```sh
neutron subnet-create --name provider-subnet --allocation-pool start=192.168.2.100,end=192.168.2.150 --dns-nameserver 8.8.4.4 --gateway 192.168.2.1 provider-net 192.168.2.0/24

neutron net-update provider-net --router:external
```
Ip ở dải subnet nên là Ip lấy trong dải external network ở phần đầu

- Tạo 1 mạng private cho project và cấp subnet cho mạng private đó
```sh
neutron net-create private-net1
neutron subnet-create --name private-subnet1  --dns-nameserver 8.8.4.4 --gateway 10.10.60.1 private-net1 10.10.60.0/24
```

- Tạo router và kết nối router với provider network
```sh
neutron router-create private-router
neutron router-gateway-set private-router provider-net
```

- Kết nối router với subnet vừa tạo ra trên mạng private-net1
```sh
neutron router-interface-add private-router private-subnet1
```
##7.2 Tạo máy ảo
- Tạo flavor nano
```sh
openstack flavor create --id 0 --vcpus 1 --ram 64 --disk 1 m1.nano
```
- Thêm rule cho default security group
```sh
openstack security group rule create --proto icmp default
openstack security group rule create --proto tcp --dst-port 22 default
```
Kiểm tra các mạng hiện tại có thể sử dụng trên project
```sh
openstack network list
+--------------------------------------+--------------+--------------------------------------+
| ID                                   | Name         | Subnets                              |
+--------------------------------------+--------------+--------------------------------------+
| ec45b68b-4927-461e-96f3-d68a4568be92 | provider-net | d0331e7d-cf89-49a9-ae3d-6ea3c8a39055 |
| 98eab2e2-0f09-47d9-9265-eb5db901630f | private-net1 | b54062a8-5fff-4930-a97d-4cbc2fa20c91 |
+--------------------------------------+--------------+--------------------------------------+

```
- Tạo một máy ảo
```
openstack server create --flavor m1.nano --image cirros --nic net-id=98eab2e2-0f09-47d9-9265-eb5db901630f --security-group default test1
```
Lưu ý là ở đây net-id là id của private-net1, khi các bạn sử dụng câu lệnh trên cần sử dụng net-id trên máy của các bạn.

- Kiểm tra trạng thái của máy ảo
```
openstack server list
+--------------------------------------+-------+--------+-------------------------+
| ID                                   | Name  | Status | Networks                |
+--------------------------------------+-------+--------+-------------------------+
| 84f53062-539b-4cec-932b-73d531624941 | test1 | ACTIVE | private-net1=10.10.60.3 |
+--------------------------------------+-------+--------+-------------------------+
```
- Chạy thử máy ảo

Truy cập vào horizon->Compute->instance, click vào instance bạn vừa mới tạo ở bước trước và chọn console, bạn sẽ tương tác được với máy ảo thông qua trình duyệt
![create-instance-success.png](./img/create-instance-success.png)

End.
