==========
 RBD 镜像
==========

.. index:: Ceph Block Device; mirroring

RBD 映像可在两个 Ceph 集群间异步地镜像。此能力有两种模式可用：

* **Journal-based**: This mode uses the RBD journaling image feature to ensure
  point-in-time, crash-consistent replication between clusters. Every write to
  the RBD image is first recorded to the associated journal before modifying the
  actual image. The remote cluster will read from this associated journal and
  replay the updates to its local copy of the image. Since each write to the
  RBD image will result in two writes to the Ceph cluster, expect write
  latencies to nearly double when using the RBD journaling image feature.

* **Snapshot-based**: This mode uses periodically scheduled or manually
  created RBD image mirror-snapshots to replicate crash-consistent RBD images
  between clusters. The remote cluster will determine any data or metadata
  updates between two mirror-snapshots and copy the deltas to its local copy of
  the image. With the help of the RBD fast-diff image feature, updated data
  blocks can be quickly computed without the need to scan the full RBD image.
  Since this mode is not as fine-grained as journaling, the complete delta 
  between two snapshots will need to be synced prior to use during a failover
  scenario. Any partially applied set of deltas will be rolled back at moment
  of failover.

.. note:: journal-based mirroring requires the Ceph Jewel release or later;
   snapshot-based mirroring requires the Ceph Octopus release or later.

Mirroring is configured on a per-pool basis within peer clusters and can be
configured on a specific subset of images within the pool or configured to
automatically mirror all images within a pool when using journal-based
mirroring only. Mirroring is configured using the ``rbd`` command. The
``rbd-mirror`` daemon is responsible for pulling image updates from the remote,
peer cluster and applying them to the image within the local cluster.

Depending on the desired needs for replication, RBD mirroring can be configured
for either one- or two-way replication:

* **One-way Replication**: When data is only mirrored from a primary cluster to
  a secondary cluster, the ``rbd-mirror`` daemon runs only on the secondary
  cluster.

* **Two-way Replication**: When data is mirrored from primary images on one
  cluster to non-primary images on another cluster (and vice-versa), the
  ``rbd-mirror`` daemon runs on both clusters.

.. important:: Each instance of the ``rbd-mirror`` daemon must be able to
   connect to both the local and remote Ceph clusters simultaneously (i.e.
   all monitor and OSD hosts). Additionally, the network must have sufficient
   bandwidth between the two data centers to handle mirroring workload.


.. Pool Configuration

存储池配置
==========

以下步骤演示了如何用 ``rbd`` 命令执行基本的管理任务，并配置\
镜像。镜像是以 Ceph 集群的存储池为单位配置的。

以下的存储池配置步骤在两个互联集群上都要执行一次。这些步骤\
假设有两个集群，为清晰起见，分别命名为 local 和 remote ，\
二者都可以从一台主机访问。

关于如何连接到不同的 Ceph 集群，请参考 `rbd`_ 手册页。

.. note:: 以下实例中的集群名对应着同名的 Ceph 配置文件（如
   ``/etc/ceph/site-b.conf`` ）。如何配置多个集群请参考
   `ceph-conf`_ 文档。


.. Enable Mirroring

启用镜像功能
------------

要用 ``rbd`` 命令启用存储池的镜像功能，可指定
``mirror pool enable`` 命令、存储池名字、和镜像模式： ::

        rbd mirror pool enable {pool-name} {mode}

其中，镜像模式可以是 ``image`` 或 ``pool`` ：

* **image**: 配置为 ``image`` 模式时，需\ `显式地开启`_\ 各个\
  镜像的镜像功能。
* **pool**: 配置为 ``pool`` 模式时，存储池内所有启用了
  journaling 功能的映像都会被镜像。

例如： ::

        $ rbd --cluster site-a mirror pool enable image-pool image
        $ rbd --cluster site-b mirror pool enable image-pool image


.. Disable Mirroring

禁用镜像功能
------------

要用 ``rbd`` 命令禁用存储池的镜像功能，可指定
``mirror pool disable`` 命令和存储池名字： ::

        rbd mirror pool disable {pool-name}

用这个方法禁用一个存储池的镜像功能时，此存储池内所有明确启用\
了镜像功能的所有映像，其镜像功能都会被禁用。

