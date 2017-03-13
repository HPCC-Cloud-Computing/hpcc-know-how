
OpenStack Newton All-in-one Install Guide
===================

Bài viết hướng dẫn cài OpenStack bản Newton theo mô hình All-in-one trên Centos 7-64bit

**Lưu ý:**

 - Đăng nhập với quyền root trên tất cả các bước cài đặt.
 - Password thống nhất cho tất cả các dịch vụ là `bkcloud`
 - Trước khi cài đặt, cần cập nhật các gói phần mềm

	     # yum update  
	     # yum upgrade

#1. Chuẩn bị môi trường cài đặt
##1.1 Cấu hình mạng
 Ta cần cấu hình 2 card mạng. Card mạng thứ nhất - **eth0** thuộc dải `10.10.10.0/24` (management network) và card mạng thứ hai - **eth1** thuộc dải `192.168.2.0/24` (external network)
 
Thiết lập địa chỉ IP tĩnh cho eth0 và eth1.

    eth0: 10.10.10.50
    eth1: 192.168.2.50

Chỉnh sửa file `/etc/sysconfig/network-scripts/ifcfg-eth0` như sau:

	# Not Internet connected (OpenStack management network)
    TYPE=Ethernet
    BOOTPROTO=none
    DEFROUTE=yes
    PEERDNS=yes
    PEERROUTES=yes
    IPV4_FAILURE_FATAL=no
    IPV6INIT=yes
    IPV6_AUTOCONF=yes
    IPV6_DEFROUTE=yes
    IPV6_PEERDNS=yes
    IPV6_PEERROUTES=yes
    IPV6_FAILURE_FATAL=no
    IPV6_ADDR_GEN_MODE=stable-privacy
    NAME=eth0
    DEVICE=eth0
    ONBOOT=yes
    IPADDR=10.10.10.50
    NETMASK=255.255.255.0
    ZONE=public

Chỉnh sửa file `/etc/sysconfig/network-scripts/ifcfg-eth1` như sau:

	# For exposing OpenStack API over the Internet
    TYPE=Ethernet
    BOOTPROTO=none
    DEFROUTE=yes
    PEERDNS=yes
    PEERROUTES=yes
    IPV4_FAILURE_FATAL=no
    IPV6INIT=yes
    IPV6_AUTOCONF=yes
    IPV6_DEFROUTE=yes
    IPV6_PEERDNS=yes
    IPV6_PEERROUTES=yes
    IPV6_FAILURE_FATAL=no
    IPV6_ADDR_GEN_MODE=stable-privacy
    NAME=eth1
    DEVICE=eth1
    ONBOOT=yes
    IPADDR=192.168.2.50
    NETMASK=255.255.255.0
    GATEWAY=192.168.2.1
    DNS1=8.8.8.8
    ZONE=public

Khởi động lại card mạng sau khi thiết lập IP tĩnh

    ifdown eth0 && ifup eth0
    ifdown eth1 && ifup eth1

Chỉnh sửa file /etc/hosts để phân giải IP sang hostname và ngược lại.

    127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
    ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
    
    10.10.10.50     controller

Kiểm tra kết nối tới gateway và internet sau khi thiết lập xong.

    # ping 192.168.2.50 -c 4
    PING 192.168.2.50 (172.16.69.1) 56(84) bytes of data.
    64 bytes from 192.168.2.50: icmp_seq=1 ttl=64 time=0.253 ms
    64 bytes from 192.168.2.50: icmp_seq=2 ttl=64 time=0.305 ms
    64 bytes from 192.168.2.50: icmp_seq=3 ttl=64 time=0.306 ms
    64 bytes from 192.168.2.50: icmp_seq=4 ttl=64 time=0.414 ms


    # ping google.com -c 4
    PING google.com (74.125.204.113) 56(84) bytes of data.
    64 bytes from ti-in-f113.1e100.net (74.125.204.113): icmp_seq=1 ttl=41 time=58.3 ms
    64 bytes from ti-in-f113.1e100.net (74.125.204.113): icmp_seq=2 ttl=41 time=58.3 ms
    64 bytes from ti-in-f113.1e100.net (74.125.204.113): icmp_seq=3 ttl=41 time=58.3 ms
    64 bytes from ti-in-f113.1e100.net (74.125.204.113): icmp_seq=4 ttl=41 time=58.3 ms


    # ping -c 4 controller  
    PING compute1 (10.10.10.50) 56(84) bytes of data.
    64 bytes from controller (10.10.10.50): icmp_seq=1 ttl=64 time=0.263 ms
    64 bytes from controller (10.10.10.50): icmp_seq=2 ttl=64 time=0.202 ms
    64 bytes from controller (10.10.10.50): icmp_seq=3 ttl=64 time=0.203 ms
    64 bytes from controller (10.10.10.50): icmp_seq=4 ttl=64 time=0.202 ms

