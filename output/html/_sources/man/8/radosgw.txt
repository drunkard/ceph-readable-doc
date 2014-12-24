==================================
 radosgw -- rados REST 风格的网关
==================================

.. program:: radosgw

提纲
====

| **radosgw**


描述
====

**radosgw** 是 RADOS 对象存储的一个 HTTP REST 网关，是 Ceph 分布式存储系统的\
一部分。它是用 libfcgi 实现的一个 FastCGI 模块，可联合任何支持 FastCGI 功能\
的网页服务器使用。


选项
====

.. option:: -c ceph.conf, --conf=ceph.conf

   用指定的 *ceph.conf* 配置文件而非默认的 ``/etc/ceph/ceph.conf`` 来确定启\
   动时所需的监视器地址。

.. option:: -m monaddress[:port]

   连接到指定监视器，而非通过 ceph.conf 查询。

.. option:: -i ID, --id ID

   设置 radosgw 名字的 ID 部分。

.. option:: -n TYPE.ID, --name TYPE.ID

   设置网关的 rados 用户名（如 client.radosgw.gateway ）。

.. option:: --cluster NAME

   设置集群名称（默认： ceph ）

.. option:: -d

   在前台运行，日志记录到标准错误

.. option:: -f

   在前台运行，日志记录到正常位置

.. option:: --rgw-socket-path=path

   指定 unix 域套接字的路径

.. option:: --rgw-region=region

   radosgw 所在辖区

.. option:: --rgw-zone=zone

   radosgw 所在的域


配置
====

当前，使用 RADOS 网关最简单的方法就是通过 Apache 与 mod_fastcgi::

	FastCgiExternalServer /var/www/s3gw.fcgi -socket /tmp/radosgw.sock

	<VirtualHost *:80>
	  ServerName rgw.example1.com
	  ServerAlias rgw
	  ServerAdmin webmaster@example1.com
	  DocumentRoot /var/www

	  RewriteEngine On
	  RewriteRule ^/([a-zA-Z0-9-_.]*)([/]?.*) /s3gw.fcgi?page=$1&params=$2&%{QUERY_STRING} [E=HTTP_AUTHORIZATION:%{HTTP:Authorization},L]

	  <IfModule mod_fastcgi.c>
	    <Directory /var/www>
	      Options +ExecCGI
	      AllowOverride All
	      SetHandler fastcgi-script
	      Order allow,deny
	      Allow from all
	      AuthBasicAuthoritative Off
	    </Directory>
	  </IfModule>

	  AllowEncodedSlashes On
	  ServerSignature Off
	</VirtualHost>

与之对应的 radosgw 脚本为 /var/www/s3gw.fcgi::

	#!/bin/sh
	exec /usr/bin/radosgw -c /etc/ceph/ceph.conf -n client.radosgw.gateway

若要以独立进程运行 radosgw 守护进程，需把配置写入 ceph.conf ，配置段落应以 \
'client.radosgw.' 打头，并在 /etc/init.d/radosgw 内指定：

.. code-block:: ini

	[client.radosgw.gateway]
	host = gateway
	keyring = /etc/ceph/keyring.radosgw.gateway
	rgw socket path = /tmp/radosgw.sock

你还得给 radosgw 生成一个密钥，用于和集群认证： ::

	ceph-authtool -C -n client.radosgw.gateway --gen-key /etc/ceph/keyring.radosgw.gateway
	ceph-authtool -n client.radosgw.gateway --cap mon 'allow rw' --cap osd 'allow rwx' /etc/ceph/keyring.radosgw.gateway

并把密钥加入集群： ::

	ceph auth add client.radosgw.gateway --in-file=keyring.radosgw.gateway

现在可以启动 Apache 和 radosgw 守护进程了： ::

	/etc/init.d/apache2 start
	/etc/init.d/radosgw start


记录使用日志
============

**radosgw** 会异步地维护使用率日志，它会累积用户操作统计并周期性地刷回。可用 \
**radosgw-admin** 访问和管理日志。

记录的信息包括数据传输总量、操作总量、成功操作总量。这些数据是按小时记录到桶\
所有者名下的，除非操作是针对服务的（如罗列桶时），这时会记录到操作用户名下。

下面是个配置实例：

.. code-block:: ini

	[client.radosgw.gateway]
	rgw enable usage log = true
	rgw usage log tick interval = 30
	rgw usage log flush threshold = 1024
	rgw usage max shards = 32
	rgw usage max user shards = 1

碎片总数决定着总共需要多少对象来保存使用日志信息。每用户碎片数确定了为单个用\
户保存使用信息需多少对象。 tick interval 可配置刷回日志的间隔秒数， \
flush threshold 决定了保留的日志条数达到多少才调用异步刷回。


使用范围
========

**radosgw** 是 Ceph 分布式文件系统的一部分，更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`ceph <ceph>`\(8)
:doc:`radosgw-admin <radosgw-admin>`\(8)
