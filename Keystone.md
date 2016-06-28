## Hướng dẫn cài đặt OpenStack từ docs.

### Mô hình cài đặt
- Lưu ý:
 - Sử dụng mô hình cài đặt thu gọn gồm 1 node Controller và nhiều node Compute
 
![Topo-liberty](/images/Topo-OpenStack-Liberty.jpg)

### Các thiết lập cơ bản

- Các thao tác được thực hiện với tài khoản root.
- Chạy các lệnh dưới ngay sau khi cài đặt xong máy ảo.
- Mô hình phải đảm bảo cấu hình đúng dải IP ở trên.
- Phiên bản hệ điều hành cho các máy là Ubuntu Server 14.04-x 64 bit
```sh
root@controller:~# lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 14.04.3 LTS
Release:        14.04
Codename:       trusty
```


#### Node Controller

- Khai báo các gói để cài đặt OpenStack Liberty

```sh
apt-get -y install software-properties-common
add-apt-repository -y cloud-archive:liberty 
```

##### Thiết lập IP, hostname

- Bạn có thể sửa IP phù hợp với máy bạn, tốt nhất là sử dụng theo IP mà chúng tôi hướng dẫn.
- Thiết lập hostname với tên là `controller` 

```sh 
echo "controller" > /etc/hostname
hostname -F /etc/hostname
```

Thiết lập địa chỉ IP

- Sao lưu file cấu hình của card mạng

`
cp /etc/network/interfaces /etc/network/interfaces.bak
`

- Sử dụng script dưới để cấu hình IP tĩnh cho card mạng.

```sh
cat << EOF > /etc/network/interfaces

# NIC loopback
auto lo
iface lo inet loopback

# NIC MGNG
auto eth0
iface eth0 inet static
address 10.10.10.120
netmask 255.255.255.0

# NIC EXTERNAL
auto eth1
iface eth1 inet static
address 192.168.1.120
netmask 255.255.255.0
gateway 192.168.1.1
dns-nameservers 8.8.8.8


EOF

```

Cấu hình file /etc/hosts để phân giản IP cho các node

```sh
cat << EOF > /etc/hosts 
127.0.0.1   controller localhost
10.10.10.120    controller
10.10.10.121    compute1

EOF
```

Update và khởi động lại node `controller`

```sh
apt-get update -y && apt-get upgrade -y && apt-get dist-upgrade -y && init 6
```

Đăng nhập với IP mới của node controller

### Cài đăt các gói phần mềm

#### Cài đặt gói OpenStack Client

```sh
apt-get -y install python-openstackclient
```

#### Cài đặt My SQL

Trong quá trình cài đặt yêu cầu nhập mật khẩu My SQL, sử dụng mật khẩu Welcome123 để thống nhất.

```sh
apt-get -y install mariadb-server python-pymysql
```

Tạo file với nội dung sau

```sh
cat << EOF  > /etc/mysql/conf.d/mysqld_openstack.cnf

[mysqld]
bind-address = 10.10.10.120

[mysqld]
default-storage-engine = innodb
innodb_file_per_table
collation-server = utf8_general_ci
init-connect = 'SET NAMES utf8'
character-set-server = utf8
EOF

``` 

Khởi động lại MYSQL

```sh
service mysql restart
```

#### Cài đặt Message queue

Cài đặt gói rabbitmq

```sh
apt-get -y install rabbitmq-server
```

Tạo tài khoản `openstack` cho rabbitmq

```sh 
rabbitmqctl add_user openstack Welcome123
```

Cấp quyền cho tài khoản openstack 

```sh
rabbitmqctl set_permissions openstack ".*" ".*" ".*"
```

#### Cài đặt dịch vụ Keystone

##### Tạo database cho keystone

Đăng nhập vào MariaDB

```sh
mysql -u root -pWelcome123
```

Tạo DB tên là keystone và gán quyền

```sh
CREATE DATABASE keystone;

GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
IDENTIFIED BY 'Welcome123';

GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
IDENTIFIED BY 'Welcome123';
flush privileges;

quit

```

