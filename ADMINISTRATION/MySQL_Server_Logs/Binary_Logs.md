# MySQL Binary Logs 

## Introduction to the MySQL binary logs 

`Binary log` là các file lưu trữ những thay đổi đối với db MySQL. Các log này chứa một chuỗi các sự kiện (events) đại diện cho các thay đổi dữ liệu và các đối tượng trong db như bảng, view, database, v.v.

**Ví dụ:** Khi bạn tạo một bảng mới, MySQL sẽ ghi lại câu lệnh `CREATE TABLE` tương ứng vào binary log. Tương tự, khi bạn xóa một dòng trong bảng, MySQL sẽ ghi lại câu lệnh `DELETE` vào binary log.

Tuy nhiên, nếu bạn thực thi một câu lệnh `SELECT`, MySQL sẽ không ghi nó vào binary log, vì `SELECT` không làm thay đổi dữ liệu trong database.

**MySQL sử dụng binary log cho các mục đích sau:**

- Replication (nhân bản dữ liệu): Binary log cung cấp một cách đáng tin cậy và hiệu quả để đồng bộ dữ liệu giữa các MySQL server.
- Recovery (khôi phục dữ liệu): Binary log đóng vai trò quan trọng trong việc khôi phục dữ liệu đến một thời điểm cụ thể (`point-in-time recovery`).

**Các loại binary log trong MySQL:**

- Statement-based (STATEMENT): ghi log dựa trên câu lệnh SQL 
- Row-based (ROW): ghi log dựa trên từng dòng dữ liệu bị thay đổi.
- Mixed format (MIXED): sử dụng kết hợp cả hai định dạng trên.

Mỗi định dạng đều có ưu điểm và trường hợp sử dụng riêng.

**NOTE:** Việc bật binary logging sẽ tạo thêm overhead, có thể ảnh hưởng đến hiệu năng. Vì vậy, bạn cần theo dõi chặt chẽ. Ngoài ra, bạn có thể muốn [tắt nó trên các môi trường development và test](https://www.mysqltutorial.org/mysql-administration/mysql-disable-binary-logs/).

## Binary log configuration 

Dưới đây là các cấu hình binary log thường gặp:

### 1. Enabling binary logging 

MySQL mặc định đã bật binary log.

```sql
mysql> show global variables like 'log_bin';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_bin       | ON    |
+---------------+-------+
1 row in set (0.00 sec)
```

Nếu chưa enable ta sẽ chỉnh sửa trong file config MySQL (`/etc/mysql/mysql.conf.d/mysqld.conf`)

```bash
log-bin=mysql-bin
```

### 2. Binary log format 
Biến `binlog_format` lưu định dạng binary log 

```sql
mysql> show global variables like 'binlog_format';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| binlog_format | ROW   |
+---------------+-------+
1 row in set (0.01 sec)
```

=> MySQL đang sử dụng định dạng binary log kiểu `ROW`

Để chỉ định định dạng khác, bạn thêm dòng sau vào file cấu hình:

```bash
binlog_format=STATEMENT
```

Các giá trị hợp lệ gồm: `ROW`, `STATEMENT`, và `MIXED`.

### 3. Binary Log Location 

Mặc định, MySQL lưu binary log trong thư mục data.

Nếu bạn muốn lưu ở vị trí khác, bạn có thể chỉ định đường dẫn tuyệt đối trong file cấu hình:

```bash
log-bin=/absolute/path/to/binary/logs/
```

### 4. Binary Log Retention

Binary log có thể chiếm nhiều dung lượng đĩa theo thời gian. Để tránh vấn đề về lưu trữ, bạn có thể kiểm soát thời gian tồn tại của các file log thông qua chính sách retention.

Bạn có thể thiết lập:

- số ngày lưu log
- kích thước tối đa mỗi file log
- số lượng file log

**Ví dụ:**

```bash
expire_logs_days=7
```

```bash
max_binlog_size=100M
```

- Thiết lập kích thước tối đa của mỗi file binary log là 100MB. Bạn có thể dùng đơn vị: byte, KB (K), MB (M), hoặc GB (G).

```bash
max_binlog_files=10
```

- Thiết lập số lượng tối đa file binary log là 10. Khi đạt giới hạn này, MySQL sẽ xóa (purge) các file log cũ hơn để tạo chỗ cho file mới.

### 5. Encryption of binary logs

Để tăng cường bảo mật, bạn có thể mã hóa binary log bằng tùy chọn:

```bash
encrypt-binlog=1
```

## Binary log statements 

Dưới đây là các câu lệnh phổ biến nhất để làm việc với binary log:

### 1. SHOW BINARY LOGS 
Để hiển thị danh sách các binary log hiện có

```sql
mysql> SHOW BINARY LOGS;
+---------------+-----------+-----------+
| Log_name      | File_size | Encrypted |
+---------------+-----------+-----------+
| binlog.000001 |       180 | No        |
| binlog.000002 |       404 | No        |
| binlog.000003 |      1022 | No        |
| binlog.000004 |     50732 | No        |
| binlog.000005 |       180 | No        |
| binlog.000006 |       180 | No        |
| binlog.000007 |       180 | No        |
| binlog.000008 |       180 | No        |
| binlog.000009 |       157 | No        |
| binlog.000010 |       180 | No        |
| binlog.000011 |       157 | No        |
+---------------+-----------+-----------+
11 rows in set (0.00 sec)
```

### 2. SHOW MASTER STATUS 

Để hiển thị thông tin về binary log của server nguồn (source server) và vị trí replication

```sql
mysql> SHOW MASTER STATUS;
+---------------+----------+--------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+--------------+------------------+-------------------+
| binlog.000011 |      157 |              |                  |                   |
+---------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

### 3. PURGE BINARY LOGS

Để xóa các binary log cũ dựa trên một file cụ thể, bạn dùng câu lệnh `PURGE BINARY LOGS`. Ví dụ sau sẽ xóa các file `000001` và `000002`:
- Xóa TẤT CẢ binary log CŨ HƠN file được chỉ định
```sql
PURGE BINARY LOGS TO 'binlog.000003';
```

### 4. FLUSH BINARY LOGS

Để buộc MySQL đóng file log hiện tại và tạo file log mới, bạn sử dụng:

```sql
FLUSH BINARY LOGS;
```

Nếu bạn thực thi một câu lệnh làm thay đổi database, bạn sẽ thấy một file log mới được tạo.

## The mysqlbinlog utility: examining binary log files

Để xem các câu lệnh SQL được ghi trong binary log, bạn sử dụng công cụ `mysqlbinlog`.

**Ví dụ:**

```bash
mysqlbinlog /var/lib/mysql/binlog.000003
```

Nếu muốn ghi ra 1 file khác:

```bash
mysqlbinlog /var/lib/mysql/binlog.000003 > ~/binlog.txt
```