例如： ::

        $ rbd --cluster site-a mirror pool disable image-pool
        $ rbd --cluster site-b mirror pool disable image-pool


Bootstrap Peers
---------------

In order for the ``rbd-mirror`` daemon to discover its peer cluster, the peer
needs to be registered to the pool and a user account needs to be created.
This process can be automated with ``rbd`` and the
``mirror pool peer bootstrap create`` and ``mirror pool peer bootstrap import``
commands.

To manually create a new bootstrap token with ``rbd``, specify the
``mirror pool peer bootstrap create`` command, a pool name, along with an
optional friendly site name to describe the local cluster::

        rbd mirror pool peer bootstrap create [--site-name {local-site-name}] {pool-name}

The output of ``mirror pool peer bootstrap create`` will be a token that should
be provided to the ``mirror pool peer bootstrap import`` command. For example,
on site-a::

        $ rbd --cluster site-a mirror pool peer bootstrap create --site-name site-a image-pool
        eyJmc2lkIjoiOWY1MjgyZGItYjg5OS00NTk2LTgwOTgtMzIwYzFmYzM5NmYzIiwiY2xpZW50X2lkIjoicmJkLW1pcnJvci1wZWVyIiwia2V5IjoiQVFBUnczOWQwdkhvQmhBQVlMM1I4RmR5dHNJQU50bkFTZ0lOTVE9PSIsIm1vbl9ob3N0IjoiW3YyOjE5Mi4xNjguMS4zOjY4MjAsdjE6MTkyLjE2OC4xLjM6NjgyMV0ifQ==

To manually import the bootstrap token created by another cluster with ``rbd``,
specify the ``mirror pool peer bootstrap import`` command, the pool name, a file
path to the created token (or '-' to read from standard input), along with an
optional friendly site name to describe the local cluster and a mirroring
direction (defaults to rx-tx for bidirectional mirroring, but can also be set
to rx-only for unidirectional mirroring)::

        rbd mirror pool peer bootstrap import [--site-name {local-site-name}] [--direction {rx-only or rx-tx}] {pool-name} {token-path}

For example, on site-b::

        $ cat <<EOF > token
        eyJmc2lkIjoiOWY1MjgyZGItYjg5OS00NTk2LTgwOTgtMzIwYzFmYzM5NmYzIiwiY2xpZW50X2lkIjoicmJkLW1pcnJvci1wZWVyIiwia2V5IjoiQVFBUnczOWQwdkhvQmhBQVlMM1I4RmR5dHNJQU50bkFTZ0lOTVE9PSIsIm1vbl9ob3N0IjoiW3YyOjE5Mi4xNjguMS4zOjY4MjAsdjE6MTkyLjE2OC4xLjM6NjgyMV0ifQ==
        EOF
        $ rbd --cluster site-b mirror pool peer bootstrap import --site-name site-b image-pool token


.. Add Cluster Peer Manually

手动增加互联的集群
------------------

Cluster peers can be specified manually if desired or if the above bootstrap
commands are not available with the currently installed Ceph release.

The remote ``rbd-mirror`` daemon will need access to the local cluster to
perform mirroring. A new local Ceph user should be created for the remote
daemon to use. To `创建一个 Ceph 用户`_, with ``ceph`` specify the
``auth get-or-create`` command, user name, monitor caps, and OSD caps::

        ceph auth get-or-create client.rbd-mirror-peer mon 'profile rbd' osd 'profile rbd'

The resulting keyring should be copied to the other cluster's ``rbd-mirror``
daemon hosts if not using the Ceph monitor ``config-key`` store described below.

To manually add a mirroring peer Ceph cluster with ``rbd``, specify the
``mirror pool peer add`` command, the pool name, and a cluster specification::

        rbd mirror pool peer add {pool-name} {client-name}@{cluster-name}

For example::

        $ rbd --cluster site-a mirror pool peer add image-pool client.rbd-mirror-peer@site-b
        $ rbd --cluster site-b mirror pool peer add image-pool client.rbd-mirror-peer@site-a

By default, the ``rbd-mirror`` daemon needs to have access to a Ceph
configuration file located at ``/etc/ceph/{cluster-name}.conf`` that provides
the addresses of the peer cluster's monitors, in addition to a keyring for
``{client-name}`` located in the default or configured keyring search paths
(e.g. ``/etc/ceph/{cluster-name}.{client-name}.keyring``).