##### Cài đặt Keystone

Cấu hình không cho Keystone tự động khởi động.

```sh
echo "manual" > /etc/init/keystone.override
```

Cài đặt các gói dành cho Keystone

```sh
apt-get -y install keystone apache2 libapache2-mod-wsgi memcached python-memcache
```

- Sao lưu file cấu hình của keystone.
```sh
cp /etc/keystone/keystone.conf /etc/keystone/keystone.conf.bak
```

- Xóa file keystone gốc 
```sh
rm /etc/keystone/keystone.conf
```


- Tạo file keystone mới bằng lệnh `vi /etc/keystone/keystone.conf` chứa nội dung dưới.

```sh 

[DEFAULT]
log_dir = /var/log/keystone

admin_token = Welcome123
public_bind_host = 10.10.10.120
admin_bind_host = 10.10.10.120

[assignment]
[auth]
[cache]
[catalog]
[cors]
[cors.subdomain]
[credential]
[database]
connection = mysql+pymysql://keystone:Welcome123@10.10.10.120/keystone


[domain_config]
[endpoint_filter]
[endpoint_policy]
[eventlet_server]
[eventlet_server_ssl]
[federation]
[fernet_tokens]
[identity]
[identity_mapping]
[kvs]
[ldap]
[matchmaker_redis]
[matchmaker_ring]
[memcache]
servers = localhost:11211


[oauth1]
[os_inherit]
[oslo_messaging_amqp]
[oslo_messaging_qpid]
[oslo_messaging_rabbit]
[oslo_middleware]
[oslo_policy]
[paste_deploy]
[policy]
[resource]
[revoke]
driver = sql

[role]
[saml]
[signing]
[ssl]
[token]
provider = uuid
driver = memcache

[tokenless_auth]
[trust]
[extra_headers]
Distribution = Ubuntu

```



Đồng bộ database cho keystone

```sh
su -s /bin/sh -c "keystone-manage db_sync" keystone
```

- Cấu hình apache cho Keystone

```sh
echo "ServerName 10.10.10.164" > /etc/apache2/conf-available/servername.conf
```

Tạo file `/etc/apache2/sites-available/wsgi-keystone.conf` với nội dung sau


```sh
Listen 5000
Listen 35357

<VirtualHost *:5000>
    WSGIDaemonProcess keystone-public processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-public
    WSGIScriptAlias / /usr/bin/keystone-wsgi-public
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    <IfVersion >= 2.4>
      ErrorLogFormat "%{cu}t %M"
    </IfVersion>
    ErrorLog /var/log/apache2/keystone.log
    CustomLog /var/log/apache2/keystone_access.log combined

    <Directory /usr/bin>
        <IfVersion >= 2.4>
            Require all granted
        </IfVersion>
        <IfVersion < 2.4>
            Order allow,deny
            Allow from all
        </IfVersion>
    </Directory>
</VirtualHost>

<VirtualHost *:35357>
    WSGIDaemonProcess keystone-admin processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-admin
    WSGIScriptAlias / /usr/bin/keystone-wsgi-admin
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    <IfVersion >= 2.4>
      ErrorLogFormat "%{cu}t %M"
    </IfVersion>
    ErrorLog /var/log/apache2/keystone.log
    CustomLog /var/log/apache2/keystone_access.log combined

    <Directory /usr/bin>
        <IfVersion >= 2.4>
            Require all granted
        </IfVersion>
        <IfVersion < 2.4>
            Order allow,deny
            Allow from all
        </IfVersion>
    </Directory>
</VirtualHost>

```

Cấu hình virtual host cho keystone

```sh 
ln -s /etc/apache2/sites-available/wsgi-keystone.conf /etc/apache2/sites-enabled
```

Khởi động lại apache

```sh
service apache2 restart
```

Xóa SQLite mặc định của keystone

```sh
rm -f /var/lib/keystone/keystone.db
```

Khai báo biến môi trường để cài đặt KeyStone

