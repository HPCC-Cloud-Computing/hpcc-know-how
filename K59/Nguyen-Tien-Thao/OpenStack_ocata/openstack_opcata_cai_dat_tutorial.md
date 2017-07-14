# Cài đặt openstack ocata

## 1. Tổng quát về hệ thống

- Hệ thống gồm 2 máy ảo với cấu hình như sau:

```txt
Máy 1: Đặt tên host là controller1. Cấu hình:

OS: Ubuntu Server 16.04 LTS
Phần cứng: RAM 4GB, HDD 80G, 2 NIC

Máy 2: Đặt tên host là compute1. Cấu hình:

OS: Ubuntu Server 16.04 LTS
Phần cứng: RAM 4GB, HDD 80G, 2 NIC
```

- Cấu hình network của hệ thống như sau

![https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/openstack_ocata/card_mang_openstack_ocata.png](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/openstack_ocata/card_mang_openstack_ocata.png)

## 2. Cài đặt trên node controller

### 2.1. Cài đặt môi trường

### 2.1.1 Configure network interface

- Dùng vim để chỉnh sửa file /etc/network/interface với nội dung như sau:

```sh
auto lo
iface lo inet loopback

#management_net
auto ens3
iface ens3 inet static
	address 10.10.10.11
	gateway 10.10.10.1
	broadcast 10.10.10.255
	dns-nameservers 8.8.8.8 8.8.4.4
	netmask 255.255.255.0

#external_net
auto ens9
iface ens9 inet manual
	up ip link set dev $IFACE up
	down ip link set dev $IFACE down
```

- Khởi động lại hệ thống để áp dụng những thay đổi

### 2.1.2 Configure name resolution

- Set hostname của node là controller bằng cách sử dụng vim chỉnh sửa file /etc/hostname với nội dung sau:

```sh
controller
```

- Chỉnh sửa file /etc/hosts với nội dung như sau:

```sh
127.0.0.1	localhost
127.0.1.1	vthao


#controller
10.10.10.11 controller

#compute
10.10.10.10 compute1

```

### 2.1.3 Kiểm tra lại kết nối

```sh
vthao@controller:~$ ping -c 4 openstack.org
PING openstack.org (162.242.140.107) 56(84) bytes of data.
64 bytes from 162.242.140.107: icmp_seq=1 ttl=49 time=306 ms
64 bytes from 162.242.140.107: icmp_seq=2 ttl=50 time=205 ms
64 bytes from 162.242.140.107: icmp_seq=3 ttl=50 time=251 ms
64 bytes from 162.242.140.107: icmp_seq=4 ttl=50 time=272 ms

```

```sh
vthao@controller:~$ ping -c 4 compute1
PING compute1 (10.10.10.10) 56(84) bytes of data.
64 bytes from compute1 (10.10.10.10): icmp_seq=1 ttl=64 time=1.09 ms
64 bytes from compute1 (10.10.10.10): icmp_seq=2 ttl=64 time=1.52 ms
64 bytes from compute1 (10.10.10.10): icmp_seq=3 ttl=64 time=1.74 ms
64 bytes from compute1 (10.10.10.10): icmp_seq=4 ttl=64 time=1.38 ms

```

### 2.1.4 Cài đặt NTP

- Cài đặt package: `apt install chrony`

- sử dụng vim mở file `/etc/chrony/chrony.conf` và thay thế dòng
`pool 2.debian.pool.ntp.org offline iburst`
bởi các dòng

```sh
server 1.vn.pool.ntp.org iburst
server 0.asia.pool.ntp.org iburst
server 3.asia.pool.ntp.org iburst
```

- Cho phép các node khác kết nối tới chrony daemon trên node controller bằng cách thêm dòng `allow 10.10.10.0/24` vào file `/etc/chrony/chroy/conf`

- Restart NTP service:

    `service chrony restart`

- Kiểm tra lại hoạt động của NTP:

```sh
vthao@controller:~$ chronyc sources
210 Number of sources = 3
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^* time.vng.vn                   2   6   377    19   -163us[ -337us] +/-   89ms
^- 82.200.209.236                2   8   377   209  +7577us[+7116us] +/-  319ms
^- y.ns.gin.ntt.net              2   9   377    17   +187ms[ +187ms] +/-  310ms

```

### 2.1.5 Kích hoạt Openstack repo

```sh
apt install software-properties-common
add-apt-repository cloud-archive:ocata
```

### 2.1.6 Hoàn thành cài đặt

- Cập nhật các gói phần mềm
    `apt update && apt dist-upgrade`

- Cài đặt Openstack client
    `apt install python-openstackclient`

- Khởi động lại máy ảo.

### 2.1.7 Cài đặt SQL

- Cài đặt package:  `apt install mariadb-server python-pymysql`
- Tạo file `/etc/mysql/mariadb.conf.d/99-openstack.cnf` và thêm nội dung sau:

    ```sh
    [mysqld]
    bind-address = 10.10.10.11

    default-storage-engine = innodb
    innodb_file_per_table = on
    max_connections = 4096
    collation-server = utf8_general_ci
    character-set-server = utf8

    ```
- Restart database service
- Để bảo mật cho cơ sở dữ liệu chạy lệnh `mysql_secure_installation`
    Nếu hỏi mật khẩu thì điền `123456` còn lại đồng ý theo các tùy chỉnh mặc định.

- Kiểm tra lại bằng cách sử dụng câu lệnh `mysql` kết quả như sau:

```sh
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 41
Server version: 10.0.29-MariaDB-0ubuntu0.16.04.1 Ubuntu 16.04

Copyright (c) 2000, 2016, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>

```

