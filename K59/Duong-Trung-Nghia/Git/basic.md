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
##Cơ chế hoạt động của Git 
- Cách lưu trữ dữ liệu thay đổi:
    * Các hệ thống VCS khác (CVS, Subversion, Perforce, Bazaar,...) coi thông tin được lưu trữ như là một tập hợp các tập tin và các thay đổi được thực hiện trên mỗi tập tin theo thời gian [hình minh hoạ](https://git-scm.com/figures/18333fig0104-tn.png)
    * Mỗi lần bạn "commit", hoặc lưu lại trạng thái hiện tại của dự án trong Git, về cơ bản Git "chụp một bức ảnh" ghi lại nội dung của tất cả các tập tin tại thời điểm đó và tạo ra một tham chiếu tới "ảnh" đó. Để hiệu quả hơn, nếu như tập tin không có sự thay đổi nào, Git không lưu trữ tập tin đó lại một lần nữa mà chỉ tạo một liên kết tới tập tin gốc đã tồn tại trước đó [hình minh hoạ](https://git-scm.com/figures/18333fig0105-tn.png)
- Phần Lớn Thao Tác Diễn Ra Cục Bộ
    * Các thao tác trong Git phần lớn đều diễn ra cục bộ, chỉ khi bạn cần đưa mới dữ liệu về hoặc đẩy dữ liệu lên máy chủ từ xa thì lúc đó bạn mới cần đến mạng.
- Git Mang Tính Toàn Vẹn
    * Mọi thứ trong Git được "băm" (checksum or hash) trước khi lưu trữ và được tham chiếu tới bằng mã băm đó. Cơ chế mà Git sử dụng cho việc băm này được gọi là mã băm SHA-1. Đây là một chuỗi được tạo thành bởi 40 ký tự của hệ cơ số 16 (0-9 và a-f) và được tính toán dựa trên nội dung của tập tin hoặc cấu trúc thư mục trong Git
    * Thực tế, Git không sử dụng tên của các tập để lưu trữ mà bằng các mã băm từ nội dung của tập tin vào một cơ sở dữ liệu có thể truy vấn được.
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



##Thao tác với Git
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
    * Loại Bỏ Tập Tin Đã Tổ Chức
        ```
        git reset HEAD <file>
    * Phục Hồi Tập Tin Đã Thay Đổi
        ```
        git checkout -- <file>
5. Làm Việc Từ Xa
    * Hiển Thị Máy Chủ
        ```
        git remote [-v]
        -v: Hiển thị địa chỉ thư mục remote
    * Thêm Các Kho Chứa Từ Xa: thêm mới một kho chứa Git từ xa bằng một tên rút gọn để bạn có thể tham chiếu dễ dàng
        ``` 
        git remote add [shortname] [url]
    * Truy Cập Và Kéo Về Từ Máy Chủ Trung Tâm
        ```
        git fetch [remote-name]:  kéo tất cả dữ liệu về kho chứa trên máy của bạn - nó không tự động tích hợp với bất kỳ thay đổi nào mà bạn đang thực hiện
         git pull : tự động truy xuất và sau đó tích hợp nhánh từ xa vào nhánh nội bộ
    * Đẩy Lên Máy Chủ Trung Tâm
        ```
        git push [tên-máy-chủ] [tên-nhánh]
    * Kiểm Tra Một Máy Chủ Trung Tâm
        ```
        git remote show [tên-trung-tâm]
        vd: git remote show origin
    * Xóa Và Đổi Tên Từ Xa
        ```
        git remote rename <old_name> <new_name>: đổi tên một nhánh
        git remote rm <name>: xoá một nhánh 

        
## Phân nhánh trong Git 
- Git không lưu trữ dữ liệu dưới dạng một chuỗi các thay đổi hoặc delta, mà thay vào đó là một chuỗi các ảnh (snapshot)
- Giả sử bạn có một thư mục chứa ba tập tin, và bạn tổ chức tất cả chúng để commit. Quá trình tổ chức các tập tin sẽ thực hiện băm từng tập, lưu trữ phiên bản đó của tập tin trong kho chứa Git (Git xem chúng như là các blob), và thêm mã băm đó vào khu vực tổ chức
- Lệnh git commit khi chạy sẽ băm tất cả các thư mục trong dự án và lưu chúng lại dưới dạng đối tượng tree. Sau đó Git tạo một đối tượng commit có chứa các thông tin mô tả (metadata) và một con trỏ trỏ tới đối tương tree gốc của dự án vì thế nó có thể tạo lại ảnh đó khi cần thiết
- Kho chứa Git của bạn bây giờ có chứa năm đối tượng: một blob cho nội dung của từng tập tin, một "cây" liệt kê nội dung của thư mục và chỉ rõ tên tập tin nào được lưu trữ trong blob nào, và một commit có con trỏ trỏ tới cây gốc và tất cả các thông tin mô tả commit
    ![Hình minh hoạ](https://git-scm.com/figures/18333fig0301-tn.png)
- Nếu bạn thực hiện một số thay đổi và commit lại thì commit tiếp theo sẽ lưu một con trỏ tới commit ngay trước nó
    ![Hình minh hoạ](https://git-scm.com/figures/18333fig0302-tn.png)
- Một nhánh trong Git đơn thuần là một con trỏ có khả năng di chuyển được, trỏ đến một trong những commit này. Tên nhánh mặc định của Git là master. Như trong những lần commit đầu tiên, chúng đều được trỏ tới nhánh master. Và mỗi lần bạn thực hiện commit, nó sẽ được tự động ghi vào theo hướng tiến lên
- Khi bạn tạo 1 nhánh mới, git sẽ tạo một con trỏ mới, cùng trỏ tới commit hiện tại (mới nhất) của bạn. 
- Làm sao git biết bạn làm việc với nhánh nào? Git giữ một con trỏ đặc biệt có tên HEAD - đây là một con trỏ tới nhánh nội bộ mà bạn đang làm việc
    ```
    Để tạo nhánh mới: git branch <branch_name>
    Để di chuyển giữa các nhánh: git checkout <branch_name>
- Bản chất tạo một nhánh trong Git thực tế là một tập tin đơn giản chứa một mã băm SHA-1 có độ dài 40 ký tự của commit mà nó trỏ tới, chính vì thế tạo mới cũng như hủy các nhánh đi rất đơn giản. Tạo mới một nhánh nhanh tương đương với việc ghi 41 bytes vào một tập tin (40 ký tự cộng thêm một dòng mới). Đây là điểm khác biệt của Git so với các VCS khác khi chúng phải copy toàn bộ project sang một thư mục thứ 2 -> tốn tài nguyên hơn nhiều.

- [Cơ Bản Về Phân Nhánh và Tích Hợp](https://git-scm.com/book/vi/v1/Ph%C3%A2n-Nh%C3%A1nh-Trong-Git-C%C6%A1-B%E1%BA%A3n-V%E1%BB%81-Ph%C3%A2n-Nh%C3%A1nh-v%C3%A0-T%C3%ADch-H%E1%BB%A3p)
    ```
    Tạo một nhánh và chuyển sang nhánh đó đồng thời: 
        git checkout -b <branch_name>
    Tích hợp 2 nhánh (đưa 2 con trỏ trỏ cùng vào 1 vị trí):
        git merge <ten_nhanh_can_tich_hop>
- Nhánh Remote
    - Kéo về
        ```
        git fetch <branch>
        git pull <branch> (kéo về và tích hợp)
    - Đẩy Lên
        ```
        git push <remote> <branch>
    - Theo Dõi Các Nhánh
        ```
        git checkout --track <remotename>/<branch>
    - Xóa Nhánh Trung Tâm
        ```
        git push [remotename] :[branch]
