# MySQL Backup 

## MySQL backup types 

Trong MySQL, bạn có thể sao lưu toàn bộ database server, bao gồm cả database và các file cấu hình; loại này được gọi là `physical backup`

Ngoài ra, bạn cũng có thể sao lưu các database hoặc bảng cụ thể, gọi là `logical backup`

### Logical backups

Logical backup cho phép bạn tái tạo lại cấu trúc bảng và dữ liệu mà không cần sao chép trực tiếp các file dữ liệu vật lý. Nó xuất cấu trúc và dữ liệu database thành các câu lệnh SQL 

Một công cụ phổ biến để thực hiện logical backup là `mysqldump`

Các công cụ này sẽ tạo ra file backup chứa các câu lệnh `CREATE TABLE` và `INSERT`, cho phép khôi phục lại bảng và dữ liệu.

**Cú pháp:**

```bash
mysqldump -u [username] -p [password] [database_name] > backup.sql
```

Logical backup mang lại sự linh hoạt cao, nhưng thời gian restore thường chậm hơn so với physical backup.

### Physical backups
Physical backup sao chép trực tiếp các file dữ liệu nhị phân, tạo ra bản sao chính xác của database tại một thời điểm cụ thể. Vì vậy, việc restore từ physical backup sẽ nhanh hơn so với logical backup.

Công cụ dùng cho physical backup là `Percona XtraBackup`. Đây là một công cụ miễn phí, mã nguồn mở và mạnh mẽ dành cho MySQL.

**Cú pháp:**

```bash
xtrabackup --backup --user=[username] --password=[password] --target-dir=/path/to/backup
```

## Hot & Cold backups
Ngoài việc phân loại theo logical và physical, bạn cũng có thể phân loại backup dựa trên trạng thái hoạt động của database trong quá trình backup 

Hot backup và Cold backup đề cập đến cách database được xử lý trong khi đang thực hiện backup 

### Hot backups 

Hot backup được thực hiện khi database vẫn đang chạy và phục vụ request. Phương pháp này đảm bảo hệ thống luôn sẵn sàng và giảm thiểu gián đoạn cho người dùng trong quá trình backup.

`Percona XtraBackup` có thể thực hiện hot backup, cho phép sao lưu trạng thái database mà không cần khóa bảng (lock tables), đồng thời vẫn giữ database hoạt động.

```bash
xtrabackup --backup --user=[username] --password=[password] --target-dir=/path/to/backup
```

### Cold backups 
Cold backup được thực hiện khi database đang offline và không có thao tác ghi nào xảy ra. Điều này đảm bảo một bản snapshot nhất quán của toàn bộ database tại thời điểm backup.

**Các bước thực hiện:**

1. Dừng MySQL server: 

  ```bash
  systemctl stop mysql.service
  ```

2. Sao lưu database ra file:

  ```bash
  mysqldump -u [username] -p[password] [database_name] > backup.sql
  ```

3. Khởi động lại MySQL server:

  ```bash
  systemctl restart mysql.service
  ```

  