```sh
export OS_TOKEN=Welcome123
export OS_URL=http://10.10.10.164:35357/v3
export OS_IDENTITY_API_VERSION=3
```

Tạo user, endpoint, role, tenant cho Keystone

```sh
openstack service create --name keystone --description "OpenStack Identity" identity
openstack endpoint create --region RegionOne identity public http://10.10.10.120:5000/v2.0
openstack endpoint create --region RegionOne identity internal http://10.10.10.120:5000/v2.0
openstack endpoint create --region RegionOne identity admin http://10.10.10.120:35357/v2.0

openstack project create --domain default --description "Admin Project" admin
openstack user create  --domain default --password Welcome123 admin
openstack role create admin
openstack role add --project admin --user admin admin
openstack project create --domain default --description "Service Project" service
openstack project create --domain default --description "Demo Project" demo
openstack user create --domain default --password Welcome123 demo
openstack role create user
openstack role add --project demo --user demo user
```

- Hủy 02 biến môi trường đã khai báo trước đó
```sh
unset OS_TOKEN OS_URL
```

- Tạo file admin.sh với nội dung dưới bằng lệnh `vi admin.sh`

```sh
export OS_PROJECT_DOMAIN_ID=default
export OS_USER_DOMAIN_ID=default
export OS_PROJECT_NAME=admin
export OS_TENANT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=Welcome123
export OS_AUTH_URL=http://10.10.10.120:35357/v3
export OS_IDENTITY_API_VERSION=3
```

- Phân quyền cho file `admin.sh`
```sh
chmod +x admin.sh
```

- Chạy lênh dưới để khai báo biến môi trường
```sh
source admin.sh
```

- Kiểm tra xem keystone hoạt động tốt hay chưa bằng lệnh
```sh
openstack token issue
```

- Kết quả tương tự như dưới
```sh
+------------+----------------------------------+
| Field      | Value                            |
+------------+----------------------------------+
| expires    | 2015-11-17T09:53:40.242778Z      |
| id         | de796ac24b2545efb99487d9ff4e981a |
| project_id | c685a5fa3e474261b678aeb59332ce0d |
| user_id    | 818e335d15484101b6a2a69e5f9d4f61 |
+------------+----------------------------------+
```

##### Cài đặt GLANCE 
- Glance chỉ cần cài đặt trên Controller
- Tạo database và phân quyền bằng các lệnh dưới
```sh
mysql -u root -pWelcome123

CREATE DATABASE glance;
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY 'Welcome123';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'Welcome123';
quit;
```

- Tạo user, endpoint, gán role cho glance trong keystone
```sh
openstack user create --domain default --password Welcome123 glance
openstack role add --project service --user glance admin
openstack service create --name glance --description "OpenStack Image service" image

openstack endpoint create --region RegionOne image public http://10.10.10.120:9292
openstack endpoint create --region RegionOne image internal http://10.10.10.120:9292
openstack endpoint create --region RegionOne image admin http://10.10.10.120:9292
```

- Cài đặt các gói trong glance
```sh
apt-get -y install glance python-glanceclient
```

- Sao lưu file cấu hình gốc của glance 
```sh
cp /etc/glance/glance-api.conf /etc/glance/glance-api.conf.bak
```

- Xóa file glance gốc 
```sh
rm /etc/glance/glance-api.conf 
```

- Tạo file `glance-api.conf` với bằng lệnh `vi /etc/glance/glance-api.conf` với nội dung sau

```sh
[DEFAULT]
notification_driver = noop
verbose = True

[database]
connection = mysql+pymysql://glance:Welcome123@10.10.10.120/glance
backend = sqlalchemy

[glance_store]
default_store = file
filesystem_store_datadir = /var/lib/glance/images/

[image_format]
[keystone_authtoken]
auth_uri = http://10.10.10.120:5000
auth_url = http://10.10.10.120:35357
auth_plugin = password
project_domain_id = default
user_domain_id = default
project_name = service
username = glance
password = Welcome123


[matchmaker_redis]
[matchmaker_ring]
[oslo_concurrency]
[oslo_messaging_amqp]
[oslo_messaging_qpid]
[oslo_messaging_rabbit]
[oslo_policy]
[paste_deploy]
flavor = keystone

[store_type_location_strategy]
[task]
[taskflow_executor]
```
- Sao lưu file `/etc/glance/glance-registry.conf`
```sh
cp /etc/glance/glance-registry.conf /etc/glance/glance-registry.conf.bak
```

