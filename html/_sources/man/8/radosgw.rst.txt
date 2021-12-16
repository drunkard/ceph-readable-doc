:orphan:

==================================
 radosgw -- rados REST 风格的网关
==================================

.. program:: radosgw

提纲
====

| **radosgw**


描述
====

:program:`radosgw` 是 RADOS 对象存储的一个 HTTP REST 网关，
是 Ceph 分布式存储系统的一部分。
它是用 libfcgi 实现的一个 FastCGI 模块，
可联合任何支持 FastCGI 功能的网页服务器使用。


选项
====

.. option:: -c ceph.conf, --conf=ceph.conf

   用指定的 ``ceph.conf`` 配置文件而非默认的 \
   ``/etc/ceph/ceph.conf`` 来确定启动时所需的监视器地址。

.. option:: -m monaddress[:port]

   连接到指定监视器，而非通过 ``ceph.conf`` 查询。

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

   指定 Unix 域套接字的路径

.. option:: --rgw-region=region

   radosgw 所在 region

.. option:: --rgw-zone=zone

   radosgw 所在的区域


配置
====

先前的 RADOS 网关配置依赖 ``Apache`` 和 ``mod_fastcgi`` ；
现在则用 ``mod_proxy_fcgi`` 替换了 ``mod_fastcgi`` ，
``mod_proxy_fcgi`` 的工作方式不同于传统的 FastCGI 模块，
它需要 ``mod_proxy`` 模块所支持的 FastCGI 协议。
所以，要处理 FastCGI 协议，服务器需同时有
``mod_proxy`` 和 ``mod_proxy_fcgi`` 模块。
不像 ``mod_fastcgi`` ， ``mod_proxy_fcgi`` 不能启动应用进程。
某些平台提供了 ``fcgistarter`` 来实现此功能。
然而， FastCGI 应用框架有可能具备\
外部启动或进程管理功能。

``Apache`` 可以通过本机 TCP 连接或 Unix 域套接字使用
``mod_proxy_fcgi`` 模块。不支持 Unix 域套接字的 ``mod_proxy_fcgi`` ，
像 Apache 2.2 和 2.4 的早期版本，
必需通过本机 TCP 连接。
之后的 Apache （如 2.4.9 ）以及更高版本支持 unix 域套接字，
因此它们支持 unix 域套接字配置、而非本机 TCP 。

下面的步骤展示了一些配置文件内容，
有 Ceph 配置文件 ``/etc/ceph/ceph.conf`` ，
和网关配置文件 ``/etc/httpd/conf.d/rgw.conf`` （基于 RPM 的发行版），
或者 ``/etc/apache2/conf-available/rgw.conf`` （基于 Debian 的发行版），
本机 TCP 和通过 unix 域套接字的配置都有：

#. 对于搭载 Apache 2.2 以及 Apache 2.4 早期版本的发行版来说，
   它们使用 TCP 且不支持 Unix 域套接字，
   把下面的内容加进 ``/etc/ceph/ceph.conf`` 文件： ::

    [client.radosgw.gateway]
    host = {hostname}
    keyring = /etc/ceph/ceph.client.radosgw.keyring
    rgw socket path = ""
    log file = /var/log/ceph/client.radosgw.gateway.log
    rgw frontends = fastcgi socket_port=9000 socket_host=0.0.0.0
    rgw print continue = false

#. 把下列内容加入网关配置文件：

   在 Debian/Ubuntu 上，加入 ``/etc/apache2/conf-available/rgw.conf`` ::

        <VirtualHost *:80>
        ServerName localhost
        DocumentRoot /var/www/html

        ErrorLog /var/log/apache2/rgw_error.log
        CustomLog /var/log/apache2/rgw_access.log combined

        # LogLevel debug

        RewriteEngine On

        RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization},L]

        SetEnv proxy-nokeepalive 1

        ProxyPass / fcgi://localhost:9000/

        </VirtualHost>

   在 CentOS/RHEL 上，加入 ``/etc/httpd/conf.d/rgw.conf``::

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

#. 对于搭载了支持 Unix 域套接字的 Apache 2.4.9 及更高版的发行版，\
   可使用下列配置： ::

    [client.radosgw.gateway]
    host = {hostname}
    keyring = /etc/ceph/ceph.client.radosgw.keyring
    rgw socket path = /var/run/ceph/ceph.radosgw.gateway.fastcgi.sock
    log file = /var/log/ceph/client.radosgw.gateway.log
    rgw print continue = false

#. 把下列内容加入网关配置文件中：

   在 CentOS/RHEL 上，加入 ``/etc/httpd/conf.d/rgw.conf``::

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

   请注意， ``Apache 2.4.7`` 不支持 Unix 域套接字，
   所以必须配置成本机 TCP 。
   Unix 域套接字支持存在于 ``Apache 2.4.9`` 及其后续版本中。

#. 给 radosgw 生成一个密钥，用于到集群认证。 ::

    ceph-authtool -C -n client.radosgw.gateway --gen-key /etc/ceph/keyring.radosgw.gateway
    ceph-authtool -n client.radosgw.gateway --cap mon 'allow rw' --cap osd 'allow rwx' /etc/ceph/keyring.radosgw.gateway

#. 把密钥导入集群。 ::

    ceph auth add client.radosgw.gateway --in-file=keyring.radosgw.gateway

#. 启动 Apache 和 radosgw 。

   Debian/Ubuntu::

        sudo /etc/init.d/apache2 start
        sudo /etc/init.d/radosgw start

   CentOS/RHEL::

        sudo apachectl start
        sudo /etc/init.d/ceph-radosgw start


记录使用日志
============

:program:`radosgw` 会异步地维护使用率日志，
它会累积用户操作统计并周期性地刷回。
可用 :program:`radosgw-admin` 访问和管理日志。

记录的信息包括数据传输总量、
操作总量、成功操作总量。
这些数据是按小时记录到桶所有者名下的，
除非操作是针对服务的（如罗列桶时），
这时会记录到操作用户名下。

下面是个配置实例：

.. code-block:: ini

    [client.radosgw.gateway]
    rgw enable usage log = true
    rgw usage log tick interval = 30
    rgw usage log flush threshold = 1024
    rgw usage max shards = 32
    rgw usage max user shards = 1

碎片总数决定着总共需要多少对象来保存使用日志信息。
每用户碎片数确定了为单个用户保存使用信息需多少对象。
tick interval 可配置刷回日志的间隔秒数，
flush threshold 决定了保留的日志条数\
达到多少才调用异步刷回。


使用范围
========

:program:`radosgw` 是 Ceph 的一部分，这是个伸缩力强、开源、
分布式的存储系统，更多信息参见 https://docs.ceph.com 。


参考
====

:doc:`ceph <ceph>`\(8)
:doc:`radosgw-admin <radosgw-admin>`\(8)
