# Kiến trúc, mối tương quan giữa các thành phần trong Freerer
![hinh anh](http://image.prntscr.com/image/34ab0c8458b04a0abcc68d2eea46d7ac.png)

- 1.Người dùng gửi yêu cầu backup 
Người dùng hoặc admin gửi yêu cầu thực hiện backup trên horizon. 

- 2. Xác thực với keystone
Xác thực với Keystone quyền hạn của user

- 3.Gọi Feezer API
Yêu cầu được gửi đến thành phần Freezer API

- 4. Ghi database
Frezzer nhận được yêu cầu, cập nhật trạng thái và ghi vào database Elasticsearch

- 5. Freezer Scheduler nhận jobs
Freezer quản lý job backup bằng gửi lời gọi đến Freezer API để lấy jobs backup.

- 6. Xác thực với Keystone
Freezer API xác thực với Keystone, cấp quyền backend vào Swift cho Freezer Agent

- 7. Feezer agent thực hiện backup 
Sau khi nhận được yêu cầu backup từ Freezer scheduler, Freezer agent thực hiện backup và sao lưu vào Swift
- 8. Backup mount local
Trong trường hợp không sử dụng API, sau khi lên lịch backup, Freezer Schedule gửi yêu cầu backup đên Freezer Agent thực hiện backup. Tiến trình backup thực hiện sao lưu dữ liệu xuống phân vùng mount (có thể là NFS, GlusterFS). Dữ liệu được load lên RAM máy local trước khi được nghi xuống phân vùng mount

- 9. Backup remote FS
Khi thực hiện quá trình backup, ngoài việc sao lưu dữ liệu đến phân vùng mount ta có thể đẩy dữ liệu backup đến một server khác thông qua SSH.

## 1.Freezer Agent backup workflow
![](http://image.prntscr.com/image/cd1712df843246dc8ee2045de612a4e3.png)

 - 1: Freezer Scheduler gửi yêu cầu tạo backup tới Freezer Agent
 - 2: Nếu trước đó đã có backup, Freezer Agent kiểm tra backup metadata từ Object Storage (là vùng lưu trữ các bản backup), Object Storage sẽ xác thực với Keystone trước khi trả backup metadata về Freezer Agent.
 - 3: Freezer Agent tiến hành backup, đẩy lên Object Storage, Object storage xác thực với Keystone, sau khi hoàn tất upload, Object Storage phản hồi lại cho Freezer Agent.
 - 4: Freezer Agent trả lại backup metadata cho Freezer Scheduler, sau đó Freezer Scheduler upload các thông tin về bản backup lên Elastic Search.

## 2.Freezer Scheduler workflow
![](http://image.prntscr.com/image/f1ef98a30d97482690a936b05923e9d3.png)
 - 1: Freezer Scheduler tiến hành xác thực với Keystone (Username và password), nếu đúng Keystone sẽ gửi về 1 token.
 - 2: Freezer Scheduler gửi yêu cầu lấy job list (là  tác vụ backup được đặt lịch) thông qua API cùng với token, sau khi được xác thực, job list sẽ được trả về từ Elastic Search.
 - 3: Khi tiến hành backup, Freezer Scheduler update thông tin job lên Elastic Search qua Freezer API.
 - 4: Freezer Agent tiến hành Backup và trả lại metadata cho Freezer Scheduler.
 - 5: Freezer Scheduler update thông tin job đã hoàn thành lên Elastic Search.
 - 6: Nếu Freezer Scheduler có backup metadata, nó sẽ gửi thông tin về bản backup lên Elastic Search.
