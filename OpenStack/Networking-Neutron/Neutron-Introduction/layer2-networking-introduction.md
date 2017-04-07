#Giới thiệu về Layer 2 networking
Trong hệ thống OpenStack, các dịch vụ về OpenStack Networking có trách nhiệm xây dựng và quản lý hệ thống mạng ảo cho. Khi triển khai hạ tầng mạng ảo, chúng ta quan tâm tới việc triển khai hệ thống mạng ở Layer 2 và Layer 3. Ở bài viết này chúng ta sẽ tập trung tìm hiểu vấn đề triển khai Layer 2 

Ở Layer 2, vấn đề quan trọng nhất mà chúng ta quan tâm là xây dựng được hệ thống các mạng cục bộ (Local Area Network - LAN).Trên cloud có thể tồn tại nhiều mạng cục bộ, bao gồm mạng công khai chia sẻ, tất cả mọi project đều có thể sử dụng và mạng nội bộ mang tính riêng biệt, chỉ có project sở hữu mới có thể sử dụng. Các loại mạng cục bộ khác nhau có thể triển khai trên OpenStack cloud là: VLAN, VXLAN, GRE, Flat. 

Tuy cách thức hoạt động và khả năng của từng loại mạng là khác nhau, nhưng về cơ bản thì các mạng cục bộ đều có thành phần là các switch, dây dẫn và các máy khách kết nối vào mạng. Trong mạng cục bộ, thì switch là thành phần trung tâm, là thiết bị mà các máy khách kết nối đến, và kết nối với các switch khác để tạo thành một mạng lưới hoàn chỉnh. Việc thiết kế hệ thống mạng theo các phương pháp khác nhau sẽ giúp tạo ra các loại mạng nội bộ khác nhau.

Trong loạt bài này sẽ giới thiệu về Việc triển khai Layer 2 Networking trên OpenStack, trong đó bài viết này sẽ giới thiệu tổng quan về mạng cục bộ và các loại mạng cục bộ tiêu biểu.
##1 Giới thiệu về mạng cục bộ
###1.1 Mạng cục bộ là gì ?
Như chúng ta đã biết, mạng là khái niệm để chỉ một hệ thống các máy tính kết nối với nhau thông qua các phương tiện truyền dẫn. Trong mô hình phân Lớp của OSI, các máy tính kết nối với nhau chủ yếu thông qua 2 layer là layer 2 và layer 3. Mạng cục bộ là mạng được triển khai thông qua Layer 2, trong đó các máy tính được kết nối với nhau thông qua các switch.
![Lan diagram.png](./img/LAN-diagram.png)

Trong mạng cục bộ, các máy tính gửi nhận các gói tin thông qua địa chỉ MAC chứ không phải là địa chỉ IP. Để gửi một gói tin tới một máy tính khác, máy gửi cần phải biết được địa chỉ vật lý (MAC) của máy nhận. Sau đó máy gửi sẽ đóng gói bản tin vào một MAC frame và gửi đi. Các switch trong mạng có nhiệm vụ chuyển tiếp MAC frame này tới máy đích dựa trên địa chỉ MAC đích, việc chuyển tiếp này được thực hiện ở Layer 2 thông qua các công nghệ chuyển tiếp khác nhau. Các công nghệ chuyển tiếp này sẽ dẫn tới các loại mạng cục bộ khác nhau. 

