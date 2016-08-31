

# Các thành phần trong Project Freezer

## 1. Freezer WebUI
 Giao diện web tương tác với Freezer API để thực hiện việc config và thay đổi thông số. Nó cung cấp hầu hết các tính năng tương tự như Freezer Agent CLI, các tùy chọn đặt lịch nâng cao như đồng bộ việc backup giữa nhiều node, đặt metric, báo cáo.

## 2. Freezer Scheduler
 Thành phần được cài đặt phía client, chạy trên các node cần thực hiện backup. Bao gồm một daemon nhận dữ liệu từ freezer API và thực thi các jobs (VD như backup, restore, quản trị, lấy thông tin, thực thi các script trước và sau backup) bằng cách chạy Freezer Agent. Các metrics và exit codes được trả về bởi Freezer Agent được bắt lại và trả về Freezer API. Freezer scheduler quản lý các tiến trình và đồng bộ nhiều tác bụ thực hiện trên một node hay nhiều node. Trạng thái và kết quả của tác vụ được trả về qua API. Freezer Scheduler thực hiện việc upload thông tin các jobs vào API bằng cách đọc các file trên file system. Có thể cấu hình job session hoặc các thông số khác như Freezer API interval polling. 

## 3. Freezer Agent
 Được viết bằng Python, chạy phía client, thực hiện việc backup dữ liệu. Nó có thể chạy độc lập hoặc được điều khiển bởi Freezer Scheduler. Freezer Agent linh hoạt trong việc thực thi backup, restore hay các tác vụ khác trong hệ thống đang chạy. Để đảm bảo tính nhất quán của dữ liệu, tốc độ, hiệu năng, tài nguyên sử dụng,... Freezer Agent cung cấp các lựa chọn sau để thực hiện các bản backup tối ưu nhât:
 - Segment size: kích thước bộ nhớ sử dụng.
 - Queue size: tối ưu việc backup khi I/O, bandwidth, RAM hay CPU bị giới hạn.
 - I/O Affinity và Process Priority : sử dụng với I/O thời gian thực và ưu tiên các process người dùng.
 - Giới hạn băng thông
 - Mà hóa phía người dùng (AES-256-CFB)
 - Nén : sử dụng nhiều thuật toán như zlib, bzip2, xz/lzma
 - Upload song song tới hệ thống lưu trữ : backup vào swift , remote node bằng SSH, ...
 - Thực thi việc backup incremental theo dạng file (tar), dạng block (như rsync) và backup differential.
 - Chạy trên nhiều nền tảng: Linux, Windows, BSD, OSX.
 - Tự động xóa các bản backup cũ.

## 4. Freezer API
 API được dùng để lưu trữ và cung cấp metadata cho Freezer Web UI và Freezer Scheduler. API cũng được dùng để lưu các thông tin session cho việc đồng bộ backup nhiều node. Không có dữ liệu tải thực nào được lưu trong API.

## 5. DB Elasticsearch
 Backend cho API để lưu trữ và nhận metric, thông tin metadata session, tình trạng job,...

Nguồn: 

https://github.com/openstack/freezer

https://wiki.openstack.org/wiki/Freezer
