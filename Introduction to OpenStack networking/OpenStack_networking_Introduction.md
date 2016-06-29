#Giới thiệu về các networking trong OpenStack
OpenStack Networking có vai trò quản lý các dịch vụ về hệ thống mạng trong môi trường OpenStack. Khi xây dựng nên hệ thống OpenStack, chúng ta quan tâm nhiều tới Việc triển khai hạ tầng mạng trên Layer2 và Layer3. Trong đó, ở Layer 2, thì vấn đề quan trọng nhất mà chúng ta quan tâm là xây dựng được các mạng cục bộ (Local Area Network -LAN). Có nhiều loại mạng cục bộ khác nhau có thể triển khai đồng thời trong OpenStack: VLAN, VXLAN, GRE, Local, Flat. 

Tuy cách thức hoạt động và khả năng của từng loại mạng là khác nhau, tuy nhiên về cơ bản thì các mạng cục bộ đều bao gồm các thành phần là các switch, dây dẫn và các máy khách kết nối vào mạng. Trong mạng cục bộ, thì switch là thành phần trung tâm, là thiết bị mà các máy khách kết nối đến, và kết nối với các switch khác để tạo thành một mạng lưới hoàn chỉnh. Vì vậy, khi xây dựng môi trường mạng ảo trong OpenStack, một điểm mà chúng ta cần chú trọng là việc lựa chọn công nghệ để xây dựng các switch ảo và lên kế hoạch cấu hình các switch ảo đó trong môi trường vật lý. Hiện tại OpenStack hỗ trợ rất nhiều plugin ảo hóa switch, tuy nhiên do thời gian tìm hiểu có hạn, trong bài viết này chúng ta sẽ tìm hiểu 2 plugin phổ biến là Linux-bridge và OpenVSwitch.

Nội dung bài viết gồm có 3 phần:

Phần 1: Giới thiệu về mạng cục bộ, các thiết bị trong mạng cục bộ và các loại mạng cục bộ

Phần 2: Giới thiệu về Linux bridge và cách xây dựng các mạng cục bộ trong OpenStack sử dụng Linux bridge

Phần 3: Giới thiệu về Open vSwitch và cách xây dựng các mạng cục bộ trong OpenStack sử dụng Linux bridge 