Ở đây, chúng ta có thể hiểu, trong mạng cục bộ, máy gửi chỉ có trách nhiệm đánh địa chỉ gửi và địa chỉ nhận rồi gửi gói tin thông qua cổng mạng tới node tiếp theo trong mạng (thường là 1 switch). Còn phương pháp để gói tin đó gửi tới đích, hoặc làm thế nào để phân chia một hệ thống mạng vật lý ra thành nhiều mạng cục bộ  (multi segment local network), điều đó là phụ thuộc vào cách chúng ta sử dụng loại mạng cục bộ nào.
###1.2 Nguyên tắc chuyển tiếp dữ liệu trong mạng cục bộ
##2 Các loại mạng cục bộ
###2.1 Mạng Flat
Mạng flat là loại mạng cục bộ mà các switch truyền nhận gói tin mà không phải đóng gói thêm các thông tin chuyển tiếp. Gói tin được chuyển tiếp qua hệ thống switch mà không bị biến đổi, vẫn giữ nguyên trạng thái là một gói tin L2 frame.
![flat network.png](./img/flat-network.png)

Trên một miền quảng bá vật lý, không thể triển khai nhiều hơn một mạng flat, do tính chất của mạng flat không thể phân chia miền quảng bá ra thành nhiều mạng nhỏ hơn( điều này xuất phát từ bản chất của mạng flat là trong quá trình  chuyển tiếp gói tin từ máy gửi qua máy đích,các switch không đóng gói thêm thông tin định danh gì cho gói tin , do đó nếu trên một miền quảng bá triển khai nhiều hơn một mạng flat, thì các switch trên miền quảng bá không thể phân biệt được gói tin nào của máy nào trong mạng nào. Chính các thông tin định danh gắn liền với các gói tin sẽ cho biết gói tin đó thuộc mạng nào khi gói tin được chuyển tiếp giữa các switch trong miền quảng bá. Chúng ta sẽ làm rõ điều này ở các loại mạng khác).

Quá trình gửi tin trong mạng flat network được mô tả thông qua ví dụ sau:

Như trong hình vẽ, ta có 2 mạng ```flat network 1``` và ```flat network 2``` được xây dựng trên 2 miền quảng bá khác nhau. Trong mạng flat network 1, Máy ```Computer 1``` muốn gửi một gói tin L2 frame tới máy ```Computer 2``` thông qua layer 2. Quá trình chuyển tin xảy ra như sau:
- Computer 1 cấu hình L2 frame với địa chỉ nguồn là MAC của máy 1, địa chỉ đích là MAC của máy 2 
- Computer 1 gửi tới node kế tiếp (Switch 1) thông qua card mạng nối nó tới mạng  flat network 1. 

- Khi nhận được gói tin, Switch 1 kiểm tra địa chỉ của gói tin và tìm kiếm trong bảng định tuyến của nó. Sau đó nó chuyển tiếp gói tin tới ```Switch 2```.

-  Khi gói tin tới Switch 2, Switch 2 tra bảng định tuyến để tìm đích kế tiếp cho gói tin, vì vậy sau đó nó gửi gói tin tới ```Switch 4```. 

-  Khi gói tin tới Switch 4, vì Switch 4 kết nối trực tiếp với máy 2 nên sau khi tra bảng định tuyến, nó gửi gói tin tới Computer 2.

-  Gói tin tới Computer 2. Việc truyền gói tin L2 frame từ máy Computer 1 tới máy Computer 2 hoàn tất. 

Có thể thấy trong quá trình chuyển tiếp gói tin giữa các switch, gói tin được giữ nguyên, không thêm thông tin định dang mạng nào đính kèm vào gói tin. 
###2.2 Mạng VLAN
Một vấn đề thường xuyên gặp phải khi thiết kế hệ thống mạng là, cần phân chia một miền quảng bá lớn ra thành nhiều mạng cục bộ độc lập với nhau. Chúng ta có thể sử dụng công nghệ VLAN để giải quyết vấn đề này.
![VLAN network.png](./img/VLAN-network.png)

Công nghệ VLAN cho phép chúng ta chia một miền quảng bá vật lý ra thành nhiều mạng cục bộ logic. Mỗi mạng cục bộ được đặc trưng bởi một định danh, đó là VLAN ID. Khi sử dụng công nghệ VLAN, 2 máy trong cùng một mạng cục bộ có thể kết nối với nhau qua mạng cục bộ Layer 2 đó, nhưng nếu 2 máy ở hai mạng cục bộ khác nhau thì không thể kết nối với nhau qua Layer 2, cho dù 2 máy đó cùng nằm trên một mạng quảng bá.