##1.2 Cài đặt NTP
Cài gói chrony:

    # yum install chrony

Mở file /etc/chrony.conf và tìm các dòng dưới

    server 0.debian.pool.ntp.org offline minpoll 8
    server 1.debian.pool.ntp.org offline minpoll 8
    server 2.debian.pool.ntp.org offline minpoll 8
    server 3.debian.pool.ntp.org offline minpoll 8

Thay bằng các dòng sau

    server 1.vn.pool.ntp.org iburst
    server 0.asia.pool.ntp.org iburst
    server 3.asia.pool.ntp.org iburst

Khởi động lại dịch vụ NTP

    # systemctl enable chronyd.service
    # systemctl start chronyd.service

Kiểm tra lại hoạt động của NTP bằng lệnh dưới

    # chronyc sources
    
    210 Number of sources = 3
    MS Name/IP address         Stratum Poll Reach LastRx Last sample
    ===============================================================================
    ^+ time.vng.vn                   2   9   377   246   -656us[-1449us] +/-  133ms
    ^* 218.189.210.3                 2   9   377   243  +2890us[+2096us] +/-  149ms
    ^+ 123.108.200.124               3   9   377   114    -14ms[  -14ms] +/-  251ms


##1.3 Cài đặt repos để cài OpenStack Newton

Cài đặt gói để cài OpenStack Newton

    # yum install centos-release-openstack-newton

Cập nhật các gói phần mềm

    # yum upgrade

Cài đặt các gói client của OpenStack

    # yum install python-openstackclient


Khởi động lại máy, đăng nhập lại chuyển sang quyền root và thực hiện các bước tiếp theo.

##1.4 Cài đặt SQL database

Cài đặt gói phần mềm

    # yum install mariadb mariadb-server python2-PyMySQL

Cấu hình cho MariaDB, tạo file `/etc/my.cnf.d/openstack.cnf` với nội dung sau:

    [mysqld]
    bind-address = 10.10.10.50
    
    default-storage-engine = innodb
    innodb_file_per_table
    max_connections = 4096
    collation-server = utf8_general_ci
    character-set-server = utf8

Khởi động lại MariaDB

    # systemctl enable mariadb.service
    # systemctl start mariadb.service

Thiết lập password (bkcloud) và các cấu hình cơ bản cho MariaDB

    # mysql_secure_installation

##1.5 Cài đặt RabbitMQ

Cài đặt gói phần mềm

    # yum install rabbitmq-server

Cấu hình RabbitMQ, tạo user openstack với mật khẩu là `bkcloud`:

    # rabbitmqctl add_user openstack bkcloud

Gán quyền read, write cho tài khoản openstack trong RabbitMQ

    # rabbitmqctl set_permissions openstack ".*" ".*" ".*"

##1.6 Cài đặt Memcached

Cài đặt gói phần mềm

    # yum install memcached python-memcached

Sửa file `/etc/sysconfig/memcached`, thay dòng `-l 127.0.0.1` thành `-l 10.10.10.50`

Khởi động lại memcache

    # systemctl enable memcached.service
    # systemctl start memcached.service

#2. Cài đặt Keystone

##2.1 Tạo database cài đặt các gói và cấu hình keystone

Đăng nhập vào MariaDB

    mysql -u root -p

Tạo user, database cho keystone

    CREATE DATABASE keystone;
    
    GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
    IDENTIFIED BY 'bkcloud';
    
    GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
    IDENTIFIED BY 'bkcloud';
    flush privileges;
    exit;

##2.2 Cài đặt và cấu hình keystone
Cài đặt gói cho keystone

    # yum install openstack-keystone httpd mod_wsgi

