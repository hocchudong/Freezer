#Một số ghi chú khi tìm hiểu code

## 1. Điều chỉnh kích thước từng segment upload lên Swift (Byte), thực hiện trên node client
```
vim /usr/local/lib/python2.7/dist-packages/freezer/openstack/osclients.py

def download_image(self, image):
        """
        ...
        return utils.ReSizeStream(stream, image.size, 1000000) #Sua thanh kich thuoc mong muon

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

## 4. Fix bug không xóa Snapshot Volume sau khi backup volume Snapshot (sử dụng backend Ceph và đặt `rbd_flatten_volume_from_snapshot = false`)
```
vim /usr/local/lib/python2.7/dist-packages/freezer/openstack/backup.py
def backup_cinder_by_glance(self, volume_id):
    ...
    LOG.debug("Deleting temporary volume")
    cinder.volumes.delete(copied_volume)
    LOG.debug("Deleting temporary snapshot")
    client_manager.clean_snapshot(snapshot)
    LOG.debug("Deleting temporary image")
    client_manager.get_glance().images.delete(image.id)
    ...
```

## 5. Fix bug không xóa VM Snapshot sau khi Backup lên Swift xong 
*Nguyên nhân do sử dụng Ceph làm Backend Glance, Glance api v2 không thể xóa image (Image status owner: None), do đó phải dùng Glance api v1 để xóa*

```
vim /usr/local/lib/python2.7/dist-packages/freezer/openstack/osclients.py

def __init__(self, auth_url, auth_method='password', **kwargs):
    ...
    self.glancev1 = None
    ...

def get_glance_v1(self):
    """
    Get glanceclient instance
    :return: glanceclient instance
    """
    if not self.glancev1:
        self.glancev1 = self.create_glance_v1()
    return self.glancev1
    ...

def create_glance_v1(self):
    """
    Use pre-initialized session to create an instance of glance client.
    :return: glanceclient instance
    """
    if 'endpoint_type' in self.client_kwargs.keys():
        self.client_kwargs.pop('endpoint_type')
    if 'insecure' in self.client_kwargs.keys():
        self.client_kwargs.pop('insecure')
    self.glancev1 = glance_client('1', session=self.sess,
                                **self.client_kwargs)
    return self.glancev1

```

```
vim /usr/local/lib/python2.7/dist-packages/freezer/openstack/backup.py

def backup_nova(self, instance_id):
    ...
    glancev1 = client_manager.get_glance_v1()
    glancev1.images.delete(image.id)
    ...
def backup_cinder_by_glance(self, volume_id):
    ...
    client_manager.get_glance_v1().images.delete(image.id)
    ...
``` 

## 6. Fix bug không xóa Image sau khi restore volume 
*Chú ý chỉ thực hiện được khi Cinder và Glance không cùng Ceph backend*

```
vim /usr/local/lib/python2.7/dist-packages/freezer/openstack/restore.py
    def restore_cinder_by_glance(self, volume_id, restore_from_timestamp):
        ...
        client_manager = self.client_manager
        cinder = client_manager.get_cinder()
        volume_id_raw = client_manager.get_cinder().volumes.create(size,
                                                        imageRef=image.id)
        volume_id_str = str(volume_id_raw)
        volume_id = volume_id_str.split(':')[1].strip(' ').strip('>')
        LOG.info(volume_id)
        volume = cinder.volumes.get(volume_id)
        while volume.status != 'available':
            time.sleep(5)
            try:
                volume = cinder.volumes.get(volume_id)
            except Exception as e:
                LOG.error(e)

        self.client_manager.get_glance_v1().images.delete(image) #Nếu sử dụng Ceph backend phải dùng glance api v1
        ...
    def restore_nova(self, instance_id, restore_from_timestamp,
                     nova_network=None):
        glancev1 = self.client_manager.create_glance_v1()
        glancev1.images.delete(image.id)
        ...

```

## 7. Fix bug không xóa Image sau khi restore VM
*Chú ý chỉ thực hiện được khi Nova và Glance không cùng Ceph backend*
```
vim /usr/local/lib/python2.7/dist-packages/freezer/openstack/restore.py

    def restore_nova(self, instance_id, restore_from_timestamp,
                     nova_network=None):
        ...
        glancev1 = self.client_manager.create_glance_v1()
        glancev1.images.delete(image.id)
        ...

