# Threading model - Nova Architech

OpenStack là một hệ thống phân tán, bao gồm rất nhiều các service phối hợp với nhau để quản lý và phân phối tài nguyên đám mây hiệu quả. Một trong các yêu cầu quan trọng nhất khi xây dựng các service trong OpenStack, đó là chúng ta phải đảm bảo tất cả các service làm việc với hiệu năng - performance cao nhất, đảm bảo không có service nào tạo ra ngẽn cổ chai (bottleneck), ảnh hưởng tới khả năng xử lý của toàn bộ hệ thống.

Một trong các tiêu chí để đánh giá hiệu năng của một service, đó là tính **concurrency** của của service. Tính **concurrency** của một service cho biết service đó có khả năng xử lý bao nhiêu công việc trong một khoảng thời gian _delta_. Để tạo ra khả năng concurrency cho các service, có rất nhiều phương pháp, như multiprocessing, multithreading, asynchronous,vv... Giải pháp được các OpenStack service sử dụng để tạo ra tính concurrency, đó là sử dụng mô hình **green thread model** thông qua các thư viện Python **Eventlet** và **greenlet**.

Tại sao OpenStack lại lựa chọn mô hình **green thread model** chứ không phải các mô hình concurrency programming khác? Bài viết này sẽ giúp chúng ta giải đáp câu hỏi này, thông qua việc trình bày các vấn đề sau:

- Concurrency programming là gì?
- Một số giải pháp để thực hiện concurrent programming như MultiProccessing, MultiThreading và implement của các giải pháp trên trong Python
- Mô hình green thread model và các implement của green thread model trong Python: eventlet và greenlet
- Tại sao giải pháp green thread model - eventlet và greenlet lại được OpenStack Project lựa chọn.

## 1. Concurrent programming

### Khái niệm concurrency và các môi trường có tính concurrent

Trước tiên, ta cần phải hiểu, **concurrency** có nghĩa là gì ?

Theo định nghĩa, concurrency có nghĩa là: xảy ra đồng thời. Hai sự kiện được gọi là concurrent với nhau nếu như chúng cùng được thực hiện trong cùng 1 đơn vị thời gian - **time interval**. Nếu một môi trường thực thi cho phép 2 hay nhiều công việc - **tasks** được thực hiện trong cùng 1 khoảng thời gian **time interval**, thì ta gọi môi trường thực thi đó có tính **concurrency**.

Chúng ta cùng xem một số ví dụ về môi trường có tính **concurrency**:

- Concurrency trên môi trường thực thi là hệ điều hành:

Hệ điều hành đa nhiệm là một môi trường có tính concurrency. Khi chạy trên hệ điều hành đa nhiệm, các chương trình độc lập với nhau có thể cùng được thực thi trong một đơn vị thời gian - **time interval**, cho dù môi trường phần cứng phía dưới là CPU đơn nhân, đơn luồng hay đa nhân, đa luồng. Nếu thực thi trên phần cứng đơn luồng, hệ điều hành sẽ chia một **time interval** thành nhiều **time slice**, và mỗi chương trình sẽ được thực thi trong 1 **time slice**, sau khi hết time slice, HĐH sẽ sử dụng **context switching** để tạm ngừng việc thực thi chương trình này và thực thi chương trình tiếp theo trong hàng đợi trong time slice tiếp theo.

- Concurrency trên môi trường thực thi là chương trình

Một chương trình được gọi là có tính **concurrency** nếu như chương trình đó thực hiện được nhiều công việc cùng một khoảng thời gian **time interval**. Chúng ta lưu ý rằng, việc thực thi được nhiều công việc cùng 1 lúc trong chương trình, không đồng nghĩa hoàn toàn với việc thực thi được công việc nhanh chóng. Trong một số trường hợp, concurrency được sử dụng để giúp máy tính giải quyết một bài toán nhanh hơn ( bằng cách chia bài toán ra thành nhiều công việc nhỏ và sử dụng tính concurrency để thực hiện nhiều công việc nhỏ một lúc). Nhưng trong một số trường hợp khác, tính concurrency được sử dụng để giúp chương trình làm được nhiều công việc trong một khoảng thời gian, trong khi tốc độ thực hiện từng công việc chỉ đóng vai trò thứ yếu (*).

Một ví dụ trong thực tế cho (*), đó là trường hợp WebServer. Các website luôn muốn giữ được càng nhiều người truy cập vào website của mình cùng 1 lúc càng tốt. Để làm được việc đó, thì tốc độ người dùng truy cập vào website ( tức là khoảng thời gian từ lúc người dùng gửi yêu cầu đến website server cho đến khi website trả về trang web - tốc độ thực thi công việc) không phải là điều quan trọng nhất. Điều quan trọng nhất trong ví dụ này là có tối đa bao nhiêu người đồng thời được website server phục vụ trong một khoảng thời gian ( số lượng - **capacity**). Mục tiêu trong ví dụ này là phần mềm phải phục vụ được tối đa số lượng connections đến trong 1 khoảng thời gian.

### Concurrent programming và các vấn đề cần giải quyết trước khi xây dựng một concurrent program

Concurrent programming được hiểu là các phương pháp thiết kế và xây dựng nên một concurrent program, hay một chương trình có khả năng thực hiện nhiều công việc trong một đơn vị thời gian - delta. Như đã giới thiệu, trong bài viết này sẽ giới thiệu các phương pháp concurrent programming sau: Multi processing, multithreading, green thread model.

Ba phương pháp trên đều có những ưu điểm và nhược điểm khác nhau, và mỗi phương pháp phù hợp với một số ứng dụng cụ thể trong thực tiễn. Vậy với một ứng dụng cụ thể, chúng ta sẽ sử dụng phương pháp nào trong các phương pháp trên để xây dựng chương trình ? Để có thể lựa chọn được phương pháp phù hợp với chương trình sắp xây dựng, trước tiên chúng ta phải phân tích xem chương trình concurrent programming sẽ được xây dựng có các các đặc điểm nào là quan trọng? Sau đó, chúng ta sẽ đi tiến hành phân tích các đặc điểm này để chọn ra phương pháp phù hợp nhất với các đặc điểm này, sau đó thiết kế hệ thống dựa theo phương pháp đã chọn.

Các đặc điểm quan trọng của một chương trình concurrent programming là: Phân tách hệ thống (decomposition), truyền thông (communication), đồng bộ hóa (synchronization).

- **Decomposition**

Decomposition là qúa trình chia nhỏ vấn đề và giải pháp thành nhiều thành phần. Đôi lúc, các thành phần sẽ là các phần được chia theo logic chức năng ( ví dụ như: searching, sorting, calculating, input, output, etc). Ở một góc nhìn khác, các thành phần cũng có thể được chia theo  logic tài nguyên (file, communication, printer, database, etc.). Sự phân tách (decomposition) của một giải pháp phần mềm liên quan đến vấn đề thiết kế hệ thống (design process).
Đối với concurrent programming, vấn đề cần giải quyết là làm sao để chia tách hệ thống thành các thành phần có thể thực thi đồng thời với nhau (  concurrently executing parts). Vấn đề này cần phải được giải quyêt ngay trong khâu thiết kế hệ thống, và phải được tích hợp vào khi xây dựng mô hình giải pháp cho hệ thống.

Khi tiến hành phân tích và thiết kế hệ thống, chúng ta sẽ tìm ra được mô hình của hệ thống phù hợp với vấn đề. Nếu mô hình hệ thống mà chúng ta tìm được không phù hợp để áp dụng parallel programming và distributed programming, bạn sẽ cần sử dụng mô hình lập trình tuần tự (sequential programming).

- **Communication**

