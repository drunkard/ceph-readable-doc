:orphan:

=========================================
 rbd-mirror -- 可同步 RBD 映像的守护进程
=========================================

.. program:: rbd-mirror

提纲
====

| **rbd-mirror**


描述
====

:program:`rbd-mirror` 是个守护进程，用于在 Ceph 集群间异步\
地镜像 RADOS 块设备（ rbd ）映像。它可以在远端集群重放本地\
集群内的映像变更，以实现异地灾备。

它用 RADOS 协议连接远端集群，需要从默认路径搜索 ceph.conf
配置文件、监视器地址、及其认证信息，即
``/etc/ceph/$cluster.conf`` 、 ``/etc/ceph/$cluster.keyring``
和 ``/etc/ceph/$cluster.$name.keyring`` ，其中 ``$cluster``
表示自定义的集群名字、 ``$name`` 表示连接时所用的 rados 用\
户，如 ``client.rbd-mirror`` 。


选项
====

.. option:: -c ceph.conf, --conf=ceph.conf

   在启动期间，读取指定的配置文件 ``ceph.conf`` ，而不是默认\
   的 ``/etc/ceph/ceph.conf`` ，并从中提取监视器地址。

.. option:: -m monaddress[:port]

   连接到指定监视器（而非在 ``ceph.conf`` 里面找）。

.. option:: -i ID, --id ID

   为 rbd-mirror 指定名字的 ID 部分。

.. option:: -n TYPE.ID, --name TYPE.ID

   设置网关所需的 rados 用户名（如 client.rbd-mirror ）。

.. option:: --cluster NAME

   指定集群名字（默认为 ceph ）。

.. option:: -d

   在前台运行，日志显示到 stderr 。

.. option:: -f

   在前台运行，日志走向不受影响。


使用范围
========

:program:`rbd-mirror` 是 Ceph 的一部分，这是个伸缩力强、开源、\
分布式的存储系统，更多信息参见 Ceph 文档 http://ceph.com/docs 。


参考
====

:doc:`rbd <rbd>`\(8)
