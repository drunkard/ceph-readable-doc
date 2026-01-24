==========
 故障排除
==========

慢或卡住的操作
==============
.. Slow/stuck operations

有时候 CephFS 的操作会卡住，解决这种问题首先要定位导致卡住的源头：
源头可能是这三个：

#. 在客户端这边
#. MDS
#. 连接客户端和 MDS 的网络。

首先，按照 :ref:`slow_requests` 里的步骤，检查客户端有没有卡住的操作、
或者 MDS 有没有卡住的操作。

转储（ dump ）出 MDS 的缓存。 MDS 缓存的内容将用于诊断问题的性质。
执行下列命令来转储 MDS 缓存：

.. prompt:: bash #

   ceph daemon mds.<name> dump cache /tmp/dump.txt

.. note:: 不受 systemd 控制的 MDS 服务会把文件
   ``dump.txt`` 转储到运行 MDS 的机器上。
   受 systemd 控制的 MDS 服务会把文件
   ``dump.txt`` 转储到 MDS 容器中的 tmpfs 中。
   用 `nsenter(1)` 查找 ``dump.txt`` 或指定另一个系统级的路径。

如果 MDS 设置了较高的日志级别，那么 ``dump.txt`` 差不多肯定有我们需要的信息，
能够诊断和解决导致 CephFS 操作卡住的问题。


.. _slow_requests:

慢请求（ MDS ）
---------------
.. Slow requests (MDS)

用管理套接字罗列出当前的操作，在 MDS 主机上执行下列命令：

.. prompt:: bash #

   ceph daemon mds.<name> dump_ops_in_flight

找到卡住的命令，并检查它们为何被卡住。
通常，最后一个 "event （事件）" 可能是尝试获取锁，
或者是正在将操作发送到 MDS 日志。如果它正在等待 OSD ，修复它们即可。

如果操作卡在某一个索引节点（ inode ）上，那么很可能是客户端持有某些能力，
耽搁了其他客户端使用该节点。
这种情况可能是由于客户端尝试刷回脏数据造成的，
也可能是由于您在 CephFS 的分布式文件锁代码\
（文件 "capabilities （能力）"， ["caps"] 系统）中遇到了缺陷。

如果您能确定那个命令是由于能力代码中的缺陷而卡住的，
重启 MDS 即可。重启那个 MDS 大概就能解决问题。

如果 MDS 上没有报告任何慢请求，并且没有迹象表明客户端行为异常，
那么要么是客户端本身存在问题，要么是客户端的请求无法到达 MDS 。


.. _cephfs_dr_stuck_during_recovery:

恢复期间卡住了
==============
.. Stuck during recovery

卡在 up:replay
--------------
.. Stuck in up:replay

如果您的 MDS 卡在 ``up:replay`` 状态，说明日志可能很长。
``MDS_HEALTH_TRIM`` 集群警告的存在就说明 MDS
尚未修剪完自己的日志，跟不上集群的当前状态。
如果日志过于长，可能需要数小时来处理。
虽然无法解决这个问题，但还是有办法加快速度的： 

临时禁用 MDS 调试日志，把 MDS 调试级别降低到 ``0`` 。
即使在默认设置下， MDS 也会把一些信息记录到内存中，
以便在遇到致命错误时转储。
你可以关闭所有日志记录，执行下列命令：

.. code:: bash

   ceph config set mds debug_mds 0
   ceph config set mds debug_ms 0
   ceph config set mds debug_monc 0

注意，你把 ``debug_mds`` 、 ``debug_ms`` 、和 ``debug_monc``
设置成 ``0`` 之后，如果 MDS 发生故障，
则几乎没有任何调试信息用以确定致命错误发生的原因。
如果你能算出 ``up:replay`` 完成的时间，
应该在进入下一状态前恢复这些配置： 

.. code:: bash

   ceph config rm mds debug_mds
   ceph config rm mds debug_ms
   ceph config rm mds debug_monc