Cấu hinh file /etc/keystone/keystone.conf với các yêu cầu sau:

 - Trong phần [database], cấu hình truy cập đến database:

        [database]
	    ...
	    connection = mysql+pymysql://keystone:bkcloud@controller/keystone

 - Trong phần [token], cấu hình nhà cung cấp thẻ Fernet:

	    [token]
		...
		provider = fernet

Đồng bộ database dịch vụ xác thực:

    su -s /bin/sh -c "keystone-manage db_sync" keystone

Thiết lập Fernet key

    # keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
    # keystone-manage credential_setup --keystone-user keystone --keystone-group keystone

Khởi động dịch vụ xác thực

    # keystone-manage bootstrap --bootstrap-password bkcloud \
      --bootstrap-admin-url http://controller:35357/v3/ \
      --bootstrap-internal-url http://controller:35357/v3/ \
      --bootstrap-public-url http://controller:5000/v3/ \
      --bootstrap-region-id RegionOne

Cấu hình apache cho keysonte. Sửa file `/etc/httpd/conf/httpd.conf`, tìm option ServerName và sửa thành:

    ServerName controller

Tạo link để cấu hình virtual host cho dịch vụ keysonte trong apache

    # ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/

Khởi động lại apache

    # systemctl enable httpd.service
    # systemctl start httpd.service

Khai báo sử dụng token để xác thực.

    $ export OS_USERNAME=admin
    $ export OS_PASSWORD=bkcloud
    $ export OS_PROJECT_NAME=admin
    $ export OS_USER_DOMAIN_NAME=Default
    $ export OS_PROJECT_DOMAIN_NAME=Default
    $ export OS_AUTH_URL=http://controller:35357/v3
    $ export OS_IDENTITY_API_VERSION=3

##2.3 Tạo domain, projects, users, and roles

Tạo project có tên là `service` để chứa các user service của openstack

    # openstack project create --domain default --description "Service Project" service

Tạo project tên là `demo`

    # openstack project create --domain default --description "Demo Project" demo

Tạo user tên là `demo`

    openstack user create demo --domain default --password bkcloud

Tạo role tên là `user`

    openstack role create user

Gán tài khoản `demo` có role là `user` vào project `demo`

    openstack role add --project demo --user demo user

##2.4 Kiểm chứng lại các bước cài đặt keysonte

Vô hiệu hóa cơ chế xác thực bằng token tạm thời trong keysonte bằng cách xóa `admin_token_auth` trong các section `[pipeline:public_api]`, `[pipeline:admin_api]` và `[pipeline:api_v3]` của file `/etc/keystone/keystone-paste.ini`

Bỏ thiết lập trong biến môi trường của OS_AUTH_URL và OS_PASSWORD bằng lệnh

    # unset OS_AUTH_URL OS_PASSWORD

Gõ lần lượt 2 lệnh dưới sau đó nhập mật khẩu

    openstack --os-auth-url http://controller:35357/v3 \
      --os-project-domain-name Default --os-user-domain-name Default \
      --os-project-name admin --os-username admin token issue
      
    và 
    
    openstack --os-auth-url http://controller:5000/v3 \
    --os-project-domain-name default --os-user-domain-name default \

##2.5 Tạo script biến môi trường cho client

Tạo file admin-openrc chứa nội dung sau:

    export OS_PROJECT_DOMAIN_NAME=Default
    export OS_USER_DOMAIN_NAME=Default
    export OS_PROJECT_NAME=admin
    export OS_USERNAME=admin
    export OS_PASSWORD=bkcloud
    export OS_AUTH_URL=http://controller:35357/v3
    export OS_IDENTITY_API_VERSION=3
    export OS_IMAGE_API_VERSION=2

Tạo file demo-openrc chứa nội dung sau:

    export OS_PROJECT_DOMAIN_NAME=Default
    export OS_USER_DOMAIN_NAME=Default
    export OS_PROJECT_NAME=demo
    export OS_USERNAME=demo
    export OS_PASSWORD=bkcloud
    export OS_AUTH_URL=http://controller:5000/v3
    export OS_IDENTITY_API_VERSION=3
    export OS_IMAGE_API_VERSION=2

