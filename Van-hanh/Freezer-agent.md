Freezer tự động thêm tiền tố "freezer_" vào container name, nếu không cung cấp containner name thì mặc định sẽ là "freezer__backup".

Thực thi backup có thể thông qua câu lệnh hoặc định nghĩa job trong file cấu hình job tại freezer/freezer/specs/job-backup.conf.example. Câu lệnh sẽ ghi đè lên file cấu hình


# 1. File cấu hình, thư mục

## 1.1 Backup

```
$ sudo freezer-agent --path-to-backup /data/dir/to/backup
--container freezer_new-data-backup --backup-name my-backup-name --storage swift
```

Bởi mặc định --mode fs được thiết lập. Câu lệnh sẽ nén nội dung trong thư mục /data/dir/to/backup, sau đó phân đoạn file và uploaded vào Swift container freezer_new-data-backup với backup name my-backup-name.

Bây giờ ta sẽ kiểm tra backup trong file log tại /root/.freezer/freezer.log


## 1.2 Khôi phục
```
sudo freezer-agent --action restore --restore-abs-path /data/dir/to/backup
--container freezer_new-data-backup--backup-name my-backup-name --storage swift
```