重放速度加快后，计算一下 MDS 的完成时间。
可以检查日志重放状态： 

.. code:: bash

   $ ceph tell mds.<fs_name>:0 status | jq .replay_status
   {
     "journal_read_pos": 4195244,
     "journal_write_pos": 4195244,
     "journal_expire_pos": 4194304,
     "num_events": 2,
     "num_segments": 2
   }

当 ``journal_read_pos`` 追上 ``journal_write_pos`` 时，
重放就完成了。重放期间，写入位置不会改变。
跟踪读取位置的进展，就可以计算出预计完成的时间。
当日志重放操作耗时超过 30 秒时， MDS 会发一个 `MDS_ESTIMATED_REPLAY_TIME` 警告。
警告消息中包含日志重放的预计完成时间： ::

  mds.a(mds.0): replay: 50.0446% complete - elapsed time: 582s, estimated time remaining: 581s


.. _cephfs_troubleshooting_avoiding_recovery_roadblocks:

避免恢复障碍
------------
.. Avoiding recovery roadblocks

在恢复文件系统时，需要做以下几件事： 

* **拒绝所有到客户端的重连。** 屏蔽所有现存的 CephFS 会话，
  会导致所有挂载都将挂起或不可用。

  .. prompt:: bash #

     ceph config set mds mds_deny_all_reconnect true

  记着在所有 MDS 守护进程都活跃后撤销此操作。

  .. note:: 此操作不会阻止新会话连接。
     想要那个效果的话，用文件系统的 ``refuse_client_session`` 配置选项。

* **延长 MDS 心跳宽限期。** 这样可以避免\
  更换正在执行某些操作时看起来像 “卡死” 的 MDS 。
  有时，一个 MDS 在恢复时的某个操作可能比预期的时间要长
  （从程序员的角度来看）。当恢复所需的时间已经超过正常时间时，
  就更有可能出现这种情况（要不您怎么会读本文档呢）。
  通过延长心跳宽限期可以避免不必要的替换死循环，
  执行下列命令： 

   .. prompt:: bash #

      ceph config set mds mds_heartbeat_grace 3600

  .. note:: 这样做的效果是，即使 MDS 的内部 “心跳（ heartbeat ）” 机制\
     在一小时内没有重置（跳动），它也会继续向监视器发送信标。
     以前实现这一功能的机制是通过 ``mds_beacon_grace`` 监视器选项。

* **禁用打开文件表（ open-file-table ）的预取功能。** 通常，
  MDS 会在恢复期间预取目录内容，以预热缓存。
  在长时间恢复过程中，缓存可能已经很热\ **而且很大**\ 。
  如果缓存已经够热且巨大，这样的预取功能就没必要、
  且与预期效果不相符。
  执行下列命令禁用 open-file-table 预取功能：

  .. prompt:: bash #

     ceph config set mds mds_oft_prefetch_dirfrags false

* **关闭客户端。** 客户端重新连接到刚刚变为 ``up:active`` 状态的 MDS 时，
  可能会给刚恢复正常的文件系统带来新的负载。
  通常需要一些维护，然后才能允许客户端\
  连接到文件系统、并恢复到正常工作载荷。
  例如，如果恢复时间过长是因为重放读取的日志过大，
  那么加快日志的修剪可能是明智之举。

  可以手动或用 ``refuse_client_session`` 可调选项拒绝客户端会话，
  比如下面的例子： 

  .. prompt:: bash #

     ceph fs set <fs_name> refuse_client_session true

  此命令能防止客户端和 MDS
  建立新会话。

* **不要调整 max_mds** 在故障排除或恢复过程中，
  更改文件系统的配置变量 ``max_mds`` 看似是个好主意，
  但实际上未必。
  更改 ``max_mds`` 可能会进一步破坏集群的稳定性。
  如果形势所迫，必须更改 ``max_mds`` ，应该执行命令更改 ``max_mds`` ，
  还要加上确认标记（ ``--yes-i-really-mean-it`` ）。