```
## 7. Fix bug không restore from date

Hiện tại ở bản master tính đến ngày 08/11/2016 khi thực hiện restore theo thời gian ta gặp phải lỗi này

![](http://image.prntscr.com/image/9d52ac1c998946a6b1ff8e8f4e48b55f.png)

###**CODE**:
```
vim /usr/local/lib/python2.7/dist-packages/freezer/storage/base.py
```

![](http://image.prntscr.com/image/d47c9baf36654d4aae4b6a4cdd4ba79d.png)
```
vim /usr/local/lib/python2.7/dist-packages/freezer/storage/physical.py
```

![](http://image.prntscr.com/image/fddf337bceaf4cfbb2a4113206b76c36.png)

###**FIX BUG**:

```
vim /usr/local/lib/python2.7/dist-packages/freezer/job.py
```

![](http://image.prntscr.com/image/4830ae120baf497e9ce5ce97345d5117.png)

```
vim /usr/local/lib/python2.7/dist-packages/freezer/storage/base.py
```

![](http://image.prntscr.com/image/ebab93eff6c34fd8914b74960fb41261.png)
```
vim /usr/local/lib/python2.7/dist-packages/freezer/storage/physical.py
```
![](http://image.prntscr.com/image/55e86bbf2fa64778825b6cfd639294b6.png)


## 8. Tổ chức các file trong thư mục backup
![](http://image.prntscr.com/image/e46faeafd28041e2bd28abbb0872b5fb.png)
Một thư mục backup bao gồm các thư mục con
 - Data: chứa file backup (full và incremental)
 - Metadata: chứa metadata của bản backup, gồm các thông tin về công cụ mã hóa, nén và lưu trữ
    Một metadata file như sau:
   `{"encryption": false, "compression": "gzip", "engine_name": "tar"}`
 
 
VD về một thư mục backup:
Trong đó:
 - `zabbix_test`: tên của bản backup được khai báo khi thực hiện backup
 - `1478581647`: Linux epoch time tại thời điểm bản backup được khởi tạo full
 - `0_1478581647`: bản backup đầu tiên từ lúc backup full tại thời điểm `1478581647`

Khôi phục lại một bản backup
`gzip -d < file.data | tar xvf - `


## 9. Fix bug không restore được qua SSH
*Do SFTP đã bị đóng phiên, cần mở phiên mới để lấy metadata từ remote Server về*
```
vim /usr/local/lib/python2.7/dist-packages/freezer/storage/ssh.py
    ...
    def get_file(self, from_path, to_path):
    ssh = paramiko.SSHClient()
    ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())

    ssh.connect(self.remote_ip, username=self.remote_username,
                key_filename=self.ssh_key_path, port=self.port)
    self.ssh = ssh
    self.ftp = self.ssh.open_sftp()
    self.ftp.get(from_path, to_path)
    ...
```

## 10. Fix bug khi Backup qua SSH phải tạo thư mục với đường là hostname và tên bản backup trước trên Remote Host (VD: `/root/metadata/tar/zabbix_long_ssh`)
*Thiếu đoạn code khởi tạo path*
```
vim /usr/local/lib/python2.7/dist-packages/freezer/storage/physical.py
    def get_level_zero(self,
                       engine,
                       hostname_backup_name,
                       recent_to_date=None):
        path = self.metadata_path(
            engine=engine,
            hostname_backup_name=hostname_backup_name)
        try:
            self.create_dirs(path)
        except:
            pass
```

## 11. Freezer sử dụng thư viện paramiko để giao tiếp SFTP với Remote FS, có thể sử dụng thư viện này thông qua ví dụ sau:
    import paramiko
    hostname = '172.16.69.179'
    port = 22
    username = 'root'
    password = 'a'
    t = paramiko.Transport((hostname, port))
    t.connect(username=username, password=password)
    sftp = paramiko.SFTPClient.from_transport(t)
    sftp.mkdir(path)