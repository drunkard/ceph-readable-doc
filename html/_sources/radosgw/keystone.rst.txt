============================
 与 OpenStack Keystone 对接
============================
.. Integrating with OpenStack Keystone

Ceph 对象网关可以与 Keystone 对接，它是 OpenStack 的鉴权服务。\
这需要让网关把 Keystone 当作用户认证机构，经过 Keystone 授权、\
允许访问网关的用户， Ceph 对象网关内也会自动创建此用户（如果\
此前还没有）。 Keystone 认定有效的令牌，网关也会认为有效。

与 Keystone 对接相关的网关配置选项有： ::

	[client.radosgw.gateway]
	rgw keystone api version = {keystone api version}
	rgw keystone url = {keystone server url:keystone server admin port}
	rgw keystone admin token = {keystone admin token}
	rgw keystone admin token path = {path to keystone admin token} #preferred
	rgw keystone accepted roles = {accepted user roles}
	rgw keystone token cache size = {number of tokens to cache}
	rgw keystone implicit tenants = {true for private tenant for each new user}

也能配置 Keystone 服务的租户、用户名、密码（适用于 v2.0 版的
OpenStack Identity API ），与 OpenStack 服务的配置过程相似，这\
样可避免在配置文件中设置共享密钥 ``rgw keystone admin token`` ，\
因为这在生产环境下是不推进的配置方法。此处，服务的租户凭证应该\
有管理员权限，详情见 `Openstack Keystone 文档`_\ ，里面详细解\
释了机制。必需的配置选项有： ::

   rgw keystone admin user = {keystone service tenant user name}
   rgw keystone admin password = {keystone service tenant user password}
   rgw keystone admin password = {keystone service tenant user password path} # preferred
   rgw keystone admin tenant = {keystone service tenant name}

Ceph 对象网关的用户被映射为 Keystone 的 ``tenant`` 。 Keystone
用户具有不同的角色，角色可能对应着不止一个租户。 Ceph 拿到票据\
后，它会检查其租户、以及给此票据分配的用户角色，然后根据配置的
``rgw keystone accepted roles`` 决定接受、或拒绝此请求。

对于 v3 版本的 Openstack Identity API ，需要把
``rgw keystone admin tenant`` 换成： ::

        rgw keystone admin domain = {keystone admin domain name}
        rgw keystone admin project = {keystone admin project name}

For compatibility with previous versions of ceph, it is also
possible to set ``rgw keystone implicit tenants`` to either
``s3`` or ``swift``.  This has the effect of splitting
the identity space such that the indicated protocol will
only use implicit tenants, and the other protocol will
never use implicit tenants.  Some older versions of ceph
only supported implicit tenants with swift.


Ocata （及后续版本）
--------------------
.. Ocata (and later)

Keystone 自身作为对象存储服务的入口（ endpoint ），需要配置为\
指向 Ceph 对象网关。 ::

  openstack service create --name=swift \
                           --description="Swift Service" \
                           object-store
  +-------------+----------------------------------+
  | Field       | Value                            |
  +-------------+----------------------------------+
  | description | Swift Service                    |
  | enabled     | True                             |
  | id          | 37c4c0e79571404cb4644201a4a6e5ee |
  | name        | swift                            |
  | type        | object-store                     |
  +-------------+----------------------------------+

  openstack endpoint create --region RegionOne \
       --publicurl   "http://radosgw.example.com:8080/swift/v1" \
       --adminurl    "http://radosgw.example.com:8080/swift/v1" \
       --internalurl "http://radosgw.example.com:8080/swift/v1" \
       swift
  +--------------+------------------------------------------+
  | Field        | Value                                    |
  +--------------+------------------------------------------+
  | adminurl     | http://radosgw.example.com:8080/swift/v1 |
  | id           | e4249d2b60e44743a67b5e5b38c18dd3         |
  | internalurl  | http://radosgw.example.com:8080/swift/v1 |
  | publicurl    | http://radosgw.example.com:8080/swift/v1 |
  | region       | RegionOne                                |
  | service_id   | 37c4c0e79571404cb4644201a4a6e5ee         |
  | service_name | swift                                    |
  | service_type | object-store                             |
  +--------------+------------------------------------------+

  $ openstack endpoint show object-store
  +--------------+------------------------------------------+
  | Field        | Value                                    |
  +--------------+------------------------------------------+
  | adminurl     | http://radosgw.example.com:8080/swift/v1 |
  | enabled      | True                                     |
  | id           | e4249d2b60e44743a67b5e5b38c18dd3         |
  | internalurl  | http://radosgw.example.com:8080/swift/v1 |
  | publicurl    | http://radosgw.example.com:8080/swift/v1 |
  | region       | RegionOne                                |
  | service_id   | 37c4c0e79571404cb4644201a4a6e5ee         |
  | service_name | swift                                    |
  | service_type | object-store                             |
  +--------------+------------------------------------------+

