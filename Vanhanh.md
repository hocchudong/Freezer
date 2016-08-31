

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
