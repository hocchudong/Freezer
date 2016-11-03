Freezer tự động thêm tiền tố "freezer_" vào container name, nếu không cung cấp containner name thì mặc định sẽ là "freezer__backup".

Thực thi backup có thể thông qua câu lệnh hoặc định nghĩa job trong file cấu hình job tại `freezer/freezer/specs/job-backup.conf.example`. Câu lệnh sẽ ghi đè lên file cấu hình


# 1. File cấu hình, thư mục

## 1.1 Backup

### 1.1.1 Lưu trữ Swift
```
$sudo freezer-agent --path-to-backup /data/dir/to/backup
--container freezer_new-data-backup --backup-name my-backup-name --storage swift
```

Mặc định `--mode fs` được thiết lập. Câu lệnh sẽ nén nội dung trong thư mục `/data/dir/to/backup`, sau đó phân đoạn file và uploaded vào Swift container `freezer_new-data-backup` với backup name `my-backup-name`.

Bây giờ ta sẽ kiểm tra backup trong file log tại `/root/.freezer/freezer.log`

### 1.1.2 Lưu trữ SSH server 

```
$ sudo freezer-agent --path-to-backup /data/dir/to/backup
--container /remote-machine-path/ --backup-name my-backup-name
--storage ssh --ssh-username ubuntu --ssh-key ~/.ssh/id_rsa
--ssh-host 8.8.8.8
```

Chú ý: ở bản master ngày 03/11 đang có bug. Một là sửa trực tiếp vào code, hai là làm như sau

Giải sử ta cần đẩy file adminv3.sh từ zabbix host sang SSH server địa chỉ 172.16.69.177

```
root@zabbix:~# freezer-agent --path-to-backup /root/adminv3.sh --container /root/freezerssh --backup-name ha-bk2609 --storage ssh --ssh-username root --ssh-key /root/.ssh/id_rsa --ssh-host 172.16.69.177 --log-file=/root/logfile.log
```
ta sẽ nhận được log

```
Critical Error: [Errno 2] No such file
```

Trên server 172.16.69.177 tạo thư mục

```
 mkdir -p /root/freezerssh/metadata/tar/zabbix_ha-bk2609
```

zabbix_ha-bk2609 là hostname_backupname

## 1.2 Khôi phục

```
sudo freezer-agent --action restore --restore-abs-path /data/dir/to/backup
--container freezer_new-data-backup--backup-name my-backup-name --storage swift
```

# 2. LVM

## 2.1 Backup
Backup logical volume sử dụng cơ chế lvm snapshot

Giả sử ta có volume group là `havg`, logical volume là `halv`. thư mục mount logical volume này là `/mnt/halvm `

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

Backup mysql sử dụng cơ chế snapshot của LVM, cần phải cung cấp các thông tin về MySQL host, tài khoản có quyền lock DB, Freezer-agent sẽ tiến hành lock DB MySQL trước khi snapshot LV, ở đây các thông tin được cung cấp qua file `/root/.freezer/db.conf`

Ví dụ: Thư mục mysql_dir là /mnt/mysqldir được mount với logical volume `/dev/havg2/halv2` và file cấu hình `/root/.freezer/db.conf`

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

# 4. Nova

## 4.1 Backup
```
freezer-agent --nova-inst-id af700fe8-0235-4a1f-b6ee-335589cf4d4d  --debug --container vm_cr --backup-name vm_long_snap --storage swift --log-file /root/logvmha
```

## 4.2 Restore
```
freezer-agent --action restore --container vm_cr --nova-inst-id af700fe8-0235-4a1f-b6ee-335589cf4d4d -nova-restore-network d9bb6580-393d-4a87-b73b-d2efe6255125   --restore-from-date "2016-09-20T09:39:16" --log-file /root/nova-restore2.txt
```

# 5. Cinder

## 5.1 Backup Snapshot

```
freezer-agent --cinder-vol-id d3537664-84b2-457a-a13c-66efd2e4e1d1 --path-to-backup /root/admin-openrc --container volume_con2  --storage swift --log-file /root/logvolume_longlq
```

## 5.2 Restore Snapshot
```
freezer-agent --action restore --cinder-vol-id d3537664-84b2-457a-a13c-66efd2e4e1d1 --container volume_con --storage swift --log-file /root/logvolume_longlq
```



## 5.3 Backup using cinder-backup
### 5.3.1 Backup full

```
freezer-agent --cindernative-vol-id d3537664-84b2-457a-a13c-66efd2e4e1d1 --path-to-backup /root/admin-openrc  --log-file /root/logvolume_longlq
```

### 5.3.2 Backup Incremental

```
freezer-agent --cindernative-vol-id d3537664-84b2-457a-a13c-66efd2e4e1d1 --incremental  --path-to-backup /root/admin-openrc  --log-file /root/logvolume_longlq
```

## 5.4 Restore using cinder-backup

```
freezer-agent --action restore --cindernative-vol-id d3537664-84b2-457a-a13c-66efd2e4e1d1 --log-file /root/logvolume_longlq
```