### 2.1.8 Message queue

- Cài đặt gói : `apt install rabbitmq-server`
- Add user `openstack` với passwork `123456`
    `rabbitmqctl add_user openstack 123456`
- Gán quyền đọc và viết cho user `openstack`:
    `rabbitmqctl set_permissions openstack ".*" ".*" ".*"`

### 2.19 Memcached

- cài đặt package: `apt install memcached python-memcache`
- sử dụng vim mở file /etc/memcached.conf tìm dòng `-l 127.0.0.1` và thay bằng dòng `-l 10.0.0.11`
- Restart memcached service: `service memcached restart`

## 2.2 Cài đặt keystone

### 2.2.1 Tạo database

- Đăng nhập vào cơ sở dữ liệu:
    `mysql`
- Tạo `keystone` database:
   `MariaDB [(none)]> CREATE DATABASE keystone;`
- Cấp quyền truy cập vào cơ sở dữ liệu `keystone`

```sql
MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
IDENTIFIED BY '123456';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
IDENTIFIED BY '123456';
```

- Thoát khỏi sql: `exit`

### 2.2.2 Cài đặt và cấu hình keystone

- Cài đặt package:  `apt install keystone`
- Chỉnh sửa file `/etc/keystone/keystone.conf` cụ thể như sau:

    - Trong section [database] thêm dòng: `connection = mysql+pymysql://keystone:123456@controller/keystone`

    - Trong section [token] thêm vào dòng `provider = fernet`
- Đồng bộ database cho keystone:
    `su -s /bin/sh -c "keystone-manage db_sync" keystone`
- Thiết lập `Fernet` key

```sh
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
```

- Khởi động identity service

```sh
keystone-manage bootstrap --bootstrap-password 123456 \
  --bootstrap-admin-url http://controller:35357/v3/ \
  --bootstrap-internal-url http://controller:5000/v3/ \
  --bootstrap-public-url http://controller:5000/v3/ \
  --bootstrap-region-id RegionOne
```

- Cấu hình apache cho `keystone`:
    - Mở file `/etc/apache2/apache2.conf` và thêm vào dòng `ServerName controller`

- Restar apache service và loại bỏ cơ sở dữ liệu SQLite mặc định:
    - `service apache2 restart`
    - `rm -f /var/lib/keystone/keystone.db`

- Cấu hình cho tài khoản admin

```sh

export OS_USERNAME=admin
export OS_PASSWORD=123456
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://controller:35357/v3
export OS_IDENTITY_API_VERSION=3
```

### 2.2.3 Tạo domain, projects, users, and roles

- Tạo `service` project:

    `openstack project create --domain default \
  --description "Service Project" service`
- Tạo `demo` project:

     `openstack project create --domain default \
  --description "Demo Project" demo`
- Tạo `demo` user:

    `openstack user create --domain default \
  --password-prompt demo`
  
    Nhập password là `123456`
- Tạo role với tên `user`

    `openstack role create user`
- Gán tài khoản `demo` có role là `user` vào project `demo`:

    `openstack role add --project demo --user demo user`

### 2.2.4 Kiểm chứng lại các bước cài đặt keystone

- Vô hiệu hóa cơ chế xác thực bằng token tạm thời trong keysonte bằng cách xóa admin_token_auth trong các section [pipeline:public_api], [pipeline:admin_api] và [pipeline:api_v3] của file /etc/keystone/keystone-paste.ini

- Bỏ thiết lập trong biến môi trường của OS_TOKEN và OS_URL bằng lệnh

    `unset OS_TOKEN OS_URL`
- Với vai trò admin yêu cầu một token xác thực:
    `openstack --os-auth-url http://controller:35357/v3 \
    --os-project-domain-name default --os-user-domain-name default \
    --os-project-name admin --os-username admin token issue`
    Nhập vào mật khẩu: `123456`
- Với vai trò demo yêu cầu một token xác thực:
    `openstack --os-auth-url http://controller:5000/v3 \
  --os-project-domain-name default --os-user-domain-name default \
  --os-project-name demo --os-username demo token issue`
  Nhập vào mật khẩu: `123456`

### 2.2.5 Tạo script biến môi trường

- Tạo file admin-openrc với nội dung:

```sh
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=123456
export OS_AUTH_URL=http://controller:35357/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2

```

- Tạo file demo-openrc với nội dung:

```sh
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=demo
export OS_USERNAME=demo
export OS_PASSWORD=123456
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```

- Sử dụng script: `source  admin-openrc`
- Chạy thử để kiểm tra script:  `openstack token issue`. Nếu kết quả như sau là thành công:

```sh
root@controller:~# openstack token issue
+------------+---------------------------------------------------------------------------------------------------------------------------------+
| Field      | Value                                                                                                                           |
+------------+---------------------------------------------------------------------------------------------------------------------------------+
| expires    | 2017-07-05T05:12:51+0000                                                                                                        |
| id         | gAAAAABZXGdDMtAI2MxTqoaTbuMmbooPqTmTsA_WpLrJYLmJuXVyH7AYy-G5IDBH6CMB8QfZAvtsR6UZFHXQIqNU5-yryqr6DVkR7mkR-                       |
|            | UJnPnbidO48QNHGD9nbEFlY-URpBCt44tkW50zLe9zJBa6QO6jZNZfJijpMGuTlaTUMkQQgwf8ykVc                                                  |
| project_id | 5030fd08e3e24ce2a2da4587b0108f62                                                                                                |
| user_id    | 90dd1c6486864c709b2dcdeec669cd92                                                                                                |
+------------+---------------------------------------------------------------------------------------------------------------------------------+

```