Chạy script admin-openrc

    # . admin-openrc

Gõ lệnh dưới để kiểm tra biến môi trường ở trên đã chính xác hay chưa

    # openstack token issue
    
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

#3. Cài đặt Glance
###3.1 Tạo database cho glance
Đăng nhập vào mysql

    mysql -u root -p

Tạo database và gán các quyền cho user `glance` trong database

    CREATE DATABASE glance;
    GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY 'bkcloud';
    GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'bkcloud';
    FLUSH PRIVILEGES;
    exit;

###3.2 Cấu hình xác thực cho dịch vụ glance

Lấy thông tin xác thực bằng cách sử dụng file `admin-openrc`

    # .  admin-openrc

Tạo user `glance`

    openstack user create glance --domain default --password bkcloud

Gán quyền `admin` cho user `glance` và project `service`

    openstack role add --project service --user glance admin    

Tạo dịch vụ có tên là `glance`

    openstack service create --name glance --description "OpenStack Image service" image

Tạo các endpoint cho dịch vụ `glance`

    openstack endpoint create --region RegionOne image public http://controller:9292
    
    openstack endpoint create --region RegionOne image internal http://controller:9292
    
    openstack endpoint create --region RegionOne image admin http://controller:9292

###3.3 Cài đặt các gói và cấu hình cho dịch vụ glance

Cài đặt gói `glance`

    # yum install openstack-glance

Sửa các mục dưới đây trong  file `/etc/glance/glance-api.conf`

 - Trong section [database] cấu hình truy cập database

	    [database]
	    ...
	    connection = mysql+pymysql://glance:bkcloud@controller/glance

 - Trong section [keystone_authtoken] và [paste_deploy] cấu hình dịch vụ xác thực với Keystone khi có yêu cầu từ user hoặc nova component
 

	     [keystone_authtoken]
	    ...
	    auth_uri = http://controller:5000
	    auth_url = http://controller:35357
	    memcached_servers = controller:11211
	    auth_type = password
	    project_domain_name = Default
	    user_domain_name = Default
	    project_name = service
	    username = glance
	    password = bkcloud
    
	    [paste_deploy]
	    ...
	    flavor = keystone

 - Khai báo trong section [glance_store] nơi lưu trữ file image

	     [glance_store]
	    ...
	    stores = file,http
	    default_store = file
	    filesystem_store_datadir = /var/lib/glance/images/

Sửa các mục dưới đây trong hai file `/etc/glance/glance-registry.conf`

 - Trong section [database] cấu hình truy cập database

	    [database]
	    ...
	    connection = mysql+pymysql://glance:bkcloud@controller/glance

 - Trong section [keystone_authtoken] và [paste_deploy] cấu hình dịch vụ xác thực với Keystone khi có yêu cầu từ user hoặc nova component
 

	     [keystone_authtoken]
	    ...
	    auth_uri = http://controller:5000
	    auth_url = http://controller:35357
	    memcached_servers = controller:11211
	    auth_type = password
	    project_domain_name = Default
	    user_domain_name = Default
	    project_name = service
	    username = glance
	    password = bkcloud
    
	    [paste_deploy]
	    ...
	    flavor = keystone

Đồng bộ database cho `glance`

    # su -s /bin/sh -c "glance-manage db_sync" glance 

Khởi động lại dịch vụ `glance`

    # systemctl enable openstack-glance-api.service \
      openstack-glance-registry.service
    # systemctl start openstack-glance-api.service \
      openstack-glance-registry.service

##3.4 Kiểm chứng lại việc cài đặt glance

Khai báo biến môi trường cho dịch vụ `glance`

    # . admin-openrc 

Tải file image cho `glance`.

    # wget http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img

Upload file image vừa tải về

    # openstack image create "cirros" \
      --file cirros-0.3.4-x86_64-disk.img \
      --disk-format qcow2 --container-format bare \
      --public

Kiểm tra lại image đã có hay chưa

    # openstack image list
    
    +--------------------------------------+--------+--------+
    | ID                                   | Name   | Status |
    +--------------------------------------+--------+--------+
    | 38047887-61a7-41ea-9b49-27987d5e8bb9 | cirros | active |
    +--------------------------------------+--------+--------+