- Xóa file gốc `/etc/glance/glance-registry.conf`
```sh
rm /etc/glance/glance-registry.conf
```

- Tạo file mới bằng lệnh `vi /etc/glance/glance-registry.conf` với nội dung sau:
```

[DEFAULT]
notification_driver = noop
verbose = True


[database]
connection = mysql+pymysql://glance:Welcome123@10.10.10.120/glance
backend = sqlalchemy

[glance_store]

[keystone_authtoken]
auth_uri = http://10.10.10.120:5000
auth_url = http://10.10.10.120:35357
auth_plugin = password
project_domain_id = default
user_domain_id = default
project_name = service
username = glance
password = Welcome123


[matchmaker_redis]
[matchmaker_ring]
[oslo_messaging_amqp]
[oslo_messaging_qpid]
[oslo_messaging_rabbit]
[oslo_policy]
[paste_deploy]
flavor = keystone

```

- Đồng bộ database cho Glance
```sh
su -s /bin/sh -c "glance-manage db_sync" glance
```

- Xóa file SQLite mặc định
```sh
rm -f /var/lib/glance/glance.sqlite
```

- Khai báo thêm biến môi trường cho Glance
```sh
echo "export OS_IMAGE_API_VERSION=2" | tee -a admin.sh
source admin.sh
```

- Tải image `cirros` và up image cho glance
```sh
wget http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img

glance image-create --name "cirros" \
--file cirros-0.3.4-x86_64-disk.img \
--disk-format qcow2 --container-format bare \
--visibility public --progress
```

#### Cài đặt NOVA
#### Cài đặt NOVA trên CONTROLLER
##### Prerequisites
- Tạo database cho NOVA

```sh
mysql -u root -pWelcome123

CREATE DATABASE nova;
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY 'Welcome123';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY 'Welcome123';
exit
```


- Tạo user, gán role, endpoint cho nova

```sh
openstack user create --domain default --password Welcome123 nova

openstack role add --project service --user nova admin


openstack service create --name nova --description "OpenStack Compute" compute

openstack endpoint create --region RegionOne compute public http://10.10.10.120:8774/v2/%\(tenant_id\)s
openstack endpoint create --region RegionOne compute internal http://10.10.10.120:8774/v2/%\(tenant_id\)s
openstack endpoint create --region RegionOne compute admin http://10.10.10.120:8774/v2/%\(tenant_id\)s

```

#### Cài đặt các gói nova trên Controller
```sh
apt-get -y install nova-api nova-cert nova-conductor nova-consoleauth nova-novncproxy nova-scheduler python-novaclient
```

- Sao lưu file `/etc/nova/nova.conf`

```sh
cp /etc/nova/nova.conf  /etc/nova/nova.conf.bak
```

- Xóa file gốc 
```sh
rm /etc/nova/nova.conf
```

