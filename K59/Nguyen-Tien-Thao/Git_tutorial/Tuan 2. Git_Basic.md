# Basic Git

Trong bài báo cáo ta sẽ đi làm rõ một số vấn đề cơ bản về git. Cụ thể git là gì, sự khác biệt của git với các hệ thốn quản lý phiên bản khác, các chức năng cơ bản mà git cung cấp cho lập trình viên và cách sử các chức năng đó trong một dự án thực tế. Báo cáo sẽ chỉ nêu những chức năng thường được sử dụng ở mức cơ bản.

## Chương 1: Git là gì

Trước khi tìm hiểu git là gì ta cần tìm hiểu về Quản lý phiên bản (Version Control). Vậy Quản lý phiên bản là gì ?

### 1.Version Control

Version Control là một hệ thống để lưu lại thay đổi của các file hoặc hệ thống file theo thời gian để bạn có thể gọi lại ( khôi phục lại) một phiên bản trước đó. Version Control System(VCS) là một hệ thống cực kỳ thông minh giúp bạn làm việc đó. VCS cho phép bạn khôi phục lại một file về phiên bản trước đó, khôi phục cả dự án về phiên bản trước đó, so sánh sự thay đổi theo thời gian của từng file cũng như cả dự án, theo dõi xem ai là người thay đổi cuối cùng và nhiều hơn thế nữa.
Sử dụng VCS đồng nghĩa với việc bạn có thể thay đổi các file một các thoải mái và dễ dàng có thể quay trở lại phiên bản trước đã commit một cách dễ dàng.

Có ba hệ thống quản lý phiên bản là:

- Hệ thống quản lý phiên bản cục bộ (Local Version Control System)
- Hệ thống quản lý phiên bản tập trung (Central Version Control System)
- Hệ thống quản lý phiên bản phân tán (Distributed  Version Control System)

Sau đây ta sẽ đi làm rõ về ba hệ thống này.

#### 1.1 Local Version Control System

Đa số mọi người quản lý phiên bản bằng cách copy file vào một thư mục khác để lưu trữ (Có thể đánh dấu theo thời gian). Nó là một cách làm rất thông dụng bởi nó đơn giản nhưng nó dễ gặp lỗi. Bạn sẽ rất dễ quên rằng mình đang ở trong thư mục nào hay vô tính chỉnh sửa file hoặc chép nhầm file mà bạn không mong muốn.

Để tránh những lỗi có thể do cách làm trên các lập trình viên đã phát triển Local VCSs, một kho dữ liệu đơn giản để lưu trữ tất cả các thay đổi dưới sự kiểm soát thay đổi.

![Local VCS](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/Local%20VCS.PNG)

##### 1.2 VCS Tập trung

Một vấn đề mà các lập trình viên thường gặp phải ngoài việc lưu trữ các phiên bản là hợp tác cùng làm việc với lập trình viên trong một dự án. Và VCS tập trung được phát triển để giải quyết vấn đề này. Hệ thống có một server để duy trì các phiên bản của file và danh sách các người dùng có quyền thay đổi các file trên hệ thống tập trung đó. Những năm gần đây VCS tập trung đã trở thành chuẩn của VCS.
![VCS tập trung](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/VCS%20t%E1%BA%ADp%20trung.PNG)

Mô hình này cung cấp rất nhiều lợi thế so với việc quản lý cục bộ. Cụ thể là người dùng có thể biết một phần nào đó của những việc người khác đang làm trong dự án. Người quản lý dự án có quyền quản lý ai có thể làm gì theo ý muốn.

Bên cạnh đó hệ thống cũng có những bất cập nhất định. Cụ thể như sự cố khi máy chủ tập trung gặp phải. Ví dụ như nếu máy chỉ bị sập thì các người dùng không thể cộng tác với những người dùng khác. Nếu ổ cứng chưa dữ liệu tập trung bị hỏng và các sao lưu dự phòng chưa được thực hiện tới thời điểm đó và người dùng sẽ mất toàn bộ dữ liệu của dự án đó ngoại trừ các phiên bản đã được lưu cục bộ trên máy của mỗi người.

