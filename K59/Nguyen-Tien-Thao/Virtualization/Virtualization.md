# Virtualization(Ảo hóa)

Virtualization đề cập tới kỹ thuật để tạo ra một phiên bản ảo của một `cái gì đó`: Bao gồm phần cứng máy tính, thiết bị lưu trữ, mạng máy tính(switch ảo, router ảo).

Ta đã biết ảo hóa là tạo ra một phiên bảo áo của một `cái gì đó` dựa trên phiên bản thật của cái đó. Vậy tại sao ta phải tiến hành ảo hóa trên phiên bản thật trong khi đó ta đã có phiên bản thật. Vậy ưu điểm của ảo hóa là:

- Trong trường hợp server tập trung, các server nhỏ sẽ được thay thế bằng một server lớn để tăng hiệu năng tính toán cũng như sự linh hoạt. Lúc đó thay vì mỗi máy vật lý chạy một hệ điều hành một cách độc lập thì mỗi hệ điều hành sẽ được cài trên một máy ảo và được hypervisor(sẽ trình bày ở dưới) quản lý và sẽ linh hoạt hơn.
- Việc cấu hình và kiểm tra một máy ảo cũng dễ dàng hơn một máy vật lý.
- Một máy ảo dễ dàng di chuyển từ máy vật lý này sang máy vật lý khác(sử dụng snapshot sẽ trình bày ở dưới).

## Các công nghệ ảo hóa

### Hardware virtualization

Hardware virtualization đề cập đến kỹ thuật để tạo ra một máy ảo hoạt động giống như một máy thật với một hệ điều hành.
Các chương trình thực hiện trên máy ảo chia sẽ nhau tài nguyên phần cứng. Ví dụ như một máy tính đang cài Windows có thể tạo ra một máy ảo cài hệ điều hành Ubuntu linux và các chương trình chạy trên dựa trên Ubuntu có thể chạy được trên máy ảo.

Trong hardware virtualization, khái niệm host machine chỉ máy mà trên đó có thực hiện ảo hóa, còn guest machine là các máy ảo chạy trên host machine. Từ `host` và `guest` dùng để phân biệt chương trình chạy trên máy vật lý và chương trình chạy trên máy ảo. Phần mềm hay firmware để tạo một máy ảo gọi là hypervisor hay Virtual Machine Manager.

Có hai kiểu để ảo hóa phần cứng là:

- Full virtualization(Ảo hóa toàn bộ): Tạo ra một máy ảo mô phỏng lại một máy thật với toàn bộ các tính năng: input/output, hệ điều hành, bộ nhớ,... Với mô hình ảo hóa này guest machine coi nó như một máy tính thực sự và unaware(không ý thức) rằng nó là máy ảo. Dó đó máy ảo nào có thể ra lệnh cho những thứ mà nó nghĩ là phần cứng thực tế nhưng thực tế đó chỉ là các thiết bị được mô phỏng bởi máy chủ.
- Para virtualization(Ảo hóa một phần): Với mô hình ảo hóa này guest aware(ý thức) được rằng nó là khách(guest). Dó dó thay vì ra lệnh trực tiếp cho phần cứng thì nó sẽ ra lệnh cho hệ điều hành trên host machine.

#### Hypervisor

Hypervisor là một phần mềm nằm ngay trên phần cứng nhằm mục đích cung cấp các mội trường tách biệt gọi là partition. Mỗi phân vùng tương ứng với một máy ảo. Các máy ảo có thể cài các hệ điều hành độc lập.

Có hai loại hypervisor:

- Type 1: Native or bare-metal hypervisor: Hypervisor chạy trực tiếp trên phần cứng nó điều khiển phần cứng và quản lý các guest machine.
- Type 2: Host hypervisor: Hypervisor chạy trện một hệ điều hành như một chương trình khác và các guest được tạo ra trên hypervisor.

![https://en.wikipedia.org/wiki/Hypervisor#/media/File:Hyperviseur.png](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e1/Hyperviseur.png/400px-Hyperviseur.png)

#### OS Level Virtualization(or container Virtualization)

Là phương thức ảo hóa hệ điều hành hỗ trợ nhiều instances được cách ly dựa trên một hệ điều hành có sẵn cho nhiều người dùng khác nhau hay nói cách khác là tạo và chạy nhiều
máy ảo dùng chung một hệ điều hành.

#### Snapshots

Một snapshot là một trạng thái của máy ảo được lưu lại. Việc tạo ra snapshot của máy ảo tại một thời điểm nào đó giúp cho hệ thống có thể khôi phục lại đúng trạng thái đó. Ví dụ bạn đang có một máy ảo chạy bình thường nhưng tiếp theo bạn sẽ tiến hành một thử nghiệm mà có thể làm cho hệ thống hỏng thì trước thời điểm tiến hành thử nghiệm đó bạn có tạo một snapshot của máy ảo và sau khi thư nghiệm nếu hệ thống bị hỏng thì có thể dễ dàng khôi phục lại trạng thái an toàn. Một snapshot không chỉ có thể khôi phục lại trạng thái hệ thống tại máy hiện tại mà còn có thể sử dụng để khôi phục trạng thái đó trên một máy khác đây gọi là `Migration`.

### Desktop Virtualization

`Desktop Virtualization` là khái niệm chỉ sự tách biệt giữa `desktop` với máy vật lý.

Một kiểu của `desktop virtualization` là `virtual desktop infratructure` (VDI), có thể coi là một kiểu tiên tiến hơn là `hardware virtualization`. Thay vì tương tác trực tiếp với `host computer` thông qua bàn phím, chuột và màn hình thì bây giờ người sử dụng sẽ tương tác với `host computer` thông qua một kết nối mạng sử dụng desktop khác hoặc một thiết bị di động. Thêm vào đó, `host computer` trong trường hợp này trở thành một server có khả năng làm máy chủ của nhiều máy ảo trong cùng một thời điểm cho nhiều người dùng

### Memory virtualization

`Memory virtualization`: tập hợp tài nguyên RAM của toàn hệ thống vào một nơi  gọi là `memory pool`. `Memory pool` có thể được truy cập bởi hệ điều hành hay các ứng dụng. Với khả năng có thể kết nối mạng, các ứng dụng có thể tận dụng được rất nhiều memory để tăng hiệu suất và khả năng của ứng dụng không còn bị hạn chế.

### Network virtualization

Network virtualization là quá trình kết hợp các phần cứng, phần mềm mạng và chức năng của mạng vào một thực thể duy nhất, một `virtual network`. Nó được chia làm hai loại `external virtualization` và `internal virtualization`

- External virtualization: kết hợp nhiều mạng hay thành phần của mạng vào trong một đơn vị ảo hóa.
- Internal virtualization: cung cấp một chức năng giống như mạng đến các software container trên một hệ thống.

## Tài liệu tham khảo

1. [https://www.linkedin.com/pulse/introduction-cloud-openstack-virtualization-layman-aayush-shrut](https://www.linkedin.com/pulse/introduction-cloud-openstack-virtualization-layman-aayush-shrut)
2. [https://www.linkedin.com/pulse/what-virtual-machine-technology-jim-simpson](https://www.linkedin.com/pulse/what-virtual-machine-technology-jim-simpson)
