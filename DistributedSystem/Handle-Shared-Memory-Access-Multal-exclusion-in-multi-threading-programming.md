# Handle Shared Memory Access/Multal-exclusion In Multi-threading Programming

Bài viết này sẽ giới thiệu về các vấn đề về truy cập tài nguyên chia sẻ shared-memory trong các multi-threading program. Sau đó, bài viết sẽ phân tích một số phương pháp giải quyết các vấn đề này.

## Read-Write problem

### The problem

Read Write problem là một vấn đề khá phổ biến trong môi trường lập trình  multi-threading. Vấn đề này được mô tả như sau:

Trong một chương trình multi-threading, có một tài nguyên chung **x** mà các thread trong tiến trình muốn truy cập vào và sử dụng trong quá trình các thread đó hoạt động. Nhu cầu của các tiến trình đối với tài nguyên **x** được phân ra làm 2 loại:

- Phần lớn các thread trong chương trình chỉ có nhu cầu đọc nội dung của **x** - **read-only thread**.
- Các thread còn lại vừa đọc nội dung vừa thay đổi nội dung của **x** - Read and Write, ký hiệu là **read-write thread**.

Để hệ thống sử dụng tài nguyên chung với hiệu quả (hiệu năng) cao nhất, chúng ta phải đảm bảo các điều kiện sau:

- Các read-only thread có thể đồng thời đọc tài nguyên chung cùng một lúc.
- Khi có ít nhất một read-only thread đang đọc tài nguyên chung, không read-write thread nào được truy cập vào tài nguyên chung.
- Chỉ có một read-write thread đang truy cập vào tài nguyên chung tại một thời điểm trong hệ thống, và trong suost thời gian read-write thread  đó đang truy cập vào tài nguyên chung, thì các read-only thread và các read-write thread còn lại không được phép truy cập vào tài nguyên chung.

Vậy làm sao để chúng ta thiết kế được một cơ chế truy cập **x** thỏa mãn các điều kiện trên ?

### Solution #1

Một trong các phương pháp  được sử dụng để giải quyết read-write lock problem là sử dụng mô hình read-write lock. Mô hình read-write lock là sử dụng các thành phần sau để quản lý việc các thread truy cập vào tài nguyên chung **x**:

- Một condition-variable **read-lock** được sử dụng để xác định tài nguyên chung có đang được thread nào **read** không.
- Một condition variable **write-lock** được sử dụng để xác định tài nguyên chung có đang được thread nào **write** không.
- Một **read-counter** cho phép xác định có bao nhiêu thread đang thực hiện **read** tài nguyên chung.
- Môt **mutex** quản lý việc truy cập vào giá trị của các biến điều khiển.

Nguyên tắc hoạt động của mô hình read-write lock là:

Để một **read-only thread** có thể truy cập **x**, nó cần đảm bảo rằng trong hệ thống hiện tại không có một **read-write thread** nào đang thực hiện việc thay đổi nội dung của x bằng cách kiểm tra xem write-lock có bằng **false** hay không. Nếu write-lock bằng false (không có read-write thread nào đang thay đổi nội dung của x), read-thread này  thực hiện việc tăng read-counter lên 1 rồi thực hiện việc đọc nội dung của x. Sau khi đọc xong x, read-thread thực hiện việc giảm read-counter xuống 1.

Để một **read-write thread** được truy cập vào x, nó cần kiểm tra 2 điều kiện sau:

1. Có read-only thread nào đang đọc nội dung của x hay không ? (Thông qua việc kiểm tra giá trị của read-counter)
1. Có read-write thread nào đang thay đổi nội dung của x hay không ? (thông qua giá trị của write-lock)

Nếu cả hai điều kiện trên được thỏa mãn, read-write thread thiết lập write-lock bằng true, sau đó thực hiện việc đọc/ghi x. Sau khi thực hiện xong việc đọc/ghi x, read-write thread này thực hiện việc thiết lập write-lock bằng false.

Giải thuật của mô hình read-write lock được mô tả như sau:

```python

def setup():
    read_waiting = 0
    read_counter = 0
    write_lock = False
    mutex m
    semaphore unlock_read_sem
    semaphore write_lock_sem
    unlock_read_sem.set(0)
```

```python
def read_only_thread():
    mutex_lock(m)
        if write_lock:
            read_waiting++;
            unlock_read_sem.aquire();
    mutex_unlock(m)
    # part of code read only shared memory
    mutex_lock(m)
        read_counter--
        if(read_counter==0):
            unlock = check_unlock()
        if unlock == unlock_write:
            write_lock_sem.release()
            wait_to_write_lock_is_true()
    mutex_unlock(m)
    # part of code not read shared memory

```

```python
def write_thread():

    if write_lock || read_counter > 0:
        write_lock_sem.acquire()
    write_lock = true
    # part of code read & write shared memory
    unlock = check_unlock(write_lock,read_lock)
    if unlock == unlock_read:
        for i = 1 to read_waiting:
            unlock_read_sem.release()
            read_counter++
        write_lock = false
    else if unlock == unlock_write:
        write_lock_sem.release()
    #part of code not read-write shared memory
```

### Implement của mô hình read-write lock trong C/C++ language

Trong C/C++, mô hỉnh read-write lock được implement trong một số thư viện, trong đó có thư viện **\<pthread.h\>**: [http://pubs.opengroup.org/onlinepubs/009695399/basedefs/pthread.h.html](http://pubs.opengroup.org/onlinepubs/009695399/basedefs/pthread.h.htmll)

Trong thư viện này, chúng ta sử dụng read-write lock thông qua các function sau:

```c
#include <pthread.h>
// pthread_rwlock_destroy, pthread_rwlock_init - destroy and initialize a read-write lock object
int pthread_rwlock_destroy(pthread_rwlock_t *rwlock);
int pthread_rwlock_init(pthread_rwlock_t *restrict rwlock,
       const pthread_rwlockattr_t *restrict attr);

// pthread_rwlock_rdlock, pthread_rwlock_tryrdlock - lock a read-write lock object for reading
int pthread_rwlock_rdlock(pthread_rwlock_t *rwlock);
int pthread_rwlock_tryrdlock(pthread_rwlock_t *rwlock);

// pthread_rwlock_trywrlock, pthread_rwlock_wrlock - lock a read-write lock object for writing
int pthread_rwlock_trywrlock(pthread_rwlock_t *rwlock);
int pthread_rwlock_wrlock(pthread_rwlock_t *rwlock);

// pthread_rwlock_unlock - unlock a read-write lock object
int pthread_rwlock_unlock(pthread_rwlock_t *rwlock);
```

Chi tiết về quá trình xử lý của các phương thức trên có thể xem trong các tài liệu sau:

- [http://pubs.opengroup.org/onlinepubs/009695399/basedefs/pthread.h.html](http://pubs.opengroup.org/onlinepubs/009695399/basedefs/pthread.h.html)
- [https://docs.oracle.com/cd/E19455-01/806-5257/6je9h032u/index.html](https://docs.oracle.com/cd/E19455-01/806-5257/6je9h032u/index.html)

**Note**: từ C++14, thư viện chuẩn của C++ cho phép chúng ta sử dụng mô hình read-write lock thông qua **std::shared_mutex**:

[http://en.cppreference.com/w/cpp/thread/shared_mutex](http://en.cppreference.com/w/cpp/thread/shared_mutex)

### Implement của mô hình read-write lock trong Python multithreading sử dụng greenlet thông qua thư viện eventlet và eventlet semaphore

## Referrences

- [http://pubs.opengroup.org/onlinepubs/009695399/basedefs/pthread.h.html](http://pubs.opengroup.org/onlinepubs/009695399/basedefs/pthread.h.html)
- [https://docs.oracle.com/cd/E19455-01/806-5257/6je9h032u/index.html](https://docs.oracle.com/cd/E19455-01/806-5257/6je9h032u/index.html)