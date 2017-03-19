#Phạm vi của Nova Project
Nova Project tập trung vào việc thực hiện các công việc nằm trong những nhiệm vụ chính của Project này một cách thật sự tốt. Tài liệu này nhằm làm rõ những nhiệm vụ chính của project này.

#Các nhiệm vụ
Các nhiệm vụ của Nova Project bắt đầu với nhiệm vụ sau:

 - Xây dựng các service và các library gắn liền với các service này để cung cấp các tài nguyên tính toán có  khả năng scale to lớn (massively scalable), theo nhu cầu, self-service access.

Nhiệm vụ chính thức của project này cũng bao gồm các ví dụ về các tài nguyên tính toán: bare-metal, máy ảo và các container.
#Về các tài nguyên tính toán
No và là mọi thứ về truy cập tới các tài nguyên tính toán (compute resources). Phần này nói về các loại tài nguyên tính toán mà Nova làm việc cùng.
 
 ##1 Các máy ảo
Nhiệm vụ cơ bản của nova tập trung vào việc cung cấp truy cập tới các máy ảo chạy trên hàng loạt các hypervisors khác nhau. Đa số người dùng sử dụng Nova chỉ sử dụng duy nhất một hypervisor để cung cấp truy cập tới các servers ảo, tuy nhiên chúng ta có thể triển khai Nova đồng thời với nhiều loại hypervisor khác nhau, trong khi đó vẫn có thể đồng thời cung cấp các dịch vụ liên quan tới containers và bare-metal servers. Điều đó có nghĩa là, trên 1 hệ thống OpenStack Cloud có thể sử dụng đồng thời nhiều loại hypervisor khác nhau để quản lý các máy ảo chứ không nhất thiết là chỉ được sử dụng một hypervisor trên toàn bộ hệ thống, đồng thời với việc đó chúng ta vẫn có thể sủ dụng các chứng năng liên quan tới container và bare-metal server.
###2 Container
Nova API không phải là một sự lựa chọn phù hợp cho phần lớn các nhu cầu sử dụng container. Magnum Project sẽ là một sự lựa chọn tốt cho các nhu cầu này, MagnumProject được xây dựng bên trên Nova.
Nova cho phép bạn sử dụng các container một cách tương tự với cách mà bạn sử dụng các nhu cầu về máy ảo. 
###3 Server thật (Bare-Metal Server)
Ironic project là ý tưởng tiên phong trong việc coi các máy chủ vật lý như là các máy ảo theo nhu cầu.
Nova driver có thể cho phép các người dùng trên cloud sử dụng các tài nguyên mà Ironic kiểm soát. 
###4 Sự tương đồng giưã các driver.
Một trong các mục tiêu của Nova API là cung cấp một mô tả có tính nhất quán để truy cập vào các tài nguyên tính toán theo nhu cầu. Chúng ta sẽ không nhắm tới mục tiêu cung cấp mọi tính năng của mọi hypervisor. Khi các thông tin chi tiết của các hypervisor bên dưới được thể hiện qua các API của chúng ta, chúng ta sẽ không đạt được mục tiêu này. mà chúng ta phải tạo ra một mô tả tốt hơn, sao cho sự tương thích với các hypervisor là càng ngày càng tốt. Đó là lý do mà chúng ta nhấn mạnh tới tầm quan trọng của Tempest trong các Commanline Interface của bên thứ ba.

Chìa khóa của sự tương đồng giua các driver là nếu một tính năng được hỗ trợ ở một driver nào đó, thì những đặc điểm của tính năng này phải là như nhau như khi họ dùng một driver khác cũng hỗ trợ tính năng này. Có thể có ngoại lệ, đó là sự khác biệt giữa các driver ở các đặc tính performance, nhưng các đặc điểm của lời gọi API là phải hoàn toàn giống nhau.

Từ đặc điểm đó, nên khi có một tính năng chỉ được thêm vào một driver nào đó, chúng ta phải đảm bảo cố gắng rằng các driver còn lại cũng có tính năng này với các đặc điểm của tính năng y hệ như driver kia.
Một điều quan trọng là các driver hỗ trợ đủ các tính năng, để đảm bảo các API cung cấp một cách nhất quán các tính năng đối với mọi driver.



> Written with [StackEdit](https://stackedit.io/).