# Linux cơ bản

## 1.1 Các lệnh cơ bản

* pwd: in ra đường dẫn tuyệt đối của thư mục hiện tại

* ls: in ra nội dung của thư mục

* cd pathname: truy cập đến thư mục theo đường dẫn

* tail -n file: hiển thị nội dung n dòng cuối file mặc định xem cả file

* cat file: xem nhanh 1 file

* rm file: xoá file

  * rm -r path : xoá toàn bộ file thư mục (cả thư mục con)

* cp file dir: di chuyển file đến thư mục 

* SSH

  * mkdir: tạo thư mục

  * touch: tạo file

  * grep str file : tìm kiếm trong file

    * có thể tìm kiếm trong nhiều file
    * tuỳ chọn -i: không phân biệt chữ hoa chữ thường
    * đếm số kết quả với -c

  * find : tìm kiếm file và thư mục

    

## 1.2 File và Process trong linux
### File
* Cấu trúc hệ thống tệp
  * Một/Nhiều cây phân cấp thư mục và các tệp
  * Thư mục gốc / là điểm vào đầu tiên cho cả cây thư mục
  * Các tệp là các nút lá
* Các thư mục thông dụng trong Linux
  * / : thư mục gốc
  * /boot : thư mục chứa nhân hệ điều hành
  * /etc : thư mục chứa các tệp cấu hình
  * /dev : thư mục các tệp thiết bị
  * /home : thư mục chứa dữ liệu người sử dụng
  * /lib : thư viện hệ thống
  * /usr : thư mục ứng dụng
  * /var : thư mục dữ liệu cập nhật
* Đường dẫn tương đối và tuyệt đối
  * Truy cập tệp dùng các đường dẫn
    * / : thư mục gốc
    * ~/ : thư mục nhà
    * . : thư mục hiện tại
    * .. : thư mục cha
  * Đường dẫn tuyệt đối là đường dẫn bắt đầu từ thư mục gốc
  * Đường dẫn tương đối là đường dẫn không bắt đầu từ thư mục gốc
* Liệt kê nội dung thư mục với : ls [options][pathname]
  * -l: long list
  * -d: working directory
  * -n: user/group id's
  * -r: reverse order
  * -t: time sequence
  * -u: time---last access
  * -c: time--inode date
  * -p: identify directories
  * -R: recursive
  * -1: print one column
  * -i: print inode
* Các kiểu của tệp
  * -: tệp thông thường
  * d: thư mục
  * b: tệp đặc biệt (block)
  * c: tệp đặc biệt (ký tự)
  * l: link
  * m: phần bộ nhớ trong dùng chung
  * p: đường ống pipeline
* Các siêu ký tự
  * dấu * : dùng để thay thế cho một chuỗi kí tự bất kì kể cả xâu rỗng
  * ? : thay thế cho một kí tự bất kì
  * [] : thay thế cho một kí tự trong tập kí tự cho trước
  * [!] : thay thế cho một kí tự không có trong tập kí tự cho trước
* **Khái niệm inode**
    * 1 inode được tạo ra cho một điểm vào trên hệ thống tệp
    * nội dung của tệp lưu trong các khối dữ liệu, tệp rỗng = 1 inode không có khối dữ liệu
    * Thư mục là một tệp có nội dung là một bảng các liên kết, mỗi liên kết gắn tên
* Liên kết vật lý
  * Một liên kết vât lý là một quan hệ giữa tên tệp trong thư mục với 1 inode, 1 inode có thể có nhiều liên kết vật lý
  * Lệnh ln cho phép tạo 1 liên kết vật lý đến 1 inode (tệp) đã tồn tại, tệp mới chia sẻ cùng inode và khối dữ liệu của tệp ban đầu
  * Số liên kết vật lý đến 1 inode với lệnh ls -l, một thư mục luôn có ít nhất 2 liên kết vật lý
  * Xóa tệp với rm đồng nghĩa với xóa 1 liên kết vât lý, nếu liên kết vật lý cuối cùng trỏ đến inode được xóa thì các khối liên quan đến inode cũng được xóa theo
* Liên kết biểu tượng
  * ln -s path1 path2 : tạo ra 1 inode mới
  * inode này chứa tên (đường dẫn) của phần tử gốc
  * cho phép tránh được các hạn chế về mặt dung lượng của thiết bị
