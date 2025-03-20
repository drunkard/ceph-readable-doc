==========
 故障排除
==========

慢或卡住的操作
==============
.. Slow/stuck operations

如果遇到了明显的卡顿操作，首先要定位问题源头：是客户端、
MDS 、抑或是连接二者的网络。从存在卡顿操作的地方下手（参考下面\
的 :ref:`slow_requests` ），进而缩小范围。

我们可以这样获取线索，把 MDS 缓存转储出来看看发生了什么： ::

  ceph daemon mds.<name> dump cache /tmp/dump.txt

.. note:: `dump.txt` 这个文件在运行着 MDS 的机器上，而且，
   对于 systemd 控制着的 MDS 服务，它是在 MDS 容器内的一个 tmpfs 上。
   可以用 `nsenter(1)` 来定位 `dump.txt` ，或者指定另外一个系统级的路径。

如果 MDS 设置了较高的日志级别，那里差不多肯定有我们需要的诊断信息，
解决问题有望了。


恢复期间卡住了
==============
.. Stuck during recovery

卡在 up:replay
--------------
.. Stuck in up:replay

如果您的 MDS 卡在 ``up:replay`` 状态，说明日志可能很长。
您是否看到过 ``MDS_HEALTH_TRIM`` 集群警告，说 MDS 在修剪日志方面已经落后？
如果日志过于长，读取日志可能需要数小时。
虽然无法解决这个问题，但还是有办法加快速度的： 

把 MDS 调试级别降低到 0 。即使在默认设置下， MDS 也会把一些信息记录到内存中，
以便在遇到致命错误时转储。可以避免这种情况： 

.. code:: bash

   ceph config set mds debug_mds 0
   ceph config set mds debug_ms 0
   ceph config set mds debug_monc 0

注意，如果 MDS 发生故障，则几乎没有任何信息可以确定原因。
如果能算出 ``up:replay`` 完成的时间，
应该在进入下一状态前恢复这些配置： 

.. code:: bash

   ceph config rm mds debug_mds
   ceph config rm mds debug_ms
   ceph config rm mds debug_monc

重放速度加快后，就可以计算 MDS 的完成时间。可以检查日志重放状态来确定： 

.. code:: bash

   $ ceph tell mds.<fs_name>:0 status | jq .replay_status
   {
     "journal_read_pos": 4195244,
     "journal_write_pos": 4195244,
     "journal_expire_pos": 4194304,
     "num_events": 2,
     "num_segments": 2
   }

当 ``journal_read_pos`` 追上 ``journal_write_pos`` 时，重放就完成了。
重放期间，写入位置不会改变。
跟踪读取位置的进展，就可以计算出预计完成的时间。


避免恢复障碍
------------
.. Avoiding recovery roadblocks

在断电期间尝试紧急恢复文件系统时，需要做以下几件事： 

* **拒绝所有到客户端的重连。** 这将有效阻止所有已有的 CephFS 会话，
  因此所有挂载都将挂起或不可用。

.. code:: bash

   ceph config set mds mds_deny_all_reconnect true

  记着在所有 MDS 守护进程都活跃后撤销此操作。

.. note:: 此操作不会阻止新会话连接。想要那个效果的话，参阅文件系统的 ``refuse_clien t_session`` 设置。

* **延长 MDS 心跳宽限期。** 这样可以避免更换正在执行某些操作时看起来像 “卡死” 的 MDS 。
  有时，一个 MDS 在恢复时的某个操作可能比预期的时间要长
  （从程序员的角度来看）。当恢复所需的时间已经超过正常时间时，
  就更有可能出现这种情况（要不您怎么会读本文档呢）。
  通过延长心跳宽限期可以避免不必要的替换死循环： 

.. code:: bash

   ceph config set mds mds_heartbeat_grace 3600

.. note:: 这样做的效果是，即使 MDS 的内部 “心跳（ heartbeat ）” 机制\
   在一小时内没有重置（跳动），它也会继续向监视器发送信标。
   以前实现这一功能的机制是通过 ``mds_beacon_grace`` 监视器选项。

* **禁用打开文件表（ open file table ）的预取功能。** 通常，
  MDS 会在恢复期间预取目录内容，以预热缓存。
  在长时间恢复过程中，缓存可能已经很热\ **而且很大**\ 。
  因此，这种行为可能变得与预期效果不相符。这样禁用：