- Tạo file nova.conf với lệnh `vi /etc/nova/nova.conf` với nội dung sau:
```sh
[DEFAULT]

rpc_backend = rabbit
auth_strategy = keystone

dhcpbridge_flagfile=/etc/nova/nova.conf
dhcpbridge=/usr/bin/nova-dhcpbridge
logdir=/var/log/nova
state_path=/var/lib/nova
lock_path=/var/lock/nova
force_dhcp_release=True
libvirt_use_virtio_for_bridges=True
ec2_private_dns_show_ip=True
api_paste_config=/etc/nova/api-paste.ini
enabled_apis=ec2,osapi_compute,metadata

my_ip = 10.10.10.120

network_api_class = nova.network.neutronv2.api.API
security_group_api = neutron
linuxnet_interface_driver = nova.network.linux_net.NeutronLinuxBridgeInterfaceDriver
firewall_driver = nova.virt.firewall.NoopFirewallDriver

enabled_apis=osapi_compute,metadata
verbose = True


[database]
connection = mysql+pymysql://nova:Welcome123@10.10.10.120/nova

[oslo_messaging_rabbit]
rabbit_host = 10.10.10.120
rabbit_userid = openstack
rabbit_password = Welcome123

[keystone_authtoken]
auth_uri = http://10.10.10.120:5000
auth_url = http://10.10.10.120:35357
auth_plugin = password
project_domain_id = default
user_domain_id = default
project_name = service
username = nova
password = Welcome123

[vnc]
vncserver_listen = $my_ip
vncserver_proxyclient_address = $my_ip

[glance]
host = 10.10.10.120

[oslo_concurrency]
lock_path = /var/lib/nova/tmp

```

- Đồng bộ database cho NOVA

```sh
su -s /bin/sh -c "nova-manage db sync" nova
```

- Khởi động lại các dịch vụ của NOVA trên Controller Node
``sh
service nova-api restart
service nova-cert restart
service nova-consoleauth restart
service nova-scheduler restart
service nova-conductor restart
service nova-novncproxy restart
```

- Xóa file SQLite mặc định
```sh
rm -f /var/lib/nova/nova.sqlite
```

#### Cài đặt trên NOVA trên COMPUTE NODE
- Khai báo các gói để cài đặt OpenStack Liberty

apt-get -y install software-properties-common
add-apt-repository -y cloud-archive:liberty 

- Thiết lập IP
```sh
cat << EOF > /etc/network/interfaces

# NIC loopback
auto lo
iface lo inet loopback

# NIC MGNG
auto eth0
iface eth0 inet static
address 10.10.10.121
netmask 255.255.255.0

# NIC EXTERNAL
auto eth1
iface eth1 inet static
address 192.168.1.121
netmask 255.255.255.0
gateway 192.168.1.1
dns-nameservers 8.8.8.8
EOF

```
- Thiết lập hostname

Cấu hình file /etc/hosts để phân giản IP cho các node

```sh
cat << EOF > /etc/hosts 
127.0.0.1   controller localhost
10.10.10.120    controller
10.10.10.121    compute1

EOF
```

Update và khởi động lại node `Compute node`

```sh
apt-get update -y && apt-get upgrade -y && apt-get dist-upgrade -y && init 6
```


- Đăng nhập lại vào compute node và thực hiện các cài đặt tiếp theo


- Cài đặt gói the OpenStack client
```sh
apt-get -y install python-openstackclient
```
- Caì đặt gói cho `nova-compute`
```sh
apt-get -y install nova-compute sysfsutils
```

- Sao lưu file config cho nova
```sh
cp /etc/nova/nova.conf /etc/nova/nova.conf.bak
```

- Sửa file `vi /etc/nova/nova.conf file` với nội dung dưới

```sh
[DEFAULT]
dhcpbridge_flagfile=/etc/nova/nova.conf
dhcpbridge=/usr/bin/nova-dhcpbridge
logdir=/var/log/nova
state_path=/var/lib/nova
lock_path=/var/lock/nova
force_dhcp_release=True
libvirt_use_virtio_for_bridges=True
verbose=True
ec2_private_dns_show_ip=True
api_paste_config=/etc/nova/api-paste.ini
enabled_apis=ec2,osapi_compute,metadata

rpc_backend = rabbit
auth_strategy = keystone
my_ip = 10.10.10.121

network_api_class = nova.network.neutronv2.api.API
security_group_api = neutron
linuxnet_interface_driver = nova.network.linux_net.NeutronLinuxBridgeInterfaceDriver
firewall_driver = nova.virt.firewall.NoopFirewallDriver

verbose = True


[oslo_messaging_rabbit]
rabbit_host = 10.10.10.120
rabbit_userid = openstack
rabbit_password = Welcome123