Alternatively, the peer cluster's monitor and/or client key can be securely
stored within the local Ceph monitor ``config-key`` store. To specify the
peer cluster connection attributes when adding a mirroring peer, use the
``--remote-mon-host`` and ``--remote-key-file`` optionals. For example::

        $ cat <<EOF > remote-key-file
        AQAeuZdbMMoBChAAcj++/XUxNOLFaWdtTREEsw==
        EOF
        $ rbd --cluster site-a mirror pool peer add image-pool client.rbd-mirror-peer@site-b --remote-mon-host 192.168.1.1,192.168.1.2 --remote-key-file remote-key-file
        $ rbd --cluster site-a mirror pool info image-pool --all
        Mode: pool
        Peers: 
          UUID                                 NAME   CLIENT                 MON_HOST                KEY                                      
          587b08db-3d33-4f32-8af8-421e77abb081 site-b client.rbd-mirror-peer 192.168.1.1,192.168.1.2 AQAeuZdbMMoBChAAcj++/XUxNOLFaWdtTREEsw== 


.. Remove Cluster Peer

删除互联的集群
--------------

要用 ``rbd`` 删除镜像点 Ceph 集群，可指定 ``mirror pool peer remove``
命令、以及互联点的 UUID （可用 ``rbd mirror pool info`` 命令\
找出）： ::

        rbd mirror pool peer remove {pool-name} {peer-uuid}

例如： ::

        $ rbd --cluster site-a mirror pool peer remove image-pool 55672766-c02b-4729-8567-f13a66893445
        $ rbd --cluster site-b mirror pool peer remove image-pool 60c0e299-b38f-4234-91f6-eed0a367be08


.. Data Pools

数据存储池
----------

在目的集群创建映像时， ``rbd-mirror`` 这样选择数据集群：

#. 如果目的集群已配置了一个默认的数据存储池（用
   ``rbd_default_data_pool`` 配置选项），那就用它；
#. 否则，如果源映像位于独立的数据存储池内，而且目的集群上也有\
   同名的一个存储池，那就选用它；
#. 如果上述二者都不可行，那就不会选中数据存储池。


.. Image Configuration

映像配置
========

不像存储池配置方式，映像配置只需要操作单个镜像点 Ceph 集群就行。

被镜像的 RBD 映像需指定为主、或非主的，这是映像的属性、不是存\
储池的。被指定为非主的映像不能被修改。

某一映像的镜像功能被开启时，它会被自动晋级为主映像（在存储池\
镜像模式为 **pool** 且映像开启了 journaling 映像功能时为隐式\
的，也可以用 ``rbd`` 命令\ `显式地开启`_\ ）。


.. Enable Image Mirroring

启用基于映像的镜像
------------------

如果映像所在存储池的镜像功能配置成了 ``image`` 模式，那就得\
显式地启用各个映像的镜像功能。可以用 ``rbd`` 的
``mirror image enable`` 命令、再加上存储池、映像名和模式： ::

        rbd mirror image enable {pool-name}/{image-name} {mode}

The mirror image mode can either be ``journal`` or ``snapshot``:

* **journal** (default): When configured in ``journal`` mode, mirroring will
  utilize the RBD journaling image feature to replicate the image contents. If
  the RBD journaling image feature is not yet enabled on the image, it will be
  automatically enabled.

* **snapshot**:  When configured in ``snapshot`` mode, mirroring will utilize
  RBD image mirror-snapshots to replicate the image contents. Once enabled, an
  initial mirror-snapshot will automatically be created. Additional RBD image
  `mirror-snapshots`_ can be created by the ``rbd`` command.

例如： ::

        $ rbd --cluster site-a mirror image enable image-pool/image-1 snapshot
        $ rbd --cluster site-a mirror image enable image-pool/image-2 journal


.. Enable Image Journaling Feature

开启映像的 journaling 功能
--------------------------

RBD 镜像用 journaling 功能来保证复制的映像始终保持崩溃一致性。\
使用 ``image`` 镜像模式时，在此映像上启用镜像的同时就会自动\
启用日志功能；使用 ``pool`` 镜像模式时，必须先启用
RBD 映像日志功能，映像才能被镜像到对点集群。此功能可在创建映像\
时打开，即执行 ``rbd`` 命令时加上
``--image-feature exclusive-lock,journaling`` 选项。

