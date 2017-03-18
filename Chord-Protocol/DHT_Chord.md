#Flat Naming Solution: DHT - Chord in Distributed System
Bài viết trình bày các vấn đề sau:

- Định nghĩa của khái niệm ```name``` , và vai trò của ```Naming System``` trong Distributed System
- Flat Naming trong Distributed System và một solution cho Flat Naming - DHT/Chord

##Name, Indentifiers and Address
Trong Distributed System, ```name``` là một đối tượng được dùng để định danh - đại diện - tham chiếu một thực thể trong hệ thống. Name có thể tồn tại dưới dạng một chuỗi ký tự, một chuỗi số, vv,... Đối tượng được name đại diện có thể là một process, một nhóm các đối tượng, hoặc một thiết bị, một cảm biến nào đó trong bài toán về IoT, vv...

Identifier là một ```name``` dùng để định danh đặc biệt cho thực thể. Identifier có tính chất sau: 

- Một **identifier** chỉ đại diện cho **tối đa một thực thể** trong hệ thống. 
- Một thực thể có ít nhất một **identifier** đại diện cho.

Address là **name** có ý nghĩa như là cổng truy cập - **access point** của thực thể. Điều này nghĩa là, có được **address** của một thực thể, chúng ta có thể truyền thông trực tiếp tới thực thể đó.

##Naming System
Naming System là một hệ thống con của một Distributed System, vai trò của Naming System là phân giải tên cho toàn bộ hệ thống.

Cụ thể hơn, vấn đề mà Naming System cần giải quyết là: Khi một thực thể A trong hệ thống có được **identifier-name** của thực thể B, bây giờ thực thể A muốn truyền thông với thực thể B, nó cần phải có **address** của thực thể B. Để có được address của B, A sẽ gửi lên Naming System **identifier-name** của B. Naming System sẽ gửi trả về cho A **address** của B tương ứng với **identifier-name** mà A gửi lên, lúc này A có thể truyền thông với B thông qua **address** này.


 Để thực hiện vai trò này,các nhiệm vụ của một Naming System là:
- Lưu trữ ánh xạ giữa **identifier-name** và **address** của các thực thể
- Khi có một yêu cầu phân giải một **identifier-name** tới từ một thực thể nào đó của hệ thống, nhiệm vụ của Naming System là phải trả về cho thực thể đó **address** tương ứng với **identifier-name** đó.

