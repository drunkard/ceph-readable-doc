=====================
 Ceph 对象网关的配置
=====================

要配置 Ceph 对象网关需要一个运行着的 Ceph 存储集群，还有启用了 FastCGI 模块\
的 Apache 网页服务器。

Ceph 对象网关是 Ceph 存储集群的一个客户端，作为 Ceph 存储集群的客户端，它需要：

- A name for the gateway instance. We use ``gateway`` in this guide.
- A storage cluster user name with appropriate permissions in a keyring.
- Pools to store its data.
- A data directory for the gateway instance.
- An instance entry in the Ceph Configuration file.
- A configuration file for the web server to interact with FastCGI.


Create a User and Keyring
=========================

Each instance must have a user name and key to communicate with a Ceph Storage
Cluster. In the following steps, we use an admin node to create a keyring. 
Then, we create a client user name and key. Next, we add the 
key to the Ceph Storage Cluster. Finally, we distribute the key ring to 
the node containing the gateway instance.

.. topic:: Monitor Key CAPS

   When you provide CAPS to the key, you MUST provide read capability.
   However, you have the option of providing write capability for the monitor. 
   This is an important choice. If you provide write capability to the key, 
   the Ceph Object Gateway will have the ability to create pools automatically; 
   however, it will create pools with either the default number of placement 
   groups (not ideal) or the number of placement groups you specified in your 
   Ceph configuration file. If you allow the Ceph Object Gateway to create 
   pools automatically, ensure that you have reasonable defaults for the number
   of placement groups first. See `Pool Configuration`_ for details.


关于 Ceph 认证请参考\ `用户管理`_\ 。

#. 给这个网关创建一个密钥环： ::

	sudo ceph-authtool --create-keyring /etc/ceph/ceph.client.radosgw.keyring
	sudo chmod +r /etc/ceph/ceph.client.radosgw.keyring


#. Generate a Ceph Object Gateway user name and key for each instance. For
   exemplary purposes, we will use the name ``gateway`` after ``client.radosgw``:: 

	sudo ceph-authtool /etc/ceph/ceph.client.radosgw.keyring -n client.radosgw.gateway --gen-key


#. Add capabilities to the key. See `配置参考——存储池`_ for details
   on the effect of write permissions for the monitor and creating pools. ::

	sudo ceph-authtool -n client.radosgw.gateway --cap osd 'allow rwx' --cap mon 'allow rwx' /etc/ceph/ceph.client.radosgw.keyring


#. Once you have created a keyring and key to enable the Ceph Object Gateway 
   with access to the Ceph Storage Cluster, add the key to your 
   Ceph Storage Cluster. For example::

	sudo ceph -k /etc/ceph/ceph.client.admin.keyring auth add client.radosgw.gateway -i /etc/ceph/ceph.client.radosgw.keyring


#. Distribute the keyring to the node with the gateway instance. ::

	sudo scp /etc/ceph/ceph.client.radosgw.keyring  ceph@{hostname}:/home/ceph
	ssh {hostname}
	sudo mv ceph.client.radosgw.keyring /etc/ceph/ceph.client.radosgw.keyring

   .. note:: 如果 ``admin node`` 就是 ``gateway host`` ，那第 5 步就没必要。


创建存储池
==========

Ceph Object Gateways require Ceph Storage Cluster pools to store specific
gateway data.  If the user you created has permissions, the gateway
will create the pools automatically. However, you should ensure that you have
set an appropriate default number of placement groups per pool into your Ceph
configuration file.

.. note:: Ceph 对象网关有多个存储池，考虑到所有存储池会共用相同的 CRUSH 分级\
   结构，所以 PG 数不要设置得过大，否则会影响性能。

When configuring a gateway with the default region and zone, the naming
convention for pools typically omits region and zone naming, but you can use any
naming convention you prefer. For example:


- ``.rgw``
- ``.rgw.root``
- ``.rgw.control``
- ``.rgw.gc``
- ``.rgw.buckets``
- ``.rgw.buckets.index``
- ``.rgw.buckets.extra``
- ``.log``
- ``.intent-log``
- ``.usage``
- ``.users``
- ``.users.email``
- ``.users.swift``
- ``.users.uid``


