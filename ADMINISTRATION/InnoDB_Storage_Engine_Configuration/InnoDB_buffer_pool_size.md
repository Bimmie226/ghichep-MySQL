# innodb_buffer_pool_size: Configure InnoDB Buffer Pool Size

## Buffer pool
Trong InnoDB, buffer pool là 1 cấu trúc trong bộ nhớ dùng để cache các dữ liệu và chỉ mục được truy cập thường xuyên

Khi bạn truy vấn dữ liệu từ 1 bảng, InnoDB sẽ đọc dữ liệu từ đĩa vào buffer pool. Nếu dữ liệu đã có sẵn trong buffer pool, MySQL có thể truy xuất dữ liệu nhanh chóng từ bộ nhớ thay vì phải thực hiện thao tác I/O trên đĩa vốn tốn nhiều thời gian hơn

Buffer pool có thể cải thiện đáng kể hiệu năng đọc, đặc biệt khi hệ hống có khối lượng truy vấn đọc lớn (high read workloads). Nói chung, buffer pool càng lớn thì hiệu năng của MySQL càng tốt 

Read workload là trường hợp mà các thao tác chính của db là truy xuất dữ liệu sẵn có, tập trung vào việc truy vấn thay vì cập nhật dữ liệu

Để thay đổi kích thước buffer pool, sử dụng biến cấu hình `innodb_buffer_pool_size`

Kích thước tối ưu của buffer pool phụ thuộc vào dung lượng bộ nhớ sẵn có và đặc điểm workload của hệ thống.

Một nguyên tắc chung là bạn nên cấp phát khoảng 80% bộ nhớ khả dụng cho buffer pool trên một server chuyên dụng cho MySQL.

## Checking innodb_buffer_pool_size

Kết nối MySQL:

```bash
mysql -u root -p
```

Hiển thị giá trị hiện tại của tùy chọn `innodb_buffer_pool_size`:

```bash
SHOW GLOBAL VARIABLES LIKE 'innodb_buffer_pool_size';
```

```bash
+-------------------------+-----------+
| Variable_name           | Value     |
+-------------------------+-----------+
| innodb_buffer_pool_size | 134217728 |
+-------------------------+-----------+
1 row in set (0.18 sec)
```

Giá trị của `innodb_buffer_pool_size` được tính bằng byte, ta có thể chuyển sang megabyte bằng câu lệnh:

```bash
SELECT 
   ROUND(@@innodb_buffer_pool_size / 1024 / 1024,0) "innodb_buffer_pool_size (MB)";
```

```bash
+------------------------------+
| innodb_buffer_pool_size (MB) |
+------------------------------+
|                          128 |
+------------------------------+
1 row in set (0.01 sec)
```

Giá trị mặc định của `innodb_buffer_pool_size` trong MySQL 8.0 là 128MB.

## Changing the buffer pool size using the innodb_buffer_pool_size parameter 

Đầu tiên, thay đổi giá trị của innodb_buffer_pool_size trong file cấu hình MySQL thành giá trị lớn hơn, ví dụ 1GB, và lưu lại:

```bash
innodb_buffer_pool_size=1GB
```

Restart MySQL:

```bash
systemctl restart mysql.service
```

Kiểm tra lại:

```bash
mysql> SELECT
    ->    ROUND(@@innodb_buffer_pool_size / 1024 / 1024,0) "innodb_buffer_pool_size (MB)";
+------------------------------+
| innodb_buffer_pool_size (MB) |
+------------------------------+
|                         1024 |
+------------------------------+
1 row in set (0.00 sec)
```

**NOTE:** nếu buffer pool của bạn có kích thước ở mức nhiều GB, bạn nên chia buffer pool thành nhiều instance riêng biệt để cải thiện khả năng xử lý đồng thời. Để làm điều này, bạn điều chỉnh tham số cấu hình `innodb_buffer_pool_instances`.

## Configuring innodb_buffer_pool_size online 
MySQL cho phép bạn thiết lập `innodb_buffer_pool_size` một cách động bằng câu lệnh SET. Ví dụ:

```bash
SET GLOBAL innodb_buffer_pool_size=1073741824;
```

Điều này cho phép bạn thay đổi kích thước buffer pool mà không cần khởi động lại MySQL server. Tuy nhiên, cần lưu ý rằng thay đổi này sẽ không được lưu lại sau khi MySQL restart.