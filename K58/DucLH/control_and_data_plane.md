# Control plane và Data plane
## 
Hiện nay, trong các hệ thống mạng truyền thống, các thiết bị mạng như router,switch,... đang tồn tại cả 2 cơ chế đó là : **control plane** và **data plane**. Hai khái niệm này khá quan trọng khi tìm hiểu về các hệ thống software-defined. Vì vậy, trước khi đọc các tài liệu về SDN hay OpenDayLight, bạn cần phải hiểu được định nghĩa về 2 khái niệm này.

Đối với hệ thống mạng truyền thống, trong mỗi thiết bị mạng như router hay switch. Control plane và Data plane sẽ thực hiện các nhiệm vụ sau:
- Control plane:
  - Đó là việc kết hợp các công nghệ, các tài nguyên vật lý, các giao thức hay các service để thực hiện việc tạo ra và điều khiển các thành phần như routing table, flow-table, port ... trong các thiết bị mạng Và các thành phần được tạo ra này sẽ thực hiện theo những gì mà control plane đã chỉ định.
  - Khi một gói tin đến router đầu tiên, router này sẽ thực hiện chức năng control plane, có nghĩa là nó sẽ ra quyết định xem những thành phần nào sẽ thực hiện chức năng của mình để chuyển tiếp gói tin đến đúng đích, ví dụ như port nào sẽ được dùng, routing table sẽ làm gì,...
- Data plane:
  - Đó là các thành phần trong thiết bị mạng mà control plane tạo ra và điều khiển, ví dụ như routing table, port,... Chúng thực hiện chức năng của mình theo sự hướng dẫn của control plane.
  - Quá trình ra quyết định của control plane khi một gói tin đến chính là ra quyết định xem thành phần nào trong dataplane sẽ thực hiện chức năng của mình trong chuyển tiếp gói tin cụ thể.

Tương tự với control plane và data plane trong SDN :
- Control plane là thành phần thực hiện chức năng tạo ra một hệ thống mạng ảo bao gồm các switch, các router, các routing table hay là các port trên router, switch,... Đồng thời, control plane cũng sẽ điều khiển hành vi của các thành phần này.
- Data plane chính là các switch, các router,... mà control plane tạo ra và điều khiển. Data plane sẽ thực hiện chức năng của nó để tạo ra một hệ thống mạng hoạt động.

Trong hệ thống mạng truyền thống, các nhà cung cấp mạng gặp rất nhiều khó khăn trong việc nâng cấp hiệu năng, tính bảo mật,... của các luồng dữ liệu, nguyên nhân là do phải thực hiện thay đổi control plane trên tất các các thiết bị mạng trong mạng; Đồng thời, việc quản lý và quyết định xử lý đối với từng luồng dữ liệu cần phải được thực hiện trên các thiết bị riêng biệt như router/switch,...

Công nghệ SDN - Software defined Networking ra đời nhằm giải quyết các vấn đề đang gặp phải trên của mạng truyền thống. Về cơ bản, SDN tách biệt hai cơ chế là control plane và data plane để tạo ra một khung nhìn tổng thể về mạng.

Điểm khác biệt của SDN và hệ thống mạng truyền thống là : Nếu như trong mạng truyền thống, một router không hoạt động, tức là control plane và data plane sẽ không hoạt động, có nghĩa là cả hệ thống mạng sẽ chết. Còn với SDN, bởi vì chỉ có 1 controller đều khiển cả hệ thống mạng, cho nên sau khi thực hiện chức năng tạo ra hệ thống mạng tức là tạo ra data plane, thì data plane sẽ hoạt động theo những gì mà controller đã chỉ định mà không quan tâm controller đó đang sống hay đã chết.
Chi tiết về SDN sẽ được giới thiệu ở tài liệu khác.

## Tài liệu tham khảo

- http://sdntutorials.com/difference-between-control-plane-and-data-plane/
- https://learningnetwork.cisco.com/thread/33735
