# BÁO CÁO TÌM HIỂU VỀ OPENSTACK

## What is Cloud Computing?

Điện toán đám mây là một mô hình phục vụ truy cập từ khắp nơi một cách thuận tiện tới một kho tài nguyên tính toán chung (như mạng, máy chủ, kho dữ liệu, ứng dụng và dịch vụ) mà có thể được cung cấp một cách nhanh chóng với chi phí quản lý tối thiểu. Mô hình đám mây được cấu thành từ năm tính chất đặc trưng, ba mô hình dịch vụ và bốn mô hình triển khai.

### Các tính chất của điện toán đám mây

*Cung cấp dịch vụ ngay lập tức.* Đối tượng tiêu thụ có thể lập tức yêu cầu khả năng tính toán một cách tự động mà không cần thông qua người dùng tương tác với nhà cung cấp.

*Truy cập rộng khắp.* Tài nguyên điện toán đám mây có thể được truy cập thông qua các cơ chế tiêu chuẩn từ những nền tảng ứng dụng rất khác nhau.

*Dồn tài nguyên.* Tài nguyên tính toán của nhà cung cấp có thể được dồn lại để phục vụ nhiều người tiêu thụ khác nhau theo cơ chế *multi-tenant*, với những tài nguyên vật lý lẫn tài nguyên ảo được gán một cách tự động tùy theo yêu cầu của người tiêu thụ. Người dùng không biết về vị trí thực của các tài nguyên được cung cấp mà chỉ có thể chỉ định và quản lý chúng ở các mức độ trừu tượng cao hơn. Ví dụ về tài nguyên có thể kể đến như kho chứa, khả năng tính toán, bộ nhớ và băng thông.

*Tính mềm dẻo cao.* Khả năng tính toán có thể được cung cấp một cách mềm dẻo và tự động sao cho phù hợp với yêu cầu người dùng.

*Định lượng tài nguyên.* Hệ thống đám mây có khả năng kiểm soát và tối ưu hóa tài nguyên được sử dụng bằng cách điều chỉnh khả năng tính toán ở một mức độ nào đó sao cho phù hợp với loại dịch vụ.

### Các mô hình dịch vụ

*Software as a Service (SaaS).* Cung cấp người dùng khả năng sử dụng các ứng dụng chạy trên nền tảng đám mây. Các ứng dụng này có thể được truy cập từ nhiều giao diện người dùng khác nhau. Người sử dụng không được phép tác động đến hạ tầng đám mây mà chỉ được cung cấp những tính năng hạn chế bởi phần mềm.

*Platform as a Service (PaaS).* Cho phép người dùng triền khai các ứng dụng trên nền tảng đám mây sử dụng các ngôn ngữ lập trình, thư viện hay công cụ được hỗ trợ bởi nhà phát triển. Người dùng vẫn không được phép tác động đến hạ tầng cloud nhưng có quyển kiểm soát các ứng dụng đã được triển khai và cài đặt cấu hình cho môi trường quản lý ứng dụng,

*Infrastructure as a Service.* Cung cấp cho người dùng quyền tác động đến các tiến trình, kho chứa dữ liệu, network và các tài nguyên tính toán cơ bản khác để chạy ứng dụng. Người dùng không được can thiệp vào hạ tầng cloud nhưng có quyền kiểm soát đối với tất cả các tài nguyên mà cloud cung cấp.

### Các mô hình triển khai

####  Private Cloud

Hạ tầng cloud chỉ cho phép sử dụng bởi một tổ chức bao gồm nhiều người sử dụng. Nó được sở hữu, quản lý và điều hành bởi tổ chức đó hoặc một bên thứ ba và có thể được xây dựng bởi chính tổ chức đó hoặc không.

####  Community Cloud

 Hạ tầng cloud cung cấp cho một cộng đồng người dùng cụ thể nào đó thuộc nhiều tổ chức khác nhau.  Quyền quản lý, điều hành thuộc về một hoặc nhiều tổ chức cùng tham gia.

#### Public Cloud

