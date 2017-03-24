Linux Bridge
========
Bài viết note lại các kiến thức tìm hiểu được về Linux-bridge

#Linux Bridge – The Basics
Linux bridge là một phần mềm đươc tích hợp vào trong nhân Linux để giải quyết vấn đề ảo hóa phần network trong các máy vật lý. Mặc dù được gọi là bridge, nhưng thực chất linux-bridge tạo ra một switch ảo, sử dụng với ảo hóa KVM/QEMU để các VMs có thể kết nối được với nhau cũng như kết nối được ra bên ngoài.

Một linux-bridge có thể được tạo, xóa và quản lí nhờ command line tool được gọi là **brctl**. Để sử dụng **brctl** trên Ubuntu hoặc Debian , chúng ta cần cài package sau: 

	$ sudo apt-get install bridge-utils

Chúng ta sẽ tìm hiểu một số command line cơ bản sau:

Để tạo một bridge br0:

	$ sudo brctl addbr br0

Để remove bridge br0:

	$ sudo brctl delbr br0
	
Để thêm interface eth0 và eth1 tới bridge br0:

	$ sudo brctl addif br0 eth0
	$ sudo brctl addif br0 eth1
	
Để remove interface eth0 xuống bridge br0:

	$sudo brctl delif br0 eth0

#The Simple Use Case
Như chúng ta đã biết, khi tạo một máy ảo mới, có nhiều options cấu hình network cho máy ảo. Một trong hai options phổ biến được sử dụng đó là `bridge networking` và `network address translation (NAT)`. Vậy sự khác nhau của 2 options này là gì?

![](https://github.com/vanduc95/OpenStack_Network/blob/master/img/bridge_vs_NAT.png) 

Nhìn vào hình trên, đường thẳng cạnh "bức tường lửa" đại diện cho external network (mạng bên ngoài). Vòng tròn màu đỏ đại diện cho switch ảo sử dụng NAT. Khi sử dụng NAT, các VMs sẽ không được cấp địa chỉ của external network, thay vào đó nó sẽ được cấp một private ip từ DHCP server. Các VMs có thể kết nối được với các host trên external network nhưng ngược lại các host trên external lại không thể tạo kết nối với các VMs được. 

Để giải quyết nhược điểm này, chúng ta có thể sử dụng **linux-bridge**.

Bây giờ chúng ta sẽ tìm hiểu sâu hơn một chút về linux bridge bằng cách nhìn vào một use case cơ bản sau:  Chúng ta sẽ tạo một VM trên một host sử dụng KVM. Máy ảo này sẽ được cấu hình với một card mạng ảo (vNIC). Để VM có thể giao tiếp được với các host trên external network và ngược lại, chúng ta sẽ phải nối vNIC của VM với card mạng vật lí eth0 của host. Việc này sẽ được hỗ trợ bởi linux bridge. Hình dưới là mô hình mà chúng ta cần đạt được.

![](https://github.com/vanduc95/OpenStack_Network/blob/master/img/Linux-Bridge-Simple-UseCase.png) 

##Step-by-step guide
**Important note**: Wireless interface không thể được gắn vào một Linux host bridge, vì vậy nếu máy của bạn kết nối với external network thông qua wireless interface (wlan0), thì không thể tạo được linux bridge. Chúng ta cần sử card mạng vật lí (trong trường hợp này là eth0)

Bước 1: Tạo một linux bridge có tên là **br0**

	$ sudo brctl addbr br0

Bước 2: Gán card mạng vật lí của host (eth0) tới bridge **br0**. **Note:** Trước khi thực hiện bước này, hãy đảm bảo card mạng vật lí không có bất cứ địa chỉ IP nào được cấu hình.


	$ ifconfig eth0 0.0.0.0
	$ sudo brctl addif br0 eth0
	
Bước 3: Comment card mạng **eth0** trong file `/etc/network/interface` nếu có:

![](https://camo.githubusercontent.com/c2ec80f423ce391e1ec1af40e077575408340359/687474703a2f2f692e696d6775722e636f6d2f7a4534703271682e706e67) 

Bước 4: Khi tạo ra một bridge sử dụng **brctl**, nó sẽ tự động hủy khi khởi động lại máy.  Do vậy, để bridge không bị mất khi khởi động, ta cần cấu hình bridge trong file `/etc/network/interface` như sau:

	auto br0
	iface br0 inet dhcp
	bridge_ports eth0
	bridge_stp off # tat che do STP trong bridge
	bridge_fd 0 
	bridge_maxwait 0

**Lưu ý: **`bridge_stp on` có thể gây ra các vấn đề truyền thông tin DHCP tới máy khách. Trong hướng dẫn này, chúng ta sẽ tắt chế độ STP này.

Bước 5: Khởi động lại mạng

	$ ifdown -a && ifup -a
	
Sau bước này, cấu hình network sẽ như hình dưới:

![](https://github.com/vanduc95/OpenStack_Network/blob/master/img/ifconfig.png) 

![](https://github.com/vanduc95/OpenStack_Network/blob/master/img/brctl_show.png) 

Bước 6: Cuối cùng chúng ta tạo máy ảo. Trong phần cấu hình mạng chúng ta chọn bridge **br0**

![](https://github.com/vanduc95/OpenStack_Network/blob/master/img/create_VM.png) 

Sau khi tạo xong máy ảo, vNIC của máy ảo sẽ nhận IP của dải eth0. Và đây là kết quả:

![](https://github.com/vanduc95/OpenStack_Network/blob/master/img/result_1.png) 

Ta kiểm tra ping từ máy ảo ra ngoài và từ ngoài vào máy ảo. Kết quả trả về thành công

![](https://github.com/vanduc95/OpenStack_Network/blob/master/img/result_2.png) 

Cuối cùng nếu muốn xóa linux-bridge **br0** chúng ta sử dụng command line:

	$ ip link set br0 down
	$ brctl delbr br0

# Tài liệu tham khảo
[1] http://www.innervoice.in/blogs/2013/12/02/linux-bridge-virtual-networking/

[2] http://techgenix.com/nat-vs-bridged-network-a-simple-diagram-178/

[3] http://www.dedoimedo.com/computers/kvm-bridged.html

[4] https://www.vmware.com/support/ws55/doc/ws_net_configurations_nat.html

[5] https://wiki.linuxfoundation.org/networking/bridge

[6] http://xmodulo.com/how-to-configure-linux-bridge-interface.html