#### 1.3 VCS Phân tán (Distributed Version Control Systems DVCS)

Trong các DVCS các máy khách không chỉ sao chép về máy cục bộ các phiên bản mới nhất của các tệp tin mà sao chép toàn bộ repository. Điều này giúp cho khi máy chủ quản lý phiên bản bị sập thì bất kỳ máy khách nào cũng có thể dùng để sap chép ngược lại để khôi phục lại hệ thống.

![DVCS](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/DVCS.PNG)

### 2 Git cơ bản

#### 2.1 Điểm khác biệt giữa Git và các VCS khác

- Các hệ thống VCS khác mỗi khi người sử dụng commit VCS sẽ lưu lại toàn bộ các file. Còn với Git hệ thống sẽ lưu lại một **ảnh chụp(snapshot)** các file. Với những file đã thay đổi git sẽ tạo một phiên bản mới còn với những file không thay đổi git sẽ ánh xạ file đó vào file cũ đã không thay đổi.
- Phần lớn các thao tác trên git diễn ra cục bộ: Điều này có nghĩa là người dùng có thể thay đổi và commit thay đổi của mình ngay cả khi không có kết nối với mạng. Sau khi người dùng kết nối với mạng git sẽ tự đồng bộ với hệ thông. Điều này giúp người dùng có thể làm việc trong điều kiện không có kết nối internet.

#### 2.2 Ba trạng thái của Git

Mỗi tệp tin trong git được quản lý dựa trên 3 trạng thái:

- Modified: Trạng thái tệp tin đã thay đổi nhưng chưa được commit và cơ sở dữ liệu.
- Staged : Trạng thái bạn đã đánh dấu là sẽ commit phiên bản hiện tại vào lần commit sắp tới.
- Committed: Trạng thái dữ liệu đã được lưu trữ an toàn trong cơ sở dữ liệu

Tương ứng với ba trạng thái là ba phần riêng biệt của một dự án sử dụng Git:

- Thư mục git : Tương ứng với trạng thái commit. Nơi lưu trữ các metadata và cơ sở dữ liệu của dự án. Là phần được sao lưu về khi bản clone một dự án.
- Khu vực tổ chức(staging area): Tương ứng với trạng thái staged. B
- Thư mục làm việc: Tương ứng với trạng thái modified. Bản sao một phiên làm việc của dự án. Những tệp tin được pull về từ cơ sở dữ liệu và lưu trong ổ cứng để có thể sử dụng và chỉnh sủa.

![Ba trạng thái Git](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/Ba%20tr%E1%BA%A1ng%20th%C3%A1i%20Git.PNG)

Tiến trình công việc cơ bản của Git:

- Bạn thay đổi các tệp tin trong thư mục làm việc. Trạng thái modified
- Bạn tổ chức các tệp tin, tạo ảnh mới của tệp tin đó vào thư mục staged. Trạng thái stagad. Tương ứng với câu lênh : "Git add`
- Bạn commit ảnh của các tệp tin trong khu vực staged sẽ được lưu trữ vĩnh viện vào thư mục git. Tương ứng với câu lệnh :`Git commit -m "status"`

## Chương 2: Sử dụng cơ bản git

### 1. Cài đặt git

Để có thể sử dụng git việc đầu tiên bạn cần làm là cài đặt nó. Có nhiều cách để cài đặt trong bài sẽ chủ trình bày hai cách chủ yếu và được sử dụng nhiều nhất là cài đặt từ mã nguồn hoặc là cài đặt từ một gói có sẵn tùy thuộc vào hệ điều hành của bạn.

#### 1.1 Cài đặt từ mã nguồn

Để cài đặt được git bạn cần có các thư viện mà git sử dụng như: curl, zlib, openssl và libiconv.
Nếu bạn đang sử dụng một hệ điều hành có sử dụng yum hoặc apt-get bạn có thể sử dụng các câu lệnh sau để cài đặt các thư viên cần thiết.

