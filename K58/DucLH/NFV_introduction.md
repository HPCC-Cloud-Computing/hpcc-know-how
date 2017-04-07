#Network Function Virtualization - NFV
## Giới thiệu
Network functions virtualization (NFV) cung cấp một giải pháp để thiết kế, triển khai và quản lý các dịch vụ mạng (network service). NFV là giải pháp bổ sung đến Software-Defined Networking (NDS) trong việc quản lý mạng. Cả NFV và SDN đều chung một mục đích là quản lý mạng, nhưng theo các cách khác nhau. Trong khi SDN tách biệt control plane và forwarding plane để cung cấp một khung nhìn tổng thể của mạng, thì NFV lại chủ yếu tập trung vào việc tối ưu hóa các dịch vụ mạng (network services).
NFV được tạo ra nhằm mục đích giải quyết các thách thức về vận hành và chi phí cao trong việc quản lý các thiết bị độc quyền và đóng (cách thức triển khai, sử dụng phụ thuộc vào các nhà cung cấp phần cứng riêng biệt) đang được triển khai trong mạng viễn thông hiện nay. Bằng việc ảo hóa và hợp nhất (virtualizing and consolidating) các dịch vụ mạng đang được triển khai trên các thiết bị phần cứng chuyên dụng, sử dụng các công nghệ cloud, các nhà cung cấp dịch vụ mạng hy vọng sẽ đạt được sự linh hoạt và nhanh chóng trong việc triển khai dịch vụ mới, trong khi làm giảm chi phí triển khai (capital costs - CapEx) và chi phí vận hành (operational costs - OpEx).
Sau khi hiểu được khái niệm về NFV thì chúng ta sẽ đi qua về kiến trúc của NFV.
## Kiến trúc NFV
![](https://camo.githubusercontent.com/d133b7674908bd5716f408de55d02e5145aae06a/68747470733a2f2f7777772e73647863656e7472616c2e636f6d2f77702d636f6e74656e742f75706c6f6164732f323031352f30342f6e66762d7265706f72742d323031352d686967682d6c6576656c2d6e66762d6672616d65776f726b2e706e67)
			
 **Hình 1: Kiến trúc NFV**
            
Kiến trúc của NFV có 3 thành phần chính:
- **Network Functional vitualization (VFNs)** - Các chức năng mạng hay thiết bị mạng ảo triển khai dưới dạng phần mềm trên hạ tầng NFV.
- **NFV Infrastructure (NFVI)** - Là các tài nguyên vật lý và các công nghệ ảo hóa được sử dụng để cung cấp tài nguyên cho việc triển khai VNFs
- **NFV Management and Orchestration** - Thực hiện các chức năng điều phối, quản lý các tài nguyên phần cứng và các tài nguyên phần mềm hỗ trợ cho quá trình ảo hóa, đồng thời quản lý các VNFs


Quá trình tạo ra một dịch vụ mạng được ảo hóa cũng tương tự như quá trình tạo ra một máy ảo. Đó là việc sử dụng công nghệ ảo hóa tài nguyên vật lý thành các tài nguyên ảo hóa như vitual compute, virtual storage, virtual network. Sau đó, việc tạo ra VNF là sử dụng các tài nguyên ảo hóa này với các dịch vụ cụ thể.

Như trong phần giới thiệu đã đề cập, NFV và SDN cùng chung mục đích là quản lý mạng, nhưng vẫn đảm bảo giảm CapEx và OpEx. Dưới đây là thể hiện rằng có rất nhiều tổ chức, doanh nghiệp đang nỗ lực xây dựng các thành phần trong một hệ sinh thái NFV cởi mở và linh động.

![](https://www.opennetworking.org/images/stories/sdn-resources/sdn-nvf-solution/figure2.png)


**Hình 2: Mô hình NFV và SDN trong trong thực tế**

## Lợi ích của NFV
- Giảm chi phí triển khai (Capital Costs)
- Giảm chi phí vận hành, bảo trì
- Tăng tốc độ đưa dịch vụ mới vào thương mại
- Cung cấp nhanh chóng và linh hoạt.

## Các trường hợp sử dụng của NFV/SDN
Dưới đây là hai trường hợp sử dụng được định nghĩa trong tài liệu **ETSI NFV ISG Use Cases**. Cả hai trường hợp sử dụng này sẽ được hưởng lợi từ các đặc điểm của SDN hỗ trợ OpenFlow, đặc biệt là tính scale.
##### VIRTUAL NETWORK FUNCTION FORWARDING GRAPH
Một trong các trường hợp sử dụng của NFV là Virtual network function forwarding graph - đồ thị chuyển tiếp chức năng mạng ảo, cho phép các thiết bị mạng ảo được xâu chuỗi cùng nhau trong một cách linh hoạt.
Cụ thể trong trường hợp sử dụng này, một ứng dụng, một luồng dữ liệu phải được truyền qua một số các thiết bị mạng ảo trước khi được chuyển giao. Quá trình này được gọi như là Service Chaining (Kết nối dịch vụ). Trong hình 3 mô tả, một luồng dữ liệu được bắt nguồn từ điểm A truyền qua một thiết bị quản lý mạng (network monitoring VNF), một thiết bị cân bằng tải (load balancing VNF), và cuối cùng là một thiết bị firewall (firewall VNF) trước khi đến điểm đích là B.
![](http://103.56.157.246/wp-content/uploads/2015/03/service_chaining.png?w=300)

**Hình 3: Virtual network function forwarding graph**

Các giải pháp dựa trên phần cứng ngày nay sẽ rất khó khăn và tốn rất nhiều thời gian trong việc thực hiện xâu chuỗi các dịch vụ, bên cạnh đó còn tốn rất nhiều chi phí cho việc mở rộng và quản lý. Các thiết bị vật lý phải được cài đặt và được kết nối, sau đó được gán đến một mạng con chẳng hạn như VLAN, sub-network,... các mạng nào thường phải được giới hạn kết nối để đảm bảo tính bảo mật của dịch vụ. Việc cấu hình phải thực hiện bằng tay và rất tỉ mỉ để thực hiện việc xâu chuỗi dịch vụ.
Trong môi trường NFV, một VNF có thể được tạo ra, được update, được mở rộng và xóa bỏ một cách nhanh chóng và hiệu quả. Một ví dụ, để thêm một VNF đến một hoặc một chuỗi dịch vụ, một máy ảo sẽ được khởi tạo và sẽ được cập nhật đồ thị chuyển tiếp (forwarding graph). Việc scale cũng tương tự.
##### NFV INFRASTRUCTURE AS A SERVICE (NFVIAAS)
Đối với trường hợp sử dụng này, một nhà cung cấp dịch vụ có thể cung cấp các dịch vu sử dụng cơ sở hạ tâng NFV của nhà cung cấp dịch vụ khác. Giải pháp này có thể mở rộng phạm vi của một nhà cung cấp dịch vụ tại những vị trí họ không triển khai được cơ sở hạ tầng NFV.
Hình 4 mô tả khái niệm của NFVIaaS. Một ví dụ để hiểu rõ hơn trường hợp sử dụng này, nhà cung cấp dịch vụ X cung cấp dịch vụ load balance ảo hóa. Một số khách hàng của X các dịch vụ load balance tại những nơi mà nhà cung cấp không triển khai cơ sở hạ tầng NFV, nhưng tại nơi đó lại có một nhà cung cấp dịch vụ Z khác triển khai cơ sở hạ tầng NFV.

![](http://103.56.157.246/wp-content/uploads/2015/03/nfviaas.png?w=300)

**Hình 4: NFV Infrastructure as a service**

NFVIaaS cung cấp một phương tiện cho nhà cung cấp Z cho thuê cơ sở hạ tâng NFV (ví dụ như compute, network, hypervisor,...) để cung cấp cho nhà cung cấp X, và thông qua NFV infrastructor của Z sẽ cung cấp dịch vụ cho khách hàng. Việc cho thuê NFV infrastructure sẽ tăng khả năng cung cấp dịch vụ có sẵn và có thể mở rộng khi cần thiết.
