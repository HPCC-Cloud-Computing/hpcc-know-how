# Linux File System


Dưới góc nhìn của một người dùng, mô hình hệ thống file/folder trong hệ điều hành Linux khá đơn giản và dễ sử dụng: Hệ thống file của linux đơn giản là một cây thư mục bắt đầu từ thư mục gốc (root folder), một folder trong hệ thống chứa các folder con và các file. Các thao tác chính mà người dùng thực hiện trên hệ thống file này là các thao tác CRUD (Create - View - Update - Delete) với file/folder. Để thực hiện các thao tác này, thông tin duy nhất mà người dùng cần cung cấp cho hệ thống, đó là địa chỉ tương đối - tuyệt đối của file/folder mà người dùng muốn thao tác.

Tuy nhiên, để tạo ra sự đơn giản trong việc quản lý và trong các thao tác của người dùng trên hệ thống file, hệ điều hành Linux đã thiết lập một hệ thống quản lý file/folder phức tạp, bao gồm nhiều thành phần, nhiều cơ chế xử lý các thao tác trên file/folder cho phép hệ thống quản lý và tương tác hệ thống phần cứng lưu trữ bên dưới có tính chất không đồng nhất, nhiều định dạng(ntfs, Fatxx, ext2, ext3, ext4, xfs,...), qua đó cung cấp hệ thống file dưới một góc nhìn trong suốt và đơn giản cho người dùng.

![linux_file_system](./images/linux_file_system.png)

Hệ thống quản lý file của Linux được thiết kế như thế nào? Bao gồm các thành phần gì ? Các cơ chế nào đã được thiết lập để xử lý các nhu cầu tương tác với hệ thống file của người dùng ? Chúng ta sẽ tìm câu trả lời cho các câu hỏi trên thông qua bài viết này.
