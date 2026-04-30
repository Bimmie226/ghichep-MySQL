# innodb_flush_method: Configure InnoDB Flush Method 

## Introduction to the InnoDB Flush Method 
Phương thức flush của InnoDB quyết định cách dữ liệu được ghi từ bộ nhớ xuống đĩa. Phương thức này có ảnh hưởng lớn đến hiệu năng và độ an toàn dữ liệu 

MySQL sử dụng biến toàn cục `innodb_flush_method` để xác định phương thức flush của InnoDB 

### InnoDB flush methods for Unix-like systems 
Bảng sau liệt kê các phương thức flush chính trên các hệ Unix-like

| Method            | Number | Ý nghĩa                                                                                                                                                                                                                                                          |
| ----------------- | ------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| fsync             | 0      | Phương thức này sử dụng hàm `fsync()` của hệ điều hành để đảm bảo dữ liệu được ghi xuống đĩa, giúp tăng độ bền dữ liệu. Tuy nhiên, nó có thể ảnh hưởng đến hiệu năng do độ trễ I/O. Đây là thiết lập mặc định.                                                   |
| O_DSYNC           | 1      | Phương thức này tương tự `O_DIRECT`, bỏ qua buffer cache nhưng vẫn đồng bộ metadata, tạo sự cân bằng giữa độ bền và hiệu năng. Có thể là lựa chọn tốt nếu hệ thống file hỗ trợ `O_DSYNC` nhưng không hỗ trợ `O_DIRECT`.                                          |
| O_DIRECT          | 4      | Phương thức này bỏ qua buffer cache của hệ điều hành, ghi dữ liệu trực tiếp xuống đĩa, có thể cải thiện hiệu năng ghi bằng cách loại bỏ việc buffering hai lần. Yêu cầu hệ thống file hỗ trợ direct I/O. Có trên một số phiên bản GNU/Linux, FreeBSD và Solaris. |
| O_DIRECT_NO_FSYNC |        | Phương thức này sử dụng `O_DIRECT` cho I/O nhưng bỏ qua lời gọi hệ thống `fsync()` sau mỗi lần ghi.                                                                                                                                                              |

### InnoDB flush methods for Windows

## Checking the InnoDB flush method

```sql
mysql> show global variables like 'innodb_flush_method';
+---------------------+-------+
| Variable_name       | Value |
+---------------------+-------+
| innodb_flush_method | fsync |
+---------------------+-------+
1 row in set (0.01 sec)
```

## Setting the InnoDB Flush Method

Đầu tiên, mở file cấu hình MySQL (`/etc/mysql/mysql.conf.d/mysqld.conf`) và chỉnh sửa phương thức flush:

```bash
innodb_flush_method = O_SYNC
```

Tiếp theo, khởi động lại MySQL:

```bash
systemctl restart mysql.service
```

Kiểm tra:

```sql
mysql> show global variables like 'innodb_flush_method';
+---------------------+---------+
| Variable_name       | Value   |
+---------------------+---------+
| innodb_flush_method | O_DSYNC |
+---------------------+---------+
1 row in set (0.00 sec)
```

