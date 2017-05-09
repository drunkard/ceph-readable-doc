==========
 RBD 镜像
==========

.. index:: Ceph Block Device; mirroring

RBD 映像可在两个 Ceph 集群间异步地镜像。此能力利用了 RBD 映像\
的 journaling 功能，以此保证集群间副本在崩溃后的一致性。镜像\
是以互联集群的存储池为单位配置的，可以配置为自动地镜像存储池\
内的所有映像、或者只镜像一部分。镜像可用 ``rbd`` 命令来配置， \
``rbd-mirror`` 守护进程负责拉取远端映像的变更、互联集群、以及\
把变更应用到本地集群。

.. note:: RBD 镜像功能在 Ceph Jewel 或更高版本才具备。

.. important:: 要使用 RBD 镜像功能，你必须有两个 Ceph 集群，\
   二者都要运行 ``rbd-mirror`` 守护进程。


存储池配置
==========

以下步骤演示了如何用 ``rbd`` 命令执行基本的管理任务，并配置\
镜像。镜像是以 Ceph 集群的存储池为单位配置的。

以下的存储池配置步骤在两个互联集群上都要执行一次。这些步骤\
假设有两个集群，为清晰起见，分别命名为 local 和 remote ，\
二者都可以从一台主机访问。

关于如何用 ``rbd`` 命令连接两个不同的集群，请参考其 `rbd`_
手册页。

.. note:: 以下实例中的集群名对应着同名的 Ceph 配置文件（如
   ``/etc/ceph/remote.conf`` ）。如何配置多个集群请参考
   `ceph-conf`_ 文档。


.. _Enable Mirroring:

启用镜像功能
------------

要用 ``rbd`` 命令启用存储池的镜像功能，可指定
``mirror pool enable`` 命令、存储池名字、和镜像模式： ::

        rbd mirror pool enable {pool-name} {mode}

其中，镜像模式可以是 ``pool`` 或 ``image`` ：

* **pool**: 配置为 ``pool`` 模式时，存储池内所有启用了
  journaling 功能的映像都会被镜像。
* **image**: 配置为 ``image`` 模式时，需\ `显式地开启`_\ 各个\
  镜像的镜像功能。

例如： ::

        rbd --cluster local mirror pool enable image-pool pool
        rbd --cluster remote mirror pool enable image-pool pool


.. _Disable Mirroring:

禁用镜像功能
------------

要用 ``rbd`` 命令禁用存储池的镜像功能，可指定
``mirror pool disable`` 命令和存储池名字： ::

        rbd mirror pool disable {pool-name}

用这个方法禁用一个存储池的镜像功能时，此存储池内所有明确启用\
了镜像功能的所有映像，其镜像功能都会被禁用。

例如： ::

        rbd --cluster local mirror pool disable image-pool
        rbd --cluster remote mirror pool disable image-pool


.. _Add Cluster Peer:

增加互联的集群
--------------

为使 ``rbd-mirror`` 守护进程发现它的互联集群，得让互联点注册\
到这个存储池。要用 ``rbd`` 新增一个镜像点 Ceph 集群，需指定
``mirror pool peer add`` 命令、存储池名字、以及集群连接方式： ::

        rbd mirror pool peer add {pool-name} {client-name}@{cluster-name}

例如： ::

        rbd --cluster local mirror pool peer add image-pool client.remote@remote
        rbd --cluster remote mirror pool peer add image-pool client.local@local


.. _Remove Cluster Peer:

删除互联的集群
--------------

要用 ``rbd`` 删除镜像点 Ceph 集群，可指定 ``mirror pool peer remove``
命令、以及互联点的 UUID （可用 ``rbd mirror pool info`` 命令\
找出）： ::

        rbd mirror pool peer remove {pool-name} {peer-uuid}

例如： ::

        rbd --cluster local mirror pool peer remove image-pool 55672766-c02b-4729-8567-f13a66893445
        rbd --cluster remote mirror pool peer remove image-pool 60c0e299-b38f-4234-91f6-eed0a367be08


.. _Image Configuration:

映像配置
========

不像存储池配置方式，映像配置只需要操作单个镜像点 Ceph 集群就行。

被镜像的 RBD 映像需指定为主、或非主的，这是映像的属性、不是存\
储池的。被指定为非主的映像不能被修改。

某一映像的镜像功能被开启时，它会被自动晋级为主映像（在存储池\
镜像模式为 **pool** 且映像开启了 journaling 映像功能时为隐式\
的，也可以用 ``rbd`` 命令\ `显式地开启`_\ ）。


.. _Enable Image Journaling Support:

开启映像的 journaling 支持
--------------------------

RBD 镜像用 journaling 功能来保证复制的映像始终保持崩溃一致\
性。一个映像能被镜像到互联集群前，必须先打开 journaling 功\
能。此功能可在创建映像时打开，即执行 ``rbd`` 命令时加上
``--image-feature exclusive-lock,journaling`` 选项。

另外，在已存在的 RBD 映像上也可以动态地开启 journaling 功能。\
要用 ``rbd`` 命令开启 journaling 功能可指定 ``feature enable``
命令、存储池名和映像名、以及功能名： ::

        rbd feature enable {pool-name}/{image-name} {feature-name}

例如： ::

        rbd --cluster local feature enable image-pool/image-1 journaling

.. note:: journaling 功能依赖于 exclusive-lock （互斥锁）功\
   能。如果 exclusive-lock 功能还没启用，应该先启用它、再启\
   用 journaling 功能。

.. tip:: 你可以让所有新映像默认启用日志功能，把
   ``rbd default features = 125`` 写入配置文件即可。


.. _Enable Image Mirroring:

启用基于映像的镜像
------------------