.. _pause-purge-threads:

* **关闭异步清除线程** volumes 插件会派生线程，
  用于异步清除被销毁/删除的子卷。
  在故障排除或恢复期间，可以禁用这些清除线程，
  执行下列命令：

  .. prompt:: bash #

     ceph config set mgr mgr/volumes/pause_purging true

  恢复清除功能，执行下列命令：

  .. prompt:: bash #

     ceph config set mgr mgr/volumes/pause_purging false

.. _pause-clone-threads:

* **关闭异步克隆线程** volumes 插件会派生用于异步克隆子卷快照的线程。
  在排除故障或恢复期间，可以禁用这些克隆线程，
  执行下列命令： 

  .. prompt:: bash #

     ceph config set mgr mgr/volumes/pause_cloning true

  恢复克隆功能，执行下列命令： ::

    ceph config set mgr mgr/volumes/pause_cloning false



加快 MDS 日志修剪
=================
.. Expediting MDS journal trim

``MDS_HEALTH_TRIM`` 警告说明： MDS 日志已经过于大了。
当 MDS 日志积攒得太大后，可以用 ``mds_tick_interval``
可调选项来更改 "MDS tick interval" （ MDS 计时间隔）。
tick 计时间隔驱动着 MDS 内部的各种维护活动，
修改这个计时间隔可降低 MDS 日志的尺寸，
因为它会导致日志更频繁地修剪。

建议在文件系统负载不太大的时候修改 ``mds_tick_interval`` 。
降低 CephFS 负载有多种方法，
见 :ref:`cephfs_troubleshooting_avoiding_recovery_roadblocks` 。

此配置只影响处于 ``up:active`` 状态的 MDS 。
恢复期间， MDS 不会修剪日志。

执行下列命令来修改 ``mds_tick_interval`` 可调选项：

.. prompt:: bash #

   ceph config set mds mds_tick_interval 2


RADOS 健康状况
==============
.. RADOS Health

如果只是 CephFS 的元数据或者数据存储池的某一部分不可用、
且 CephFS 不响应，很有可能是 RADOS 本身有问题。

应该先解决 RADOS 问题，然后再着手定位 CephFS 的问题。
见 :ref:`RADOS 故障排除文档 <rados_troubleshooting>` 。


MDS 问题
========
.. The MDS

执行 ``ceph health`` 命令，如果某个操作卡在了 MDS 内部，
会出现 ``slow requests are blocked`` 消息。

消息是 ``failing to respond`` 的话，表明是客户端未能响应。

下面详细罗列了操作卡顿的潜在起因：

#. 系统过载。导致系统过载最可能的起因是遇到了\
   一个比 MDS 缓存还大的活跃文件集。

   如果你还有空闲内存，增大 ``mds_cache_memory_limit`` 配置试试。
   这个 ``mds_cache_memory_limit`` 可调选项在 :ref:`MDS 缓存尺寸
   <cephfs_cache_configuration_mds_cache_memory_limit>` 里详细论述了。
   在对 ``mds_cache_memory_limit`` 做任何调整前，先完整地阅读一下
   :ref:`MDS 缓存配置 <cephfs_mds_cache_configuration>` 这一段。

#. 有个老旧的（行为乖张）客户端；

#. 底层的 RADOS 问题。见
   :ref:`RADOS 故障排除文档 <rados_troubleshooting>` 。

除此之外，你也许遇到了新的软件缺陷，应该报告给开发者！


.. _ceph_fuse_debugging:

ceph-fuse 的调试
================
.. ceph-fuse debugging

ceph-fuse 是 CephFS 内核驱动的一个替代，它在用户空间挂载 CephFS 文件系统。
ceph-fuse 也支持 ``dump_ops_in_flight`` 功能。
执行下列命令来转储出正在进行的 ceph-fuse 操作，以便检查：

