# Kiến trúc, mối tương quan giữa các thành phần trong Freerer
![hinh anh](http://image.prntscr.com/image/6e41f527b7894992a6dae42504c9596a.png)

## 1. Backup mount local
Trong trường hợp không sử dụng API, sau khi lên lịch backup, Freezer Schedule gửi yêu cầu backup đên Freezer Agent thực hiện backup. Tiến trình backup thực hiện sao lưu dữ liệu xuống phân vùng mount (có thể là NFS, GlusterFS). Dữ liệu được load lên RAM máy local trước khi được nghi xuống phân vùng mount

## 2. Backup remote FS
Khi thực hiện quá trình backup, ngoài việc sao lưu dữ liệu đến phân vùng mount ta có thể đẩy dữ liệu backup đến một server khác thông qua SSH.

## 3.
Người dùng hoặc admin gửi yêu cầu thực hiện backup trên horizon. 

## 4. Xác thực với keystone
Xác thực với Keystone quyền hạn của user

## 5.  Gọi Feezer API
Yêu cầu được gửi đến thành phần Freezer API

## 6. Ghi database
Frezzer nhận được yêu cầu, cập nhật trạng thái và ghi vào database Elasticsearch

## 7. Freezer Scheduler nhận jobs
Freezer quản lý job backup bằng gửi lời gọi đến Freezer API để lấy jobs backup.

## 8. Xác thực với Keystone
Freezer API xác thực với Keystone, cấp quyền backend vào Swift cho Freezer Agent

## 9. Feezer agent thực hiện backup 
Sau khi nhận được yêu cầu backup từ Freezer scheduler, Freezer agent thực hiện backup và sao lưu vào Swift

