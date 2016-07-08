#OpenVswitch Installation guide self service
##Cấu hình phần cứng
![self-service-ovs-network.png](./img/self-service-ovs-network.png)
![self-service-ovs-compute.png](./img/self-service-ovs-compute.png)
!!Lưu ý Trong bài viết không sử dụng eth2, hình ảnh có tính tạm thời và sẽ được sửa trong thời gian sớm nhất.

Triển khai 2 card eth

eth0 sẽ để triển khai management network + VXLAN-GRE tunnels 10.10.10.10 và 10.10.10.11

eth1 sẽ triển khai internet 192.168.2.10 và 192.168.2.11 + Flat + VLAN


Để bắt đầu cài đặt trên controller node, ta cần tạo cơ sở dữ liệu cho Neutron:
##Cấu hình cài đặt neutron trên controller node

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
####Tải về các dịch vụ của neutron
Ta tiến hành tải về các dịch vụ của neutron trên controller node:
```sh
	source admin.sh
	apt-get update
	apt-get -y install neutron-server neutron-plugin-ml2 neutron-l3-agent neutron-dhcp-agent neutron-metadata-agent neutron-plugin-openvswitch-agent
```

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
####Cấu hình Modular Layer 2 (ML2) plug-in
Cấu hình file ```/etc/neutron/plugins/ml2_conf.ini```
```sh
[ml2]
type_drivers = flat,vlan,gre,vxlan
tenant_network_types = vlan,gre,vxlan
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
Tạo br-vlan và br-ex, kết nối br-vlan tới eth2 và br-ex tới eth1
```sh
ovs-vsctl add-br br-ex
ovs-vsctl add-port br-ex eth1
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
##Cấu hình trên compute node

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
ovs-vsctl add-br br-ex
ovs-vsctl add-port br-ex eth1
```
####Kết thúc cài đặt trên compute node
- Khởi động lại dịch vụ nova-compute
```sh
	service nova-compute restart
```
- Khởi động lại linux-bridge agent
```sh
	service neutron-openvswitch-agent restart
```
##Kiểm tra hoạt động của dịch vụ neutron
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

Các bạn có thể sử dụng mạng ảo này để chạy máy ảo.
Đã test thử và các mạng Flat, VXLAN và VLAN hoạt động bình thường