如果映像所在存储池的镜像功能配置成了 ``image`` 模式，那就得\
显式地启用各个映像的镜像功能。可以用 ``rbd`` 的
``mirror image enable`` 命令、再加上存储池和映像名： ::

        rbd mirror image enable {pool-name}/{image-name}

例如： ::

        rbd --cluster local mirror image enable image-pool/image-1


.. _Disable Image Mirroring:

禁用基于映像的镜像
------------------

要禁用某一映像的镜像功能，可用 ``rbd`` 、加
``mirror image disable`` 命令，再加上存储池名和映像名： ::

        rbd mirror image disable {pool-name}/{image-name}

例如： ::

        rbd --cluster local mirror image disable image-pool/image-1


.. _Image Promotion and Demotion:

映像的晋级和降级
----------------

.. note:: 译者注： promotion 翻译为晋级， demotion 翻译为降级。

在故障切换时，主映像标记要被挪到互联 Ceph 集群的对应映像上，\
到主映像的访问应该停止（例如关闭相应的 VM 、或者从 VM 里移除\
关联设备），降级当前的主映像，晋级新的主映像，然后在另一个集\
群上恢复访问。

.. note:: RBD 仅仅提供了实现故障有序切换所必需的工具集，你仍\
   需要一套外部机制来保障整个故障切换进程的顺利进行（例如降\
   级前先关闭映像）。

要用 ``rbd`` 命令把某一映像降级成非主的，用
``mirror image demote`` 命令，加上存储池名和映像名： ::

        rbd mirror image demote {pool-name}/{image-name}

例如： ::

        rbd --cluster local mirror image demote image-pool/image-1

要用 ``rbd`` 命令把存储池内的所有映像降级为非主的，可用
``mirror pool demote`` 命令，加上存储池名： ::

        rbd mirror pool demote {pool-name}

例如： ::

        rbd --cluster local mirror pool demote image-pool

要用 ``rbd`` 把某一映像晋级为主的，可用 ``mirror image promote``
命令、加存储池名和映像名： ::

        rbd mirror image promote [--force] {pool-name}/{image-name}

例如： ::

        rbd --cluster remote mirror image promote image-pool/image-1

要用 ``rbd`` 命令把存储池内的所有映像晋级为主的，可用
``mirror pool promote`` 命令，加上存储池名： ::

        rbd mirror pool promote [--force] {pool-name}

例如： ::

        rbd --cluster local mirror pool promote image-pool

.. tip:: 由于主、非主状态是基于单个映像的，所以有可能让两个\
   集群分摊 IO 负载、并实现故障切换、故障恢复。

.. note:: 晋级可用 ``--force`` 选项强制施行。在降级未能传达\
   到互联的 Ceph 集群时（例如 Ceph 集群故障，通讯中断），就\
   需要强制晋级。这会导致两个对等点形成裂脑（ split-brain ），\
   并且这两个映像无法回到同步状态，只能通过\ \
   `强制重新同步命令`_\ 恢复同步。


.. _Force Image Resync:

强制重新同步映像
----------------

如果 ``rbd-mirror`` 守护进程探测到了裂脑事件，它就不会再企图\
镜像受影响的映像，除非已纠正。要恢复一个映像的镜像，首先找出\
过期的映像、并\ `降级此映像`_\ ，然后向主映像发出一个重新同\
步的请求。要用 ``rbd`` 请求重新同步映像，可用
``mirror image resync`` 命令、加上存储池名和映像名： ::

        rbd mirror image resync {pool-name}/{image-name}

例如： ::

        rbd mirror image resync image-pool/image-1

.. note:: ``rbd`` 命令仅仅把这个映像标记为需要重新同步。本\
   地集群的 ``rbd-mirror`` 守护进程负责异步地重新同步。


.. _Mirror Status:

镜像状态
========

每一个主的、被镜像的映像都存储了互联集群的复制状态，这些状态\
信息可用 ``mirror image status`` 和 ``mirror pool status``
命令查看。

要用 ``rbd`` 查看映像的镜像状态，可用 ``mirror image status``
命令、加上存储池名、映像名： ::

        rbd mirror image status {pool-name}/{image-name}

例如： ::

        rbd mirror image status image-pool/image-1

要用 ``rbd`` 命令查看存储池的镜像汇总状态，可用
``mirror pool status`` 命令、加上存储池名： ::

        rbd mirror pool status {pool-name}

例如： ::

        rbd mirror pool status image-pool

.. note:: 给 ``mirror pool status`` 命令加上 ``--verbose``
   选项，它还会额外输出此存储池内每一个映像的镜像状态细节。


.. _rbd-mirror Daemon:

rbd-mirror 守护进程
===================

两边的 ``rbd-mirror`` 守护进程负责监视远端的、互联集群的映像\
日志，并在本地集群回放这些日志事件。 RBD 映像的 journaling 功\
能会在映像内按其发生顺序记录所有变更，这样可确保远端映像的崩\
溃一致镜像在本地可用。

``rbd-mirror`` 守护进程随可选的 ``rbd-mirror`` 发行版软件包\
提供。

.. important:: 每个 ``rbd-mirror`` 守护进程都要求能同时连接两\
   边的集群。
.. warning:: 每个 Ceph 集群只能运行一个 ``rbd-mirror`` 守护进\
   程。以后的 Ceph 版本会支持 ``rbd-mirror`` 守护进程的横向扩\
   展。

.. _rbd: ../../man/8/rbd
.. _ceph-conf: ../../rados/configuration/ceph-conf/#running-multiple-clusters
.. _显式地开启: #enable-image-mirroring
.. _强制重新同步命令: #force-image-resync
.. _降级此映像: #image-promotion-and-demotion