```bat
$ yum install curl-devel expat-devel gettext-devel \
  openssl-devel zlib-devel

$ apt-get install libcurl4-gnutls-dev libexpat1-dev gettext \
  libz-dev libssl-dev

```

Sau khi đã cài đặt các thư viện cần thiết bước tiếp theo là download bản mới nhất của git tại website:

`http://git-scm.com/download`

Sau đó tại thư mục đã download bạn bật terminal và gõ các lệnh sau để dịch và cài đăt:

```bat
$ tar -zxf git-1.7.2.2.tar.gz
$ cd git-1.7.2.2
$ make prefix=/usr/local all
$ sudo make prefix=/usr/local install

```

Vậy là đã cài thành công git và bạn có thể bắt đầu sử dụng. Trong quá trình sử dụng bạn cũng có thể cập nhật phiên bản mới của git bằng chinsh nó như sau:

`$ git clone git://git.kernel.org/pub/scm/git/git.git
`

#### 1.2 Cài đặt từ các gói có sẵn theo các hệ điều hành

##### 1.2.1 Cài đặt trên Linux

Nếu bạn sử dụng fedora bạn chỉ cần gõ vào terminal lệnh:
`$ yum install git-core`

Nếu sử dụng ubuntu bạn cũng chỉ cần gõ lệnh sau :
`$ apt-get install git`

##### 1.2.2 Cài đặt trên MAC

Cách đơn giản nhất để cài đặt trên MAC là sử dụng chương trình cài đặt có hỗ trợ giao diện, bạn có thể tải về từ trang web của SourceForge tại địa chỉ :
`http://sourceforge.net/projects/git-osx-installer/`

##### 1.2.3 Cài đặt trên Window

Để cài đặt bản chỉ cần download tệp tin exe tại địa chỉ `http://msysgit.github.com/` và thực hiện cài đặt. Sau đó bạn có cả hai phiên bản là command-line và giao diện chuẩn. Báo cáo sẽ hướng dẫn bạn bằng công cụ có sẵn: msysGit shell(kiểu Unix).

### 2. Cấu hình git lần đâu

Sau khi đã cài đặt git bạn cần tùy biến một số lựa chọn cho môi trường Git của bạn. Bạn sẽ chỉ cần thực hiện việc này một lần và nó sẽ tự nhớ qua các lần cập nhật.

Git cung cấp sẵn git config cho phép bạn xem và chỉnh sửa cấu hình :

- /etc/gitconfig: Chứa giá tri cho tất cả người dùng và kho chứ trên hệ thống.
- ~/.gitconfig: Riêng biệt cho tài khoản của bạn. Bạn có thể chỉ định git đọc và ghi trên tệp này bằng cách sử dụng --global.
- tệp tin config trong thư mục git(.git/config) của bất kỳ kho chứa nào bạn đang sử dụng. Chỉ áp dụng trong mỗi kho chứa. Mỗi cấp sẽ ghi đè giá trị của cấp trước nó vì thế các giá trị trong .git/config sẽ được sử dụng thay vì giá trị trong /etc/gitconfig.

#### 2.1 Cấu hình danh tính

Việc đầu tiên cần làm khi cấu hình git là cấu hình tài khoản và địa chỉ email.  Điều này quan trọng vì git sẽ sử dụng chúng cho mỗi lần commit, những thông tin này sẽ được gắn cố định vào mỗi lần commit. Để cấu hình bạn thực hiên các lệnh sau:

```bat

$ git config --global user.name "<Tên tài khoản>"
$ git config --global user.email <email>

```

Việc cấu hình này chỉ thực hiện một lần duy nhất nếu bạn sử dụng --global. Nếu muốn sử dụng tài khoản và email cho một dự án riêng biệt nào đó bạn có thể thực hiện cầu hình này mà không sử dụng thuộc tính --global.

#### 2.2 Cấu hình trình soạn thảo

