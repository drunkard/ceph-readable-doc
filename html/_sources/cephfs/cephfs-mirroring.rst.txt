.. _cephfs-mirroring:

===============
CephFS 快照镜像
===============
.. CephFS Snapshot Mirroring

CephFS 支持通过 `cephfs-mirror` 工具，把快照异步地复制到远程 CephFS 文件系统。
快照的同步方式是镜像快照数据，
然后创建一个与源快照同名（给远程文件系统上的指定目录）的远程快照。


所需条件
--------
.. Requirements

主的（本地）和次的（远程） Ceph 集群版本应该是 Pacific 或更高版本。

.. _cephfs_mirroring_creating_users:

创建用户
--------
.. Creating Users

首先，在主的/本地集群上给 `cephfs-mirror` 守护进程创建一个 Ceph 用户。
此用户需要元数据存储池的写入能力，以创建用于监视/通知操作的
RADOS 对象（索引对象），还需要数据存储池的读取能力： ::

  $ ceph auth get-or-create client.mirror mon 'profile cephfs-mirror' mds 'allow r' osd 'allow rw tag cephfs metadata=*, allow r tag cephfs data=*' mgr 'allow r'

给每个文件系统互联节点（在次要的/远程集群上）创建一个 Ceph 用户。
此用户需要具备 MDS （要拍快照）和 OSD 的全部能力： ::

  $ ceph fs authorize <fs_name> client.mirror_remote / rwps

添加互联节点时，这个用户是互联节点规范的一部分。

启动镜像守护进程
----------------
.. Starting Mirror Daemon

镜像守护进程应该用 `systemctl(1)` 单元文件启动： ::

  $ systemctl enable cephfs-mirror@mirror
  $ systemctl start cephfs-mirror@mirror

`cephfs-mirror` 守护进程可以在前台运行，如下： ::

  $ cephfs-mirror --id mirror --cluster site-a -f

.. note:: 这里指定的用户是 `mirror` ，其创建方法在
   :ref:`Creating Users<cephfs_mirroring_creating_users>` 说明。

可以部署多个 ``cephfs-mirror`` 守护进程，以实现并行同步和高可用性。
镜像守护进程用简单的 ``M/N`` 策略分担同步载荷，
其中 ``M`` 是目录数， ``N`` 是 ``cephfs-mirror`` 守护进程数量。

用 ``cephadm`` 管理 Ceph 集群时，可运行下列命令部署 ``cephfs-mirror`` 守护进程：

.. prompt:: bash $

   ceph orch apply cephfs-mirror

要部署多个镜像守护进程，执行下列命令：

.. prompt:: bash $

   ceph orch apply cephfs-mirror --placement=<placement-spec>

例如，要在不同主机上部署 3 个 `cephfs-mirror` 守护进程，执行下列命令：

.. prompt:: bash $

  $ ceph orch apply cephfs-mirror --placement="3 host1,host2,host3"

管理接口
--------
.. Interface

`Mirroring` 模块（管理器插件）提供了管理目录快照镜像的接口。
这些接口（大多）是监视器命令的套壳，用于管理文件系统镜像，是建议使用的控制接口。

镜像模块
--------
.. Mirroring Module

镜像模块负责将目录分配给镜像守护进程进行同步。
可以派生出多个镜像守护进程，以并行地同步目录快照。
派生（或终止）镜像守护进程时，
镜像模块会发现有更改的一堆镜像守护进程，
然后重新均衡地分配一下镜像守护进程上的目录，从而提供高可用性。

.. note:: 建议只部署一个镜像守护进程。
   运行多个守护进程未经测试。

镜像功能支持的文件类型如下：

- 常规文件 (-)
- 目录文件 (d)
- 符号链接 (l)

镜像会忽略其他文件类型。因此，
在同步成功的对等节点上它们也不可用。

镜像模块默认是禁用的。
要启用镜像模块，运行以下命令：

.. prompt:: bash $

   ceph mgr module enable mirroring

镜像模块提供了一系列命令，
可用于控制目录快照的镜像。
要添加或删除目录，必须在指定文件系统上启用镜像功能。
要在指定文件系统上启用镜像功能，执行下列命令：

.. prompt:: bash $

   ceph fs snapshot mirror enable <fs_name>

.. note:: "Mirroring module" （镜像模块）命令的前缀是 ``fs snapshot mirror`` 。
   这样调用就把它们和“监视器命令”区分开了，监视器命令前缀是 ``fs mirror`` 。
   用监视器命令启用镜像功能会导致镜像守护进程进入“失败（ failed ）”状态，
   原因是缺少 `cephfs_mirror` 索引对象。
   因此（在这种情况下）一定要用模块命令。

