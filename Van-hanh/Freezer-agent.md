Freezer tự động thêm tiền tố "freezer_" vào container name, nếu không cung cấp containner name thì mặc định sẽ là "freezer__backup".

Thực thi backup có thể thông qua câu lệnh hoặc định nghĩa job trong file cấu hình job tại freezer/freezer/specs/job-backup.conf.example. Câu lệnh sẽ ghi đè lên file cấu hình


# 1. File cấu hình, thư mục

## 1.1 Backup

```
$sudo freezer-agent --path-to-backup /data/dir/to/backup
--container freezer_new-data-backup --backup-name my-backup-name --storage swift
```

Bởi mặc định --mode fs được thiết lập. Câu lệnh sẽ nén nội dung trong thư mục /data/dir/to/backup, sau đó phân đoạn file và uploaded vào Swift container freezer_new-data-backup với backup name my-backup-name.

Bây giờ ta sẽ kiểm tra backup trong file log tại /root/.freezer/freezer.log


## 1.2 Khôi phục

```
sudo freezer-agent --action restore --restore-abs-path /data/dir/to/backup
--container freezer_new-data-backup--backup-name my-backup-name --storage swift
```

# 2. LVM

## 2.1 Backup
Backup logical volume sử dụng cơ chế lvm snapshot

Giả sử ta có volumgroup là havg, logical volume là halv. thư mục mount logical volume này là /mnt/halvm 

```
freezerc --lvm-srcvol /dev/havg/halvm --lvm-dirmount /mnt/ha-backup --lvm-volgroup havg --path-to-backup /mnt/halvm/ --container ha_lvm_container --mode fs --backup-name hatest
```
## 2.2 Khôi phục

```
freezer-agent --action restore --container ha_lvm_container \
--backup-name hatest --hostname cong-swift1 \
--restore-abs-path /mnt/ha-backup --storage swift
```

# 3. Mysql

## 3.1 Backup

Backup mysql sử dụng cơ chế snapshot của LVM. Ta sẽ backup thư mục của mysql.
Ví dụ: Thư mục mysql_dir là /mnt/mysqldir được mount với logical volume /dev/havg2/halv2 và file cấu hình /root/.freezer/db.conf

```
$cat /root/.freezer/db.conf

host = your.mysql.host.ip

user = backup

password = userpassword
```

Tiến hành backup

```
freezer-agent --lvm-srcvol /dev/havg2/halv2 \

--lvm-dirmount /mnt/mysql-snapshot --lvm-volgroup havg2 \

--path-to-backup /mnt/mysqldir \

--mysql-conf /root/.freezer/db.conf \

--container mysql-backup-container \

--mode mysql --backup-name mysql-ops002

```

## 3.2 Restore

```
freezer-agent --action restore --container mysql-backup-container \

--backup-name mysql-ops002 --hostname cong-swift1 \

--restore-abs-path /mnt/mysqldir --restore-from-date "2016-09-20T09:39:16" --overwrite --storage swift

```
