
# Chương 1 Sơ lược về Git

## 1.1 Git là gì?
Git là một trong những hệ thống quản lý phiên bản phân tán (VCS)


## 1.2 Git với VCS khác

Git khác với các VCS khác ở chỗ Git "nghĩ" về dữ liệu.Phần lớn các hệ thống VCS khác lưu trữ thông tin dưới dạng danh sách các tập tin được thay đổi. Các hệ thống này (CVS, Subversion, Perforce, Bazaar,...) coi thông tin được lưu trữ như là một tập hợp các tập tin và các thay đổi được thực hiện trên mỗi tập tin theo thời gian như hình sau:


![hinh14](http://i.imgur.com/IlKcZJu.png)

Trong khi Git coi dữ liệu của mình giống như 1 tập các ảnh của một hệ thống tập tin nhỏ. Việc commit hay lưu lại trạng thái hiện tại của dự án được Git chụp lại một bức ảnh, bức ảnh đó ghi lại nội dung của tất cả các tập tin tại thời điểm đó và tạo ra một tham chiếu tới ảnh đó. Đối với những tập tin không có sự thay đổi, Git sẽ không lưu lại nó nữa mà chỉ tạo một liên kết chỉ tới tập tin gốc đã tồn tại trước đó:

![hinh13](http://i.imgur.com/6d38vo7.png)

## 1.3 Ba trạng thái



Ba trạng thái của Git gồm: Modified, Staded, Commit.Committed có nghĩa là dữ liệu đã được lưu trữ một cách an toàn trong cơ sở dữ liệu. Modified có nghĩa là bạn đã thay đổi tập tin nhưng chưa commit vào cơ sở dữ liệu. Và staged là bạn đã đánh dấu sẽ commit phiên bản hiện tại của một tập tin đã chỉnh sửa trong lần commit sắp tới

Ứng với mỗi trạng thái tương ứng có 1 khu làm việc cho 1 dự án sử dụng Git:
+ Modified <-> Thư mục làm việc
+ Staged <-> Khu vực tổ chức
+ Commit <-> Thư mục git

Hình mô tả các khu vực làm việc của Git

![h12](http://i.imgur.com/6Z2eX6v.png)

Thư mục Git là nơi Git lưu trữ các "siêu dữ kiện" (metadata) và cơ sở dữ liệu cho dự án của bạn. Đây là phần quan trọng nhất của Git, nó là phần được sao lưu về khi bạn tạo một bản sao (clone) của một kho chứa từ một máy tính khác.

Thư mục làm việc là bản sao một phiên bản của dự án. Những tập tin này được kéo về (pulled) từ cơ sở dữ liệu được nén lại trong thư mục Git và lưu trên ổ cứng cho bạn sử dụng hoặc chỉnh sửa.

Khu vực khán đài là một tập tin đơn giản được chứa trong thư mục Git, nó chứa thông tin về những gì sẽ được commit trong lần commit sắp tới. Nó còn được biết đến với cái tên "chỉ mục" (index), nhưng khu vực tổ chức (staging area) đang dần được coi là tên tiêu chuẩn.

Tiến trình công việc (workflow) cơ bản của Git:

1. Bạn thay đổi các tập tin trong thư mục làm việc.
2. Bạn tổ chức các tập tin, tạo mới ảnh của các tập tin đó vào khu vực tổ chức.
3. Bạn commit, ảnh của các tập tin trong khu vực tổ chức sẽ được lưu trữ vĩnh viễn vào thư mục Git.

## 1.4 Cài đặt Git trên Unbuntu 
1. Cách 1: Cài đặt từ mã nguồn

+ Cài các thư viện cần thiết cho Git
  >$ apt-get install libcurl4-gnutls-dev libexpat1-dev gettext libz-dev libssl-dev

+ Tải phiên bản mới nhất của Git từ website:
  >http://git-scm.com/download

+ Dịch và cài đặt Git:
  >$ tar -zxf git-2.7.4.tar.gz

  >$ cd git-2.7.4

  >$ make prefix=/usr/local all

  >$ sudo make prefix=/usr/local install

+ Nếu bạn muốn tải về các bản cập nhật của Git thì dùng câu lệnh sau:
  >$ git clone git://git.kernel.org/pub/scm/git/git.git

2. Cách 2: Thông qua gói cái đặt trên HDH của bạn
   >$ apt-get install git 

  + Kiểm tra lại xem git đã cài đặt thành công hay chưa ta dùng lệnh:
    >git --version

## 1.5 Cấu hình Git lần đầu

Để tùy biến 1 số lựa chọn cho môi trường của Git bạn cần phải cấu hình Git lần đầu (chúng sẽ được ghi nhớ qua các lần cập nhật, bạn cũng có thẻ thay đổi chúng bằng việc chạy lại các lệnh):

1. Cấu hình danh tính
   >git config --global user.name <Tên tài khoản github của bạn>

   >git config --global user.email <Email bạn dùng để đăng ký github>
  
    Ví dụ: Tên tài khoản github của mình là Monlight, email sang11a1hbt@gmail.com thì mình gõ như sau: 
   - git config --global user.name "xuho"
   - git config --global user.email "sang11a1hbt@gmail.com"

2. Kiểm tra lại cấu hình 
   >git config --list

3. Cấu hình tài khoản

-  Để sử dụng git bạn cần có 1 tài khoản, bạn truy cập vào trang chủ của git để đăng ký qua link sau:
   >http://github.com

   ![h11](http://i.imgur.com/Lr1fRw2.png)

# Chương 2 Cơ bản về Git 
## 2.1 Taọ một kho chứa Git 
- Tạo một kho chứa mới trên máy tính của bạn,sử dụng lệnh: 
  > $ git init <tên kho chứa>
  
  Ví dụ: Taọ một kho chứa có tên là mygit
  > $ git init mygit

  Nó sẽ tạo một kho chứa ngay thư mục chúng ta đang đứng.

- Sao chép một kho chứa Git đã có sẵn 

   Sử dụng lệnh git clone [url]

   Ví dụ: bạn gõ lệnh
   >$ git clone https://github.com/Monlight/demo_git

   thì nó sẽ tạo ra kho chứa với tên thư mục là demo_git trên máy của bạn để chứa bản sao dữ liệu của kho Git đó

-   Khi bạn sao chép 1 kho chứa về máy và muốn đặt tên lại kho chứa thì bạn dùng câu lệnh:
    >$ git clone [đường dẫn] <tên mới của thư mục>

    >Ví dụ: $ git clone https://github.com/Monlight/demo_git myGit


## 2.2 Ghi laị thay đổi vào kho chứa
Ta có một bản sao dữ liệu của dự án để làm việc. Các tập tin trong thư mục làm việc của ta có thể ở 1 trong 2 trạng thái: tracked hoặc untracked.
+ Tập tin tracked là tập tin được theo dõi, nó đã nằm ở khu vực tổ chức (staging area) hoặc đã có mặt trong ảnh (đã được commit) ở lần trước, khi chúng ta làm việc với chúng, chúng có thể ở 1 trong 3 trạng thái sau: unmodified, modified, hoặc stage
+ Tập tin untracked là các tập tin còn lại trong thư mục làm việc của bạn (không có mặt trong ảnh lần trước, chưa được add vào staging area)

Khi bạn chỉnh sửa các tập tin, Git coi là chúng đã bị thay đổi so với lần commit trước đó. Bạn stage các tập tin bị thay đổi này và sau đó commit tất cả các thay đổi đã được staged (tổ chức) đó, và quá trình này cứ thế lặp đi lặp lại tạo ra 1 vòng đời trạng thái cho tập tin như hình sau:

![h2](http://i.imgur.com/Lth9CPR.png)

Ban đầu khi bạn tạo bản sao (clone) 1 kho chứa về máy và đổi tên thư mục thành myGit chẳng hạn thì tất cả các tập tin trong myGit sẽ ở trạng thái "tracked" và "unmodified", điều này là tất nhiên vì bạn chưa thực hiện bất kì thay đổi nào.

### Kiểm tra trangj thái của tập tin
Bạn có thể xem trạng thái này của nó bằng lệnh

- $ git status 

  ![h4](http://i.imgur.com/8XJCk9G.png)

nó sẽ hiển thị và cho bạn biết là "nothing to commit, working directory clean".

Bây giờ, ở trong thư mục đó, bạn tạo 1 file mới có tên là README.md, thì file này hiện tại đang ở trạng thái untracked, sử dụng '$ git status' kiểm tra bạn sẽ thấy README.md nằm trong trường Untracked files, và nó có màu đỏ:

- ![h5](http://i.imgur.com/1f8txKu.png)

### Theo dõi  tập tin mới
Để có thể theo dõi tập tin mới tạo này, bạn dùng lệnh
- $ git add README.md

  ![h6](http://i.imgur.com/OsTmHbC.png)

Khi đó README.md đã được stage và nó nằm trong danh sách các thay đổi chuẩn bị commit, nếu bạn tiến hành commit thì Git sẽ chụp bức ảnh mới để lưu lại phiên bản này, và README.md sẽ trở về trạng thái unmodified.

### Quản lý tập tin đã thay đổi
Điều gì xảy ra nếu ta muốn chỉnh sửa nội dung README.md trước khi commit?

Bạn có thể dùng vim, sau đó xem lại trạng thái của README.md:
- $ vim README.md
- $ git status

  ![h7](http://i.imgur.com/NvEM5vb.png)

Điều gì xảy ra thế này?

- Bây giờ README.md của tôi nằm trong cả 2 danh sách stage và unstage (với trạng thái modified) là sao

Vậy nếu bây giờ tôi tiến hành commit, thì thay đổi trong tập tin README.md sẽ được lưu đúng không? 
- Không hề, Git sẽ không lưu cho bạn đâu và kết quả là tập README.md của bạn sẽ chẳng có thay đổi gì cả bởi vì Git tổ chức tập tin chính là lúc bạn chạy lệnh ' git add '

Thế là tôi phải chạy lại lệnh ' git add ' sau đó mới được commit à?

- Đúng thế, làm đi - kết quả thu được:

  ![h8](http://i.imgur.com/HV1tkaj.png)
 

### Commit

Commit là để lưu lại dữ liệu của file vào cơ sở dữ liệu của máy bạn. 
+ Bạn chỉ có thể commit đối với các tập tin đã được tổ chức (nghĩa là các file đã nằm ở khu vực staging area sau khi thực hiện lệnh ' git add ')

  

+ Những gì chưa được tổ chức - bất kỳ tập tin nào mới được tạo ra mà chưa được ' git add ' hoặc đối với những file mà thuộc cả 2 khu vực staged và unstaged (nghiã là sau khi chúng đó được ' git add ' hoặc đã được commit bạn lại thực hiện sửa đổi nó mà chưa ' git add ' lại) - sẽ không được commit

+ Bạn có thể commit trực tiếp thông điệp bằng cách thêm vào sau commit cờ -m
  >Ví dụ: $ git commit -m "Noi dung commit"

Vấn đề đặt ra là, nếu tôi có rất nhiều file mới được tạo ra, hay có sửa đổi trên nhiều file đã được staged và tôi muốn commit hết chúng. Khi đó việc tổ chức (stage) chúng khiến tôi tốn thời gian, công việc trở lên phức tạp. Vậy có cách nào giải quyết không?

- Hỏi thông minh thế? -Câu trả lời của Git là "lối tắt". Bạn cần thêm cờ  -a  khi thực hiện ' git commit '. Git sẽ tự động thêm theo dõi cho tất cả các tập tin của bạn trước khi thực hiện commit (như thế nó sẽ cho bạn bỏ qua bước git add)
  >$ git commit -a -m "Noi dung commit"

   ![gitcommit](http://i.imgur.com/mMXuAhj.png)

### Xóa tập tin


+ Nếu bạn chỉ đơn giản xóa tập tin ra khỏi thư mục làm việc, kết quả trả về là "Changes not staged for commit: - Thay đổi không được tổ chức để commit", nếu bạn cứ tiếp tục commit thì kết quả trả về là "no changes added to commit
":

    ![h9](http://i.imgur.com/GypfWCd.png)

+ Để xóa tập tin ra khỏi Git, bạn phải xóa nó khỏi khu vực tổ chức - sử dụng lệnh  ' git rm ' lệnh này cũng sẽ xóa tập tin đó ra khỏi thư mục làm việc luôn, và bạn sẽ không thấy nó trong Git ở phiên làm việc sau nữa:

    ![h10](http://i.imgur.com/Ys14jeZ.png)

+ Nếu bạn muốn giữ tập tin đó ở trong thư mục làm việc và bạn chỉ muốn xóa chúng khỏi khu vực tổ chức thì cần dùng thêm tùy chọn --cached:
  >$ git rm --cached README.md

### Đổi tên tập tin

Dùng lệnh git mv để đổi tên tập tin từ file01.md sang newfile.md:
 >$ git mv file01.md newfile.md