..
  .. prompt:: bash #

  the command goes here - 10 Aug 2025

见文档 :ref:`用 FUSE 挂载 CephFS <cephfs_mount_using_fuse>` 。


调试输出
--------
.. Debug output

要想看到 ceph-fuse 的更多调试信息，试试在前台运行，让日志输出到\
控制台（ ``-d`` ）、打开客户端调试（ ``--debug-client=20`` ）、\
即时打印发送过的每条消息（ ``--debug-ms=1`` ）。

.. prompt:: bash #

   ceph daemon -d mds.<name> dump_ops_in_flight --debug-client=20 --debug-ms=1

如果你怀疑是监视器的问题，也可以打开监视器调试开关（ ``--debug-monc=20`` ），
执行下列命令：

.. prompt:: bash #

   ceph daemon -d mds.<name> dump_ops_in_flight --debug-client=20 --debug-ms=1 --debug-monc=20


.. _kernel_mount_debugging:

内核挂载的调试
==============
.. Kernel mount debugging

如果内核客户端有问题，诊断并修复的第一步是找出问题起因，是在内核客户端上、
还是在 MDS 上。如果内核客户端自身出现故障，其故障证据位于内核环形缓冲区里，
可以执行下列命令来检查：

.. prompt:: bash #

   dmesg

找到相关的内核状态。


慢请求
------
.. Slow requests

遗憾的是，内核客户端不支持管理套接字。可是如果你客户端的内核启用了 `debugfs
<https://docs.kernel.org/filesystems/debugfs.html>`_ ，
那么它就有相似管理套接字的接口了。

找到 ``sys/kernel/debug/ceph/`` 路径下的一个文件夹，其名字形如
``28f7427e-5558-4ffd-ae1a-51ec3042759a.client25386880`` 。
这个文件夹下的文件可以用于诊断慢请求的起因，用 ``cat`` 命令查看它们的内容。

这些文件描述如下，最有助于诊断慢请求问题的文件是 ``mdsc`` （ MDS 的当前请求）和
``osdc`` （当前正在对 OSD 进行的操作）。

* ``bdi``: 关于 Ceph 系统的 BDI 信息（脏块、已写入的等等）
* ``caps``: 文件的 caps 数据结构计数，包括内存中的和用过的
* ``client_options``: 转储挂载 CephFS 时所用的选项
* ``dentry_lru``: 转储当前内存中的 CephFS dentry
* ``mdsc``: 转储当前发给 MDS 的请求
* ``mdsmap``: 转储当前的 MDSMap 时间结和所有 MDS
* ``mds_sessions``: 转储当前与 MDS 建立的会话
* ``monc``: 转储当前从监视器获取的各种映射图，以及其它“订阅”信息
* ``monmap``: 转储当前的监视器图和所有监视器
* ``osdc``: 转储当前发往 OSD 的操作（即文件数据的 IO ）
* ``osdmap``: 转储当前的 OSDMap 时间结、存储池、所有 OSD

如果数据存储池处于 ``NEARFULL`` 状态，那么内核 CephFS 客户端\
将会切换到同步写，同步写操作非常慢。


断线后又重新挂载的文件系统
==========================
.. Disconnected+Remounted FS

因为 CephFS 有个“一致性缓存 (consistent cache)”，
如果客户端的网络连接中断时间较长， MDS 就会强制驱逐（并加进黑名单）客户端。
此时，内核客户端不能安全地写回脏数据（缓冲的），而且这样会导致数据丢失。
然而：注意一下，这种行为也是合理的、且遵循 POSIX 语义。
这个客户端必须重新挂载才能再次访问那个文件系统，这是默认行为，
但可以用 ``recover_session`` 挂载选项覆盖。见 `mount.ceph 手册页的 options 段落
<https://docs.ceph.com/en/latest/man/8/mount.ceph/#options>`_ 。

