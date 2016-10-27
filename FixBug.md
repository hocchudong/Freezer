# 1. Bug Keystone Version

Ở bản stable/mitaka Freezer agent không xác thực được với Keystone V3 do file openstack.py thiếu thông tin user_domain_id và project_domain_id

![](http://image.prntscr.com/image/d9a62cf43b9446298564a028be099e57.png)

Nội dung file openstack.py:


![]http://image.prntscr.com/image/167d4ce50e66400b908121ba58114103.png)


Fix: thêm thuộc tính  user_domain_id và project_domain_id:

![]http://image.prntscr.com/image/808d252cd6d8437e819c3e52e09050c3.png)


![]http://image.prntscr.com/image/92ac125300794678b7ebd4aab200718e.png)


![]http://image.prntscr.com/image/9ec0ec1100334bf7b05a6b6514a82f64.png)


code

```
fdf
```

#. Bug Nova Backup