## 2.3 Cài đặt glace

### 2.3.1. Tạo database

- Đăng nhập vào cơ sở dữ liệu: `mysql`
- Tạo cơ sở dữ liệu `glance`:MariaDB [(none)]> CREATE DATABASE glance;
- Cấp quyền truy nhập vào cơ sở dữ liệu keystone:

```sh
    MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' \
  IDENTIFIED BY '123456';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' \
  IDENTIFIED BY '123456';
```

- Thoát khỏi sql: `exit`

### 2.3.2 Cấu hình xác thực cho glance

- Chạy file admin-openrc đề có quyền truy cập vào CLI của quản trị viên:`. admin-openrc`

- Tạo user `glance`:`openstack user create --domain default --password-prompt glance`. Sau đó nhập vào password `123456`
- Gán quyền `admin` và project `service` cho user `glance`:
   ` openstack role add --project service --user glance admin`

- Tạo service có tên `glance`:

   ` openstack service create --name glance \
  --description "OpenStack Image" image`

- Tạo các endpoint cho service:

```sh
    openstack endpoint create --region RegionOne \
  image public http://controller:9292
    openstack endpoint create --region RegionOne \
  image internal http://controller:9292
    openstack endpoint create --region RegionOne \
  image admin http://controller:9292

```

### 2.3.3 Cài đặt và cấu hình cho glance

- Cài đặt gói `glance`:
        `apt-get -y install glance`
- Chỉnh sửa file /etc/glance/glance-api.conf như sau:
    - Trong section [database]
        - Comment dòng
        `#sqlite_db = /var/lib/glance/glance.sqlite`
        - Thêm dòng:
        `connection = mysql+pymysql://glance:123456@controller/glance`
    - Trong thẻ [keystone_authtoken]: Thêm đoạn sau:

    ```sh
        auth_uri = http://controller:5000
        auth_url = http://controller:35357
        memcached_servers = controller:11211
        auth_type = password
        project_domain_name = default
        user_domain_name = default
        project_name = service
        username = glance
        password = 123456

    ```
    - Trong thẻ `[paste_deploy]`: Thêm dòng
    `flavor = keystone`
    - Trong thẻ `[glance_store]`: Thêm đoạn

    ```sh
    stores = file,http
    default_store = file
    filesystem_store_datadir = /var/lib/glance/images/

    ```

- Chỉnh sửa file /etc/glance/glance-registry.conf như sau:   
    - Trong section [database]
        - Comment dòng
        `#sqlite_db = /var/lib/glance/glance.sqlite`
        - Thêm dòng:
        `connection = mysql+pymysql://glance:123456@controller/glance`
    - Trong thẻ [keystone_authtoken]: Thêm đoạn sau:

    ```sh
       auth_uri = http://controller:5000
        auth_url = http://controller:35357
        memcached_servers = controller:11211
        auth_type = password
        project_domain_name = default
        user_domain_name = default
        project_name = service
        username = glance
        password = 123456

    ```
    - Trong thẻ `[paste_deploy]`: Thêm dòng
    `flavor = keystone`

- Đồng bộ database cho glance:
    `su -s /bin/sh -c "glance-manage db_sync" glance`
- Khởi động lại glance service:

    ```sh
    service glance-registry restart
    service glance-api restart
    ```

### 2.3.4 Kiểm chứng việc cài đặt glance

- Chạy file admin-openrc đề có quyền truy cập vào CLI của quản trị viên:`
    `. admin-openrc`.
- Download source image:
    `wget http://download.cirros-cloud.net/0.3.5/cirros-0.3.5-x86_64-disk.img`
- Upload image vừa tải về:

```sh
    openstack image create "cirros" \
    --file cirros-0.3.5-x86_64-disk.img \
    --disk-format qcow2 --container-format bare \
    --public

```

- Xác nhận việc upload image:
    `openstack image list`
- Thu được kết quả như sau:

```sh
root@controller:~# openstack image list
+--------------------------------------+--------+--------+
| ID                                   | Name   | Status |
+--------------------------------------+--------+--------+
| 1bf0f8f6-4e31-45ab-9521-61ae01ad1df2 | cirros | active |
+--------------------------------------+--------+--------+

```

## 2.4 Cài đặt Nova

### 2.4.1 Tạo database

- Đăng nhập vào cơ sở dữ liệu: `mysql`
- Tạo các cơ sở dữ liệu nova_api, nova, nova_cell0:

```sql
    MariaDB [(none)]> CREATE DATABASE nova_api;
    MariaDB [(none)]> CREATE DATABASE nova;
    MariaDB [(none)]> CREATE DATABASE nova_cell0;
```

- Cấp quyền truy cập cơ sở dữ liệu:

```sql
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' \
  IDENTIFIED BY '123456';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' \
  IDENTIFIED BY '123456';

MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' \
  IDENTIFIED BY '123456';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' \
  IDENTIFIED BY '123456';

MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' \
  IDENTIFIED BY '123456';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' \
  IDENTIFIED BY '123456';
```

- Thoát mysql: `exit`

### 2.4.2 Tạo các endpoint cho nova