要禁用指定文件系统的镜像功能，执行下列命令：

.. prompt:: bash $

   ceph fs snapshot mirror disable <fs_name>

启用镜像功能后，可以添加目录快照的镜像对端互联节点。
互联节点按照 ``<client>@<cluster>`` 格式指定，
在本文档的其他地方叫做 ``remote_cluster_spec`` 。
添加互联节点时会分配一个唯一标识（ UUID ）。
有关如何创建用于做镜像的 Ceph 用户，
请参阅 :ref:`创建用户 <cephfs_mirroring_creating_users>` 部分。

要添加互联节点，执行下列命令：

.. prompt:: bash $

   ceph fs snapshot mirror peer_add <fs_name> <remote_cluster_spec> [<remote_fs_name>] [<remote_mon_host>] [<cephx_key>]

``<remote_cluster_spec>`` 的格式是 ``client.<id>@<cluster_name>`` 。

``<remote_fs_name>`` 是可选的，默认值是 `<fs_name>`
（在远端集群上）。

这个命令要想成功执行的话，远端集群的 Ceph 配置\
和用户密钥环必须在主集群中能访问到。例如，
如果在远程集群上创建了名为 ``client_mirror`` 的用户，
该用户在名为 ``remote_fs`` 的远程文件系统上拥有
``rwps`` 权限（参阅\ `创建用户`\ ），且远程集群名为 ``remote_ceph``
（即主集群上的远程集群配置文件名是 ``remote_ceph.conf`` ），
则运行以下命令，可以将远程文件系统添加成\
主文件系统 ``primary_fs`` 的对等文件系统：

.. prompt:: bash $

   ceph fs snapshot mirror peer_add primary_fs client.mirror_remote@remote_ceph remote_fs

为避免在主集群中维护远程集群配置文件和\
远程 ceph 用户密钥环，用户可以启动一个互联节点
（把相关远程集群的详细信息存入主集群上的\
监视器配置存储库中）。
参阅“\ :ref:`启动互联节点 <cephfs_mirroring_bootstrap_peers>`\ ”部分。

``peer_add`` 命令可以加上远程集群监视器地址和用户密钥。
不过，添加互联节点的推荐方法是启动（ bootstrap ）\
互联节点。

.. note:: 当前只支持单个互联节点。

要删除一个互联节点，执行下列命令：

.. prompt:: bash $

   ceph fs snapshot mirror peer_remove <fs_name> <peer_uuid>

要罗列出文件系统的镜像互联节点，执行下列命令：

.. prompt:: bash $

   ceph fs snapshot mirror peer_list <fs_name>

配置一个要镜像的目录，执行下列命令：

.. prompt:: bash $

   ceph fs snapshot mirror add <fs_name> <path>

要罗列配置的目录，执行下列命令：

.. prompt:: bash $

   ceph fs snapshot mirror ls <fs_name>

要停止目录快照的镜像功能，执行下列命令：

.. prompt:: bash $

   ceph fs snapshot mirror remove <fs_name> <path>

只能用目录的绝对路径。

路径是由镜像模块规范化的。这意味着 ``/a/b/.../b``
等同于 ``/a/b`` 。路径总是从 CephFS 文件系统根目录开始，
而不是从主机系统挂载点开始。

例如： ::

  $ mkdir -p /d0/d1/d2
  $ ceph fs snapshot mirror add cephfs /d0/d1/d2
  {}
  $ ceph fs snapshot mirror add cephfs /d0/d1/../d1/d2
  Error EEXIST: directory /d0/d1/d2 is already tracked

配置好一个目录的镜像后，不允许再镜像\
它的子目录或它的上级目录目录： ::

  $ ceph fs snapshot mirror add cephfs /d0/d1
  Error EINVAL: /d0/d1 is a ancestor of tracked path /d0/d1/d2
  $ ceph fs snapshot mirror add cephfs /d0/d1/d2/d3
  Error EINVAL: /d0/d1/d2/d3 is a subtree of tracked path /d0/d1/d2

:ref:`镜像状态 <cephfs_mirroring_mirroring_status>`\ 段落\
包含一些信息，是用于检查目录映射（到镜像守护进程的）和检查目录分布的命令。


.. _cephfs_mirroring_bootstrap_peers:

启动互联节点
------------
.. Bootstrap Peers