### Process
#### Giới thiệu
* 1 tiến trình bằng thực thi của một chương trình
* Mỗi tiến trình sẽ tương ứng tập các thông tin sau:
  * định danh: pid
  * tiến trình cha: ppid
  * người sở hữu: uid và nhóm: gid
  * câu lệnh khởi tạo tiến trình
  * 
    * 1 đầu vào chuẩn: stdin
    * 1 đầu ra chuẩn: stdout
    * 1 kênh báo lỗi chuẩn stderr
  * thời gian sử dụng CPU: CPU time và mức độ ưu tiên
  * thư mục hoạt động hiện tại của tiến trình
  * bảng các tham chiếu đến các file được tiến trình sử dụng
* Các tiến trình được sắp xếp để chia sẻ thời gian sử dụng CPU
#### Các kiểu tiến trình
* Các tiến trình hệ thống
  * thường thuộc về quyền root, không có giao diện tương tác
  * thường chạy dưới dạng các tiến trình ngầm (daemon)
  * đảm nhiệm các nhiệm vụ chung, phục vụ mọi người sử dụng
* Các tiến trình của người sử dụng
  * thực hiện các nhiệm vụ của một người dùng cụ thể
    * thực hiện dưới dạng một shell 
    * thực hiện dưới dạng một lệnh thông qua shell
  * thường được thực hiện quản lý bằng terminal: cp, vi, man,...
#### Lệnh ps
* Hiển thị các tiến trình
  * ngầm định ps hiển thị các tiến trình thuộc người sử dụng terminal
  * sử dụng option aux để hiển thị tất cả các tiến trình đang chạy trong máy
* Trạng thái của tiến trình
  * S: đang ngủ
  * R: đang chạy
  * T: dừng
  * Z: không xác định
* Lệnh kill
  * gửi một tín hiệu đến một tiến trình, định danh của tiến trình được xác định dưới dạng 1 tham số của lệnh
    * ngầm định tín hiệu gửi đi là tín hiệu 15
    * kill pid
    * tùy chọn -l: liệt kê tất cả các tín hiệu có thể sử dụng
  * killall: dùng để kết thúc tất cả các tiến trình của một câu lệnh thông qua việc truyền tên của câu lệnh dưới dạng một tham số
    * killall tên_lệnh
  * quyền hủy tiến trình thuộc về người sở hữu tiến trình
* Độ ưu tiên của các tiến trình
  * các tiến trình có độ ưu tiên ban đầu = 0
  * mức độ ưu tiên của tiến trình -19 đến +19, chỉ root mới có thể giảm giá trị độ ưu tiên của tiến trình. Một người sử dụng thông thường chỉ có thể giảm độ ưu tiên của tiến trình thông qua tăng độ ưu tiên của các tiến trình khác
  * Lệnh nice cho phép thay đổi độ ưu tiên của tiến trình ngay khi bắt đầu thực hiện lệnh tương ứng với tiến trình
    * nice [-n Value] [Command [Arguments ...]]
  * Lệnh renice cho phép thay đổi độ ưu tiên của một tiến trình sau khi đã chạy
* Lệnh top
  * Hiển thị thông tin các tiến trình đang chạy và cập nhật nó liên tục: %CPU, %RAM
  * top [-d] delay: tùy chọn -d cho phép xác định thời gian cập nhật thông tin theo s
  * Lệnh top cho phép người sử dụng tương tác và quản lý các tiến trình
    * phím f trong quá trình hoạt động của top cho phép thay đổi thông tin hiển thị
* Foreground và background
  * Quá trình chạy ở chế độ hiện sẽ tiến hành theo những bước sau:
    * << fork >> nhân bản tiến trình cha
    * thực hiện quá trình << wait >>, đưa tiến trình cha vào trạng thái ngủ
    * thực hiện quá trình << exec >>, thực thi tiến trình con
    * tiến trình con thực thi xong, tín hiệu đánh thức sẽ gửi đến tiến trình cha
    * trong khi chạy tiến trình con không thể thực hiện tiến trình cha
  * Quá trình chạy ở chế độ ngầm
    * cho phép thực thi độc lập giữa tiến trình cha và con VD: emacs&
    * sau khi thực hiện lệnh trên, emacs sẽ chạy ở chế độ ngầm
* Quản lý tác vụ
  * 1 tác vụ = việc thực hiện 1 câu lệnh, 1 tác vụ có thể liên quan đến 1 nhóm các tiến trình: tiến trình cha và tập tiến trình con
  * chỉ có 1 tác vụ chạy ở chế độ hiện foreground, còn tác vụ ngầm background có thể có nhiều