- Chạy file admin-openrc đề có quyền truy cập vào CLI của quản trị viên:`
    `. admin-openrc`.

- Tọa user với tên `nova`:
    `openstack user create --domain default --password-prompt nova`
    
    Nhập password `123456`
- Phân quyền `admin` cho tài khoản `nova`:
    `openstack role add --project service --user nova admin`
- Tạo service có tên `nova`:

    ```sh
    openstack service create --name nova \
    --description "OpenStack Compute" compute

    ```

- Tạo các endpoint:

    ```sh
    openstack endpoint create --region RegionOne \
    compute public http://controller:8774/v2.1
    openstack endpoint create --region RegionOne \
    compute internal http://controller:8774/v2.1
    openstack endpoint create --region RegionOne \
    compute admin http://controller:8774/v2.1
    ```

- Tạo Placement service:
    `openstack user create --domain default --password-prompt placement`
    
    Nhập vào password: `123456`
- Thêm Placement user vào project service với vai trò admin.
    `openstack role add --project service --user placement admin`
- Tạo Placement APi entry trong service catalog
    `openstack service create --name placement --description "Placement API" placement`
- Tạo các endpoint cho Placement API:

    ```sh
    openstack endpoint create --region RegionOne placement public http://controller:8778
    openstack endpoint create --region RegionOne placement internal http://controller:8778
    openstack endpoint create --region RegionOne placement admin http://controller:8778

    ```

### 2.4.3 Cài đặt và cấu hình nova

- Cài đặt các package:

    ```sh
    apt install nova-api nova-conductor nova-consoleauth \
    nova-novncproxy nova-scheduler nova-placement-api
    ```

- Chỉnh sửa file /etc/nova/nova.conf như sau:

    - Trong section [api_database]
        - Comment dòng: `#connection=sqlite:////var/lib/nova/nova.sqlite`
        - thêm dòng:
        `connection = mysql+pymysql://nova:123456@controller/nova_api`
    - Trong section [database] thêm dòng
        `connection = mysql+pymysql://nova:123456@controller/nova`
    - Trong section [DEFAULT] thêm các dòng

        ```sh
        transport_url = rabbit://openstack:123456@controller
        my_ip = 10.10.10.11
        use_neutron = True
        firewall_driver = nova.virt.firewall.NoopFirewallDriver

        ```
    - Trong section [api] thêm dòng:
        `auth_strategy = keystone`

    - Trong section [keystone_authtoken] thêm đoạn sau:

        ```sh
        auth_uri = http://controller:5000
        auth_url = http://controller:35357
        memcached_servers = controller:11211
        auth_type = password
        project_domain_name = default
        user_domain_name = default
        project_name = service
        username = nova
        password = 123456

        ```
    - Trong section [vnc] thêm các dòng:

        ```sh
        enabled = true

        vncserver_listen = $my_ip
        vncserver_proxyclient_address = $my_ip

        ```
    - Trong section [glance] thêm dòng
        `api_servers = http://controller:9292`
    - Trong section [oslo_concurrency] thêm dòng:
        `lock_path = /var/lib/nova/tmp`
    - Trong section [placement] thêm đoạn sau:

        ```sh
        os_region_name = RegionOne
        project_domain_name = Default
        project_name = service
        auth_type = password
        user_domain_name = Default
        auth_url = http://controller:35357/v3
        username = placement
        password = 123456
        ```
    - Xóa bỏ dòng `log_dir` trong section [DEFAULT]
- Đồng bộ nova-api database:
    `su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova`
- Register cell0 database:
     su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
- Tạo cell1:
    su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
- Đồng bộ hóa nova database:
    su -s /bin/sh -c "nova-manage db sync" nova
- Xác minh cell0 và cell0 đã được tọa chính xác:
    nova-manage cell_v2 list_cells

    Kết quả:

    ```sh
    root@controller:~# nova-manage cell_v2 list_cells
    +-------+--------------------------------------+
    |  Name |                 UUID                 |
    +-------+--------------------------------------+
    | cell0 | 00000000-0000-0000-0000-000000000000 |
    | cell1 | 7c431a62-d938-4487-afe0-967a45050d5a |
    +-------+--------------------------------------+

    ```

- Hoàn thành cài đặt restart lại compute service:

    ```sh
    service nova-api restart
    service nova-consoleauth restart
    service nova-scheduler restart
    service nova-conductor restart
    service nova-novncproxy restart
    ```

### 2.4.4 Thêm compute node vào cơ sở dữ liệu cell

- Chạy file admin-openrc đề có quyền truy cập vào CLI của quản trị viên:`
    `. admin-openrc`.

- Discover compute hosts:

    `su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova`

### 2.4.5 Kiểm chứng việc cài đặt

- Chạy file admin-openrc đề có quyền truy cập vào CLI của quản trị viên:`
    `. admin-openrc`.
- Liệt kê danh sách các service để xác minh: `openstack compute service list`

    ```sh
    root@controller:~# openstack compute service list
    +----+------------------+------------+----------+---------+-------+----------------------------+
    | ID | Binary           | Host       | Zone     | Status  | State | Updated At                 |
    +----+------------------+------------+----------+---------+-------+----------------------------+
    |  3 | nova-scheduler   | controller | internal | enabled | up    | 2017-07-05T09:59:09.000000 |
    |  4 | nova-consoleauth | controller | internal | enabled | up    | 2017-07-05T09:59:00.000000 |
    |  5 | nova-conductor   | controller | internal | enabled | up    | 2017-07-05T09:59:00.000000 |
    |  6 | nova-compute     | compute1   | nova     | enabled | up    | 2017-07-05T09:59:06.000000 |
    +----+------------------+------------+----------+---------+-------+----------------------------+

    ```
- Danh sách các API endpoint: `openstack catalog list`