Có rất nhiều cách thức để các switch xác định xem một máy thuộc VLAN nào, ví dụ theo cổng (port): trên Switch 1, từ port 1 -> 5 là VLAN 10, từ port 5 tới port 10 là VLAN 20,... Hoặc cũng có thể xác định theo địa chỉ IP của từng máy như sau:
![VLAN table.png](./img/VLAN-table.jpg)

Khi đã phân chia xong, các máy trên miền quảng bá sẽ được chia ra thành các mạng cục bộ. Việc tiếp theo cần phải làm là xác định cách thức để 2 máy trong cùng 1 mạng VLAN có thể kết nối với nhau. Để thực hiện quá trình này, các switch trên miền quảng bá phải hỗ trợ chuẩn 802.1Q. Chi tiết của chuẩn này có trong tài liệu của IEEE:
```sh
http://standards.ieee.org/getieee802/download/802-1Q-2014.pdf
```
Khi một máy tính gửi một bản tin L2 Frame thông qua layer 2 tới một máy khác trong cùng mạng, gói tin được gửi đi từ máy gửi tới node tiếp theo (switch) dưới dạng 1 L2 frame chuẩn

![VLAN table.png](./img/L2-Frame-format.svg.png)

Khi tới switch, switch xác định xem gói tin được gửi từ máy thuộc mạng VLAN nào. Sau đó, gói tin được điều chỉnh để thêm thông tin định danh, trở thành 1 L2 Frame theo định dạng 802.1 Q

![802-1Q-frame.png](./img/802-1Q-frame.jpg)

Chúng ta có thể thấy, 1 L2 Frame theo chuẩn 802.1 Q so với L2 Frame chuẩn có thêm thông tin định danh cho gói tin là phần 802.1 Q Tag, trong đó định danh cho gói tin thuộc mạng VLAN nào chính là ở phần VLAN ID. VLAN ID có 12 bit, do đó có thể có tối đa 2^12 = 4096 giá trị VLAN ID khác nhau, tương ứng với việc chúng ta có thể có tối đa 4096 mạng VLAN trong 1 miền quảng bá.

Dựa vào VLAN ID này và bảng định tuyến, gói tin được chuyển tiếp tới các switch tiếp theo trên đường đi tới máy đích. Tới switch trước máy đích, switch này nhận thấy máy đích kết nối tới mình nên sẽ thực hiện chuyển L2 Frame từ dạng 802.1 Q về  L2 Frame dạng chuẩn, sau đó chuyển L2 Frame chuẩn này tới máy đích. Quá trình truyền tin kết thúc. Cũng chính nhờ có VLAN ID mà switch có thể xác định máy gửi và máy đích có thuộc cùng 1 mạng nội bộ hay không, từ đó cho phép chuyển tiếp hoặc từ chối chuyển gói tin tới máy khách.

![VLAN_tag_added_removed.jpg](./img/VLAN_tag_added_removed.jpg)

Thông tin thêm về VLAN
```sh
https://en.wikipedia.org/wiki/Virtual_LAN
```
###2.3 Mạng VXLAN
Chúng ta có thể thấy, với mô hình thiết kế của VLAN, bài toán chia miền quảng bá thành các mạng cục bộ con đã được giải quyết. Tuy nhiên mạng VLAN vẫn còn nhiều hạn chế như: Số lượng mạng nội bộ là khá nhỏ (tối đa 4096 mạng / 1 miền quảng bá), có thể không đáp ứng được nhu cầu phân chia trong tương lai; cơ chế định tuyến chưa tối ưu, lý do vì ở Layer 2 không tối ưu cho việc định tuyến mà chức năng chính của nó là chuyển tiếp gói tin, điều này có thể làm giảm hiệu năng  của mạng khi miền quảng bá mở rộng,...
Để giải quyết các hạn chế của mạng VLAN, mô hình mạng VXLAN ra đời như một sự phát triển tiếp theo của VLAN. Nhiệm vụ của VXLAN là tương tự VLAN, tuy nhiên với mô hình thiết kế mới, mạng VXLAN cung cấp những khả năng mới cho việc thiết kế hệ thống mạng. Các ưu điểm của mạng VXLAN có thể kể đến như:

- Khả năng thiết kế hệ thống mạng mềm dẻo hơn: Thiết kế theo mô hình VXLAN cho phép các mạng cục bộ có thể triển khai trên một hạ tầng mạng rộng lớn hơn so với VLAN, có thể triển khai VXLAN trên một hệ thống bao gồm nhiều miền quảng bá kết nối với nhau (thông qua router), nhờ đó việc triển khai mạng VXLAN trên hệ thống mạng trở nên mềm dẻo hơn so với mạng VLAN chỉ có thể triển khai hệ thống mạng trên một miền quảng bá sử dụng các thiết bị của Layer2 (switch, bridge, ...).
- Khả năng mở rộng tốt hơn VLAN: VLAN sử dụng 12 bit để định danh mạng cục bộ, nên chỉ có thể có tối đa 4096 mạng VLAN trong hệ thống, trong khi đó mạng VXLAN có tới 24 bit để định danh mạng cục bộ nên có thể có tới 16 triệu mạng cục bộ trong hệ thống.
- Khả năng định tuyến và hiệu suất của mạng tốt hơn: VXLAN sử dụng công nghệ định tuyến của Layer 3 trong việc truyền dữ liệu, do đó khả năng định tuyến và hiệu suất truyền tin tốt hơn hẳn VLAN trong các hệ thống lớn, do mạng VLAN chỉ sử dụng các switch để di chuyển dữ liệu, mà các switch thì không phù hợp cho việc định tuyến dữ liệu trong 1 hệ thống mạng lớn.

#### Kiến trúc và phương thức hoạt động của mạng VXLAN
Như đã nói ở trên, mạng VXLAN sử dụng Layer 3 để định tuyến. Để hiểu rõ điều này, chúng ta quay lại phần đầu tiên. Như chúng ta đã nói, các loại mạng cục bộ khác nhau chủ yếu ở cách định danh mạng và cách thức truyền gói tin L2 frame tới đích. Ở mạng VXLAN, chúng ta truyền gói tin L2 frame bằng cách sử dụng giao thức MAC-in-UDP (MAC Address-in-User Datagram Protocol). Tức là phương thức để truyền gói tin L2 frame trong mạng là sử dụng IP và UDP để truyền dẫn.

Trong giao thức MAC-in-UDP, gói tin L2 frame cần gửi sẽ được đóng thêm VXLAN header để định danh mạng nội bộ, sau đó được đóng vào UDP header rồi vào IP packet để chuyển đi. Format gói tin của giao thức MAC-in-UDP như sau:

![VXLAN-packet-format.jpg](./img/VXLAN-packet-format.jpg)

Để sử dụng thiết kế của mạng VXLAN, thì các switch trên mạng phải hỗ trợ một công nghệ mới, đó là công nghệ VTEP (VXLAN Tunnel Endpoint). Với việc sử dụng công nghệ này, các switch được coi là các thiết bị VTEP. Các thiết bị VTEP có chức năng xác định các máy tính kết nối tới nó, máy nào thuộc mạng cục bộ VXLAN nào, thực hiện việc chuyển dữ liệu giữa các máy cùng một mạng VXLAN, đóng gói L2 frame vào gói tin của giao thức MAC-in-UDP rồi chuyển đi và chức năng nhận và  mở gói các gói tin của giao thức MAC-in-UDP để lấy gói tin L2 frame ban đầu và gửi tới máy đích.