* Thực thi lệnh song song
  * cmd1;cmd2
  * cdm1 && cmd2
  * cmd1 | cmd2
* Chuyển hướng kênh
  * Mỗi tiến trình sở hữu
    * 1 đầu vào chuẩn ngầm định bàn phím
    * 1 đầu ra chuẩn ngầm định terminal
    * 1 kênh báo lỗi chuẩn ngầm định terminal
  * Chuyển hướng đầu vào chuẩn (<)
    * VD: $ tee < test.txt
  * Chuyển hướng đầu ra chuẩn (>, >>)
    * $ ls > /dev/lp
    * $ ls >> test.txt
  * Chuyển hướng kênh báo lỗi
    * $ rm prog.c 2> /dev/null
    * $ gcc pro.c 2>> erreur.txt
## 1.3 User và group
### User
* Người sử dụng thông thường
* Quản trị
#### Thông tin của người sử dụng
* Tất cả thông tin lưu trong file etc/passwd 
* **Username:password:UID:GID:Info:Home:Shell**
  * Username: sử dụng để log in, chứa 1-31 kí tự
  * Password: mật khẩu đăng nhập, biểu diễn trong file etc/passwd dạng kí tự x
  * User ID (UID): UID 0 là root, mỗi user được chỉ định 1 UID từ 1-99, UID 100-999 phục vụ cho quản trị, tài khoản và nhóm hệ thống
  * Group ID (GID): ID của nhóm chính
  * User ID Info: cho phép bạn thêm thông tin người dùng như fullname, phone,..
  * Home directory: đường dẫn tuyệt đối tới thư mục người dùng đăng nhập, nếu không tồn tại thư mục người dùng sẽ thành /
  * Command/Shell: đường dẫn tuyệt đối của 1 command hoặc shel (/bin/bash)
* File /etc/shadow **User:Pwd:Last pwd change:Minimum:Maximum:Warm:Inactive:Expire**
  * User name: tên đăng nhập
  * Password: mật khẩu tối thiểu 6-8 kí tự gồm ký tự cả đặc biệt và số
  * Last password change: mật khẩu cũ tính từ 1/1/1970
  * Minimum: số ngày nhỏ nhất được yêu cầu giữa 2 lần thay đổi mật khẩu
  * Maximum: số ngày tối đa để mật khẩu được xác nhận (khi đổi mật khẩu)
  * Warn: số ngày trước khi mật khẩu hết hạn, người dùng sẽ nhận được cảnh báo
  * Inactive: số ngày sau khi mật khẩu hết hạn và tài khoản bị khóa
  * Expire: ngày tài khoản bị vô hiệu hóa
* Các lệnh thao tác
  * useradd [option] <username> Các options: 
    * -c: Thông tin user 
    * -d: Thư mục cá nhân 
    * -m: Tạo thư mục cá nhân nếu chưa tồn tại 
    * -g: nhóm người dùng
  * usermod [option] <username>
  * userdel [option] <username>
  * passwd -l <username> : khóa người dùng
  * passwd -u <username> : mở khóa
### Group
* Một người sử dụng có thể thuộc 1 hoặc nhiều nhóm
  * 1 nhóm = tên nhóm + danh sách các thành viên
  * danh sách nhóm lưu trong file /etc/group
  * root có khả năng tạo nhóm bổ sung, ngoài các nhóm hệ điều hành đã ngầm định
#### Thông tin nhóm
* File /etc/group **group_name:Password:Group ID(GID):Group List**
  * group_name: tên nhóm, lệnh ls -l, tên nhóm sẽ hiển thị trong trường group
  * password: thông thường password không được sử dụng và để trống; nếu là X có nghĩa là password được lưu /etc/gshadow
  * GID: mỗi người dùng phải chỉ định 1 group ID
  * Group List: danh sách người dùng thành viên group cách nhau dấu phảy
* File /etc/gshadow
  * group name: tên của group
  * encrypted password: mật khẩu mã hóa cho group
    * ! : người dùng được phép truy cập group với lệnh newgrp
    * null : chỉ thành viên nhóm mới log in được
  * group administrators: thành viên nhóm ở đây có thể add hoặc xóa thành viên với lệnh gpasswd
  * group members: danh sách thành viên hợp lệ không quản trị các thành viên của nhóm