```sh
root@controller:~# openstack catalog list
+-----------+-----------+-----------------------------------------+
| Name      | Type      | Endpoints                               |
+-----------+-----------+-----------------------------------------+
| keystone  | identity  | RegionOne                               |
|           |           |   admin: http://controller:35357/v3/    |
|           |           | RegionOne                               |
|           |           |   internal: http://controller:5000/v3/  |
|           |           | RegionOne                               |
|           |           |   public: http://controller:5000/v3/    |
|           |           |                                         |
| placement | placement | RegionOne                               |
|           |           |   internal: http://controller:8778      |
|           |           | RegionOne                               |
|           |           |   public: http://controller:8778        |
|           |           | RegionOne                               |
|           |           |   admin: http://controller:8778         |
|           |           |                                         |
| glance    | image     | RegionOne                               |
|           |           |   public: http://controller:9292        |
|           |           | RegionOne                               |
|           |           |   admin: http://controller:9292         |
|           |           | RegionOne                               |
|           |           |   internal: http://controller:9292      |
|           |           |                                         |
| nova      | compute   | RegionOne                               |
|           |           |   public: http://controller:8774/v2.1   |
|           |           | RegionOne                               |
|           |           |   internal: http://controller:8774/v2.1 |
|           |           | RegionOne                               |
|           |           |   admin: http://controller:8774/v2.1    |
|           |           |                                         |
+-----------+-----------+-----------------------------------------+

```

- Danh sách các image:  `openstack image list`

```sh
root@controller:~#  openstack image list
+--------------------------------------+--------+--------+
| ID                                   | Name   | Status |
+--------------------------------------+--------+--------+
| 1bf0f8f6-4e31-45ab-9521-61ae01ad1df2 | cirros | active |
+--------------------------------------+--------+--------+
```

- Kiểm tra các cell và các API: `nova-status upgrade check`

root@controller:~# nova-status upgrade check

```sh
+--------------------------------------------+
| Upgrade Check Results                      |
+--------------------------------------------+
| Check: Cells v2                            |
| Result: Success                            |
| Details: None                              |
+--------------------------------------------+
| Check: Placement API                       |
| Result: Failure                            |
| Details: Placement API endpoint not found. |
+--------------------------------------------+
| Check: Resource Providers                  |
| Result: Success                            |
| Details: None                              |
+--------------------------------------------+

```

### 2.5 Cài đặt neutron

#### 2.5.1 Tạo database

- Đăng nhập vào mysql: `mysql`
- Tạo cơ sở dữ liệu neutron:
   ` MariaDB [(none)] CREATE DATABASE neutron;`
- Cấp quyền cho sơ cở dữ liệu:

```sql
MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' \
  IDENTIFIED BY '123456';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' \
  IDENTIFIED BY '123456';

```
- Thoát mysql: exit

#### 2.5.2 Tạo user, service và endpoint

- Chạy file admin-openrc đề có quyền truy cập vào CLI của quản trị viên:`
    `. admin-openrc`.
- Tạo user có tên `neutron`: 
    `openstack user create --domain default --password-prompt neutron`

    Nhập password `123456`
- Thêm vai trò `admin` cho user `neutron`:
    `openstack role add --project service --user neutron admin`
- Tạo dịch vụ có tên `neutron`:

```sh
    openstack service create --name neutron \
  --description "OpenStack Networking" network

```

- Tạo các endpoint cho neutron:

```sh
openstack endpoint create --region RegionOne network public http://controller:9696

openstack endpoint create --region RegionOne network internal http://controller:9696

openstack endpoint create --region RegionOne network admin http://controller:9696

```

### 2.5.3 Cài đặt và cấu hình cho neutron

- Cài đặt các gói:

    ```sh
    apt install neutron-server neutron-plugin-ml2 \
    neutron-linuxbridge-agent neutron-l3-agent neutron-dhcp-agent \
    neutron-metadata-agent

    ```

- Chỉnh sửa file `/etc/neutron/neutron.conf` như sau:
    - Trong section [database]
        - Comment dòng: `#connection = sqlite:////var/lib/neutron/neutron.sqlite`
        - thêm dòng
        `connection = mysql+pymysql://neutron:123456@controller/neutron`
    - Trong section [DEFAULT] thềm các dòng:

        ```sh
        core_plugin = ml2
        service_plugins = router
        allow_overlapping_ips = true
        transport_url = rabbit://openstack:123456@controller
        auth_strategy = keystone
        notify_nova_on_port_status_changes = true
        notify_nova_on_port_data_changes = true

        ```
    - Trong section [keystone_authtoken] thêm các dòng:

        ```sh
        auth_uri = http://controller:5000
        auth_url = http://controller:35357
        memcached_servers = controller:11211
        auth_type = password
        project_domain_name = default
        user_domain_name = default
        project_name = service
        username = neutron
        password = 123456

        ```
    - Trong section [nova] thêm các dòng

        ```sh
        auth_url = http://controller:35357
        auth_type = password
        project_domain_name = default
        user_domain_name = default
        region_name = RegionOne
        project_name = service
        username = nova
        password = 123456

        ```

- Chỉnh sửa file `/etc/neutron/plugins/ml2/ml2_conf.ini` như sau:
    - Trong section [ml2] thêm các dòng 

        ```sh
        type_drivers = flat,vlan,vxlan
        tenant_network_types = vxlan
        mechanism_drivers = linuxbridge,l2population
        extension_drivers = port_security

        ```

    - Trong section [ml2_type_flat] thêm dòng: `flat_networks = provider`
    - Trong section [ml2_type_vxlan] thêm dòng: `vni_ranges = 1:1000`
    - Trong section [securitygroup] thêm dòng: `enable_ipset = true`

