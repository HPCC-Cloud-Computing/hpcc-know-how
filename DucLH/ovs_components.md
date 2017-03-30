# Open vSwitch
## Overview
Open vSwitch (OVS) là một switch dưới dạng phần mềm được xây dụng với kiến trúc nhiều tầng (multilayer) được cấp phép theo giấy phép của Open Source Apache 2. OpenVswitch có thể được định nghĩa là một nền tảng tạo ra các virtual switch, hỗ trợ interface quản lý chuẩn và mở để có thể điều khiển và lập trình bởi người dùng

OVS cũng phù hợp để hoạt động như một switch ảo trong mô trường VM. Ngoài ra, để trình diễn interface và control chuẩn đến virtual networking layer, OVS được thiết kế để hỗ trợ distribution thông qua multiple physical server. OVS hỗ trợ nhiều công nghệ ảo hóa dựa trên linux bao gồm Xen/XenServer, KVM và VirtualBox.

Phần lớn code của OVS được viết bằng C và dễ dàng để kết hợp đến các môi trường khác. Phiên bản hiện tại của OVS hỗ trợ các tính năng sau:
- Standard 802.1Q VLAN model with trunk and access ports
- NIC bonding with or without LACP on upstream switch
- NetFlow, sFlow(R), and mirroring for increased visibility
- QoS (Quality of Service) configuration, plus policing
- Geneve, GRE, GRE over IPSEC, VXLAN, and LISP tunneling
- 802.1ag connectivity fault management
- OpenFlow 1.3 plus numerous extensions
- Transactional configuration database with C and Python bindings
- High-performance forwarding using a Linux kernel module

## OVS architecture
Theo kiến trúc này, OVS có 3 thành phần chính sau đây:

- **ovs-vswitchd**, một chương trình nền thực hiện việc chuyển tiếp, ovs-vswitchd sẽ kết hợp với kennel-module để thực hiện việc chuyển tiếp dựa trên các flow. Ovs-vswicthd sử dụng giao thức Open Flow.
- **ovsdb-server**, là một server database, được ovs-vswitchd sử dụng để lưu trữ và truy vấn các cấu hình của nó. Đồng thời, người dùng cũng có thể giao tiếp với ovsdb-server thông qua giao thức ovsdb management.
- **Control cluster**, là công cụ hay các controller giao tiếp với ovsdb-server và ovs-vswitchd thông qua giao thức Open Flow và ovsdb management.
- Mục đích của kernel module là để cải thiện hiệu năng của OVS.

Sau đây, chúng ta sẽ đi tìm hiểu các thành phần trong kiến trúc của OVS. Đầu tiên là Kernel Module.
### Kernel module
OVS kernel module có một thiết kế khá đơn giản, nhằm mục đích để có thể nằm trong một số hệ thống. Nó cho phép điều khiển không gian người dùng (userspace) một cách mềm dèo thông qua xử lý các gói tin ở mức flow.

OVS kernel module được sử dụng để implement các **datapath**. Một datapath cũng giống như một switch vật lý (brigde). Mỗi datapath có thể có nhiều **vport** (tương tự như port trong các switch/brigde vật lý). Mỗi datapath cũng được liên kết với một 'flow table' mà chứa các flow.

Mỗi flow có một tập các field dùng để matching, ví dụ các trường của gói tin sẽ được xử lý hay là port được sử dụng để nhận gói tin. Mỗi flow là có một tập các hướng dẫn (instruction) để làm khi flow này phù hợp với gói tin đi vào. Tập các hướng dẫn này có thể là các hành động sẽ được thực hiện. Các hành động này có thể là xóa gói tin hoặc chuyển tiếp gói tin đến vport khác. Sau đây sẽ là một ví dụ.

