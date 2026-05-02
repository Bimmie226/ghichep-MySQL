# MySQL Slow Query Logs

## Introduction to MySQL slow query logs 

Khi bạn thực thi các truy vấn mất nhiều thời gian hơn một ngưỡng được chỉ đỉnh, MySQL sẽ coi chúng là `slow queries`

Sau đó, MySQL sẽ ghi các truy vấn chậm này vào các file log gọi là `slow query logs`

Các `slow query logs` là một công cụ rất quan trọng giúp bạn phân tích và tối ưu hiệu năng của cơ sở dữ liệu.

## MySQL slow query log configurations
### 1. Checking the current configuration

Sử dụng lệnh sau để check cấu hình `slow query log` hiện tại:

```sql
mysql> SHOW VARIABLES LIKE '%slow_query%';
+---------------------+-------------------------------------+
| Variable_name       | Value                               |
+---------------------+-------------------------------------+
| slow_query_log      | OFF                                 |
| slow_query_log_file | /var/lib/mysql/ecommercesv-slow.log |
+---------------------+-------------------------------------+
2 rows in set (0.01 sec)
```

- `slow_query_log`: cho biết slow query log có được bật hay không
- `slow_query_log_file`: Chỉ định file log nơi mysql ghi các truy vấn chậm 


### 2. Setting up a slow query log file

```bash
mkdir -p /var/log/mysql
touch /var/log/mysql/ecommercesv-slow.log
chown -R mysql:mysql /var/log/mysql/
```

### 3. Enabling slow query logs

```bash
[mysqld]
slow_query_log = 1
slow_query_log_file = /var/log/mysql/ecommercesv-slow.log
long_query_time = 1
```

- `slow_query_log = 1`: bật log
- `slow_query_log_file`: đường dẫn file log
- `long_query_time = 1`: truy vấn > 1s sẽ bị ghi log 

Sau đó, restart MySQL để áp dụng.

### 4. Executing a slow query

```sql
select sleep(3);
```

- Query này mất 3 giây → sẽ bị log (vì > 1 giây)

### 5. Analyzing slow queries

Xem log:

```bash
cat /var/log/mysql/ecommercesv-slow.log
```

```bash
/usr/sbin/mysqld, Version: 8.0.42-0ubuntu0.20.04.1 ((Ubuntu)). started with:
Tcp port: 3306  Unix socket: /var/run/mysqld/mysqld.sock
Time                 Id Command    Argument
# Time: 2026-05-02T07:23:34.945075Z
# User@Host: root[root] @ localhost []  Id:     8
# Query_time: 3.011367  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 1
SET timestamp=1777706611;
select sleep(3);
```

- Dòng cuối là query chậm 

Dùng `mysqldumpslow` để phân tích:

```bash
mysqldumpslow /var/log/mysql/ecommercesv-slow.log
```

```bash
Reading mysql slow query log from /var/log/mysql/ecommercesv-slow.log
Count: 1  Time=3.01s (3s)  Lock=0.00s (0s)  Rows=1.0 (1), root[root]@localhost
  select sleep(N)
```

- `Count`: số lần query xuất hiện
- `Time`: thời gian trung bình
- `Lock`: thời gian chờ lock
- `Rows`: số dòng xử lý 
- `select sleep(N)`: query được chuẩn hóa (ẩn giá trị cụ thể)