- Chỉnh sửa file /etc/neutron/plugins/ml2/linuxbridge_agent.ini như sau:
    - Trong section [linux_bridge] thêm dòng:
        `physical_interface_mappings = provider:ens9`
    - Trong section [vxlan] thêm các dòng

        ```sh
        enable_vxlan = true
        local_ip = 10.10.10.11
        l2_population = true

        ```
    - Trong section [securitygroup] thêm các dòng

        ```sh
        enable_security_group = true
        firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver

        ```
- Chỉnh sửa file `/etc/neutron/l3_agent.ini` như sau:
    - Trong section [DEFAULT] thêm dòng: `interface_driver = linuxbridge`
- Chỉnh sửa file `/etc/neutron/dhcp_agent.ini` như sau:
    - Trong section [DEFAULT] thêm các dòng:

        ```sh
        interface_driver = linuxbridge
        dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
        enable_isolated_metadata = true

        ```
- Chỉnh sửa file /etc/nova/nova.conf như sau
    - Trong section [neutron] thêm các dòng:

        ```sh
        url = http://controller:9696
        auth_url = http://controller:35357
        auth_type = password
        project_domain_name = default
        user_domain_name = default
        region_name = RegionOne
        project_name = service
        username = neutron
        password = 123456
        service_metadata_proxy = true
        metadata_proxy_shared_secret = 123456

        ```
- Đồng bộ database:

```sh
su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
  --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
```

- Restart Compute API service: service nova-api restart
- Restart Networking services:

    ```sh
    service neutron-server restart
    service neutron-linuxbridge-agent restart
    service neutron-dhcp-agent restart
    service neutron-metadata-agent restart
    service neutron-l3-agent restart

    ```

### 2.5.4 Kiểm chứng việc cài đặt

- Chạy file admin-openrc đề có quyền truy cập vào CLI của quản trị viên:`
    `. admin-openrc`.
- Danh sách các extension: `openstack extension list --network`

```sh
root@controller:~# openstack extension list --network

+--------------------------------------------------------+---------------------------+---------------------------------------------------------+
| Name                                                   | Alias                     | Description                                             |
+--------------------------------------------------------+---------------------------+---------------------------------------------------------+
| Default Subnetpools                                    | default-subnetpools       | Provides ability to mark and use a subnetpool as the    |
|                                                        |                           | default                                                 |
| Network IP Availability                                | network-ip-availability   | Provides IP availability data for each network and      |
|                                                        |                           | subnet.                                                 |
| Network Availability Zone                              | network_availability_zone | Availability zone support for network.                  |
| Auto Allocated Topology Services                       | auto-allocated-topology   | Auto Allocated Topology Services.                       |
| Neutron L3 Configurable external gateway mode          | ext-gw-mode               | Extension of the router abstraction for specifying      |
|                                                        |                           | whether SNAT should occur on the external gateway       |
| Port Binding                                           | binding                   | Expose port bindings of a virtual port to external      |
|                                                        |                           | application                                             |
| agent                                                  | agent                     | The agent management extension.                         |
| Subnet Allocation                                      | subnet_allocation         | Enables allocation of subnets from a subnet pool        |
| L3 Agent Scheduler                                     | l3_agent_scheduler        | Schedule routers among l3 agents                        |
| Tag support                                            | tag                       | Enables to set tag on resources.                        |
| Neutron external network                               | external-net              | Adds external network attribute to network resource.    |
| Neutron Service Flavors                                | flavors                   | Flavor specification for Neutron advanced services      |
| Network MTU                                            | net-mtu                   | Provides MTU attribute for a network resource.          |
| Availability Zone                                      | availability_zone         | The availability zone extension.                        |
| Quota management support                               | quotas                    | Expose functions for quotas management per tenant       |
| HA Router extension                                    | l3-ha                     | Add HA capability to routers.                           |
| Provider Network                                       | provider                  | Expose mapping of virtual networks to physical networks |
| Multi Provider Network                                 | multi-provider            | Expose mapping of virtual networks to multiple physical |
|                                                        |                           | networks                                                |
| Address scope                                          | address-scope             | Address scopes extension.                               |
| Neutron Extra Route                                    | extraroute                | Extra routes configuration for L3 router                |
| Subnet service types                                   | subnet-service-types      | Provides ability to set the subnet service_types field  |
| Resource timestamps                                    | standard-attr-timestamp   | Adds created_at and updated_at fields to all Neutron    |
|                                                        |                           | resources that have Neutron standard attributes.        |
| Neutron Service Type Management                        | service-type              | API for retrieving service providers for Neutron        |
|                                                        |                           | advanced services                                       |
| Router Flavor Extension                                | l3-flavors                | Flavor support for routers.                             |
| Port Security                                          | port-security             | Provides port security                                  |
| Neutron Extra DHCP opts                                | extra_dhcp_opt            | Extra options configuration for DHCP. For example PXE   |
|                                                        |                           | boot options to DHCP clients can be specified (e.g.     |
|                                                        |                           | tftp-server, server-ip-address, bootfile-name)          |
| Resource revision numbers                              | standard-attr-revisions   | This extension will display the revision number of      |
|                                                        |                           | neutron resources.                                      |
| Pagination support                                     | pagination                | Extension that indicates that pagination is enabled.    |
| Sorting support                                        | sorting                   | Extension that indicates that sorting is enabled.       |
| security-group                                         | security-group            | The security groups extension.                          |
| DHCP Agent Scheduler                                   | dhcp_agent_scheduler      | Schedule networks among dhcp agents                     |
| Router Availability Zone                               | router_availability_zone  | Availability zone support for router.                   |
| RBAC Policies                                          | rbac-policies             | Allows creation and modification of policies that       |
|                                                        |                           | control tenant access to resources.                     |
| Tag support for resources: subnet, subnetpool, port,   | tag-ext                   | Extends tag support to more L2 and L3 resources.        |
| router                                                 |                           |                                                         |
| standard-attr-description                              | standard-attr-description | Extension to add descriptions to standard attributes    |
| Neutron L3 Router                                      | router                    | Router abstraction for basic L3 forwarding between L2   |
|                                                        |                           | Neutron networks and access to external networks via a  |
|                                                        |                           | NAT gateway.                                            |
| Allowed Address Pairs                                  | allowed-address-pairs     | Provides allowed address pairs                          |
| project_id field enabled                               | project-id                | Extension that indicates that project_id field is       |
|                                                        |                           | enabled.                                                |
| Distributed Virtual Router                             | dvr                       | Enables configuration of Distributed Virtual Routers.   |
+--------------------------------------------------------+---------------------------+---------------------------------------------------------+