#4. Cài đặt Nova
##4.1 Tạo database và endpoint cho nova
Đăng nhập vào database với quyền root

    mysql -u root -p

Tạo database và gán các quyền cho user `nova` trong database

    CREATE DATABASE nova_api;
    CREATE DATABASE nova;
    GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' \
      IDENTIFIED BY 'bkcloud';
    GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' \
      IDENTIFIED BY 'bkcloud';
    GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' \
      IDENTIFIED BY 'bkcloud';
    GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' \
      IDENTIFIED BY 'bkcloud';
      exit;

Khai báo biến môi trường

    # . admin-openrc

Tạo user, phân quyền và tạo endpoint cho dịch vụ `nova`

    openstack user create nova --domain default  --password bkcloud

Phân quyền cho tài khoản `nova`

    openstack role add --project service --user nova admin

Tạo service có tên là `nova`

openstack service create --name nova --description "OpenStack Compute" compute

Tạo endpoint

    openstack endpoint create --region RegionOne compute public http://controller:8774/v2.1/%\(tenant_id\)s
    
    openstack endpoint create --region RegionOne compute internal http://controller:8774/v2.1/%\(tenant_id\)s
    

##4.2 Cài đặt các gói và cấu hình

Cài đặt gói phần mềm

    # yum install openstack-nova-api openstack-nova-conductor \
      openstack-nova-console openstack-nova-novncproxy \
      openstack-nova-scheduler openstack-nova-compute

Chỉnh sửa file /etc/nova/nova.conf như sau:

    [DEFAULT]
    . . .
    enabled_apis = osapi_compute,metadata
    transport_url = rabbit://openstack:bkcloud@controller
    auth_strategy = keystone
    my_ip = 10.10.10.50
    use_neutron = True
    firewall_driver = nova.virt.firewall.NoopFirewallDriver
    
    [api_database]
    ...
    connection = mysql+pymysql://nova:bkcloud@controller/nova_api
    
    [database]
    ...
    connection = mysql+pymysql://nova:bkcloud@controller/nova
    
    [keystone_authtoken]
    ...
    auth_uri = http://controller:5000
    auth_url = http://controller:35357
    memcached_servers = controller:11211
    auth_type = password
    project_domain_name = Default
    user_domain_name = Default
    project_name = service
    username = nova
    password = bkcloud
    
    [vnc]
    ...
    enabled = True
	vncserver_listen = 0.0.0.0
	vncserver_proxyclient_address = $my_ip
	novncproxy_base_url = http://192.168.2.50:6080/vnc_auto.html
    
    [glance]
    ...
    api_servers = http://controller:9292
    
    [oslo_concurrency]
    ...
    lock_path = /var/lib/nova/tmp

Đồng bộ database cho nova:

    # su -s /bin/sh -c "nova-manage api_db sync" nova
    # su -s /bin/sh -c "nova-manage db sync" nova

Cuối cùng, xác định xem máy của bạn hỗ trợ cách thức ảo hóa nào, chạy lênh:

    egrep -c '(vmx|svm)' /proc/cpuinfo

Nếu kết quả trả về là 1 hay lớn hơn, thì node compute của bạn đã hỗ trợ tăng tốc phần cứng mà không cần các cấu hình bổ sung. Nhưng nếu kế quả trả về là 0, nghĩa là node compute của bạn không hỗ trợ tăng tốc phần cứng, bạn phải cấu hình libvirt sử dụng QEMU thay vì KVM. Khi đó, phải sửa trong section `[libvirt]` ở file `/etc/nova/nova.conf` như sau:

    [libvirt]
    ...
    virt_type = qemu

##4.3 Kết thúc bước cài đặt và cấu hình nova

Khởi động lại các dịch vụ của `nova` sau khi cài đặt & cấu hình `nova`

    # systemctl enable openstack-nova-api.service \
      openstack-nova-consoleauth.service openstack-nova-scheduler.service \
      openstack-nova-conductor.service openstack-nova-novncproxy.service openstack-nova-compute.service
      
    # systemctl start openstack-nova-api.service \
      openstack-nova-consoleauth.service openstack-nova-scheduler.service \
      openstack-nova-conductor.service openstack-nova-novncproxy.service openstack-nova-compute.service

Khai báo biến môi trường

    # . admin-openrc

