# innodb_buffer_pool_instances: Configuring Multiple Buffer Pool Instances for Improved Concurrency in MySQL 

## Buffer pool & concurrency 

Trong InnoDB, buffer pool là một cấu trúc trong bộ nhớ dùng để lưu trữ các dữ liệu và chỉ mục được truy cập thường xuyên 

InnoDB sử dụng buffer pool để cải thiện hiệu năng của MySQL bằng cách truy cập dữ liệu trực tiếp trong bộ nhớ thay vì đọc từ đĩa 

Thông thường, bạn cấp càng nhiều bộ nhớ cho buffer pool thì hiệu năng của MySQL càng tốt.

Để thiết lập kích thước tối ưu cho buffer pool, bạn điều chỉnh biến cấu hình `innodb_buffer_pool_size`.

Khi buffer pool có kích thước lớn (ở mức nhiều GB), MySQL có thể phục vụ rất nhiều yêu cầu đữ liệu bằng cách lấy trực tiếp từ bộ nhớ. Trong trường hợp này bạn có thể gặp bottleneck do nhiều thread cùng lúc cố gắng truy cập buffer pool 

Để giảm sự tranh chấp, bạn có thể bật nhiều buffer pool. Biến cấu hình `innodb_buffer_pool_instances` cho phép bạn chỉ định số lượng instance của buffer pool 

Mặc định, `innodb_buffer_pool_instances = 1`. Để sử dụng nhiều buffer pool, bạn đặt giá trị này lớn hơn 1. Bạn có thể thiết lập tối đa lên đến 64.

**NOTE:** `innodb_buffer_pool_instances` chỉ có hiệu lực khi `innodb_buffer_pool_size` được đặt từ 1GB trở lên 

## Configuring buffer pool instances 
Mở file cấu hình MySQL và thiết lập `innodb_buffer_pool_instances` thành một giá trị (Ví du: `2`) 

```bash
[mysqld]
innodb_buffer_pool_size=2GB
innodb_buffer_pool_instances=2
```

Lưu file cấu hình và restar MySQL:

```bash
systemctl restart mysql.service
```

Kiểm tra: 

```bash
mysql> SHOW VARIABLES LIKE 'innodb_buffer_pool_instances';
+------------------------------+-------+
| Variable_name                | Value |
+------------------------------+-------+
| innodb_buffer_pool_instances | 2     |
+------------------------------+-------+
1 row in set (0.02 sec)
```

