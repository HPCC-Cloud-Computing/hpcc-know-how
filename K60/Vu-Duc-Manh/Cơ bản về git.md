# Cơ bản về git

Mục đích: Tự cấu hình và khởi động được một kho chứa, bắt đầu hay dừng theo dõi các tập tin, tổ chức/sắp xếp (stage), commit các thay đổi, bỏ qua tập tin, khôi phục lỗi, duyết lịch sử của dự án hay xem thay đổi giữa các lần commit cũng như push, pull code

## 1. Tạo một kho chứa Git

* Tạo dự án Git theo hai phương pháp chính

  * Dùng dự án hay thư mục có sẵn import vào Git
  * Tạo bản sao kho chứa Git trên một máy chủ khác

* **Khởi tạo một kho chứa từ thư mục cũ**

  * Vào thư mục gõ lệnh 

    > $ git init

  * File .git được tạo mới trong thư mục, file này là bộ khung kho chứa Git

  * Thực hiện commit đầu tiên (initial commit) để chắc chắn bắt đầu theo dõi các tập tin

    > $ git add *
    >
    > $ git commit -m"initial commit"

* **Sao chép một kho chứa đã tồn tại**

  > $ git clone "http://github.com/.." [newname]

  * Có thể sử dụng Git qua một số giao thức truyền tải khác nhau nhứ giao thức *git://* , hoặc *http(s)://* hoặc *user@server:path.git* thông qua giao thức SSH

## 2. Ghi lại thay đổi vào kho chứa

* Mỗi tập tin trong thư mục làm việc có thể ở một trong hai trạng thái:

  * tracked : là tập tin đã có trong snapshot trước, chúng có thể là
    * unmodified
    * modified
    * staged
  * untracked : là tất cả các tập tin còn lại, những tập tin không có trong snapshot (lần commit) trước hoặc không có trong khu vực tổ chức (staging area). 

* Ban đầu, khi tạo bản sao của một kho chứa, tất cả tập tin sẽ ở trạng thái tracked và unmodified

  * Khi chỉnh sửa bất kì file nào file đó sẽ có trạng thái modified
  * Sau khi thay đổi tệp tin, các file cần được stage ( git add) và sau đó commit tất cả các thay đổi đã được staged, quá trình này lập đi lăp lại
  * "git rm --cached <file>..." to unstage

* **Kiểm tra trạng thái của tập tin** 

  * sẽ hiện thông tin về kho chứa của bạn, nhánh bạn đang thao tác

  > $ git status

  * nothing to commit
  * staged 
  * modified

* **Theo dõi tập tin mới**

  > $ git add filename/directory

  File hoặc thư mục sẽ được staged, sẽ được commit trong lần commit sắp tới

* **Quản lý các tập tin đã thay đổi**

  Khi sửa file đã được theo dõi, khi kiểm tra trạng thái file đó sẽ ở trạng thái modified, chạy

  > $ git add filename

  để staged các thay đổi và nó sẽ được staged

* **Bỏ qua các tập tin**

  * Để cấu hình Git không tự động thêm một số loại tập tin hay hiển thị là không được theo dõi, tạo tập tin liệt kê các "mẫu" cần loại bỏ có tên **.gitignore**. VD

  > $cat .gitignore
  >
  > *.[oa]
  >
  > *~

  * File .gitignore sẽ loại bỏ 
    * các tập tin đuôi .o hoặc .a
    * các tập tin có đuôi là dẫu ngã ~, chúng là các giá trị tạm thời bởi nhiều chương trình soạn thảo như Emacs
  * Quy tắc mẫu có thể sử dụng trong .gitignore
    * Dòng trống hoặc bắt đầu # sẽ được bỏ qua
    * Các mẫu chuẩn sẽ hoạt động tốt
    * Mẫu có thể kết thúc bằng dấu / để chỉ định 1 thư muc
    * Tạo "mẫu phủ định" bằng cách thêm dấu ! phía trước
  * Các mẫu toàn cầu giống như biểu thức regular expression rút gọn sử dụng trong shell
    * dấu * : khớp với chuỗi kí tự bất kì cả chuỗi rỗng
    * [abc] : khớp bất kì ký tự nào trong dấu ngoặc
    * ? : khớp 1 ký tự đơn
    * [0-9] : khớp các số từ 0 đến 9

