# Data Consistency Model In Distributed System

Một trong những thành phần chính tạo ra một hệ thống thông tin, đó là các đối tượng dữ liệu. Các đối tượng dữ liệu trên hệ thống thông tin có thể tồn tại dưới những dạng khác nhau, như các cơ sở dữ liệu, các file hoặc thậm chí là các đối tượng nằm trong bộ nhớ RAM của một process. Các đối tượng dữ liệu này đóng vai trò là các thực thể lưu trữ các thông tin mà hệ thống lưu trữ, xử lý và trao đổi với các hệ thống khác. Do đó, một trong những vấn đề đầu tiên mà nhà thiết kế hệ thống cần quan tâm khi xây dựng một hệ thống thông tin, đó là vấn đề tổ chức quản lý, lưu trữ và xử lý các đối tượng thông tin này một cách hiệu quả.

Chúng ta có thể chia các hệ thống thông tin hiện tại ra làm 2 loại: Hệ thống thông tin tập trung truyền thống và hệ thống phân tán. Một trong những đặc điểm khác biệt chính giữa 2 loại hệ thống này, đó là cách thức quản lý, lưu trữ và xử lý các đối tượng dữ liệu trong một hệ thống phân tán có sự khác biệt với một hệ thống tập trung. Các đối tượng dữ liệu trong hệ thống tập trung được quản lý và lưu trữ dưới dạng một thực thể duy nhất. Còn trong một hệ thống phân tán, một đối tượng dữ liệu có thể tồn tại dưới dạng nhiều thực thể giống nhau phân tán trên các thành phần/máy tính khác nhau trong hệ thống - chúng được gọi là các bản sao (replica) của đối tượng dữ liệu.

Việc một đối tượng dữ liệu có nhiều bản sao mang lại rất nhiều lợi ích cho hệ thống phân tán, như: Cho phép tăng hiệu năng tổng thể, cũng như khả năng scalability của hệ thống, do việc trên hệ thống việc có nhiều bản sao cho phép cung cấp cho hệ thống nhiều tài nguyên hơn. Đồng thời, việc có nhiều bản sao của một đối tượng dữ liệu cho phép hệ thống có khả năng chống lỗi tốt hơn, do khi sự cố xảy ra trên một thành phần nào đó, thì các bản sao khác của các đối tượng dữ liệu vẫn tồn tại ở trên các thành phần khác trong hệ thống, do đó nội dung trên các đối tượng này vẫn được đảm bảo an toàn.

Tuy nhiên, đi cùng với các lợi ích mang lại, việc một đối tượng dữ liệu được phân tán thành nhiều replica trong hệ thống phân tán cũng đặt ra cho chúng ta một số vấn đề cần giải quyết. Các vấn đề đó là:

- Làm sao để chúng ta theo dõi được trạng thái của các bản sao, phát hiện ra các bản sao có vấn đề (như hỏng dữ liệu, mất kết nối...)?
- Trong hệ thống phân tán, khi nhiều bản sao đồng thời phục vụ cho nhiều thực thể (các tiến trình và các client) khác nhau và bị các thực thể này thay đổi nội dung, thì làm sao để chúng ta biết được, tại một thời điểm xác đinh, các replica của một đối tượng trong hệ thống có giá trị là gì ?
- Chúng ta có thể cho phép có sự sai khác về giá trị giữa các replica của cùng một đối tượng dữ liệu với nhau hay không? (*)
- Nếu chúng ta cho phép (*) xảy ra, thì làm sao để chúng ta phát hiện ra sự sai khác, và tính toán được mức độ sai khác giá trị giữa các replica này ? Và làm sao để chúng ta có thể giới hạn sự sai khác giữa các replica của một đối tượng ở một mức độ chấp nhận được ?

Ví dụ, trong hệ thống phân tán có thể xảy ra sự kiện cùng 1 lúc nhiều client yêu cầu thay đổi nội dung của một replica:

![concurrent_write_example1.png](./images/concurrent_write_example_1.png)

Hoặc khi cùng 1 lúc nhiều client yêu cầu thay đổi nội dung của nhiều replica:

![concurrent_write_example1.png](./images/concurrent_write_example_2.png)

