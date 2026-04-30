# innoDB_log_buffer_size: Configure InnoDB Log Buffer Size

Trong InnoDB, log buffer là một cấu trúc bộ nhớ dùng để lưu trữ các thay đổi dữ liệu trước khi chúng được ghi xuống các file log trên đĩa

Một kích thước log buffer tối ưu có thể cải thiện hiệu năng bằng cách giảm tần suất các thao tác I/O tốn kém 

Kích thước của log buffer được kiểm soát bởi biến cấu hình `innodb_log_buffer_size`

Giá trị mặc định của log buffer thường phù hợp với hầu hết các ứng dụng. Tuy nhiên, nếu bạn có một ứng dụng có cường độ ghi (write-intensive) cao, bạn có thể cần điều chỉnh kích thước buffer dựa trên dung lượng bộ nhớ của server và đặc điểm của ứng dụng.

Để kiểm tra xem bạn có cần điều chỉnh kích thước log buffer hay không, bạn có thể sử dụng biến trạng thái `innodb_log_waits`.

`InnoDB Log Waits` xảy ra khi một transaction không thể ghi vào log buffer vì buffer đã đầy. Biến `innodb_log_waits` lưu số lần mà tình huống chờ này xảy ra.

Nếu giá trị của `innodb_log_waits` lớn hơn 0, thì nhiều khả năng bạn cần tăng kích thước của log buffer.

## Checking InnoDB Log Waits

```sql
mysql> SHOW GLOBAL STATUS LIKE 'Innodb_log_waits';
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| Innodb_log_waits | 0     |
+------------------+-------+
1 row in set (0.02 sec)
```

- Giá trị bằng `0` -> kích thước `log buffer` hiện tại là đủ 
- Nếu giá trị > `0` -> cần tăng kích thước của `log buffer` lên mức lớn hơn 

## Configure the InnoDB Log Buffer Size: innodb_log_buffer_size 

Đầu tiên, mở file cấu hình MySQL (`/etc/mysql/mysql.conf.d/mysqld.conf`) và thiết lập giá trị dựa trên bộ nhớ khả dụng của hệ thống và yêu cầu workload:

```bash
innodb_log_buffer_size=128MB
```

Tiếp theo, khởi động lại MySQL server:

```bash
systemctl restart mysql.service
```

Kiểm tra: 

```sql
mysql> SHOW GLOBAL VARIABLES LIKE 'innodb_log_buffer_size';
+------------------------+-----------+
| Variable_name          | Value     |
+------------------------+-----------+
| innodb_log_buffer_size | 134217728 |
+------------------------+-----------+
1 row in set (0.01 sec)

mysql> SELECT ROUND(@@innodb_log_buffer_size / 1024 / 1024, 0);
+--------------------------------------------------+
| ROUND(@@innodb_log_buffer_size / 1024 / 1024, 0) |
+--------------------------------------------------+
|                                              128 |
+--------------------------------------------------+
1 row in set (0.00 sec)
```

