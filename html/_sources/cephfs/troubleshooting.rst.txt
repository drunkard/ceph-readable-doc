==========
 故障排除
==========

.. _Slow/stuck operations:

慢或卡住的操作
==============

如果遇到了明显的卡顿操作，首先要定位问题源头：是客户端、
MDS 、抑或是连接二者的网络。从存在卡顿操作的地方下手（参考下面\
的 :ref:`slow_requests` ），进而缩小范围。

RADOS 健康状况
==============

如果 CephFS 的元数据或者数据存储池的某一部分不可用、且 CephFS
不响应，很有可能是 RADOS 本身有问题，应该先解决这样的问题（
:doc:`../../rados/troubleshooting/index` ）。

MDS 问题
========

如果某个操作卡在了 MDS 内部，类似 "slow requests are blocked"
的消息最终会出现在 ``ceph health`` 里，也可能指出是客户端的问\
题，如 "failing to respond" 或其它形式的异常行为。如果 MDS 认\
定某些客户端的行为异常，你应该弄明白起因。常见起因有：

#. 系统过载（如果你还有空闲内存，增大 ``mds cache size`` 配置\
   试试，默认才 100000 。活跃文件比较多，超过 MDS 缓存容量是此\
   问题的首要起因！）
#. 客户端比较老（行为异常），或者
#. 底层的 RADOS 问题。

除此之外，你也许遇到了新的软件缺陷，应该报告给开发者！


.. _slow_requests:

慢请求（ MDS 端）
-----------------

通过管理套接字，你可以罗列当前正在运行的操作： ::

        ceph daemon mds.<name> dump_ops_in_flight

在 MDS 主机上执行。找出卡住的命令、并调查卡住的原因。通常最后\
一个“事件”（ event ）尝试过收集锁、或者把这个操作发往 MDS 日\
志。如果它是在等待 OSD ，修正即可；如果操作都卡在了某个索引节\
点上，原因可能是一个客户端一直占着能力，别人就没法拿到，也可\
能是这个客户端想刷回脏数据，还可能是你遇到了 CephFS 分布式文\
件锁的代码（文件能力子系统、 capabilities 、 caps ）缺陷。

如果起因是能力代码的缺陷，重启 MDS 也许能解决此问题。

如果 MDS 上没发现慢请求，而且也没报告客户端行为异常，那问题就\
可能在客户端上、或者它的请求还没到 MDS 上。


.. _ceph-fuse debugging:

ceph-fuse 的调试
================

ceph-fuse 也支持 dump_ops_in_flight 命令，可以查看是否卡住、卡\
在哪里了。

调试输出
--------

要想看到 ceph-fuse 的更多调试信息，试试在前台运行，让日志输出到\
控制台（ ``-d`` ）、打开客户端调试（ ``--debug-client=20`` ）、\
打印发送过的所有消息（ ``--debug-ms=1`` ）。

如果你怀疑是监视器的问题，也可以打开监视器调试开关（
``--debug-monc=20`` ）。


.. _Kernel mount debugging:

内核挂载的调试
==============

慢请求
------

遗憾的是内核客户端不支持管理套接字，可是如果你的内核启用了（如\
果限制过） debugfs ，那么它就有相似的接口了。
``sys/kernel/debug/ceph/`` 路径下有一个文件夹（其名字形如
``28f7427e-5558-4ffd-ae1a-51ec3042759a.client25386880`` ），而\
且其内包含很多文件，用 ``cat`` 命令查看它们的内容时会看到有趣\
的输出。这些文件描述如下，最有助于调试慢请求问题的文件可能是
``mdsc`` 和 ``osdc`` 。

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

如果没有卡住的请求，却有毫无进展的文件 IO ，问题也许是……

.. _Disconnected+Remounted FS:

断线后又重新挂载的文件系统
==========================

因为 CephFS 有个“一致性缓存”，如果你的网络连接中断时间较长，客\
户端就会被系统强制断开，此时，内核客户端仍然傻站着（ in a bind
）：它不能安全地写回脏数据，另外很多应用程序在 close() 时不能\
正确处理 IO 错误。这个时候，内核客户端会重新挂载这个文件系统，\
但是先前的文件系统 IO 也许能完成、也许不能，这些情况下，你也许\
得重启客户端系统。

如果 dmesg 或者 kern.log 里出现了类似消息，说明你遇到的就是这\
种情况： ::

    Jul 20 08:14:38 teuthology kernel: [3677601.123718] ceph: mds0 closed our session
    Jul 20 08:14:38 teuthology kernel: [3677601.128019] ceph: mds0 reconnect start
    Jul 20 08:14:39 teuthology kernel: [3677602.093378] ceph: mds0 reconnect denied
    Jul 20 08:14:39 teuthology kernel: [3677602.098525] ceph:  dropping dirty+flushing Fw state for ffff8802dc150518 1099935956631
    Jul 20 08:14:39 teuthology kernel: [3677602.107145] ceph:  dropping dirty+flushing Fw state for ffff8801008e8518 1099935946707
    Jul 20 08:14:39 teuthology kernel: [3677602.196747] libceph: mds0 172.21.5.114:6812 socket closed (con state OPEN)
    Jul 20 08:14:40 teuthology kernel: [3677603.126214] libceph: mds0 172.21.5.114:6812 connection reset
    Jul 20 08:14:40 teuthology kernel: [3677603.132176] libceph: reset on mds0

这是正在改善的领域，内核将很快能够可靠地向正在进行的 IO 发送错\
误代码，即便你的应用程序不能良好地应对这些情况。长远来看，在不\
违背 POSIX 语义的情况下，我们希望可以重连和回收数据（通常是其\
它客户端尚未访问、或修改的数据）。


挂载问题
========

Mount 5 Error
-------------

mount 5 错误通常是 MDS 服务器滞后或崩溃导致的。要确保至少有一\
个 MDS 是启动且运行的，集群也要处于 ``active+healthy`` 状态。


Mount 12 Error
--------------

mount 12 错误显示 ``cannot allocate memory`` ，常见于
:term:`Ceph 客户端`\ 和 :term:`Ceph 存储集群`\ 版本不匹配。用\
以下命令检查版本： ::

	ceph -v

如果 Ceph 客户端版本落后于集群，试着升级它： ::

	sudo apt-get update && sudo apt-get install ceph-common 

你也许得卸载、清理和删除 ``ceph-common`` ，然后再重新安装，以\
确保安装的是最新版。