另外，在已存在的 RBD 映像上也可以动态地开启 journaling 功能。\
要用 ``rbd`` 命令开启 journaling 功能可指定 ``feature enable``
命令、存储池名和映像名、以及功能名： ::

        rbd feature enable {pool-name}/{image-name} {feature-name}

例如： ::

        $ rbd --cluster site-a feature enable image-pool/image-1 journaling

.. note:: journaling 功能依赖于 exclusive-lock （互斥锁）功\
   能。如果 exclusive-lock 功能还没启用，应该先启用它、再启\
   用 journaling 功能。

.. tip:: 你可以让所有新映像默认启用日志功能，把
   ``rbd default features = 125`` 写入配置文件即可。


Create Image Mirror-Snapshots
-----------------------------

When using snapshot-based mirroring, mirror-snapshots will need to be created
whenever it is desired to mirror the changed contents of the RBD image. To
create a mirror-snapshot manually with ``rbd``, specify the
``mirror image snapshot`` command along with the pool and image name::

        rbd mirror image snapshot {pool-name}/{image-name}

For example::

        $ rbd --cluster site-a mirror image snapshot image-pool/image-1

By default only ``3`` mirror-snapshots will be created per-image. The most
recent mirror-snapshot is automatically pruned if the limit is reached.
The limit can be overridden via the ``rbd_mirroring_max_mirroring_snapshots``
configuration option if required. Additionally, mirror-snapshots are
automatically deleted when the image is removed or when mirroring is disabled.

Mirror-snapshots can also be automatically created on a periodic basis if
mirror-snapshot schedules are defined. The mirror-snapshot can be scheduled
globally, per-pool, or per-image levels. Multiple mirror-snapshot schedules can
be defined at any level, but only the most-specific snapshot schedules that
match an individual mirrored image will run.

To create a mirror-snapshot schedule with ``rbd``, specify the
``mirror snapshot schedule add`` command along with an optional pool or
image name; interval; and optional start time::

        rbd mirror snapshot schedule add [--pool {pool-name}] [--image {image-name}] {interval} [{start-time}]

The ``interval`` can be specified in days, hours, or minutes using ``d``, ``h``,
``m`` suffix respectively. The optional ``start-time`` can be specified using
the ISO 8601 time format. For example::

        $ rbd --cluster site-a mirror snapshot schedule add --pool image-pool 24h 14:00:00-05:00
        $ rbd --cluster site-a mirror snapshot schedule add --pool image-pool --image image1 6h

To remove a mirror-snapshot schedules with ``rbd``, specify the
``mirror snapshot schedule remove`` command with options that match the
corresponding ``add`` schedule command.

To list all snapshot schedules for a specific level (global, pool, or image)
with ``rbd``, specify the ``mirror snapshot schedule ls`` command along with
an optional pool or image name. Additionally, the ``--recursive`` option can
be specified to list all schedules at the specified level and below. For
example::

        $ rbd --cluster site-a mirror schedule ls --pool image-pool --recursive
        POOL        NAMESPACE IMAGE  SCHEDULE                            
        image-pool  -         -      every 1d starting at 14:00:00-05:00 
        image-pool            image1 every 6h                            

To view the status for when the next snapshots will be created for
snapshot-based mirroring RBD images with ``rbd``, specify the
``mirror snapshot schedule status`` command along with an optional pool or
image name::

        rbd mirror snapshot schedule status [--pool {pool-name}] [--image {image-name}]

For example::

        $ rbd --cluster site-a mirror schedule status
        SCHEDULE TIME       IMAGE             
        2020-02-26 18:00:00 image-pool/image1 


.. Disable Image Mirroring

禁用映像的镜像功能
------------------

要禁用某一映像的镜像功能，可用 ``rbd`` 、加
``mirror image disable`` 命令，再加上存储池名和映像名： ::

        rbd mirror image disable {pool-name}/{image-name}

例如： ::

        $ rbd --cluster site-a mirror image disable image-pool/image-1


.. Image Promotion and Demotion

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

        $ rbd --cluster site-a mirror image demote image-pool/image-1

要用 ``rbd`` 命令把存储池内的所有映像降级为非主的，可用
``mirror pool demote`` 命令，加上存储池名： ::

        rbd mirror pool demote {pool-name}

