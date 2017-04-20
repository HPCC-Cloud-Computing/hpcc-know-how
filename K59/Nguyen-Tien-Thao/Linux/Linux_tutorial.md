# Basic Linux

Linux có giao diện đồ họa cho người dùng và nó cũng làm việc tốt như nhưng GUI tương tự khác trên Windows và OSX. Vậy tại sao ta cần tìm hiểu về command line trong linux. Tìm hiểu về command line không có nghĩa là sẽ từ bỏ hẳn không sử dụng GUI. Có một số task sẽ thích hợp với GUI (chỉnh sửa video), một số khác lại thích hợp với command line(quản lý file). Trong bài này sẽ tập trung vào cách sử dụng command line trong linux.

## Command line là gì

Command line(hoặc terminal) là một giao diện hệ thống dựa trên text. Bạn có thể nhập vào một câu lệnh bằng bàn phím và phản hồi cũng sẽ được hiện thị cho bạn dưới dạng text.

Command thường hiện thị cho bạn bằng một dấu nhắc(prompt). Khi bạn gõ câu lệnh của bạn sẽ hiện thị sau dấu nhắc. Ví dụ:

```sh
nguyentienthao@ntttnn-laptop:/$ ls -l /home/nguyentienthao/
total 52
drwxrwxr-x  2 nguyentienthao nguyentienthao 4096 Th02  2 17:12 build
drwxr-xr-x  2 nguyentienthao nguyentienthao 4096 Th04 13 19:41 Desktop
drwxr-xr-x 16 nguyentienthao nguyentienthao 4096 Th03 25 10:02 DHBKHN
drwxrwxr-x  4 nguyentienthao nguyentienthao 4096 Th02 15 17:58 distributted
drwxr-xr-x  7 nguyentienthao nguyentienthao 4096 Th04 19 10:02 Downloads
nguyentienthao@ntttnn-laptop:/$
```

Làm rõ về ví dụ:

- dòng thứ nhất:bắt đầu bằng một dấu nhắc `nguyentienthao@ntttnn-laptop:/$`. Sau dấu nhắc là bắt đầu câu lệnh `ls`. Sau đó là một vài tham số cho câu lệnh đó(giữa câu lệnh và tham số cũng như giữa các tham số với nhau được phân tách bằng dấu cách(space)). Mỗi câu lệnh khác nhau sẽ có các tham số khác nhau ví dụ như trong câu lệnh trên tham số đầu tiên `-l` là một tham số tùy chọn(có thể có hoặc không). Các tùy chọn được thêm vào thường để thay đổi cách hoạt động của câu lệnh. Tùy chọn thường được đặt sau câu lệnh và trước các tham số khác, thường bắt đầu bằng dấu `-`.
- dòng thứ 2-5: Đầu ra của câu lệnh. Các câu lệnh thường cho ra các kết quả và được hiện thi sau câu lệnh
- Dòng cuối cùng: Dấu nhắc cho câu lệnh tiếp theo. Sau khi câu lệnh được hoàn thành dấu nhắc cho câu lệnh tiếp theo sẽ được hiện thi ra. Nếu không xuất hiện dấu nhắc câu lệnh mới có nghĩa là câu lệnh cũ đang được thực hiện.

## Basci linux

### `pwd`( Print Working Directory)

Câu lệnh cho biết bạn thư mục bạn đang làm việc. Bạn sẽ nhận thấy hầu hết các câu lệnh trong linux là viết tắt của từ hay cụm từ miêu tả chúng điều này giúp việc ghi nhớ trở nên dễ dàng

ví dụ:

```sh
nguyentienthao@ntttnn-laptop:~$ pwd
/home/nguyentienthao

```

Lưu ý: Có rất nhiều câu lệnh trong linux thực hiện dựa trên nơi thực hiện câu lệnh đó. Đôi khi bạn chuyển qua lại giữa các thư mục mà quên mất mình đang ở đâu nên chắc chắn rằng bạn thực hiện câu lệnh tại đúng nơi cần thực hiện.

### `ls`

Câu lệnh này cho biết những thứ có trong thư mục hiện tại

ví dụ:

```sh
nguyentienthao@ntttnn-laptop:~$ ls
build    distributted  official_bktranslator  Public           Videos
Desktop  Downloads     Pictures               PycharmProjects
DHBKHN   key.pem       Project 1              stock_market

```

Câu lệnh ls thực hiện không có tham số thêm sẽ cho ta danh sách thu gọn. Ngoài ra ta có thể thêm một số tham số vào sau như sau theo cú pháp sau: `ls [options] [location]`. Trong cú pháp đó những tham số đặt trong dấu `[]`là tham số tùy chọn(có thể có hoặc không).

ví dụ câu lệnh: `ls -l`

```sh
nguyentienthao@ntttnn-laptop:~$ ls -l
total 52
drwxrwxr-x  2 nguyentienthao nguyentienthao 4096 Th02  2 17:12 build
drwxr-xr-x  2 nguyentienthao nguyentienthao 4096 Th04 19 18:54 Desktop
drwxr-xr-x 16 nguyentienthao nguyentienthao 4096 Th03 25 10:02 DHBKHN
drwxrwxr-x  4 nguyentienthao nguyentienthao 4096 Th02 15 17:58 distributted
drwxr-xr-x  8 nguyentienthao nguyentienthao 4096 Th04 19 19:50 Downloads
drwxrwxr-x  4 nguyentienthao nguyentienthao 4096 Th12 10 23:04 key.pem
drwxrwxr-x 11 nguyentienthao nguyentienthao 4096 Th02 15 18:34 official_bktranslator
drwxrwxr-x  3 nguyentienthao nguyentienthao 4096 Th03 25 10:59 Pictures
drwxrwxr-x  6 nguyentienthao nguyentienthao 4096 Th12 18 17:35 Project 1
drwxr-xr-x  2 nguyentienthao nguyentienthao 4096 Th11 16 19:22 Public
drwxrwxr-x  3 nguyentienthao nguyentienthao 4096 Th04 18 15:45 PycharmProjects
drwxrwxr-x  7 nguyentienthao nguyentienthao 4096 Th12 10 12:05 stock_market
drwxrwxr-x  2 nguyentienthao nguyentienthao 4096 Th03 25 10:57 Videos

```

Tham số `-l` sẽ đưa ra danh sách cùng với các trường thông tin thêm. Cụ thể ý nghĩa các thông tin đó là: ký tự đầu tiên cho biết là file hoặc thư mục( - hoặc d) 9 ký tự tiếp theo cho biết quyền cho file hoặc thư mục đó. Số tiếp theo là số lượng block. Tiếp theo là nhóm mà file hoặc thư mục thuộc về. Tiếp theo là kích thước file sau nữa là ngày tháng thay đổi. Cuối cùng là tên của file hoặc thư mục.

Ta thấy câu lệnh `ls` có tham số thứ hai là `[location]`. Location ở đây là path(đường dẫn) tới vị trí muốn thực hiện câu lệnh `ls`.

Path trong linux có hai loại: Tuyệt đối và tương đối. Trước khi nói về hai loại đường dẫn ta cần hiểu về cách mà linux tổ chức lưu trữ dữ liệu. Linux lưu trữ các file và thư mục theo dạng cấu trúc cây, vị trí bắt đầu được gọi là `root`(ký hiệu là `/`). 

