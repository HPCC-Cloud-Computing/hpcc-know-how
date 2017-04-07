## Tìm hiểu về Block Storage service 

(Tài liệu này hướng dẫn cách cài đặt dịch vụ Cinder cho OpenStack bản Mitaka, trong mô hình 2 node là node contrller và node compute. Để cài dịch vụ Cinder, cần có 1 node storage để chứa ổ đĩa vật lý thât để cung cấp cho các máy ảo, trong docs này mình sẽ gộp chung node compute cùng với node storage bằng cách tạo thêm 1 hardware cho node compute (hoặc nếu các bạn không muốn tạo thêm có thể sử dụng ngay trên phần cứng vật lý của node compute đang chứa hệ điều hành, nhưng cấu hình sẽ khác với docs này)).


## Mục Lục

- [1. Về Block Storage](#over_view_cinder)
- [2. Cinder](#cinder)
  * [2.1 Cinder component](#cinder_component)
  * [2.2 Cinder workflow](#cinder_workflow)
- [3. Cài đặt và cấu hình trên node controller](#install_controller)
  * [3.1 Tạo database, user, service và API endpoint](#database)
  * [3.2 Cài đặt và cấu hình các component](#install_configure)
- [4. Cài đặt và cấu hình trên node storage  ](#install_storage)
  * [4.1 Tạo ổ cứng mới cho node storage](#create_hardware)
  * [4.2 Cài đặt và cấu hình](#configure_storage)
- [5. Tạo volume](#create_volume)
- [6. Mount volume vào máy ảo, tạo data và detach để kiểm tra](#mount_and_check)
  * [6.1 Mount volume vào máy ảo và tạo data](#mount)
  * [6.2 Detach volume và gán vào 1 instance khác để kiểm tra](#detach)


<a name="over_view_cinder"></a>
##1. Về Block Storage

Dịch vụ `OpenStack Block Storage`hoạt động thông qua sự tương tác của 1 loạt các daemon processes được gọi là `cinder-*` nằm trên 1 máy chủ hoặc nhiều máy.
Để quản lý dịch vụ OpenStack Block Storage, bạn nên hiểu 1 số khái niệm (như `Ephemeral Storage` và `Persistent Storage`). Bạn phải xác định rõ là bạn sẽ cài đăt trên 1 node hay nhiều node khi bạn cấu hình cho dịch vụ Block Storage trong OpenStack. 
OpenStack Block Storage cho phép bạn bổ sung thêm các khối lưu trữ cho các OpenStack Compute instance. Dịch vụ này cũng tương tụ như Amazon EC2 Elastic Block Storage (EBS) cung cấp.

  * **Ephemeral storage**: nếu bạn chỉ triển khai dịch vụ OpenStack Compute (nova) thì user của bạn sẽ không có quyền truy cập vào bất kỳ form nào của persistent storage theo mặc định. Các ổ đĩa liên kết với máy ảo của bạn lúc này là “ephemeral”, có nghĩa là các ổ đĩa này sẽ mất khi bạn xóa (terminated) máy ảo. (Lưu trữ ngắn hạn)
  
  * **Persistent storage**: các tài nguyên lưu trữ luôn tồn tại, không phụ thuộc vào trạng thái của 1 instance đang chạy. (Lưu trữ dài hạn)
  
(Ngoài Block Storage thì OpenStack còn hỗ trợ thêm 2 dạng lưu trữ khác là `Object Storage` và `File System Storage`)
  
![](./img/ephmeral vs persistent.png)

Block Storage cung cấp cho người dùng quyền truy cập vào các block-storage devide. Các user tương tác với block-storage bằng cách thêm các volume vào các instance của họ.

Các volume là liên tục: chúng có thể được tách ra từ 1 instance và gắn vào 1 instance khác mà data trong đó vẫn giữ nguyên. Block storage được thực hiện trong project Cinder của OpenStack, Block Storage hỗ trợ multiple back end trong form của các driver. Do đó, bạn nên chọn storage back end được hỗ trợ bởi 1 Block Storage driver.

Hầu hết các block storage driver cho phép instance truy cập trực tiếp vào thiết bị phần cứng cơ sở. Điều này giúp tăng tốc quá trình read/wrtie IO.

![](./img/persistent support back-end.png)

(Link tìm hiểu thêm: http://docs.openstack.org/ops-guide/arch_storage.html)

<a name="cinder"></a>
##2. Cinder component

<a name="cinder_component"></a>
###2.1 Cinder component

OpenStack Block Storege service (cinder) dùng để thêm các persistent storage đến 1 máy ảo. Block Storage cung cấp 1 cơ  sơ hạ tầng để quản lý các volume và tương tác với OpenStack Compute để cung cấp volume cho các instance. Dịch vụ này cũng cho phép quản lý volume snapshots và volume types.


Các thành phần của cinder:

![](./img/cinder_component.png)

**cinder-api:** chấp nhận các API request và chuyển chúng đến cho cinder-volume để họat động

**cinder-volume:** tương tác trực tiếp với Block Storage service, và các tiến trình như cinder-scheduler thông qua 1 message queue. Dịch vụ cinder-volume phản hồi lại các message tới, sau đó đọc và viết các request gửi tới Block Storage để duy trì trạng thái. Nó có thể tương tác với nhiều nhà cung cấp storage thông qua 1 driver architecture.

**cinder-scheduler daemon:** chọn ra node cung cấp storage tối ưu để tạo các volume. Dịch vụ này tương tự như nova-scheduler.

**cinder-backup daemon:** cung cấp dịch vụ backing up volume. Cũng giống như cinder-volume, cinder-backup cũng có thể tương tác với nhiều nhà cung cấp storage thông qua 1 driver architecture.

**messaging queue:** định hướng cho các tin nhắn giữa các tiến trình của Block Storage.

<a name="cinder_workflow"></a>
###2.2 Cinder workflow

![](./img/cinder_and_nova.png)

1 volume sẽ được tạo thông qua `cinder create command`. Dòng lệnh này sẽ tạo ra 1 `logical volume` (lv) vào trong `volume group` (VG) với tên gọi “`cinder-volumes`”. Để attach volume mới được tạo này vào 1 instance, ta sử dụng `nova volume-attach command`. Lệnh này sẽ tạo ra 1 `iSCSI` duy nhất để truyền data giữa node compute và node storage. 

Sau khi attach volume thì node compute chứa instance được attach có thể active iCSI để truyền data đến node storage. Như trong hình trên thì volume được gán vào instance đã được mount vào ổ đĩa `/dev/vda` trên instance.

<a name="install_controller"></a>
##3. Cài đặt và cấu hình trên node controller

Trước khi cài đặt và cấu hình cho dịch vụ Block Storage, ta phải tạo database, user và các API endpoint.

<a name="database"></a>
###3.1 Tạo database, user, service và API endpoint

- Đăng nhập vào databasa server và tạo database cho cinder:

```sh
$ mysql -u root -p

CREATE DATABASE cinder;
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'localhost' \
  IDENTIFIED BY 'Welcome123';
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'%' \
  IDENTIFIED BY 'Welcome123';
exit;
```
- Source lại  file admin.sh
```sh
$ . admin-openrc
```
- Tạo 1 cinder user: (lưu ý user pass work các bạn nhập là Welcome123 cho đồng bộ)
```sh
$ openstack user create --domain default --password-prompt cinder
User Password:
Repeat User Password:
+-----------+----------------------------------+
| Field     | Value                            |
+-----------+----------------------------------+
| domain_id | e0353a670a9e496da891347c589539e9 |
| enabled   | True                             |
| id        | bb279f8ffc444637af38811a5e1f0562 |
| name      | cinder                           |
+-----------+----------------------------------+
```
-  Thêm quyền admin cho cinder user:
```sh
$ openstack role add --project service --user cinder admin
```
- Tạo các entity của dịch vụ cinder và cinderv2 (do Block Storage yêu cầu có 2 service entities)
```sh
$ openstack service create --name cinder \
  --description "OpenStack Block Storage" volume
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Block Storage          |
| enabled     | True                             |
| id          | ab3bbbef780845a1a283490d281e7fda |
| name        | cinder                           |
| type        | volume                           |
+-------------+----------------------------------+
$ openstack service create --name cinderv2 \
  --description "OpenStack Block Storage" volumev2
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Block Storage          |
| enabled     | True                             |
| id          | eb9fd245bdbc414695952e93f29fe3ac |
| name        | cinderv2                         |
| type        | volumev2                         |
+-------------+----------------------------------+
```
- Tạo các endpoint API:

  * Tạo endpoint cho cinder
	```sh
  $ openstack endpoint create --region RegionOne \
    volume public http://controller:8776/v1/%\(tenant_id\)s
  +--------------+-----------------------------------------+
  | Field        | Value                                   |
  +--------------+-----------------------------------------+
  | enabled      | True                                    |
  | id           | 03fa2c90153546c295bf30ca86b1344b        |
  | interface    | public                                  |
  | region       | RegionOne                               |
  | region_id    | RegionOne                               |
  | service_id   | ab3bbbef780845a1a283490d281e7fda        |
  | service_name | cinder                                  |
  | service_type | volume                                  |
  | url          | http://controller:8776/v1/%(tenant_id)s |
  +--------------+-----------------------------------------+

  $ openstack endpoint create --region RegionOne \
    volume internal http://controller:8776/v1/%\(tenant_id\)s
  +--------------+-----------------------------------------+
  | Field        | Value                                   |
  +--------------+-----------------------------------------+
  | enabled      | True                                    |
  | id           | 94f684395d1b41068c70e4ecb11364b2        |
  | interface    | internal                                |
  | region       | RegionOne                               |
  | region_id    | RegionOne                               |
  | service_id   | ab3bbbef780845a1a283490d281e7fda        |
  | service_name | cinder                                  |
  | service_type | volume                                  |
  | url          | http://controller:8776/v1/%(tenant_id)s |
  +--------------+-----------------------------------------+

  $ openstack endpoint create --region RegionOne \
    volume admin http://controller:8776/v1/%\(tenant_id\)s
  +--------------+-----------------------------------------+
  | Field        | Value                                   |
  +--------------+-----------------------------------------+
  | enabled      | True                                    |
  | id           | 4511c28a0f9840c78bacb25f10f62c98        |
  | interface    | admin                                   |
  | region       | RegionOne                               |
  | region_id    | RegionOne                               |
  | service_id   | ab3bbbef780845a1a283490d281e7fda        |
  | service_name | cinder                                  |
  | service_type | volume                                  |
  | url          | http://controller:8776/v1/%(tenant_id)s |
  +--------------+-----------------------------------------+
  ```
  
  * Tạo endpoint cho cinderv2

  ```sh
  $ openstack endpoint create --region RegionOne \
    volumev2 public http://controller:8776/v2/%\(tenant_id\)s
  +--------------+-----------------------------------------+
  | Field        | Value                                   |
  +--------------+-----------------------------------------+
  | enabled      | True                                    |
  | id           | 513e73819e14460fb904163f41ef3759        |
  | interface    | public                                  |
  | region       | RegionOne                               |
  | region_id    | RegionOne                               |
  | service_id   | eb9fd245bdbc414695952e93f29fe3ac        |
  | service_name | cinderv2                                |
  | service_type | volumev2                                |
  | url          | http://controller:8776/v2/%(tenant_id)s |
  +--------------+-----------------------------------------+
  
  $ openstack endpoint create --region RegionOne \
    volumev2 internal http://controller:8776/v2/%\(tenant_id\)s
  +--------------+-----------------------------------------+
  | Field        | Value                                   |
  +--------------+-----------------------------------------+
  | enabled      | True                                    |
  | id           | 6436a8a23d014cfdb69c586eff146a32        |
  | interface    | internal                                |
  | region       | RegionOne                               |
  | region_id    | RegionOne                               |
  | service_id   | eb9fd245bdbc414695952e93f29fe3ac        |
  | service_name | cinderv2                                |
  | service_type | volumev2                                |
  | url          | http://controller:8776/v2/%(tenant_id)s |
  +--------------+-----------------------------------------+
  
  $ openstack endpoint create --region RegionOne \
    volumev2 admin http://controller:8776/v2/%\(tenant_id\)s
  +--------------+-----------------------------------------+
  | Field        | Value                                   |
  +--------------+-----------------------------------------+
  | enabled      | True                                    |
  | id           | e652cf84dd334f359ae9b045a2c91d96        |
  | interface    | admin                                   |
  | region       | RegionOne                               |
  | region_id    | RegionOne                               |
  | service_id   | eb9fd245bdbc414695952e93f29fe3ac        |
  | service_name | cinderv2                                |
  | service_type | volumev2                                |
  | url          | http://controller:8776/v2/%(tenant_id)s |
  +--------------+-----------------------------------------+
  ```
  
<a name="install_configure"></a>
###3.2 Cài đặt và cấu hình các component

- Cài đặt package:
```sh
apt-get install cinder-api cinder-scheduler
```
- Sao lưu file cấu hình trước khi chỉnh sửa:
```sh
cp /etc/cinder/cinder.conf /etc/cinder/cinder.conf.orig
```
- Chỉnh sửa file `/etc/cinder/cinder.conf` như dưới đây (các dấu ... thể hiện các diongf option khác chứ không phải là1 dòng configure)

  * Trong section `[database]`, cấu hình lại database access:
  ```sh
  [database]
  ...
  connection = mysql+pymysql://cinder:Welcome123@controller/cinder
  ```
  * Trong section `[DEFAULT]` và `[oslo_messaging_rabbit]`, chỉnh sửa để truy cập được đến RabbitMQ
  ```sh
  [DEFAULT]
  ...
  rpc_backend = rabbit

  [oslo_messaging_rabbit]
  ...
  rabbit_host = controller
  rabbit_userid = openstack
  rabbit_password = Welcome123
  ```
  * Trong section `[DEFFAULT]` và `[keystone_authtoken]`. chỉnh sửa để có thể truy cập tới dịch vụ xác thực
  ```sh
  [DEFAULT]
  ...
  auth_strategy = keystone
  
  [keystone_authtoken]
  ...
  auth_uri = http://controller:5000
  auth_url = http://controller:35357
  memcached_servers = controller:11211
  auth_type = password
  project_domain_name = default
  user_domain_name = default
  project_name = service
  username = cinder
  password = Welcome123
  ```
  (Ngoài các option ở trên trong section `[keystone_authtoken]`, nếu còn các option khác trong section này thì các bạn comment hết lại hoặc xóa đi)

  * Trong section `[DEFAULT]`, chỉnh lại địa chỉ ip thành ip của node controller
  ```sh
  [DEFAULT]
  ...
  my_ip = 10.0.0.11
  ```
  * Trong section `[oslso_concurrency]`, chỉnh lại lock path:
  ```sh
  [oslo_concurrency]
  ...
  lock_path = /var/lib/cinder/tmp
  ```
- Lưu lại file `/etc/cinder/cinder.conf` và sau đó đồng bộ database cho cinder:
```sh
# su -s /bin/sh -c "cinder-manage db sync" cinder
```

<a name="configure_nova"></a>
###3.3 Cấu hình lại dịch vụ Compute để sử dụng Block Storage

-  Chỉnh sửa file `/etc/nova/nova.conf` như sau:
```sh
[cinder]
os_region_name = RegionOne
```
- Khởi động lại Compute API:
```sh
# service nova-api restart
```
- Khởi động lại các service của Block Storage:
```sh
# service cinder-scheduler restart
# service cinder-api restart
```
- Kiểm tra lại kết quả cài đặt:
```sh
root@controller:/home/hamanhdong# cinder service-list
+------------------+------------+------+---------+-------+----------------------------+-----------------+
|      Binary      |    Host    | Zone |  Status | State |         Updated_at         | Disabled Reason |
+------------------+------------+------+---------+-------+----------------------------+-----------------+
| cinder-scheduler | controller | nova | enabled |   up  | 2016-07-14T08:40:26.000000 |        -        |
+------------------+------------+------+---------+-------+----------------------------+-----------------+
```

<a name="install_storage"></a>
##4. Cài đặt và cấu hình trên node storage

Phần này hướng dẫn cách cài đặt và cấu hình cho dịch vụ Block Storage trên node storage (chứa tài nguyên vật lý thật). Hướng dẫn sử dụng /dev/sdb, nhưng bạn có thể thay thế 1 giá trj khác cho node cụ thể của bạn.

Trước khi cài đặt, bạn phải tạo thêm 1 ổ cứng cho node storage. Thực hiện như sau:


<a name="create_hardware"></a>
###4.1 Tạo ổ cứng mới cho node storage

1. Vào phần setting, chọn add thêm hardware

![](./img/create hardware .png)

2. Chọn next, trong mục virtual disk type, tích chọn `SCSI` sau đó bấm next

3. Trong cửa sổ Disk, tích chọn `Create new virtual` rồi bấm next

4.  Chọn kích thước của ổ cứng mà bạn muốn tạo (nên cân nhắc với lượng ổ cứng còn trống trên  máy bạn), sau đó ấn next và chọn finish.
Kết quả: 

![](./img/hardware result.png)


<a name="configure_storage"></a>
###4.2 Cài đặt và cấu hình

Dịch vụ Block Storage  sử dụng `LVM driver` và cung cấp logical volume cho các instance thông qua `iSCSI`.
(LVM cung cấp phương pháp phân bổ space trên thiết bị lưu trữ 1 cách linh hoạt hơn so với các chương trình phân vùng thông thường
LVM là 1 abstraction layer nằm giữa hệ điều hành và ổ đĩa cứng vật lý. Điều này có nghĩa là ổ cứng vật lý của bạn và các partitions không còn ràng buộc với ổ đĩa cứng và các partition mà chúng nằm trên. Thay vào đó, các ổ đĩa cứng và partition mà hệ điều hành của bạn “nhìn” thấy có thể là 1 số các ổ đĩa cứng riêng biệt gộp lại với nhau hoặc trong 1 phần mềm RAID bất kỳ.
Để quản lý LVM, có các GUI tool sẵn có nhưng để hiểu rõ những gì chạy bên dưới, bạn nên sử dụng các dòng lệnh.
Hầu hết các lệnh trong LVM tương tự nhau. Mỗi lệnh hợp lệ đều phải được bắt đầu bởi 1 trong các lệnh dưới:
	* Physical Volume = pv
	* Volume Group = vg
	* Logical Volume = lv
)

(SCSI là 1 disk protocol tạo “đường hầm” bên trong Ethernet, được hỗ trợ với các dịch vụ Compute, Object Storeage và Image)

+ **Chuẩn bị trước khi cài đặt và cấu hình:**

- Cài đặt gói tiện ích hỗ trợ:
```sh
# apt-get install lvm2
```
- Kiểm tra lại các ổ cứng đã có trên node storage:
```sh
root@compute1:/home/compute# fdisk -l

Disk /dev/sda: 32.2 GB, 32212254720 bytes
255 heads, 63 sectors/track, 3916 cylinders, total 62914560 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00073bf5

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048    60817407    30407680   83  Linux
/dev/sda2        60819454    62912511     1046529    5  Extended
/dev/sda5        60819456    62912511     1046528   82  Linux swap / Solaris

Disk /dev/sdb: 10.7 GB, 10737418240 bytes
255 heads, 63 sectors/track, 1305 cylinders, total 20971520 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000
```
Ổ cứng `/dev/sdb` trên kia chính là ổ cứng mà chúng ta đã tạo từ bước trước, chưa chứa gì (`/dev/sda` đã chứa hệ điều hành). Nếu chưa có bạn nên xem lại bước tạo ổ cứng bên trên.

- Tạo volume vật lý LVM /dev/sdb
```sh
# pvcreate /dev/sdb
Physical volume "/dev/sdb" successfully created
```
- Tạo LVM volume group cho cinder-volume:
```sh
# vgcreate cinder-volumes /dev/sdb
Volume group "cinder-volumes" successfully created
```
Dịch vụ Block Storage sẽ tạo volume hợp lý trong volume group này.

Chỉ có các instance mới có thể truy cập vào các volume của BlockStorage. Tuy nhiên, hệ điều hành bên dưới lại quản lý các thiết bị liên kết với các volume. Theo mặc định,công cụ `LVM volume scanning` sẽ quét các thư mục `/dev` cho các block storage device để chứa các volume. Nếu các project sử dụng LVM trên các volume của project đó, công cụ scanning sẽ tự phát hiện các volume và cố gắng để cache chúng. Điều này có thể gây ra 1 loạt các vấn đề với cả hệ điều hành và các project volume. Bạn phải cấu hình lại LVM để chỉ scan các device có chứa cinder-volume trong group.

- Chỉnh sửa file `/etc/lvm/lvm.conf` theo các bước dưới đây:

  * Trong phần devices, thêm 1 bộ lọc để chỉ chấp nhận thiết bị `/dev/sdb` và từ chối tất cả các thiết bị khác.
  ```sh
  devices {
  ...
  filter = [ "a/sdb/", "r/.*/"]
  ```
  Mỗi item trong mảng lọc bắt đầu bằng ký tự `'a'` để chấp nhận (accept), bắt đầu bằng ký tự `'r'` là để từ chối (reject). Mảng phải kết thúc với ký hiệu `“r./*/”` để từ chối tất cả các thiết bị còn lại. Bạn có thể sử dụng lênh `vgs -vvvv`  để kiểm tra bộ lọc.

  **Chú ý:**
	* Nếu node storage của bạn sử dụng LVM trên ổ đĩa chứa hệ điều hành, bạn phải thêm 	thiết bị liên quan vào bộ lọc. Ví dụ: nếu `/dev/sda` chứa hệ điều hành thì bạn phải thêm 	vào như sau:
  ```sh
	filter = [ "a/sda/", "a/sdb/", "r/.*/"]
  ```
	* Nếu node compute của bạn sử dụng LVM trên ổ đĩa chứa hệ điều hành, bạn cũng phải 	sửa lại bộ lọc trong file `/etc/lvm/lvm.conf` trên các node đó để thêm vào như trên. Ví dụ 	nếu hệ điều hành ở /dev/sda thì bạn phải thêm:
	```sh
	filter = [ "a/sda/", "r/.*/"]
	 ```
  
- Cài đặt package:
```sh
# apt-get install cinder-volume
```
 - Sao lưu file `/etc/lvm/lvm.conf` trước khi chỉnh sửa:
```sh
cp /etc/lvm/lvm.conf /etc/lvm/lvm.conf.orig
```
- Chỉnh sửa file `/etc/cinder/cinder.conf` như sau:

  * Trong section `[database]`, cấu hình để truy cập được đến database
  ```sh
  [database]
  ...
  connection = mysql+pymysql://cinder:Welcome123@controller/cinder
  ```
  * Trong section `[DEFAULT]` và `[oslo_messaging_rebbit]`, cấu hình để truy cập được tới RabbitMQ message queue
  ```sh
  [DEFAULT]
  ...
  rpc_backend = rabbit
  
  [oslo_messaging_rabbit]
  ...
  rabbit_host = controller
  rabbit_userid = openstack
  rabbit_password = Welcome123
  ```
  * Trong section `[DEFFAULT]` và section `[keystone_authtoken]`, cấu hình để truy cập được tới dịch vụ xác thực:
    ```sh
  [DEFAULT]
  ...
  auth_strategy = keystone
  
  [keystone_authtoken]
  ...
  auth_uri = http://controller:5000
  auth_url = http://controller:35357
  memcached_servers = controller:11211
  auth_type = password
  project_domain_name = default
  user_domain_name = default
  project_name = service
  username = cinder
  password = Welcome123
  ```
  (**Chú ý:** Ngoài các option ở trên trong section [keystone_authtoken], nếu còn các option khác trong section này thì các bạn comment hết lại hoặc xóa đi)

  * Trong section `[DEFFAULT]`, chỉnh lại địa chỉ IP là Ip của node storage (trong trường hợp này là Ip của node compute)
  ```sh
  [DEFAULT]
  ...
  my_ip = 10.10.10.41
  ```
  * Trong section `[lvm]`, cấu hình LVM back-end với LVM driver, volume group và giao thức iSCSI:
  ```sh
  [lvm]
  ...
  volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
  volume_group = cinder-volumes
  iscsi_protocol = iscsi
  iscsi_helper = tgtadm
  ```
  * Trong section `[DEFFAULT]`, enable cho LVM back-end và thêm vào vị trí của dịch vụ Image
  ```sh
  [DEFAULT]
  …
  
  enabled_backends = lvm
  glance_api_servers = http://controller:9292
  ```
  * Trong section `[oslo_concurrency]`, chỉnh lại lock path:
  ```sh
  [oslo_concurrency]
  ...
  lock_path = /var/lib/cinder/tmp
  ```
  * Lưu lại file `/etc/lvm/lvm.conf`, kết thúc quá trình cấu hình.

- Khởi động lại các service của Cinder:
```sh
# service tgt restart
# service cinder-volume restart
```
- Kiểm tra lại kết quả cài đặt bằng lệnh:
```sh
root@compute1:/home/compute# cinder service-list
+------------------+------------+------+---------+-------+----------------------------+-----------------+
|      Binary      |    Host    | Zone |  Status | State |         Updated_at         | Disabled Reason |
+------------------+------------+------+---------+-------+----------------------------+-----------------+
| cinder-scheduler | controller | nova | enabled |   up  | 2016-07-14T15:07:00.000000 |        -        |
+------------------+------------+------+---------+-------+----------------------------+-----------------+
```

<a name="create_volume"></a>
##5. Tạo volume

1 volume là 1 khối thiết bị để lưu trữ, giống như USB. Bạn có thể gán chúng cho 1 instance bất kỳ. 

Để tạo 1 volume, source file demo.sh để tạo trên project demo
```sh
$ . demo.sh
```
Tạo 1 volume mới với tên là `volumev0`, kích thước `2Gb`
```sh
$ openstack volume create --size 2 volume0
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| attachments         | []                                   |
| availability_zone   | nova                                 |
| bootable            | false                                |
| consistencygroup_id | None                                 |
| created_at          | 2016-07-22T14:30:48.391027           |
| description         | None                                 |
| encrypted           | False                                |
| id                  | 603a747d-7b29-44ff-90cd-ce097246ef1b |
| multiattach         | False                                |
| name                | volume0                              |
| properties          |                                      |
| replication_status  | disabled                             |
| size                | 2                                    |
| snapshot_id         | None                                 |
| source_volid        | None                                 |
| status              | creating                             |
| type                | None                                 |
| updated_at          | None                                 |
| user_id             | 684286a9079845359882afc3aa5011fb     |
+---------------------+--------------------------------------+
```
Kiểm tra lại trạng thái của volume mới tạo (lưu ý phải ở trạng thái `available`)
```sh
$ openstack volume list
+--------------------------------------+--------------+-----------+------+-------------+
| ID                                   | Display Name | Status    | Size | Attached to |
+--------------------------------------+--------------+-----------+------+-------------+
| 603a747d-7b29-44ff-90cd-ce097246ef1b | volume0      | available |    2 |             |
+--------------------------------------+--------------+-----------+------+-------------+
```
Tạo 2 máy ảo là `test_cinder1` và `test_cinder2` và kiểm tra `id` của mỗi máy bằng lệnh:
```sh
root@controller:/home/ha# nova list
+--------------------------------------+--------------+---------+------------+-------------+-------------------------------------------+
| ID                                   | Name         | Status  | Task State | Power State | Networks                                  |
+--------------------------------------+--------------+---------+------------+-------------+-------------------------------------------+
| 72a4860e-33d9-402a-9e4e-710683bad297 | test_cinder1 | SHUTOFF | -          | Shutdown    | selfservice=192.168.20.101, 192.168.1.105 |
| 28a63719-9900-45e2-8266-ddab9593f2c7 | test_cinder2 | ACTIVE  | -          | Running     | selfservice=192.168.20.102, 192.168.1.106 |
+--------------------------------------+--------------+---------+------------+-------------+-------------------------------------------+
```
Lấy `id` của máy ảo thứ nhất (tên là `test_cinder1`) để `attach volumev0` vào:

(Ta có thể `attach volume` cho máy ảo ngay cả khi nó `đang chạy`)
```dh
$ openstack server add volume 72a4860e-33d9-402a-9e4e-710683bad297 volumev0
```
Kiểm tra lại trạng thái của volumev0 sau khi đã attach vào máy ảo `test_cinder1`:
```sh
root@controller:/home/ha# openstack volume list
+--------------------------------------+--------------+--------+------+---------------------------------------+
| ID                                   | Display Name | Status | Size | Attached to                           |
+--------------------------------------+--------------+--------+------+---------------------------------------+
| 603a747d-7b29-44ff-90cd-ce097246ef1b | volume0      | in-use |    2 | Attached to test_cinder1 on /dev/vdb  |
+--------------------------------------+--------------+--------+------+---------------------------------------+
```
Tiến hành bật máy ảo `test_cinder1` lên và login vào, kiểm tra trạng thái của volume vừa gán cho máy ảo `test_cinder1` để biết nó được gán vào ổ đĩa nào bằng lệnh:
```sh
$ sudo fdisk -l
```
Kết quả:

![](./img/test1_login.png)

Dựa vào kết quả trên, ta thấy volume đã được gán vào ổ đĩa /dev/vdb trên máy ảo.

<a name="mount_and_check"></a>
##6. Mount volume vào máy ảo, tạo data và detach để kiểm tra

<a name="mount"></a>
###6.1 Mount volume vào máy ảo và tạo data

Tạo file system với định dạng ext4 cho ổ đĩa /dev/vdb (ổ đĩa mà volume0 đã được gán vào)

![](./img/mkfs.png)

Sau đó kiểm tra lại kết quả:

![](./img/blkid.png)

Tiến hành mount volume cho máy ảo:

Đầu tiên, ta tạo 1 thư mục mới để chứa volume sẽ được mount vào.
Ví dụ, tạo 1 thư mục `test` trong `/mnt`, sau đó mount `volume0` được gán trên ổ đĩa `/dev/vdb` vào thư mục test mới tạo:
```sh
$ sudo mount -t ext4 /dev/vdb /mnt/test
```
Kiểm tra lại kết quả bằng lệnh:

![](./img/df -h instance1.png)

Nếu kết quả như hình trên chứng tỏ đã mount thành công, bây giờ ta có thể sử dụng thư mục này để lưu trữ như 1 thư mục bình thường trên máy. Ta tạo thử 1 file `abc.txt` trong thư mục `/mnt/test` với nội dung bất kỳ, ví dụ: 

![](./img/abc.png)

<a name="detach"></a>
###6.2 Detach volume và gán vào 1 instance khác để kiểm tra

Sử dụng giao diện bằng cách đăng nhập vào horizon

Vào Project  -> Volumes -> Manage Attachments 

![](./img/manager attach.png)

Sau đó chọn `Detach Volume` để detach `volume0` ra khỏi máy ảo `test_cinder1`

![](./img/detack volume.png)

Chọn lại vào `Manage Attachments` của `volumev0`, và tiến hành attach volumev0 này vào máy ảo `test_cinder2`

Login vào máy ảo `test_cinder2` và kiểm tra lại xem `volume0` đã được attach vào ổ đĩa nào:

![](./img/instance2.png)

Như kết quả trong hình trên, `volumev0` đã được attach vào ổ đĩa `/dev/vdb` trong máy ảo `test_cinder2`.

Sau đó, tạo 1 thư mục mới `test2` trong thư muc `/mnt` để mount volumev0 vào và xem kết quả:

![](./img/mount to test2.png)

Sau khi mount thành công volumev0 vào thư mục `/mnt/test2`, tiên hành kiểm tra lại data trong volume0:

![](./img/ok instance 2.png)

Kết quả là data trong volumev0 đã được tạo trên instance 1 vẫn được giữ nguyên sau khi detach khỏi instance 1 và attach vào instance mới.