例如： ::

        $ rbd --cluster site-a mirror pool demote image-pool

要用 ``rbd`` 把某一映像晋级为主的，可用 ``mirror image promote``
命令、加存储池名和映像名： ::

        rbd mirror image promote [--force] {pool-name}/{image-name}

例如： ::

        $ rbd --cluster site-b mirror image promote image-pool/image-1

要用 ``rbd`` 命令把存储池内的所有映像晋级为主的，可用
``mirror pool promote`` 命令，加上存储池名： ::

        rbd mirror pool promote [--force] {pool-name}

例如： ::

        $ rbd --cluster site-a mirror pool promote image-pool

.. tip:: 由于主、非主状态是基于单个映像的，所以有可能让两个\
   集群分摊 IO 负载、并实现故障切换、故障恢复。

.. note:: 晋级可用 ``--force`` 选项强制施行。在降级未能传达\
   到互联的 Ceph 集群时（例如 Ceph 集群故障，通讯中断），就\
   需要强制晋级。这会导致两个对等点形成裂脑（ split-brain ），\
   并且这两个映像无法回到同步状态，只能通过\ \
   `强制重新同步命令`_\ 恢复同步。


.. Force Image Resync

强制重新同步映像
----------------

如果 ``rbd-mirror`` 守护进程探测到了裂脑事件，它就不会再企图\
镜像受影响的映像，除非已纠正。要恢复一个映像的镜像，首先找出\
过期的映像、并\ `降级此映像`_\ ，然后向主映像发出一个重新同\
步的请求。要用 ``rbd`` 请求重新同步映像，可用
``mirror image resync`` 命令、加上存储池名和映像名： ::

        rbd mirror image resync {pool-name}/{image-name}

例如： ::

        $ rbd mirror image resync image-pool/image-1

.. note:: ``rbd`` 命令仅仅把这个映像标记为需要重新同步。本\
   地集群的 ``rbd-mirror`` 守护进程负责异步地重新同步。


.. Mirror Status

镜像状态
========

每一个主的、被镜像的映像都存储了互联集群的复制状态，这些状态\
信息可用 ``mirror image status`` 和 ``mirror pool status``
命令查看。

要用 ``rbd`` 查看映像的镜像状态，可用 ``mirror image status``
命令、加上存储池名、映像名： ::

        rbd mirror image status {pool-name}/{image-name}

例如： ::

        $ rbd mirror image status image-pool/image-1

要用 ``rbd`` 命令查看存储池的镜像汇总状态，可用
``mirror pool status`` 命令、加上存储池名： ::

        rbd mirror pool status {pool-name}

例如： ::

        $ rbd mirror pool status image-pool

.. note:: 给 ``mirror pool status`` 命令加上 ``--verbose``
   选项，它还会额外输出此存储池内每一个映像的镜像状态细节。


.. rbd-mirror Daemon

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
.. warning:: 小于 Luminous 的版本：每个 Ceph 集群只能运行一个
   ``rbd-mirror`` 守护进程。

每个 ``rbd-mirror`` 守护进程都应该使用唯一的 Ceph 用户 ID 。\
要\ `_创建一个 Ceph 用户`\ ，用 ``ceph`` 命令，加上
``auth get-or-create`` 、用户名、监视器能力、和 OSD 能力： ::

  ceph auth get-or-create client.rbd-mirror.{unique id} mon 'profile rbd-mirror' osd 'profile rbd'

``rbd-mirror`` 守护进程可以用 ``systemd`` 管理，用户 ID 作为\
守护进程例程： ::

  systemctl enable ceph-rbd-mirror@rbd-mirror.{unique id}

``rbd-mirror`` 也能放在前台运行，命令如下： ::

  rbd-mirror -f --log-file={log_path}


.. _rbd: ../../man/8/rbd
.. _ceph-conf: ../../rados/configuration/ceph-conf/#running-multiple-clusters
.. _显式地开启: #enable-image-mirroring
.. _强制重新同步命令: #force-image-resync
.. _降级此映像: #image-promotion-and-demotion
.. _创建一个 Ceph 用户: ../../rados/operations/user-management#add-a-user
.. _mirror-snapshots: #create-image-mirror-snapshots