Khi một gói tin đến một vport, kernel module xử lý gói tin này bằng các trích xuất các trường matching và thực hiện tìm kiếm trong bảng các flow. Nếu tìm thấy một flow phù hợp với gói tin, kernel module sẽ thực hiện các hướng dẫn và các hành động theo như flow này đã định nghĩa. Nếu không tìm thấy gói tin nào phù hợp, Kernel sẽ gửi các gói tin này đến userspace để được xử lý. Như một phần trong quá trình xử lý của nó, phần userspace của OVS là **ovs-vswitchd** sẽ thực hiện thiết lập một flow mới để xử lý các gói tin tương tự trong tương lai tại kernel.
[hình ảnh]
Nhưng chúng ta có thể thấy trong hình, các quyết định về cách xử lý một gói tin được tạo ra bởi userspace, vì vậy gói tin đầu tiên của một flow mới tạo ra sẽ đến ovs-switchd, trong khi các gói tin tiếp theo sẽ tìm được một entry cach trong kernel.
### Datapath flow
OVS sử dụng nhiều kiểu flow khác nhau cho các mục đích khác nhau. Kiểu flow chủ yếu trong OVS là OpenFlow flow. OpenFlow controller sẽ sử dụng các flow này để định nghĩa một chính sách của switch. OpenFlow flow hỗ trợ wildcard(ký tự đại diện cho biết một field sẽ được áp dụng cho tất cả gói tin khi tìm kiếm), độ ưu tiên (khi có nhiều flow phù hợp với gói tin thì sẽ sử dụng độ ưu tiên), và multiple table (để thực hiện tìm kiếm trên một chuỗi các table theo kỹ thuật pipeline). Khi điều khiển sử dụng in-band (khi truyền tải dữ liệu và điều khiển trên cùng một đường truyền), thì Open vSwitch sẽ thiết lập một số flow ẩn, với độ ưu tiên cao hơn một controller hay một user có thể làm, mà không thể thấy được thông qua OpenFlow.

OVS còn sử dụng loại flow thứ 2, được gọi là 'datapath' hay 'kernel', Nó không hỗ trợ độ ưu tiên và chỉ có duy nhất một bảng, việc này khiến cho kiểu flow phù hợp với việc caching. OpenFlow flow và datapath flow hỗ trợ các hành động khác nhau và số port khác nhau. Datapath là một implementation chi tiết mà là đối tượng để thay đổi trong các version tiếp theo của OVS. Còn với version hiện tại của OVS, thì implementation hardware switch không cần thiết sử dụng kiến trúc này.

Việc phân chia các trường dưới đây mô tả cách các datapath flow tìm kiếm các packet phù hợp. Nếu bất kỳ đoạn này trong các đoạn này bị bỏ đi trong cú pháp của flow, thì các trường này sẽ được xem là wildcard( phù hợp với tất cả gói tin): Vì vây, nếu tất cả bị đi, thì flow sẽ phù hợp với tất cả gói tin. Ký hiệu * và ANY đại diện cho wildcard.
- in_port=port_no: Trùng với port vật lý port_no. Các port của switch được đánh số, có thể xem thông qua dhcp show.
- dl=vlan: Trùng với IEEE 802.1q 

Các hành động mà kernel module có thể thực hiện khi một flow được xác định sẽ được mô tả ở phần sau.

Một chú ý cần thiết nữa là user và controller chỉ có thể điều khiển duy nhất bảng OpenFlow flow. Open vSwitch sẽ tự quản lý bảng datapath flow, vì vậy người dùng thông thường thì không nên quan tâm đến chúng.

## Các thành phần và tool của Open vSwitch
OVS có hai thành phần chính, đó là:
- ovs-vswitchd, một chương trình nền được chạy trong switch, nó sẽ kết hợp với một Linux kernel module cho việc chuyển tiếp dựa trên flow.
- ovsdb-server, một database server mà ovs-vswitchd truy vấn đến để lấy thông tin cấu hình của nó.