.. code:: bash

   ceph config set mds mds_oft_prefetch_dirfrags false

* **关闭客户端。** 客户端重新连接到刚刚变为 ``up:active`` 状态的 MDS 时，
  可能会给刚恢复正常的文件系统带来新的负载。
  在重新接受工作载荷之前，可能需要进行一些常规维护。
  例如，如果恢复时间过长是因为重放读取的日志过大，
  那么加快日志的修剪可能是明智之举。

  您可以手动或使用新的文件系统可调选项来完成： 

.. code:: bash

   ceph fs set <fs_name> refuse_client_session true

  它可以防止所有客户端和 MDS 建立新会话。

* **不要调整 max_mds** 在故障排除或恢复过程中，
  更改 FS 配置变量 ``max_mds`` 有时会被当作一个很好的步骤。
  相反，这样做可能会进一步破坏集群的稳定性。
  如果形势所迫，必须更改 ``max_mds`` ，应该执行命令更改 ``max_mds`` ，
  还要加上确认标记（ ``--yes-i-really-mean-it`` ）。

* **关闭异步清除线程** volumes 插件会产生线程，
  用于异步清除被销毁/删除的子卷。为帮助故障排除或恢复工作，
  可以用以下方法禁用这些清除线程： 

.. code:: bash

    ceph config set mgr mgr/volumes/pause_purging true

  恢复清除功能，执行： ::

    ceph config set mgr mgr/volumes/pause_purging false

* **关闭异步克隆线程** volumes 插件会产生用于异步克隆子卷快照的线程。
  为了帮助排除故障或进行恢复，
  可以用以下方法禁用这些克隆线程： 

.. code:: bash

    ceph config set mgr mgr/volumes/pause_cloning true

  恢复克隆功能，执行： ::

    ceph config set mgr mgr/volumes/pause_cloning false


加快 MDS 日志修剪
=================
.. Expediting MDS journal trim

如果您的 MDS 日志过大（也许您的 MDS 曾长时间卡在 up:replay 状态！），
就需要让 MDS 更频繁地修剪它的日志。
您可以通过 ``MDS_HEALTH_TRIM`` 警告来了解日志是否过大。

要做到这一点，可以调整的主要是修改 MDS 的 tick 间隔。
tick 时间间隔驱动着 MDS 中的多项维护活动。
强烈建议在文件系统负载不太大的时候修改 tick 间隔。
此配置只影响处于 ``up:active`` 状态的 MDS 。
恢复期间， MDS 不会修剪日志。

.. code:: bash

   ceph config set mds mds_tick_interval 2


RADOS 健康状况
==============
.. RADOS Health

如果 CephFS 的元数据或者数据存储池的某一部分不可用、且 CephFS 不响应，
很有可能是 RADOS 本身有问题，应该先解决这样的问题
（ :doc:`../../rados/troubleshooting/index` ）。


MDS 问题
========
.. The MDS

如果某个操作卡在了 MDS 内部，类似 "slow requests are blocked" 的消息\
最终会出现在 ``ceph health`` 里；也可能指出是客户端的问题，
如 "failing to respond" 或其它形式的异常行为。
如果 MDS 认定某些客户端的行为异常，你应该弄明白起因。常见起因有：

通常是这些起因：

#. 系统过载（如果你还有空闲内存，增大 ``mds cache memory limit`` 配置试试，
   默认才 1GiB ；活跃文件比较多，超过 MDS 缓存容量是此问题的首要起因！）

#. 客户端比较老（行为乖张）；

#. 底层的 RADOS 问题。

除此之外，你也许遇到了新的软件缺陷，应该报告给开发者！


.. _slow_requests:

慢请求（ MDS 端）
-----------------

通过管理套接字，你可以罗列当前正在运行的操作： ::

        ceph daemon mds.<name> dump_ops_in_flight

在 MDS 主机上执行。找出卡住的命令、并调查卡住的原因。
通常最后一个“事件”（ event ）尝试过收集锁、或者把这个操作发往 MDS 日志。
如果它是在等待 OSD ，修正即可；如果操作都卡在了某个索引节点上，
原因可能是一个客户端一直占着能力，别人就没法拿到，
也可能是这个客户端想刷回脏数据，还可能是你遇到了 CephFS 分布式文件锁的\
代码（文件能力子系统、 capabilities 、 caps ）缺陷。

如果起因是能力代码的缺陷，重启 MDS 也许能解决此问题。

