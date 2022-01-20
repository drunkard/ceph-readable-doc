==================
 RBD 永久只读缓存
==================

.. index:: Ceph Block Device; Persistent Read-only Cache

共享、只读的父映像缓存
======================
.. Shared, Read-only Parent Image Cache

从父映像 `克隆的 RBD 映像`_ 只对父映像更改了很小一部分。例如，
在一个 VDI 应用案例中， VM 都是从同一个基础映像克隆来的，
最初的差异只有主机名和 IP 地址。在启动期间，所有这些 VM 都会\
读取同一父映像数据的一部分。如果我们有这个父映像的本地缓存，
在这个有缓存的主机上就可以提速读取操作。
还能减少客户端到集群的网络流量。RBD 缓存必须在 ``ceph.conf`` 里显式地启用。
``ceph-immutable-object-cache`` 守护进程负责把父映像的内容缓存在本地磁盘上，
以后再有那些数据的读取就从本地缓存中读取。

.. note:: RBD 共享、只读的父映像缓存需要 Ceph Nautilus 或更高版本。

.. ditaa::

            +--------------------------------------------------------+
            |                         QEMU                           |
            +--------------------------------------------------------+
            |                librbd (cloned images)                  |
            +-------------------+-+----------------------------------+
            |      librados     | |  ceph--immutable--object--cache  |
            +-------------------+ +----------------------------------+
            |      OSDs/Mons    | |     local cached parent image    |
            +-------------------+ +----------------------------------+


启用 RBD 的共享只读父映像缓存功能
---------------------------------
.. Enable RBD Shared Read-only Parent Image Cache

要启用 RBD 共享、只读的父映像缓存功能，需要把下列选项加进
``ceph.conf`` 文件的 ``[client]`` `段`_ 下： ::

        rbd parent cache enabled = true
        rbd plugins = parent_cache


不可变对象缓存守护进程
======================
.. Immutable Object Cache Daemon

介绍及常规选项
--------------
.. Introduction and Generic Settings

``ceph-immutable-object-cache`` 守护进程负责把父映像内容缓存进本地的缓存目录里。
要想提高性能，建议用 SSD 作为底层存储器。

这个守护进程的关键组件有：

#. **基于 IPC 的域套接字:** 这个守护进程在启动时会监听本地域套接字，
   并等待 librbd 客户端连接。

#. **基于 LRU 的晋级、降级策略:** 这个守护进程\
   会在内存里维护各个缓存文件的命中统计信息。
   如果缓存容量达到了配置的阈值，它就会降级冷缓存。

#. **基于文件的缓存库:** 这个守护进程\
   维护着一个简单的基于文件的缓存库。需要晋级时，
   会从 RADOS 集群取回 RADOS 对象、并存储在本地的缓存目录里。

打开各个克隆的 rbd 映像时， ``librbd`` 会尝试通过它的 Unix 域套接字连接到\
缓存守护进程。成功连接后， ``librbd`` 会和那个守护进程协调后续的读取操作。
如果有没缓存的读取，缓存守护进程会把那个 RADOS 对象晋级到本地缓存目录，
这样下次读取这个对象时就可以从缓存读取。缓存守护进程维护着简单的 LRU 统计信息，
以便容量不足时可以按需逐出冷缓存文件。

这里是一些重要的缓存配置选项：

``immutable_object_cache_sock``

:描述: 域套接字路径， librbd 客户端和
       ceph-immutable-object-cache 守护进程之间通讯用的。
:类型: String
:是否必需: No
:默认值: ``/var/run/ceph/immutable_object_cache_sock``


``immutable_object_cache_path``

:描述: 不可变对象缓存的数据目录。
:类型: String
:是否必需: No
:默认值: ``/tmp/ceph_immutable_object_cache``


``immutable_object_cache_max_size``

:描述: 不可变缓存的最大尺寸。
:类型: Size
:是否必需: No
:默认值: ``1G``


``immutable_object_cache_watermark``

:描述: 缓存的最大量（最高水位），数值在 (0, 1) 二者之间。如果缓存的尺寸\
       达到了这个阈值，守护进程就根据 LRU 统计信息开始删除冷缓存。
:类型: Float
:是否必需: No
:默认值: ``0.9``

可选的 ``ceph-immutable-object-cache`` 软件包里包含这个守护进程。

.. important:: ``ceph-immutable-object-cache`` 守护进程要能连接 RADOS 集群。


如何运行不可变对象缓存守护进程
------------------------------
.. Running the Immutable Object Cache Daemon

``ceph-immutable-object-cache`` 守护进程应该用一个唯一的 Ceph 用户 ID 。
要 `新建一个 Ceph 用户`_ ，用 ``ceph`` 命令加上 ``auth get-or-create`` 子命令、
用户名、监视器能力、和 OSD 能力： ::

  ceph auth get-or-create client.ceph-immutable-object-cache.{unique id} mon 'allow r' osd 'profile rbd-read-only'

``ceph-immutable-object-cache`` 守护进程可以用 ``systemd`` 管理，
指定用户 ID 作为守护进程例程： ::

  systemctl enable ceph-immutable-object-cache@ceph-immutable-object-cache.{unique id}

``ceph-immutable-object-cache`` 也能在前台运行，用 ``ceph-immutable-object-cache`` 命令::

  ceph-immutable-object-cache -f --log-file={log_path}


QoS 选项
--------
.. QOS Settings

不可变对象缓存支持节流，由下列选项控制：

``immutable_object_cache_qos_schedule_tick_min``

:描述: 不可变对象缓存的最小时间片。
:类型: Milliseconds
:是否必需: No
:默认值: ``50``


``immutable_object_cache_qos_iops_limit``

:描述: 不可变对象缓存 IO 操作期望的每秒限额。
:类型: Unsigned Integer
:是否必需: No
:默认值: ``0``


``immutable_object_cache_qos_iops_burst``

:描述: 不可变对象缓存 IO 操作允许的爆发量。
:类型: Unsigned Integer
:是否必需: No
:默认值: ``0``


``immutable_object_cache_qos_iops_burst_seconds``

:描述: 不可变对象缓存 IO 操作允许的爆发时长。
:类型: Seconds
:是否必需: No
:默认值: ``1``


``immutable_object_cache_qos_bps_limit``

:描述: 不可变对象缓存 IO 期望的每秒的字节数限额。
:类型: Unsigned Integer
:是否必需: No
:默认值: ``0``


``immutable_object_cache_qos_bps_burst``

:描述: 不可变对象缓存 IO 允许的字节数爆发量限额。
:类型: Unsigned Integer
:是否必需: No
:默认值: ``0``


``immutable_object_cache_qos_bps_burst_seconds``

:描述: 不可变对象缓存 IO 允许的爆发秒数。
:类型: Seconds
:是否必需: No
:默认值: ``1``

.. _克隆的 RBD 映像: ../rbd-snapshot/#layering
.. _段: ../../rados/configuration/ceph-conf/#configuration-sections
.. _新建一个 Ceph 用户: ../../rados/operations/user-management#add-a-user

