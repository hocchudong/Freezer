# A.Môi trường cài đặt
 - OS: Ubuntu 14.04.5 LTS 
 - Kernel: 3.16.0-77-generic
 - Platform: OpenStack Mitaka

# B. Cài đặt
## 1. Tải source code của freezer-web-ui, sử dụng bản master
```
git clone https://github.com/openstack/freezer-web-ui.git
cd freezer-web-ui
git checkout stable/mitaka
```

## 2. Lỗi không hiển thị các bản backup
### 2.1. Sửa file `/root/freezer-web-ui/disaster_recovery/backups/views.py`

```
class IndexView(tables.DataTableView):
    name = _("Backups")
    slug = "backups"
    table_class = freezer_tables.BackupsTable
    template_name = "disaster_recovery/backups/index.html"

    @shield('Unable to retrieve backups.', redirect='backups:index')
    def get_data(self):
        filters = self.table.get_filter_string() or {}
        return freezer_api.Backup(self.request).list(search=filters)
```



### 2.2. Sửa file `/root/freezer-web-ui/disaster_recovery/api/api.py`

```
class Backup(object):

    def __init__(self, request):
        self.request = request
        self.client = client(request)

    def list(self, json=False, limit=500, offset=0, search=None):
        if search:
            search = {"match": [{"_all": search}, ], }

        backups = self.client.backups.list_all(limit=limit,
                                           offset=offset,
                                           search=search)

```

## 3. Tiến hành cài đặt freezer-web-ui
```
python setup.py install
cp freezer-web-ui/disaster_recovery/enabled/_5050_freezer.py  /usr/share/openstack-dashboard/openstack_dashboard/enabled/_5050_freezer.py
```

## 4. Lỗi không load được freezer UI 
### 4.1. Sửa file `/usr/share/openstack-dashboard/openstack_dashboard/local/local_settings.py`, thêm
```
FREEZER_API_URL = 'http://10.10.10.160:9090' #IP của node Freezer-api
```

### 4.2. Sửa file `/usr/share/openstack-dashboard/static/freezer/js/freezer.actions.action.js`, bổ xung vào cuổi 
```
$(function () {
    hideEverything();
    setActionOptions();
    setStorageOptions();
    setModeOptions();
});
```

## 5. Khởi động lại apache2 và memcached
```
service apache2 restart
service memcache restart
```

## 6. Giao diện quản lý của Freezer trên Dashboard
![](http://image.prntscr.com/image/0df9e3d29892491e86830c5e9192c9d8.png)