Để thực hiện các chức năng trên, một thiết bị VTEP bao gồm 2 thành phần chính: thành phần thứ nhất là một switch kết nối và chuyển tiếp dữ liệu giữa các máy tính trong một mạng VXLAN cùng kết nối trực tiếp tới thiết bị VTEP, thành phần thứ 2 là thiết bị đóng gói, mở gói và truyền gói tin MAC-in-UDP thông qua giao thức IP. Thiết bị VTEP có địa chỉ IP xác định, do đó các thiết bị VTEP có thể gửi gói tin IP đến các thiết bị VTEP khách thông qua mạng IP để thực hiện chức năng gửi-nhận các gói tin IP.

![VTEP-device.jpg](./img/VTEP-device.jpg)

#### Phương thức hoạt động của mạng VXLAN
Sau khi tìm hiểu được các thành phần và đơn vị dữ liệu của mạng VXLAN, chúng ta sẽ tìm hiểu cách mà các thiết bị trong mạng VXLAN thực hiện chức năng chuyển tiếp dữ liệu giữa 2 máy trong cùng 1 mạng nội bộ. Chúng ta xét ví dụ sau đây:

![VXLAN-data-flow.png.jpg](./img/VXLAN-data-flow.png)

Ta có 3 máy A, B, C cùng thuộc một mạng cục bộ VXLANID 10. Ta xét 2 trường hợp:

Trường hợp 1, máy A muốn gửi cho máy C một gói tin L2 frame. Lúc này máy A sẽ đẩy lên thiết bị VTEP 1 gói tin L2 frame này với địa chỉ đích là MAC của máy C, thiết bị VTEP-1 nhận được gói tin từ A, xác định được địa chỉ đích của gói tin là C cũng kết nối trực tiếp vào thiết bị và cùng nằm trên 1 mạng cục bộ  VXLAN10 với A, do đó nó chuyển trực tiếp gói tin tới C. Trong trường hợp này chúng ta không cần phải đóng gói và mở gói  gói tin MAC-in-UDP. Thiết bị VTEP đóng vai trò như là 1 switch (mà thật ra thiết bị này là Switch hỗ trợ VXLAN mà thôi).

Trường hợp 2, máy A muốn gửi cho máy B một gói tin L2 frame. Lúc này máy A cũng đấy gói tin L2 frame với địa chỉ đích là MAC của máy B lên thiết bị VTEP 1. Tuy nhiên lúc này máy B không kết nối trực tiếp với VTEP 1, nên quá trình chuyển gói tin tới máy B xảy ra như sau:

Gói tin L2 frame nguyên bản (orginial) được máy A chuyển tới VTEP 1 thông qua mạng vật lý. VTEP 1 nhận được gói tin L2 frame này, nó hiểu được địa chỉ đích là máy B. Lúc này, tại VTEP 1, nó xác định xem máy B được kết nối trực tiếp với thiết bị VTEP nào bằng cách tra trong cơ sở dữ liệu của nó. Sau khi VTEP 1 biết được VTEP 2 là thiết bị kết nối trực tiếp với máy B, nó tiến hành đóng gói dữ liệu:

- Gói tin L2 frame ban đầu được thêm vào VXLAN header để định danh mạng cục bộ.

-  Sau đó đóng gói vào UDP packet với UDP port tương ứng với UDP port dành cho VXLAN.

-   Tiếp đó UDP packet này được đóng gói vào IP packet với địa chỉ IP đích là địa chỉ IP của VTEP 2. 

Sau khi tiến hành đóng gói xong, VTEP 1 thực hiện việc truyền gói tin IP này tới VTEP 2 thông qua hệ thống mạng bằng giao thức IP. 

Sau khi được chuyển tiếp, định tuyến, gói tin tới được VTEP 2. Tại VTEP 2, thiết bị này thực hiện giải nén theo các bước sau:

- Thực hiện việc mở gói tin IP mà nó nhận được ( vì địa chỉ đích là địa chỉ của nó), nó nhận được gói tin UDP. 

- Kiểm tra port của gói tin UDP, gói tin UDP này có UDP port là port dành cho VXLAN, đo đó  nó tiếp tục mở gói tin UDP và xử lý gói tin nhận được theo phương thức xử lý gói tin VXLAN.

-  Dữ liệu nhận trong gói tin UDP bao gồm bản tin L2 nguyên bản và phần VXLAN header. Dựa vào VXLAN header, VTEP 2 biết được máy đích thuộc mạng VXLAN nào nhờ vào VXLAN ID chứa trong VXLAN header (VNID). Đồng thời dựa vào địa chỉ đích của gói tin L2 nguyên bản, VTEP 2 biết được máy đích kết nối với nó ở cổng (port) nào. 

- VTEP 2 chuyển tiếp gói tin L2 nguyên bản của máy A tới máy B thông qua port mà máy B kết nối với nó. Lúc này máy B nhận được bản tin L2 từ máy A gửi tới ban đầu. Quá trình truyền tin kết thúc.

Như vậy có thể thấy ở VXLAN có 2 đặc điểm chính sau:

- Sử dụng VXLAN ID để xác định các mạng VXLAN cục bộ có trong hệ thống mạng. VLANID có độ dài là 24 bit, do đó có tối đa 16 triệu mạng VXLAN trong một hệ thống mạng. 
- Việc truyền bản tin L2 Frame giữa 2 máy cùng một mạng cục bộ nằm ở xa nhau sử dụng truyền tin trên mạng IP + UDP, có thực hiện việc đóng gói và mở gói ở các thiết bị đầu và cuối.

###2.4 Mạng GRE
Để hiểu được nguyên tắc hoạt động của Mạng GRE, trước tiên chúng ta cần hiểu về khái niệm GRE. Theo tài liệu ```rfc2784```, thì GRE - Generic Routing Encapsulation là một giao thức (protocol) cho phép đóng gói một gói tin của giao thức A cùng với GRE header vào trong một gói tin của giao thức B, sau đó vận chuyển đến một thiết bị đích có khả năng giải nén gói tin giao thức B, giải mã GRE header rồi chuyển tiếp gói tin của giao thức A đến đích. Format của giao thức GRE như sau:

```sh
   ---------------------------------
    |                               |
    |       Delivery Header         |
    |                               |
    ---------------------------------
    |                               |
    |       GRE Header              |
    |                               |
    ---------------------------------
    |                               |
    |       Payload packet          |
    |                               |
    ---------------------------------
```
Trong đó, format của GRE Header như sau:
```sh
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |C|       Reserved0       | Ver |         Protocol Type         |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |      Checksum (optional)      |       Reserved1 (Optional)    |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```
(Lưu ý là các trường optional có thể được thay đổi tùy vào mục đích sử dụng của GRE. Ở đây chúng ta quan tâm nhất đến trường Protocol Type. Trường này cho biết giao thức được đóng gói (Giao thức A đã nói ở trên) vào trong gói tin của giao thức bên ngoài (Giao thức B) là giao thức nào.

Như vậy, chúng ta có thể hiểu mục đích của giao thức GRE là sử dụng một giao thức B để vận chuyển gói tin của giao thức A bằng các thiết bị phù hợp. Điều này giúp cho việc vận chuyển các gói tin trên một giao thức có thể trở nên linh hoạt hơn mà không phụ thuộc vào các đặc điểm của giao thức A. Chính ứng dụng này cho phép chúng ta sử dụng giao thức GRE để vận chuyển một bản tin MAC Frame ở layer2 thông qua giao thức IP, đây chính là phương thức mà mạng GRE hoạt động, điều mà chúng ta sẽ tìm hiểu ngay ở phần dưới.
####Sử dụng giao thức GRE để xây dựng mạng nội bộ GRE trong OpenStack
Mục tiêu chính của mạng nội bộ là chuyển tiếp được các gói tin MAC Frame giữa các thiết bị với nhau. Bằng việc sử dụng giao thức GRE, chúng ta có thể xây dựng mạng nội bộ  phân bố ở trên các miền quảng bá khác nhau bằng cách sử dụng GRE protocol để vận chuyển bản tin L2 Frame giữa các miền quảng bá khác nhau này. Mạng GRE trong OpenStack được xây dựng trên kiến trúc như sau:

Như đã nói ở phần trước, việc phân loại các mạng nội bộ là dựa trên cách các thiết bị trung gian (switch) phân chia các máy tính trong mạng vật lý thành các mạng nội bộ như thế nào, và cách chuyển tiếp L2 frame giữa các thiết bị trung gian này như thế nào. Ở mạng GRE, các vấn đề này được giải quyết bằng cách cụ thể hóa format của GRE protocol: Các mạng cục bộ trong cùng 1 hệ thống mạng vật lý được phân biệt với nhau bằng ID nằm trong GRE Header. Payload cần truyền đi là L2 Frame và giao thức được sử dụng để vận chuyển là IP protocol.Format của GRE Protocol như sau:

![GRE-Protocol-Format.png](./img/GRE-Protocol-Format.png)

Quá trình chuyển tiếp gói tin L2 frame giữa 2 máy client ở 2 vùng mạng khác nhau diễn ra như sau:
![GRE-Data-Flow.png](./img/GRE-Data-Flow.png)
Giả sử máy A cần gửi một L2 Frame tới máy B. Máy A sẽ đánh địa chỉ gửi là MAC của A, địa chỉ đích của Frame là MAC của B rồi gửi L2 Frame này qua đường truyền vật lý tới Swich kết nối trực tiếp với nó là Switch C. Tại Switch C, phân tích MAC đích cho thấy Máy B không kết nối trực tiếp tới Switch, nó đóng gói bản tin L2 Frame của A gửi tới theo các bước sau:

- Thêm GRE Header vào với ID của mạng nội bộ mà A và B là thành viên.
- Đóng gói dữ liệu vào IP packet với địa chỉ IP đích là địa chỉ của Switch D là Switch kết nối trực tiếp với máy B bằng cách tra trong cơ sở dữ liệu.

Sau đó, gói tin được gửi đi thông qua hệ thống mạng bằng giao thức IP tới Switch D. Tại Switch D thực hiện quá trình mở gói gói tin IP như sau:

- Sau khi mở gói tin IP Packet, Switch D kiểm tra ID của mạng nội bộ của gói tin và địa chỉ đích của L2 Frame, sau đó tra trong cơ sở dữ liệu để xác định cổng kết nối với máy B.

- Switch D tiến hành loại bỏ GRE Header, sau đó chuyển tiếp L2 Frame nguyên gốc từ Máy A qua cổng kết nối với Máy B. Gói tin được chuyển tới Máy B, quá trình chuyển gói tin kết thúc.

Có thể thấy mạng GRE rất giống với mạng VXLAN. Cả 2 loại mạng này đều sử dụng giao thức IP để chuyển tiếp các gói tin giữa các switch thuộc các miền mạng khác nhau, đồng thời cả 2 loại mạng này đều hỗ trợ tối đa 2^24 = 16 triệu mạng nội bộ. Sự khác biệt nằm ở chỗ, mạng VXLAN sử dụng 3 lớp VXLAN Header + UDP + IP để đóng gói gói tin, trong khi đó mạng GRE chỉ sử dụng 2 lớp là GRE Header + IP để đóng gói, do đó các thiết bị đầu cuối mạng GRE không phải đóng gói và mở gói tầng UDP, đồng thời kích thước gói tin chuyển tiếp mạng GRE sẽ nhỏ hơn kích thước gói tin chuyển tiếp của mạng VXLAN.