[keystone_authtoken]
auth_uri = http://10.10.10.120:5000
auth_url = http://10.10.10.120:35357
auth_plugin = password
project_domain_id = default
user_domain_id = default
project_name = service
username = nova
password = Welcome123



[vnc]
enabled = True
vncserver_listen = 0.0.0.0
vncserver_proxyclient_address = $my_ip
novncproxy_base_url = http://192.168.1.120:6080/vnc_auto.html

[glance]
host = 10.10.10.120

[oslo_concurrency]
lock_path = /var/lib/nova/tmp
```

- Khởi động lại `nova-compute` trên Controller
```sh
service nova-compute restart
```

- Xóa file SQLite mặc định
```sh
rm -f /var/lib/nova/nova.sqlite
```
-  Kiểm tra lại dịch vụ của NOVA trên Controller 
```sh
nova service-list
``

- Kết quả sẽ như sau:
```sh
+----+------------------+------------+----------+---------+-------+----------------------------+-----------------+
| Id | Binary           | Host       | Zone     | Status  | State | Updated_at                 | Disabled Reason |
+----+------------------+------------+----------+---------+-------+----------------------------+-----------------+
| 1  | nova-cert        | controller | internal | enabled | up    | 2015-11-17T10:00:41.000000 | -               |
| 2  | nova-consoleauth | controller | internal | enabled | up    | 2015-11-17T10:00:44.000000 | -               |
| 3  | nova-scheduler   | controller | internal | enabled | up    | 2015-11-17T10:00:44.000000 | -               |
| 4  | nova-conductor   | controller | internal | enabled | up    | 2015-11-17T10:00:36.000000 | -               |
| 6  | nova-compute     | compute1   | nova     | enabled | up    | 2015-11-17T10:00:44.000000 | -               |
+----+------------------+------------+----------+---------+-------+----------------------------+-----------------+

```


### Cài đặt Neutron trên Controller Node
- Tạo database cho Neutron
```sh
mysql -u root -pWelcome123

CREATE DATABASE neutron;
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'Welcome123';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'Welcome123';
exit;
```


- Tạo user, gán role, endpoint cho neutron

```sh

openstack user create --domain default --password Welcome123 neutron

openstack role add --project service --user neutron admin

openstack service create --name neutron --description "OpenStack Networking" network

openstack endpoint create --region RegionOne network public http://10.10.10.120:9696
openstack endpoint create --region RegionOne network internal http://10.10.10.120:9696
openstack endpoint create --region RegionOne network admin http://10.10.10.120:9696

```

- Cài đặt các thành phần cho NEUTRON trên Controller Node

```sh
apt-get -y install neutron-server neutron-plugin-ml2 \
neutron-plugin-linuxbridge-agent neutron-l3-agent neutron-dhcp-agent \
neutron-metadata-agent python-neutronclient
```

- Sao lưu file cấu hình

```sh
cp /etc/neutron/neutron.conf /etc/neutron/neutron.conf.bak

```

- Xóa file `/etc/neutron/neutron.conf`

```sh
rm /etc/neutron/neutron.conf
```

- Tạo file neutron.conf với lệnh `vi /etc/neutron/neutron.conf` chứa nội dung sau

```sh
[DEFAULT]
core_plugin = ml2
service_plugins = router
allow_overlapping_ips = True
rpc_backend = rabbit

auth_strategy = keystone

notify_nova_on_port_status_changes = True
notify_nova_on_port_data_changes = True
nova_url = http://10.10.10.120:8774/v2

verbose = True

[matchmaker_redis]
[matchmaker_ring]
[quotas]
[agent]
root_helper = sudo /usr/bin/neutron-rootwrap /etc/neutron/rootwrap.conf

[keystone_authtoken]
auth_uri = http://10.10.10.120:5000
auth_url = http://10.10.10.120:35357
auth_plugin = password
project_domain_id = default
user_domain_id = default
project_name = service
username = neutron
password = Welcome123


[database]
connection = mysql+pymysql://neutron:Welcome123@10.10.10.120/neutron


[nova]
auth_url = http://10.10.10.120:35357
auth_plugin = password
project_domain_id = default
user_domain_id = default
region_name = RegionOne
project_name = service
username = nova
password = Welcome123

[oslo_concurrency]
lock_path = $state_path/lock
[oslo_policy]
[oslo_messaging_amqp]
[oslo_messaging_qpid]

[oslo_messaging_rabbit]
rabbit_host = 10.10.10.120
rabbit_userid = openstack
rabbit_password = Welcome123

[qos]
```