Hạ tầng cloud được cung cấp cho nhu cầu sử dụng công cộng. Quyền quản lý, điều hành thuộc về các doanh nghiệp, các viện nghiên cứu, các tổ chức chính phủ và được xây dựng bởi các tổ chức này.

#### Hybrid Cloud.

Được kết hợp từ các mô hình trên theo các quy tắc tổ chức riêng biệt.

## Kiến trúc hệ thống và các thành phần của OpenStack

### OpenStack architecture

Các thành phần của OpenStack có thể được phân thành ba lớp:

*   Control: Bao gồm các dịch vụ API, giao diện web, cơ sở dữ liệu và bus thông điệp
*   Network: Vận hành các dịch vụ mạng
*   Compute: Một virtualization hypervisor cung cấp các dịch vụ để xử lý đối với các máy ảo

Trong một phiên bản triển khai cỡ nhỏ, cơ sở dữ liệu và dịch vụ chuyển tin được vận hành trên control node, chúng cũng có thể tạo thành các node riêng trong những cấu trúc phức tạp hơn.

Các thành phần/dịch vụ cốt lõi của OpenStack bao gồm:

*   Dashboard (horizon)
*   Identity management (keystone)
*   Image management (glance)
*   Networking (neutron)
*   Compute service (nova)
*   Block storage (cinder)
*   Object storage (swift)

### Dashboard

Bảng điều khiển của OpenStack (còn gọi với tên horizon) cung cấp một giao diện web cho cả quản trị viên lẫn người sử dụng đám mây. Sử dụng giao diện này, cả QTV và NSD có thể cung cấp, quản lý, và điều khiển các tài nguyên đám mây. 

Người dùng có thể thêm người sử dụng, định nghĩa cấu trúc cho một instance, tải lên các images của máy ảo, quản lý mạng, thiết lập các nhóm bảo mật, chạy một instance và truy cập vào instance đó thông qua màn hình console.

Horizon được triển khai trên framework Django.

### Keystone

Dịch vụ xác thực danh tính (keystone) của OpenStack là một dịch vụ chung, cung cấp các dịch vụ xác thực danh tính và quyền trong suốt toàn bộ hạ tầng đám mây với sự hỗ trợ đối với nhiều dạng thức xác thực khác nhau. Keystone đóng vai trò quản lý projects, người dùng và quyền truy cập tài nguyên.

Tất cả mọi thứ trong OpenStack đều tồn tại trong một tenant (project). Mỗi tenant là một nhóm các đối tượng như users, instances và networks. Mỗi người dùng phải ghi danh đối với một tenant nào đó để có thể được authenticate và tham gia giao tiếp với các thành phần khác của hệ thống.

Keystone cũng lưu trữ các catalog bao gồm các dịch vụ và các điểm cuối của từng thành phần trong một cụm. Do mỗi thành phần có một API đầu ra riêng biệt nên bằng cách khai báo chúng cho Keystone, người dùng chỉ cần biết địa chỉ của Keystone server để có thể tương tác với một phân cụm bất kỳ. Ví dụ, nếu muốn biết địa chỉ endpoint của một thành phần nào đó, người dùng chỉ việc yêu cầu Keystone tra cứu và trả về kết quả. Nếu như có sự thay đổi nào đó đối với các endpoints, Keystone sẽ cập nhật lại bản ghi và người dùng sẽ biết đến sự thay đổi này tại lần yêu cầu sau đó.

Keystone sử dụng cơ chế xác thực mặc định là cơ chế tài khoản/mật khẩu. Sau khi xác thực danh tính thành công, Keystone sẽ cấp phát cho người dùng một token thuộc một trong các loại UUID, PKI, PKIZ và Fernet để sử dụng cho các yêu cầu sau đó. Token này có chức năng giúp hệ thống xác định quyền của người dùng đối với tài nguyên được yêu cầu và ra quyết định chấp nhận hay từ chối yêu cầu này. Keystone cũng có thể được tích hợp với các hệ thống bảo mật khác với các chính sách bảo mật riêng biệt và mạnh mẽ hơn.

### Glance

