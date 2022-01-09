:orphan:

===================================================================
 ceph-immutable-object-cache -- 用于不可变对象缓存的 Ceph 守护进程
===================================================================

.. program:: ceph-immutable-object-cache

提纲
====

| **ceph-immutable-object-cache**


描述
====

:program:`ceph-immutable-object-cache` 是个守护进程，
用于缓存多个 Ceph 集群的 RADOS 对象。在遇到请求时，
它会把对象提升到本地目录，以后有请求就用这些缓存的对象\
响应。

它用 RADOS 协议连接本地集群，
靠默认搜索路径寻找 ceph.conf 文件、
监视器地址及其认证信息，即
``/etc/ceph/$cluster.conf`` 、 ``/etc/ceph/$cluster.keyring`` 、
和 ``/etc/ceph/$cluster.$name.keyring`` ，其中
``$cluster`` 是人类可读的集群名、 ``$name`` 是用来连接的 rados 用户，
例如 ``client.ceph-immutable-object-cache`` 。


选项
====

.. option:: -c ceph.conf, --conf=ceph.conf

   用 ``ceph.conf`` 配置文件，而非默认的
   ``/etc/ceph/ceph.conf`` 来确定启动期间所需的监视器地址。

.. option:: -m monaddress[:port]

   连接到指定监视器（而非通过 ``ceph.conf`` 查找）。

.. option:: -i ID, --id ID

   设置 ceph-immutable-object-cache 名字的 ID 部分。

.. option:: -n TYPE.ID, --name TYPE.ID

   给网关设置 rados 用户名（例如 client.ceph-immutable-object-cache ）

.. option:: --cluster NAME

   设置集群名（默认： ceph ）

.. option:: -d

   在前台运行，日志发往 stderr

.. option:: -f

   在前台运行，日志写入往常位置。


使用范围
========

:program:`ceph-immutable-object-cache` 是 Ceph 的一部分，
这是个伸缩力强、开源、分布式的存储系统，
更多信息参见 https://docs.ceph.com 。


参考
====

:doc:`rbd <rbd>`\(8)
