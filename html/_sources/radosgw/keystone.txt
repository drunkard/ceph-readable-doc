============================
 与 OpenStack Keystone 对接
============================

It is possible to integrate the Ceph Object Gateway with Keystone, the OpenStack
identity service. This sets up the gateway to accept Keystone as the users
authority. A user that Keystone authorizes to access the gateway will also be
automatically created on the Ceph Object Gateway (if didn't exist beforehand). A
token that Keystone validates will be considered as valid by the gateway.

The following configuration options are available for Keystone integration::

	[client.radosgw.gateway]
	rgw keystone url = {keystone server url:keystone server admin port}
	rgw keystone admin token = {keystone admin token}
	rgw keystone accepted roles = {accepted user roles}
	rgw keystone token cache size = {number of tokens to cache}
	rgw keystone revocation interval = {number of seconds before checking revoked tickets}
	rgw s3 auth use keystone = true
	nss db path = {path to nss db}

A Ceph Object Gateway user is mapped into a Keystone ``tenant``. A Keystone user
has different roles assigned to it on possibly more than a single tenant. When
the Ceph Object Gateway gets the ticket, it looks at the tenant, and the user
roles that are assigned to that ticket, and accepts/rejects the request
according to the ``rgw keystone accepted roles`` configurable.


Kilo 之前
---------

Keystone 自身作为对象存储服务的入口（ endpoint ），需要配置为指向 Ceph 对\
象网关。 ::

	keystone service-create --name swift --type object-store
	keystone endpoint-create --service-id <id> \
		--publicurl   http://radosgw.example.com/swift/v1 \
		--internalurl http://radosgw.example.com/swift/v1 \
		--adminurl    http://radosgw.example.com/swift/v1


从 Kilo 起
----------

Keystone 自身作为对象存储服务的入口（ endpoint ），需要配置为指向 Ceph 对\
象网关。 ::

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


The keystone URL is the Keystone admin RESTful API URL. The admin token is the
token that is configured internally in Keystone for admin requests.

The Ceph Object Gateway will query Keystone periodically for a list of revoked
tokens. These requests are encoded and signed. Also, Keystone may be configured
to provide self-signed tokens, which are also encoded and signed. The gateway
needs to be able to decode and verify these signed messages, and the process
requires that the gateway be set up appropriately. Currently, the Ceph Object
Gateway will only be able to perform the procedure if it was compiled with
``--with-nss``. Configuring the Ceph Object Gateway to work with Keystone also
requires converting the OpenSSL certificates that Keystone uses for creating the
requests to the nss db format, for example::

	mkdir /var/ceph/nss

	openssl x509 -in /etc/keystone/ssl/certs/ca.pem -pubkey | \
		certutil -d /var/ceph/nss -A -n ca -t "TCu,Cu,Tuw"
	openssl x509 -in /etc/keystone/ssl/certs/signing_cert.pem -pubkey | \
		certutil -A -d /var/ceph/nss -n signing_cert -t "P,P,P"


OpenStack 的 keystone 组件也可以用自签名的 SSL 证书来终结，\
要使 radosgw 有能力与这种 keystone 交互，你可以在运行
radosgw 的节点上安装 keystone 的 SSL 证书；另外， radosgw
也可以配置为根本不校验 SSL 证书（类似加了 ``--insecure``
开关的 openstack 客户端请求），即把
``rgw keystone verify ssl`` 配置为 ``false`` 。