Kiểm tra lại các dịch vụ của nova đã được cài đặt thành công hay chưa:

    # openstack compute service list
    +----+------------------+----------+----------+---------+-------+----------------------------+
    | ID | Binary           | Host     | Zone     | Status  | State | Updated At                 |
    +----+------------------+----------+----------+---------+-------+----------------------------+
    |  1 | nova-conductor   | RAPID031 | internal | enabled | up    | 2017-03-10T04:58:27.000000 |
    |  3 | nova-consoleauth | RAPID031 | internal | enabled | up    | 2017-03-10T04:58:26.000000 |
    |  4 | nova-scheduler   | RAPID031 | internal | enabled | up    | 2017-03-10T04:58:26.000000 |
    |  6 | nova-compute     | RAPID031 | nova     | enabled | up    | 2017-03-10T04:58:24.000000 |
    +----+------------------+----------+----------+---------+-------+----------------------------+

#5. Cài đặt neutron
Có 2 cơ chế cung cấp network cho các máy ảo là:

 - Provider network (không sử dụng L3 agent trong Neutron)
 - Self-service network:

Trong hướng dẫn này sẽ lựa chọn cơ chế Self-service để viết tài liệu


##5.1 Tạo database, user và endpoint cho neutron

Đăng nhập vào MySQL

	mysql -uroot -p

Tạo database và phân quyền

	CREATE DATABASE neutron;
	GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'bkcloud';
	GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'bkcloud';        
	FLUSH PRIVILEGES;
	exit;

Khai báo biến môi trường

	# . admin-openrc

Tạo tài khoản tên `neutron`, thêm tài khoản `neutron` vào project `service` với quyền của  `admin`:

    # openstack user create neutron --domain default --password bkcloud
    # openstack role add --project service --user neutron admin

Tạo dịch vụ tên là `neutron`

    openstack service create --name neutron --description "OpenStack Networking" network

Tạo các endpoint cho dịch vụ neutron trong keystone

    # openstack endpoint create --region RegionOne network public http://controller:9696
    
    # openstack endpoint create --region RegionOne network internal http://controller:9696
        
    # openstack endpoint create --region RegionOne network admin http://controller:9696

##5.2 Cấu hình và cài đặt neutron

Cài đặt các gói phần mềm của `neutron`

    # yum install openstack-neutron openstack-neutron-ml2 \
      openstack-neutron-linuxbridge ebtables

Cấu hình cho dịch vụ `neutron`. Chỉnh sửa file `/etc/neutron/neutron.conf` như sau:


    [DEFAULT]
    ...
    core_plugin = ml2
    service_plugins = router
    allow_overlapping_ips = True
    transport_url = rabbit://openstack:bkcloud@controller
    auth_strategy = keystone
    notify_nova_on_port_status_changes = True
    notify_nova_on_port_data_changes = True
    
    [database]
    ...
    connection = mysql+pymysql://neutron:bkcloud@controller/neutron
    
    [keystone_authtoken]
    ...
    auth_uri = http://controller:5000
    auth_url = http://controller:35357
    memcached_servers = controller:11211
    auth_type = password
    project_domain_name = Default
    user_domain_name = Default
    project_name = service
    username = neutron
    password = bkcloud
    
    [nova]
    ...
    auth_url = http://controller:35357
    auth_type = password
    project_domain_name = Default
    user_domain_name = Default
    region_name = RegionOne
    project_name = service
    username = nova
    password = bkcloud
    
    [oslo_concurrency]
    ...
    lock_path = /var/lib/neutron/tmp

Cài đặt và cấu hình plug-in `Modular Layer 2 (ML2)`. Chỉnh sửa file `/etc/neutron/plugins/ml2/ml2_conf.ini` như sau:

    [ml2]
    ...
    type_drivers = flat,vlan,vxlan
    tenant_network_types = vxlan
    mechanism_drivers = linuxbridge,l2population
    extension_drivers = port_security
    
    [ml2_type_flat]
    ...
    flat_networks = provider
    
    [ml2_type_vxlan]
    ...
    vni_ranges = 1:1000
    
    [securitygroup]
    ...
    enable_ipset = True