See `配置参考——存储池`_ for details on the default pools for
gateways. See `Pools`_ for details on creating pools. As already said, if
write permission is given, Ceph Object Gateway will create pools automatically.
To create a pool manually, execute the following::

	ceph osd pool create {poolname} {pg-num} {pgp-num} {replicated | erasure} [{erasure-code-profile}]  {ruleset-name} {ruleset-number}

.. tip:: Ceph 允许多级 CRUSH 和多种 CRUSH 规则集，这样在配置你自己的网关时就\
   有很大的灵活性。像 ``rgw.buckets.index`` 这样的存储池就可以利用副本数适当\
   的 SSD 存储池取得高性能；后端存储也可以从更经济的纠删编码的存储中获益，还\
   可能利用缓存层提升性能。

完成此步骤之后，执行下列命令确认前述存储池是否都创建了： ::

	rados lspools


为 Ceph 新增网关配置
====================

Add the Ceph Object Gateway configuration to your Ceph Configuration file in
``admin node``. The Ceph Object Gateway configuration requires you to
identify the Ceph Object Gateway instance. Then, you must specify the host name
where you installed the Ceph Object Gateway daemon, a keyring (for use with
cephx), the socket path for FastCGI and a log file.

For distros with Apache 2.2 and early versions of Apache 2.4 (RHEL 6, Ubuntu
12.04, 14.04 etc), append the following configuration to ``/etc/ceph/ceph.conf``
in your ``admin node``::

	[client.radosgw.gateway]
	host = {hostname}
	keyring = /etc/ceph/ceph.client.radosgw.keyring
	rgw socket path = ""
	log file = /var/log/radosgw/client.radosgw.gateway.log
	rgw frontends = fastcgi socket_port=9000 socket_host=0.0.0.0
	rgw print continue = false


.. note:: Apache 2.2 and early versions of Apache 2.4 do not use Unix Domain
   Sockets but use localhost TCP.

For distros with Apache 2.4.9 or later (RHEL 7, CentOS 7 etc), append the
following configuration to ``/etc/ceph/ceph.conf`` in your ``admin node``::

	[client.radosgw.gateway]
	host = {hostname}
	keyring = /etc/ceph/ceph.client.radosgw.keyring
	rgw socket path = /var/run/ceph/ceph.radosgw.gateway.fastcgi.sock
	log file = /var/log/radosgw/client.radosgw.gateway.log
	rgw print continue = false


.. note:: ``Apache 2.4.9`` supports Unix Domain Socket (UDS) but as
   ``Ubuntu 14.04`` ships with ``Apache 2.4.7`` it doesn't have UDS support and
   has to be configured for use with localhost TCP. A bug has been filed for
   backporting UDS support in ``Apache 2.4.7`` for ``Ubuntu 14.04``.
   See: `Backport support for UDS in Ubuntu Trusty`_

Here, ``{hostname}`` is the short hostname (output of command ``hostname -s``)
of the node that is going to provide the gateway service i.e, the
``gateway host``.

The ``[client.radosgw.gateway]`` portion of the gateway instance identifies this
portion of the Ceph configuration file as configuring a Ceph Storage Cluster
client where the client type is a Ceph Object Gateway (i.e., ``radosgw``).


.. note:: The last line in the configuration i.e, ``rgw print continue = false``
   is added to avoid issues with ``PUT`` operations.

Once you finish the setup procedure, if you encounter issues with your
configuration, you can add debugging to the ``[global]`` section of your Ceph
configuration file and restart the gateway to help troubleshoot any
configuration issues. For example::

	[global]
	#append the following in the global section.
	debug ms = 1
	debug rgw = 20


Distribute updated Ceph configuration file
==========================================

The updated Ceph configuration file needs to be distributed to all Ceph cluster
nodes from the ``admin node``.

It involves the following steps:

#. Pull the updated ``ceph.conf`` from ``/etc/ceph/`` to the root directory of
   the cluster in admin node (e.g. ``my-cluster`` directory). The contents of
   ``ceph.conf`` in ``my-cluster`` will get overwritten. To do so, execute the
   following::

		ceph-deploy --overwrite-conf config pull {hostname}

   Here, ``{hostname}`` is the short hostname of the Ceph admin node.

#. Push the updated ``ceph.conf`` file from the admin node to all other nodes in
   the cluster including the ``gateway host``::

		ceph-deploy --overwrite-conf config push [HOST] [HOST...]

   Give the hostnames of the other Ceph nodes in place of ``[HOST] [HOST...]``.


Copy ceph.client.admin.keyring from admin node to gateway host
==============================================================

As the ``gateway host`` can be a different node that is not part of the cluster,
the ``ceph.client.admin.keyring`` needs to be copied from the ``admin node`` to
the ``gateway host``. To do so, execute the following on ``admin node``::

	sudo scp /etc/ceph/ceph.client.admin.keyring  ceph@{hostname}:/home/ceph
	ssh {hostname}
	sudo mv ceph.client.admin.keyring /etc/ceph/ceph.client.admin.keyring


.. note:: The above step need not be executed if ``admin node`` is the
   ``gateway host``.


Create Data Directory
=====================

Deployment scripts may not create the default Ceph Object Gateway data
directory. Create data directories for each instance of a ``radosgw``
daemon (if you haven't done so already). The ``host`` variables in the
Ceph configuration file determine which host runs each instance of a
``radosgw`` daemon. The typical form specifies the ``radosgw`` daemon,
the cluster name and the daemon ID.

To create the directory on the ``gateway host``, execute the following::

	sudo mkdir -p /var/lib/ceph/radosgw/ceph-radosgw.gateway


Adjust Socket Directory Permissions
===================================

On some distros, the ``radosgw`` daemon runs as the unprivileged ``apache``
UID, and this UID must have write access to the location where it will write
its socket file.

To grant permissions to the default socket location, execute the following on
the ``gateway host``::

	sudo chown apache:apache /var/run/ceph


Change Log File Owner
=====================

On some distros, the ``radosgw`` daemon runs as the unprivileged ``apache`` UID,
but the ``root`` user owns the log file by default. You must change it to the
``apache`` user so that Apache can populate the log file. To do so, execute
the following::

	sudo chown apache:apache /var/log/radosgw/client.radosgw.gateway.log


Start radosgw service
=====================

The Ceph Object gateway daemon needs to be started. To do so, execute the
following on the ``gateway host``:

On Debian-based distros::

	sudo /etc/init.d/radosgw start

On RPM-based distros::

	sudo /etc/init.d/ceph-radosgw start


Create a Gateway Configuration file
===================================

On the host where you installed the Ceph Object Gateway i.e, ``gateway host``,
create an ``rgw.conf`` file. Place the file in ``/etc/apache2/conf-available``
directory for ``Debian-based`` distros and in ``/etc/httpd/conf.d`` directory
for ``RPM-based`` distros. It is a Apache configuration file which is needed
for the ``radosgw`` service. This file must be readable by the web server.

Execute the following steps:

#. Create the file:

   For Debian-based distros, execute::

	sudo vi /etc/apache2/conf-available/rgw.conf

   For RPM-based distros, execute::

	sudo vi /etc/httpd/conf.d/rgw.conf

#. For distros with Apache 2.2 and early versions of Apache 2.4 that use
   localhost TCP and do not support Unix Domain Socket, add the following
   contents to the file::

	<VirtualHost *:80>
	ServerName localhost
	DocumentRoot /var/www/html

	ErrorLog /var/log/httpd/rgw_error.log
	CustomLog /var/log/httpd/rgw_access.log combined

	# LogLevel debug

	RewriteEngine On

	RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization},L]

	SetEnv proxy-nokeepalive 1

	ProxyPass / fcgi://localhost:9000/

	</VirtualHost>

   .. note:: For Debian-based distros replace ``/var/log/httpd/``
      with ``/var/log/apache2``.