Trong các trường hợp này, làm sao để chúng ta đảm bảo rằng đối tượng dữ liệu của chúng ta sẽ thay đổi giá trị theo một quy tắc xác định. Và làm sao để chúng ta có thể xác định được các đối tượng - cũng như các replica của đối tượng dữ liệu đó sau các tương tác của các thực thể có giá trị là gì?

Để giải quyết các vấn đề đã nêu ra, chúng ta cần sử dụng một mô hình nhất quán trong hệ thống. Mô hình nhất quán (consistency model) trong một hệ thống phân tán là tập hợp các cơ chế, phương thức được sử dụng để duy trì tính nhất quán của hệ thống phân tán đó. Một hệ thống được gọi là có tính nhất quán, nếu như các thay đổi tác động tới dữ liệu/thông tin trong các bộ nhớ (bao gồm cả bộ nhớ trong - RAM và bộ nhớ ngoài - Hard Disk) của hệ thống phân tán đó được xử lý theo một quy tắc xác định. Mô hình nhất quán xác lập một thỏa thuận giữa hệ thống và những người lập trình ứng dụng Client, trong đó hệ thống đảm bảo cho chúng ta rằng, nếu Client tuân thủ các nguyên tắc xử lý dữ liệu trong thỏa thuận, thì dữ liệu lưu trữ trong hệ thống sẽ có tính nhất quán, và chúng ta có thể xác định trước được kết quả đầu ra của quá trình xử lý một dữ liệu/thông tin trong hệ thống.

Qua các phân tích trên, chúng ta có thể hiểu được rằng, tính consistency là một tính chất quan trọng mà chúng ta cần quan tâm tới khi phân tích, đánh giá và thiết kế một hệ thống phân tán. Hơn thế, khi đánh giá một hệ thống phân tán dưới góc nhìn tổng quát, chúng ta sẽ nhận ra rằng, trong các hệ thống phân tán, ba tính chất  **Consistency**, **Availability** và **Partition Tolerance**  sẽ có sự tác động, ảnh hưởng lẫn nhau. Eric Brewer đã phát biểu định lý CAP về mối liên hệ giữa các tính chất **Consistency**, **Availability** và **Partition Tolerance**. Trong định lý CAP. 3 tính chất **Consistency**, **Availability** và **Partition Tolerance** được định nghĩa như sau:

- **C**onsistency: Tính chất Consistency trong định lý CAP đảm bảo các đối tượng dữ liệu trong hệ thống sẽ được nhìn nhận và xử lý như là chỉ có một bản sao của đối tượng dữ liệu đó tồn tại trên hệ thống. Điều này tương đương với việc, tất cả mọi thay đổi mà các Client bên ngoài tác động lên các bản sao của một đối tượng dữ liệu trong hệ thống sẽ được xử lý lần lượt, theo thứ tự (ordered), chứ không thể đồng thời xảy ra.
- **A**vailable: Một hệ thống phân tán có tính Available khi một node không lỗi (non-failing node) trong hệ thống luôn đáp ứng tất cả mọi request mà nó nhận được trong mọi trường hợp, kể cả trong trường hợp kết nối giữa node xử lý request với một số node khác trong hệ thống (network failure).
- **P**artition Tolerance: Tính chống lỗi của hệ thống khi bị phân mảnh. Một hệ thống phân tán bị rơi vào trạng thái phân vùng (Partitioned) nếu như chúng ta có thể chia các thực thể trong hệ thống thành nhiều vùng (partition), sao cho hai thực thể nằm trong cùng một vùng có thể kết nối với nhau, nhưng hai thực thể nằm ở hai vùng khác nhau không kết nối được với nhau. Một hệ thống được gọi là có tính chất Partition Tolerance, nếu như hệ thống đó luôn hoạt động và xử lý thông tin chính xác trong mọi trường hợp, kể cả khi hệ thống rơi vào trạng thái phân vùng - Partitioned State [3]

Nội dung của định lý CAP là:

**Một hệ thống phân tán  chỉ có thể có được tối đa 2 tính chất trong 3 tính chất: Consistency, Availability và Partition Tolerance** [1]