* **Xem các thay đổi Staged và Unstaged**

  > $ git diff 

  * Cho biết chính xác các thay đổi trong Project của bạn, thay đổi nào được staged thay đổi nào chưa được staged
  * git diff sẽ so sánh thư mục làm việc của bạn với khu vực tổ chức staging

  > $ git diff --cached 
  >
  > $ git diff --staged

  * 2 lệnh trên tương đương nhau, lệnh 2 hỗ trợ từ Git 1.6.1, so sánh những thay đổi đã được staged với commit trước đó

* **Commit thay đổi**

  > $ git commit

* **Bỏ qua khu vực tổ chức**

  * Bỏ qua bước staged, khi commit Git sẽ tự động thêm tất cả các tập tin đã được theo dõi trước khi thực hiện lệnh commit

    > $ git commit -a 

* **Xóa tập tin**

  * Để xóa tập tin, bạn phải xóa nó khỏi danh sách được theo dõi (xóa khỏi khu vực tổ chức) và sau đó commit

  * > $ git rm

  * Nếu tập tin sửa đã được thêm vào danh sách staged, bạn ép Git xóa với tùy chọn -f. Với chức năng này bạn có thể tránh trường hợp xóa nhầm file không thể khôi phục từ git

  * Bạn có thể giữ tập tin làm việc nhưng không staged chúng và bạn quên thêm nó vào file .gitignore và đã vô tình tổ chức chúng

  * > $ git rm --cached readme.txt

  * Xóa tập tin với mẫu

  * > $ git rm log/\*.log : xóa các tập tin .log trong thư mục log

* **Di chuyển tập tin**

  * Git không theo dõi việc di chuyển tập tin một cách rõ ràng. Khi đổi tên một tập tin, Git không lưu trữ thông tin gì

  * > $ git mv file_from file_to

  * Lệnh trên coi như việc đổi tên 1 tập tin

## 3. Xem lịch sử Commit

**Xem lịch sử Commit**

> $ git log

* Với tùy chọn -p : nó sẽ hiển thị diff của từng commit 
* Thêm -2 để hiển thị 2 commit gần nhất
* --word-diff : cung cấp việc xem thay đổi một các tổng quát hơn hoặc để xem diff một cách tổng quát 
  * thêm tham số -U1: để giảm phần hiển thị diff xuống 1 dòng
* --stat: xem 1 số thống kê tóm tắt cho mỗi commit 
* --pretty: thay đổi phần hiển thị ra theo các cách khác nhau và cung cấp 1 số lựa chọn sẵn
  * cú pháp ` git log --pretty=option`
  * oneline: in mỗi commit trên 1 dòng
  * short, full, fuller: hiển thị gần như tương tự nhau với ít nhiều thông tin theo cùng thứ tự
  * format : cho phép chỉ định định dạng riêng của phần hiển thị

**Giới hạn thông tin đầu ra**

* `git log -<n>`  với n là số nguyên dương bất kì
* --since và --until
  * `git log --since=2.weeks` hiển thị thông tin trong vòng 2 tuần gần nhất
  * tham số thời gian hỗ trợ nhiều định dạng, chỉ định ngày cụ thể 2008-01-15 hay "2 years 1 day 3 minutes ago"
* --author: hiển thị commit tên tác giả thỏa mãn điều kiện nhất định
* --committer: chỉ hiển thị các commit mà tên người commit thỏa mãn điều kiện

## 4. Git phục hồi

**Phục hồi**: phục hồi phần nào đó

* **Thay đổi commit cuối cùng**

  * > $ git commit --amend

    * chạy lại commit vừa commit trước đó có thể sửa nội dung commit hoặc thêm file 

* **Loại bỏ tập tin đã tổ chức**

  * Khi vô tình git add *, bạn muốn xóa file nào đó khỏi khu vực tổ chức

  * > $ git reset HEAD <file>

* **Phục hồi tập tin đã thay đổi**

  * Xóa bỏ những thay đổi trong 1 file nhất định

  * > $ git checkout -- <file>

  * Lệnh này khá nguy hiểm khi làm mất hoàn toàn những thay đổi và không phục hồi lại được