Glance là thành phần quản lý image của OpenStack. Một image là một tập hợp các file tương ứng với một hệ điều hành cụ thể được sử dụng để tạo ra hoặc tái tạo một server. Glance cung cấp các dịch vụ và thư viện hỗ trợ nhằm lưu trữ, duyệt, chia sẻ, phân phát và quản lý các images, các dữ liệu khác phục vụ khởi tạo tài nguyên tính toán và siêu dữ liệu.

Để chuẩn bị vận hành một instance, một bản sao của một image được lựa chọn để đưa vào bộ đệm của compute node dùng để khởi động instance. Các instance sau đó chạy trên cùng một compute node và sử dụng cùng disk image sẽ dùng bản sao trong bộ đệm. Mỗi image lưu trữ trong Glance đã có một hệ điều hành được cài đặt đầy đủ nhưng các thành phần như SSH host key, địa chỉ MAC của các thiết bị mạng được gỡ bỏ để có thể được sử dụng lại mà không phải cấu hình lại toàn bộ hệ điều hành, tránh những xung đột giữa các bản sao của các instances. Các thông tin đặc trưng cho host sẽ được khởi tạo trong quá trình boot, việc yêu cầu các thông tin này được thực hiện bởi một đoạn script gọi là cloud-init.

Các images cũng có thể được cá nhân hóa cho các mục đích khác bên cạnh việc cài đặt hệ điều hành. Nếu một instance được yêu cầu khởi động nhiều lần thì các thao tác cấu hình lặp lại có thể được thực hiện trước và gắn vào disk image. Ví dụ, nếu một disk image được sử dụng để xây dựng một cụm web server, việc cài đặt trước webserver packages lên disk image trước khi lưu nó trong Glance sẽ giúp tiết kiệm băng thông và thời gian hơn là việc cài đặt và cấu hình lặp lại nhiều lần mỗi khi chạy instance.

Cách đơn giản nhất để tạo một disk image là cài đặt máy ảo thủ công, loại bỏ các thông tin đặc trưng cho host và yêu cầu các thông tin này khi chạy bằng cách định nghĩa cloud-init cho image (hầu hết các bản phân phối đều đi kèm đoạn script cloud-init riêng, người dùng chỉ việc thêm chúng vào image).

### Neutron

Sau khi xác thực qua Keystone và thiết lập disk image bằng Glance, tài nguyên tiếp theo được yêu cầu để chạy instance là một virutal network. Neutron là một API frontend giúp quản lý nền tảng Software Defined Networking (SDN). Khi sử dụng Neutron để triển khai một dự án OpenStack, mỗi tenants có thể tạo ra các mạng con ảo và cô lập. Mỗi mạng con này được kết nối tới nhờ các router ảo giúp định tuyến giữa các mạng ảo. Một router ảo có thể được gán với một gateway ngoài, và một instance có thể kết nối với bên ngoài thông qua địa chỉ IP động.

Các thao tác quản lý trực tiếp đối với hạ tầng virtual network được thực hiện bởi Open vSwitch hay các plugins tương tự. Neutron có khả năng sử dụng nhiều plugins cùng lúc để xử lý đối với các thiết bị mạng.

Networking là thành phần phức tạp nhất để cấu hình và duy trì trong cấu trúc OpenStack, bởi nó được xây dựng dựa trên những khái niệm về hạ tầng mạng cơ bản nhất. Người sử dụng cần làm chủ các vấn đề cốt lõi của mạng máy tính để có thể cài đặt Neutron một cách thuần thục.

### Nova

Dịch vụ tính toán của OpenStack được gọi với cái tên *Nova*, cung cấp dịch vụ hỗ trợ quản lý các đơn vị máy chủ ảo dùng để tổ chức các ứng dụng đa tầng, khai phá dữ liệu lớn và tính toán hiệu năng cao. Nova đơn giản hóa thao tác quản lý thông quan một abstraction layer giao tiếp với một hệ thống siêu giám sát (hypervisor) hỗ trợ.

