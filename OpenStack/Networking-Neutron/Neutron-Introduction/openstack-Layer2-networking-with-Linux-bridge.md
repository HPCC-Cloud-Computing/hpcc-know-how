Ở bài viết trước chúng ta đã tìm hiểu về các loại mạng nội bộ tiêu biểu có thể sử dụng trong thực tế. Ở bài viết này,chúng ta cùng tìm hiểu xem cách thức các loại mạng này triển khai trên OpenStack như thế nào khi sử dụng Linux Bridge plugin
#Linux bridge và triển khai của các loại mạng sử dụng Linux Bridge plugin trong OpenStack
##1 Giới thiệu về Linux Bridge plugin
Như đã giới thiệu ở phần trước, Linux Bridge là một loại switch ảo được thiết kế theo chuẩn 802.1D được sử dụng trong hệ điều hành Linux. Tuy nhiên, với các chức năng của 1 switch cơ bản, Một mình Linux brige là không đủ khả năng để xây dựng nên các mạng phức tạp như VLAN, VXLAN. Do đó, khi xây dựng hệ thống mạng trong OpenStack sử dụng Linux Bridge plugin, Neutron sẽ kết hợp các Linux bridge với các loại thiết bị ảo khác và cả các thiết bị phần cứng để tạo nên hệ thống mạng ảo phục vụ cho các máy ảo. Ta sẽ xem xem các thiết bị này được xây dựng như thế nào trong các loại mạng.
##2 Triển khai các loại mạng cục bộ với Linux Bridge plugin
###2.1 Mạng Flat
Chúng ta đã biết, trong mạng Flat, các gói tin được các switch chuyển tiếp mà không được gắn thêm các thông tin định danh. Chính vì lý do đó, mà trong môi trường OpenStack, mỗi một card mạng vật lý chỉ có thể triển khai được một mạng Flat. 

Khi triển khai mạng Flat, sẽ có một bridge ảo được tạo ra trên các node và nối trực tiếp tới card mạng vật lý. Các máy ảo sử dụng mạng Flat đó, mỗi máy sẽ nối 1 card mạng ảo vào bridge ảo này để kết nối với mạng Flat. Lưu ý, mạng Flat sẽ được triển khai các card mạng cùng thuộc một mạng vật lý. Tức là nếu ở máy A mạng flat X được triển khai trên card mạng A1, ở máy B mạng flat X được triển khai trên card mạng B1 thì A1 và B1 sẽ kết nối tới cùng một mạng vật lý, và các card mạng vật lý này không cần phải có địa chỉ IP
![LinuxBridge-Flat.png](./img/LinuxBridge-Flat.png)

Kiểm tra trên compute node sau khi kết nối 3 máy ảo vào cùng 1 mạng Flat:
```sh
bridge name	bridge id					STP enabled		interfaces
brqdf517cb8-c4		8000.000c299ddd0d	no				eth1
														tap2ad6ce6e-84
														tap41e05531-0f
														tap5ba48d69-2b
```
Trong đó ```brqxxxx``` là bridge ảo được tạo ra, bridge ảo trên nối trực tiếp đến card mạng vật lý eth1, và nối với 3 máy ảo thông qua 3 tap device ```tapxxxx```

Nếu muốn triển khai nhiều hơn một mạng Flat trên hệ thống, thì trên mỗi máy phải có số lượng card vật lý phục vụ cho các mạng Flat bằng số lượng mạng Flat, vì mỗi một card mạng vật lý chỉ có thể triển khai 1 mạng Flat.
###2.2 Mạng VLAN
Để triển khai mạng VLAN bằng Linux Bridge plugin, Linux Bridge sẽ kết hợp với thiết bị VLAN-interface ảo có tên là ethx.VLAN_ID,trong đó VLAN_ID là ID của mạng VLAN. Mỗi một mạng VLAN sẽ tạo ra một Linux bridge và một VLAN-Device. Các máy ảo thuộc cùng một mạng VLAN trên cùng một node sẽ kết nối card mạng ảo của máy ảo đó với LinuxBridge quản lý VLAN đó
![LinuxBridge-VLAN.png](./img/LinuxBridge-VLAN.png)
Linux bridge sẽ kết hợp với VLAN-Device để thực hiện được chức năng của 1 switch vật lý trong mạng VLAN, cụ thể như sau:

