# Swift Architect

**Core source**: [https://www.swiftstack.com/docs/introduction/openstack_swift.html](https://www.swiftstack.com/docs/introduction/openstack_swift.html)

## Swift Architecture Overview - Processes

Mô hình cơ bản của một Swift Cluster được mô tả bằng hình vẽ dưới đây:

![https://docs.openstack.org/admin-guide/_images/objectstorage-arch.png](https://docs.openstack.org/admin-guide/_images/objectstorage-arch.png)

Một Swift cluster là một hệ thống phân tán bao gồm một tập hợp các Server vật lý, các Swift Processes (proxy, account, container and object process) và Swift Services được triển khai trên các Server này. Trong hệ thống Swift, các Server được gọi là các Node.

Trong hệ thống Swift, các Node chạy proxy process được gọi là Proxy Node. Các Node chạy  account, container and object process đựoc gọi là Storage Node. Các Storage Node cũng chính là các Node lưu trữ các thực thể trong hệ thống (Container, Account and Data Object). Storage Node cũng chạy một loạt các Swift Services khác để duy trì tính nhất quán cho các thực thể mà  Storage Node đó lưu trữ.

Bây giờ, chúng ta sẽ tìm hiểu vai trò của từng thành phần trong hệ thống Swift.

## Proxy Server

Proxy Node có nhiệm vụ kết nối người dùng với phần còn lại của hệ thống Swift. Khi nhận được request gửi đến, Proxy Node có nhiệm vụ tìm xem tài nguyên Request muốn truy cập (account, container và object) đang nằm tại vị trí Node nào trong hệ thống, thông qua cơ chế Ring (**if a valid request is sent to Swift then the proxy server will verify the request, determine the correct storage nodes responsible for the data (based on a hash of the object name) and send the request to those servers concurrently.**). Sau đó, Proxy Node sẽ chuyển hướng yêu cầu tới Storage Node - Object Servers đang chứa tài nguyên Request muốn truy cập. Các Public API của hệ thống Swift được cung cấp ra bên ngoài thông qua Proxy Server.

Proxy Server có khả năng xử lý một lượng lớn các lỗi. Ví dụ, nếu như một Storage Node không thể xử lý một request PUT một data Object mới, Proxy Server sẽ thực hiện việc tắt Storage Node đó trong Ring và ngừng chuyển tiếp các yêu cầu phân giải tới Storage Node đó.

Khi người dùng upload một data object **x** tới một Storage Node hoặc download  **x**, **x** được truyền trực tiếp qua Proxy Server. Proxy không cache **x** lại buffer.

## Account Layer

Account Server xử lý các request liên quan tới metadata của account, hoặc cung cấp danh sách các container có trong account đó. Thông tin của các Account được lưu trên SQLite database file nằm trên đĩa cứng của Storage Node.

## Container Layer

Container Server xử lý các request liên quan tới metadata của một contaniner, hoặc cung cấp danh sách các object nằm trong container đó. Chúng ta cần lưu ý rằng, danh sách các object này không chứa thông tin liên quan tới vị trí của object trên hệ thống, danh sách này chỉ đơn giản chỉ ra là những object nào nằm trong container này. Thông tin của các container được lưu trên SQLite database file nằm trên đĩa cứng của Storage Node.

## Object Layer

Object Server process xử lý các các request liên quan tới các Data Object thực sự lưu trong các Drive (Các ổ đĩa cứng) của Storage Node chạy process đó.Object được lưu trữ trong Drive bằng cách sử dụng đường dẫn tạo ra bởi đường dẫn của partition chứa nó kết hợp với ID của Data Object đó và time_stamp (time\_stamp cho phép lưu trữ nhiều version cho một Data Object). Object metadata được lưu trữ trong **file’s extended attributes (xattrs)**, điều này có nghĩa là object metadata nằm trên cùng một Drive với Object Content.

## Swift Overview — Cluster Architecture

Trong phần này, chúng ta phân tích cấu trúc của một Swift Cluster

### Node

Node là một Server Vật lý chạy một hoặc nhiều Swift Process. Một tập hợp các Node chạy các process khác nhau sẽ kết hợp với nhau tạo thành một Swift Cluster hoàn chỉnh.

## Regions

Region là một tập các Node nằm tách biệt nhau về mặt vật lý - Thậm chí là địa lý. Một Cluster có ít nhất một region. Một Cluster có >= 2 Region được gọi là multi-region cluster.

![https://www.swiftstack.com/docs/_images/cluster-regions.png](https://www.swiftstack.com/docs/_images/cluster-regions.png)

## Zone

Zone là một tập hợp các Node trong một Region. Một node chỉ thuộc về một Zone và một Region có thể có một hoặc nhiều Zone.
Trên triển khai thực tế, các Zone trên thực tế có thể là các Rack khác nhau của một DataCenter. Trong các kịch bản triển khai phổ biến nhất, Một Cluster có một Region và Region đó là tập hợp của nhiều Zone.

In OpenStack Object Storage, data is placed across different tiers of failure domains. First, data is spread across regions, then zones, then servers, and finally across Drives. Data is placed to get the highest failure domain isolation. If you deploy multiple regions, the Object Storage service places the data across the regions. Within a region, each replica of the data should be stored in unique zones, if possible. If there is only one zone, data should be placed on different servers. And if there is only one server, data should be placed on different Drives.

![https://www.swiftstack.com/docs/_images/cluster-zones.png](https://www.swiftstack.com/docs/_images/cluster-zones.png)

## Data Placement

Với tính chất sao lưu dữ liệu và fault-tolerance của hệ thống phân tán, trong một Cluster, một Data Object sẽ được sao lưu (replicate) thành nhiều Replica lưu trữ tại các vị trí khác nhau trong Cluster đó.

Khi một Swift Service cần xác định vị trí của một thực thể, việc đầu tiên Service đó làm là xác định thực thể định vị là loại thực thể nào (Account, Container hay Data Object). Sau đó, Service đó sẽ sử dụng  Ring tương ứng với loại thực thể của thực thể đó. Có 3 Ring được sử dụng trong Swift, đó là account Ring, container Ring và Data Object Ring.

Mỗi một Ring trong 3 Ring trên đều là các Modified Consistent hashing Ring, các ring này được phân phối tới mọi node trên Cluster. Một Modified Consistent hashing Ring chứa một cặp 2 bảng Lookup Table được Swift process và Swift service sử dụng để xác định vị trí của các thực thể. Trong 2 Lookup Table trên, một bảng cung cấp thông tin về các Drives có trong Cluster, bảng còn lại được sử dụng để xác định các thực thể (account, container or object data) nằm ở vị trí nào trong hệ thống ? Trước khi chúng ta tìm hiểu về các Ring và làm thế nào để Swift tạo ra các Ring, chúng ta sẽ tìm hiểu về các khái niệm partition và replica, vì 2 khái niệm này là 2 khái niệm quan trọng cần nắm được trước khi tìm hiểu Ring là gì.

## Partition

Chúng ta đã biết, Swift sử dụng **modified consistent hashing ring** để xác định một thực thể nằm ở vị trí nào trong hệ thống. Cơ chế quan trọng nhất được thể hiện trong ring là **Hashing**. Khi một Process như Proxy Server process muốn thực hiện việc xác định vị trí của một thực thể, nó sẽ xem thực thể đó là loại thực thể nào và tìm tới Ring tương ứng với loại thực thể đó. Để xác định vị trí của một thực thể, một Ring sẽ được gán một miền giá trị. Ví dụ như với một ring được cấu hình **power=20**, thì miền giá trị được gán cho ring đó là [0, 2^20 -1]. Sau đó, ring sẽ được chia ra làm nhiều phần, mỗi phần tương ứng với một dải giá trị trong miền giá trị của ring. Các phần đó được gọi là các **partitions**, mỗi một parttion sẽ tương ứng với một vị trí lưu trữ dữ liệu xác định trên hệ thống

Để xác định vị trí của thực thể **k**, đầu tiên chúng ta cần thực hiện quá trình Hashing. Sau khi thực hiện Hashing, chúng ta sẽ lấy một phần trong giá trị thu được để định vị. Phần giá trị đó là **x**, và **x** phải nằm trong miền giá trị của Ring. Sau đó, chúng ta sẽ xem giá trị x nằm trong partititon nào, thì partition đó chính là vị trí vật lý của thực thể **k**.

Ở đây, chúng ta có thể thấy, một partition trên ring sẽ tương ứng với một vị trí vật lý xác định trên hệ thống. Sự tương ứng về vị trí ở đây được thể hiện thông qua cơ chế ánh xạ sau: Một partition tham chiếu một thư mục nằm trong một Drive ( một Disk).

**note**: sự tham chiếu tới 1 thư mục trong 1 Drive ở đây mang tính tương đối, vì ở phần sau, theo cơ chế sao lưu partition, chúng ta sẽ thấy một partition có nhiều bản sao, tham chiếu tới các thư mục nằm trên các Drive khác nhau.
![https://www.swiftstack.com/docs/_images/storage-node-partition.png](https://www.swiftstack.com/docs/_images/storage-node-partition.png)

Như vậy, chúng ta có thể thấy cấu trúc lưu trữ bên trong Storage Node là: Một **Storage Node** chứa các **Drive**, và một **Drive** chứa các **Partition**.

Trong khi số lượng Partition trong một Cluster không thay đổi, thì số lượng Drive có trong Cluster đó có thể thay đổi. Ví dụ ban đầu, Chúng ta có 150 partition và 2 Drive, khi đó mỗi một Drive sẽ có 75 partition. Sau đó, nếu chúng ta thêm một Drive vao trong hệ thống, thì hệ thống của chúng ta sẽ có 3 Drive, do đó mỗi một Drive lúc này sẽ có 50 partition.

Partition là đơn vị nhỏ nhất mà Swift làm việc. Các thực thể sẽ được lưu trữ trong các partition, process tìm kiếm các thực thể thông qua partition, và khi một Drive mới được thêm vào hệ thống, một số partition sẽ được di chuyển qua Drive mới đó. Khi hệ thống được thực hiện scale up, thì số lượng partition có trong hệ thống vẫn được giữ nguyên.

**Note**: một Drive chỉ giữ một phần nhỏ các partition trong tập các partiton có trên ring.

## Durability

Swift duy trì  sự ổn định thông qua cơ chế sao lưu dữ liệu, một thực thể trong hệ thống sẽ được nhân bản ra nhiều bản sao và lưu trữ tại các vị trí khác nhau trong hệ thống. Cơ chế sao lưu đảm bảo rằng, khi một phần hệ thống, ví dụ như một ổ đĩa bị gặp sự cố, thì các Data Object vẫn còn các bản sao dự phòng trên các bản sao ở các ổ đĩa khác, do đó dữ liệu được đảm bảo độ an toàn.

Số lượng replica được sử dụng phổ biến là 3. Cơ chế replication của hệ thống Swift bao gồm cả các cơ chế xử lý các trường hợp các Drive trong hệ thống gặp sự cố: Khi một Drive trong hệ thống bị hỏng, các process chịu trách nhiệm sao lưu dữ liệu như replication/auditing processes sẽ được thông báo, sau đó các process này sẽ sao lưu những dữ liệu trên Drive bị hỏng ra một vị trí khác trong hệ thống. Xác suất toàn bộ các bản sao dữ liệu của một partition trên toàn bộ hệ thống bị hỏng là nhỏ, do đó, chúng ta nói rằng Swift có tính chất ổn định.

Ở phần trước chúng ta đã nói rằng, proxy server sẽ sử dụng hashing ring để xác định vị trí của các thực thể trên Cluster. Sau khi chúng ta hiểu được cơ chế sao lưu của hash, chúng ta có thể hiểu rằng, một thực thể có thể có **n** vị trí trên cluster, vì partition chứa thực thể đó có **n** vị trí (1 partition được sao lưu n lần trên n vị trí khác nhau trong một cluster).

![https://www.swiftstack.com/docs/_images/object-ring-mapping.png](https://www.swiftstack.com/docs/_images/object-ring-mapping.png)

## The Ring

Một Ring thể hiện sự mapping giữa tên của một thực thể (Data Object, Account, Container) với vị trí vật lý (nơi lưu trữ nội dung) của thực thể đó. Hệ thống Swift có các Ring riêng cho Account, Container và Data Object. Khi các thành phần khác muốn tương tác với  các loại đối tượng này, chúng sẽ sử dụng Ring tương ứng đẻ xác định đối tượng mà chúng muốn tương tác nằm ở vị trí nào trong hệ thống. Ring xác định vị trí của một Data Object trên hệ thống thông qua zone, device, partition và replica.

Ring thực hiện lookup thông qua logic ring và 2 table sau: devices list and the devices lookup table.

Device list là danh sách các **Drive** được add vào Ring. Một row trong Device list chứa các thôn tin xác định vị trí - địa chỉ truy cập của một **Drive** trên hệ thống: DriveID, zone, weight, IP, port và tên Drive.

![https://www.swiftstack.com/docs/_images/partition-device-table.png](https://www.swiftstack.com/docs/_images/partition-device-table.png)

Device lookup table là một table chứa một danh sách các partition, mỗi row trong table này chứa các thông tin về vị trí của các replica của một partition:

- ID của partition
- Vị trí vật lý của replica thứ i của partition này (i =0,1,2...) trên hệ thống (được xác định bằng DriveID của Drive mà replica này nằm trên).

Ví dụ về Device Lookup table (về mặt triển khai, thì Partition lại là collumn)

![https://www.swiftstack.com/docs/_images/partition-replica-table.png](https://www.swiftstack.com/docs/_images/partition-replica-table.png)

Ví dụ,  Partition 3 có 3 replica (0,1,2) nằm trên các Drive có DriveID lần lượt là là 4, 10, 0

Quay trở lại với vấn đề lookup data trên Proxy server process. Proxy server process sẽ thực hiện việc lookup vị trí chứa data của một thực thể (Account, Container, Data Object) bằng cách Hashing tên của thực thể  cần lookup được một giá trị là Y. Sau đó thuật toán Lookup lấy ra 1 phần thông tin của Y - **Y\_HEAD\_N** ( thực tế là n bit đầu của Y) rồi xác định xem **Y\_HEAD\_N** tương ứng với PartitionID nào.  Partition có PartitionID này chính là Partition đang chứa dữ liệu của thực thể đang lookup. Sau đó hệ thống sẽ tìm xem các replica của partition này đang nằm trên các Device nào thông qua Device Lookup Table. Sau đó, hệ thống sẽ lấy ra thông tin về các Device này thông qua Device List table:  ID number, zone, weight, IP, port, và tên của Device. Dựa vào các thông tin này, Proxy Server process sẽ truy cập vào được các Device đang chứa data của thực thể đang lookup.

Tiếp theo, chúng ta sẽ xem một Ring được tạo ra như thế nào, số lượng parititon trên một Ring, và làm sao để mapping một parition vào một drive trên hệ thống.

## Building a Ring

Khi một ring được tạo ra, số lượng partition được tạo ra trên ring đó được tính toán từ một tham số được gọi là **partition power**. Parititon power cũng như số lượng partition trên Cluster là cố định và chỉ được thiết lập một lần vào lúc bắt đầu xây dựng cluster. Trong suốt quá trình Cluster vận hành sau đó, chúng ta không thể thay đổi tham số partiton power và số lượng partiton trên cluster. Công thức tính số lượng partition trong Swift là **partition\_number = 2^ partition\_power**. Ví dụ, nếu ta chọn partition\_number = 12, thì số lượng partition trên cluster sẽ là 2^12 = 4096.

Trong quá trình khởi tạo cluster, sau khi chúng ta đã xác định số lượng partition có trên Cluster, thì công việc tiếp theo của hệ thống là gán mỗi một partition trên ring với một Drive tương ứng. Khi chúng ta cần rebuilt lại Ring - còn gọi là rebanlancing, chỉ có một số các partition trên các drive phải di chuyển qua các drive khác drive trước đó bị ảnh hưởng. Rebalancing thường được thực hiện khi có một drive mới được gắn vào cluster, hoặc khi một drive bị gỡ khỏi cluster.

Việc hệ thống quyết định một partition sẽ nằm trên drive nào phụ thuộc vào các tham số sau: Số lượng Partition Replica trên Cluster, cơ chế Replica Lock, và Device Weight.

## Partition Replica Count

Như đã trình bày, số lượng Partition trên một Cluster là cố định. Tuy nhiên một Partition trên Cluster được sao lưu thành k bản sao trên các Drive (thường thì k =3). Do đó số lượng Partition Replica trên các Drive của Cluster sẽ là **partition\_number\*k**. Ví dụ nếu chúng ta có số lượng partition trên Cluster là 4096 và k =3, thì số lượng Partition Replica trên các Drive sẽ là 4096*3 = 12288.

## Weight

Swift thiết lập thuộc tính Weight cho từng drive trong Cluster. Người dùng có thể tự thiết lập weight cho từng Drive, và giá trị của weight được thiết lập vào lúc Drive được thêm vào Cluster. Giá trị weight của một Drive tỉ lệ thuận với số lượng Partition cũng như số lượng Partition Replica được gán với Drive đó. Một drive có weight càng lớn thì số lượng Partition - Partition replica tham chiếu tới Drive đó càng nhiều.

## Unique-as-possible Placement

Thuật toán gán một Partition vào một Drive sử dụng các thông số liên quan tới vị trí vật lý của một Drive (Region,Zone, Node-Server, Drive). Mục tiêu của thuật toán này, đó là làm sao cho các Replica của một Partition được lưu trữ ở các Drive càng cách xa nhau càng tốt, để khi sự cố xảy ra, xác xuất tất cả các Replica của Parittion đó đều bị hỏng là nhỏ nhất.

Sau khi chúng ta thực hiện xong công việc gán các Partition trên Ring vào Drive, thì các Ring trên hệ thống cho các thực thể được tạo ra: một Account Ring, một Container Ring, một Object Ring.

## References

1. [https://julien.danjou.info/blog/2012/openstack-swift-consistency-analysis](https://julien.danjou.info/blog/2012/openstack-swift-consistency-analysis)
1. [https://docs.openstack.org/developer/swift/overview_architecture.html](https://docs.openstack.org/developer/swift/overview_architecture.html)
1. [https://docs.openstack.org/admin-guide/objectstorage-arch.html](https://docs.openstack.org/admin-guide/objectstorage-arch.html)
1. [https://www.swiftstack.com/docs/introduction/openstack_swift.html](https://www.swiftstack.com/docs/introduction/openstack_swift.html)