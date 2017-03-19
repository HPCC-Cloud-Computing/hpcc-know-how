#OpenStack Glance
#Mục lục
<h4><a href="#overview">1. Tổng quan về OpenStack Glance</a></h4>
<ul>
<li><a href="#glossary">1.1. Một số thuật ngữ</a></li>
<li><a href="#intro">1.2. Giới thiệu về Glance</a></li>
<li><a href="#component">1.3. Các thành phần của Glance</a></li>
<li><a href="#arch">1.4. Kiến trúc của Glance</a></li>
<li><a href="#formats">1.5. Các định dạng lưu trữ image của Glance</a></li>
<li><a href="#flow">1.6. Luồng trạng thái của Glance</a></li>
<li><a href="#img">1.7. Image và Instance</a></li>
</ul>
<h4><a href="#install">2. Cài đặt dịch vụ Glance</a></h4>
<ul>
<li><a href="#prerequisites">2.1. Tạo database, dịch vụ xác thực và API endpoints cho Glance</a></li>
<li><a href="#ins_conf">2.2. Cài đặt và cấu hình các thành phần của Glance</a></li>
<li><a href="#verify">2.3. Kiểm chứng lại việc cài đặt Glance</a></li>
</ul>
---

<h2><a name="overview">1. Tổng quan về OpenStack Glance</a></h2>
<h3><a name="glossary">1.1. Một số thuật ngữ</a></h3>
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
<h3><a name="intro">1.2 Giới thiệu về Glance</a></h3>
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
<img src="./img/overview-glance.jpg"/>

<h3><a name="component">1.3 Các thành phần của Glance</a></h3>

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
<li><b>VMWare: </b>
<div>ESX/ESXi hoặc vCenter Server.</div>
</li>
<li><b>Amazon S3</b>
<div>Dịch vụ của Amazon</div>
</li>
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
<h3><a name="arch">1.4 Kiến trúc của Glance</a></h3>
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
<h3><a name="formats">1.5 Các định dạng lưu trữ image của Glance</a></h3>
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

<h3><a name="flow">1.6 Luồng trạng thái của Glance</a></h3>
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

<h3><a name="img">1.7 Image and Instance</a></h3>
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
<h2><a name="install">2. Cài đặt dịch vụ Glance</a></h2>
<b>Lưu ý: </b>Password thống nhất cho tất cả các dịch vụ là Welcome123
<h3><a name="prerequisites">2.1 Tạo database, dịch vụ xác thực và API endpoints cho Glance</a></h3>
<a name="2.1.1."> </a> 
#### 2.1.1 Tạo database cho `glance`
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

<a name="2.1.2."> </a> 
#### 2.1.2. Cấu hình xác thực cho dịch vụ `glance`
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

<a name="2.1.3."> </a> 
#### 2.1.3. Tạo các endpoints
- Tạo các endpoint cho dịch vụ `glance`
	```sh
	openstack endpoint create --region RegionOne image public http://controller:9292

	openstack endpoint create --region RegionOne image internal http://controller:9292

	openstack endpoint create --region RegionOne image admin http://controller:9292
	```
	
<h3><a name="ins_conf">2.2 Cài đặt và cấu hình các thành phần của Glance</a></h3>
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
<h3><a name="verify">2.3 Kiểm chứng lại việc cài đặt Glance</a></h3>
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
