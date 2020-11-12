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
## 4. Cấu hình Replication trên MySQL ntn? <a name="4"></a>
Sau đây mình sẽ hướng dẫn mọi người cấu hình Replication trên ubuntu 20.04 nhé!
Trước tiên chúng ta phải có 2 máy thông nhau trước để kiểm tra 2 máy có thông nhau hay không cta sử dụng câu lệnh ping
Giả sử ta có 2 có địa chỉ IP là :
192.168.0.1 (Master) và 192.168.0.2(Slave)
Từ Master cta thực hiện ping :
```
ping 192.168.0.1
```
Sau khi Ping thành công cta bắt đầu thực hiện cài MySQL trên cả Master và Slave bằng câu lệnh
```
sudo apt-get install mysql-server mysql-client
```
Đầu tiên chúng ta thực hiện cấu hình trên Master trước .Trước tiên là file conf của mysql
Mở terminal và gõ:
```
vim /etc/mysql/mysql.conf.d/mysqld/cnf
```
Tiếp theo bạn cần config các thứ sau
```
bind-address  = 127.0.0.1
```
Đây là giá trị mặc định khi bạn cài mysql cũng có thể chính là @localhost hoặc dòng này đã bị comment bạn cần sửa thành địa chỉ ip của master
```
bind-address  = 192.168.0.1
```
Nhiệm vụ này giúp Slave có thể biết được rằng nó đang đọc file của máy nào để thực hiện sao chép chuẩn xác nhất.
Tiếp theo bạn cần config server-id. Bạn có thể đăt giá trị là bất kỳ số nguyên nào, tuy nhiên hãy bảo đảm rằng trong hệ thống master/salve sẽ ko có sự trùng nhau. Và ở đây mình xin set là 1.
```
server-id = 1
```
Chuyển sang dòng log_bin. Đây là nơi giữ các chi tiết thực sự của bản sao. Các Slave sao chép tất cả các thay đổi. Đối với bước này, chúng ta chỉ cần bỏ ghi chú dòng tham chiếu đến log_bin:
```
log_bin                 = /var/log/mysql/mysql-bin.log
```
Thêm vào cuối file kích hoạt plugin nhân bản:
```
plugin-load=mysql_clone.so
```
Sau khi cấu hình xong thực hiện khởi động lại mysql:
```
systemctl restart mysql
```
Bạn truy cập lại vào mysql:
```
mysql -u root -p(password)
```
Đầu tiên cta cần phải tạo 1 user cho việc thực hiện Replication:
```
create user 'repl_user'@'%' identified by 'password';
```
Sau đó gán quyền replication cho user:
```
grant replication slave on *.* to repl_user@'%';
```
Sau đó tạo 1 clone user và gán quyền backup cho nó:
```
create user 'clone_user'@'%' identified by 'password';
grant backup_admin on *.* to 'clone_user'@'%';
flush privileges;
exit
```
Đã hoàn thành cấu hình trên master bây giờ cta tiếp tục cấu hình trên slave
Bước đầu tiên cta làm y như trên master chỉnh sửa file config của mysql:
```
bind-address  = 192.168.0.2
server-id = 2
log_bin                 = /var/log/mysql/mysql-bin.log
```
Sau đó tôi muốn Slave của mình chỉ thực hiện đọc tôi thêm vào cuối file:
```
read_only=1
```
Tiếp theo thực hiện xác định máy chủ riêng
```
report-host=node01.srv.world
```
Chúng ta tiếp tục cần xác định relay log:
```
relay-log=/var/log/mysql/node01-relay-bin
relay-log-index=/var/log/mysql/node01-relay-bin
```
cuối cùng là kích hoạt plugin sao chép:
```
plugin-load=mysql_clone.so
```
Khởi động lại mysql và truy cập lại vào mysql . Bạn sẽ cần tạo ra 1 user clone và gán quyền clone_admin cho nó:
```
 create user 'clone_user'@'%' identified by 'password';
 grant clone_admin on *.* to 'clone_user'@'%';
 flush privileges; 
 exit
 ```
 Vậy là bạn đã cấu hình xong Master và Slave. Trên Máy chủ slave, chạy công việc nhân bản để sao chép dữ liệu trên master và bắt đầu sao chép.
Sau khi bắt đầu sao chép, hãy đảm bảo sao chép hoạt động bình thường để tạo cơ sở dữ liệu thử nghiệm hoặc chèn dữ liệu thử nghiệm, v.v...
Thiết lập nhân bản với localhost:
```
 set global clone_valid_donor_list = '127.0.0.1:3306';
```
Thực hiện chạy clone:
```
clone instance from clone_user@127.0.0.1:3306 identified by 'password';
```
Tiếp theo cta cần xác nhận tình trạng của clone :
```
select ID,STATE,SOURCE,DESTINATION,BINLOG_FILE,BINLOG_POSITION from performance_schema.clone_status; 
```
Tiếp tục thự hiện cài đặt sao chép:
```
mysql> change master to
master_host='127.0.0.1',
master_ssl=1,
master_log_file='mysql-bin.000001',
master_log_pos=1405
```
Cuối cùng bạn bật Slave lên :
```
 start slave user='repl_user' password='password';
```
vậy là bạn đã cấu hình Replication thành công .Nếu bạn muốn xem trạng thái của Slave sử dụng câu lệnh:
```
show slave status\G
```
vậy là đã hoàn thành . Hi vọng qua bài viết của mình giúp các bạn có thể hiểu hơn về replication trong Mysql :)))))