#. For distros with Apache 2.4.9 or later that support Unix Domain Socket,
   add the following contents to the file::

	<VirtualHost *:80>
	ServerName localhost
	DocumentRoot /var/www/html

	ErrorLog /var/log/httpd/rgw_error.log
	CustomLog /var/log/httpd/rgw_access.log combined

	# LogLevel debug

	RewriteEngine On

	RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization},L]

	SetEnv proxy-nokeepalive 1

	ProxyPass / unix:///var/run/ceph/ceph.radosgw.gateway.fastcgi.sock|fcgi://localhost:9000/

	</VirtualHost>


Restart Apache
==============

The Apache service needs to be restarted to accept the new configuration.

For Debian-based distros, run::

	sudo service apache2 restart

For RPM-based distros, run::

	sudo service httpd restart

Or::

	sudo systemctl restart httpd


Using The Gateway
=================

To use the REST interfaces, first create an initial Ceph Object Gateway
user for the S3 interface. Then, create a subuser for the Swift interface.
See the `Admin Guide`_ for more details on user management.

Create a radosgw user for S3 access
------------------------------------

A ``radosgw`` user needs to be created and granted access. The command
``man radosgw-admin`` will provide information on additional command options.

To create the user, execute the following on the ``gateway host``::

	sudo radosgw-admin user create --uid="testuser" --display-name="First User"

The output of the command will be something like the following::

	{"user_id": "testuser",
	"display_name": "First User",
	"email": "",
	"suspended": 0,
	"max_buckets": 1000,
	"auid": 0,
	"subusers": [],
	"keys": [
	{ "user": "testuser",
	"access_key": "I0PJDPCIYZ665MW88W9R",
	"secret_key": "dxaXZ8U90SXydYzyS5ivamEP20hkLSUViiaR+ZDA"}],
	"swift_keys": [],
	"caps": [],
	"op_mask": "read, write, delete",
	"default_placement": "",
	"placement_tags": [],
	"bucket_quota": { "enabled": false,
	"max_size_kb": -1,
	"max_objects": -1},
	"user_quota": { "enabled": false,
	"max_size_kb": -1,
	"max_objects": -1},
	"temp_url_keys": []}


.. note:: 验证访问时， ``keys->access_key`` 和 ``keys->secret_key`` \
   的值是必需的。


创建一个 Swift 用户
-------------------

如果要通过 Swift 访问，必须创建一个 Swift 子用户。需要分两步完成，\
第一步是创建用户，第二步创建密钥。

在 ``gateway host`` 主机上进行如下操作：

创建 Swift 用户： ::

	sudo radosgw-admin subuser create --uid=testuser --subuser=testuser:swift --access=full

此命令的输出类似如下： ::

	{ "user_id": "testuser",
	"display_name": "First User",
	"email": "",
	"suspended": 0,
	"max_buckets": 1000,
	"auid": 0,
	"subusers": [
	{ "id": "testuser:swift",
	"permissions": "full-control"}],
	"keys": [
	{ "user": "testuser:swift",
	"access_key": "3Y1LNW4Q6X0Y53A52DET",
	"secret_key": ""},
	{ "user": "testuser",
	"access_key": "I0PJDPCIYZ665MW88W9R",
	"secret_key": "dxaXZ8U90SXydYzyS5ivamEP20hkLSUViiaR+ZDA"}],
	"swift_keys": [],
	"caps": [],
	"op_mask": "read, write, delete",
	"default_placement": "",
	"placement_tags": [],
	"bucket_quota": { "enabled": false,
	"max_size_kb": -1,
	"max_objects": -1},
	"user_quota": { "enabled": false,
	"max_size_kb": -1,
	"max_objects": -1},
	"temp_url_keys": []}

创建用户的密钥： ::

	sudo radosgw-admin key create --subuser=testuser:swift --key-type=swift --gen-secret

