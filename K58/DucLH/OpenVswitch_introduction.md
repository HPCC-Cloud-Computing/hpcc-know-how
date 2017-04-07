# Open Vswitch Introduction (OVS)
## Giới thiệu về Open Vswicth
Bài viết này sẽ đề cập đến các khái niệm cơ bản của OVS, bao gồm các khái niệm cơ bản, liệt kê các tính năng của OVS, kiến trúc, cài đặt và xây dựng một mô hình mạng nho nhỏ sử dụng OVS.

Đầu tiên, chúng ta cần biết Open Vswitch là gì? Và vì sao lại có Open Vswitch?

Nhưng chúng ta đã biết, công nghệ ảo hóa tạo ra các virtual machine(VM) với yêu cầu là kết nối mạng giữa các VM và với mạng Internet. Trong hypervisor linux-based, yêu cầu này đã được giải quyết bởi linux brigde (một công nghệ tạo ra virtual switch),công nghệ này đảm bảo nhanh và rất đáng tin cậy. Tuy nhiên, linux brigde lại tỏ ra không phù hợp trong việc triển khai ảo hóa multi-server, nguyên nhân là môi trường multi-server có yêu cầu lượng băng thông lớn, các server có tính động cao,... Vì vậy, OVS đã được tạo ra để đáp ứng yêu cầu này.

OpenVswitch có thể được định nghĩa là một nền tảng tạo ra các virtual switch, mà hỗ trợ interface quản lý chuẩn và mở để có thể điều khiển và lập trình bởi người dùng.

Kiến trúc cơ bản của OVS được thể hiện ở hình sau:

![](http://www.yet.org/images/posts/ovs-archi.png)

Theo kiến trúc này, OVS có 3 thành phần chính sau đây:
- Ovs-vswitchd, một chương trình chạy nền thực hiện việc chuyển tiếp, ovs-vswitchd sẽ kết hợp với kennel-module để thực hiện việc chuyển tiếp dựa trên các flow. Ovs-vswicthd sử dụng giao thức Open Flow.
- Ovsdb-server, là một server database, được ovs-vswitchd sử dụng để lưu trữ và truy vấn các cấu hình của nó. Đồng thời, người dùng cũng có thể giao tiếp với ovsdb-server thông qua giao thức ovsdb management.
- control cluster, là công cụ hay các controller giao tiếp với ovsdb-server và ovs-vswitchd thông qua giao thức Open Flow và ovsdb management.

## Open Vswitch Lab
Sau đây, chúng ta sẽ thực hiện một mô hình nho nhỏ sử dụng OpenVswitch.Mục đích của Lab này là để làm quen với một số câu lệnh cơ bản với OVS và hiểu được cách tạo một virtual switch và kết nối switch đó đến với các máy ảo. Mô hình lab chúng ta sẽ xây dựng như sau:

![](https://img.youtube.com/vi/rYW7kQRyUvA/mqdefault.jpg)

Cụ thể,chúng ta sẽ thực hiện các bước sau:
- Sử dụng OpenVswitch để tạo ra một virtual switch. Giả sử switch này được đặt tên là br0.
- Kết nối br0 đến port (trong Lab này là eth0) đang kết nối ra internet.
- Tạo ra 2 Tap là vport1, vport2 (2 card mạng ảo) và kết nối đến br0
- Thay đổi cấu hình mạng cho 2 VM để kết nối đến 2 Tap vừa tạo ra
- Kiểm tra kết quả thu được.

Đầu tiên, Sử dụng câu lệnh sau để tạo virtual switch:
```
# ovs-vsctl add-br br0

```
Kiểm tra kết quả bằng câu lệnh:
```
# ovs-vsctl show

```
Chúng ta sẽ thu được kết quả như sau:

![](https://github.com/huynhducbk95/networking_document/blob/master/image/create_br0.jpg?raw=true)

Tiếp theo, chúng ta sẽ thực hiện kết nối br0 với card mạng đang kết nối ra internet:
```
# ovs-vsctl add-port br0 eth0

```
Sau khi thực hiện xong bước này, tạm thời máy tính của chúng ta sẽ bị mất mạng. Nguyên nhân là do ban đầu, eth0 được kết nối đến IP Stack(là các ip sẽ được cấp khi máy tính thực hiện kết nối đến internet) và máy tính sẽ kết nối được internet. Trong khi đó, br0 được tạo ra đang bị tách biệt và chưa được cấp địa chỉ ip, do đó, khi kết nối eth0 và br0 thì sẽ làm mất kết nối đến IP Stack.

Để giải quyết việc này, chúng ta thực hiện 2 bước. Đầu tiên, thực hiện câu lệnh sau để xóa địa chỉ ip hiện tại của eth0.
```
# ifconfig eth0 0

```
Tiếp theo, xin cấp phát địa chỉ ip cho br0:
```
# dhclient br0

```
Kết quả là chúng ta đã kết nối được internet. 

![](https://github.com/huynhducbk95/networking_document/blob/master/image/fix_ping_gg.jpg?raw=true)

Sau đó, chúng ta sẽ tạo ra 2 Tap (card mạng ảo) để thêm vào cho br0.
```
# ip tuntap add mode tap vport1
# ip tuntap add mode tap vport2
# ifconfig vport1 up
# ifconfig vport2 up

```
Và thực hiện câu lệnh sau để add 2 tap vừa tạo vào br0:
```
# ovs-vsctl add-port mybridge vport1 -- add-port mybridge vport2

```
Kết quả chúng ta có được như sau:

![](https://github.com/huynhducbk95/networking_document/blob/master/image/check_add_ports.jpg?raw=true)

Sau khi add 2 Tap đến br0. Bước cuối cùng là chúng ta tạo ra 2 VM, với cấu hình mạng là chế độ bridge và card mạng là vport1 và vport2 trên từng VM.

![](https://github.com/huynhducbk95/networking_document/blob/master/image/configure_vm1.jpg?raw=true)

Như vậy là chúng ta đã tạo ra được một virtual switch br0 có chứa 1 port kết nối đến eth0, 1 port là vport1 kết nối đến VM1 và 1 port là vport2 kết nối đến VM2. Và 2 VM này đều được ping đến nhau và kết nối ra internet thông qua eth0. Chúng ta có thể kiểm tra ping ra internet trên vm1.

![](https://github.com/huynhducbk95/networking_document/blob/master/image/check_ping_vm1_gg.jpg?raw=true)

Chúng ta cũng có thể sử dụng câu lệnh sau để xem thông tin các port và các VM đang kết nối đến br0.
```
# ovs-appctl fdb/show br0

```
Kết quả hiển thị như sau:

![](https://github.com/huynhducbk95/networking_document/blob/master/image/appctl.jpg?raw=true)
## Kết luận
Trên đây là như kiến thức cơ bản nhất về Open Vswitch. Bạn có thể tìm thấy tài liệu chi tiết về Open Vswitch trên trang web [openvswitch.org](http://openvswitch.org)
## Tài liệu tham khảo

- https://www.youtube.com/watch?v=rYW7kQRyUvA
- http://docs.openvswitch.org/en/latest/
- https://sreeninet.wordpress.com/2014/01/02/openvswitch-and-ovsdb/