Bạn sử dụng câu lệnh sau để cấu hình trình soạn thảo cho git:
`$ git config --global core.editor <tên trinh soạn thảo>`

ví dụ :
`$ git config --global core.editor emacs`

#### 2.3 Cấu hình công cụ so sánh thay đổi

`$ git config --global merge.tool <tên công cụ>`
ví dụ :`$ git config --global merge.tool vimdiff`

 ngoài ra git còn chấp nhận nhiều công cụ như:kdiff3, tkdiff, meld, xxdiff, emerge, vimdiff, gvimdiff, ecmerge, và opendiff

#### 2.4 Sau khi đã cấu hình bạn nên kiểm tra lại cấu hình

câu lệnh: `git config --list`

kết quả :

```txt
user.name=Nguyễn Tiến Thạo
user.email=nguyentienthao96@gmail.com
core.repositoryformatversion=0
core.filemode=false
core.bare=false
core.logallrefupdates=true
core.symlinks=false
core.ignorecase=true
remote.origin.url=https://github.com/NTT-TNN/Basic_knowledge.git
remote.origin.fetch=+refs/heads/*:refs/remotes/origin/*
branch.master.remote=origin
branch.master.merge=refs/heads/master

```

### 3. Tạo một kho chứ git

Có hai cách để tạo một kho chứ git

- Tạo kho chứa từ thư mục cũ
- Sao chép một kho chứa đã tồn tại

#### 3.1 Tạo kho chứa từ một thư mục cũ

Nếu bạn đang có một thư mục chứa dự án mà bạn muốn theo dõi dự án này trong git. Bạn gõ lệnh sau vào terminal tại thư mục chứa dự án đó.

`$ git init`

Lệnh này tạo ra một thư mục mới có tên `.git` thư mục này chưa tất cả các tệp tin cần thiết cho kho chứa git. Bây giờ chưa hề có gì trong dự án của bạn được theo dõi hết.

Nếu bạn muốn kiểm soát phiên bản cho các tệp tin sẵn có bạn nên thiết lập theo dõi các tệp tin đó và thực hiện commit lần đầu tiên. Để có thể theo dõi tệp tin bạn sử dụng lệnh `git add`
. Để có thể commit những thay đổi bạn có thể sử dụng lệnh : `git commit -m "các thay đổi"`

#### 3.2 Sao chép một kho chứa đã tồn tại

Nếu như bạn muốn sao chép một bản sao của một kho chứa đã có sẵn ví dụ như dự án bạn muốn đóng gói vào bạn cần sử dụng câu lệnh :`git clone url`
ví dụ:` git clone https://github.com/NTT-TNN/Basic_knowledge.git
`
Sau đó thư mục chứa kho dữ liệu bạn sao chép sẽ được lưu tại thư mục bạn thực hiện câu lệnh.

#### 3.3 Ghi lại thay đổi vào kho chứa

Bạn đã tạo được một kho chứa và bản sao dữ liệu để làm viêc. Bây giờ bạn sẽ làm việc trên bản sao đó và một khi đạt đến một trạng thái nào mà bạn muốn lưu vào ghi lại bạn sẽ cần commit chúng vào kho chứa.

Mỗi tệp tin trong thư mục làm việc của bạn sẽ có thể ở một trong hai trạng thái:

- Tracked: là tệp tin đã có trong danh sách theo dõi trước. Chúng có thể ở một trong ba trạng thái: Modified, unmodified và staged.
- Untracked: Các file còn lại. Cụ thể là các file trong thư mục làm việc mà không có ảnh(lần commit) trước hoặc các file không thuộc vùng staging. Ban đầu khi bạn clone một kho chứa về tất cả các file sẽ ở trạng thái tracked và unmodified vì bạn mới tải chúng về và chưa thực hiện thay đổi nào.

