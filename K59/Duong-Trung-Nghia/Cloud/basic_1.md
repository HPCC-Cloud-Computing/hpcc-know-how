# Ảo hoá (`Virtualization`)

* Ảo hoá là gì ?
    * Ảo hóa là công nghệ được thiết kế để tạo ra tầng trung gian giữa hệ thống phần cứng máy tính và phần mềm chạy trên nó. Ý tưởng của công nghệ ảo hóa máy chủ là từ một máy vật lý đơn lẻ có thể tạo thành nhiều máy ảo độc lập. Mỗi một máy ảo đều có một thiết lập nguồn hệ thống riêng rẽ, hệ điều hành riêng và các ứng dụng riêng.
    * Ảo hóa có nguồn gốc từ việc phân chia ổ đĩa, chúng phân chia một máy chủ thực thành nhiều máy chủ logic. Một khi máy chủ thực được chia, mỗi máy chủ logic có thể chạy một hệ điều hành và các ứng dụng độc lập. 
    * Vào những năm 1990, ảo hóa được chủ yếu sử dụng để tái tạo lại môi trường người dùng trực tiếp trên một phần của phần cứng máy lớn. 
* Máy ảo là gì?

* Lợi ích của ảo hoá
    * Giảm thiểu chi phí: Ảo hóa được coi là một công nghệ giúp các doanh nghiệp cắt giảm chi tiêu hiệu quả với khả năng tận dụng tối đa năng suất của các thiết bị phần cứng -> tiết kiệm không gian sử dụng, nguồn điện và giải pháp tỏa nhiệt trong trung tâm dữ liệu
    * Giảm thời gian thiết lập máy chủ, kiểm tra phần mềm trước khi đưa vào hoạt động
* Nhược điểm của ảo hoá:
    * Giải pháp ảo hóa có điểm nút sự cố (single point of failure): Khi một máy, mà trên đó, mọi giải pháp ảo hóa đang chạy, gặp sự cố hay khi chính giải pháp ảo hóa gặp sự cố, sẽ làm crash mọi thứ
    * Ảo hóa yêu cầu phần cứng mạnh: Nếu cỗ máy được sử dụng không đủ mạnh, vẫn có thể triển khai các giải pháp ảo hóa nhưng khi mà không có đủ sức mạnh CPU và RAM cho chúng, nó sẽ thực sự làm gián đoạn công việc.
    * Ảo hóa có thể dẫn đến hiệu năng thấp: Một ứng dụng có thể không có vấn đề gì khi chạy không ảo hoá, nhưng khi được triển khai trong môi trường ảo hoá, các vấn đề bắt đầu nổi lên.
    * Ứng dụng ảo hóa không phải luôn luôn khả dụng:  Cơ sở dữ liệu là một trong những ví dụ phổ biến nhất của những ứng dụng như thế. Cơ sở dữ liệu đòi hỏi thường xuyên vận hành đĩa và khi có sự chậm trễ trong việc đọc ghi từ đĩa cứng bởi ảo hóa, điều này có thể làm hoàn trả lại toàn bộ ứng dụng vô ích. Thường, không ai nói rằng có thể ảo hóa một ứng dụng cơ sở dữ liệu – thậm chí một ứng dụng tài chính theo thời gian thực cũng chạy thành công trên môi trường ảo hóa.
    * Rủi ro lỗi vật lý cao: Vd. Bạn có thể chạy 5 server ảo trên 1 server vật lý, nhưng chỉ bởi 1 lỗi nào đó trong phần cứng của server vật lý có thể dẫn đến xung đột của 5 con server này. 
* Mục tiêu của ảo hóa:  
    Ảo hóa xoay quanh 4 mục tiêu chính : **Availability**, **Scalability**, **Optimization** và **Management**  
    * **Availability:** các ứng dụng vẫn có khả năng hoạt động liên tục khi phần cứng gặp sự cố, khi nâng cấp hoặc di chuyển
    * **Scalability:**  khả năng tùy biến, thu hẹp hay mở rộng mô hình server dễ dàng mà không làm gián đoạn ứng dụng
    * **Optimization:** sử dụng triệt để nguồn tài nguyên phần cứng và tránh lãng phí bằng  cách giảm số lượng thiết bị vật lý cần thiết
    * **Management:** khả năng quản lý tập trung, giúp việc quản lý trở nên dễ dàng hơn bao giờ hết