如果 ``dmesg`` 里出现了类似消息，说明你遇到的就是这种情况： ::

    [Fri Aug 15 02:38:10 2025] ceph: mds0 caps stale
    [Fri Aug 15 02:38:28 2025] libceph: mds0 (2)XXX.XX.XX.XX :6800 socket closed (con state OPEN)
    [Fri Aug 15 02:38:28 2025] libceph: mds0 (2)XXX.XX.XX.XX:6800 session reset
    [Fri Aug 15 02:38:28 2025] ceph: mds0 closed our session
    [Fri Aug 15 02:38:28 2025] ceph: mds0 reconnect start
    [Fri Aug 15 02:38:28 2025] ceph: mds0 reconnect denied


挂载问题
========

挂载错误 5
----------
.. Mount 5 Error

``mount 5`` 错误通常是 MDS 服务器滞后或崩溃导致的。

要确保至少有一个 MDS 是启动且在运行，集群也要处于 ``active+healthy`` 状态。

挂载错误 12
-----------
.. Mount 12 Error

mount 12 错误显示 ``cannot allocate memory`` ，说明 :term:`Ceph 客户端`\ 和
:term:`Ceph 存储集群`\ 版本不匹配。用以下命令检查版本：

.. prompt:: bash #

   ceph -v

如果 Ceph 客户端版本落后于集群，升级它：

.. prompt:: bash #

   sudo apt-get update && sudo apt-get install ceph-common 

如果这样没能解决问题，那就 uninstall 、 autoclean 、再 autoremove 掉
``ceph-common`` 软件包，然后再重新安装，以确保安装的是最新版。


动态调试
========
.. Dynamic Debugging

CephFS 内核驱动程序的动态调试功能允许启用或禁用调试日志记录。
内核驱动程序的日志会写入内核环形缓冲区，并且可以用 ``dmesg(1)`` 工具查看。
默认情况下，调试日志记录是禁用的，因为启用调试日志记录\
可能会导致系统运行缓慢和 I/O 吞吐量下降。

对 CephFS 模块开启动态调试。

请看： https://github.com/ceph/ceph/blob/master/src/script/kcon_all.sh

注意：运行上述脚本可启用 CephFS 内核驱动、 libceph 和\
内核 RBD 模块的调试日志记录。如果要打开指定组件
（例如， CephFS 内核驱动）的调试日志记录，执行下列命令：

.. prompt:: bash #

   echo 'module ceph +p' > /sys/kernel/debug/dynamic_debug/control

要禁用调试日志记录，执行下列命令：

.. prompt:: bash #

   echo 'module ceph -p' > /sys/kernel/debug/dynamic_debug/control


转储内存中的日志
================
.. In-memory Log Dump

日志级别设置得低于 ``10`` 时，可以设置
``mds_extraordinary_events_dump_interval`` 来转储内存日志。
``mds_extraordinary_events_dump_interval`` 是指发生异常事件时，
转储最近内存日志的时间间隔，以秒为单位。

异常事件包括：

* 客户端驱逐
* 未收到监视器发来的信标 ACK
* 内部心跳未收到

内存日志转储（ In-memory Log Dump ）默认是禁用的，以防生产环境中日志文件膨胀。

执行下列两个命令来启用内存日志转储：

#. 
   .. prompt:: bash #

      ceph config set mds debug_mds <log_level>/<gather_level>

   ``log_level`` 要设置得低于 ``10`` 。把 ``gather_level`` 设置得大于 ``10`` 。
   这两个值设置好后，就启用了内存日志转储。
#. 
   .. prompt:: bash #

      ceph config set mds mds_extraordinary_events_dump_interval <seconds>

   启用后， MDS 会每隔 ``mds_extraordinary_events_dump_interval`` 秒\
   检查一次异常事件。如果有异常事件发生，
   MDS 会把包含着相关事件细节的内存日志转储到 Ceph MDS 日志中。