如果 MDS 上没发现慢请求，而且也没报告客户端行为异常，
那问题就可能在客户端上、或者它的请求还没到 MDS 上。


.. _ceph_fuse_debugging:

ceph-fuse 的调试
================
.. ceph-fuse debugging

ceph-fuse 也支持 dump_ops_in_flight 命令，可以查看是否卡住、卡在哪里了。

调试输出
--------

要想看到 ceph-fuse 的更多调试信息，试试在前台运行，让日志输出到\
控制台（ ``-d`` ）、打开客户端调试（ ``--debug-client=20`` ）、\
打印发送过的所有消息（ ``--debug-ms=1`` ）。

如果你怀疑是监视器的问题，也可以打开监视器调试开关（ ``--debug-monc=20`` ）。


.. _kernel_mount_debugging:

内核挂载的调试
==============
.. Kernel mount debugging

如果内核客户端有问题，最重要的是找出问题起因是在内核客户端上、
还是在 MDS 上，通常都比较容易发现。如果内核客户端直接垮了，
``dmesg`` 里会有输出信息，要收集好它们、还有任何不对劲的内核状态。

慢请求
------
.. Slow requests

遗憾的是内核客户端不支持管理套接字，可是如果你的内核启用了
（如果限制过） debugfs ，那么它就有相似的接口了。
``sys/kernel/debug/ceph/`` 路径下有一个文件夹（其名字形如
``28f7427e-5558-4ffd-ae1a-51ec3042759a.client25386880`` ），
而且其内包含很多文件，用 ``cat`` 命令查看它们的内容时会看到有趣的输出。
这些文件描述如下，最有助于调试慢请求问题的文件可能是 ``mdsc`` 和 ``osdc`` 。

* bdi: 关于 Ceph 系统的 BDI 信息（脏块、已写入的等等）
* caps: 文件的 caps 数据结构计数，包括内存中的和用过的
* client_options: 倒出挂载 CephFS 时所用的选项
* dentry_lru: 倒出当前内存中的 CephFS dentry
* mdsc: 倒出当前发给 MDS 的请求
* mdsmap: 倒出当前的 MDSMap 时间结和所有 MDS
* mds_sessions: 倒出当前与 MDS 建立的会话
* monc: 倒出当前从监视器获取的各种映射图，以及其它“订阅”信息
* monmap: 倒出当前的监视器图和所有监视器
* osdc: 倒出当前发往 OSD 的操作（即文件数据的 IO ）
* osdmap: 倒出当前的 OSDMap 时间结、存储池、所有 OSD

如果数据存储池处于 NEARFULL 状态，那么内核 cephfs 客户端\
将会切换到同步写，此时会非常慢。


断线后又重新挂载的文件系统
==========================
.. Disconnected+Remounted FS

因为 CephFS 有个“一致性缓存”，如果你的网络连接中断时间较长，
客户端就会被系统强制断开，此时，内核客户端仍然傻站着（ in a bind ）：
它不能安全地写回脏数据，另外很多应用程序在 close() 时不能正确处理 IO 错误。
这个时候，内核客户端会重新挂载这个文件系统，
但是先前的文件系统 IO 也许能完成、也许不能，
这些情况下，你也许得重启客户端系统。

如果 dmesg 或者 kern.log 里出现了类似消息，说明你遇到的就是这种情况： ::

    Jul 20 08:14:38 teuthology kernel: [3677601.123718] ceph: mds0 closed our session
    Jul 20 08:14:38 teuthology kernel: [3677601.128019] ceph: mds0 reconnect start
    Jul 20 08:14:39 teuthology kernel: [3677602.093378] ceph: mds0 reconnect denied
    Jul 20 08:14:39 teuthology kernel: [3677602.098525] ceph:  dropping dirty+flushing Fw state for ffff8802dc150518 1099935956631
    Jul 20 08:14:39 teuthology kernel: [3677602.107145] ceph:  dropping dirty+flushing Fw state for ffff8801008e8518 1099935946707
    Jul 20 08:14:39 teuthology kernel: [3677602.196747] libceph: mds0 172.21.5.114:6812 socket closed (con state OPEN)
    Jul 20 08:14:40 teuthology kernel: [3677603.126214] libceph: mds0 172.21.5.114:6812 connection reset
    Jul 20 08:14:40 teuthology kernel: [3677603.132176] libceph: reset on mds0

