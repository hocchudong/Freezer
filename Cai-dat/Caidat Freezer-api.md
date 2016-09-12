# A.Môi trường cài đặt
2 Server: OpenStack Controller, Freezer-api
OS: Ubuntu 14.04.5 LTS 
Kernel: 3.16.0-77-generic
Platform: OpenStack Mitaka

# B. Cài đặt
## 1.Cài đặt Freezer (thực hiện trên node Freezer)
### 1.1.Cài đặt git
```
apt-get install git -y
apt-get install python-pip -y
```
### 1.2. Cài đặt các thư viện cần thiết
```
pip install wrapt
pip install markupsafe
```
### 1.3. Cài đặt freezer-api
```
git clone https://git.openstack.org/openstack/freezer-api.git
cd freezer-api
git checkout master
pip install ./
```

### 1.4. Copy file cấu hình vào thư mục `/etc/freezer`
cp etc/freezer/freezer-api.conf.sample /etc/freezer/freezer-api.conf
cp etc/freezer/freezer-paste.ini /etc/freezer/freezer-paste.ini
cp etc/freezer/policy.json /etc/freezer/policy.json

### 1.5. Chỉnh sửa file cấu hình
Cấu hình cho keystone và elastic search
```
...
[keystone_authtoken]
auth_uri=http://10.10.10.157:5000/v3 
identity_uri=http://10.10.10.157:35357	
auth_version = v3 
admin_tenant_name = service
admin_user = freezer
admin_password = Welcome123

[storage]
hosts='http://localhost:9200' #khai báo IP của node ElasticSearch
number_of_replicas=0 #Số bản sao của DB.

```

### 1.6. Tạo service quản lý bởi init cho freezer-api, `vim /etc/init/freezer-api.conf`
```
description "Freezer AIP Service"

# Service level
start on runlevel [2345]

# When to stop the service
stop on runlevel [016]

# Automatically restart process if crashed
respawn


# Specify working directory
chdir /usr/local/bin/

# Specify the process/command to start, e.g.
exec freezer-api

```

## 2. Cài đặt ElasticSearch, có thể tách ra thành 1 node riêng, ở đây mình cài chung trên node Freezer.
### 2.1. Thêm Java PPA vào apt
```
add-apt-repository -y ppa:webupd8team/java
apt-get update
```

### 2.2. Cài đặt Java 8
```
apt-get -y install oracle-java8-installer
```
### 2.3. Import ElasticSearch public GPG key vào apt
```
wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```

### 2.4. Tạo ElasticSearch source list
```
echo "deb http://packages.elastic.co/elasticsearch/2.x/debian stable main" | sudo tee -a /etc/apt/sources.list.d/elasticsearch-2.x.list
apt-get update
```

### 2.5 Cài đặt ElasticSearch
```
apt-get -y install elasticsearch
```

### 2.5 Chỉnh sửa file cấu hình `/etc/elasticsearch/elasticsearch.yml`
```
network.host: localhost #Có thể sửa thành 0.0.0.0 nếu muốn tất cả các Server ngoài truy cập vào
```

### 2.6. Khởi động lại ElasticSearch
```
service elasticsearch restart
```

## 3. Cấu hình Keystone (thực hiện trên node Keystone)
### 3.1. Tạo Keystone endpoint v2.0
```
openstack endpoint create --region RegionOne identity public http://10.10.10.157:5000/v2.0

openstack endpoint create --region RegionOne identity internal http://10.10.10.157:5000/v2.0

openstack endpoint create --region RegionOne identity admin http://10.10.10.157:35357/v2.0
```

### 3.2. Tạo user, service và endpoint cho Freezer
```
openstack user create freezer --domain default --password Welcome123
openstack role add --project service --user freezer admin
openstack service create --name freezer --description "Freezer Backup Service" backup
openstack endpoint create --region RegionOne backup public  http://10.10.10.160:9090 #IP của node Freezer
openstack endpoint create --region RegionOne backup admin http://10.10.10.160:9090 #IP của node Freezer
openstack endpoint create --region RegionOne backup internal http://10.10.10.160:9090 #IP của node Freezer
```

### 3.3. Tạo DB cho freezer
```
freezer-manage db sync
```

### 3.4. Khởi động lại freezer-api
```
service freezer-api restart
```

### 3.5 Test freezer-api
```

```
