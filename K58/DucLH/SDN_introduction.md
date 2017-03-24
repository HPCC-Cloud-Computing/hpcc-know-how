# Software-Defined Networking
## Giới thiệu
Trong hệ thống mạng truyền thống chúng ta đã biết, trong các thiệt bị mạng như router, switch đều tồn tại 2 cơ chế là control plane và data plane để thực hiện chức năng định tuyến và chuyển tiếp dữ liệu qua mạng. Cách thiết kế này đáp ứng được nhu cầu về mạng trong các giao đoạn trước đây của người dùng và các nhà cung cấp mạng. Tuy nhiên, càng ngày nhu cầu về một hệ thống mạng thông minh, tối ưu và có khả năng cải thiện hiệu năng tùy vào từng tình huống cụ thể đang được các nhà cung cấp dịch vụ mạng quan tâm mạnh mẽ. Và hệ thống mạng truyền thống không thể đáp ứng được yêu cầu này. Một trường hợp cụ thể, nếu một nhà quản trị mạng muốn thay đổi hành vi của mạng trên các bảng định tuyến hay thay đổi việc xử lý các gói tin trên các router hay switch, không còn cách nào khác, các nhà quản trị phải thực hiện việc thay đổi này trên từng thiết bị phần cứng và nếu như trên một hệ thống mạng rộng lớn thì việc này là không thể. Do đó, cần có một biện pháp để giải quyết vấn đề này.
Software-Defined Networking (SDN) tạm dịch qua là mạng được định nghĩa bằng phần mềm, kiến trúc này đã ra đời để giải quyết vấn đề còn tồn tại trong các hệ thống mạng truyền thống. SDN là một kiến trúc mạng linh động, cho phép các ngày quản trị mạng có thể **tự lập trình** để tạo ra một hệ thống mạng linh động, dễ dàng điều khiển, thay đổi và quản lý các hành vi thông qua một giao diện mở và trừu tượng hóa các chức năng của phần dưới. Cơ chế của SDN là tách biệt control plane và data plane, trong đó chức năng control có thể lập trình được.
## Các khái niệm
Tài liệu này sẽ sử dụng thuật ngữ sau:
- **Resource**: là thành phần vật lý hoặc ảo hóa có sẵn trong một hệ thống mạng. Tài nguyên có thể rất đơn giản (vd: 1 cổng port, 1 queue) hoặc phức tạp - tập hợp các tài nguyên (vd: một thiết bị mạng như switch, router).
- **Network device**: Là một thiết bị mạng thực hiện 1 hoặc nhiều hoạt động liên quan đến các packet như thao tác trên gói tin hay chuyển tiếp gói tin. Trong mô hình SDN này, network device có thể là physical hoặc là virtual. Nó cũng được xem là tài nguyên mạng.
- **Interface**: là một điểm thực hiện việc liên kết giữa 2 thực thể mạng. Khi các thực thể được đặt ở 2 vị trí khác nhau, interface thông thường là các giao thức mạng. Nếu các thực thể được đặt trong cùng một vị trí, interface là các API, inter-process communication (IPC), hoặc một giao thức mạng.
- **Application (App)**: Trong mô hình SDN, application là một phần của software mà sử dụng các dịch vụ bên dưới để thực hiện một chức năng. Hoạt động của ứng dụng có thể được tham số hóa, tức là sẽ được truyền một số tham số tại mỗi lần gọi. Ứng dụng là một thành phần độc lập của software và không cung cấp bất kỳ một interface đến các ứng dụng hoặc service khác.
- **Service**: Là một phần của software mà thực hiện một hoặc nhiều chức năng, và cung cấp các API đến các application hoặc các service khác.

## Đặc điểm của SDN
SDN có các đặc tính sau:
- **Logically centralized intelligence**: Trong kiến trúc SDN, control được tách biệt với forward và chúng giao tiếp với nhau thông qua interface (Interface được sử dụng trong SDN là OpenFlow). Bằng cách tập trung điều khiển, SDN cho phép tạo ra các quyết định tốt hơn thông qua một khung nhìn tổng thể về mạng, trái ngược với các hệ thống mạng trước đây, được xây dựng trên khung nhìn hệ thống đơn lẻ, có nghĩa là các node không biết được trạng thái tổng thể của mạng.
- **Programmability**: Có nghĩa là việc điều khiển mạng (chức năng control) có thể lập trình được bằng các phần mềm được cung cấp bởi các nhà cung cấp mạng. Khả năng có thể lập trình được cho phép cấu hình mạng một cách tự động, phù hợp với các thay đổi khi cần. Ngoài ra, Các phần mềm điều khiển này cũng có thể được viết bởi người dùng bởi vì việc sử dụng các Open API và open standand.
- **Abstaction**: Trong SDN, các ứng dụng và dịch vụ tầng trên sử dụng các dịch vụ của tầng dưới thông qua interface. Các dịch vụ tầng dưới sẽ được trựu tượng hóa thông qua các tầng trừu tượng (Abstraction layer)


## Kiến trúc của SDN
Hình dưới đây sẽ thể hiện rõ kiến trúc của SDN:
![](http://imgh.us/SDN_architecture.png)


Kiến trúc của SDN được chia ra thành các phần tách biệt và giao tiếp với nhau thông qua các open API. Cụ thể các phần như sau:
- **Forwarding plane** - Là tập hợp các tài nguyên chịu thực hiện nhiệm vụ xử lý các gói tin trong đường truyền dữ liệu dự trên thông tin định tuyến nhận được từ control plane. Các hành động của Forwarding plane bao gồm chuyển tiếp, xóa hoặc chỉnh sửa các gói tin.
- **Operational plane** - là các tài nguyên chịu trách nhiệm quản lý trạng thái hoạt động của các thiết bị mạng, ví dụ như một thiết bị đang hoạt động hay không hoạt động, số lượng các cổng port, và trạng thái của mỗi port. Operational plane là điểm kết thúc của các dịch vụ và ứng dụng thực hiện management-plane. Các thiết bị thường được liên kết tới Operational plane là port, memory,...
- ** Control plane** - Là các chức năng chịu trách nhiệm tạo ra các quyết định về cách các gói tin được chuyển tiếp trên một hay nhiều thiết bị mạng và truyền các quyết định này xuống các thiết bị mạng để thực hiện việc chuyển tiếp gói tin. Nhiệm vụ chính của control plane điều khiển việc chuyển tiếp dữ liệu.
- **Management plane** - Là tập hợp các chức năng chịu trách nhiệm cho việc quản lý, cấu hình và duy trì một hoặc nhiều thiết bị mạng hoặc một phần của thiết bị mạng. Management Plane tương tác hầu hết với Operation Plane
- **Application Plane** - tập hợp các application hoặc các service thực hiện chức năng lập trình các hành vi của network.

Tất cả các thành phần giao tiếp với nhau thông qua interface
Ngoài ra, sơ đồ trên còn đề cập đến 4 tầng trừu tượng hóa (abstraction layer):
- **Device and resource abstraction layer (DAL)** - Trừu tượng hóa các tài nguyên thực hiện operational plane và forwarding plane trong các thiết bị đối với control plane và management plane.
- **Control Abstraction Layer (CAL)** - trừu tượng hóa Control-Plane Southbound Interface và DAL đối với các ứng dụng và dịch vụ của Control plane
- **Management Abstraction Layer (MAL)** - trừu tượng hóa Management Southbound Interface và DAL đối với các application và dịch vụ của Management Plane
- **Network Service Abstraction Layer (NSAL)** - Cung cấp các dịch vụ trừu tượng hóa để cho các application và các dịch vụ khác sử dụng.