- Khi cần chuyển tiếp một gói tin L2 Frame từ một máy ảo qua một máy khác cùng thuộc VLAN nhưng thuộc node khác, Linux bridge sẽ chuyển tiếp gói tin đó qua VLAN-Device. VLAN-Device sẽ đóng thêm VLAN Header với ID là ID của mạng VLAN rồi chuyển tiếp ra mạng vật lý qua card vật lý eth1

- Khi tiếp nhận một gói tin L2 Frame từ mạng vật lý bên ngoài, gói tin này sẽ đi qua card eth1 vào các VLAN-Device. Các VLAN-Device sẽ kiểm tra VLAN ID của gói tin này, nếu gói tin này có VLAN ID hợp lệ, VLAN-Device sẽ loại bỏ phần VLAN header và chuyển L2 Frame tới Linux Bridge. Linux Bridge sẽ kiểm tra địa chỉ MAC đích rồi forward gói tin L2 Frame tới cổng kết nối với máy có MAC đích.

- Khi chuyển tiếp giữa các máy VLAN cùng nằm trong 1 node, Linux bridge hoạt động như một switch bình thường (không cần các hoạt động gắn-bỏ VLAN Header)
Kiểm tra bridge trên compute node sau khi kết nối 2 máy ảo vào cùng 1 mạng VLAN có id là 101:
```sh
root@compute:/home/cong# brctl show
bridge name			bridge id				STP enabled			interfaces
brq759532d6-47		8000.000c299ddd0d			no					eth1.101
																	tapb017d852-b1
																	tapbf90271c-cc

```
Lưu ý là tuy chỉ có thể triển khai một mạng Flat trên một card mạng vật lý, tuy nhiên nếu sau khi triển khai mạng Flat đó rồi, thì trên card mạng vật lý đó ta vẫn có thể triển khai thêm các mạng VLAN. 
###2.3 Mạng VXLAN
Khi triển khai mạng VXLAN bằng Linux Bridge plugin, các linux bridge sẽ kết hợp với VXLAN-Device để thực hiện các chức năng của một thiết bị VTEP. VXLAN-Device được đặt tên tương ứng với ID của mạng VXLAN mà nó được triển khai lên.
```sh
root@compute:/home/cong# brctl show
bridge name			bridge id				STP enabled		interfaces
brq06d9fc97-f8			8000.52480fb859f8		no				tap3cf57cad-23
																vxlan-57
```
Phương thức hoạt động của tổ hợp linux bridge + VXLAN-Device như sau:
- Khi cần chuyển tiếp một gói tin L2 Frame từ một máy ảo tới một máy ảo khác thuộc cùng mạng nội bộ nhưng nằm trên node khác, Linux bridge chuyển tiếp gói tin này tới VXLAN-Device. VXLAN-Device sẽ thực hiện công việc thêm VXLAN header với ID là ID của mạng VXLAN, rồiđóng gói gói tin vào UDP +IP packet. Cuối cùng gói tin IP packet này được vận chuyển với IP nguồn là IP của card vật lý mà mạng VXLAN được triển khai trên đó, IP đích là IP của card mạng vật lý triển khai VXLAN đó của node chứa máy ảo đích.

- Khi tiếp nhận một gói tin IP packet từ mạng bên ngoài gửi vào, gói tin sẽ đi qua các card mạng vật lý và vào các VXLAN-Device. VXLAN-Device sẽ mở gói và kiểm tra VXLAN ID của gói tin L2 Frame. Nếu gói tin có VXLAN ID hợp lệ với mạng VXLAN này, VXLAN-Device sẽ chuyển tiếp gói tin L2 Frame nguyên bản tới Linux-bridge. Ở đây dựa theo dữ liệu mà gói tin được chuyển tiếp tới cổng kết nối với máy có MAC đích.

Trong trường hợp chuyển tiếp giữa các máy cùng mạng nội bộ và cùng nằm trên một node, gói tin được chuyển tiếp thông qua Linux Bridge.

Trên một địa chỉ IP của một card mạng vật lý có thể triển khai nhiều mạng VXLAN. Khi cấu hình hệ thống mạng của OpenStack, chúng ta phải xác định địa chỉ IP cho card mạng vật lý sẽ triển khai các mạng VXLAN và sử dụng địa chỉ IP này để cấu hình mạng. Điều này là cần thiết, vì mạng VXLAN sử dụng giao thức IP để chuyển tiếp dữ liệu giữa các node.

Linux-bridge plugin hiện thời không hỗ trợ mạng GRE.
