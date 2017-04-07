# Data Consistency Model In Distributed System


Một trong những thành phần chính tạo ra một hệ thống thông tin, đó là các đối tượng dữ liệu. Các đối tượng dữ liệu trên hệ thống thông tin có thể tồn tại dưới những dạng khác nhau, như các cơ sở dữ liệu, các file hoặc thậm chí là các đối tượng nằm trong bộ nhớ RAM của một process. Tuy nhiên, cách thức sử dụng và quản lý các đối tượng dữ liệu trong một hệ thống phân tán có sự khác biệt với một hệ thống đơn tiến trình truyền thống. Khác với các hệ thống đơn tiến trình truyền thống thường chỉ lưu trữ và quản lý các đối tượng dữ liệu dưới dạng một thực thể duy nhất, trong một hệ thống phân tán, một đối tương dữ liệu có thể tồn tại dưới dạng nhiều thực thể giống nhau - được gọi là các bản sao (replica) của đối tượng. Việc tạo ra nhiều bản sao cho một đối tượng dữ liệu có rất nhiều ưu điểm, ví dụ như cho phép tăng hiệu năng tổng thể của hệ thống, do việc trên hệ thống việc có nhiều bản sao cho phép cung cấp nhiều tài nguyên hơn. Đồng thời, việc có nhiều bản sao của một đối tượng dữ liệu trên hệ thống cho phép hệ thống có khả năng chống lỗi tốt hơn, do khi sự cố xảy ra trên một thành phần nào đó, thì các bản sao khác của các đối tượng dữ liệu vẫn tồn tại ở trên các thành phần khác trong hệ thống, do đó hệ thống vẫn có khả năng phục hồi lại dữ liệu cho người dùng.

Như vậy, chúng ta có thể thấy cơ chế sao lưu (data replication) cung cấp rất nhiều lợi ích cho hệ thống phân tán. Tuy nhiên,đi cùng với các lợi ích mang lại, việc một đối tượng dữ liệu có nhiều bản sao trong hệ thống phân tán cũng đặt ra cho chúng ta một số vấn đề cần giải quyết. Đó là trong hệ thống phân tán, khi nhiều bản sao đồng thời phục vụ cho nhiều thực thể (các tiến trình và cácclient) khác nhau và bị các thực thể này thay đổi nội dung, làm sao để chúng ta xác định được tại một thời điểm xác định, đối tượng dữ liệu cũng như các bản sao của nó trên hệ thống sẽ có giá trị là gì ?

Ví dụ, trong hệ thống phân tán có thể xảy ra sự kiện cùng 1 lúc nhiều client yêu cầu thay đổi nội dung của một replica:

![concurrent_write_example1.png](./images/concurrent_write_example_1.png)

Hoặc khi cùng 1 lúc nhiều client yêu cầu thay đổi nội dung của nhiều replica:

![concurrent_write_example1.png](./images/concurrent_write_example_2.png)

Trong các trường hợp này, làm sao để chúng ta đảm bảo rằng đối tượng dữ liệu của chúng ta sẽ thay đổi giá trị theo một quy tắc xác định, và chúng ta có thể xác định được giá trị của đối tượng - cũng như giá trị của các replica của đối tượng dữ liệu đó sau các tương tác của các thực thể?

Để thực hiện được những điều này, chúng ta cần sử dụng một mô hình nhất quán trong hệ thống. Mô hình nhất quán (consistency model) trong một hệ thống phân tán là tập hợp các cơ chế, phương thức được sử dụng để duy trì tính nhất quán của hệ thống phân tán đó. Một hệ thống được gọi là có tính nhất quán, nếu như các thay đổi tác động tới dữ liệu/thông tin trong các bộ nhớ (bao gồm cả bộ nhớ trong - RAM và bộ nhớ ngoài - Hard Disk) của hệ thống phân tán đó được xử lý theo một quy tắc xác định. Mô hình nhất quán xác lập một thỏa thuận giữa hệ thống và những người lập trình ứng dụng Client, trong đó hệ thống đảm bảo cho chúng ta rằng, nếu Client tuân thủ các nguyên tắc xử lý dữ liệu trong thỏa thuận, thì dữ liệu lưu trữ trong hệ thống sẽ có tính nhất quán, và chúng ta có thể xác định trước được kết quả đầu ra của quá trình xử lý một dữ liệu/thông tin trong hệ thống.

Trong bài viết này, chúng ta sẽ phân tích các consistency model trong hệ thống phân tán, qua đó hiểu được điểm mạnh, điểm yếu của từng consistency model. Sau đó, tùy thuộc vào hệ thống phân tán mà chúng ta cần xây dựng là loại hệ thống gì, có những đặc điểm gì mà chúng ta có thể lựa chọn một consistency model phù hợp cho hệ thống phân tán của chúng ta. Nhưng trước hết, chúng ta sẽ làm rõ khái niệm trung tâm của bài viết, đó là: Tính consistency của một hệ thống phân tán là gì.


## 1. Consistency in Distributed System

Như đã giới thiệu ở phần đầu bài viết, tính nhất quán của một hệ thống là sự đảm bảo các thay đổi tác động tới một đối tượng dữ liệu trong hệ thống sẽ được xử lý theo một quy tắc xác định chứ không phải là các thay đổi thực hiện theo một phương thức ngẫu nhiên, và chúng ta có thể xác định trước được trạng thái của dữ liệu sau khi thực hiện một loạt các thay đổi này. Tính nhất quán của một hệ thống phân tán được đánh giá bởi nhiều tiêu chí đưa ra bởi nhiều mô hình đánh giá khác nhau, và mỗi một consistency model sẽ cho hệ thống của chúng ta một đặc điểm về tính consistency khác nhau. Một hệ thống phân tán sử dụng một consistency model xác định có thể đạt được một một số tiêu chí nào đó, nhưng có thể sẽ không thỏa mãn một số tiêu chí khác trong các tiêu chí đánh giá tính consistency của hệ thống. Do đó, tùy thuộc vào đặc điểm hệ thống của chúng ta là loại hệ thống gì, cần đạt được các tiêu chí nào trong số các tiêu chí, mà chúng ta lựa chọn mô hình consitency phù hợp với các tiêu chí đó.

Vì vậy, trước khi đi vào phân tích các consistency model, trước tiên chúng ta sẽ xem tính consistency của một hệ thống phân tán được đánh giá dựa trên các tiêu chí nào. Chúng ta sẽ sử dụng 2 phương pháp đánh giá bao gồm nhiều tiêu chí khác nhau để đánh giá tính consistency của một hệ thống phân tán, đó là 2 phương pháp **ACID** và **BASE**

## 1.1 Phương pháp ACID

## 1.2 Phương pháp BASE