Khi bạn chỉnh sửa các tệp tin chúng sẽ chuyển sang trạng thái modified sau đó nếu bạn muốn commit các tệp tin đó bạn cần đưa chúng vào khu vực stage và thực hiện commit. Cứ như vậy lặp lại.
![Lifecycle](https://raw.githubusercontent.com/NTT-TNN/Basic_knowledge/master/images/File%20Status%20LifeCycle.PNG)

##### **Để kiểm tra trạng thái của tệp tin bạn có thể sử dụng lệnh`git status`**

```bat
$ git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#   new file:   README
#
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#
#   modified:   benchmarks.rb
#
```

Để theo dõi một tệp tin mới bạn sẽ sử dụng lệnh `git add <teentfile>` khi đó file sẽ đưa vào khu vực staging và sẽ được commit vào thư mục git tại lần commit sau.

Quản lý tệp tin đã thay đổi:
Giả sử bạn có một file muốn thay đổi và bạn tiến hành thay đổi file đó. Sau khi thay đổi xong bạn thực hiện câu lệnh `git status` và bạn thấy kết quả như sau:

```bat
$ git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#   new file:   README
#
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#
#   modified:   benchmarks.rb
#
```

Điều này có nghĩa là tệp tin này đã được theo dõi trước đó và đã có thay đổi trong thư mục làm việc nhưng chưa được thay đổi tại vùng staging. Để thay đổi vào vùng staging thực hiện cậu lệnh `git add`. Sau đó thực hiện lệnh git status sẽ thấy kết quả là:

```bat
$ git add benchmarks.rb
$ git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#   new file:   README
#   modified:   benchmarks.rb
#
```

Điều này có nghĩa là tệp tin đã được đưa vào vùng staging và đợi đưa vào thư mục git tại lần commit tiếp theo. Bây giờ bạn nhận ra là bạn cần sửa lại một chi tiết nhỏ trước khi commit và thư mục git và bạn sẽ mở file ra và sửa nó. Sau đó bạn chạy `git status` và bạn sẽ thấy kết quả:

```bat
$ vim benchmarks.rb
$ git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#   new file:   README
#   modified:   benchmarks.rb
#
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#
#   modified:   benchmarks.rb
#

```

Bạn sẽ thấy một vấn đề xảy ra đó là file `benchmarks.rb` nằm trong cả hai khu vực là staged và unstaged. Nếu bây giờ bạn thực hiện commit phiên bản đã được staged(tức là phiên bản đã add lần trước ) sẽ được thêm vào thư mục git chứ không phải phiên bản bạn đã sửa tại thư mục làm việc hiện tại. Điều này có nghĩa là nếu bạn muốn đưa tệp tin đã chỉnh sửa hiện tại vào thư mục git bạn cần thực hiện lệnh `git add`. Sau đó sẽ là  `git commit`.

```bat
$ git add benchmarks.rb
$ git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#   new file:   README
#   modified:   benchmarks.rb
#
```

##### **Bỏ qua các tệp tin**.

Trong một số trường hợp bạn sẽ không muốn đưa một số file vào thư mục git khi đó bạn có thể liệt kê các tệp tin này trong một thư mục có tên là `.gitignore`. Đây là một ví dụ

```bat
$ cat .gitignore
*.[oa]
*~
```

Quy tắc trong file `.gitignore` như sau :

- Dòng trống hoặc bắt đầu với # sẽ được bỏ qua
- Các chuẩn toàn cầu hoạt động tốt hay các biểu thức chính quy(regular expression)
- mẫu có kết thúc bằng đấu / chỉ định một thư mục.
- bạn có thể sử dụng mẫu phủ định bằng cách thêm ! đằng trước.

##### **Xem các thay đổi một cách chi tiết**

Nếu `git status` chỉ cung cấp cho bạn các thay đổi một cách chung chung. Thì git cũng cung cấp một câu lệnh khác giúp bạn xem rõ hơn bạn đã thay đổi cái gì đó là `git diff`. Cụ thể `git diff` có hai dạng là:

- `git diff` không sủ dụng tham số :Câu lệnh này so sánh những gì bạn đã thay đổi tại thư mục làm việc với những gì bạn đã thêm vào khu vực staging. ví dụ :

```bat
$ git diff
diff --git a/benchmarks.rb b/benchmarks.rb
index 3cb747f..da65585 100644
--- a/benchmarks.rb
+++ b/benchmarks.rb
@@ -36,6 +36,10 @@ def main
           @commit.parents[0].parents[0].parents[0]
         end

+        run_code(x, 'commits 1') do
+          git.commits.size
+        end
+
         run_code(x, 'commits 2') do
           log = git.commits('master', 15)
           log.size
```

- `git diff --cached` hoặc `git diff --staged`(cho phiên bản từ 1.6.1 trở đi): Lệnh này so sánh những thay đổi giữa file được đưa vào stage và file đã được commit. Ví dụ:

```bat
$ git diff --cached
diff --git a/README b/README
new file mode 100644
index 0000000..03902a1
--- /dev/null
+++ b/README2
@@ -0,0 +1,5 @@
+grit
+ by Tom Preston-Werner, Chris Wanstrath
+ http://github.com/mojombo/grit
+
+Grit is a Ruby library for extracting information from a Git repository
```

##### **Bỏ qua khu vực tổ chức:**

Mặc dù khu vực tổ chức là một cách làm việc hay nhưng đôi khi chúng khiến quy trình làm việc trở nên phức tạp. Nếu bạn muốn bỏ qua bước này git cung cấp sắn cho bạn một lỗi tắt. Chỉ cần thêm `-a` vào trước khi thực hiện `git commit` git sẽ tự động thêm tất cả các file đã được theo dõi trước đó vào thư mục git, cho phép bạn bỏ qua bước `git add`. ví dụ

```bat
$ git status
# On branch master
#
# Changes not staged for commit:
#
#   modified:   benchmarks.rb
#
$ git commit -a -m 'added new benchmarks'
[master 83e38c7] added new benchmarks
 1 files changed, 5 insertions(+), 0 deletions(-)
 ```

##### **Xem lịch sử commit:**

Sau khi đã commit rất nhiều lần hoặc bạn sao chép một kho chứa với các commit có sẵn. Chắc chắn bạn sẽ muốn xem lại lịch sử các commit và một cách đơn giản nhất là sử dụng lệnh `git log`. Ví dụ

```bat

$ git log
commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    changed the version number

commit 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Sat Mar 15 16:40:33 2008 -0700

    removed unnecessary test code

commit a11bef06a3f659402fe7563abf99ad00de2209e6
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Sat Mar 15 10:31:28 2008 -0700

    first commit
```

`git log` mặc định sẽ hiện thị các commit theo trình tự thời gian từ mới nhất. `git log` có nhiều tham số tùy với nhiều thứ mà người dùng muốn hiển thị.

##### **Xóa tệp tin**

Để xóa một tệp tin khỏi git, bạn phải xóa nó khỏi danh sách các tệp tin được theo dõi(chính xác là xóa các tệp tin trong vùng stage) sau đó commit. Lệnh `git rm` thực hiện việc đó và nó cũng xóa tệp tin đó khỏi thư mục làm việc vì thế bạn sẽ không còn thấy nó như một tệp tin chưa được theo dõi( Có nghĩa là nó đã được xóa hoàn toàn).

Sau khi thực hiện lệnh ta sẽ được kết quả:

```text

On branch master
Your branch is ahead of 'origin/master' by 2 commits.
  (use "git push" to publish your local commits)
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

deleted:    testrm.txt

```

Nếu bạn chỉ xóa tệp tin trong thư mục làm việc nó sẽ hiện thị trong phần thay đổi mà chưa được tổ chức để commit.

```txt

Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

modified:   Tuan 2/Tuan 2. Git_Basic.md
deleted:    test.txt

no changes added to commit (use "git add" and/or "git commit -a")

```

Git cũng có chức năng cho phép bạn giữ một file trong thư mục làm việc mà không đưa vào khu vực stage(không muốn theo dõi nó):
`$ git rm --cached readme.txt`

##### **Làm việc từ xa**