## 5. Git làm việc từ xa

* **Hiển thị máy chủ** 

  * Xem git trên máy cấu hình tới máy chủ nào

    * > ​		$ git remote

    * Liệt kê ngắn gọn ccacs máy chủ từ xa đã được chỉ định, nếu sao chép từ kho chứa sẵn nó sẽ hiện bản gốc origin

    * Tùy chọn `git remote -v`  để hiển thị địa chỉ của máy chủ

* **Thêm các kho từ xa**

  * Thêm mới một kho chứa Git từ xa như 1 tên rút gọn

  * > $ git remote add shortname  [url]

  * Bạn có thể sử dụng shortname bên trên để truy cập như là 1 địa chỉ

  * Để duyệt qua/truy cập tất cả thông tin mà Paul có mà bạn chưa có trong kho chứa

    * > $ git fetch shortname

* **Truy cập và kéo về từ máy chủ trung tâm**

  * > $ git fetch [remote-name]

  * Truy cập vào dự án từ xa và kéo xuống toàn bộ dữ liệu mà bạn chưa có trong đó

  * Nếu bản sao được tạo từ một kho chứa nào đó khác, kho chứa đó sẽ có tên ngầm định origin, chạy `git fetch origin` sẽ truy xuất bất kỳ thay đổi mới nào được đẩy lên máy chủ từ sau khi bạn sao chép hoặc lần truy xuất cuối cùng

  * Các thay đổi bạn đang được thực hiện phải tích hợp thủ công vào kho chứa nội bộ khi sẵn sàng, `git fetch` chỉ kéo tất cả dữ liệu về

  * Nếu 1 nhánh được cài đặt theo dõi 1 nhánh từ xa khác

    * > $ git pull

    * Lệnh trên sẽ tự động truy xuất và sau đó tích hợp nhánh từ xa vào nhánh nội bộ

  * `git clone` tự động cài nhánh chính nội bộ để theo dõi nhánh chính trên remote master branch- nơi bạn sao chép về. Vì vậy `git pull` sẽ truy xuất dữ liệu từ máy chủ trung tâm tới nơi lần đầu bạn sao chép và cố gắng tích hợp chúng và kho chứa hiện thời bạn đang làm việc

* **Đẩy lên máy chủ trung tâm** 

  * > $ git push ten-máy-chủ tên-nhánh

* **Kiểm tra một máy chủ trung tâm**

  * > $ git remote show [tên-trung-tâm]

* **Xóa và đổi tên từ xa**

  * > $ git remote rename name1 name_new

  * Lện trên cũng thay đổi cả nhánh trung tâm/từ xa của bạn

  * > $ git remote rm remote_name

  * Xóa máy chủ remote_name

## 6. Git đánh dấu

* Git có khả năng đánh dấu các mốc quan trọng trong lịch sử của dự án.

* **Liệt kê Tag**

  * > $ git tag
    >
    > $ git tag -1 'v1.4.2.*'

* **Thêm tag mới**

  * Có 2 loại tag chính

    * lightweight: giống như một nhánh mà không có sự thay đổi- nó chỉ trỏ đến 1 commit nào đó
    * annotated: được lưu trữ là những đối tượng đầy đủ trong cơ sở dữ liệu của Git
      * annotated tag được băm
      * chứa tên người tag, địa chỉ email, ngày tháng, thông điệp kèm theo
      * có thể được ký và xác thực bằng GNU Privacy Guard

  * Tạo annoted tag

    * > $ git tag -a v1.4 -m 'my version 1.4'

    * -m: để truyề nội dung/thông điệp cho tag

    * > $ git show tag_name

    * để hiển nội dung cùng với commit

  * **Signed tags** 

    * dùng tham số -s

    * > $ git tag -s v1.5 -m 'my signed 1.5 tag'

  * Tạo lightweight tag

    * git tag v1.4-lw
    * khi chạy `git show` sẽ chỉ hiện commit

  * Xác thực tag

    * > $ git tag -v [tên-tag]

    * cần public key của người ký để xác thực

  * Tag muộn

    * Đánh tag cho commit trước đó

    * > $ git tag -a v1.2 -m 'version1.2' 9fceb02

    * 9fceb02 là 1 phần của mã băm

  * Chia sẻ tag

    * Mặc định `git push` không truyền các tags, để truyền tag dùng

    * > $ git push origin [tên-tag]
      >
      > $ git push origin --tags : đẩy tất cả tag lên máy chủ 

