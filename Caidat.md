
# Cài đặt Freezer Agent
## Chuẩn  bị cài đặt
- Freezer Agent bao gồm hai thành phần: Freezer Agent và Freezer Scheduler
- Cài đặt Freezer Agent từ source
- Chọn phiên bản Freezer Agent phù hợp với OpenStack vesion
- Freezer Scheduler stable/Liberty và stable/Kilo chỉ làm việc với Keystone API v2.0

##Yêu cầu 
Freezer Agent yêu cầu các gói cài đặt sau:

- python
- pthon-dev
- GNU Tar >= 1.26
- gzip, bzip2, xz
- OpenSSL
- python-swiftclient
- python-keystoneclient
- pymongo
- PyMySQL
- libmysqlclient-dev
- sync

## Cài đặt trên Ubuntu
- Cài đặt các gói phụ thuộc

```
apt-get install -y python-dev python-pip git openssl gcc  make automake libssl-dev python-git build-essential libffi-dev
```

- Clone phiên bản Freezer Client với git

```
git clone -b [branch] https://github.com/openstack/freezer.git
```
- Tiến hành cài đặt các gói phụ thuộc với pip

```
cd freezer/
```

```
sudo pip install -r requirements.txt
```
- Cài đặt freezer từ source

```
sudo python setup.py install
```
