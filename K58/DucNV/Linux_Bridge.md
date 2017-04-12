Linux Bridge
========
Bài viết note lại các kiến thức tìm hiểu được về Linux-bridge

# Linux Bridge – The Basics

Linux bridge là một phần mềm được tích hợp vào trong nhân Linux để giải quyết vấn đề ảo hóa phần network trong các máy vật lý. Mặc dù được gọi là bridge, nhưng thực chất linux-bridge tạo ra một switch ảo, sử dụng với ảo hóa KVM/QEMU để các VMs có thể kết nối được với nhau cũng như kết nối được ra bên ngoài.

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

# The Simple Use Case

Bây giờ chúng ta sẽ tìm hiểu sâu hơn một chút về linux bridge bằng cách nhìn vào một use case cơ bản sau:  Chúng ta sẽ tạo một VM trên một host sử dụng KVM. Máy ảo này sẽ được cấu hình với một card mạng ảo (vNIC). Để VM có thể giao tiếp được với các host trên external network và ngược lại, chúng ta sẽ phải nối vNIC của VM với card mạng vật lí eth0 của host thông qua Linux bridge. Hình dưới đây là mô hình mà chúng ta cần đạt được.

![Imgur](http://i.imgur.com/pntn9Rm.png)

## Step-by-step guide
**Important note**: Về cơ bản, nếu máy của bạn kết nối với external network thông qua wireless interface (wlan0), thì không thể tạo được linux bridge. Để có thể tạo linux bridge với wireless interface, chúng ta cần phải cấu hình thêm một số bước. Do vậy, để đơn giản, trong hướng dẫn này chúng ta sẽ sử dụng card mạng vật lí (eth0).

Các bước trong hướng dẫn sử dụng với quyền **root**

Bước 1: Tạo một linux bridge có tên là `br0`

	# brctl addbr br0

Bước 2: Gán card mạng vật lí của host (eth0) tới bridge `br0`.  **Note:** Trước khi thực hiện bước này, hãy đảm bảo card mạng vật lí không có bất cứ địa chỉ IP nào được cấu hình.


	# ifconfig eth0 0.0.0.0
	# brctl addif br0 eth0

Bước 3:  Cấp phát địa chỉ IP cho br0 sử dụng `dhclient`

	# dhclient br0
		
Sau bước này, chúng ta đã tạo thành công linux bridge, cấu hình network sẽ như hình dưới:

![Imgur](http://i.imgur.com/rAR11XI.png)

![Imgur](http://i.imgur.com/equlXKs.png) 

**Lưu ý:** Bridge `br0` được tạo ở trên chỉ được cấu hình tạm thời và sẽ mất khi khởi động lại máy. Nếu bạn muốn tự động tạo bridge khi máy khởi động, ta cần cấu hình bridge trong file `/etc/network/interface` như sau`
		
	# Comment 2 dòng dưới nếu có trong file	
	# auto eth0
	# iface eth0 inet dhcp
	
	auto br0
	iface br0 inet dhcp
	bridge_ports eth0
	bridge_stp off # tat che do STP trong bridge
	bridge_fd 0 
	bridge_maxwait 0

Bước 4: Sau khi tạo xong bridge, chúng ta đi đến phần tạo máy ảo. Trong phần cấu hình mạng chúng ta chọn bridge **br0**

![Imgur](http://i.imgur.com/62X0JRw.png)

Sau khi tạo xong máy ảo, vNIC của máy ảo sẽ nhận IP của dải eth0. Và đây là kết quả:

![Imgur](http://i.imgur.com/m5uOjNb.png) 

Ta kiểm tra ping từ máy ảo ra ngoài và từ ngoài vào máy ảo. Kết quả trả về thành công

![Imgur](http://i.imgur.com/vNkqf4k.png)

Cuối cùng nếu muốn xóa linux-bridge `br0` chúng ta sử dụng command line:

	$ ip link set br0 down
	$ brctl delbr br0

# Lab

Phần trên chúng ta đã được hướng dẫn tạo một birdge chỉ gắn với một card mạng vật lí `eth0`. Trong phần này, chúng ta sẽ tạo ra một bridge `br0` được gắn tới cả 2 card mạng `eth0` và `eth1` và làm thế nào để khi tạo máy ảo với bridge `br0` ta có thể quyết định xem máy ảo sẽ nhận địa chỉ IP của `eth0` hoặc `eth1`.

Mô hình lab: `eth0: 192.168.122.0/24`, `eth1: 192.168.2.0/24`

**Các bước thực hiện:**

 - Cấu hình tạo bridge trên file `/etc/network/interface` , comment cấu hình 2 card mạng `eth0` và `eth1` như sau:

		    # The primary network interface
    		#auto eth0
    		#iface eth0 inet static
    		
    		#auto eth1
    		#iface eth1 inet dhcp
    		
    		auto br0
    		iface br0 inet dhcp
	    		bridge_ports eth0 eth1
	    		bridge_stp off
	    		bridge_fd 0
	    		bridge_maxwait 0

 - Khởi động lại card mạng:
 
	     # ifdown -a & ifup -a

 - Kiểm  tra 2 card mạng đã được gắn vào `br0` hay chưa?
 
 ![Imgur](http://i.imgur.com/aWI13hO.png)

Sau khi đã tạo xong bridge, chúng ta có thế bắt đầu tạo máy ảo. Điều cần nói ở đây là khi tạo máy ảo với bridge `br0`, nếu chúng ta muốn gán máy ảo với địa chỉ IP của dải mạng `eth1` mà không phải `eth0` thì phải làm thế nào? Ta lưu tâm đến một tham số là **priority** cho port. Có nghĩa là port nào có chỉ số **priority** cao hơn thì khi máy ảo gắn bridge đó vào thì nó sẽ nhận được IP của port có priority cao hơn.

Câu lệnh để gán thông số priority cho port:

    # brctl setportprio <tên bridge> <tên port> <chỉ số priority>

Trong ví dụ này ta sẽ thiết lập **priority** của port cắm `eth1` cao hơn port cắm vào `eth0`.

    # brctl setportprio br0 eth0 10
    # brctl setportprio br0 eth1 20

Khi đó, khi khởi động máy ảo, nó sẽ nhận dải IP của `eth1:192.168.2.0/24` , cụ thể như hình sau:

![Imgur](http://i.imgur.com/dEydLfw.png)

# Kết luận

Trên đây là những hướng dẫn cơ bản nhất để tạo ra và sử dụng một Linux bridge. Ở bài viết sau chúng ta sẽ tìm hiểu thêm về giao thức STP (Spanning Tree Protocol) và cách cấu hình cũng như cách chuyển tiếp hay loại bỏ các gói tin trên Linux bridge.

# Tài liệu tham khảo
[1] http://www.innervoice.in/blogs/2013/12/02/linux-bridge-virtual-networking/

[2] http://www.dedoimedo.com/computers/kvm-bridged.html

[3] https://wiki.linuxfoundation.org/networking/bridge

[4] http://xmodulo.com/how-to-configure-linux-bridge-interface.html


