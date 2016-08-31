
Freezer là một Backup Restore DR as a Service giúp sao lưu và khôi phục tự động

Tính năng của Freezer:

- Sao lưu Filesytem sử dụng snapshot
- Hỗ trợ mã hóa dữ liệu backup mạnh: AES-256-CFB
- Sao lưu Filesytem tree trực tiếp không cần thực hiện volume snapshot
- Backup Journal MongoDB tree sử dụng lvm snapshot. Đẩy dữ liệu backup vào Swift
- Sao lưu MySQL DB sử dụng lvm snapshot
- Khôi phục dữ liệu vào filesytem tự động theo thời gian được chỉ định trước
- Tiêu tốn dung lượng lưu trữ thấp
- Chính sách backup linh hoạt (backup incremental hay backup differential)
- Dữ liệu backup được lưu dưới định dạng tar, hỗ trợ backup incremental
- Hỗ trợ nhiều thuật toán nén dữ liệu (zlib, bzip2, xz)
- Xóa tự động các file backup cũ
- Hỗ trợ lưu trữ backup xuống nhiều loại backend như Swift, local file system, remote server thông qua ssh)
- Hỗ trợ nhiều nền tảng (Linux, Windows, BSD, OSX)
- Quản lý nhiều jobs (chạy nhiều backup trên cùng một node)
- Đồng bộ dữ liệu backup và khôi phục trên nhiều node
- Quản lý trên Openstack Horizon
- Có thể chạy script, command trước khi hoặc sau tiến trình backup