OVS cũng hỗ trợ một số các công cụ khác:
- ovs-dpcl, một công cụ giúp hiển thị và cấu hình các datapath flow.
- ovs-vsctl, một công cụ giúp truy vấn và cập nhật các cấu hình của ovs-switchd. ovs-dpcl sẽ điều khiểu Fast Path và ovs-vsctl sẽ điều khiển Slow Path. Thông thường, người dùng sẽ sử dụng ovs-vsctl để cấu hình, còn ovs-dpcl sẽ sử dụng cho mục đích debug.
- ovs-appctl, công cụ gửi các command đến ovs-vswitchd.
- ovs-ofctl, công cụ truy vấn và điều khiển thành phần OpenFlow trong OVS
- ovsdb-tool, một command-line tool cho việc quản lý các file trong database
- ovsdb-client, một command-line client cho việc tương tác với một process ovsdb-server đang chạy.

Chúng ta sẽ đi vào tìm hiểu 2 thành phần chính đó là ovs-vswitchd và ovsdb-server.
### ovs-vswitchd
ovs-vswitch là một chương trình nền dùng để điều khiển và quản lý tất cả các OVS switch trong local machine. Nó là một core component của hệ thống:
- Kết nối với SDN controller sử dụng OpenFLow.
- Kết nối với ovsdb-server thông qua OVSDB protocol.
- Kết nối với Linux kernel module thông qua netlink.

Cơ chế phân loại gói tin hỗ trợ hiệu quả trong việc tìm kiếm flow với ký tự đại diện và các quy tắc ký tự đại diện này cho việc xử lý nhanh chóng được sử dụng bởi datapath. Datapath sẽ lấy thông tin cấu hình của nó từ database mỗi khi khởi động. Nó thiếp lập các OVS datapath và sau đó các hoạt động switching thông qua mỗi brigde được mô tả trong file cấu hình của nó. Khi database thay đổi, ovs-vswitchd sẽ tự động cập nhật file cấu hình của nó để phù hợp với thay đổi. Ngoài ra, chương trình nền này cũng kiểm tra counter của datapath flow để xử lý các flow hết hạn và các requet stats.

Chỉ có một instance của ovs-vswitchd chạy trên local machine tại một thời điểm. Và một instance này sẽ quản lý tất cả các switch instance, nó có thể quản lý một số lượng tối đa switch mà OVS datapath có thể hỗ trợ. ovs-vswitchd thực hiện tất cả công việc quản lý cần thiết của OVS datapath. Vì vậy, các công cụ khác, chẳng hạn như ovs-dpcl, không cần quản lý datapath. Trong thực tế, việc cấu hình datapath flow với ovs-dpcl được thực hiện khi ovs-vswitchd đang chạy có thể gây trở ngạy với các hoạt động của nó.
### ovsdb-server
ovsdb-server cung cấp RPC(Remote Proceduce Call) interface đến một hoặc nhiều OVS database (OVSDBs). Nó hỗ trợ JSON-RPC client kết nối thông qua TCP/IP hoặc Unix domain socket. Các database này sẽ lưu trữ cấu hình switch, chẳng hạn như brigde, interface, địa chỉ OVSDB quản lý và OpenFlow controller.

Cấu hình này được lưu trữ trên disk và không bị mất đi khi reboot. Implementation của  databse này là dựa trên log, có nghĩa là nó không chỉ lưu trữ trạng thái của database mà còn lưu trữ một log change, nơi mà tất cả thay đổi sẽ được lưu trữ.

Server tương tác với OVSDB manager và ovs-vswitchd sử dụng OVSDB protocol, nhằm cho phép lập trình truy cập đến OVS database.

#### Database schema
Một database với cơ chế là lưu trữ cấu hình cho một OVS deamon. Ở mức cấu hình tầng cao nhất cho deamon là bảng Open_vSwitch, bảng này phải có chính xác một record. Các record trong bảng khác chỉ có ý nghĩa khi chúng được đi đến một cách trực tiếp hoặc gián tiếp thông từ bảng Open_vSwitch. Các record mà không được tiếp cận từ bảng Open_vSwitch sẽ tự động được xóa từ database, ngoại trừ các record trong một số bảng 'root set' riêng biệt.