# Phân nhánh trong Git 

## 1. Nhánh cơ bản

* Tạo nhánh

  > $ git branch new

* Chuyển nhánh

  > $ git checkout new

## 2. Cơ bản về phân nhánh và tích hợp

* Tạo nhánh mới và đồng thời chuyển tới nhánh đó

  * > $ git checkout -b iss53

* Merge branch nhánh vào master

  * > $ git checkout master
    >
    > $ git merge sub-branch

* Khi tích hợp, nếu có sự thay đổi cùng nội dung của 1 file ở hai nhánh khác nhau, Git sẽ báo conflict 

  * Xử lý conflict bằng tay

## 3. Quản lý các nhánh

* Danh sách các nhánh

  * > $ git branch

* Commit mới nhất trên mỗi nhánh

  * > $ git branch -v

* Xem nhánh đã tích hợp vào nhánh hiện tại chưa và ngược lại với

  * git sử dụng tích hợp 3 chiều và tự động tìm cha chung giữa 2 nhánh cần merge

  * > $ git branch --merged
    >
    > $ git branch --no-merged

## 4. Quy trình làm việc phân nhánh

* **Nhánh lâu đời** 
  * Thực hiện phân nhánh dự án lớn với nhánh master là mã nguồn đã phát hành hoặc chuẩn bị phát hành
  * Các nhánh khác dần được phân theo mức độ ổn định, nhánh đang phát triển thường ở cuối sau khi phát triển thành công thì thực hiện merge code vào nhánh tầng trên như là các xi-lô
* **Nhánh chủ đề** 
  * Thực hiện phân nhánh theo chủ đề như một tính năng của sản phẩm
  * Khi nhánh phát triển xong hợp lí bạn có thể merge code vào branch chính

## 5. Nhánh Remote

* **Nhánh remote** là các tham chiếu tới trạng thái của các nhánh trên khoa chứa trung tâm của bạn, hoạt động như các bookmark nhắc nhở bạn các nhánh trên kho chứa trung tâm của bạn ở đâu vào lần cuối kết nối

* Khi clone code git tự động tạo remote origin, tải dữ liệu về và đặt con trỏ tới nhánh origin/master, con trỏ này không thể di chuyển

  * git cung cấp nhánh master riêng cùng 1 vị trí oringin/master để bắt đầu làm việc

  * khi origin/master được cập nhật từ một người nào đó, để cập nhật code dùng lệnh

    * > $ git fetch origin

* Đẩy nhánh lên máy chủ

  * > $git push (remote) (branch)

#### Theo dõi các nhánh

* Checkout nhánh nội bộ từ 1 nhánh trung tâm tự động tạo ra 1 tracking branch

* Nếu đang trên tracking branch `git push` sẽ tự động biết nó đẩy lên nhánh nào máy chủ nào.

  * Chạy `git pull` để truy xuất toàn bộ các tham chiếu từ xa

* Để theo dõi 1 nhánh

  * > $ git checkout --track origin/sub_branch

  * Nhánh nội bộ sẽ tự đống push pull từ oringin/sub_branch

#### Xóa nhánh trung tâm

> $git push [remotename]  : [branch]

## 6. Rebasing

* Có thể sử dụng thay merge để tích hợp code

* Ví dụ có nhánh master là nhóm chính và nhánh experiment

* > $ git checkout experiment
  >
  > $ git rebase master

  * Nó sẽ sử dụng các commit trên nhánh experiment tạo lại trên nhánh master bằng cách tìm cha chung của 2 nhánh, tìm sự khác biệt trong mỗi commit giữa experiment và master, khôi phục nhánh master về cùng 1 commit với nhánh bạn đang rebase, và áp dụng các thay đổi
  * Sau đó thực hiện fast-forward merge 