* Các loại ảo hóa
1. Ảo hóa phần cứng  
    Ảo hóa phần cứng hay ảo hóa nền tảng hướng tới việc tạo ra một máy ảo có thể hoạt động như một máy tính thực với một hệ điều hành. Vd: một máy tính đang chạy Mac OS có thể chứa một máy ảo trông như một máy thực với hệ điều hành Ubuntu Linux.
    * Một số thuật ngữ trong ảo hóa phần cứng:
        * `Host machine`: Cỗ máy thật mà trên đó ảo hóa diễn ra
        * `Guest machine`: Máy ảo
        * `Hypervisors`: Phần mềm tạo ra máy ảo trên phần cứng máy chủ  
    1. Ảo hóa toàn phần (`Full virtualization`)
        * Toàn bộ phần cứng của máy tính sẽ được ảo hóa hết để một hệ điều hành ảo khác có thể chạy trên đó một cách đầy đủ và bình thường, không bị thay đổi hay chỉnh sửa. Khi được ảo hóa toàn phần thì máy ảo có thể truy cập và sử dụng hết mọi tính năng của từng phần cứng một, bao gồm cả BIOS, driver, các lệnh nhập/xuất dữ liệu, truy cập bộ nhớ...
        * Ứng dụng: chia sẻ một máy tính cho nhiều người sử dụng cùng lúc, cách ly các tài khoản người dùng với nhau cũng như để tăng cường tính bảo mật, độ ổn định và hiệu suất làm việc của một hệ thống máy tính
    1. Ảo hỏa cục bộ (`Partial virtualization`)
        * Ảo hóa một phần chỉ tiến hành ảo hóa một số phần cứng nhất định của máy tính nên nó không đủ tài nguyên để vận hành một hệ điều hành ảo hoàn chỉnh, thay vào đó nó chỉ cho phép chúng ta chạy một số phần mềm mà thôi 
        * Ưu điểm của áo hóa một phần là nó dễ triển khai hơn ảo hóa toàn phần, nó tỏ ra hữu ích khi người ta chỉ muốn dùng máy ảo để chạy một phần mềm quan trọng nào đó, họ sẽ dùng ảo hóa một phần để tạo ra đủ tài nguyên cần thiết để chạy nó mà không cần phải ảo hóa cả một hệ thống phức tạp. Bởi vì nếu dùng ảo hóa toàn phần chỉ để chạy một phần mềm duy nhất thì coi như là ta đã lãng phí tài nguyên máy tính một cách vô ích
    1. Ảo hóa song song (`Paravirtualization`)  
        * Ảo hóa song song không mô phỏng phần cứng để chạy hệ điều hành ảo mà thay vào đó nó sẽ tạo một một lớp giao diện phần mềm (hay một tập lệnh API) để các hệ điều hành ảo và hypervisor có thể giao tiếp với nhau, và xem API đó như là ngôn ngữ chung giữa 2 phía, mục đích là để giảm thiểu thời gian cần thiết mỗi khi thi hành các câu lệnh trên hệ thống
1. Ảo hóa desktop  
    * Thường thấy trong hệ thống máy chủ của các doanh nghiệp, công ty.
    Ví dụ:   
    * Trong công ty có một máy chủ trung tâm, chứa toàn bộ dữ liệu, phần mềm và các chương trình cần thiết để các nhân viên có thể sử dụng. Tuy nhiên do có quá nhiều nhân viên, họ có thể ngồi ở phòng riêng hay nhà riêng, người ta không thể đầu tư cho mỗi người một cái máy tính đầy đủ như thế, thay vào đó người ta tạo ra cái gọi là Desktop ảo.
    * một máy chủ trung tâm có thể tạo ra nhiều Desktop ảo (giống như Desktop máy tính của bạn vậy). Mỗi một nhân viên sẽ được cấp một Desktop ảo của máy chủ đó và cái hay của nó là người ta có thể ngồi làm việc từ xa, dùng một máy tính khác hay thậm chí là các thiết bị di động như điện thoại hay tablet để truy cập vào Desktop ảo và bắt đầu làm việc. Họ có thể sử dụng mọi phần mềm và dữ liệu có trên Desktop ảo, tất cả các dữ liệu sẽ được xử lý và lưu trữ từ xa ngay trên máy chủ trung tâm. Người nhân viên không cần phải có một máy tính quá cao cấp để có thể làm việc với máy chủ trên. Như vậy, nhờ có Dekstop ảo mà chỉ cần một máy chủ, ta có thể phân phát cho nhiều người làm việc cùng lúc trên máy tính đó mà vẫn đảm bảo được tính hiệu quả và độ an toàn của dữ liệu

1. Ảo hóa RAM    
    * RAM ảo được tạo ra từ việc gộp chung toàn bộ số RAM thực đang có trong các máy tính của một Data Center và tạo thành một "cục" RAM (memory pool) chung cho toàn hệ thống. Các máy tính con trong hệ thống máy chủ hay các ứng dụng con có thể truy cập và sử dụng số RAM ảo mày mà không bị giới hạn về mặt phần cứng và có thể dùng số RAM đó để làm bộ nhớ cache tốc độ cao hay làm bộ nhớ cho CPU và GPU
    * Ưu điểm của RAM ảo là nó cho phép các ứng dụng có thể tận dụng được số RAM cực kỳ lớn, giảm thiểu tình trạng chờ đợi quá lâu do thiếu RAM và tăng hiệu suất máy tính, tận dụng số RAM nhàn rỗi trong hệ thống máy chủ
    * Trên máy tính cá nhân, đặc biệt các dòng Windows, người dùng cũng có thể sử dụng một phần ổ cứng chia ra để làm RAM ảo, mục đích là để giảm tải gánh nặng xử lý trên RAM thật khi RAM thật không đủ để xử lý các ứng dụng. Hoặc như phân vùng Swap trong các hệ điều hành Linux là một ví dụ.

1. Ảo hóa mạng (`Network virtualization`)  
    * Là quá trình kết hợp các phần cứng, phần mềm mạng và chức năng của mạng vào một thực thể duy nhất. Một virtual network, nó được chia làm hai loại `external virtualization` và `internal virtualization`
        * External virtualization: kết hợp nhiều mạng hay thành phần của mạng vào trong một đơn vị ảo hóa.
        * Internal virtualization: cung cấp một chức năng giống như mạng đến các software container trên một hệ thống.
1. Máy chủ ảo (`VPS - Virtual Private Server`) 
    * Là nhiều máy chủ ảo chạy trên một máy chủ thực. Một máy chủ có thể tạo ra nhiều máy chủ ảo để vận hành các website. Ưu điểm của VPS là nó giúp người ta có thể tiết kiệm đáng kể chi phí dùng để đầu tư cho việc mua, thuê server

