# innodb_dedicated_server: Configure InnoDB Dedicated Server

InnoDB cung cấp tùy chọn `innodb_dedicated_server`, cho phép tự động cấu hình các biến quan trọng. 

Bạn chỉ nên bật `innodb_dedicated_server` nếu bạn có một instance MySQL chạy trên server chuyên dụng (dedicated server), nơi MySQL có thể sử dụng toàn bộ tài nguyên hệ thống.

Không khuyến khích bật `innodb_dedicated_server` khi MySQL chạy trên một server phải chia sẻ tài nguyên với các ứng dụng khác.

## Enabling innodb_dedicated_server

Mở file cấu hình MySQL

```bash
innodb_dedicated_server=ON
```

## Verifying the configuration

```sql
SHOW GLOBAL VARIABLES LIKE 'innodb_dedicated_server';
+-------------------------+-------+
| Variable_name           | Value |
+-------------------------+-------+
| innodb_dedicated_server | ON    |
+-------------------------+-------+
1 row in set (0.03 sec)
```