Ví dụ về Naming System là hệ thống DNS: khi một trình duyệt trên một máy trong mạng cần lấy địa chỉ - **address** tương ứng với tên một trang web - **identifier-name**, nó sẽ gửi lên DNS System tên trang web đó, ví dụ nó sẽ gửi lên tên trang web **"facebook.com"**. Lúc này, nhiệm vụ của DNS System là phải gửi về **địa chỉ IP** - address tương ứng với tên trang web facebook.com được lưu trữ trên DNS System, ví dụ như **31.13.95.36**. 
![DNS_resolution_name](http://anouar.adlani.com/assets/images/posts_illustration/2011-12-21-useful-dig-command-to-troubleshot-your-domains/how_dns_works.png)
Ở trong ví dụ này, thực thể yêu cầu phân giải tên là **trình duyệt**, thực thể cần phân giải tên là **trang web facebook.com**, identifier-name của trang web này là **facebook.com**, address của **trang web facebook.com** tương ứng với identifier-name trên là **31.13.95.36**

##Flat Naming
Flat Naming là một cơ chế để đặt **identifier-name** (gọi tắt là identifier) cho một thực thể trong hệ thống. Khi sử dụng Flat Naming, **identifer** của một thực thể trong hệ thống chỉ đơn giản là một chuỗi ký tự không có cấu trúc, quy tắc. Identifier này không chứa bất cứ thông tin nào về vị trí của thực thể trong hệ thống.
Có 4 phương pháp để xây dựng Naming System cho một Distributed System sử dụng Flat Naming: 
- Boardcasting
- Forwarding pointer
- Home-based approaches
- Distributed Hash Tables (DHTs)

Trong bài viết này chúng ta sẽ tìm hiểu phương pháp Distributed Hash Tables (DHTs).

###Distributed Hash Tables
source: https://en.wikipedia.org/wiki/Distributed_hash_table
Distributed Hash Table (DHT) - Bảng băm phân tán, là một lớp các hệ thống phân tán cung cấp dịch vụ tìm kiếm (lookup service). Về chức năng, DHT có chức năng tương tự như một bảng băm: 
- Các cặp (key,value) sẽ được đưa vào lưu trữ trong hệ thống DHT.
- Nhiệm vụ duy trì ánh xạ cho các cặp (key,value) được phân tán trong các node của hệ thống DHT
- Các node tham gia hệ thống DHT có thể sử dụng lookup service mà DHT cung cấp: khi một node đã có được một key, nó có thể tìm được value tương ứng với key đó.

![DHT](https://upload.wikimedia.org/wikipedia/commons/thumb/9/98/DHT_en.svg/500px-DHT_en.svg.png)

Chúng ta có thể thấy, đặc điểm của DHT là:

- Toàn bộ hệ phân tán tham gia xây dựng Naming System cho chính hệ phân tán đó.
- Các node tham gia vừa đóng vai trò là node tham gia hệ thống Naming System, vừa là các node sử dụng dịch vụ mà Naming System cung cấp.

Chúng ta có thể nhận ra DHT có các đặc điểm khác với một số Naming System khác như DNS:

- DNS chỉ sử dụng một số các máy chủ xác định trong toàn bộ hệ phân tán để làm nhiệm vụ xây dựng Naming System.
- Các node khác trong hệ thống không tham gia vào DNS, nhưng vẫn có thể sử dụng dịch vụ mà DNS cung cấp.
- Để sử dụng DNS, các node trong hệ thống cần có được Access Point của DNS. Ví dụ như DNS Access Point của Google DNS là 8.8.8.8 

Bây giờ chúng ta đi tới xét một loại DHT cụ thể, đó là Chord.

###Chord

Quay trở lại vấn đề đặt ra ban đầu, nhiệm vụ của một Naming System là phân giải tên : cho đầu vào một **identifier**, chúng ta cần tìm **address** tương ứng với **identifier** này.

Vậy Chord giải quyết nhiệm vụ này như thế nào ? Chúng ta cùng xem các nguyên tắc thiết kế Chord:

- Mỗi một node trong hệ thống sẽ chọn một  **m-bit idenifier** - thường là chọn random
- Có thể truy cập tới tất cả các node thông qua network address của nó.
- Như đã nói ở trên, Chord là một DHT, do đó hệ thống này cho phép các node trong Chord lưu trữ các cặp **key-value**. Để thực hiện nhiệm vụ của Naming System là phân giải tên, mỗi một thực thể cần định danh trên hệ thống cũng sẽ được gán một **m-bit identifier**, tương ứng với **m-bit identifier** này là một **object-address **của thực thể đó, object-address này sẽ chứa address của thực thể, **m-bit identifier** và  **object-address** trở thành một cặp **key-value** được lưu trữ trên các node của Chord.

Để lưu trữ các cặp **key-value** trên các node của hệ thống, nguyên lý của Chord quy định rằng mỗi cặp **key-value** có **key** là **k** sẽ được lưu trữ trên node có **m-bit identifier** là **i** nhỏ nhất thỏa mãn điều kiện: **i** >= **k** (###) (chúng ta cần nhớ rằng, k cũng là 1 m-bit identifier). Node này được gọi là **succesor of k**, hay còn gọi là **succ(k)**.

Ghi chú: (###) Điều kiện trên chỉ đúng với đa số các node trên hệ thống. Điều kiện thật sự để một node **i** là successor của **k** là **k** nằm giữa **prev-i** và **i**  ( k # prev-i, k có thể == i), trên ring logic của hệ thống. Phần thuật toán của Chord chúng ta sẽ nói rõ hơn về điều kiện này.

Trở lại nhiệm vụ của Naming System, ngoài nhiệm vụ lưu trữ các cặp key - value, Naming System còn có nhiệm vụ tìm kiếm **address** tương ứng với **identifier**, hay  tìm kiếm value tương ứng với một key cho trước. Nhiệm vụ tương ứng trên Chord đối với nhiệm vụ này, đó là giải quyết bài toán:
```
Trên node có identifier n , ta có m-bit identifier k, tìm node succ(k)? (*)
```

*Tại sao nhiệm vụ phân giải **identifier** của Naming System lại tương ứng với nhiệm vụ giải quyết bài toán (*) ?*
 
 Ví dụ về ngữ cảnh cụ thể của bài toán phân giải tên trên một hệ thống sử dụng Chord như sau:

Hệ thống Distributed System của chúng ta đã được cài đặt Chord. Trong qúa trình hoạt động, **process z** trên node có **identifier n** nhận được tin nhắn từ **thiết bị t** trên hệ thống có **identifier k**. Để có thẻ phản hồi cho **t**, **z** cần phân giải **identifier k** sang **address** của t, bằng cách gửi tới node chứa nó (node n) một thông điệp yêu cầu phân giải tên, với các nội dung:
- Address của chính nó (z-address)
- Identifier cần phân giải (k)

Node n sẽ xử lý thông điệp này. Nếu nó không phải là **succ(k)**, nó sẽ không thể đáp ứng yêu cầu phân giải. Lúc này nó sẽ chuyển thông điệp này tới một node khác trên hệ thống thông qua một thuật toán được implement trên Chord, thuật toán này chính là thuật toán được Chord sử dụng để giải quyết bài toán (*).  Thông điệp yêu cầu phân giải được chuyển đi trong hệ thống cho đến khi tới node **succ(k)**. Lúc này node **succ(k)** nhận được thông điệp và nhận thấy nó có thể phân giải được **identifier k**, nó tiến hành phân giải **k**, sau đó nó gửi kết qủa phân giải là **address của t** về cho **z** (vì trong tin nhắn yêu cầu phân giải, **z** đã điền address của nó vào). Lúc này **z** nhận được **address** của **t**, qúa trình hệ thống Chord thực thi nhiệm vụ phân giải **identifier** kết thúc.
Quay trở lại với bài toán (*):
```
Trên node có identifier n , ta có m-bit identifier k, tìm node succ(k)?
```
Bài toán này được giải quyết như sau:
![chord_algorithm](https://c1.staticflickr.com/3/2692/32055771334_cffea55c09_b.jpg)

1 - Các node trên hệ thống sẽ được sắp xếp trên một ring, theo thứ tự tăng dần Identifier của chúng.
2 - Mỗi node **p** trên hệ thống sẽ chứa một bảng chứa m dòng, được gọi là FingerTable. Dòng thứ i trên bảng này có nội dung là 
```
FTp[i] = succ(p + 2^(i-1))
```
3 - Nếu một node **n** trên logic ring nhận được yêu cầu phân giải identifier **k**:

- Đầu tiên, node **n** sẽ kiểm tra xem nó có phải là node chứa **k** không, bằng cách kiểm tra điều kiện  **k** có nằm giữa **n** và **prev-n** trên vòng logic  hay không ? ( với k # prev-n, k có thể == n).  Ví dụ, nếu k = 30, n = 1 với vòng logic ở hình vẽ trên, thì n là **succ(k)**.
- Nếu n không phải là **succ(k)**, nó sẽ chuyển tiếp - forward yêu cầu phân giải k sang node có identifier là q với q được chọn theo nguyên tắc sau:
	- Nếu k nằm giữa **n** và **next-n** ( next-n là node ở kế tiếp n theo chiều kim đồng hồ trên ring logic), q là **next-n**
	- Nếu k không nằm giữa **n** và **next-n**:
		- Sử dụng FingerTable, chúng ta tìm chỉ số j (1<=j<=m) thỏa mãn: k nằm giữa FTp[j] và FT p[j+1] trên vòng tròn logic. Nếu tồn tại chỉ số j thỏa mãn điều kiện trên thì q là FTp[j]
		- Nếu không có j thỏa mãn điều kiện trên ( k > FTp[m]) thì q là FTp[m].



 



> Written with [StackEdit](https://stackedit.io/).