* Các lệnh thao tác
  * groupadd <groupname> : tạo nhóm
  * groupdel <groupname> : xóa nhóm

## 1.4 Các tệp tin hệ thống trong linux
* /-Root 
  * Là thư mục gốc của hệ thống tệp
* /sbin-System Binaries
  * Chứa các file thực thi dạng nhị phân của chương trình cơ bản giúp hệ thống hoạt động
  * Các lệnh trong /sbin được sử dụng với mục đích quản trị hệ thống, yêu cầu quyền root
* /bin-User Binaries
  * Chứa dạng nhị phân của nhiều ứng dụng có cả cho việc bảo trì hệ thống của root, cũng như các lệnh cho người dùng
* /boot-Boot Loader Files
  * Chứa các tệp tin khởi động, nhân kernel, grub
* /dev-Device Files
  * Chứa các file thiết bị, các thiết bị phần cứng được xem như là các file như /dev/sda, /dev/cdrom,..
* /etc-Configuration Files
  * Chứa file cấu hình cho các chương trình hoạt động
  * /etc/resolv.conf : cấu hình DNS server
  * /etc/network : cấu hình quản lý network
  * /etc/rc.d : chứa các scripts dùng để start,stop kiểm tra status cho các chương trình
* /home-Home directories
  * Chứa thông tin dữ liệu, cấu hình riêng cho từng user
* /lib-System Libraries
  * Chứa các file thư viện hỗ trợ cho các file thực binary
* /mnt-Mount Directory
  * Chứa các thư mục dùng để system admin thực hiện quá trình mount. Tạo ra các thư mục để 'gắn' các phân vùng ổ đĩa cứng cũng như các thiết bị khác, sau đó ổ đĩa được truy cập từ đây như 1 thư mục
* /opt-Optional add-on Applications
  * Chứa các phần mềm và phần mở rộng không trong cài đặt mặc định
* /proc-Process Information
  * Chứa thông tin về quá trình xử lý của hệ thống
  * 1 pseudo filesystem chứa các thông tin về process đang chạy
  * 1 virtual filesystem chứa các thông tin về tài nguyên hệ thống
* /tmp-Temporary Files
  * thư mục tạm thời, được làm rỗng sau khi khởi động lại hay shutdown
* /usr-User Programs
  * Chứa các file binary, library, tài liệu, source-code cho các chương trình
  * /usr/bin chứa các file binary cho các chương trình của user. Khi user thưc thi lệnh ban đầu nó sẽ tìm kiếm trong /bin, không có sẽ tìm trong /usr/bin
  * /usr/sbin chứa các file binary cho system admin, nếu không tìm thấy file ở /sbin sau đó sẽ tìm ở trong /usr/sbin
  * /usr/lib chứa các file libraries cho /usr/bin và usr/sbin
  * /usr/local chứa chương trình của các user, các chương trình này được cài từ source 
* /media-Removable Media Devices
  * Chứa thư mục dùng để mount các thiết bị removable CDROM, Floppy
* /var-Variables Files
  * Chứa các file có sự thay đổi trong quá trình hoạt động của hệ điều hành cũng như các ứng dụng
    * /var/log: nhật ký của hệ thống
    * /var/lib: database file
    * /var/email: email
    * /var/spool: hàng đợi in ấn
    * /var/lock: lock file
    * /var/tmp: file tạm thời cho quá trình reboot
    * /var/www: dữ liệu cho trang web


## 1.5 Nano editor

* Mở file 

  * > nano filename

* Các phím tắt 

  | Command  | Giải thích                                               |
  | -------- | :------------------------------------------------------- |
  | CTRL + A | Chuyển tới đầu dòng                                      |
  | CTRL + E | Chuyển tới cuối dòng                                     |
  | CTRL + O | Lưu file                                                 |
  | CTRL + W | Tìm kiếm trong văn bản, gõ ALT + W để tìm tới từ kế tiếp |
  | CTRL + K | Xóa và lưu dòng được chọn trong bộ nhớ đệm               |
  | CTRL + U | Dán dòng trong bộ nhớ đệm                                |
  | CTRL + X | Thoát Nano                                               |
  | CTRL + _ | Tới dòng chỉ định và số cột                              |
  | ALT + A  | Chọn văn bạn sử dụng nút di chuyển từ con trỏ chuột      |

  

