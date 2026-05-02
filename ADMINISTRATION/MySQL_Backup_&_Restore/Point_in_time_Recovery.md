# MySQL Point-in-time Recovery

Tìm hiểu khái niệm `point-in-time recovery` trong MySQL và cách khôi phục database về một thời điểm cụ thể trong quá khứ 

## Introduction
Point-in-time recovery cho phép bạn khôi phục database MySQL về một thời điểm cụ thể trong quá khứ.

Cơ chế này dựa vào 2 thành phần chính:
- Full backup: cung cấp trạng thái ban đầu của database
- Binary logs: ghi lại toàn bộ thay đổi của database, cho phép “phát lại” (replay) đến thời điểm mong muốn

Nếu bạn không có full backup hoặc không bật binary log, thì không thể thực hiện point-in-time recovery.

## Kiểm tra binary log và backup toàn bộ database 

```sql
mysql> show global variables like 'log_bin';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_bin       | ON    |
+---------------+-------+
1 row in set (0.01 sec)
```

### Tạo database mẫu và backup 

Tạo db:

```sql
CREATE DATABASE mydb;

USE mydb;
```

Tạo bảng: 

```sql
CREATE TABLE IF NOT EXISTS contacts (
    id INT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(255) NOT NULL,
    last_name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL
);
```

Insert data:

```sql
INSERT INTO contacts (first_name, last_name, email) 
VALUES
('John', 'Doe', 'john.doe@example.com'),
('Jane', 'Smith', 'jane.smith@example.com'),
('Bob', 'Johnson', 'bob.johnson@example.com');
```

### Backup full

```bash
mysqldump -u root -p mydb > ~/backup/mydb.sql
```

## Thay đổi dữ liệu (giả lập lỗi)

Bước 1: 

```bash
mysql -u root -p
```

Bước 2: 

```sql
USE mydb;
```

Bước 3: Insert thêm

```sql
INSERT INTO contacts VALUES(4, 'Bob','Climo','bob.climo@example.com');
```

Bước 4: Xóa nhầm toàn bộ

```sql
DELETE FROM contacts;
```

- Do quên WHERE → xóa hết dữ liệu

## Kiểm tra binary log 

```sql
SHOW MASTER STATUS;
```

```sql
mysql> SHOW MASTER STATUS;
+---------------+----------+--------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+--------------+------------------+-------------------+
| binlog.000004 |     1893 |              |                  |                   |
+---------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

## Xác định thời điểm lỗi 

Dùng `mysqlbinlog`:

```bash
mysqlbinlog --verbose /var/lib/mysql/binlog.000004 | grep -i -C 10 "delete"
```

```sql
BINLOG '
4cL1aRMBAAAARAAAAH8GAAAAAFoAAAAAAAEABG15ZGIACGNvbnRhY3RzAAQDDw8PBvwD/AP8AwAB
AQACA/z/AAcbXT8=
4cL1aSABAAAAxwAAAEYHAAAAAFoAAAAAAAEAAgAE/wABAAAABABKb2huAwBEb2UUAGpvaG4uZG9l
QGV4YW1wbGUuY29tAAIAAAAEAEphbmUFAFNtaXRoFgBqYW5lLnNtaXRoQGV4YW1wbGUuY29tAAMA
AAADAEJvYgcASm9obnNvbhcAYm9iLmpvaG5zb25AZXhhbXBsZS5jb20ABAAAAAMAQm9iBQBDbGlt
bxUAYm9iLmNsaW1vQGV4YW1wbGUuY29tA2dslw==
'/*!*/;
### DELETE FROM `mydb`.`contacts`
### WHERE
###   @1=1
###   @2='John'
###   @3='Doe'
###   @4='john.doe@example.com'
### DELETE FROM `mydb`.`contacts`
### WHERE
###   @1=2
###   @2='Jane'
###   @3='Smith'
###   @4='jane.smith@example.com'
### DELETE FROM `mydb`.`contacts`
### WHERE
###   @1=3
###   @2='Bob'
###   @3='Johnson'
###   @4='bob.johnson@example.com'
### DELETE FROM `mydb`.`contacts`
### WHERE
###   @1=4
###   @2='Bob'
###   @3='Climo'
###   @4='bob.climo@example.com'
# at 1862
#260502 16:24:49 server id 1  end_log_pos 1893 CRC32 0xf373f663         Xid = 58
COMMIT/*!*/;
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
DELIMITER ;
```

- Thời điểm xóa: 2026-05-02 16:24:49

## Thực hiện Point-in-time Recovery

Bước 1: Restore full backup

```bash
mysql -u root -p mydb < ~/backup/mydb.sql
```

Bước 2: Replay binary log đến thời điểm trước lỗi

```bash
mysqlbinlog --stop-datetime="2026-05-02 16:24:49" --verbose /var/lib/mysql/binlog.000004 | mysql -u root -p
```

- `mysqlbinlog`: đọc binary log và in ra các câu SQL
- `| mysql`: chạy các câu SQL đó 

Bước 3: Kiểm tra kết quả

```sql
mysql> select * from contacts
    -> ;
+----+------------+-----------+-------------------------+
| id | first_name | last_name | email                   |
+----+------------+-----------+-------------------------+
|  1 | John       | Doe       | john.doe@example.com    |
|  2 | Jane       | Smith     | jane.smith@example.com  |
|  3 | Bob        | Johnson   | bob.johnson@example.com |
|  4 | Bob        | Climo     | bob.climo@example.com   |
+----+------------+-----------+-------------------------+
4 rows in set (0.00 sec)
```

