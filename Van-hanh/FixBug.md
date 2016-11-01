# 1. Bug Keystone Version

Ở bản stable/mitaka Freezer agent không xác thực được với Keystone V3 do file openstack.py thiếu thông tin user_domain_id và project_domain_id. Hiện tại ở bản master đã code đã fig bug này

![](http://image.prntscr.com/image/d9a62cf43b9446298564a028be099e57.png)

Nội dung file openstack.py:


![](http://image.prntscr.com/image/167d4ce50e66400b908121ba58114103.png)


Fix: thêm thuộc tính  user_domain_id và project_domain_id:

![](http://image.prntscr.com/image/808d252cd6d8437e819c3e52e09050c3.png)


![](http://image.prntscr.com/image/92ac125300794678b7ebd4aab200718e.png)


![](http://image.prntscr.com/image/9ec0ec1100334bf7b05a6b6514a82f64.png)


code

```
"""
(c) Copyright 2015,2016 Hewlett-Packard Development Company, L.P.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
"""
import os


class OpenstackOptions:
    """
    Stores credentials for OpenStack API.
    Can be created using
    >> create_from_env()
    or
    >> create_from_dict(dict)
    """
    def __init__(self, user_name, tenant_name, project_name, auth_url,
                 password, identity_api_version, tenant_id=None,
                 region_name=None, endpoint_type=None, cert=None,
                 insecure=False, verify=True, project_domain_name= None, user_domain_name = None ):
        self.user_name = user_name
        self.tenant_name = tenant_name
        self.auth_url = auth_url
        self.password = password
        self.tenant_id = tenant_id
        self.project_name = project_name
        self.identity_api_version = identity_api_version
        self.region_name = region_name
        self.endpoint_type = endpoint_type
        self.cert = cert
        self.insecure = insecure
        self.verify = verify
        self.project_domain_name = project_domain_name
        self.user_domain_name = user_domain_name
        if not (self.password and self.user_name and self.auth_url and
           (self.tenant_name or self.project_name)):
            raise Exception("Please set up in your env:"
                            "OS_USERNAME, OS_TENANT_NAME/OS_PROJECT_NAME,"
                            " OS_AUTH_URL, OS_PASSWORD")

    @property
    def os_options(self):
        """
        :return: The OpenStack options which can have tenant_id,
                 auth_token, service_type, endpoint_type, tenant_name,
                 object_storage_url, region_name
        """
        return {'tenant_id': self.tenant_id,
                'tenant_name': self.tenant_name,
                'project_name': self.project_name,
                'identity_api_version': self.identity_api_version,
                'region_name': self.region_name,
                'endpoint_type': self.endpoint_type,
                'project_domain_name': self.project_domain_name,
                'user_domain_name': self.user_domain_name}

    @staticmethod
    def create_from_env():
        return OpenstackOptions.create_from_dict(os.environ)

    @staticmethod
    def create_from_dict(src_dict):
        return OpenstackOptions(
            user_name=src_dict.get('OS_USERNAME', None),
            tenant_name=src_dict.get('OS_TENANT_NAME', None),
            project_name=src_dict.get('OS_PROJECT_NAME', None),
            auth_url=src_dict.get('OS_AUTH_URL', None),
            identity_api_version=src_dict.get('OS_IDENTITY_API_VERSION',
                                              '2.0'),
            password=src_dict.get('OS_PASSWORD', None),
            tenant_id=src_dict.get('OS_TENANT_ID', None),
            region_name=src_dict.get('OS_REGION_NAME', None),
            endpoint_type=src_dict.get('OS_ENDPOINT_TYPE', None),
            cert=src_dict.get('OS_CERT', None),
            insecure=False,
            verify=True,
            project_domain_name=src_dict.get('OS_PROJECT_DOMAIN_NAME', None),
            user_domain_name=src_dict.get('OS_USER_DOMAIN_NAME', None)
        )

```

# 2. Bug Nova Backup

Khi backup VM thì đầu tiên sẽ tạo  bản snapshot của VM, upload image vào Swift và sau đó xóa image đi. Khi freezer tiến hành backup thì gặp bug MemoryError do quá trình dowload image những image dung lượng lớn vào RAM rồi đẩy vào swift. Hiện tại tính đến ngày 27/10/2016 ở cả branch master và stable/mitaka đều dính bug này 

![](http://image.prntscr.com/image/270b0d48e16a45b1a257283df3511ee0.png)


Fix: trong file tại đường dẫn: /usr/lib/python2.7/dist-packages/keystoneauth1/session.py đưa dòng lệnh

```
text = self._remove_service_catalog(response.text)
```
vào try catch

```
try:
    text = self._remove_service_catalog(response.text)
except MemoryError:
    text = '{body is to huge to show, sorry}'
```
![](http://image.prntscr.com/image/6d0cf5e58c23431e981a46494023cf4f.png)

Quá trình này sẽ bỏ qua quá trình dowload image vào RAM do tràn bộ nhớ và đẩy trực tiếp xuống Swift.