Sau khi bạn chia hệ thống thành nhiều thành phần nhỏ có khả năng **concurrently executing**, bạn sẽ nhận ra các thành phần của hệ thống sẽ cần phải truyền thông với nhau. Ở đây chúng ta sẽ thấy, khi hệ thống làm việc, những câu hỏi sau sẽ được đặt ra : Sự truyền thông sẽ diễn ra như thế nào  nếu các thành phần nằm ở các process khác nhau, hoặc ở các máy tính khác nhau ? Các thành phần khác nhau có chia sẻ chung vùng nhớ nào không ? Làm thế nào để một thành phần nhận được tín hiệu một thành phần khác đã hoàn thành công việc ? Thành phần nào sẽ được chạy đầu tiên ? Làm sao để một thành phần trong hệ thống biết được một thành phần khác đang bị lỗi ? Các câu hỏi vừa nêu ở trên là những câu hỏi chính chúng ta cần phải trả lời để giải quyết vấn đề truyền thông khi thiết kế một hệ thống sử dụng mô hình parallel programming và distributed programming.

- **Synchronization**

Chúng ta đã thực hiện việc chia hệ thống thành nhiều thành phần và xác định sự truyền thông giữa chúng. Đồng thời, khi thiết kế hệ thống chúng ta cũng đã xác định được vai trò của từng thành phần trong hệ thống. Lúc này, chúng ta sẽ thấy, các thành phần trong hệ thống sẽ cần phải hợp tác (coordinated) với nhau để giải quyết bài toán ban đầu đặt ra. Các thành phần trong hệ thống khi hoạt động cũng cần phải phối hợp với nhau theo một trình tự nhất định. Tất cả các thành phần sẽ cùng hoạt động một lúc, hay sẽ có một số thành phần hoạt động và một số thành phần ở trạng thái chờ ? Làm thế nào để xác định quyền và thứ tự truy cập vào một tài nguyên chung giữa hai hay nhiều thành phần? Thành phần nào sẽ được truy cập trước tiên ? Nếu một thành phần thực hiện xong công việc mà nó được giao trước các thành phần khác, nó có được giao công việc mới không ? ...vv...

 DCS (decomposition, communication, and synchronization) là các vấn đề cơ bản nhất mà chúng ta cần phải giải quyết khi tiếp cận và sử dụng mô hình parallel hay distributed programming. Bên cạnh đó, việc xác định vị trí của DCS cũng rất quan trọng. Trong lúc phát triển hệ thống, chúng ta sẽ thấy có hàng loạt layer trong hệ thống có thể áp dụng DCS vào. Các đặc điểm và cách áp dụng DCS ở từng layer cũng sẽ có sự khác nhau.
Concurrency có thể xảy ra ở các tầng sau:

- Instruction level
- Routine (function/procedure) level - threading programming
- Object level - CORBA (Common Object Request Broker Architecture) standard
- Application level

Hai mô hình cơ bản được sử dụng để tạo ra các concurrent program, đó là lập trình song song (parallel programming) và lập trình phân tán (distributed programming). Đây là 2 mô hình lập trình (programming paradigms) khác nhau, nhưng có sự giao thoa lẫn nhau. Mô hình parallel programming - lập trình song song tạo ra các chương trình có nhiều luồng xử lý - thread trên một tiến trình - process, mỗi thread sẽ giải quyết một công việc, và các thread của tiến trình có thể hoạt động đồng thời với nhau. Trong khi đó, mô hình distributed programming cho phép tạo ra một chương trình có nhiều tiến trình phân bố trên nhiều máy khác nhau, các tiến trình này có thể hoạt động đồng thời và song song với nhau để xử lý các công việc, và mỗi tiến trình là một đơn vị xử lý riêng biệt độc lập với nhau. Trong các phần tiếp theo, chúng ta sẽ tìm hiểu về các khái niệm thread, process, đặc điểm của 2 mô hình xây dựng concurrent programm là MultiProccessing và MultiThreading, cũng như cách thức mà 2 mô hình này giải quyết bài toán concurrency.

## 2. Process và MultiProcessing

### 2.1 Process

Process - tiến trình là một đơn vị công việc được tạo ra bởi hệ điều hành (operating system). Một chương trình có thể bao gồm một hoặc nhiều process. Một process trong hệ điều hành có thể là do chương trình của người dùng tạo ra, nhưng cũng có thể là các tiến trình do hệ điều hành tạo ra để quản lý hệ thống. Các hệ điều hành đa nhiệm có thể cùng một lúc thực thi hàng trăm process trong một đơn vị thời gian.

2 thành phần quan trọng nhất của một process, đó là thông tin - information  và luồng chỉ dẫn - control flows. Một process được tạo ra để thực thi một công việc. Công việc mà process phải thực hiện là biến đổi các thông tin. Control flow chỉ ra process sẽ thực hiện công việc như thế nào. Thông tin được process biến đổi có thể là các dữ liệu được lưu trữ trong bộ nhớ chính - các biến, mảng trong chương trình, cũng có thể là các tài nguyên khác trong hệ thống như các tập tin, màn hình, vv...

Khi một process được tạo ra, hệ điều hành sẽ tạo cho process này một không gian địa chỉ - **address space**, sau đó luồng chỉ dẫn của process này sẽ được hệ điều hành nạp lên một vùng nhớ trong không gian địa chỉ này. Khi process được hệ điều hành cung cấp Processor, các câu lệnh trong luồng chỉ dẫn sẽ được nạp vào processor, proccessor thực thi chỉ dẫn, và process lúc này chuyển sang chế độ hoạt động (active). Do tính chất của hệ điều hành đa nhiệm là phải thực thi hàng trăm tiến trình trong một khoảng thời gian, trong khi số lượng processor vật lý có hạn (thường nhỏ hơn rất nhiều so với số lượng tiến trình), do đó hệ điều hành chỉ cấp processor cho tiến trình trong một khoảng thời gian rất nhỏ - được gọi là _time slice_. Hết time_slice, việc thực thi luồng chỉ dẫn của tiến trình hiện tại được tạm ngừng, hệ điều hành chuyển Processor cho một tiến trình khác, và tiến trình hiện tại chuyển sang trạng thái chờ (wait) cho đến khi hệ điều hành cấp Processor cho nó trong lần tiếp theo.

Để quản lý các processs, mỗi khi một process mới được tạo ra, hệ điều hành sẽ tạo ra các thông tin để quản lý process đó:
**proccess id**, **process state** và một entry trong **process table**,vv... Các thông tin này được lưu trữ trong một PCB - Process Control Block. Hệ điều hành sẽ dựa trên các thông tin lưu trong PCB để quản lý và kiểm tra trạng thái của process từ đó đưa ra các quyết định như lập lịch cấp phát CPU, cấp phát tài nguyên, bộ nhớ, quyền đọc ghi dữ liệu lên các thiết bị khác cho process...

#### _Hai loại process trong hệ thống_

Có hai loại process trong hệ thống, đó là **user process** và **system process**.

System proccess là các process do hệ điều hành tạo ra, có vai trò quản lý và giám sát toàn bộ hệ thống. Nhiệm vụ của các tiến trình này là phân phối bộ nhớ (memory), swapping page giữa bộ nhớ trong(RAM) và bộ nhớ ngoài (như là hard drive), kiểm tra các thiết bị đang kết nối tới hệ thống, vv... Đồng thời system proccess cũng thực thi một số chức năng phục vụ cho user proccess như thực thi các I/O request, phân phối và quản lý bộ nhớ, vv...