添加互联节点（通过 `peer_add` ）时，要求在主集群（管理器主机和\
运行镜像守护进程的主机）上能看到互联节点的集群配置和用户密钥环。
这可以通过启动和导入一个互联节点的令牌来避免。
互联节点的启动包括在互联集群上创建一个启动令牌： ::

  $ ceph fs snapshot mirror peer_bootstrap create <fs_name> <client_entity> <site-name>

例如： ::

  $ ceph fs snapshot mirror peer_bootstrap create backup_fs client.mirror_remote site-remote
  {"token": "eyJmc2lkIjogIjBkZjE3MjE3LWRmY2QtNDAzMC05MDc5LTM2Nzk4NTVkNDJlZiIsICJmaWxlc3lzdGVtIjogImJhY2t1cF9mcyIsICJ1c2VyIjogImNsaWVudC5taXJyb3JfcGVlcl9ib290c3RyYXAiLCAic2l0ZV9uYW1lIjogInNpdGUtcmVtb3RlIiwgImtleSI6ICJBUUFhcDBCZ0xtRmpOeEFBVnNyZXozai9YYUV0T2UrbUJEZlJDZz09IiwgIm1vbl9ob3N0IjogIlt2MjoxOTIuMTY4LjAuNTo0MDkxOCx2MToxOTIuMTY4LjAuNTo0MDkxOV0ifQ=="}

`site-name` 指的是用户定义的字符串，用于标识远程文件系统。在 `peer_add` 接口的\
上下文中， `site-name` 就是 `remote_cluster_spec` 里传入的集群名（ `cluster_name` ）。

在主集群里导入启动令牌，用命令： ::

  $ ceph fs snapshot mirror peer_bootstrap import <fs_name> <token>

例如： ::

  $ ceph fs snapshot mirror peer_bootstrap import cephfs eyJmc2lkIjogIjBkZjE3MjE3LWRmY2QtNDAzMC05MDc5LTM2Nzk4NTVkNDJlZiIsICJmaWxlc3lzdGVtIjogImJhY2t1cF9mcyIsICJ1c2VyIjogImNsaWVudC5taXJyb3JfcGVlcl9ib290c3RyYXAiLCAic2l0ZV9uYW1lIjogInNpdGUtcmVtb3RlIiwgImtleSI6ICJBUUFhcDBCZ0xtRmpOeEFBVnNyZXozai9YYUV0T2UrbUJEZlJDZz09IiwgIm1vbl9ob3N0IjogIlt2MjoxOTIuMTY4LjAuNTo0MDkxOCx2MToxOTIuMTY4LjAuNTo0MDkxOV0ifQ==


.. _cephfs_mirroring_mirroring_status:

开始镜像快照
------------
.. Snapshot Mirroring

要启动快照镜像，在主集群中给已配置的目录创建一个快照： ::

  $ mkdir -p /d0/d1/d2/.snap/snap1

镜像状态
--------
.. Mirroring Status

CephFS 镜像模块提供了 `mirror daemon status` 接口，用于检查镜像守护进程的状态： ::

  $ ceph fs snapshot mirror daemon status
  [
    {
      "daemon_id": 284167,
      "filesystems": [
        {
          "filesystem_id": 1,
          "name": "a",
          "directory_count": 1,
          "peers": [
            {
              "uuid": "02117353-8cd1-44db-976b-eb20609aa160",
              "remote": {
                "client_name": "client.mirror_remote",
                "cluster_name": "ceph",
                "fs_name": "backup_fs"
              },
              "stats": {
                "failure_count": 1,
                "recovery_count": 0
              }
            }
          ]
        }
      ]
    }
  ]

每个镜像守护进程实例都会显示一个条目，以及已配置的互联节点和基本统计信息。
想知道更详细的统计信息，可以用管理套接字接口，详情如下。

CephFS 镜像守护进程提供用于查询镜像状态的管理员套接字命令。
要查看和镜像状态有关的命令，用： ::

  $ ceph --admin-daemon /path/to/mirror/daemon/admin/socket help
  {
      ....
      ....
      "fs mirror status cephfs@360": "get filesystem mirror status",
      ....
      ....
  }

以 ``fs mirror status`` 为前缀的命令可查看镜像状态，适用于启用了镜像功能的文件系统。
注意， `cephfs@360` 是按照 `filesystem-name@filesystem-id` 这个格式。
需要使用这种格式，是因为镜像守护进程会异步地收到有关文件系统镜像状态的通知
（文件系统可以用同一名称删除和重新创建）。

