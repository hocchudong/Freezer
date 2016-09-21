# A.Môi trường cài đặt
 - OS: Ubuntu 14.04.5 LTS 
 - Kernel: 3.16.0-77-generic
 - Platform: OpenStack Mitaka

# B. Cài đặt
## 1. Tải source code của freezer-web-ui, sử dụng bản stable/mitaka
```
git clone https://github.com/openstack/freezer-web-ui
cd freezer-web-ui
git checkout stable/mitaka
```

## 2. Cài đặt freezer-web-ui
```
python setup.py install
cp freezer-web-ui/disaster_recovery/enabled/_5050_freezer.py  /usr/share/openstack-dashboard/openstack_dashboard/enabled/_5050_freezer.py
```

## 3. Sửa file '/usr/share/openstack-dashboard/openstack_dashboard/local/local_settings.py', thêm
```
FREEZER_API_URL = 'http://10.10.10.160:9090' #IP của node Freezer-api
```

## 4. Khởi động lại apache2 và memcached
```
service apache2 restart
service memcache restart
```

## 5. Giao diện quản lý của Freezer trên Dashboard
![](http://image.prntscr.com/image/0df9e3d29892491e86830c5e9192c9d8.png)