```
- Danh sách các agent: `openstack network agent list`

```sh
root@controller:~# openstack network agent list
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
| ID                                   | Agent Type         | Host       | Availability Zone | Alive | State | Binary                    |
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
| 3c5c8393-4bae-47ab-8fdb-af0ebadedb55 | L3 agent           | controller | nova              | True  | UP    | neutron-l3-agent          |
| 4ee1f06f-68cb-430f-bb55-e9ebb68e02c0 | Linux bridge agent | controller | None              | True  | UP    | neutron-linuxbridge-agent |
| a8c859a7-591d-4e42-a1c5-1955900df5e4 | Linux bridge agent | compute1   | None              | True  | UP    | neutron-linuxbridge-agent |
| e02c31d8-cf0e-49ab-8798-7b66492147a0 | Metadata agent     | controller | None              | True  | UP    | neutron-metadata-agent    |
| ff14a089-ca97-47fe-bf78-3fa91db6eaff | DHCP agent         | controller | nova              | True  | UP    | neutron-dhcp-agent        |
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+

```

### 2.6 Cài đặt dashboard

### 2.6.1 Cài đặt và cấu hình dashboard

- Cài đặt package:` apt install openstack-dashboard`
-Chỉnh sửa file `/etc/openstack-dashboard/local_settings.py` như sau:

    - sửa dòng :`OPENSTACK_HOST = "127.0.0.1"` =>`OPENSTACK_HOST = "controller"`
    - Thêm dòng: `SESSION_ENGINE = 'django.contrib.sessions.backends.cache'` vào trước dòng

        ```py
        CACHES = {
            'default': {
                'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
                'LOCATION': 'controller:11211',
            }
        }

        ```
    - Sửa đoạn

        ```py
        CACHES = {
            'default': {
                'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
                'LOCATION': 'controller:11211',
            }
        }

        ```

    - Thành đoạn

        ```py
            CACHES = {
                'default': {
                    'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
                    'LOCATION': 'controller:11211',
                }
            }

        ```

    - Sửa dòng :
`OPENSTACK_KEYSTONE_URL = "http://%s:5000/v2.0" % OPENSTACK_HOST` Thành dòng
    `OPENSTACK_KEYSTONE_URL = "http://%s:5000/v3" % OPENSTACK_HOST`
    - Bỏ comment dòng: `OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True`
    - Bỏ commment đoạn :

        ```py
        OPENSTACK_API_VERSIONS = {
            "identity": 3,
            "image": 2,
            "volume": 2,
        }

        ```
    - Bỏ comment dòng: `OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = "Default"`
    - Sửa dòng: `OPENSTACK_KEYSTONE_DEFAULT_ROLE = "_membet_"`
    => `OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"`

- Restart web server `service apache2 reload`

- Xác nhận bằng cách truy cập bằng trình duyệt vào `http://controller/horizon`

## 3. Cài đặt trên node compute

### 3.1. Cài đặt môi trường

### 3.1.1 Configure network interface

- Dùng vim để chỉnh sửa file /etc/network/interface với nội dung như sau:

```sh
auto lo
iface lo inet loopback

#management_net
auto ens3
iface ens3 inet static
	address 10.10.10.10
	gateway 10.10.10.1
	broadcast 10.10.10.255
	dns-nameservers 8.8.8.8 8.8.4.4
	netmask 255.255.255.0

#external_net
auto ens9
iface ens9 inet manual
	up ip link set dev $IFACE up
	down ip link set dev $IFACE down
```

- Khởi động lại hệ thống để áp dụng những thay đổi

### 3.1.2 Configure name resolution

- Set hostname của node là controller bằng cách sử dụng vim chỉnh sửa file /etc/hostname với nội dung sau:

```sh
compute1
```

- Chỉnh sửa file /etc/hosts với nội dung như sau:

```sh
127.0.0.1	localhost
127.0.1.1	vthao


#controller
10.10.10.11 controller

#compute
10.10.10.10 compute1

```

### 3.1.3 Kiểm tra lại kết nối

```sh
vthao@controller:~$ ping -c 4 openstack.org
PING openstack.org (162.242.140.107) 56(84) bytes of data.
64 bytes from 162.242.140.107: icmp_seq=1 ttl=49 time=306 ms
64 bytes from 162.242.140.107: icmp_seq=2 ttl=50 time=205 ms
64 bytes from 162.242.140.107: icmp_seq=3 ttl=50 time=251 ms
64 bytes from 162.242.140.107: icmp_seq=4 ttl=50 time=272 ms

```

