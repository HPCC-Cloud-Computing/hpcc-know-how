# Tìm hiểu về docker

## 1. Giới thiệu

Containerization là quá trình đóng gói các thành phần và các phụ thuộc của ứng dụng đó theo một chuẩn nào đó, một cách độc lập và lightweight, cái đó được gọi là container.

`Container = app + support file`

Docker là một nền tảng containerization được phát triển để đơn giản hóa và chuẩn hóa việc triển khai trong các môi trường khác nhau

## 2. Docker

Docker là một phần mềm containerization được sử dụng phổ biến.

![https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/docker/Container-Overview.png](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/docker/Container-Overview.png)

Hình trên cho ta thấy mối quan hệ giữa các containers với host system. Container cô lập hóa các ứng dụng và sử dụng tài nguyên hệ điều hành cái mà đã được ảo hóa bởi Docker. Ở bên phải của hình ta có thể thấy container được xây dựng theo các (layer)tầng và các container có thể chia sẻ việc sử dụng các layer nền tảng (underlying) giúp giảm việc sử dụng tài nguyên.

### Ưu điểm chính của Docker

- Sử dụng ít tài nguyên(lightweight): Thay vì ảo hóa toàn bộ hệ điều hành, container cô lập ở mức process và sử dụng nhân của máy chủ.
- Khả năng di chuyển: Tất cả các thứ cần thiết cho một ứng dụng đã được đóng gói trong một container cho phép ứng dụng có thể chạy trên bất kỳ máy chủ Docker nào. Đây là sự khác biệt giữa Docker container với virtual machine.
- Predictability: host không cần quan tâm đến cái gì chạy bên trong của container và container cũng không cần quan tâm đến cái gì chạy trên host. Giao diện chuẩn hóa giúp giải quyết vấn đề này

Thông thường khi thiết kế một ứng dụng sử dụng Docker cách tốt nhất là tách các chức năng thành các container riêng biệt. Điều này cho phép dễ dàng mở rộng và cập nhật trong tương lai. Tính linh hoạt này là một trong những lý do khiến nhiều người sử dụng Docker để phát triển và triển khai.

## Image

Image là một template chỉ đọc dùng để tạo docker container. Thông thường một image được xây dựng trên cơ sở của một image khác. Ví dụ bạn có thể xây dựng một image cái mà dựa trên cơ sở ubuntu image nhưng trên đó bạn cài đặt apache web server và ứng dụng của bạn cùng với các thành phần cần thiết để chạy ứng dụng của bạn. Bạn có thể tự tạo ra image hoặc sử dụng các image đã được tạo bởi những người khác và được công bố trên một registry.

## Docker registries

Docker registries là một kho lưu trữ các docker image. Docker hub và Docker Cloud là các public registries, nơi mà ai cũng có thể sử dụng. Theo mặc định docker sẽ tìm kiếm các image trên Docker Hub. Bạn thậm chí còn có thể chạy

## Tài liệu tham khảo

1. [https://www.digitalocean.com/community/tutorials/the-docker-ecosystem-an-introduction-to-common-components](https://www.digitalocean.com/community/tutorials/the-docker-ecosystem-an-introduction-to-common-components)
1. [https://docs.docker.com/engine/docker-overview](https://docs.docker.com/engine/docker-overview/)