Một lưu ý ở đây, là khái niệm Consistency trong định lý CAP được hiểu là khái niệm "strong consistency". Strong consistency là khái niệm cơ bản để định nghĩa tính chất consistency của hệ thống thông tin. Tuy nhiên, do theo định lý CAP, chúng ta không thể có được cả 3 tính chất Consistency (strong consistency), Availability và Partition Tolerance trong một hệ thống phân tán, do đó các consistency model được phát triến sau này đã định nghĩa thêm một khái niệm consistency có mức độ consistency yếu hơn, nhưng phù hợp hơn và dễ đạt được hơn với các hệ thống phân tán, được gọi là **weak consistency**. Trong các phần sau của bài viết, các bạn sẽ thấy rằng, có một số consistency model sẽ cho phép hệ thống đạt được 2 tính chất Available và Partition Tolerance, trong khi vẫn đảm bảo hệ thống có được tính chất "weak consistency".

Trong bài viết này, chúng ta sẽ phân tích các consistency model trong hệ thống phân tán, qua đó hiểu được điểm mạnh, điểm yếu của từng consistency model. Sau đó, tùy thuộc vào hệ thống phân tán mà chúng ta cần xây dựng là loại hệ thống gì, có những đặc điểm gì mà chúng ta có thể lựa chọn một consistency model phù hợp cho hệ thống phân tán của chúng ta. Nhưng trước hết, chúng ta sẽ làm rõ khái niệm trung tâm của bài viết, đó là: Tính consistency của một hệ thống phân tán là gì.

## 1. Consistency in Distributed System

Như đã giới thiệu ở phần đầu bài viết, khi một hệ thống phân tán sử dụng một mô hình nhất quán dữ liệu, thì các thay đổi tác động tới một đối tượng dữ liệu trong hệ thống sẽ được xử lý theo một quy tắc xác định, chứ không phải là các thay đổi thực hiện theo một phương thức ngẫu nhiên, và chúng ta có thể xác định trước được trạng thái của các dữ liệu trong hệ thống sẽ như thế nào sau khi các thao tác thay đổi diễn ra.

Với sự phát triển của các hệ thống phân tán, nhiều consistency model đã được xây dựng và phát triển. Mỗi một consistency model đều có những đặc điểm riêng, cũng như có các điểm mạnh, điểm yếu khác nhau. Để đánh giá sự hiệu quả của một consistency model, cũng như để so sánh giữa các consistency model với nhau, thì chúng ta cần phải đánh giá được một hệ thống phân tán sau khi áp dụng một consistency model xác định sẽ có tính nhất quán như thế nào. Tính nhất quán của một hệ thống phân tán có thể được đánh giá, đo đạc và định nghĩa thông qua nhiều tiêu chí đưa ra bởi nhiều mô hình đánh giá khác nhau. Và khi đã sử dụng một mô hình đánh giá cụ thể để đánh giá tính nhất quán của hệ thống phân tán, chúng ta sẽ thấy rằng, một hệ thống phân tán sử dụng một consistency model xác định có thể đạt được một một số tiêu chí nào đó, nhưng có thể sẽ không thỏa mãn một số tiêu chí khác trong các tiêu chí đánh giá tính consistency của hệ thống. Khi đã biết được điểm mạnh, điểm yếu của từng consistency model rồi, thì tùy thuộc vào đặc điểm hệ thống phân tán mà chúng ta đang xây dựng là loại hệ thống gì, cần đạt được các tiêu chí nào trong số các tiêu chí đánh giá tính nhất quán, mà chúng ta lựa chọn mô hình consitency phù hợp với các tiêu chí đó để áp dụng vào hệ thống của chúng ta.

Chúng ta sẽ xem tính consistency của một hệ thống phân tán được đánh giá dựa trên các tiêu chí nào, bằng các mô hình đánh giá nào. Trước tiên, chúng ta sẽ tìm hiểu về 2 mô hình đánh giá tính nhất quán tổng quát nhất, đó là 2 mô hình **ACID** và **BASE**

### 1.1 Mô hình ACID

Phương pháp ACID đánh giá tính nhất quán của một hệ thống phân tán thông qua việc phân tích xem các giao dịch được thực hiện trong hệ thống phân tán đó được các tiêu chí nào trong 4 tiêu chí sau: **A**tomicity, **C**onsistency, **I**solation, **D**urability

**Chú thích:** Trong phương pháp ACID thì khái niệm **giao dịch** chính là chỉ các thao tác thay đổi các đối tượng dữ liệu mà các thực thể tác động lên hệ thống như: Tạo đối tượng dữ liệu mới, cập nhật nội dung đối tượng, xóa đối tượng,...