Sau đây là danh sách các table và chức năng của chúng trong Open_vSwitch database:
- Open_vSwitch: Open vSwitch configuration.
- Brigde: Brigde configuration.
- Port: Port configuration.
- Interface: Một thiết bị mạng vật lý trong một Port.
- Flow_Table: OpenFlow table configuration.
- QoS: Quality of Service configuration.
- Queue: QoS output queue.
- Mirror: Port mirroring.
- Controller: OpenFlow controller configuration.
- Manager: OVSDB management configuration.'
- NetFlow: NetFlow configuration.
- SSL: SSL configuration.
- sFlow: sFlow configuration.
- IPFIX: IPFIX configuration.
- Flow_Sample_Conllector_Set: Flow_Sample_Conllector_Set configuration.

Tuy nhiên, chúng ta không cần phải tương tác với tất cả những bảng này, mà chúng ta có thể sử dụng một số bảng, được gọi là Core Table. Hình dưới đây sẽ cho chúng ta thấy một quan hệ giữa các core table này.
[hình ảnh 2.3]

#### ovsdb-tool
ovsdb-tool là một công cụ command-line cho việc quản lý các file của OpenvSwitch database. Nó không tương tác trực tiếp với server OpenvSwitch database đang chạy. Ví dụ, chúng ta có thể sử dụng chương trình ovsdb-tool để thấy change log của database với:
```
	ovsdb-tool show-log [-mmm] <file>
```
#### ovsdb-client
ovsdb-client là một công cụ command-line cho việc tương tác với một tiến trình ovsdb-server đang chạy. Mỗi câu lệnh kết nối đến OVSDB server, với địa chỉ mặc định là unix:/var/run/openvswitch/db.sock.
## ovs-dpctl
ovs-dpctl có thể tạo ra, thay đổi và xóa Open vSwitch datapath. Một local machine có thể có bất ký số lượng datapath. ovs-dpctl nói chuyện trực tiếp với Linux kernel module.

Một datapath được tạo ra sẽ được liên kết với chỉ duy nhất một thiết bị mạng, một thiết bị mạng ảo đôi khi được gọi là 'local port' của datapath. Nếu ovs-vswitchd được sử dụng, thì ovs-vsctl phải được sử dụng thay cho ovs-dpctl.

ovs-dpctl có thể viết datapath flow đến kernel module, sử dụng các trường được mô tả trong phần datapath ở trên để xây dưng các flow mong muốn. Để thực hiện được việc tạo ra datapath flow, ovs-dpctl hỗ trợ câu lệnh *add-flow* và *add-flows* với một số trường *action=target[,target...]*

Các trường này xác định một danh sách các hành động được thực hiện trên một gói tin khi flow này phù hợp với gói tin đó. Target này có thể là port để chỉ định port vật lý cho đầu ra của gói tin, hoặc là một trong số danh sách sau đây:
[hình ảnh]
### ovs-vsctl
ovs-vsctl hỗ trợ cấu hình ovs-vswitchd bằng cách cung cấp một interface đến configuration của nó trong database.

ovs-vsctl kết nối đến một process ovsdb-server mà đang chứa một database configuration của Open vSwitch. Sử dụng kết nối này, nó truy vấn và có thể thực hiện thay đổi đến database, phụ thuộc vào câu lệnh được thực hiện. Sau đó, nếu nó đã thực hiện mọi thay đổi, một cách mặc định là nó sẽ chờ ovs-vswitchd hoàn thành việc cấu hình lại chính nó trước khi kết nối đó kết thúc. ovs-vsctl có thể thực hiện bất kỳ số lượng câu lệnh trong một lần chạy, được thực hiện như là một giao dịch đơn đến cơ sở dữ liệu.

Dưới đây là một số câu lệnh được hỗ trợ bởi ovs-vsctl:
- ovs-vsctl add-br <brigde>
- ovs-vsctl list-br
- ovs-vsctl add-port <brigde> <port>
- ovs-vsctl list-ports <brigde>
- ovs-vsctl get-manager <brigde>
- ovs-vsctl get-controller <brigde>
- ovs-vsctl list <table>

### ovs-appctl
