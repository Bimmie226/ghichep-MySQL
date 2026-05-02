# mysqldump 

## Introduction 

`mysqldump` dùng để tạo backup cho các database MySQL 

Công cụ này cho phép bạn xuất (dump) cấu trúc và/hoặc dữ liệu của một hoặc nhiều database ra file, sau đó có thể dùng file này để khôi phục lại database.

Trong thực tế, `mysqldump` thường dùng cho:
- backup và restore
- migration database
- chuyển database giữa các server 

## Cú pháp cơ bản của `mysqldump`

### 1. Dump một hoặc nhiều bảng

```bash
mysqldump -u root -p [options] db_name [tbl_name ...] > output_file
```

### 2. Dump một hoặc nhiều database

```bash
mysqldump -u root -p [options] --databases db_name ... > output_file
```

### 3. Dump toàn bộ database 

```bash
mysqldump -u root -p [options] --all-databases > output_file
```

## Các option của `mysqldump`

| Option              | Short | Ý nghĩa                                                                            |
| ------------------- | ----- | ---------------------------------------------------------------------------------- |
| --user              | -u    | username                                                                           |
| --password          | -p    | password                                                                           |
| --add-drop-database |       | thêm `DROP DATABASE` trước `CREATE DATABASE`                                       |
| --add-drop-table    |       | thêm `DROP TABLE` trước `CREATE TABLE`                                             |
| --add-drop-trigger  |       | thêm `DROP TRIGGER` trước `CREATE TRIGGER`                                         |
| --add-locks         |       | thêm `LOCK TABLES` và `UNLOCK TABLES`                                              |
| --all-databases     | -A    | dump tất cả database                                                               |
| --databases         | -B    | dump nhiều database                                                                |
| --no-data           | -d    | chỉ dump structure (không có data)                                                 |
| --result-file       | -r    | ghi output vào file                                                                |
| --routines          | -R    | include stored procedure/function                                                  |
| --no-create-db      | -n    | bỏ `CREATE DATABASE`                                                               |
| --no-create-info    | -t    | không include `CREATE TABLE` - Chỉ backup dữ liệu thôi, KHÔNG backup cấu trúc bảng |

