# Git Basic Part 2

Ở phần trước ta đã tìm hiểu các khái niệm bơ bản về các VCS và git cũng như các câu lệch để làm việc với một project được quản lý bởi git. Trong part 1 chỉ đề cập đến việc làm việc với git một cách đơn lẻ nhưng một tính năng quan trọng của git là giúp mọi người cùng làm việc, hợp tác với nhau trong một project. Phần hai sẽ đi làm rõ cách để có thể cùng nhau hợp tác làm một project với git.

## Phân nhánh trong git

Phân nhánh là bạn tạo ra một bản sao của project và tiếp tục làm việc trên bản sao đó mà không sợ ảnh hướng đến bản chính.

Để có thể hiểu cách git phân nhánh ta cần nhớ lại [Part 1](https://github.com/NTT-TNN/Basic_knowledge/blob/master/Git_tutorial/Git_Basic_Part1.md) đã nói về cách lưu trữ dự liệu. Cụ thể git không lưu trữ một chuỗi các thay đổi mà thay vào đó git lưu chữ một chuỗi các snapshots. Khi bạn commit git lưu trữ đối tượng commit và nó chứa một con trỏ trỏ tới snapshot của nội dung của bạn đã stage cùng với tác giả và messages, cũng có thể có không hoặc nhiều con trỏ khác trỏ tới một commit hoặc một vài commit cha trực tiếp của commit đó(commit đầu tiền không có cha).

Để hiểu rõ hơn chúng ta xét ví dụ: Giả sử bạn có một thư mục gồm ba tệp tin `README test.rb LICENSE` và bạn tiến hành đưa chúng vào khu vực stage để tiến hành commit. Quá trình stage các tệp tin sẽ được băm từng tệp (thuật toán SHA-1), và lưu trữ phiên bản đó của tệp tin trong kho chứa của git(git gọi chúng là blob) và thêm mã băm vào khu vực stage.

```sh
 git add README test.rb LICENSE
 git commit -m 'initial commit of my project'
```

Khi bạn tiến hành commit git sẽ băm tất cả các thư mục trong project và lưu chúng dưới dạng đối tượng `tree` sau đó git tạo đối tượng commit có chứa thông tin mô tả(metadata) và con trỏ trỏ tới đối tượng `tree`. Bây giờ kho chứa git của bạn có năm đối tượng: 3 blob mỗi blob cho nội dung của từng tệp, một `tree` liệt kê nội dung của thư mục và cho biết tệp tin nào tương ứng với blob nào(như hình dưới).

![CTB](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/CTB.png)

Sau khi thực hiện một số thay đổi và commit lại thì commit sau sẽ trỏ tới commit ngay trước đó. Ví dụ như thực hiện hai commit thì lịch sử của dự án sẽ như hình bên dưới:

![3commit](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/LichSu3commit.png)

Ta đã biết về commit trong git vậy nhánh là gì? Nhánh trong git là một con trỏ có thể di chuyển được trỏ tới một trong những commit. Tên nhánh mặc định của git là `master`. Mỗi lần thực hiện commit nó sẽ được ghi theo hướng tiến lên như hình dưới đây:

![committienlen](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/08eb5a1e210a3c831174ced8ab26f69b0b5d402c/images/commitienlen.png)

Đó là ta thực hiện trên nhánh mặc định của git(master) vậy chuyện gì xảy ra nếu ta tạo một nhánh mới. Nếu bạn tạo một nhánh mới(git branch <tên nhánh>) git sẽ tạo ra một con trỏ trỏ tới commit mới nhất như hình dưới đây:

![anh nhanh moi](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/nhanhmoi.png)

Vậy bây giờ bạn có hai con trỏ trỏ tới commit vậy làm sao viết bạn đang dùng con trỏ nào(nhánh nào). Để biết đang làm việc trên nhánh nào git sử dụng một con trỏ đặc biệt có tên là HEAD. Lưu ý khi bạn tạo một nhánh mới git chỉ tạo một nhánh mới chứ không chuyển sang nhanh đó(không chuyển con trỏ HEAD sang nhánh đó). Để chuyển giữa các nhánh bạn có thể sử dụng câu lệnh
`git checkout <tên nhánh>` ví dụ `git checkout testing` lệch này chuyển con trỏ HEAD sang nhánh testing.

![chuyencontro](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/chuyencontroHEAd.png).

Vậy ý nghĩa của việc tạo ra nhánh là gì? Hãy tạo một file mới và thực hiện commit như sau:

```sh
vim test.rb
$ git commit -a -m 'made a change'
```

![chuyennhanhhead](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/chuyennhanhhead.png)

Bây giờ nhánh mà con trỏ HEAD trỏ tới sẽ dịch chuyển về phía trước theo từng commit. Còn nhánh master vẫn trỏ tới commit hiện tại tại lần bạn thực hiện `git  checkout`. Vậy nếu bây giơ ta chuyển lại về nhánh master điều gì sẽ xảy ra: `git checkout master`

![chuyenvemaster](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/chuyenvemaster.png)

Lệch này thực hiện hai công việc. Thứ nhất nó di chuyển con trỏ HEAD về nhánh master, thứ hai nó phục hồi tất cả các file trong thư mục làm việc trở lại trạng thái của lần snapshot mà nhánh master trỏ tới. Bây giờ trên nhánh master hãy tạo một vài thay đổi và commit một lần nữa:

```sh
vim test.rb
$ git commit -a -m 'made other changes'

```

Bây giờ lịch sử project của bạn được tách ra làm hai nhánh. Bạn tạo mới một nhánh(testing) và chuyển sang nhánh đó thực hiện một vài thay đổi sau đó lại quay lại nhánh chính(master) cũng thực hiện một vài thay đổi. Những thay đổi này là độc lập với nhau ở hai nhánh riêng biệt của project bạn có thể di chuyển giữa các nhánh bằng lệch `git checkout`.

![hainhanh](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/hainhanh.png)

Ta đã hiểu về cách git tạo nhánh và chuyển nhánh. Vấy vấn đề đặt ra là bây giờ có hai nhánh làm sao để có thể merge hai nhánh đó vào. Để có thể merge nhánh testing vào nhánh master ta cần chuyển vào nhành master và tiến hành merge như sau:

```txt
git checkout master
git merge testing

```

Nếu việc merge thành công bạn sẽ nhận được kết quả

```txt
Updating 6a84c02..702f5c0
Fast-forward
 onbranchtesing.txt | 0
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 onbranchtesing.txt

```

Đó là với trường hợp không có conflict. Đôi khi việc merge này trở nên phức tạp hơn nếu có xảy ra conflict. Conflict xảy ra khi bạn cùng thay đổi nội dung của một tệp tin ở hai nhánh khác nhau git sẽ không thể tự động merge. Khi đó nếu bạn thực hiện `git merge testing` bạn sẽ nhận được kết quả

```txt
Auto-merging onbranchtesing.txt
CONFLICT (content): Merge conflict in onbranchtesing.txt
Automatic merge failed; fix conflicts and then commit the result.

```

Điều này có nghĩa là git không thể commit merge được. Nó tạm dừng quá trình này cho tới khi bạn giải quyết xong xung đột. Bạn có thể xem tệp tin nào chưa được merge sau khi xung đột bằng câu lệch `git status`

```txt
On branch master
Your branch is ahead of 'origin/master' by 2 commits.
  (use "git push" to publish your local commits)
You have unmerged paths.
  (fix conflicts and run "git commit")

Unmerged paths:
  (use "git add <file>..." to mark resolution)

both modified:   onbranchtesing.txt

```

Với tệp tin nào chưa được merge git sẽ liêt kê là unmerged. Git cũng thêm các dấu hiệu riêng trong tệp tin xung đột bạn có thể mở tệp tin và giải quyết xung đột một cách thủ công. Tệp  tin sẽ chưa phần như sau:

```txt
<<<<<<< HEAD

trên nhánh master
=======
thực hiện trên nhánh tesing
>>>>>>> testing
```

Điều này có nghĩa là phiên bản trong nhánh HEAD(ở đây là master) mọi thứ trên ký hiệu ======= còn lại là ở dưới là trên nhánh master. Để giải quyết bạn chọn nội dung của một trong hai phần hoặc gộp chúng lại. Sau khi giải quyết xong chạy lệch `git add` để đánh giấu giấu là chúng đã được giải quyết.

Sau khi đã làm xong công việc trên một nhánh bạn có thể xóa nhánh đó bằng câu lệch: `git brach -d <tên nhánh>` ví dụ : `git branch -d testing`.

Bạn đã tạo mới, merge, xóa nhánh. Cùng tìm hiểu thêm một số tính năng để quản lý nhánh một cách dễ dàng và thuận tiện. Lệch `git branch` nếu sử dụng không có tham số sẽ cho ta danh sách tất cả các nhánh đang có.

```sh
git branch
  iss53
* master
  testing
```

Luôn có một nhánh có ký tự * đứng trước cho biết rằng bạn đang làm việc trên nhánh đó. Ngoài ra để xem các commit mới nhất trên từng nhánh ta có thể dùng câu lệch `git branch -v`.

```sh
$ git branch -v
  iss53   93b412c fix javascript issue
* master  7a98805 Merge branch 'iss53'
  testing 782fd34 add scott to the author list in the readmes
```

Git cũng cung cấp tính năng để bạn tìm và lọc các nhánh dựa trên việc đã merge hay chưa. Để biết nhanh nào đã merge vào nhánh hiện tại bạn có thể sử dụng câu lệch :`git branch --merged`. Kết quả có dạng như sau:

```bat
$ git branch --merged
  testing
* master
```

Trong danh sách có nhánh testing đã được merge vào trước đó. Nhánh này không có dấu * trước có nghĩa là nó đã có thể xóa đi mà không lo mất dữ liệu vì dự liệu đã được merge vào nhánh hiện tại. Câu lệch trái ngược với câu lệch `git branch --merged` là câu lệch `git branch --no-merged` để hiện thị các nhánh chưa được merge.

```sh
$ git branch --no-merged
  testing2
```

Bởi vì nhánh này chưa được merge vào nên nếu bạn tiến hành xóa git sẽ báo lỗi để đảm bảo dữ liệu không bị mất.

```sh
$ git branch -d testing
error: The branch 'testing' is not an ancestor of your current HEAD.
If you are sure you want to delete it, run 'git branch -D testing'.
```

Ngoài cách sử dụng merge cũng có một cách khác để bạn tích hợp hai nhánh lại với nhau đó là sử dụng rebase. Vậy rebase là gì và nó giống và khác merge như thế nào?

Rebase cũng như merge là một cách để bạn có thể tích hợp hai nhánh vào bới nhau. Để hiểu sự giống và khác nhau ta hãy cùng xét ví dụ:

B1. Đầu tiên ta sẽ tạo một project mới: `test_merge_rebase`.
B2. Tạo file README trên nhánh master

```sh
touch README.md
echo "tao file README tren nhanh master" >> README.md
git add README.md
git commit -m 'Create README'

```

B3. Tạo hai nhánh mới từ nhánh master

```sh
git branch rebase
git branch merge

```

B4. Checkout đến từng nhánh và thực hiện vài thay đổi sau đó commit

```sh
git checkout rebase
touch rebase.txt
echo "mot vai dong" >> rebase.txt
git add rebase.txt
git commit -m 'Create rebase.txt file'

```

```sh
git checkout merge
touch merge.txt
echo "mot vai dong" >> hello.txt
git add merge.txt
git commit -m 'Create merge.txt file'

```

B5. Checkout về nhánh master update và commit

```sh
git checkout master
echo "viet tren nhanh master" >> README.md
git add README.md
git commit -m 'Update README'

```

B6. Bây giờ nhánh master đã thay đổi ta cần tích hợp thay đổi đó vào hai nhánh rebase và merge bằng hau cách khác nhau như sau:

```sh
git checkout rebase
git rebase master

```

```sh
git checkout merge
git merge master

```

B7. Bây giờ hãy tiến hành so sánh 2 source-tree của hai nhánh.

Nhánh merge

```sh
git checkout merge
git log --graph --pretty=oneline --abbrev-commit

```

```s
*   e7b4171 Merge branch 'master' into merge
|\  
| * cccfae1 Update README
* | 0caa5e8 Create merge.txt file
|/  
* 35179d4 Create README

```

Nhánh rebase

```sh
git checkout rebase
git log --graph --pretty=oneline --abbrev-commit

```

```s

* 8dd1ecf Create rebase.txt file
* cccfae1 Update README
* 35179d4 Create README

```


### Kết Luận

1. Kết quả trên hai nhánh được tích hợp từ nhánh master vào có nội dung giống nhau chỉ khác nhau source-tree
1. Trên mỗi nhánh commit đầu tiên đều là commit mới nhất
1. Trên nhánh rebase mọi người sẽ thấy commit trên nhánh rebase nằm trên commit mới nhất của nhánh master. Còn trên nhánh master commit của nhánh master nằm trên commit của nhánh merge và cũng có một commit mới được tạo ra khi tiến hành merge. Điều này có nghĩa là bạn sẽ sử dụng rebase khi bạn muốn commit của bạn luôn luôn là mới nhất còn sử dụng merge khi muốn mọi commit đúng theo thứ tự.
1. Ngoài ra bạn cũng thấy source-tree của trường hợp merge có dạng phân nhánh nên sẽ trở nên rối nếu có rất nhiều nhánh còn rebase thì ngược lại hoàn toàn là một đường thằng.
1. Rebase thường được sử dụng trên nhánh riêng vì sử dụng rebase giúp history commit trên nhánh của bạn luôn hiện ở trên đầu giúp dễ quản lý.

## Commit convention

Ta đã biết sự tiện lợi của git trong quản lý phiên bản và làm việc nhóm. Bây giờ giả sửa bạn có một project lớn và bạn đã chỉnh sửa rất nhiều lần với nhiều lần commit và sau đó bạn muốn khôi phục lại một trạng thái nào đó. Lúc đó bạn sẽ phải đọc lại nội dung commit vậy làm sao để có thể viết nội dung cho một lần commit để có thể đọc lại một cách dễ dàng. Bạn cần có biết một số quy ước khi viết một commit.

Commit nên viết một cách tương đối ngắn gọn nhưng phải đủ dễ hiểu cho người đọc. Cụ thể format như sau:

- Phần chủ đề: Thông tin cơ bản về thay đổi
- Dòng trắng
- Phần body: lý do thay đổi nội dung cụ thể thay đổi.

Nhưng không có nghĩa là mọi commit đều cần có format như vậy một số commit đơn giản chỉ cần dòng đầu tiên là đủ.
Sau đây là format chi tiết cho các dòng:

### Phần chủ đề

[Chủ đề] tóm tắt

Các Chủ đề thường dùng là:

- Fix
- add: thêm chức năng mới
- modify: sửa chữa nâng cấp tính năng
- remove: xóa file

Viết hóa chữ cái đầu của chủ đề loại. Kết thúc dòng một không nên là dấu chấm câu để tiết kiệm ký tự. Độ dài không nhiều hơn 50 ký tự lý do nên để ngắn hơn nhiều hơn  50 ký tự vì khi hiện thị git sẽ tự cắt đi những ký tự sau ký tự 50 và thay vào đó là dấu ... như hình sau:
![50](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/50.png)

Tóm tắt: tóm tắt nội dung thay đổi của commit

### Phần body

git không tự động wrap text nên bạn cần làm việc đó một cách thủ công. Khuyến cáo 72 ký tự một dòng là hợp lý(do một terminal có 80 cột sử dụng 4 cột để căn lề trái, 4 cột để căn lề phải vậy còn 72 ký tự).

Phần này trình bày cụ thể về thay đổi như tại sao lại thay đổi và thay đổi như thế nào.



