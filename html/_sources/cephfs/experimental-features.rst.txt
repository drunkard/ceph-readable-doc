.. Experimental Features

实验性功能
==========

CephFS includes a number of experimental features which are not fully stabilized
or qualified for users to turn on in real deployments. We generally do our best
to clearly demarcate these and fence them off so they can't be used by mistake.

Some of these features are closer to being done than others, though. We describe
each of them with an approximation of how risky they are and briefly describe
what is required to enable them. Note that doing so will *irrevocably* flag maps
in the monitor as having once enabled this flag to improve debugging and
support processes.


.. Inline data

内联数据
--------
By default, all CephFS file data is stored in RADOS objects. The inline data
feature enables small files (generally <2KB) to be stored in the inode
and served out of the MDS. This may improve small-file performance but increases
load on the MDS. It is not sufficiently tested to support at this time, although
failures within it are unlikely to make non-inlined data inaccessible

Inline data has always been off by default and requires setting
the "inline_data" flag.


.. Mantle: Programmable Metadata Load Balancer

Mantle: 可编程的元数据负载均衡器
--------------------------------

Mantle is a programmable metadata balancer built into the MDS. The idea is to
protect the mechanisms for balancing load (migration, replication,
fragmentation) but stub out the balancing policies using Lua. For details, see
:doc:`/cephfs/mantle`.


.. Snapshots

快照
----
Like multiple active MDSes, CephFS is designed from the ground up to support
snapshotting of arbitrary directories. There are no known bugs at the time of
writing, but there is insufficient testing to provide stability guarantees and
every expansion of testing has generally revealed new issues. If you do enable
snapshots and experience failure, manual intervention will be needed.

Snapshots are known not to work properly with multiple filesystems (below) in
some cases. Specifically, if you share a pool for multiple FSes and delete
a snapshot in one FS, expect to lose snapshotted file data in any other FS using
snapshots. See the :doc:`/dev/cephfs-snapshots` page for more information.

出于某些含糊的实现原因，内核客户端仅支持最多 400 个快照（
http://tracker.ceph.com/issues/21420 ）。

在 Mimic 版之前，快照功能是用 allow_new_snaps 标记屏蔽的。


.. Multiple filesystems within a Ceph cluster

单个 Ceph 集群内建多个文件系统
------------------------------
Code was merged prior to the Jewel release which enables administrators
to create multiple independent CephFS filesystems within a single Ceph cluster.
These independent filesystems have their own set of active MDSes, cluster maps,
and data. But the feature required extensive changes to data structures which
are not yet fully qualified, and has security implications which are not all
apparent nor resolved.

There are no known bugs, but any failures which do result from having multiple
active filesystems in your cluster will require manual intervention and, so far,
will not have been experienced by anybody else -- knowledgeable help will be
extremely limited. You also probably do not have the security or isolation
guarantees you want or think you have upon doing so.

Note that snapshots and multiple filesystems are *not* tested in combination
and may not work together; see above.

Multiple filesystems were available starting in the Jewel release candidates
but must be turned on via the ``enable_multiple`` flag until declared stable.


LazyIO
------
LazyIO 放松了 POSIX 语义的要求，允许读、写缓冲的存在，即使有\
多个客户端上的多个应用程序同时打开一个文件时也允许。应用程序\
自行负责缓存一致性的管理。


.. Previously experimental features

曾经是实验性的功能
==================

.. Directory Fragmentation

目录分片
--------
在 *Luminous* (12.2.x) 版以前，目录分片功能是实验性的，现在会\
给新文件系统默认启用。旧版 Ceph 上的文件系统若要启用目录分片功\
能，可给此文件系统设置 ``allow_dirfrags`` 标记。 ::

    ceph fs set <filesystem name> allow_dirfrags 1


.. Multiple active metadata servers

多活元数据服务器
----------------
在 *Luminous* 12.2.x 版之前，在单个文件系统内运行多活\
元数据服务器是实验性的功能；现在，默认就允许在新文件系统上创建\
多活元数据服务器。

用较老版本的 Ceph 创建的文件系统仍然需要明确启用多活\
元数据服务器，如下： ::

    ceph fs set <file system name> allow_multimds 1

注意，活跃 MDS 集群的默认尺寸（ ``max_mds`` ），其初始值仍然是
1 。

