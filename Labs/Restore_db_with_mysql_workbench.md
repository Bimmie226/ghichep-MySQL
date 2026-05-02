# Import file sql vào trong MySQL WorkBench

Bài viết này sẽ giúp các bạn Restore lại Database thông qua file `db_name.sql`

Các bạn có thể tải các file `db_name.sql` mà mình đã lưu:

| STT | Database Name                                                                                                                               |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | [e_commerce](https://github.com/Bimmie226/Database-Management-Systems---PTIT/blob/main/Store%20backup%20db%20file/e_commerce_backup.sql.gz) |
| 2   | [cinema](https://github.com/Bimmie226/Database-Management-Systems---PTIT/blob/main/Store%20backup%20db%20file/cinema_backup.sql.gz)         |

## 0. Chuẩn bị
Trước khi vào thực hiện bạn hãy chắc chắn cho mình rằng trên máy của bạn đã có 2 thứ:

- MySQL WorkBench
- MySQL server

Nếu chưa có bạn hãy thực hiện theo mình:

### 0.1 Download MySQL installer

Các bạn có thể download [tại đây](https://dev.mysql.com/downloads/installer/)

![alt text](../images/import_01.png)

### 0.2 Download MySQL WorkBench & Server

Mở `MySQL Installer` trên máy và thực hiện lần lượt các bước sau:

![alt text](../images/import_02.png)

![alt text](../images/import_03.png)

Nhấp vào dấu `+` của 2 trường này:

![alt text](../images/import_06.png)

![alt text](../images/import_07.png)

Nhấp vào `next` sau đó chọn `execute` sau đó `next` tiếp đến đoạn nhập mk `root`(HÃY LUÔN GHI NHỚ MK NÀY):

![alt text](../images/import_08.png)

Nhập mật khẩu sau đó chọn `next` đến cuối và `finish`

## 1. Import file `.sql` trong MySQL WorkBench

![alt text](../images/import_09.png)

Nhấp vào dấu `+` để tạo 1 connection

![alt text](../images/import_10.png)

- `1`: Nhập tên của connection này
- `2`: IP của host (Ở đây do không có server riêng nên ta sẽ để là localhost)
- `3`: Port mặc định của MySQL
- `4`: Ta login bằng user nào (Ở đây là `root`)
- `5`: Click vào đây để nhập mk (Là mk `root` ta tạo ở trên)

![alt text](../images/import_11.png)

![alt text](../images/import_12.png)

Nhấp vào connection này

![alt text](../images/import_14.png)

Ta sẽ tạo 1 database mới ở đây:
- Nếu bạn tải file `cinema.sql.gzip` bạn nên đặt tên là `cinema` 

![alt text](../images/import_15.png)

Chạy và sau đó sử dụng database này:

![alt text](../images/import_13.png)


Chọn `Server` -> `Data Import` để Import file `.sql` (Lưu ý là bạn nhớ giải nén file `.sql` bạn tải về ở trên nhé)

![alt text](../images/import_16.png)

Ta chọn `import from self-contained file` và đường dẫn đén file `.sql` đã lưu trên máy

Chọn `Default Target Schema` và chọn db bạn vừa tạo (Ở đây là `e_commerce`)

![alt text](../images/import_17.png)

Sau đó chọn `Import Progress` và `Start Import`

![alt text](../images/import_18.png)

Ok, như vậy ta đã import thành công. Ta sẽ thử viết 1 query để check:

![alt text](../images/import_19.png)

![alt text](../images/import_20.png)

Như vậy ta đã import thành công và đã restore lại được database. Chúc các bạn thành công!!