#OpenStack Nova Architect
Nova là tập hợp của một loạt các process (service), mỗi một process thực thi một chức năng. Giao diện người dùng thông qua REST-API, trong khi đó bên trong Nova Project các tiến trình giao tiếp với nhau thông qua một cơ chế RPC message.
API server xử lý các REST request bằng cách thực thi các hành động như đọc/ghi dữ liệu vào database, gửi RPC message tới các nova service khác
```text
AMQP is the messaging technology chosen by the OpenStack cloud. The AMQP broker, either RabbitMQ or Qpid, sits between any two Nova components and allows them to communicate in a loosely coupled fashion. 
```
việc truyền và nhận các tin nhắn RPC được thực hiện thông qua thư viện oslo.messaging, một lớp abstract phía bên trên của các message queues. Hầu hết các thành phần chính của nova đều có khả năng chạy trên nhiều server cùng 1 lúc, và sẽ có một manager service chuyên thu nhận các RPC message từ các service khác. Một trong các thành phần chính có ngoại lệ là nova-compute, thành phần này chỉ có một tiến trình duy nhất chạy trên hypervisor (hệ thống ảo hóa) mà nó quản lý (trừ khi chúng ta sử dụng VMware hoặc ironic driver). Bên cạnh đó, manager service cũng thường có các task định kỳ. Thông tin chi tiết về hệ thống RPC, đọc tiếp tại [AMQP and Nova](http://docs.openstack.org/developer/nova/rpc.html)

Nova cũng sử dụng cơ sở dữ liệu được chia sẻ  (một cách logic)  giữa tất cả các thành phần. Tuy nhiên, để phục vụ yêu cầu nâng cấp, cơ sở dữ liệu được truy cập thông qua một lớp object để đảm bảo rằng sau khi một hệ thống được nâng cấp phần điều khiển, thì thành phần điều khiển đã được nâng cấp này vẫn có thể giao tiếp với một nova-compute chạy ở phiên bản trước đó. Để thực hiện được điều này, nova-compute trao đổi các yêu cầu trao đổi dữ liệu với  DB  thông qua các thông điệp RPC tới  một  service đóng vai trò trung tâm quản lý được gọi là nova-conductor
Để có thể mở rộng sự triển khai nova theo chiều ngang ( horizontally expand) , chúng ta sử dụng một kiến trúc triển khai rời rạc, được gọi là Cells. [Thông tin thêm về Cells](http://docs.openstack.org/developer/novthông đia/cells.html)

##Các thành phần trong nova
Sơ đồ triển khai nova với neutron-networking như sau:
![Nova components](http://docs.openstack.org/developer/nova/_images/architecture.svg)
Vai trò của các thành phần trong sơ đồ trên:

 - DB: Cơ sở dữ liệu SQL dùng để lưu trữ dữ liệu
 - API: thành phần tiếp nhận HTTP request từ bên ngoài và từ các service khác, thực hiện chuyển đổi các câu lệnh và giao tiếp với các thành phần khác thông qua oslo.messaging queue hoặc HTTP
 - Scheduler: Thành phần quyết định host nào sẽ chứa instance mới tạo ra.
 - Network (Neutron): quản lý ips, các bridge ảo, các mạng ảo
 - Compute: Quản lý việc giao tiếp với hypervisor và các máy ảo.
 - Conductor: Xử lý các yêu cầu liên quan tới việc điều chỉnh (khởi tạo/ thay đổi kích thước máy ảo), hoạt động giống như một database proxy. Giải thích khái niệm proxy: **proxy** có nghĩa là ủy nhiệm, ủy quyền. Tức là một thành phần nào sử dụng conductor làm database proxy sẽ giao tiếp với nova-conductor và yêu cầu nó trao đổi dữ liệu với database chứ không trực tiếp giao tiếp với database nữa

Trong khi mọi service đều được thiết kế để có thể scale theo chiều ngang, thì bạn sẽ có nhiều computes service hơn các service còn lại.

> Written with [StackEdit](https://stackedit.io/).