![Root](http://2.bp.blogspot.com/-ZI4tLeIo24A/UeJQk5sgDeI/AAAAAAAAArs/1zkTUKjLzac/s1600/linux.png)

Bây giờ ta sẽ nói về hai loại đường dẫn trong linux. Thứ nhất đường dẫn tuyệt đối là đường dẫn bắt đầu từ thư mục root(`/`). Ví dụ như trong hình: `/home/jono/photos`. Đường đẫn này được gọi là tuyệt đối do nó luôn xuất phát từ thư mục gốc nên việc đặt nó ở vị trí nào cũng sẽ cho cùng kết quả. Loại đường dẫn thứ hai là đường dẫn tương đối, khác với tuyệt đối đường dẫn tương đối lại phụ thuộc vào vị trí sử dụng nó. Cụ thể đường dẫn tuyệt đối là đường dẫn từ nơi đặt nó đến các file(thư mục) khác trong hệ thống. Ví dụ. Nếu ta đang ở thư mục `jono` ta muốn đi tới thu mục ảnh ta chỉ cần sử dụng đường dẫn: `photos`. Nhưng nếu ta đang ở thư mục `cory` đường dẫn này là không đúng do trong thư mục `cory` không có thư mục `photos`. Để tạo sự linh hoạt cho đường dẫn chúng ta cần biết thêm về một số ký hiệu đặc biệt thường dùng trong đường dẫn:

- ~: nó là ký hiệu dùng thay cho thư mục home của bạn. Ví dụ nếu thư mục home của bạn là `home/nguyentienthao` nếu bạn muốn truy cập vào thư mục `Documents` trong thư mục home của bạn bạn có thể sử dụng cả hai cách đều như nhau: `home/nguyentienthao/Documents` hoặc `~/Documents`
- .(dot): Nó tham chiếu tới thư mục hiện tại của bạn.
- ..(dotdot): Nó tham chiếu tới thư mục cha của thư mục hiện tại. Xét ví dụ trong hình nếu bạn đang ở thư mục `photos`  đường dẫn `../works` sẽ cho phép bạn truy cập thư mục `works` bằng cách đi lên thư mục cha (`jono`) của thư mục `photos` sao đó xuống thư mục `works`.

### cd [location]

Dùng để di chuyển giữa các thư mục

Ví dụ:

```sh
nguyentienthao@ntttnn-laptop:~/Downloads$ cd demo/
nguyentienthao@ntttnn-laptop:~/Downloads/demo$
```

Ta thấy việc gõ một đường dẫn đôi khi rất phức tạp và dễ bị lỗi chính tả. Linux cung cấp một tính năng gọi là `Tab Completion`. Nó giúp việc gõ đường dẫn trở nên đơn giản và thú vị hơn. Cụ thể khi bạn bắt đầu gõ đường dẫn bạn có thể ấn phím `Tab` trên bàn phím linux sẽ tự động điền đường dẫn còn thiếu cho bạn. Ví dụ như khi bạn gõ `cd B` sau đó bạn gõ `Tab` linux sẽ tự thêm vào cho câu lệnh trở thành:
`nguyentienthao@ntttnn-laptop:~/DHBKHN$ cd Basic_knowledge/`. Đôi khi bạn gõ phím Tab mà linux sẽ không tự động điền đường dẫn điều đó có nghĩa không có đường dẫn nào thỏa mãn hoặc có một vài đường dẫn thỏa mãn. Nếu có một vào đường dẫn thỏa mãn khi bạn ấn phím Tab một lần nữa linux sẽ đưa ra danh sách các đường dẫn thỏa mãn như sau: `nguyentienthao@ntttnn-laptop:~$ cd D
Desktop/   DHBKHN/    Downloads/`. Khi đó bạn có thể gõ tiếp tục các ký tự để phân biệt các đường dẫn đó và sử dụng Tab để hoàn thành đường dẫn mình cần.

Trong linux cũng có một số thư mục hệ thống cần chú ý:

- /etc: Chứa các file cấu hình của hệ thống
- /var/log: Chứa các file logs cho các chương trình hệ thống phức tạp.
- /bin: Chứa các chương trình thực thi được sử dụng bởi tất cả người dùng
- /usr/bin: Chứa các chương trình thực thi của người dùng

Trước khi tìm hiểu các câu lệnh thường dùng trong linux chúng ta cần tìm hiểu một vài điểm thú vị của linux:

- Extensionless System: Ta đã biết trong hệ điều hành chẳng hạn như windows mỗi file sẽ có một một định dạng(loại của file) nhất định và loại của file được quyết định bởi 2-4 ký tự cuối cùng của tên file(sau dấu chấm). Ví dụ:

```txt
file.exe - File thực thi hoặc chương trình
file.txt - File text
file.png, file.gif, file.jpg - ảnh.
```

Khác với các hệ thống khác linux không sử dụng 2-4 ký tự đó để xác định loại file mà linux xem nội dung bên trong file để xác định loại của file. Ví dụ bạn có một bức ảnh lưu với tên `anh.png` và sau đó bạn đổi tên thành `anh.txt` hệ thống linux sẽ vẫn coi file đó là một ảnh. Vậy nhiều khi bạn sẽ không biết loại của file là gì may mắn là linux cũng cung cấp câu lệnh cho biết loại của file: `file [path]`

- Case Sensitive: Linux là hệ thống phân biệt chữ hoa và chữ thường ví dụ hai file sau là hoàn toàn khác biết: `test.txt` và `Test.txt`.
- Space: Tệp tin trong linux hoàn toàn có thể sử dụng dấu cách(space) nhưng việc sử dụng dấu cách trong tên đôi khi gây ra lỗi. ví dụ bạn muốn cd vào một thư mục có tên chưa dấu cách. Ví dụ thư mục có tên `Holiday Photos`. ta thực hiện câu lệnh :`cd Holiday Photos` và sẽ nhận được thông báo `bash: cd: Holiday: No such file or directory`. Câu lệnh cd sẽ nghĩ đây là hai tham số và nó sẽ tiến hành cd vào thư mục có tên là tham số thứ nhất `Holiday` và nó báo là đó không phải là tên file hay thư mục. Vậy giải pháp là gì nếu muốn cd vào một thư mục có tên chứa đấu cách. Có hai giải pháp có thể thực hiện là:

- Quotes: Ta có thể sử dụng qoutes để bao quanh toàn bộ như sau: `cd 'Holiday Photos'` or `cd "Holiday Photos"` cả single qoutes và double qoutes đều cho kết quả giống nhau.
- Escape Characters: `cd Holiday\ Photos`.  Đây cũng là cách mà linux sẽ thực hiện khi bạn sử dụng Tab completion đã trình bày ở trên.

### File ẩn trong linux

Trong linux nếu bạn muốn một file hay một thư mục ẩn đi bạn chỉ cần thêm dấu chấm vào trước tên hoặc thư mục đó. Nếu muốn thư mục đó hiện ta cũng chỉ cần loại bỏ dấu chấm. Ví dụ những tên tin ẩn trong linux: `.anh.png`, `.Documents`, `.git`. NẾu bạn muốn hiển thị các file ẩn trong linux bạn thêm tham số `-a` vào câu lệnh `ls`. Ví dụ:

```sh
nguyentienthao@ntttnn-laptop:~$ ls -a
.              .git                   official_bktranslator
..             .gitconfig             Pictures
.adobe         .git-credential-cache  .pki
.atom          .git-credentials       .profile
.aws           .gnome                 Project 1
.bash_history  .gnupg                 Public
.bash_logout   .goldendict            .PyCharmCE2017.1
.bashrc        .gphoto                PycharmProjects
.boto          .gsutil                .selected_editor

```

Việc sử dụng các dòng lệnh trong linux đôi khi gây ra sự khó khăn vì người dùng phải nhớ các câu lệnh cũng như các tham số. Linux cung cấp một công cụ để giải thích những câu lệnh trong hệ thống như chúng dùng để làm gì, cách bạn sử dụng câu lệnh đó cũng như các tham số của câu lệnh đó. man <Câu lệnh cần tra cứu>

ví dụ: `man ls`

```sh
LS(1)                            User Commands                           LS(1)

NAME
       ls - list directory contents

SYNOPSIS
       ls [OPTION]... [FILE]...

DESCRIPTION
       List  information  about  the FILEs (the current directory by default).
       Sort entries alphabetically if none of -cftuvSUX nor --sort  is  speci‐
       fied.

       Mandatory  arguments  to  long  options are mandatory for short options
       too.

       -a, --all
              do not ignore entries starting with .

       -A, --almost-all
              do not list implied . and ..

       --author

```

### Quyền trong linux

Quyền trong linux dùng để đặc tả các hành động cho người dùng được thực hiện hoặc không được thực hiện với file hoặc thư mục. Một file trong linux có 3 quyền là đọc, viết và thực thi chúng được đặc tả bởi ba ký tự:

- r(read): Bạn có thể đọc nội dung của file.
- w(write): Bạn có thể thay đổi nội dung của file
- x(excute): Bạn có thể thực thi(chạy) file nếu nó là chương trình hoặc script.

Với mỗi file ta định nghĩa 3 nhóm người để đặc tả quyền cho từng nhóm:

- owner: Một người duy nhất là chủ của file. Thường là người tạo file.
- group: Mỗi file thuộc về một nhóm.
- others: Những người còn lại không thuộc hai đối tượng trên.

Để có thể xem các quyền ta sử dụng câu lệnh `ls` với tham số `-l` như sau:`ls -l [path]`

Ví dụ:

```sh
nguyentienthao@ntttnn-laptop:~/official_bktranslator$ ls -l decode.py
-rw-rw-r-- 1 nguyentienthao nguyentienthao 5199 Th02 15 18:28 decode.py

```

Ký tự đầu tiền cho biết là file hay thư mục. Trong trường hợp này là file(`-`) nếu thư mục sẽ là `d`. Ba ký tự tiếp theo đặc tả quyền dành cho owner. Ký tự `-` cho biết là không được thực hiện quyền đó. Ba ký tự tiếp theo cho group và 3 ký tự sau nữa là cho other.

Để thay đổi quyền ta sử dụng câu lệnh:

`chmod [permissions] [path]`

Tham số permissions trong câu lệnh chmod có 3 thành phần. Thành phần thứ nhất là đối tượng mà quyền được thay đổi. Thành phần thứ hai là thêm quyền hoặc loại bỏ quyền(+ hoặc -). Thành phần cuối cùng là quyền(r,w,x). Bạn có thể thêm hoặc bớt nhiều quyền một lúc. 

Ví dụ:

```sh
nguyentienthao@ntttnn-laptop:~/official_bktranslator$ chmod u+rwx decode.py
nguyentienthao@ntttnn-laptop:~/official_bktranslator$ ls -l decode.py
-rwxrw-r-- 1 nguyentienthao nguyentienthao 5199 Th02 15 18:28 decode.py

```

### Piping and redirection

Mỗi một chương trình chúng ta thực hiện bằng commamd lien có ba luồng dữ liệu chuẩn kết nối tới nó:

- STDIN(1): Đầu vào chuẩn.
- STDOUT(2): Đầu ra chuẩn. Dữ liệu được in ra bở chương trình. Mặc định sẽ là terminal
- STDERR(3): Đầu ra lỗi. Dùng để in ra lỗi. Mặc định là termial.

Piping và resirection là bằng một cách nào đó chúng ta sẽ kết nối giữa chương trình và file để direct dữ liệu.

#### Redirecting tới một file

Thông thường chúng sẽ xem được đầu ra ở màn hình(terminal) nhưng nếu chúng ta muốn lưu lại đẩu ra vào một file chúng ta sẽ sử dụng thêm toán tử `>`. Ví dụ:

`nguyentienthao@ntttnn-laptop:~/official_bktranslator$ ls > ls.txt`

và đây là kết quả file ls.txt

```txt
app.js
computeBLEU.py
config
config.json
data
data_utils.py
data_utils.pyc
decode.py
ls.txt
models
node_modules
package.json
public
requirements.txt
result.vi
routes
test.py
train
train2.txt
translate.py
views
```

Nếu sử dụng toán tử `>` thì nếu file đó chưa tồn tại hệ thống sẽ tự tạo file mới và in vào đó. Nếu file đã tồn tại hệ thống sẽ ghi lại dữ liệu của file đó. Để có thể ghi vào cuối của file đã tồn tại và ta sử dụng toán tử `>>`.

Ví dụ:

```sh
nguyentienthao@ntttnn-laptop:~/official_bktranslator$ ls > ls.txt
nguyentienthao@ntttnn-laptop:~/official_bktranslator$ ls -l >> ls.txt
```

Ta sẽ thu được file ls.txt như sau:

```txt
app.js
computeBLEU.py
total 17732
-rw-rw-r--   1 nguyentienthao nguyentienthao     2299 Th02 15 18:34 app.js
-rw-rw-r--   1 nguyentienthao nguyentienthao     5825 Th02 15 18:28 computeBLEU.py

```