**Atomicity** - Tính toàn vẹn / tính nguyên tố: Tiêu chí này đề cập tới sự toàn vẹn của một giao dịch. Một giao dịch được coi là có tính toàn vẹn khi nó thỏa mãn cả 2 điều kiện sau trong mọi điều kiện, kể cả khi hệ thống gặp sự cố:

- Nếu một giao dịch được gọi là thành công, tất cả các thay đổi mà nó tạo ra được thực hiện thành công trên hệ thống.
- Nếu hệ thống thực hiện thất bại một trong các thay đổi trong giao dịch, thì toàn bộ giao dịch đó bị coi là thất bại. Và khi một giao dịch thất bại, không có thay đổi nào mà nó tạo ra được thực hiện trên hệ thống.

Ví dụ, xét một giao dịch: thêm dữ liệu vào đối tượng, được tạo ra trên hệ thống. Giao dịch này thực hiện 2 thao tác thay đổi sau: Thêm **X** vào file **X1** và thêm **Y** vào file **Y2**. Trong trường hợp giao dịch thành công, thì hệ thống phải thực hiện thành công cả hai công việc là thêm **X** vào file **X1** và thêm **Y** vào file **Y2**. Trong trường hợp hệ thống thêm thành công **X** vào file **X1**, nhưng thất bại trong việc thêm **Y** vào **Y2**, thì giao dịch này thất bại. Khi giao dịch thất bại, nếu hệ thống có tính nguyên tố, thì hoặc thao tác thêm **X** vào **X1** không được thực hiện, hoặc nếu thao tác này đã được thực hiện, thì hệ thống phải thực hiện việc **revert**: đưa file **X1** quay trở lại trạng thái trước khi **X** được thêm vào **X1**

**Consistency** - Tính nhất quán của một giao dịch. Một giao dịch có tính nhất quán, nếu như sau khi hoàn tất giao dịch, dữ liệu trên hệ thống ở một trạng thái hợp lệ. Nếu như trong quá trình thực hiện giao dịch xảy ra lỗi, thì hệ thống phải tự động **roll back** về trạng thái trước khi thực hiện giao dịch.Tính chất này đảm bảo cho hệ thống luôn ở trạng thái hợp lệ cho dù giao dịch có được thực hiện thành công hay không.

**Isolation** - Tính độc lập, tách biệt của giao dịch. Tính chất này quy định khi trong hệ thống cùng một lúc nhận được 2 giao dịch cùng một lúc muốn truy cập vào cùng 1 đối tượng dữ liệu trong hệ thống, thì các giao dịch phải được thực hiện như là chỉ có một giao dịch được thực hiện trong hệ thống tại thời điểm đó. Tính chất này đảm bảo rằng, việc thực hiện các thay đổi do giao dịch này tạo ra không ảnh hưởng tới việc thực hiện các thay đổi do giao dịch kia tạo ra, nếu 2 giao dịch trên được thực hiện đồng thời trên cùng 1 đối tượng dữ liệu. Tiêu chí độc lập có nhiều mức độ khác nhau bằng cách sử dụng các phương thức đồng bộ hóa khác nhau.

**Durability** - Tính ổn định, bền vững của giao dịch. Tính chất này quy định rằng, khi một giao dịch đã được thực hiện thành công, hệ thống phải đảm bảo trạng thái mới của đối tượng dữ liệu được lưu lại an toàn trong hệ thống, ngay cả trong trường hợp sau khi thực hiện giao dịch, hệ thống xảy ra sự cố như server bị lỗi phần mềm, mất điện, hỏng ổ cứng,...

Phương thức đánh giá ACID được sử dụng nhiều trong việc thiết kế và phát triển các hệ quản trị cơ sở dữ liệu, do đặc điểm của phương thức này đề cao tính consistency của các đối tượng dữ liệu - một trong các yếu tố quan trọng nhất trong cơ sở dữ liệu. Tuy nhiên, cùng với việc đề cao quá mức tính consistency của dữ liệu, phương pháp ACID lại xem nhẹ tính Scalability cũng như Performance trong các hệ thống phân tán, mà đây lại là những tính chất rất cần thiết đối với các hệ thống phân tán. Vì lý do này, phương pháp ACID không đánh giá được toàn vẹn các tính chất của một hệ thống phân tán.