- Cấu hình cho (ML2) plug-in
```sh
cp /etc/neutron/plugins/ml2/ml2_conf.ini  /etc/neutron/plugins/ml2/ml2_conf.ini.bak
```

- Sửa file /etc/neutron/plugins/ml2/ml2_conf.ini  với nội dung sau

```
[ml2]
tenant_network_types = vxlan
type_drivers = flat,vlan,vxlan
mechanism_drivers = linuxbridge,l2population
extension_drivers = port_security


[ml2_type_flat]
flat_networks = public

[ml2_type_vlan]

[ml2_type_gre]
[ml2_type_vxlan]
vni_ranges = 1:1000

[ml2_type_geneve]
[securitygroup]
enable_ipset = True

```

#### Configure the Linux bridge agent
- Sao lưu cấu hình cho file `linuxbridge_agent.ini `

```sh
cp /etc/neutron/plugins/ml2/linuxbridge_agent.ini /etc/neutron/plugins/ml2/linuxbridge_agent.ini.bak

- Sửa file linuxbridge_agent.ini  bằng lệnh `/etc/neutron/plugins/ml2/linuxbridge_agent.ini` với nội dung dưới


```sh

[linux_bridge]
physical_interface_mappings = public:eth1

[vxlan]
enable_vxlan = True
local_ip = 10.10.10.120
l2_population = True


[agent]
prevent_arp_spoofing = True


[securitygroup]
enable_security_group = True
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver

```

### Cấu hình cho layer-3 agent

- Sao lưu file cấu hình
```sh
cp /etc/neutron/l3_agent.ini /etc/neutron/l3_agent.ini.bak
```

Xóa file `/etc/neutron/l3_agent.ini`
```sh
rm /etc/neutron/l3_agent.ini
```
-Sửa file  /etc/neutron/l3_agent.ini bằng lệnh `vi /etc/neutron/l3_agent.ini` với nội dung dưới


```sh

[DEFAULT]
interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver
external_network_bridge =
verbose = True


[AGENT]

```

#### Cấu hình  DHCP agent
- Sao lưu file `dhcp_agent.ini` 
```sh
cp /etc/neutron/dhcp_agent.ini /etc/neutron/dhcp_agent.ini.bak
```

- Sửa file `dhcp_agent.ini` bằng lệnh `vi /etc/neutron/dhcp_agent.ini` với nội dung dưới

```sh

[DEFAULT]
interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = True

verbose = True
dnsmasq_config_file = /etc/neutron/dnsmasq-neutron.conf

[AGENT]
```


- Tạo file `vi /etc/neutron/dnsmasq-neutron.conf` với nội dung sau:

```sh
echo "dhcp-option-force=26,1450" > /etc/neutron/dnsmasq-neutron.conf 
```

- Cấu hình metadata agent
- Sao lưu file ` cp /etc/neutron/metadata_agent.ini`

```sh
cp /etc/neutron/metadata_agent.ini /etc/neutron/metadata_agent.ini.bak
```

-Sửa file sau với lệnh `vi /etc/neutron/metadata_agent.ini`  chứa nội dung dưới

```sh
DEFAULT]
auth_uri = http://10.10.10.120:5000
auth_url = http://10.10.10.120:35357
auth_region = RegionOne
auth_plugin = password
project_domain_id = default
user_domain_id = default
project_name = service
username = neutron
password = Welcome123