此命令的输出类似如下： ::

	{ "user_id": "testuser",
	"display_name": "First User",
	"email": "",
	"suspended": 0,
	"max_buckets": 1000,
	"auid": 0,
	"subusers": [
	{ "id": "testuser:swift",
	"permissions": "full-control"}],
	"keys": [
	{ "user": "testuser:swift",
	"access_key": "3Y1LNW4Q6X0Y53A52DET",
	"secret_key": ""},
	{ "user": "testuser",
	"access_key": "I0PJDPCIYZ665MW88W9R",
	"secret_key": "dxaXZ8U90SXydYzyS5ivamEP20hkLSUViiaR+ZDA"}],
	"swift_keys": [
	{ "user": "testuser:swift",
	"secret_key": "244+fz2gSqoHwR3lYtSbIyomyPHf3i7rgSJrF\/IA"}],
	"caps": [],
	"op_mask": "read, write, delete",
	"default_placement": "",
	"placement_tags": [],
	"bucket_quota": { "enabled": false,
	"max_size_kb": -1,
	"max_objects": -1},
	"user_quota": { "enabled": false,
	"max_size_kb": -1,
	"max_objects": -1},
	"temp_url_keys": []}


访问验证
========

然后你得验证一下刚创建的用户是否能访问网关。


测试 S3 访问
------------

You need to write and run a Python test script for verifying S3 access. The S3
access test script will connect to the ``radosgw``, create a new bucket and list
all buckets. The values for ``aws_access_key_id`` and ``aws_secret_access_key``
are taken from the values of ``access_key`` and ``secret_key`` returned by the
``radosgw_admin`` command.

Execute the following steps:

#. You will need to install the ``python-boto`` package.

   For Debian-based distros, run::

		sudo apt-get install python-boto

   For RPM-based distros, run::

		sudo yum install python-boto

#. Create the Python script::

	vi s3test.py

#. Add the following contents to the file::

	import boto
	import boto.s3.connection
	access_key = 'I0PJDPCIYZ665MW88W9R'
	secret_key = 'dxaXZ8U90SXydYzyS5ivamEP20hkLSUViiaR+ZDA'
	conn = boto.connect_s3(
	aws_access_key_id = access_key,
	aws_secret_access_key = secret_key,
	host = '{hostname}',
	is_secure=False,
	calling_format = boto.s3.connection.OrdinaryCallingFormat(),
	)
	bucket = conn.create_bucket('my-new-bucket')
	for bucket in conn.get_all_buckets():
		print "{name}\t{created}".format(
			name = bucket.name,
			created = bucket.creation_date,
	)

   Replace ``{hostname}`` with the hostname of the host where you have
   configured the gateway service i.e, the ``gateway host``.

#. Run the script::

	python s3test.py

   The output will be something like the following::

		my-new-bucket 2015-02-16T17:09:10.000Z

Test swift access
-----------------

Swift access can be verified via the ``swift`` command line client. The command
``man swift`` will provide more information on available command line options.

To install ``swift`` client, execute the following:

   For Debian-based distros::

		sudo apt-get install python-setuptools
		sudo easy_install pip
		sudo pip install --upgrade setuptools
		sudo pip install --upgrade python-swiftclient

   For RPM-based distros::

		sudo yum install python-setuptools
		sudo easy_install pip
		sudo pip install --upgrade setuptools
		sudo pip install --upgrade python-swiftclient

To test swift access, execute the following::

	swift -A http://{IP ADDRESS}/auth/1.0 -U testuser:swift -K ‘{swift_secret_key}’ list

Replace ``{IP ADDRESS}`` with the public IP address of the gateway server and
``{swift_secret_key}`` with its value from the output of
``radosgw-admin key create`` command executed for the ``swift`` user.

For example::

	swift -A http://10.19.143.116/auth/1.0 -U testuser:swift -K ‘244+fz2gSqoHwR3lYtSbIyomyPHf3i7rgSJrF/IA’ list

The output should be::

	my-new-bucket




.. _配置参考——存储池: ../config-ref#pools
.. _Pool Configuration: ../../rados/configuration/pool-pg-config-ref/
.. _Pools: ../../rados/operations/pools
.. _用户管理: ../../rados/operations/user-management
.. _Backport support for UDS in Ubuntu Trusty: https://bugs.launchpad.net/ubuntu/+source/apache2/+bug/1411030
.. _Admin Guide: ../admin