Chính vì vậy, để đánh giá tính consistency của hệ thống phân tán,một phương pháp mới đã được sử dụng, cho phép đánh giá cân bằng giữa 2 tính chất consistency và scalability của hệ thống phân tán, đó là phương pháp BASE (Basically Available, Soft-State, Eventually Consistent)

### 1.2 Mô hình BASE

Mô hình BASE(**B**asically **A**vailable, **S**oft-State, **E**ventually Consistent) điều chỉnh lại 4 tiêu chí trong ACID để có thể đánh giá toàn diện hơn về tính consistency trong các hệ thống phân tán. Phương pháp base đánh giá tính consistency của một hệ thống phân tán dựa trên các tiêu chí sau:

- **Basically Available**: Tính chất Basically Available quy định hệ thống cần phải phản hồi mọi request từ Client gửi tới, tuy nhiên, kết quả mà phản hồi của hệ thống trả về có thể là một thông báo lỗi, hoặc các node khác nhau trên hệ thống được phép trả về các kết quả không giống nhau. Tính chất  **Basically Available** cho phép các replica của một đối tượng dữ liệu trong hệ thống tại một thời điểm xác định có thể có sự sai khác với nhau ở một mức độ chấp nhận được.

- **Soft State**: Tính chất này quy định trạng thái của hệ thống có tính "mềm dẻo", tức là sau một khoảng thời gian **t**, hệ thống có thể thay đổi trạng thái (hệ thống được gọi là thay đổi trạng thái khi một số đối tượng dữ liệu trong hệ thống thay đổi nội dung), cho dù trong khoảng thời gian **t** đó không có request nào từ phía Client được gửi tới hệ thống. Một hệ thống phân tán có tính chất Soft State, nếu như theo thời gian các replica của một đối tượng dữ liệu trong hệ thống đó sẽ cập nhật với nhau để dữ liệu trong các replica này được đồng bộ hóa với nhau (tức là dữ liệu trong các replica trở lại giống nhau), hoặc làm giảm thiểu tối đa sự khác biệt giữa các replica. Hành động cập nhật lẫn nhau giữa các replica của một đối tượng dữ liệu dẫn tới tính chất **Eventualy Consistent** của hệ thống.

- **Eventualy Consistent**: Một hệ thống phân tán có tính chất **Eventualy Consistent**, nếu như sau khi các Client sử dụng hệ thống ngừng gửi các request yêu cầu thay đổi nội dung của một đối tượng dữ liệu , hệ thống này có thể tự động cập nhật các replica của đối tượng dữ liệu đó, để dần dần đưa đối tượng dữ liệu đó trở về trạng thái **consistent** (trạng thái mà mọi replica của đối tượng dữ liệu đó có nội dung giống nhau).

Chúng ta có thể thấy, mô hình BASE cung cấp một thước đo có tính mở hơn, có nhiều mức độ hơn và dễ dàng đạt được hơn mô hình ACID. Theo quan điểm của Eric Brewer, thì  BASE và ACID là 2 đầu của thang đo mức độ consistency của hệ thống. Và một hệ thống phân tán trên thực tế sẽ có mức độ consistency nằm giữa BASE và ACID. Tính consistency của một hệ thống phân tán nghiêng về mô hình BASE hay mô hình ACID phụ thuộc vào mô hình consistency mà hệ thống đó sử dụng.

(To be continued)

## Tài liệu tham khảo

[1] Towards robust distributed systems. Eric A.Brewer UC Berkeley and Inktomi. PODC '00 Proceedings of the nineteenth annual ACM symposium on Principles of distributed computing.

[2] Eric Brewer, "CAP twelve years later: How the "rules" have changed", Computer, Volume 45, Issue 2 (2012), pg. 23–29.

[3] Brewer's conjecture and the feasibility of consistent, available, partition-tolerant web services. Seth Gilbert - Massachusetts Institute of Technology, Cambridge, MA; Nancy Lynch - Massachusetts Institute of Technology, Cambridge, MA.

[4] [http://www.dataversity.net/acid-vs-base-the-shifting-ph-of-database-transaction-processing/](http://www.dataversity.net/acid-vs-base-the-shifting-ph-of-database-transaction-processing/)

[5] Principles of transaction-oriented database recovery. Theo Haerder - Univ. of Kaiserslautern, West Germany. Andreas Reuter - IBM Research Laboratory, San Jose, CA