这是正在改善的领域，内核将很快能够可靠地向正在进行的 IO 发送错误代码，
即便你的应用程序不能良好地应对这些情况。长远来看，在不违背 POSIX 语义的情况下，
我们希望可以重连和回收数据（通常是其它客户端尚未访问、或修改的数据）。


挂载问题
========

挂载错误 5
----------
.. Mount 5 Error

mount 5 错误通常是 MDS 服务器滞后或崩溃导致的。
要确保至少有一个 MDS 是启动且运行的，集群也要处于 ``active+healthy`` 状态。

挂载错误 12
-----------
.. Mount 12 Error

mount 12 错误显示 ``cannot allocate memory`` ，常见于 :term:`Ceph 客户端`\ 和
:term:`Ceph 存储集群`\ 版本不匹配。用以下命令检查版本： ::

	ceph -v

如果 Ceph 客户端版本落后于集群，试着升级它： ::

	sudo apt-get update && sudo apt-get install ceph-common 

你也许得卸载、清理和删除 ``ceph-common`` ，然后再重新安装，以确保安装的是最新版。


动态调试
========
.. Dynamic Debugging

你可以对 CephFS 模块开启动态调试。

请看： https://github.com/ceph/ceph/blob/master/src/script/kcon_all.sh


转储内存中的日志
================
.. In-memory Log Dump

在较低级别的调试过程中（日志级别 < 10 ），可以设置
``mds_extraordinary_events_dump_interval`` 来转储内存日志。
``mds_extraordinary_events_dump_interval`` 是指，当发生非正常
（ Extra-Ordinary ）事件时，转储最近内存日志的时间间隔（以秒为单位）。

非正常事件分为以下几类：

* 客户端驱逐
* 未收到监视器发来的信标 ACK
* 内部心跳未收到

内存日志转储（ In-memory Log Dump ）默认是禁用的，以防生产环境中日志文件膨胀。
以下命令可依次启用它： ::

  $ ceph config set mds debug_mds <log_level>/<gather_level>
  $ ceph config set mds mds_extraordinary_events_dump_interval <seconds>

要启用内存日志转储， ``log_level`` 应小于 10 ， ``gather_level`` 应该 >= 10 。
启用后， MDS 会每隔 ``mds_extraordinary_events_dump_interval`` 秒\
检查一次异常事件，如果有异常事件发生，
MDS 会把包含着相关事件细节的内存日志转储到 ceph-mds 日志中。

.. note:: 对于较高的日志级别（ log_level >= 10 ），没必要转储内存日志，
   而较低的收集级别（ gather_level < 10 ）又不足以收集内存日志。
   因此， debug_mds 中，日志级别 >=10 或收集级别 < 10 会阻止启用内存日志转储。
   在这种情况下，出现故障时，需要把 mds_extraordinary_events_dump_interval
   的值重置为 0 ，然后再用上述命令启用。

内存日志转储可以这样禁用： ::

  $ ceph config set mds mds_extraordinary_events_dump_interval 0


升级后无法访问文件系统
======================
.. Filesystems Become Inaccessible After an Upgrade

.. note::
   在升级前运行此程序，可以避免出现 ``operation not permitted``
   （不允许操作）的错误。截至 2023 年 5 月， Nautilus （含）之后的升级\
   似乎会出现这里讨论的那种 ``operation not permitted`` 错误。

**如果**

您的 CephFS 文件系统中的数据存储池和元数据存储池是用 ``ceph fs new`` 命令创建的
（这意味着它们不是用默认值创建的）。

**或者**

您已经有一个 CephFS 文件系统，并且正在升级到新的、高于 Nautilus 的 Ceph 主版本 

**那么**

为了使文档中的 ``ceph fs authorize...`` 命令按文档所述的行为运行
（并避免除 ``client.admin`` 用户外的所有用户们在做文件 I/O 时出现
'operation not permitted' 错误、或类似的安全相关问题），您必须首先运行：

.. prompt:: bash $

   ceph osd pool application set <your metadata pool name> cephfs metadata <your ceph fs filesystem name>

和

.. prompt:: bash $

   ceph osd pool application set <your data pool name> cephfs data <your ceph fs filesystem name>

否则，当 OSD 收到读取或写入数据（不是目录信息，而是文件数据）的请求时，
它们将不知道要查找哪个 Ceph 文件系统名字。
存储池名字也是如此，因为主要版本的“默认值”本身发生了变化，从::

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