User procces là các process do các user trong hệ thống tạo ra và khởi chạy. User process thực thi mã nguồn của bản thân nó, và tương tác với hệ điều hành qua việc sử dụng các lời gọi hệ thống ( system function calls). Một user process có 2 trạng thái làm việc là **user mode** và **kernel mode**. Khi một user process thực thi mã nguồn của bản thân nó, process đó sẽ ở trạng thái **user mode**. Khi ở chế độ **user mode**, process đó không thể thực thi các privileged machine instructions. Khi user process thực hiện các  system function calls, nó sẽ thực thi các instruction của hệ điều hành cung cấp. Lúc này proccess sẽ được giữ ở trạng thái **kernel mode** cho đến khi system function calls mà nó gọi tới thực thi xong. Một process đang ở trạng thái **kernel mode** không thể bị OS switch sang một process khác thế chỗ nó.

#### _Anatomy of a Process_

SOURCE: [https://gabrieletolomei.wordpress.com/miscellanea/operating-systems/in-memory-layout/
](https://gabrieletolomei.wordpress.com/miscellanea/operating-systems/in-memory-layout/)

Như đã giới thiệu, một process khi được khởi tạo sẽ được OS cấp cho một không gian bộ nhớ - address space. Không gian bộ nhớ này được process dùng để chứa mã nguồn sẽ được nạp vào processor, cũng như các tài nguyên tính toán của process đó (như biến, hằng, mảng, vv...). Phần này sẽ giúp chúng ta biết address space chứa những thành phần nào và chức năng của chúng trong process.

Chúng ta xem xét một process được chạy trên môi trường cụ thể: **Linux OS** trên vi xử lý **32-bit x86 architecture**
![memory_layout](https://gabrieletolomei.files.wordpress.com/2013/10/memory_layout.jpg?w=960)

Ta biết rằng, mỗi một process sở hữu một không gian địa chỉ riêng - address space. Không gian địa chỉ này là không gian ảo -  virtual address space. Trong trường hợp mà chúng ta đang xét, hệ điều hành là loại 32bit, do đó address space của process có độ rộng là 2^32 bit = 4 GB
Address space của một process được chia ra làm 2 phần lớn: **kernel space** và **user mode space**.  Kernel space trong Linux 32 bit thường có độ rộng là 1GB và có vị trí xác định từ **0x c000 0000** tới **ox ffff ffff**. User mode space có độ rộng là 3GB, có giới hạn từ **0x 0000 0000** tới **0x bfff ffff**

Khi một process đến lượt thực thi, nó được OS giao cho processor để thực thi mã nguồn của nó. Để thực thi một process thì 2 yếu tố quan trọng nhất là instruction và data. instruction chính là mã nguồn của process, còn data của process sẽ tồn tại bên trong một không gian địa chỉ của riêng process đó, được gọi là virtual address space. Với một hệ điều hành 32 bit, không gian địa chỉ này có độ rộng là 2^32 =  4GB, và đây là một không gian địa chỉ ảo (**virtual address space**). Mỗi một process sở hữu một virtual address space riêng.

![window-virtual-address-space](https://i-msdn.sec.s-msft.com/en-us/windows/hardware/drivers/gettingstarted/images/virtualaddressspace01.png)

Việc ánh xạ giữa địa chỉ ảo của một ô nhớ trong process với địa chỉ thật của nó trên RAM, giữa virtual address space với không gian địa chỉ thật trên bộ nhớ vật lý (RAM) của máy tính được thành phần  **page tables** thực hiện.

Vì sao chúng ta lại sử dụng **virtual address space** - VSA thay vì sử dụng trực tiếp địa chỉ các ô nhớ trên RAM ? Việc sử dụng virtual address space mạng lại các lợi ích sau:

- Tính tách biệt (isolate) giữa các process: Khi sử dụng VSA, toàn bộ không gian bộ nhớ mà process nhìn thấy chỉ bao gồm những dữ liệu của bản thân process đó, process đó không thể thấy được các dữ liệu của process khác do các VSA của các process là tách biệt nhau, và 1 process xác định chỉ có thể sử dụng VSA của bản thân nó chứ không thể sử dụng VSA của process khác.
- Process có thể sử dụng các các dải địa chỉ ảo liên tiếp nằm kề nhau trên virtual address space, mặc dù những địa chỉ liên tiếp nhau trên VSA không nhất thiết có địa chỉ ánh xạ tương ứng nằm kề nhau trên bộ nhớ vật lý.
- Trên môi trường multitasking, nhờ có VSA mà tổng dung lượng bộ nhớ của các process có thể vượt qúa tổng dung lượng của bộ nhớ vật lý.

Quay trở lại với việc phân tích address space của một process. Bây giờ chúng ta sẽ xem từng thành phần trong User mode space chứa gì.
![program_in_memory](https://gabrieletolomei.files.wordpress.com/2013/10/program_in_memory2.png?w=960)

- Text: Text segment, hay còn được gọi là code segment, là nơi chứa mã nguồn của chương trình.

Dưới góc độ virtual address space, thì text segment của các process là độc lập với nhau. Nhưng, với các chương trình được chạy từ cùng 1 binary code (ví dụ như 2 terminal process) lại có thể có text segment  ánh xạ về cùng một địa chỉ vật lý. Ta gọi tính chất này của text segment là **shareable**
![mapping-example](https://c1.staticflickr.com/1/414/32784339355_8ca5365f56_o.png)

Việc 2 textsegment của nhiều chương trình chạy từ cùng 1 binary code dựa theo cơ chế *memory mapped files*: Khi 2 process cùng được chạy từ 1 file, qúa trình xảy ra như sau: process đầu tiên sẽ tìm xem file mã nguồn mà process đó sẽ chạy đã có ở trên RAM chưa, nếu chưa có thì sẽ xảy ra page fault trên ram, và kernel sẽ load file đó lên RAM, vùng nhớ RAM chứa file đó lúc này được mapping vào virtual address space của process 1 ở vị trí text segment. Khi chúng ta chạy process 2, do file trên đã load vào ram, nên lúc này sẽ không xảy ra page fault nữa, và lúc này kernel sẽ ánh xạ trực tiếp vùng nhớ RAM đã chứa nội dung của file vào virtual address space của process 2.
Text segment là vùng nhớ **Read-only/Execute** (chỉ được đọc/thực thi) để ngăn process vô tình tự thay đổi mã nguồn của bản thân nó.

- Data: Data segment - “Initialized data segment”, là phần trong user address space chứa các global varibale và static varibale trong chương trình. Datasegment lại được chia làm 2 phần nhỏ:  **Read-only area** (i.e., RoData): chỉ cho phép đọc dữ liệu trong ô nhớ và **Read-Write area**: cho phép đọc-ghi dữ liệu lên ô nhớ.

Ví dụ xét đoạn mã C sau nằm ở global (nằm ngoài hàm main()):

```c
char s[] = "hello world";
int debug = 1;
```

2 biến trên sẽ được lưu vào Read-Write area (khi biên dịch thì biến s[] trở thành một vùng nhớ có kích thước 11 byte, debug trở thành một vùng nhớ có kích thước 4 byte. Việc thay đổi gía trị của s đơn thuần là việc thay đổi gía trị vùng nhớ đó, hay nói cách khác là sau khi biên dịch xong thì s sẽ ành xạ tới một địa chỉ duy nhất và và không thể trỏ sang vùng nhớ khác được. Điều này áp dụng với những trường hợp tương tự. Compiler khi biến một biến thành vùng nhớ sẽ lưu thông tin vùng nhớ đó dưới dạng địa chỉ đầu của vùng nhớ + kích thước của vùng nhớ đó)

Một trường hợp khác cũng nằm trong global area:
const char* str = "hello world";
<http://stackoverflow.com/questions/9834067/difference-between-char-and-const-char>

```html
const char*
is a mutable pointer to an immutable character/string. You cannot change the contents of the location(s) this pointer points to.
```

Chúng ta có thể thấy rằng câu lệnh khai báo trên sẽ tạo ra 2 vùng nhớ, một vùng nhớ để lưu gía trị của str **pointer**, một vùng nhớ để lưu mảng char "hello world". gía trị của str pointer sẽ chính là địa chỉ đầu của vùng nhớ lưu mảng char "hello world".
Trong trường hợp này,  vùng nhớ lưu mảng char "hello world" sẽ nằm ở Read-only Area, còn vùng nhớ lưu str pointer sẽ nằm ở Read-Write Area, vì chúng ta có thể trỏ pointer sang vùng nhớ khác, hay nói cách khác là giá trị của pointer str có thể được thay đổi bởi chương trình.

Tương tự, trường hợp static int i = 10; thì i sẽ được lưu trong vùng nhớ nằm ở Read-write Area.

- BSS: BSS segment -  “Uninitialized data segment”, các ô nhớ trong vùng nhớ này đều được OS khởi tạo với giá trị  là 0 trước khi thực thi chương trình.

BSS segment bắt đầu ở điểm kết thúc của data segment, và nó chứa toàn bộ những biến  global và static variable mới được khai báo nhưng chưa được khởi tạo giá trị hoặc được khởi tạo với giá trị bằng 0. Ví dụ, với câu lệnh khai báo static int i, thì i sẽ được đặt trong BSS segment.

#### *Lý do chúng ta có 2 phân vùng BSS segment và data segment*

<http://stackoverflow.com/questions/9535250/why-is-the-bss-segment-required>
<https://www.motherboardpoint.com/threads/what-for-bss-segment.95280/>

Chúng ta xét một ví dụ cụ thể, xét đoạn mã sau:

```c
int x1 = 0;
int x2 = 0;
...
int x100 = 0;
```

- Trường hợp 1: Nếu như chúng ta chứa mọi biến khởi tạo với gía trị 0 ( chúng ta coi như mọi biến mới được khai báo nhưng chưa được khởi tạo sẽ nhận giá trị 0 lúc bắt đầu thực thi chương trình) trong data segment, lúc này tất nhiên BSS segment sẽ không cần thiết nữa. Lúc này, khi chương trình biên dịch ra mã Assembly, thì nội dung đoạn mã nguồn trên sẽ được dịch thành (dưới dạng pseudo code):

```c
Khai báo và gán địa chỉ x1;
Khai báo và gán địa chỉ x2;
...
Khai báo và gán địa chỉ x100;

Gán vùng nhớ x1 = 0;
Gán vùng nhớ x2 = 0;
...
Gán vùng nhớ x100 = 0;
```

Tổng dung lượng bộ nhớ cần để lưu đoạn mã trên là 100x(kích thước câu lệnh khai báo địa chỉ ) + 100 x kích thước câu lệnh gán vùng nhớ

Nếu sử dụng BSS segment, đoạn mã trên khi biên dịch ra assembly có dạng như sau:

```c
Khai báo và gán địa chỉ x1;
Khai báo và gán địa chỉ x2;
...
Khai báo và gán địa chỉ x100;

memset(start_bss, 0, length_bss)
```

với lengbss là tổng dung lượng các biến chứa trong Bss segment, được compiler tính toán ra trong lúc compile chương trình.

Chúng ta có thể thấy, nếu dùng bss segment, thay vì chúng ta phải lưu 100 lệnh gán gía trị 0 cho các biến chưa khởi tạo hoặc khởi tạo với gía trị =0, lúc này chúng ta chỉ cần lưu 1 biến set tất cả các ô nhớ của bss segment với gía trị =0. Điều này cho chúng ta thấy việc sử dụng bss segment giúp cho file thực thi (là file được compiler biên dịch ra từ file mã nguồn C language) có kích thước nhỏ hơn.

- Stack: Stack là vùng nhớ chứa stack của chương trình - program stack. Stack này có nhiệm vụ lưu trữ mọi dữ liệu mà mộ lời gọi hàm trong chương trình yêu cầu. Cụ thể, tập hợp các biến - gía trị được chứa trong một hàm được gọi là **stack frame**, chứa mọi automatic variables (là các local variable trong thân hàm và các parameter được nạp vào input của hàm) và địa chỉ trả về của hàm - caller return's address. Đây là cơ chế biểu diễn phương thức hàm đệ quy trong C - recursive functions: tập hợp các biến của một hàm đang chạy hoàn toàn độc lập với các biến của một hàm khác.

Stack segment có thể tự điều chỉnh kích thước mỗi khi có một hàm mới được gọi. Dữ liệu của hàm mới được gọi được push vào đầu stack.

- Heap: Heap segment là nơi chứa các dynamic allocate data, tức là chứa các vùng nhớ mà kích thước của chúng chỉ được xác định tại thời điểm chương trình chạy, và không thể xác định cố định bởi compiler khi biên dịch chương trình. Ví dụ:

```c
 int i,n;
  char * buffer;

  printf ("How long do you want the string? ");
  scanf ("%d", &i);

  buffer = (char*) malloc (i+1);
```

Lúc này kích thước của vùng nhớ mà buffer pointer trỏ tới chỉ được xác định khi chương trình đã chạy, và người dùng nhập con số vào scan input. Sau đó câu lệnh **(char*) malloc (i+1);** sẽ tạo ra một vùng nhớ với kích thước **(i+1) *kích thước của ký tự** trong Heap và sau đó buffer sẽ được gán gía trị là địa chỉ đầu của vùng nhớ này.

*Vì sao virtual address space tồn tại 2 phần lớn: **user mode space** và **kernel mode space***  ?

Để hiểu được điều này, chúng ta sẽ tìm hiểu các vấn đề sau:

- Vai trò của user mode space và kernel mode space.
- Sự tương tác giữa user mode space và kernel mode space thông qua system call, và cách mà process thực hiện tương tác này.

Trong hai bộ phân lớn của virtual address space, thì user mode space là không gian địa chỉ mà một process được toàn quyền tác động, lưu trữ mọi dữ liệu mà process đó quản lý. Kernel mode space là không gian địa chỉ dành riêng để chứa một số thành phần của kernel, trong đó có một vùng chứa các câu lệnh kernel - **kernel instruction** cho phép thực thi các chức năng của hệ thống.

Khi thực hiện các câu lệnh chứa trong mã nguồn của process - text segment , process sẽ ở trạng thái **user mode**, lúc này các câu lệnh của process này chỉ có quyền truy cập vào không gian bộ nhớ **User mode space** và thực hiện các câu lệnh có địa chỉ nằm bên trong không gian bộ nhớ này, nếu các câu lệnh này cố tình truy cập vào các ô nhớ có địa chỉ nằm trong không gian **Kernel mode space** thì sẽ tạo ra 1 **exception interrupt**  tác động lên CPU và khiến cho chương trình bị crash.

Tuy nhiên, trong nhiều trường hợp, process của người dùng cần thực hiện mốt số câu lệnh liên quan tới chức năng của hệ thống, vì dụ như: In ra màn hình dòng chữ "Hello world". Vấn đề là thao tác in ra màn hình của hệ thống chỉ có thể được thực hiện bởi **kernel instruction**. Nhưng như ta đã nói ở phần trước, các câu lệnh trong text segment không thể chuyển trực tiếp - như là dùng lệnh jump để chuyển tới **kernel instruction**. Do vậy chúng ta cần phải sử dụng một cơ chế đặc biệt để có thể processor chuyển tới thực hiện kernel instruction - sử dụng system call. Ta xét một ví dụ sau:

```assembly

        .global _start

        .text
_start:
        # write(1, message, 13)
        mov     $1, %rax                # system call 1 is write
        mov     $1, %rdi                # file handle 1 is stdout
        mov     $message, %rsi          # address of string to output
        mov     $13, %rdx               # number of bytes
        syscall                         # invoke operating system to do the write

        # exit(0)
        mov     $60, %rax               # system call 60 is exit
        xor     %rdi, %rdi              # we want return code 0
        syscall                         # invoke operating system to exit
message:
        .ascii  "Hello, world\n"
```

Nguồn: <http://cs.lmu.edu/~ray/notes/linuxsyscalls/>

Khi một process được trao CPU, việc nó làm là thực thi các câu lệnh (instruction) có trong mã nguồn của nó trên RAM.  Các thực thể có thể tồn tại trong một câu lệnh là: Tập các thanh ghi mà CPU cho phép process tác động tới và một không gian địa chỉ bộ nhớ xác định. Khi chương trình cần thực thi một chức năng do kernel instruction cung cấp, nó sẽ ghi vào một register đặc biệt một gía trị đặc biệt tương ứng với việc gọi system call. Trong đoạn mã trên, chúng ta có thể thấy register đó là **rax register**, các gía trị đặc biệt là 1, 60. Trong đó system call tương ứng với gía trị rax = 1 là system call có chức năng in ra màn hình.

Như vậy, chúng ta hiểu rằng câu lệnh trong process chuyển processor tới thực hiện system call bằng sử dụng các thanh ghi các gía trị đặc biệt để báo hiệu cho processor chuyển qua thực hiện system call tương ứng. Đây cũng chính là cơ chế tương tác giữa user mode space và kernel mode space.

Vì tất cả mọi process đều có  sử dụng system call và có phần dữ liệu kernel giống nhau, do đó process chia sẻ virtual address space với kernel bằng cách tách riêng một phần virtual address space cho kernel - kernel mode space. Với mọi process thì kernel mode space đều nằm ở cùng 1 vị trí từ  **0x c000 0000** tới **ox ffff ffff**, và cùng ánh xạ tới cùng một vùng nhớ trên RAM - hay kernel mode space cũng là shareable memory. Tính chất này giúp cho vùng kernel mode space là đồng nhất với nhau ở mọi process, và hơn thế nữa giúp giảm chi phí chuyển đổi khi process thực hiện chuyển đổi từ process này sang process khác, cũng như chi phí khi process chuyển từ user mode sang kernel mode để thực hiện kernel instruction trong kernel mode space.

### 2.2 Multi-Processing Model

Multi-Processing Model là mô hình được sử dụng để tạo ra các concurrent program. Các chương trình được xây dựng dựa trên Multi-Processing Model có khả năng xử lý được nhiều tác vụ trong một khoảng thời gian bằng cách tạo ra nhiều process trên nhiều máy khác nhau. Có rất nhiều chuẩn quy ước cũng như các thư viện hỗ trợ lập tình multi-processing, trong đó phổ biến nhất là chuẩn MPI và các thư viện được xây dựng dựa trên MPI cho C như MPICH, OpenMPI,vv...

#### Các ưu điểm của multi-processing model

#### Các nhược điểm của multi-processing model

## 3. Thread and MultiThreading

Ở đây chúng ta sẽ đi vào một trường hợp cụ thể, đó là mô hình thread trong hệ điều hành Linux. Thread trong Windows và các hệ điều hành khác có cơ chế hoạt động tương tự như mô hình thread trên Linux.

### 3.1 Thread trong Linux

Trong Linux, thread được định nghĩa như sau:

- Một thread được định nghĩa là một _luồng thực thi_ - **stream of executable code** của một Linux proces mà được lập lịch độc lập bởi **scheduler** - thành phần phân phối proccessor của hệ điều hành.
- Thread là một thành phần của process, một process có thể có một hoặc nhiều thread, process có nhiều thread được gọi là multithreading process.
- Trong process, sẽ có một process đóng vai trò **primary thread**, thread này sẽ thực thi luồng điều khiển chính của process, các thread còn lại gọi là **sub-thread**.
- Trong các tiến trình multi-theading, mỗi một thread chứa một luồng điều khiển (khối mã nguồn được lập trình viên viết để thực thi trên thread) riêng, và một thread ( mà không phải là primary thread) sẽ được sinh ra ở một thời điểm xác định bởi process quản lý nó. Sau khi thread được process sinh ra, luồng điều khiển của thread đó sẽ được thực thi độc lập và đồng thời (concurrency) với các thread còn lại trong process.

![multithreading process](http://i.imgur.com/MJxFjwK.png)

#### **The Anatomy of a Thread**

Chúng ta cùng xem cấu tạo của thread thể hiện trên bộ nhớ máy tính.
Đây là ví dụ về một process có chứa nhiều thread:
![process_contains_multiple_threads](https://c1.staticflickr.com/3/2689/32675096571_1e3d1e0634_o.png)

 Như chúng ta thấy, layout của thread được nhúng vào layout của process chứa nó. các thành phần chính của process là stack segment, data segment (Heap + Bss + Data), và text segment. Với thread, các thành phần cần thiết để một thread thực thi là thread stack, thread excutable code, thread data segment.Qua ví dụ trên, ta có thể thấy, thread  excutable code là một thành phần nằm trong text segment, thread data segment cũng chính là data segement của process ( khi thread cần tạo ra một vùng nhớ động (ví dụ như khi trong thread excutable code có chứa câu lệnh buffer = (char*) malloc (i+1);), thì vùng nhớ động đó sẽ được allocate trong heap segment của process). Còn thread stack sẽ được tạo ra bên trong stack segment của process, và các thread khác nhau sẽ sở hữu các stack độc lập với nhau. Thread stack cũng chứa các **stack frame** do các function trong thread excutable code khi thực thi tạo ra như là process stack. một stack frame trong thread stack cũng chứa passing parameter, local variable, và return address của function trong thread.

Thread chia sẻ hầu hết tài nguyên của nó với các threads khác trong process. Thread context định nghĩa các thành phần sau do thread tạo ra: Thread id, tập các register như stack pointer và program counter, và thread stack. Thread chia sẻ tất cả các tài nguyên như process, memory với các thread khác.
Thread tồn tại trong process, và nó có chung vitual address space với process, do đó nó có thể nhìn thấy mọi địa chỉ trong virtual address space.

Có một câu hỏi được đặt ra, là tại sao thread chia sẻ hầu hết tài nguyên của nó với các thread khác trong process, và thread dùng chung virtual address space với process, mà chương trình sau:

```python
#!/usr/bin/python

import thread
import time

# Define a function for the thread
def print_resource( threadName, delay):
        print y

def main():
        y = 16
        # Create two threads as follows
        try:
                thread.start_new_thread( print_resource, ("Thread-1", 2, ) )
                thread.start_new_thread( print_resource, ("Thread-2", 4, ) )
        except:
                print "Error: unable to start thread"
main()
while 1:
        pass
```

lại in ra

```bash
cong@cong-HP-ProBook-450-G1:~$ python threading.py
Unhandled exception in thread started by <function print_resource at 0x7fc7c06c0500>
Traceback (most recent call last):
Unhandled exception in thread started by <function print_resource at 0x7fc7c06c0500>
Traceback (most recent call last):
  File "threading.py", line 6, in print_resource
  File "threading.py", line 6, in print_resource
        print y
NameError: print y
global name 'y' is not definedNameError: global name 'y' is not defined
```

Ta thấy rõ ràng y là một local variable của main thread, theo như lý thuyết thì các câu lệnh trong thread vẫn có thể truy cập được vào ô nhớ chứa y, nhưng tại sao hiện tượng trên lại xảy ra, có gì mâu thuẫn giữa ví dụ trên và lý thuyết hay không ?

Câu trả lời là giữa ví dụ trên và lý thuyết không có gì mâu thuẫn. Lý do là, nếu chúng ta có địa chỉ của ô nhớ chứa local variable y, và viết lại hàm *print_resource* bằng assembly và sử dụng câu lệnh in ra gía trị chứa trong địa chỉ ô nhớ trên, thì màn hình sẽ hiện ra gía trị của biến y. Tức là vẫn đảm bảo thread truy cập được vào mọi ô nhớ (tất nhiên trừ kernel mode space) trong virtual address space. Còn lý do tại sao mà trong chương trình trên chúng ta không lấy được gía trị biến y, là vì chúng ta không lập trình chương trình trên băng assembly mà là lập trình bằng python, lúc này trong câu lệnh **print y** complier cần phải có địa chỉ của biến y, nhưng theo luật LEGB của python thì trong thân của hàm **print_resource( threadName, delay)** không thể truy cập đến local variable y trong hàm main(), do đó màn hình hiển thị ra kết qủa trên. Như vậy là ở đây không xảy ra mâu thuẫn gì cả. Nếu chúng ta lập trình bằng C, chúng ta vẫn có thể truy cập được vào các ô nhớ trong virtual address space.

```c
#include <iostream>
#include <cstdlib>
#include <pthread.h>

using namespace std;

#define NUM_THREADS     5

void *PrintHello(void* y_pointer) {
   int y_receiver = *(int*)y_pointer;
   cout << y_receiver;
   cout <<'\n';
   pthread_exit(NULL);
}

int main () {
   pthread_t threads[NUM_THREADS];
   int rc;
   int i;
   int y = 9;
   int* y_pointer = &y;

   for( i=0; i < NUM_THREADS; i++ ){
      rc = pthread_create(&threads[i], NULL, PrintHello, y_pointer);

      if (rc){
         cout << "Error:unable to create thread," << rc << endl;
         exit(-1);
      }
   }

   pthread_exit(NULL);
}

```

lưu chương trình trên vào file ```threading.cpp```, sau đó các bạn biên dịch chương trình trên bằng lệnh sau:

```bash
g++ threading.cpp -lpthread
```

rồi chạy file được tạo ra sau khi biên dịch (trên máy tính của tôi là file ```a.out```):

```bash
cong@cong-HP-ProBook-450-G1:~$ ./a.out
9
9
9
9
9
```

các bạn có thể thấy, các câu lệnh trong sub_thread vẫn truy cập được vào local variable ```y``` trong primary_thread.

Như vậy, chúng ta đã hiểu multiprocessing và multitheading là gì, cũng như các đặc điểm của process và thread khi chúng hoạt động trong OS. Trong phần tiếp theo, chúng ta sẽ xem một chương trình Python implement mô hình multithreading trong thế nào, và một thành phần quan trọng trong Python VM dùng để xử lý vấn đề synchronization trong multithreading, đó là GIL - Global Interpreter Lock

## 4. Green thread model, Greenlet và Eventlet

### 4.1 Global Interpreter Lock - GIL

Trước khi nói về multithreading trong Python và GIL, đầu tiên chúng ta sẽ tìm hiểu xem làm thế nào để một chương trình viết bằng ngôn ngữ Python trở thành một process chạy trên máy tính.

Như chúng ta đã đã biết, một chương trình viết bằng Python trong một file *program.py* được thực thi

### 4.2 Green thread

Green thread được định nghĩa là các **thread** được lập lịch thực thi bởi **máy ảo của các ngôn ngữ** - như Python VM hay Java VM, thay vì được lập lịch thực thi bởi scheduler của hệ điều hành. Ở trong ngữ cảnh này, chúng ta cần phải hiểu **thread** là một  **stream of executable code**, có nghĩa là thread trong ngữ cảnh này thực tế là một đoạn mã nguồn. Khi sử dụng green thread, trên thực tế VM của ngôn ngữ sẽ quản lý nhiều khối mã nguồn độc lập với nhau, và thời điểm một khối mã nguồn nào đó trong chương trình được thực thi sẽ là do VM của ngôn ngữ quyết định, khác với khái niệm thread trong hệ điều hành ở chỗ thời điểm các thread hệ điều hành được processor thực thi do OS scheduler quyết định.

Một chương trình green thread thường chỉ sử dụng một luồng điều khiển để thực thi tất cả mọi thread trong chương trình. Do đó không thể có nhiều hơn một thread trong chương trình được thực thi cùng 1 lúc.

Khi sử dụng greenlet, việc lập lịch cho các thread trong chương trình là nhiệm vụ của người lập trình. Người lập trình sẽ tạo ra các thread, xác định thread nào sẽ được chạy vào thời điểm nào, và tại thời điểm nào tạm ngừng thực thi thread hiện tại để chuyển sang thực thi một thread khác trong chương trình....vv...

Khi sử dụng green thread, người lập trình cần thận trọng khi thiết kế các thread, vì nếu như một trong số các thread có chứa các vòng lặp vô hạn, thì khi vòng lặp đó được thực thi và thread chứa vòng lặp đó bị treo, toàn bộ chương trình sẽ bị treo.

Như đã trình bày ở trên, chương trình sử dụng green thread chỉ có một luồng điều khiển cho toàn bộ các thread, do đó chương trình này không tận dụng các ưu điểm của các multiprocessor. Việc chạy các chương trình sử dụng green thread trên máy tính đơn lõi so với việc chạy các chương trình này trên máy tính đa lõi là không có sự khác biệt.

Với mô hình green thread, chúng ta có thể điều khiển được việc thực thi và tạm ngừng một thread trong chương trình, do đó giúp chúng ta dễ dàng thực hiện quá trình đồng bộ hơn so với việc sử dụng multithreading với hệ điều hành.

### 4.3 Greenlet

Thư viện **Greenlet** là một implement library của **green thread** trong ngôn ngữ lập trình Python. Khi sử dụng thư viện này, chương trình của chúng ta tạo ra các *greenlet*, là các khối mã nguồn độc lập với nhau. Chương trình của bạn sử dụng các **greenlet** này bằng cách thực thi các câu lệnh trong các greenlet, đồng thời bạn có thể điều khiển được quá trình chương trình nhảy từ một greenlet này sang một greenlet khác để thực thi các câu lệnh chứa trong greenlet đó. Việc chuyển đổi giữa các greenlet trong chương trình được gọi là *switching*. Trước khi chuyển đổi, greenlet chuẩn bị chuyển đổi phải xác định rõ ràng greenlet mà chương trình muốn chuyển qua.

Khi bạn tạo ra một greenlet, nó khởi tạo một stack rỗng, mỗi một greenlet có một stack riêng. Khi bạn chuyển tới thực thi các câu lệnh trong stack của greenlet **a**, khối lệnh của greenlet **a** được nạp vào stack và các câu lệnh trong khối lệnh của greenlet a được stack pop ra lần lượt để chương trình thực thi. Một câu lệnh được thực thi có thể là một câu lệnh thông thường, hoặc một câu lệnh *switch*. Khi hệ thống thực thi câu lệnh **switch** sang greenlet **b**, việc pop các câu lệnh trong stack của greenlet **a** ra được tạm dừng, và hệ thống chuyển qua stack của greentlet được chỉ định trong câu lệnh switch (greenlet b) để thực thi các câu lệnh chứa trong stack của greenlet b. Khi hệ thống chuyển về thực hiện tiếp greenlet **a** các câu lệnh chứa trong stack của greenlet a lại tiếp tục được pop ra từ điểm mà greenlet a bị tạm ngưng trước đó và được chương trình tiếp tục thực thi. Khi tất cả các câu lệnh của greenlet **a** được chương trình thực  thi xong, stack của greenlet a chuyển sang trạng thái **empty**, lúc này greenlet a chuyển sang trạng thái **dead**. Một exception không được handle xuất hiện trong một greenlet cũng có thể khiến greenlet đó bị rơi vào trạng thái dead.

Vậy khi một greenlet **die** , luồng điều khiển của hệ thống chuyển qua khối lệnh nào ? Trong mô hình greenlet, mọi greenlet đều có một **parent** greenlet. Parent của một greenlet **a** chính là greenlet chứa câu lệnh khởi tạo greenlet **a**. Khi một greenlet **die**, luồng điều khiển của hệ thống tiếp tục thực thi các câu lệnh trong parent greenlet của greenlet vừa mới **die**. Bằng cách này, các greenlet trong hệ thống được tổ chức thành một tree, trong đó root của tree đó chính là **main** greenlet, hay là luồng điều khiển chính của chương trình.

Như ta đã nói, các greenlet trong chương trình có thể switch qua lại nhau khi chúng ta sử dụng phương thức switch(). Khi gọi tới câu lệnh này, chúng ta có thể truyền kèm theo các object tới greenlet đích, đây là một cách tiện lợi để có thể trao đổi thông tin giữa các greenlet với nhau.

Ví dụ:

```python
def test1(x, y):
    z = gr2.switch(x+y)
    print z

def test2(u):
    print u
    gr1.switch(42)

gr1 = greenlet(test1)
gr2 = greenlet(test2)
gr1.switch("hello", " world")

```

Trong chương trình trên, chúng ta có ra 3 greenlet gr1 và gr2 và **main greenlet**. gr1 và gr2 nhận main greenlet là parent, do 2 greenlet này được tạo ra trong thân của main greenlet. Sau câu lệnh ```gr1.switch("hello", " world")```, hệ thống chuyển sang thực thi các câu lệnh của greenlet 1, đó chính là hàm test1, với x và y được truyền vào greenlet gr1 là **hello** và **world**. Do vậy, trong hàm test1, ta có x = "hello" và y = "world". Sau đó, câu lệnh ```z = gr2.switch(x+y)``` chuyển chương trình sang thực hiện greenlet gr2, với tham số truyền vào là "hello world". gr2 thực hiện hàm test2, in ra tham số "hello world" rồi switch qua gr1 với tham số truyền vào gr1 là 42. Lúc này gr1 đang tạm dừng tại câu lệnh ```z = gr2.switch(x+y)```, đo đó khi gr2 thực hiện switch sang, z sẽ nhận giá trị là 42. Sau đó câu lệnh ```print z``` được thực hiện, và kết quả hiện ra trên màn hình là :

```bash
hello world
42
```

Như chúng ta thấy trong ví dụ trên, ta có thể thấy greenlet đích nhận các đối tượng do greenlet nguồn gửi qua như là giá trị trả về của câu lệnh ```z = gr2.switch(x + y)``` tại vị trí mà trước đó trong quá khứ greenlet này đã tạm dừng. Tuy nhiên, cho dù câu lệnh switch() sẽ không ngay lập tức trả về kết quả, nhưng câu lệnh này sẽ trả về kết quả tại một thời điểm trong tương lai, khi mà một greenlet khác trong hệ thống switch trở lại greenlet này.

Điều này có nghĩa là `x = g.switch(y)` trong greenlet _g_prev_ được thực hiện, _g_prev_ sẽ chuyển _object y_ tới greenlet _g_, và sau đó một greenlet khác trong hệ thống khi switch back trở lại greenlet _g_prev_ bằng một câu lệnh switch khác như là `g_prev.switch(z)`, thì object `z` sẽ được nạp vào greenlet `g_prev` dưới dạng kết quả trả về của câu lệnh `g.switch(y)`, hay nói cách khác lúc này ta sẽ nhận được kết quả là `x = z`

Greenlet không thể đồng nhất với Python thread, mỗi thread trong hệ thống sẽ chứa một main greenlet riêng biệt, do đó không thể trộn hoặc switch giữa các greenlet nằm ở các thread khác nhau với nhau.

#### 4.4 Eventlet

Eventlet là thư viện implement green thread model (base greenlet) được sử dụng trong OpenStack. Cơ chế hoạt động của eventlet khi xử lý vấn đề request concurrency như sau:

Một chương trình python sử dụng eventlet khi được chạy sẽ chỉ tạo ra 1 process/ 1 thread trong hệ thống. Process này có khả năng xử lý đồng thời rất nhiều request  trong một khoảng thời gian, bằng cách tạo ra một loạt các greenlet. Mỗi một greenlet được tạo ra là một handler độc lập, chịu trách nhiệm xử lý một request. Đặc điểm của các greenlet này, đó là khi một greenlet xử lý reqest, nó được process thực hiện liên tục mà không bị switch sang các greenlet khác, trừ trường hợp greenlet đó phải chờ System I/O ready. Lúc này, như đã trình bày ở phần trước, greenlet này sẽ tạm thời được sleep để chờ System I/O ready, và lúc này process sẽ sử dụng một module đặc biệt là [monkey-patching](http://eventlet.net/doc/patching.html) để switch sang một greenlet khác đang không phải chờ System I/O. Process tạm thời ngừng phục vụ request hiện tại, và chuyển sang phục vụ một request khác, request hiện tại sẽ có cơ hội được phục vụ trở lại khi System I/O ready. Đây là cơ chế giúp Evenlet tạo ra tính chất **non-blocking I/O** - process liên tục thực thi các greenlet mà không ngừng lại khi System I/O busy xảy ra.

Một trong những hạn chế của chương trình sử dụng Evenlet, đó là do chỉ tạo ra 1 process/ 1 thread, do đó hệ thống không có khả năng xử lý cùng 1 lúc nhiều request **cùng một lúc **( việc xử lý nhiều request cùng một lúc chỉ có thể được thực hiện khi sử dụng multi-processing/multi-threading).

#### 4.5 Non-blocking I/O and Eventlet

Như đã nói, khi thiết kế các service trong OpenStack, chúng ta cần lựa chọn giải pháp để tạo ra tính concurrency cho các service. Các giải pháp đã nêu ra ở các phần trước như MultiProccessing hay MultiThreading đều giải quyết được vấn đề concurrency, nhưng giải pháp được lựa chọn để sử dụng trong OpenStack service, đó là green threading model với 2 library **greenlet** và **eventlet**. Green threading model được lựa chọn làm giải pháp khi xây dựng OpenStack service, vì các OpenStacks service đạt được tính concurrency khi chúng ta sử dụng non-blocking I/O.

Nhiệm vụ của các service trong OpenStack tương tự như nhiệm vụ của các web server, đó là phục vụ các request gửi tới service đó. Tính concurrency của OpenStacks service được đánh giá thông qua số lượng request mà service đó có khả năng phục vụ trong cùng một khoảng thời gian -time slice. Như đã nói , để phục vụ cùng một lúc nhiều request trong một time slice, chúng ta có thể sử dụng multi-processing, multi-threading hoặc green-thread. Nhưng lựa chọn giải pháp nào trong các giải pháp đã nêu ra ở trên, lại phụ thuộc vào tính chất của request mà service cần phục vụ.

Khi nhận được một request từ một Client nào đó gửi tới, nhiệm vụ của service là phải tạo ra một luồng xử lý độc lập - **hanlder** để phục vụ request mà service nhận được

![Server_Architect](./image/Server_Architect.png)

Luồng xử lý độc lập này sẽ thực hiện các thao tác sau để phục vụ một request:

- Nhận request từ dispactcher, sau đó đọc nội dung request để hiểu được request đang cần server phục vụ.
- Lấy ra tài nguyên mà request yêu cầu: Handler sẽ lấy tài nguyên từ hệ thống, ví dụ như tải một file ảnh, một đoạn video, một tài liệu, hoặc truy xuất 1 dữ liệu SQL vv...
- Gửi phản hồi -response về Client: Handler lấy tài nguyên vừa lấy được từ hệ thống đưa vào response, sau đó truyền response qua connection (mạng) về Client.

Trong 3 thao tác trên, các thao tác lấy tài nguyên và gửi phản hồi về client là các thao tác tiêu tốn rất nhiều thời gian vào việc chờ đợi I/O của hệ thống trở về trạng thái ready. Trong thao tác lấy tài nguyên của hệ thống theo yêu cầu từ request, thì khi hệ thống đang giao I/O cho một handler khác trong server hay một process khác sử dụng, thì hệ thống sẽ báo I/O đang ở trạng thái busy, và lúc này handler không thể sử dụng I/O của hệ thống đề đọc/ghi tài nguyên - ví dụ như ổ đĩa cứng của hệ thống không thể cùng lúc phục vụ 2 handler/tiến trình, nếu như dữ liệu mà 2 tiến trình này muốn đọc nằm trên 2 vùng nhớ vật lý khác nhau. Lúc này hanldler phải chờ cho đến bao giờ proccess/handler kia sử dụng xong I/O - I/O trở về trạng thái ready thì mới có thể yêu cầu hệ thống giao I/O cho mình. Tương tự như vậy, thao tác gửi phản hồi cho Client cũng tiêu tốn thời gian chờ đợi connection I/O , nếu như connection của hệ thống đang được một handler/process khác sử dụng để gửi dữ liệu. Trong trường hợp handler ghi phản hồi ra bộ đệm của connection I/O, thì handler lại phải chờ đợi I/O  của ổ đĩa cứng ready thì mới có thể ghi response vào bộ đệm, vì bộ đệm của connection thường nằm trên một vùng nhớ của ổ đĩa cứng. Từ những vấn đề này, chúng ta có thể thấy, các handler trong server phải tiêu tốn rất nhiều thời gian để chờ đợi đến lượt các loại I/O trong hệ thống phục vụ nó.

Để tạo ra luồng xử lý độc lập nói trên, chúng ta có 3 giải pháp:

- Sử dụng mô hình multi-processing, với handler là các proccess.
- Sử dụng mô hình multi-threading, với các handler là các thread.
- Sử dụng mô hình green thread model, với các hanlder là các greenlet.

Ở đây, chúng ta sẽ có một câu hỏi, đó là 3 giải pháp trên đối mặt với vấn đề các handler phải chờ đợi **I/O ready** như thế nào ?

- Đối với mô hình multi-processing và multi-threading, khi chuyển qua phục vụ một  handler phải chờ đợi **I/O ready**, kernel của hệ thống sẽ sleep thread/process đang phải chờ đợi và chuyển qua phục vụ các processor/thread khác. Hiện tượng này được gọi là **I/O blocking**.
- Một cách xử lý khác trong mô hình multi-processing và multi-threading, đó là chúng ta sử dụng **non-blocking I/O**. Khi sử dụng cách tiếp cận này, khi handler gọi đến một câu lệnh sử dụng System I/O, nếu như System I/O đang ở trạng thái busy, thì hệ thống sẽ báo lỗi **busy I/O** cho process. Lúc này process hiểu được System I/O đang busy, và lúc này process có thể thực hiện các câu lệnh khác không liên quan tới I/O và sau đó kiểm tra lại xem System I/O đã ready hay chưa ? Nếu System I/O đã ready, process sẽ thực hiện lại câu lệnh sử dụng System I/O mà lúc trước nó chưa thực hiện được.

Như vậy, trong trường hợp sử dụng  **I/O blocking**, kernel hệ thống sẽ sleep process tới khi **System I/O ready**. Còn trong trường hợp sử dụng **non-blocking I/O**, thay vì sleep process, kernel hệ thống sẽ báo lỗi tới process, lúc này process sẽ không bị sleep mà có thể thực hiện tiếp các câu lệnh tiếp theo phía sau câu lệnh I/O (tất nhiên, các câu lệnh phía sau cũng có thể chính là các câu lệnh xử  lý trường hợp process nhận được báo lỗi I/O từ kernel của hệ thống như trong ví dụ của bài viết sau: <https://blogs.gnome.org/markmc/2013/06/04/async-io-and-python/>).

Đối với trường hợp **non-blocking I/O**, ngoài giải pháp báo lỗi, chúng ta còn có thể sử dụng một số giải pháp khác như asynchronous function - thực hiện câu lệnh System I/O mà không cần đợi kết quả trả về, các behavior của câu lệnh asynchronous sẽ được thực hiện vào một khoảng thời gian khi mà System I/O của hệ thống ready, callback - đi kèm với asynchronous function - các phương thức xử lý kết quả trả về của một asynchronous funtion, vv... Mục tiêu khi sử dụng **non-blocking I/O**, đó là tạo ra một cơ chế cho phép khi một process thực hiện các câu lệnh liên quan tới System I/O  khi System I/O đang busy, thì process đó sẽ không bị kernel sleep nữa, mà process đó sẽ được tiếp tục thực hiện các câu lệnh phía sau câu lệnh System I/O đó.

- Mô hình green thread model cũng sử dụng **non-blocking I/O** để giải quyết vấn đề System I/O, tuy nhiên cách thực hiện **non-blocking I/O** của mô hình này khác so với cách thực hiện của multi-processing/multi-threading. Khi một greenlet đang phải chờ đợi **I/O ready** từ hệ thống, process sẽ tạm thời sleep greenlet đang chờ **I/O ready** rồi switch sang thực hiện một greenlet khác. (Trước khi switch sang greenlet khác, greenlet đang chờ **I/O ready** sẽ được gán một call_back, cho phép nó xác định được khi nào nó được sử dụng System I/O). Vì tất cả các greenlet của hệ thống đều cùng nằm chung một process/thread, do đó việc switch từ greenlet này sang greenlet khác trong process đó thực chất không làm ngừng lại quá trình hoạt động của process, mà chỉ đơn giản là process đó thay vì thực hiện các câu lệnh thuộc greenlet này thì chuyển qua thực hiện các câu lệnh thuộc greenlet khác. Vì vậy, process sử dụng green thread model sẽ không bị sleep khi System I/O đang busy, mà chỉ có các greenlet đang chờ I/O ready mới chuyển sang trạng thái **"sleep"**. Chính vì vậy, mô hình green thread model thỏa mãn tính chất của  **non-blocking I/O**

Bên cạnh việc green thread model thỏa mãn tính chất **non-blocking I/O**, thì green thread model còn một số ưu điểm nữa so với mô hình multi-processing và multi-threading. Đó là  việc một process thực hiện switch giữa hai greenlet trong cùng process đó ít tốn kém chi phí hơn nhiều so với việc kernel switch giữa 2 thread hoặc 2 process, cũng như chi phí bộ nhớ dành cho các greenlet ít hơn  so với chi phí bộ nhớ để tạo ra các process/thread độc lập.

Chính vì những ưu điểm kể trên của green -thread model, nên OpenStack services đã sử dụng mô hình green-thread model để giải quyết vấn đề concurrency, thay vì sử dụng multi-processing/multi-threading.

## Reference

1. [https://vietstack.wordpress.com/2015/08/24/openstack-eventlet-notices/](https://vietstack.wordpress.com/2015/08/24/openstack-eventlet-notices/)
1. [http://techbackground.blogspot.com/2013/03/eventlet-snippets.html](http://techbackground.blogspot.com/2013/03/eventlet-snippets.html)
1. [https://en.wikipedia.org/wiki/Coroutine](https://en.wikipedia.org/wiki/Coroutine)
1. [http://eventlet.net/doc/](http://eventlet.net/doc/)
1. <http://www.hydrogen18.com/blog/fast-async-python-eventlet.html>
1. <https://vaidik.in/blog/understanding-non-blocking-io-with-python-part-1.html>
1. <https://blogs.gnome.org/markmc/2013/06/04/async-io-and-python/>
1. Parallel and Distributed Programming Using C++, Cameron Hughes, Tracey Hughes
