#Một số ghi chú khi tìm hiểu code

## 1. Điều chỉnh kích thước từng segment upload lên Swift (Byte), thực hiện trên node client
```
vim /usr/local/lib/python2.7/dist-packages/freezer/openstack/osclients.py

def download_image(self, image):
        """
        ...
        return utils.ReSizeStream(d, image.size, 1000000) #Sua thanh kich thuoc mong muon

 ```

## 2. Điều chỉnh kích thước chunk download từ Swift (Byte), thực hiện trên node client
```
vim /usr/local/lib/python2.7/dist-packages/freezer/openstack/restore.py

def _create_image(self, path, restore_from_timestamp):
        """
        ...

        stream = swift.get_object(self.container, "%s/%s" % (path, backup),
                                  resp_chunk_size=10000000) #Sua thanh kich thuoc mong muon
        ...

```

## 3. Backup metadata sau khi thực hiện backup VM

```
freezer-agent --nova-inst-id cfa52799-3d89-4895-92fc-73d0c39f2907 --path-to-backup /root/adminv3.sh --debug --container vm_cr2 --backup-name vm_long_snap --storage swift --log-file /root/logvmha


Backup metadata received: {"ssh_port": 22, "consistency_checksum": "", "curr_backup_level": 0, "backup_name": "vm_long_snap", "container": "vm_cr2", "compression": "gzip", "dry_run": "", "hostname": "controller", "storage": "swift", "vol_snap_path": "/root/adminv3.sh", "os_auth_version": "", "client_os": "linux2", "time_stamp": 1478349463, "container_segments": "", "ssh_username": "", "path_to_backup": "/root/adminv3.sh", "ssh_key": "", "proxy": "", "always_level": "", "max_level": "", "backup_media": "nova", "ssh_host": "", "mode": "fs", "fs_real_path": "/root/adminv3.sh", "action": "backup", "client_version": "3.0.0", "log_file": "/root/logvmha"}
```