该命令目前只提供有关镜像状态的最基本信息： ::

  $ ceph --admin-daemon /var/run/ceph/cephfs-mirror.asok fs mirror status cephfs@360
  {
    "rados_inst": "192.168.0.5:0/1476644347",
    "peers": {
        "a2dc7784-e7a1-4723-b103-03ee8d8768f8": {
            "remote": {
                "client_name": "client.mirror_remote",
                "cluster_name": "site-a",
                "fs_name": "backup_fs"
            }
        }
    },
    "snap_dirs": {
        "dir_count": 1
    }
  }

上面命令输出中的 `Peers` 一段显示的是互联节点信息，包括唯一的互联节点标识
（ UUID ）和镜像规范。删除现有互联节点时需要使用互联节点 ID （ peer-id ），
在\ `镜像模块和接口`\ 小节说过了。

以 ``fs mirror peer status`` 为前缀的命令能查看互联节点的同步状态。
命令格式为 `filesystem-name@filesystem-id peer-uuid`::

  $ ceph --admin-daemon /var/run/ceph/cephfs-mirror.asok fs mirror peer status cephfs@360 a2dc7784-e7a1-4723-b103-03ee8d8768f8
  {
    "/d0": {
        "state": "idle",
        "last_synced_snap": {
            "id": 120,
            "name": "snap1",
            "sync_duration": 3,
            "sync_time_stamp": "274900.558797s",
            "sync_bytes": 52428800
        },
        "snaps_synced": 2,
        "snaps_deleted": 0,
        "snaps_renamed": 0
    }
  }

当重启守护进程和/或把目录重新分配给另一个镜像守护进程时
（假设部署了多个镜像守护进程），包括 `snaps_synced` 、 `snaps_deleted` 和
`snaps_renamed` 在内的这些同步统计信息将被重置。

目录状态可以是以下之一： ::

  - `idle`: 此目录当前尚未同步
  - `syncing`: 此目录当前正在同步
  - `failed`: 此目录已达到最大连续失败数

当现在正同步某个目录时，镜像守护进程会把它标记为 `syncing` ，并且
`fs mirror peer status` 会在 `current_syncing_snap` 内显示正在同步的快照： ::

  $ ceph --admin-daemon /var/run/ceph/cephfs-mirror.asok fs mirror peer status cephfs@360 a2dc7784-e7a1-4723-b103-03ee8d8768f8
  {
    "/d0": {
        "state": "syncing",
        "current_syncing_snap": {
            "id": 121,
            "name": "snap2"
        },
        "last_synced_snap": {
            "id": 120,
            "name": "snap1",
            "sync_duration": 3,
            "sync_time_stamp": "274900.558797s",
            "sync_bytes": 52428800
        },
        "snaps_synced": 2,
        "snaps_deleted": 0,
        "snaps_renamed": 0
    }
  }

同步完成后，镜像守护进程仍然把它标记为 `idle` 。

当某个目录同步的连续失败次数达到设定值时，镜像守护进程会把它标记为 `failed` 。
稍后会重试同步这些目录。默认情况下，目录被标记为失败的连续失败次数由
`cephfs_mirror_max_consecutive_failures_per_directory` 配置选项控制
（默认值： 10 ），失败目录的重试间隔由
`cephfs_mirror_retry_failed_directories_interval` 配置选项控制（默认值：60s）。

例如，添加一个普通文件进行同步会导致失败状态： ::

  $ ceph fs snapshot mirror add cephfs /f0
  $ ceph --admin-daemon /var/run/ceph/cephfs-mirror.asok fs mirror peer status cephfs@360 a2dc7784-e7a1-4723-b103-03ee8d8768f8
  {
    "/d0": {
        "state": "idle",
        "last_synced_snap": {
            "id": 121,
            "name": "snap2",
            "sync_duration": 5,
            "sync_time_stamp": "500900.600797s",
            "sync_bytes": 78643200
        },
        "snaps_synced": 3,
        "snaps_deleted": 0,
        "snaps_renamed": 0
    },
    "/f0": {
        "state": "failed",
        "snaps_synced": 0,
        "snaps_deleted": 0,
        "snaps_renamed": 0
    }
  }

系统允许用户添加不存在的目录进行同步。镜像守护进程会把此类目录标记为失败并重试
（频率较低）。这个目录创建后，镜像守护进程会在同步成功后清除之前标记的失败状态。