```sh
vthao@controller:~$ ping -c 4 controller
PING controller (10.10.10.10) 56(84) bytes of data.
64 bytes from controller (10.10.10.10): icmp_seq=1 ttl=64 time=1.09 ms
64 bytes from controller (10.10.10.10): icmp_seq=2 ttl=64 time=1.52 ms
64 bytes from controller (10.10.10.10): icmp_seq=3 ttl=64 time=1.74 ms
64 bytes from controller (10.10.10.10): icmp_seq=4 ttl=64 time=1.38 ms

```

### 3.1.4 Cài đặt NTP

- Cài đặt package: `apt install chrony`

- sử dụng vim mở file `/etc/chrony/chrony.conf` và thay thế dòng
`pool 2.debian.pool.ntp.org offline iburst`
bởi các dòng

```sh
server controller iburst
```

`

- Restart NTP service:

    `service chrony restart`

- Kiểm tra lại hoạt động của NTP:

```sh
vthao@controller:~$ chronyc sources
210 Number of sources = 1
  MS Name/IP address         Stratum Poll Reach LastRx Last sample
  ===============================================================================
  ^* controller                    3    9   377   421    +15us[  -87us] +/-   15ms

```

### 3.1.5 Kích hoạt Openstack repo

```sh
apt install software-properties-common
add-apt-repository cloud-archive:ocata
```

### 3.1.6 Hoàn thành cài đặt

- Cập nhật các gói phần mềm
    `apt update && apt dist-upgrade`

- Cài đặt Openstack client
    `apt install python-openstackclient`

- Khởi động lại máy ảo.

### 3.2 Cài đặt nova

### 3.2.1 Cài đặt và cấu hình

- Cài đặt package: `apt install nova-compute`

- Chỉnh sửa file `/etc/nova/nova.conf` như sau:

    - Trong section [DEFAULT] thêm các dòng

        ```sh
        transport_url = rabbit://openstack:123456@controller
        my_ip = 10.10.10.10
        use_neutron = True
        firewall_driver = nova.virt.firewall.NoopFirewallDriver

        ```
    - Trong section [api] thêm dòng
        `auth_strategy = keystone`
    - Trong section [keystone_authtoken] thêm đoạn sau:

        ```sh
        auth_uri = http://controller:5000
        auth_url = http://controller:35357
        memcached_servers = controller:11211
        auth_type = password
        project_domain_name = default
        user_domain_name = default
        project_name = service
        username = nova
        password = 123456

        ```
    - Trong section [vnc] thêm các dòng

    ```sh
        enabled = True
        vncserver_listen = 0.0.0.0
        vncserver_proxyclient_address = $my_ip
        novncproxy_base_url = http://controller:6080/vnc_auto.html

    ```

    - Trong section [glance] thêm dòng
        `api_servers = http://controller:9292`
    - Trong section [oslo_concurrency]

        - Loại bỏ các `lock_path` khác
        - thêm dòng
        `lock_path = /var/lib/nova/tmp`
    - Loại bỏ dòng `log_dir` trong section [DEFAULT]
    - Trong section [placement]

        - Loại bỏ dòng `os_region_name = openstack`
        - thêm các dòng

        ```sh
        os_region_name = RegionOne
        project_domain_name = Default
        project_name = service
        auth_type = password
        user_domain_name = Default
        auth_url = http://controller:35357/v3
        username = placement
        password = 123456

        ```

### 3.2.2 Hoàn thành cài đặt

- Xác định xem node compute có hỗ trợ tăng tốc phần cứng không bằng câu lệnh:`egrep -c '(vmx|svm)' /proc/cpuinfo`
    - Nếu kết quả trả về là 1 hoặc lớn hơn nghĩa là node compute hỗ trỡ tăng tốc phần cứng mà không yêu cầu cấu hình,
    - Nếu kết quả trả về là 0 nghĩa là node compute không hỗ trợ tăng tốc phần cứng do đó bạn phải cấu hình libvirt để sử dụng QEMY thay vì KVM bằng cách chỉnh sửa section `[libvirt]` trong file `/etc/nova/nova-compute.conf` như sau:

        ```sh
        [libvirt]
        # ...
        virt_type = qemu
        ```
- Restart compute service : `service nova-compute restart`

### 3.3 Cài đặt neutron

- Cài đặt package: `apt install neutron-linuxbridge-agent`
- Chỉnh sửa file `/etc/neutron/neutron.con`f như sau:
    - Comment tất cả các connection trong section [database]
    - Trong section [DEFAULT] thêm các dòng

        ```sh
        transport_url = rabbit://openstack:123456@controller
        auth_strategy = keystone

        ```
    - Trong section [keystone_authtoken] thêm các dòng

    ```sh
    auth_uri = http://controller:5000
    auth_url = http://controller:35357
    memcached_servers = controller:11211
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    project_name = service
    username = neutron
    password = 123456

    ```
- Chỉnh sửa file `/etc/neutron/plugins/ml2/linuxbridge_agent.ini` như sau:
    - Trong section [linux_bridge] thêm dòng: `physical_interface_mappings = provider:ens9`
    - Trong section [vxlan] thêm các dòng:

        ```sh
        enable_vxlan = true
        local_ip = 10.10.10.10
        l2_population = true

        ```
    - Trong section [securitygroup] thêm các dòng

        ```sh
        enable_security_group = true
        firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver

        ```

- Chỉnh sửa file  /etc/nova/nova.conf như sau:
    - Trong section [neutron] thêm các dòng

        ```sh
        url = http://controller:9696
        auth_url = http://controller:35357
        auth_type = password
        project_domain_name = default
        user_domain_name = default
        region_name = RegionOne
        project_name = service
        username = neutron
        password = 123456
        ```

- Hoàn thành cài đặt restart các service:

    ```sh
    service nova-compute restart
    service neutron-linuxbridge-agent restart

    ```