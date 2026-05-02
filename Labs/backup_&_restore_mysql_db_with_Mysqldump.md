# Back Up và Restore MySQL Databases với Mysqldump
Tệp backup được tạo ra bởi mysqldump về cơ bản là 1 tập lệnh mysql có thể được sử dụng để tạo lại bản gốc của cơ sở dữ liệu được backup. Mysqldump cũng có thể tạo ra các tệp ở định dạng CSV và XML

Bạn cũng có thể sử dụng mysqldump để chuyển cơ sở dữ liệu MySQL của bạn sang một máy chủ MySQL khác

Nếu bạn không backup cơ sở dữ liệu của mình, lỗi phần mềm hoặc lỗi ổ cứng có thể là thảm họa khiến hỏng toàn bộ cơ sở dữ liệu và không thể khôi phục chúng được. Để giúp bạn tiết kiệm nhiều thời gian, tôi khuyên bạn nên thường xuyên sao lưu cơ sở dữ liệu của mình.

## I. Cú pháp mysqldump

Trước khi đi vào cách sử dụng mysqldump, hãy bắt đầu bằng cách xem lại cú pháp cơ bản

Cú pháp cơ bản:

```sql
mysqldump [options] > file.sql
```

- `options` - option của mysqldump
- `file.sql` - tệp được sao lưu

Để sử dụng được myqldump, phải truy cập được máy chủ mysql.

## II. Backup cơ sở dữ liệu đơn giản

Ta có thể dùng mysqldump để sao lưu 1 csdl

Ví dụ: Tạo 1 bản backup có tên là `mydb` bằng cách sử dụng quyền user là root và lưu nó vào tệp có tên là `mydb_backup.sql` ta sử dụng câu lệnh sau:

```bash
mysqldump -u root -p mydb > mydb_backup.sql
```

Nếu bạn login bằng 1 user khác, cũng có quyền sao lưu mà không cần mật khẩu, bạn có thể bỏ qua option `-u` và `-p`

## III. Backup nhiều databases

Để backup nhiều cơ sở dữ liệu bằng 1 lệnh, bạn cần sử dụng option `--databases` và theo sau đó là các cơ sở dữ liệu bạn muốn sao lưu. 

```bash
mysqldump -u root -p --databases db_name_a db_name_b > db_a_b_backup.sql
```

Lệnh trên sẽ tạo ra 1 tệp sao lưu cả cơ sở dữ liệu a và b

## IV. Backup tất cả các cơ sở dữ liệu trong server

```bash
mysqldump -u root -p --all-databases > all_db_backup.sql
```


## V. Backup tất cả db vào nhiều file

mysqldump không cung cấp tùy chọn sao lưu tất cả cơ sở dữ liệu vào nhiều file nhưng bạn có thể dễ dàng thực hiện nó bằng cách sử dụng vòng lặp FOR ở trong bash:

```bash
for DB in $(mysql -e 'show databases' -s --skip-column-names); do
    mysqldump "$DB" > "${DB}_backup.sql"
done
```

## VI. Tạo 1 bản nén sao lưu db

Nếu kích thước db lớn, việc nén lại sẽ là 1 ý tưởng tốt để tạo ra file dump. 

```bash
mysqldump db_name | gzip > db_name.sql.gz
```

## VII. Tạo ra bản backup với timestamp

Nếu bạn muốn giữ nhiều bản backup trong cùng 1 folder, bạn có thể thêm ngày hiện tại để phân biệt ngày sao lưu giữa các tệp:

```bash
mysqldump db_name > db_name-$(date +%Y%m%d).sql
```

## VIII. Restore lại db đã backup

```bash
mysqld  database_name < file.sql
```

Trong hầu hết các trường hợp, bạn sẽ cần phải tạo cơ sở dữ liệu đã tồn tại, thì việc trước tiên bạn cần xóa chúng.

Trong ví dụ sau, lệnh đầu tiên sẽ tạo ra một cơ sở dữ liệu có tên là `database_name` và sau đó sẽ import file backup `database_name.sql` vào nó:


```bash
mysql -u root -p -e "create database database_name";
mysql -u root -p database_name < database_name.sql
```

## IX. Restore lại 1 db từ 1 file sao lưu all db

```bash
mysql --one-database database_name < all_databases.sql
```

## X. Export hoặc Import 1 db bằng 1 dòng lệnh

```bash
mysqldump -u root -p database_name | mysql -h remote_host -u root -p remote_database_nam
```

## XI. Tự động backup với cron job

Tự động hóa sao lưu cơ sở dữ liệu cũng đơn giản như là một công việc định kì, lệnh mysqldump sẽ được chạy ở thời gian bạn chỉ định.

**NOTE:** Cần có file `~/.my.cnf` 

### Cronjob thực hiện backup và nén cơ sở dữ liệu vào 3h sáng mỗi ngày

Script backup + nén:

```bash
vi /usr/local/bin/mysql_backup.sh
```

```bash
#!/bin/bash

BACKUP_DIR="/backup/mysql"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

for DB in $(mysql -e "SHOW DATABASES" -s --skip-column-names \
            | grep -Ev "(information_schema|mysql|performance_schema|sys)"); do
    
    mysqldump  --single-transaction --quick "$DB" \
        | gzip > "$BACKUP_DIR/${DB}_backup_${DATE}.sql.gz"

done

exit 0
```

Cấp quyền thực thi:

```bash
chmod +x /usr/local/bin/mysql_backup.sh
```


Tạo cronjob chạy lúc 3:00 sáng mỗi ngày

```bash
crontab -e
```

Thêm dòng sau:

```bash
0 3 * * * /usr/local/bin/mysql_backup.sh >> /var/log/mysql_backup.log 2>&1
```

- `>> .log`: ghi log
- `2>&1`: gộp stderr vào stdout

### Cronjob thực hiện xóa backup sau 30 ngày

```bash
vi /usr/local/bin/cleanup_mysql_backup.sh
```

```bash
#!/bin/bash

BACKUP_DIR="/backup/mysql"
DAYS=30

find "$BACKUP_DIR" -type f \
  \( -name "*.sql" -o -name "*.sql.gz" \) \
  -mtime +$DAYS \
  -exec rm -f {} \;

exit 0
```

Cấp quyền thực thi:

```bash
chmod +x /usr/local/bin/cleanup_mysql_backup.sh
```

```bash
crontab -e

# Thêm dòng sau vào
30 2 * * * /usr/local/bin/cleanup_mysql_backup.sh >> /var/log/cleanup_mysql_backup.log 2>&1
```