在远程文件系统的 .snap 目录中手动增加新快照或新目录，
会导致配置的对应目录进入失败状态。在远程文件系统中操作： ::

  $ ceph fs subvolume snapshot create cephfs subvol1 snap2 group1
  or
  $ mkdir /d0/.snap/snap2

  $ ceph --admin-daemon /var/run/ceph/cephfs-mirror.asok fs mirror peer status cephfs@360 a2dc7784-e7a1-4723-b103-03ee8d8768f8
  {
    "/d0": {
        "state": "failed",
        "failure_reason": "snapshot 'snap2' has invalid metadata",
        "last_synced_snap": {
            "id": 120,
            "name": "snap1",
            "sync_duration": 3,
            "sync_time_stamp": "274900.558797s"
        },
        "snaps_synced": 2,
        "snaps_deleted": 0,
        "snaps_renamed": 0
    },
    "/f0": {
        "state": "failed",
        "snaps_synced": 0,
        "snaps_deleted": 0,
        "snaps_renamed": 0
    }
  }

当远程文件系统删除快照或目录后，镜像守护进程将在成功地同步\
以前积攒的待处理快照（如有的话）后清除 failed 状态。

.. note:: 把远程文件系统当作只读的。 CephFS 本身没有什么必须要做的操作。
   但是， mds 能力配置正确的话，在远程文件系统上，用户就无法对目录拍快照。

禁用镜像功能后，对应文件系统的 `fs mirror status` 命令就不会显示在命令帮助中。

指标
----
.. Metrics

CephFS 把镜像指标导出成了 :ref:`Labeled Perf Counters` ， OCP/ODF Dashboard 将\
使用这些计数器来做 Geo Replication 监控。这些指标可用于衡量 cephfs_mirror 同步的进度，
从而提供监控功能。 CephFS 会导出以下镜像指标，用 ``counter dump`` 命令可显示这些指标。

.. list-table:: 镜像状态指标
   :widths: 25 25 75
   :header-rows: 1

   * - 名字
     - 类型
     - 描述
   * - mirroring_peers
     - Gauge
     - 镜像涉及的互联节点数量
   * - directory_count
     - Gauge
     - 需要同步的目录总数
   * - mirrored_filesystems
     - Gauge
     - 要镜像的文件系统总数
   * - mirror_enable_failures
     - Counter
     - 镜像功能启用失败

.. list-table:: 复制指标
   :widths: 25 25 75
   :header-rows: 1

   * - 名字
     - 类型
     - 描述
   * - snaps_synced
     - Counter
     - 成功同步的快照总数
   * - sync_bytes
     - Counter
     - 已同步的总字节数
   * - sync_failures
     - Counter
     - 快照同步的失败总数
   * - snaps_deleted
     - Counter
     - 删掉的快照总数
   * - snaps_renamed
     - Counter
     - 重命名的快照总数
   * - avg_sync_time
     - Gauge
     - 所有快照同步花费的时间平均值
   * - last_synced_start
     - Gauge
     - 上次同步快照的同步开始时间
   * - last_synced_end
     - Gauge
     - 上次同步快照的同步结束时间
   * - last_synced_duration
     - Gauge
     - 上次同步的持续时间
   * - last_synced_bytes
     - counter
     - 上次同步的快照，同步过去的总字节数

配置选项
--------
.. Configuration Options

.. confval:: cephfs_mirror_max_concurrent_directory_syncs
.. confval:: cephfs_mirror_action_update_interval
.. confval:: cephfs_mirror_restart_mirror_on_blocklist_interval
.. confval:: cephfs_mirror_max_snapshot_sync_per_cycle
.. confval:: cephfs_mirror_directory_scan_interval
.. confval:: cephfs_mirror_max_consecutive_failures_per_directory
.. confval:: cephfs_mirror_retry_failed_directories_interval
.. confval:: cephfs_mirror_restart_mirror_on_failure_interval
.. confval:: cephfs_mirror_mount_timeout
.. confval:: cephfs_mirror_perf_stats_prio

重新添加互联节点
----------------
.. Re-adding Peers

给另一个集群的文件系统重新添加（重新分配）一个互联节点时，要确保所有\
镜像守护进程都已停止向这个互联节点的同步。可以用 `fs mirror status` admin socket
命令来检查（命令输出中应该不会显示 `Peer UUID` ）。而且，想要把这个互联节点\
重新添加给另一个文件系统的话，建议先清除它上面已经同步的目录
（尤其是新的主文件系统中可能存在的同名目录）。
如果还是把互联节点重新添加到先前同步的同一个主文件系统，那就不需要清除。