nova_metadata_ip = 10.10.10.120

metadata_proxy_shared_secret = Welcome123
verbose = True
```

- Thêm vào file /etc/nova/nova.conf  trên node Controller đoạn dưới cùng dưới

```
[neutron]
url = http://10.10.10.120:9696
auth_url = http://10.10.10.120:35357
auth_plugin = password
project_domain_id = default
user_domain_id = default
region_name = RegionOne
project_name = service
username = neutron
password = Welcome123

service_metadata_proxy = True
metadata_proxy_shared_secret = Welcome123
```

- Đồng bộ database cho NVOA

```sh
su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
  --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
```

- Khởi động lại `nova-api`
```sh
service nova-api restart
```

- Khởi động lại các dịch vụ của NEUTRON trên CONTROLLER NODE

```sh
service neutron-server restart
service neutron-plugin-linuxbridge-agent restart
service neutron-dhcp-agent restart
service neutron-metadata-agent restart
service neutron-l3-agent restart
```

-  Xóa file SQLite mặc định của OpenStack
```sh
rm -f /var/lib/neutron/neutron.sqlite
```
#### Cài đặt thành phần của neutron trên COMPUTE NODE
- Cài đặt `linuxbridge-agent` trên node Compute
```sh
apt-get -y install neutron-plugin-linuxbridge-agent
```

- Sao lưu file  `/etc/neutron/neutron.conf`
```sh
cp /etc/neutron/neutron.conf /etc/neutron/neutron.conf.bak
```

-  Sửa file với lệnh `vi /etc/neutron/neutron.conf`  chứa nội dung sau.

```sh

[DEFAULT]
core_plugin = ml2

rpc_backend = rabbit
auth_strategy = keystone
verbose = True

[matchmaker_redis]
[matchmaker_ring]
[quotas]
[agent]
root_helper = sudo /usr/bin/neutron-rootwrap /etc/neutron/rootwrap.conf

[keystone_authtoken]
auth_uri = http://10.10.10.120:5000
auth_url = http://10.10.10.120:35357
auth_plugin = password
project_domain_id = default
user_domain_id = default
project_name = service
username = neutron
password = Welcome123


[database]
# connection = sqlite:////var/lib/neutron/neutron.sqlite

[nova]
[oslo_concurrency]
lock_path = $state_path/lock
[oslo_policy]
[oslo_messaging_amqp]
[oslo_messaging_qpid]

[oslo_messaging_rabbit]
rabbit_host = 10.10.10.120
rabbit_userid = openstack
rabbit_password = Welcome123

[qos]

```

### Configure the Linux bridge agent

- Sao lưu file `/etc/neutron/plugins/ml2/linuxbridge_agent.ini`
```sh
cp /etc/neutron/plugins/ml2/linuxbridge_agent.ini /etc/neutron/plugins/ml2/linuxbridge_agent.ini.bak
```

- Sửa file bằng lệnh `vi /etc/neutron/plugins/ml2/linuxbridge_agent.ini` với nội dung sau:

```sh

[linux_bridge]
physical_interface_mappings = public:eth1

[vxlan]
enable_vxlan = True
local_ip = 10.10.10.121
l2_population = True

[agent]
prevent_arp_spoofing = True

[securitygroup]
enable_security_group = True
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```

- Thêm vào dưới cùng file `/etc/nova/nova.conf` trên Compute node với nội dung dưới

```sh
[neutron]
url = http://10.10.10.120:9696
auth_url = http://10.10.10.120:35357
auth_plugin = password
project_domain_id = default
user_domain_id = default
region_name = RegionOne
project_name = service
username = neutron
password = Welcome123
```

- Khởi động lại nova-compute
```sh
service nova-compute restart
```
- Khởi động lại Linux bridge agent

```sh
service neutron-plugin-linuxbridge-agent restart
```

##### Cai dat dashboad tren CONTROLLER

```sh
apt-get -y install openstack-dashboard
```

- Đăng nhập vào controller với IP `192.168.1.120/horizon`