.. note:: If your radosgw ``ceph.conf`` sets the configuration option
	  ``rgw swift account in url = true``, your ``object-store``
	  endpoint URLs must be set to include the suffix
	  ``/v1/AUTH_%(tenant_id)s`` (instead of just ``/v1``).

The Keystone URL is the Keystone admin RESTful API URL. The admin token is the
token that is configured internally in Keystone for admin requests.

OpenStack 的 Keystone 组件也可以用自签名的 SSL 证书来终结，\
要使 radosgw 有能力与这样的 Keystone 交互，你可以在运行
radosgw 的节点上安装 Keystone 的 SSL 证书；另外， radosgw
也可以配置为根本不校验 SSL 证书（类似加了 ``--insecure``
开关的 Openstack 客户端请求），即把
``rgw keystone verify ssl`` 配置为 ``false`` 。


.. _Openstack Keystone 文档: http://docs.openstack.org/developer/keystone/configuringservices.html#setting-up-projects-users-and-roles


跨 Project(Tenant) 访问
-----------------------
.. Cross Project(Tenant) Access

In order to let a project (earlier called a 'tenant') access buckets belonging to a different project, the following config option needs to be enabled::

   rgw swift account in url = true

The Keystone object-store endpoint must accordingly be configured to include the AUTH_%(project_id)s suffix::

   openstack endpoint create --region RegionOne \
       --publicurl   "http://radosgw.example.com:8080/swift/v1/AUTH_$(project_id)s" \
       --adminurl    "http://radosgw.example.com:8080/swift/v1/AUTH_$(project_id)s" \
       --internalurl "http://radosgw.example.com:8080/swift/v1/AUTH_$(project_id)s" \
       swift
  +--------------+--------------------------------------------------------------+
  | Field        | Value                                                        |
  +--------------+--------------------------------------------------------------+
  | adminurl     | http://radosgw.example.com:8080/swift/v1/AUTH_$(project_id)s |
  | id           | e4249d2b60e44743a67b5e5b38c18dd3                             |
  | internalurl  | http://radosgw.example.com:8080/swift/v1/AUTH_$(project_id)s |
  | publicurl    | http://radosgw.example.com:8080/swift/v1/AUTH_$(project_id)s |
  | region       | RegionOne                                                    |
  | service_id   | 37c4c0e79571404cb4644201a4a6e5ee                             |
  | service_name | swift                                                        |
  | service_type | object-store                                                 |
  +--------------+--------------------------------------------------------------+

Keystone 与 S3 API 对接
-----------------------
.. Keystone integration with the S3 API

It is possible to use Keystone for authentication even when using the
S3 API (with AWS-like access and secret keys), if the ``rgw s3 auth
use keystone`` option is set. For details, see
:doc:`s3/authentication`.

Service token support
---------------------

Service tokens can be enabled to support RadosGW Keystone integration
to allow expired tokens when coupled with a valid service token in the request.

Enable the support with ``rgw keystone service token enabled`` and use the
``rgw keystone service token accepted roles`` option to specify which roles are considered
service roles.

The ``rgw keystone expired token cache expiration`` option can be used to tune the cache
expiration for an expired token allowed with a service token, please note that this must
be lower than the ``[token]/allow_expired_window`` option in the Keystone configuration.

Enabling this will cause an expired token given in the X-Auth-Token header to be allowed
if coupled with a X-Service-Token header that contains a valid token with the accepted
roles. This can allow long running processes using a user token in X-Auth-Token to function
beyond the expiration of the token.
