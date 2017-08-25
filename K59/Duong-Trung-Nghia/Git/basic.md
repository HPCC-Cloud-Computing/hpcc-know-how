#Version Control System - VCS
- **Vì sao nên dùng VCS**: 
    * Giúp ta khôi phục lại phiên bản cũ của các file, khôi phục lại phiên bản cũ của toàn bộ dự án, xem lại các thay đổi đã được thực hiện theo thời gian, xem ai là người thực hiện thay đổi cuối cùng có thể gây ra sự cố, hay xem ai là người đã gây ra sự cố đó và còn nhiều hơn thế nữa. 
    * Sử dụng VCS còn đồng nghĩa với việc khi bạn làm rối tung mọi thứ lên hay vô tình xoá mất các file đi, bạn có khôi phục lại chúng một cách dễ dàng. Hơn nữa, tất cả quá trình này có thể được thực hiện rất nhanh chóng và không hề tốn quá nhiều công sức

##Centralized Version Control Systems - CVCSs
- Cơ chế quản lý: các hệ thống này, ví dụ như CVS, Subversion, và Perforce, bao gồm một máy chủ có chứa tất cả các tập tin đã được "phiên bản hoá" (versioned), và danh sách các máy khách có quyền thay đổi các tập tin này trên máy chủ trung tâm đó. Trong vòng nhiều năm, mô hình này đã trở thành tiêu chuẩn cho việc quản lý phiên bản
- Ưu điểm: 
    * Tất cả người dùng đều biết một phần nào đó những việc mà những người khác trong dự án đang làm
    * Người quản lý có quyền quản lý ai có thể làm gì theo ý muốn; và việc này dễ dàng hơn nhiều so với việc phải quản lý ở từng cơ sở dử liệu ở từng máy riêng biệt.
- Nhược điểm:
    * Nếu máy chủ đó không hoạt động trong một giờ, nghĩa là trong khoảng thời gian đó không ai có thể cộng tác với những người còn lại hoặc lưu trữ các thay đổi đã được phiên bản hoá của bất kỳ tập tin nào mà người đó đang thao tác
    * Nếu ổ cứng lưu trữ cơ sở dữ liệu trung tâm bị hỏng, và các sao lưu dự phòng chưa được tạo ra tính đến thời điểm đó, bạn sẽ mất toàn bộ lịch sử của dự án đó, ngoại trừ những phiên bản cục bộ mà người dùng có được trên máy tính cá nhân
    * Các hệ thống quản lý phiên bản cục bộ phải đối diện với vấn đề tương tự như thế này mỗi khi toàn bộ lịch sử của dự án được lưu ở một nơi, bạn có nguy cơ mất tất cả.

##Distributed Version Control Systems - DVCSs
- Cơ chế quản lý: Trong các DVCS (ví dụ như Git, Mercurial, Bazaar hay Darcs), các máy khách không chỉ "check out" (sao chép về máy cục bộ) phiên bản mới nhất của các tập tin: chúng sao chép (mirror) toàn bộ kho chứa (repository)
- Ưu điểm so với CVCSs: 
    * Nếu như một máy chủ nào mà các hệ thống quản lý phiên bản này (mỗi máy khách là một hệ thống riêng biệt) đang cộng tác ngừng hoạt động, thì kho chứa từ bất kỳ máy khách nào cũng có thể dùng để sao chép ngược trở lại máy chủ để khôi phục lại toàn bộ hệ thống
    * Ngoài ra, phần lớn các hệ thống này xử lý rất tốt việc quản lý nhiều kho chứa từ xa, vì thế bạn có thể cộng tác với nhiều nhóm người khác nhau theo những cách khác nhau trong cùng một dự án.

#GIT
##Cơ chế hoạt động của GIT
- Git chỉ thêm mới dữ liệu
   + Khi bạn thực hiện các hành động trong Git, phần lớn tất cả hành động đó đều được thêm vào cơ sở dữ liệu của Git -> hoàn toàn không sợ mất dữ liệu, và việc làm hỏng có thể quay lại trạng thái trước một cách dễ dàng.
- 3 trạng thái của Git:
   + committed: dữ liệu đã được lưu trữ một cách an toàn trong cơ sở dữ liệu
   + modified: bạn đã thay đổi tập tin nhưng chưa commit vào cơ sở dữ liệu
   + staged: đã đánh dấu sẽ commit phiên bản hiện tại của một tập tin đã chỉnh sửa trong lần commit sắp tới
   ![Hình minh hoạ](https://git-scm.com/figures/18333fig0106-tn.png)
   + Tiến trình công việc (workflow) cơ bản của Git:
        1. Thay đổi tập tin trong thư mục làm việc
        2. Tổ chức các tập tin, tạo mới snapshot các tập tin đó vào Staging area
        3. Commit và snapshot của các tập tin sẽ được lưu vào thư mục git

##Thao tác với GIT
1. Tạo Một Kho Chứa Git
    * Khởi Tạo Một Kho Chứa Từ Thư Mục Cũ
       ``` 
       git init
    * Sao Chép Một Kho Chứa Đã Tồn Tại
        ```
        - git clone <url>
        - git clone <url> repo_local_name
2. Ghi Lại Thay Đổi vào Kho Chứa
    * Kiểm Tra Trạng Thái Của Tập Tin
        ```
        git status
    * Theo Dõi Các Tập Tin Mới: theo dõi các tập tin mới tạo
        ```
        git add <file_name>
    * Stage file đã được thay đổi
        ``` 
        git add <file_name>
    * Bỏ Qua Các Tập Tin
        ```
        - Tạo file có tên .gitignore
        - Thêm các file không muốn đưa vào git vào trong file này
        - Quy tắc: 
            Dòng trống hoặc bắt đầu với # sẽ được bỏ qua.  
            Các mẫu chuẩn toàn cầu hoạt động tốt.
            Mẫu có thể kết thúc bằng dấu gạch chéo (/) để chỉ định một thư mục.
            Bạn có thể có "mẫu phủ định" bằng cách thêm dấu cảm thám vào phía trước (!).
    * Commit Thay Đổi
        ```
        git commit -m [message]
    * Bỏ Qua Khu Vực Tổ Chức
        ```
        git commit -a -m 
    * Xoá Tập Tin: để xoá một tập tin khỏi Git, bạn phải xoá nó khỏi danh sách được theo dõi (chính xác hơn, xoá nó khỏi khu vực tổ chức) và sau đó commit. Lệnh git rm sẽ giúp ta làm cả 2 việc đó.
        ```
        git rm <file_name>
        ```
        **Chú ý**: Một chức năng hữu ích khác có thể bạn muốn sử dụng đó là giữ tập tin trong thư mục làm việc nhưng không thêm chúng vào khu vực tổ chức. Hay nói cách khác bạn muốn lưu tập tin trên ổ cứng nhưng không muốn Git theo dõi chúng nữa
        ```
        git rm --cached <file_name>
3. Xem Lịch Sử Commit
     ```
     git log
4. Phục Hồi
    * Thay Đổi Commit Cuối Cùng
        ```
         git commit --amend
        
        
    