.. note:: 设置了较高的日志级别（ ``log_level`` >= ``10`` ）后，
   就没必要转储内存日志。较低的收集级别（ ``gather_level`` < ``10`` ）
   不足以收集内存日志。也就是说，在 ``debug_mds`` 中，
   日志级别大于等于 ``10`` 或收集级别小于 ``10`` 的时候就无法启用内存日志转储。
   在这种情况下，出现故障时，必须把 ``mds_extraordinary_events_dump_interval``
   的值重置为 ``0`` ，然后再用上述命令启用。

禁用内存日志转储可以执行下列命令：

.. prompt:: bash #

   ceph config set mds mds_extraordinary_events_dump_interval 0


升级后无法访问文件系统
======================
.. Filesystems Become Inaccessible After an Upgrade

.. note::
   在升级前运行此程序，可以避免出现 ``operation not permitted``
   （不允许操作）的错误。截至 2023 年 5 月，
   Nautilus （含）之后的升级似乎会出现这里讨论的那种
   ``operation not permitted`` 错误。

**如果**

您的 CephFS 文件系统中的数据存储池和元数据存储池是用
``ceph fs new`` 命令创建的
（这意味着它们不是用默认值创建的）。

**或者**

您已经有一个 CephFS 文件系统，并且正在升级到新的、
高于 Nautilus 的 Ceph 主版本 

**那么**

为了使文档中的 ``ceph fs authorize...`` 命令按文档所述的行为运行
（并避免除 ``client.admin`` 用户外的所有用户们在做文件 I/O 时出现
'operation not permitted' 错误、或类似的安全相关问题），
您必须首先运行：

.. prompt:: bash $

   ceph osd pool application set <your metadata pool name> cephfs metadata <your ceph fs filesystem name>

和

.. prompt:: bash $

   ceph osd pool application set <your data pool name> cephfs data <your ceph fs filesystem name>

否则，当 OSD 收到读取或写入数据
（不是目录信息，而是文件数据）的请求时，
它们将不知道要查找哪个 Ceph 文件系统名字。
存储池名字也是如此，因为主版本的“默认值”本身发生了变化，从::

   data pool=fsname
   metadata pool=fsname_metadata

变成了： ::

   data pool=fsname.data and
   metadata pool=fsname.meta

所有用 ``client.admin`` 进行挂载的都不会遇到这个问题，
因为 admin 密钥赋予了全部权限。

临时解决办法是把挂载请求改为 'client.admin' 用户及其密钥。
另一个不那么彻底但也能凑合的办法是把用户的 osd cap 改为
``caps osd = "allow rw"`` ，然后删除 ``tag cephfs data=....`` 。 


禁用 volumes 插件
=================
.. Disabling the Volumes Plugin

在某些情况下，可能需要禁用 Volumes 插件，以防止影响 Ceph 集群的其他部分。
详情见 :ref:`disabling-volumes-plugin` 。


报告问题
========
.. Reporting Issues

如果你确信发现了问题，报告时请附带尽可能多的信息，特别是重要信息：

* 客户端和服务器所安装的 Ceph 版本；
* 你在用内核、还是用户空间客户端；
* 如果你在用内核客户端，是什么版本？
* 有多少个客户端在用？什么样的负载？
* 如果某个系统“卡住”了，它影响所有客户端呢还是只影响一个？
* 关于 Ceph 的健康状况消息；
* 崩溃时写入日志的回调栈。

如果你觉得自己发现了一个缺陷，请在\ `缺陷追踪器`_\ 提交。\
一般问题的话可以发邮件到 `ceph-users 邮件列表`_\ 询问。

.. _缺陷追踪器: http://tracker.ceph.com
.. _ceph-users 邮件列表:  http://lists.ceph.com/listinfo.cgi/ceph-users-ceph.com/
