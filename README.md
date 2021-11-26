# REPLICATION
## Nội dung bài viết.
- [1. Khái niệm ](#1)
- [2. Lợi ích của Replication](#2)
- [3. Replication hoạt động ntn?](#3)
- [4. Cấu hình Replication](#4)



Đã baoh bạn tự hỏi trong 1 hệ thống quản trị dữ liệu khi xảy ra vấn đề các dữ liệu bị mất hết thì cta phaỉ làm sao để khắc phục sự cố này không ?
Hôm nay tôi sẽ gthieu cho mọi ng 1 cách đơn giản để giải quyết vấn đề này đó là sử dụng REPLICATION
Vậy thì REPLICATION là gì?
## 1.Khái niệm <a name="1"></a>
MySQL Replication là một quá trình cho phép bạn dễ dàng duy trì nhiều bản sao của dữ liệu MySQL bằng cách cho họ sao chép tự động từ một master tạo ra một cơ sở dữ liệu slave. Điều này rất hữu ích vì nhiều lý do bao gồm việc tạo điều kiện cho sao lưu cho dữ liệu, một cách để phân tích nó mà không sử dụng các cơ sở dữ liệu chính, hoặc chỉ đơn giản là một phương tiện để mở rộng ra.
Đơn giản bạn có thể hình dung khi cta cbi tài liệu cho 1 cuộc họp để đề phòng trong quá trình cb tài liệu bị mất hay vô tình bị hỏng cta sao chép nó ra thành nhiều lần để khi xảy ra sự cố chúng ta có thể sử dụng bản thay thế .
Replication mặc định là không đồng bộ, slave không cần phải kết nối vĩnh viễn để nhận được cập nhật từ master. Tùy thuộc vào cấu hình, bạn có thể sao chép tất cả các cơ sở dữ liệu, cơ sở dữ liệu đã chọn, hoặc thậm chí bảng được lựa chọn trong một cơ sở dữ liệu. Thật vậy, Replication có ý nghĩa là “nhân bản”, là có một phiên bản giống hệt phiên bản đang tồn tại, đang sử dụng. Với một cơ sở dữ liệu có nhu cầu lưu trữ lớn, thì đòi hỏi cơ sở dữ liệu phải toàn vẹn, không bị mất mát trước những sự cố ngoài dự đoán là rất cao. Vì vậy, người ta nghĩ ra khái niệm (slave) “nhân bản”, tạo một phiên bản cơ sở dữ liệu giống hệt cơ sở dữ liệu đang tồn tại, và lưu trữ ở một nơi khác, đề phòng có sự cố.
## 2.Vậy thì lợi ích của Replication là gì? <a name="2"></a>
Như đã nói ở trên nó giúp cta có thêm 1 bản sao dữ liệu dự phòng ngoài ra việc tạo ra 1 slave cũng giúp giảm tải cho cơ sở dữ liệu server master, tải trọng của server được phân tải cho các con slave, cải thiện hiệu năng cho toàn hệ thống. Trong môi trường này, tất cả các quá trình ghi và cập nhật đều phải diễn ra trên server master, bên cạnh đó quá trình đọc được diễn ra trên một hoặc nhiều con slave. Chính vì vậy mô hình này giúp tăng đáng kể hiệu năng của toàn hệ thống.
Tính bảo mật dữ liệu cao - vì dữ liệu được sao chép đến các slave, và các slave có thể tạm dừng quá trình sao chép, nó có thể chạy các dịch vụ sao lưu trên các slave mà không làm hư hỏng dữ liệu tổng thể tương ứng.
Tính phân tích - dữ liệu trực tiếp có thể được tạo ra trên master, trong khi phân tích các thông tin có thể xảy ra trên các slave mà không ảnh hưởng đến hiệu suất của master.
Tính phân phối dữ liệu từ xa - bạn có thể sử dụng replication để tạo ra một bản sao của dữ liệu cho một trang web từ xa để sử dụng, mà không cần truy cập thường xuyên vào con master.
## 3. Vậy thì REPLICATION hoạt động ntn? <a name="3"></a>
vậy thì để biết Replication hoạt dộng ntn? Hãy quan sát hình sau đây:
<img src=https://viblo.asia/uploads/4e1dca7d-eee1-4fd8-ad30-4451f1396cc4.png>
Bạn có thể thấy Replication nhân bản 1 master ra thành 1 cơ sở dữ liêu slave .
Khi cta thử hiện bất kì thao tác cập nhật hay thay dổi dữ liệu nào thì thao tác đó sẽ được thực hiện trên master trước ,khi đó các kết nối từ web app tới Master DB sẽ mở một Session_Thread khi có nhu cầu ghi dữ liệu. Session_Thread sẽ ghi các statement SQL vào một file binlog. 
Lúc này master DB sẽ mở một Dump_Thread và gửi binlog tới cho I/O_Thread mỗi khi I/O_Thread từ Slave DB yêu cầu dữ liệu.
Trên mỗi Slave DB sẽ mở một I/O_Thread kết nối tới Master DB thông qua network, giao thức TCP (với MySQL 5.5 replication chỉ hỗ trợ Single_Thread nên mỗi Slave DB sẽ chỉ mở duy nhất một kết nối tới Master DB, các phiên bản sau 5.6, 5.7 hỗ trợ mở đồng thời nhiều kết nối hơn) để yêu cầu binlog.
Sau khi Dump_Thread gửi binlog tới I/O_Thead, I/O_Thread sẽ có nhiệm vụ đọc binlog này và ghi vào relaylog.
Đồng thời trên Slave sẽ mở một SQL_Thread, SQL_Thread có nhiệm vụ đọc các event từ relaylog và apply các event đó vào Slave => quá trình replication hoàn thành.
Về logic mỗi Slave DB sẽ chỉ nhận dữ liệu từ Master DB, mọi hành động cập nhật dữ liệu BẮT BUỘC phải được thực hiện trên Master. Về nguyên tắc nếu ghi dữ liệu trực tiếp lên Slave DB => hỏng replication. Nhưng thực chất ta hoàn toàn có thể ghi dữ liệu trên Slave miễn sao khi Slave đọc binlog và apply không đụng gì tới những trường dữ liệu mà ta mới ghi vào thì sẽ không bị lỗi (mục này sẽ nói thêm ở các phần sau)
Với MySQL 5.5 thì mỗi slave sẽ chỉ có một slave_thread connect tới Master, tuy nhiên từ phiên bản 5.6 chúng ta có thể cấu hình nhiều slave_thread để việc apply bin log tới các slave nhanh hơn.
## Cấu hình MySQL Replication Master - Slave

MySQL replication là một tiến trình cho phép sao chép dữ liệu của MySQL một cách tự động từ máy chủ Master sang máy chủ Slave. Nó vô cùng hữu ích cho việc backup dữ liệu hoặc sử dụng để phân tích mà không cần thiết phải truy vấn trực tiếp tới CSDL chính hay đơn giản là để mở rộng mô hình.

Bài lab này sẽ thực hiện với mô hình 2 máy chủ: 1 máy chủ master sẽ gửi thông tin, dữ liệu tới một máy chủ slave khác (Có thể chung hoặc khác hạ tầng mạng). Để thực hiện, trong ví dụ này sử dụng 2 IP:

- IP Master: 10.10.10.1
- IP Slave: 10.10.10.2

### 1. Cấu hình trên máy chủ Master

#### Tạm dừng dịch vụ MySQL

> systemctl stop mysqld

#### Khai báo cấu hình cho Master

Thêm các dòng sau vào file cấu hình `/etc/my.cnf`

```
[mysqld]
...
bind-address=10.10.10.1
log-bin=/var/lib/mysql/mysql-bin
server-id=101
```

- `bind-address`: Cho phép dịch vụ lắng nghe trên IP. Mặc định là 127.0.0.1 - localhost
- `log-bin`: Thư mục chứa log binary của MySQL, dữ liệu mà Slave lấy về thực thi công việc replicate.
- `server-id`: Số định danh Server

#### Khởi động dịch vụ MySQL

> systemctl start mysqld

Đăng nhập vào MySQL, tạo một user sử dụng trong quá trình replication

> mysql -uroot -p

```
mysql> grant replication slave on *.* to replica@'10.10.10.2' identified by 'password';

Query OK, 0 rows affected (0.00 sec)
mysql> flush privileges;

```

Khóa tất cả các bảng và dump dữ liệu <a name='1' />

```
mysql> flush tables with read lock;
mysql> show master status;

+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      592 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

```

> **Chú ý**: Ghi nhớ thông tin này để khai báo khi [cấu hình trên Slave](#2)

Mở một cửa sổ terminal khác để dump dữ liệu

> \# mysqldump -u root -p --all-databases --lock-all-tables --events > mysql_dump.sql 

Quá trình dump hoàn thành, quay trở lại terminal trước để Unlock các bảng

```
mysql> unlock tables; 
mysql> exit
```

Chuyển dữ liệu vừa dump sang máy chủ slave.

### 2. Cấu hình máy chủ Slave

Thêm các dòng sau vào file cấu hình `my.cnf` trên máy chủ Slave. Mục đích là định danh máy chủ slave và chỉ ra nơi lưu trữ bin-log.

```
[mysqld]
...
log-bin=/var/lib/mysql/mysql-bin
server-id=102
```

Khởi động lại MySQL

> systemctl restart mysqld

Khôi phục lại dữ liệu vừa dump trên Master. Ví dụ, file dump được copy về để ở thư mục /tmp

> mysql -u root -p < /tmp/mysql_dump.sql

<a name='2' />

Sau khi xong, đăng nhập vào MySQL để cấu hình Repilcate Master Slave

> mysql -uroot -p.

```
mysql> change master to
    -> master_host='10.10.10.1',
    -> master_user='replica',
    -> master_password='password',
    -> master_log_file='mysql-bin.000001',
    -> master_log_pos=592;
 mysql> start slave;
 mysql> show slave status\Gd
 ```

**Chú ý**: Điền thông tin `log_file` và `log_pos` trùng khớp với thông số mà ta đã lấy ở bước [trên](#1).

## Bỏ qua câu lệnh Replication bị lỗi

Đăng nhập vào mysql và thực hiện bỏ qua 1 câu query bị lỗi:

```
STOP SLAVE;
SET GLOBAL SQL_SLAVE_SKIP_COUNTER = 1;
START SLAVE;
```

- https://www.howtoforge.com/how-to-repair-mysql-replication
```
vậy là đã hoàn thành . Hi vọng qua bài viết của mình giúp các bạn có thể hiểu hơn về replication trong Mysql :)))))
