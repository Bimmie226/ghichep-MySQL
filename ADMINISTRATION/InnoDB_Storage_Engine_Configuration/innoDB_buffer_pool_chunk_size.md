# innodb_buffer_pool_chunk_size: Configure Buffer Pool Chunk Size 

## Introduction to buffer pool chunk size configuration variable innodb_buffer_pool_chunk_size 

Buffer pool trong innodb là một cấu trúc trong bộ nhớ dùng để lưu trữ dữ liệu được truy cập thường xuyên 

InnoDB sử dụng buffer pool để cải thiện hiệu năng cơ sở dữ liệu bằng cách truy cập dữ liệu trực tiếp từ bộ nhớ thay vì đọc từ đĩa.

Biến cấu hình `innodb_buffer_pool_size` xác định kích thước của buffer pool.

Nếu buffer pool có kích thước lớn (ở mức nhiều GB), bạn có thể bật nhiều buffer pool bằng biến cấu hình `innodb_buffer_pool_instances` để cải thiện khả năng xử lý đồng thời (concurrency).

Biến `innodb_buffer_pool_chunk_size` là một tùy chọn cấu hình xác định kích thước chunk được sử dụng khi thay đổi kích thước buffer pool.

`innodb_buffer_pool_chunk_size` ảnh hưởng gián tiếp đến kích thước buffer pool, vì tổng kích thước buffer pool là bội số của: `innodb_buffer_chunk_size * innodb_buffer_pool_instances`

Trong MySQL 8.0, giá trị mặc định của `innodb_buffer_pool_chunk_size` là 128MB. Phạm vi hợp lệ của nó là từ 1MB đến 512MB. Để tránh các vấn đề về hiệu năng, số lượng chunk không nên vượt quá 1000.

Khi thiết lập biến `innodb_buffer_pool_chunk_size`, tích của: `innodb_buffer_pool_chunk_size * innodb_buffer_pool_instances` không được vượt quá `innodb_buffer_pool_size`. Nếu vượt quá MySQL sẽ tự động cắt giảm giá trị về: `innodb_buffer_pool_size / innodb_buffer_pool_instances`

Nếu bạn thay đổi innodb_buffer_pool_chunk_size, MySQL sẽ tự động điều chỉnh innodb_buffer_pool_size để bằng hoặc là bội số của: `innodb_buffer_pool_chunk_size * innodb_buffer_pool_instances`

trong quá trình khởi tạo buffer pool: `innodb_buffer_pool_size = n * (innodb_buffer_pool_chunk_size * innodb_buffer_pool_instances)`

## Checking innodb_buffer_pool_chunk_size value 

```bash
mysql> SHOW GLOBAL VARIABLES LIKE 'innodb_buffer_pool_chunk_size';
+-------------------------------+-----------+
| Variable_name                 | Value     |
+-------------------------------+-----------+
| innodb_buffer_pool_chunk_size | 134217728 |
+-------------------------------+-----------+
1 row in set (0.01 sec)
```

Giá trị chunk size được tính theo byte. Mặc định, `innodb_buffer_pool_chunk_size` là `134217728` byte, tương đương `128MB`.

## Configuring innodb_buffer_pool_chunk_size 
Để thiết lập giá trị cho innodb_buffer_pool_chunk_size, bạn cần thay đổi trong file cấu hình MySQL:

```bash
[mysqld]
innodb_buffer_pool_chunk_size=134217728
```

