CephFS 快照
===========
.. CephFS Snapshots

CephFS 支持快照，通常是在 ``.snap`` 目录下调用 mkdir 。注意，\
这是个隐藏的、特殊目录，罗列目录时不可见。

概览
----
.. Overview

通常，快照做的事正如其名：它们创建了一个拍下快照的那个时刻的文件系统视图。
有几个重要功能使得 CephFS 快照不同于你期望的样子：

* 任意子树。快照可以在你选定的任何目录拍下，而且涵盖那个目录下的所有、
  存在于文件系统上的数据。
* 异步性。如果你创建了一个快照，缓冲的数据是延后刷出的，包括其它客户端的数据。
  因此，“创建”快照是非常迅速的。

重要的数据结构
--------------
.. Important Data Structures

* SnapRealm: 只要你在目录层次内的新点上（或者，已经拍过快照的 inode
  被移出了它的父级快照）创建了一个快照，就会创建 `SnapRealm` 。
  SnapRealm 都包含一个 `sr_t srnode` 、和 `inodes_with_caps` ，都是快照的一部分。
  客户端们也有一个 SnapRealm 概念，维护的数据较少，
  但它是用于把 `SnapContext` 关联到各个打开、要写入的文件。
* sr_t: `sr_t` 是磁盘上的快照元数据。它是所在目录的一部分，
  包含顺序计数器、时间戳、关联的快照 ID 的列表、和 `past_parent_snaps` 。
* SnapServer: SnapServer 管理着快照 ID 的分配、快照删除、
  并追踪着文件系统内有效快照的各个列表。一个文件系统只有一个 snapserver 例程。
* SnapClient: SnapClient 用于和 snapserver 通讯，
  每个 MDS rank 都有它们自己的 snapclient 例程。
  SnapClient 也会在本地缓存有效的快照。

创建快照
--------
.. Creating a snapshot

CephFS 的快照功能在新文件系统上是默认启用的，要在现有文件系统上启用，
用以下命令。

.. code::

       $ ceph fs set <fs_name> allow_new_snaps true

快照功能启用后， CephFS 内的所有目录都有一个特殊的 ``.snap`` 目录。
（你如果愿意，可以用 ``client snapdir`` 选项配置个不同的名字。）

要创建一个 CephFS 快照，在 ``.snap`` 下创建一个子目录就行，
名字随你。例如，要给 /1/2/3/ 目录创建一个快照，
可以用 ``mkdir /1/2/3/.snap/my-snapshot-name`` 。

.. note::
   快照名不能以下划线（ "_" ）打头，
   因为这样的名字是为内部用途保留的。

.. note::
   快照名长度不能超过 240 个字符。
   因为 MDS 要在内部使用长快照名，格式是：
   `_<SNAPSHOT-NAME>_<INODE-NUMBER>` 。通常，文件名不能超过 255 个字符，
   而 `<node-id>` 占用 13 个字符，
   长快照名最多就只能有 255 - 1 - 1 - 13 = 240 个字符。

这会作为一个用 CEPH_MDS_OP_MKSNAP 打了标签的 `MClientRequest` 传给 MDS 服务器，
起初在 Server::handle_client_mksnap() 里处理。它从 `SnapServer` 分到了\
一个 `snapid` ，用新的 SnapRealm 规划了一个新的 inode ，并像往常一样提交到 MDLog 。
提交后，它会调用 `MDCache::do_realm_invalidate_and_update_notify()` ，
就会通知所有客户端，内容是 /1/2/3/ 下的文件们多了一层帽子，就是新的 SnapRealm 。
客户端们收到通知后，它们会更新客户端这边的 SnapRealm 层次结构，
把 /1/2/3/ 下的文件连接到这个新的 SnapRealm 、
并为新的 SnapRealm 生成一个 `SnapContext` 。

注意，这 *不是* 快照创建过程中的同步部分。

更新快照
--------
.. Updating a snapshot

如果你删除一个快照，走的过程相似。如果你把一个 inode 挪出它的父级 SnapRealm ，
重命名代码会给重命名过的 inode 创建一个新的 SnapRealm （如果还没有 SnapRealm ），
把原来父级 SnapRealm 上有效快照们的 ID 保存到新 SnapRealm 的 `past_parent_snaps` 里，
然后再走一次与新建快照相似的流程。

生成 SnapContext
----------------
.. Generating a SnapContext

RADOS `SnapContext` 包括一个快照顺序 ID （ `snapid` ）、和这个对象牵涉的所有快照 ID 。
要生成这个列表，我们把 SnapRealm 关联的 `snapids` 、和 `past_parent_snaps` 里的\
所有 `snapids` 结合起来，过时的 `snapids` 会被 SnapClient 缓存的有效快照过滤出去。

存入快照数据
------------
.. Storing snapshot data

文件数据存储在 RADOS “自己管理的”快照中。客户端在向 OSD 们写入文件数据时会\
小心地使用正确的 `SnapContext` 。

存入快照元数据
--------------
.. Storing snapshot metadata

拍过快照的 dentry （以及它们的 inode ）将作为目录的一部分以内联方式存储。
*所有的 dentry* 包含有 `first` 和 `last` snapid 的都是有效的。
（没拍过快照的 dentry 有 `last` 集合指向 CEPH_NOSNAP ）。

快照回写
--------
.. Snapshot writeback

有大量的代码可以有效地处理回写。客户端收到一条 `MClientSnap` 消息时，
它会更新本地 `SnapRealm` 表达式、以及它指向特定 `Inodes` 的连接，
并给这个 `Inode` 生成一个 `CapSnap` 。 `CapSnap` 会作为能力回写的一部分刷出去、
而且，如果存在脏数据， `CapSnap` 会阻塞新数据的写入，直到快照完全被刷回到 OSD 。

在 MDS 里，作为刷回的、常规流程的一部分，我们生成表示快照的 dentry 。
含有凸出的 `CapSnap` 数据的 dentry 会被保持在挂单状态、且在日志中。

删除快照
--------
.. Deleting snapshots

在目录的 .snaps 目录内调用 rmdir 就能删除快照。（试图删除有快照的目录 *会失败* ；
必须先删除所有快照。）一旦被删除，它们就进入 `OSDMap` 里的已删除快照列表、
之后文件数据就会被 OSD 删除。元数据会在目录对象被读入并再次写回时被清理掉。

硬链接
------
.. Hard links

有多个硬链接的 inode 会被挪进一个虚拟的全局 SnapRealms ，这个虚拟 SnapRealms
统管文件系统内的所有快照。这个 inode 的数据会保留给所有新快照，
这些保留的数据会覆盖所有与此 inode 相关的快照内链接。

多文件系统情况
--------------
.. Multi-FS

快照功能和多个文件系统还没有很好地对接。特别是，各个 MDS 集群是独立地分配 `snapids` 的；
如果你有多个文件系统共享着同一个存储池（通过命名空间），它们的快照会冲突、而且\
删除一个会导致另一个的文件数据丢失。（这甚至是不可见的，不会向用户抛出错误。）
如果各个 FS 都有它们自己的存储池，也许可以正常工作，但是这种场景还没完全测试，
有可能和期望的不一样。

.. Note:: 为避免 mon-managed 快照和文件系统快照的快照 ID 冲突，
   不允许有 mon-managed 快照的存储池加入文件系统。
   此外，也不能在已加入文件系统的存储池中创建 mon-managed 快照。