Sau khi thiết lập xong virtual network, cần thêm một cặp khóa và một security group để một instance có thể bắt đầu vận hành. Cặp khóa này là khóa SSH của người dùng, cặp khóa này có thể là khóa cố định hoặc được sinh ra cho mục đích dùng một lần. Khóa công khai trong cặp khóa này được đăng kí là authorized key cho giao thức SSH để không phải yêu cầu khóa khi thiết lập kết nối. Trước khi kết nối SSH được thiết lập, một security group phải được mở. Security group chính là một tường lửa tại hạ tầng cloud, instances có thể giao tiếp với nhau trong cùng một security group, tuy nhiên cần phải thêm các điều kiện để các kết nối ICMP hay SSH có thể được thiết lập đối với instances ở ngoài group.

Tại thời điểm này, một instance đã sẵn sàng được kích hoạt. Các định danh của tài nguyên được cung cấp cho Nova, Nova sẽ kiểm soát tài nguyên nào đang được sử dụng bởi hypervisor nào, bên cạnh đó nó cũng phân bổ instance cho một compute node. Compute node này sẽ lấy Glance image, tạo ra các thiết bị mạng ảo và bắt đầu boot cho instance. Các thông tin cấu hình sau boot được yêu cầu thông qua đoạn mã cloud-init.

### Cinder

Cinder là thành phần quản lý kho dữ liệu khối (block storage) của OpenStack. Các volumes có thể được tạo ra và gán với các instances, sau đó, chúng được dùng trên các instances như các thiết bị khối (block devices). Một block device là một thiết bị di chuyển dữ liệu theo từng khối, chúng là thể hiện của các thiết bị nhớ ngoài hoặc các phân vùng có thể định địa chỉ trên bộ nhớ. Trên một instance, các thiết bị khối có thể được phân vùng và một hệ thống file có thể được khởi tạo và mount (mounting là thao tác gán một phân vùng trên ổ cứng cho một thư mục, dữ liệu trên phân vùng đó có thể được truy cập từ thư mục này).

Cinder còn có thể quản lý các snapshots (trạng thái của một hệ thống tại một thời điểm nào đó). Có thể chụp các snapshots từ block volumes hoặc từ các instances. Các snapshots này có thể được dùng lại sau đó như là nguồn phục vụ thao tác boot.

Logical Volume Manager (LVM) là phương thức mặc định phục vụ cho việc định vị, lưu trữ và quản lý các volumes và snapshots của Cinder.

### Swift

Swift là thành phần kho quản lý đối tượng (object storage). Object storage là hệ thống chỉ lưu trữ nội dung, các files được lưu trữ trong object storage không có các thông tin metadata như với block storage. Swift được cấu thành từ hai tầng: proxy và storage engine. Proxy là một API giao tiếp với người dùng, được cấu hình để giao tiếp với storage engine đại diện cho người dùng. Storage engine là một kho lưu trữ phân tán và được sao lưu, với các backends phổ biến là GlusterFS và Ceph.

### Các thành phần khác

Bên cạnh các thành phần cốt lõi, OpenStack còn cung cấp nhiều dịch vụ hữu ích khác, ở đây xin liệt kê đến hai thành phần là **Ceilometer** và **Heat**

Ceilometer là thành phần có chức năng thu thập dữ liệu từ xa bằng cách đo đạc các tài nguyên đang được sử dụng bởi quá trình triển khai OpenStack. Mỗi mẫu được ceilometer theo dõi một cách thường xuyên, tập hợp các mẫu được tổng hợp thành các thống kê, những thống kê này cho thấy một cái nhìn khái quát về cách thức các tài nguyên đang được sử dụng.

Heat là thành phần phục vụ quá trình chạy đồng thời nhiều instances cùng lúc. Heat lưu trữ các file gọi là templates dùng để định nghĩa instance nào được kích hoạt và thứ tự cũng như sự phụ thuộc giữa chúng. Trong quá trình kích hoạt các instances, các thông tin về thứ tự kích hoạt, sự phụ thuộc giữa các instance, dữ liệu chia sẻ và các cấu hình sau boot sẽ được điều phối thông qua Heat.

## Quản lý danh tính với Keystone


