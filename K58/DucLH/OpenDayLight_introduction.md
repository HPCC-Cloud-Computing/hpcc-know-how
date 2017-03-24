#OpenDayLigth Introduction
## Giới thiệu
OpenDayLight(ODL) là một platform mã nguồn mở cho Software-Defined Networking(SDN) mà cung cấp một giao thức mở(open protocol) để hỗ trợ trung tâp hóa, khả năng tự lập trình điều khiển và quản lý các thiết bị mạng. Giống như các SDN controller khác, OpenDayLight hỗ trợ OpenFlow, cung cấp một interface cho phép kết nối đến các thiết bị mạng một cách nhanh chóng để thực hiện tối ưu hiệu năng network.

Quá trình cài đặt OpenDayLight không giống như cài đặt một phần mềm thông thường, mà là việc kết hợp cài đặt các thành phần khau nhau, các tính năng phù hợp tùy theo những gì người dùng cần. Vì vậy, trước khi đi vào cài đặt OpenDayLight, chúng ta sẽ tìm hiểu bước tranh toàn cảnh về ODL để hiểu được các chức năng và cách thiết lập một môi trường network.
## Các tính năng của OpenDayLight
Điểm khác biệt lơn nhất để so sánh ODL với các SDN controller khác được thể hiện ở các điểm sau:
- Một kiến trúc các microservice, ở đây một microservice có thể là một protocol hay là một service mà một người dùng muốn cài đặt lên ODL controller để thực hiện các yêu cầu của họ. ví dụ như: một Plugin cung cấp việc kết nối các thiết bị thông qua giao thức OpenFlow hay BGP; hay là một service phụ vụ authentication, authorization, and accounting;....
- Hỗ trợ một phạm vi rộng lớn và có khả năng mở rộng thêm các giao thức không chỉ là OpenFlow, mà còn là SNMP, NETCONF, OVSDB, BGP, PCEP, LISP,...
- Hỗ trợ cho việc phát triển các tính năng mới.

OpenDayLight thực hiện các chức năng sau:
- Trung tâm hóa việc lập trình điều khiển các thiết bị mạng vật lý và mạng ảo trong toàn bộ mạng.
- Điều khiển các thiết bị với các giao thức chuẩn, mở.
- Cung cấp các abstraction đối với các tầng bên dưới để các deverloper có thể tạo ra các application mới.


## Các tool chính trong OpenDayLight
1. Karaf

Là công cụ cung cấp các tính năng mà bạn muốn thực hiện và được triển khai trên OpenDayLight của mình.

Thông thường, OpenDayLight sau khi cài đặt sẽ không có một tính năng nào. Điều này có nghĩa là, sau khi cài đặt xong OpenDayLight, bạn sẽ cài đặt các thành phần bạn muốn sử dụng Karaf console để thêm tính năng cho network. Danh sách các tính năng và tác dụng của chúng được liêt kê chi tiết tại địa chỉ [Karaf feature](http://docs.opendaylight.org/en/stable-boron/getting-started-guide/karaf_features.html)

2. DLUX

Là một giao diện web mà ODL cung cấp để giúp bạn quản lý network của mình. Tính năng này cần được cài đặt sử dụng karaf với tên là **odl-dlux-core**.

3. Network embedded Experience (NeXt)

comming soon

## Cài đặt OpenDayLight
Trong tài liệu này, OpenDayLight được cài đặt trên ubuntu 14.04.

Các gói yêu cầu trước khi cài đặt OpenDayLight là: Java Runtime Environment (JRE) với phiên bản 1.8 trở lên

Sau khi cài đặt xong JRE, chúng ta sẽ bắt đầu cài đặt ODL:
- Download phiên bản mới nhất của ODL tại trang chủ opendaylight.org:

```
$ sudo wget https://nexus.opendaylight.org/content/repositories/opendaylight.release/org/opendaylight/integration/distribution-karaf/0.5.2-Boron-SR2/distribution-karaf-0.5.2-Boron-SR2.tar.gz
```
- untaz file vừa tải về:

```
sudo tar -xzvf distribution-karaf-0.5.2-Boron-SR2.tar.gz

```
- cd vào thư mục distribution-karaf-0.5.2-Boron-SR2 và chạy câu lệnh sau:

```
./bin/karaf

```
Sau khi chạy câu lệnh trên, OpenDayLight sẽ được cài đặt, đồng thời cũng hiển thị giao diện karaf console.

Như vậy là ODL đã được cài đặt. Tuy nhiên, ODL sau khi cài xong sẽ chưa có tính năng nào. Vì vậy, để cài thêm tính năng sẽ sử dụng karaf console.
Để cài đặt một tính năng là feature1, sử dụng câu lệnh:
```
feature:install <feature1>

```
Để cài nhiều tính năng cùng một lúc:
```
feature:install <feature1> <feature2> ... <featureN-name>

```
Để liệt kê các tính năng đã cài đặt:
```
feature:list -i

```