Cấu hình `linuxbridge`. Chỉnh sửa file `/etc/neutron/plugins/ml2/linuxbridge_agent.ini` như sau:

    [linux_bridge]
    physical_interface_mappings = provider:eth1
    
    [vxlan]
    enable_vxlan = True
    local_ip = 10.10.10.50
    l2_population = True
    
    [securitygroup]
    ...
    enable_security_group = True
    firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver

Cấu hình `l3-agent`. Chỉnh sửa file `/etc/neutron/l3_agent.ini` như sau:

    [DEFAULT]
    ...
    interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver

Cấu hình `DHCP Agent`. Chỉnh sửa file `/etc/neutron/dhcp_agent.ini` như sau:

    [DEFAULT]
    ...
    interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver
    dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
    enable_isolated_metadata = True

Cấu hình `metadata agent`. Chỉnh sửa file `/etc/neutron/metadata_agent.ini` như sau:

    [DEFAULT]
    ...
    nova_metadata_ip = controller
    metadata_proxy_shared_secret = bkcloud

Cấu hình dịch vụ Compute (Nova) để có thể sử dụng Neutron. Chỉnh sửa file `/etc/nova/nova.conf`, trong phần [neutron], cấu hình như sau:

    [neutron]
    ...
    url = http://controller:9696
    auth_url = http://controller:35357
    auth_type = password
    project_domain_name = Default
    user_domain_name = Default
    region_name = RegionOne
    project_name = service
    username = neutron
    password = bkcloud
    service_metadata_proxy = True
    metadata_proxy_shared_secret = bkcloud

Đồng bộ database cho `neutron`

    # ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
    
    # su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
      --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron

Khởi động lại `nova-api`

    # systemctl restart openstack-nova-api.service

Khởi động lại các dịch vụ của `neutron`

    # systemctl enable neutron-server.service \
      neutron-linuxbridge-agent.service neutron-dhcp-agent.service \
      neutron-metadata-agent.service neutron-l3-agent.service
      
    # systemctl start neutron-server.service \
      neutron-linuxbridge-agent.service neutron-dhcp-agent.service \

Kiểm tra lại hoạt động của các dịch vụ trong `neutron`

    # openstack network agent list
    +--------------------------------------+--------------------+----------+-------------------+-------+-------+---------------------------+
    | ID                                   | Agent Type         | Host     | Availability Zone | Alive | State | Binary                    |
    +--------------------------------------+--------------------+----------+-------------------+-------+-------+---------------------------+
    | 28d9083f-7ac2-4e3e-a601-8c239ee84cef | Metadata agent     | RAPID031 | None              | True  | UP    | neutron-metadata-agent    |
    | 4d10137f-d3e8-4752-9556-834f76b655a1 | DHCP agent         | RAPID031 | nova              | True  | UP    | neutron-dhcp-agent        |
    | 86f23cf9-182d-433a-9264-492a697aecc4 | L3 agent           | RAPID031 | nova              | True  | UP    | neutron-l3-agent          |
    | a544540d-d2af-4cbe-a346-501354841c7c | Linux bridge agent | RAPID031 | None              | True  | UP    | neutron-linuxbridge-agent |
    +--------------------------------------+--------------------+----------+-------------------+-------+-------+---------------------------+

#6. Cài đặt Horizon (dashboard)

Cài đặt các thành phần cho dashboad

    # yum install openstack-dashboard

Tìm các dòng sau trong file `/etc/openstack-dashboard/local_settings` và chỉnh sửa như bên dưới

```sh
OPENSTACK_HOST = "controller"
```

```sh
ALLOWED_HOSTS = ['*', ]
```

```sh
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'

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
OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True
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

Khởi động lại web server 

    # systemctl restart httpd.service memcached.service

Mở web với địa chỉ http://192.168.2.50/dashboard để truy cập vào horizon

**Lưu ý**: Nếu không truy cập vào được horizon, có thể do host đang chạy firewalld và SELinux, chúng ta cần disable chúng. Xem chi tiết tại [đây](https://www.server-world.info/en/note?os=CentOS_7&p=openstack_newton&f=10)

#6. Tài liệu tham khảo
[1] https://docs.openstack.org/newton/install-guide-rdo/

[2] https://www.server-world.info/en/note?os=CentOS_7&p=openstack_newton&f=10
