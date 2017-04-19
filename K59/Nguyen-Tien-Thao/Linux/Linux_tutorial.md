# Command line trong Linux

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

## Một số câu lệch cơ bản

- `pwd`( Print Working Directory): Câu lệnh cho biết bạn thư mục bạn đang làm việc. Bạn sẽ nhận thấy hầu hết các câu lệnh trong linux là viết tắt của từ hay cụm từ miêu tả chúng điều này giúp việc ghi nhớ trở nên dễ dàng.

ví dụ:

```sh
nguyentienthao@ntttnn-laptop:~$ pwd
/home/nguyentienthao

```

Lưu ý: Có rất nhiều câu lệnh trong linux thực hiện dựa trên nơi thực hiện câu lệnh đó. Đôi khi bạn chuyển qua lại giữa các thư mục mà quên mất mình đang ở đâu nên chắc chắn rằng bạn thực hiện câu lệnh tại đúng nơi cần thực hiện.

- `ls`: Câu lệnh này cho biết những thứ có trong thư mục hiện tại.

ví dụ:

```sh
nguyentienthao@ntttnn-laptop:~$ ls
build    distributted  official_bktranslator  Public           Videos
Desktop  Downloads     Pictures               PycharmProjects
DHBKHN   key.pem       Project 1              stock_market

```

Câu lệnh ls thực hiện không có tham số thêm sẽ cho ta danh sách thu gọn. Ngoài ra ta có thể thêm một số tham số vào sau như sau theo cú pháp sau: `ls [options] [location]`. Trong cú pháp đó những tham số đặt trong dấu `[]`là tham số tùy chọn(có thể có hoặc không).
