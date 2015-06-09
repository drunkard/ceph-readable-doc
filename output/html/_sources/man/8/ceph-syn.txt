:orphan:

===================================
 ceph-syn -- ceph 的人造负载生成器
===================================

.. program:: ceph-syn

提纲
====

| **ceph-syn** [ -m *monaddr*:*port* ] --syn *command* *...*


描述
====

**ceph-syn** 是个适用于 Ceph 分布式文件系统的简单的人造载荷生成器。它通过用\
户空间客户端库在当前运行着的文件系统上生成简单的载荷，此文件系统不必通过 \
ceph-fuse(8) 或内核客户端挂载。

一或多个 ``--syn`` 命令参数规定特定的载荷，具体文档如下。


选项
====

.. option:: -d

   启动后离开控制台并进入后台。

.. option:: -c ceph.conf, --conf=ceph.conf

   启动时用 *ceph.conf* 配置文件而非默认的 ``/etc/ceph/ceph.conf`` 来确定监\
   视器地址。

.. option:: -m monaddress[:port]

   连接到指定监视器（而不是在 ``ceph.conf`` 里查找）。

.. option:: --num_client num

   模拟 num 个不同的客户端，都位于独立的线程中。

.. option:: --syn workloadspec

   运行给定载荷。可以指定任意多次，通常依次被运行。


载荷
====

命令行中每个载荷前都要加 ``--syn`` 。下面是个不太完整的列表。

:command:`mknap` *path* *snapname*
  在 *path* 下创建名为 *snapname* 的快照。

:command:`rmsnap` *path* *snapname*
  删除 *path* 路径下名为 *snapname* 的快照。

:command:`rmfile` *path*
  删除或断链 *path* 。

:command:`writefile` *sizeinmb* *blocksize*
  创建一个文件，以我们的客户端 ID 命名，写入 *blocksize* 个尺寸为 *sizeinmb* \
  MB 的数据块。

:command:`readfile` *sizeinmb* *blocksize*
  读文件，以我们的客户端 ID 命名，写入 *blocksize* 个尺寸为 *sizeinmb* MB 的\
  数据块。

:command:`rw` *sizeinmb* *blocksize*
  写文件，然后再读出，像上面的一样。

:command:`makedirs` *numsubdirs* *numfiles* *depth*
  创建深度为 *depth* 级的分级目录，各目录有 *numsubdirs* 个子目录和 \
  *numfiles* 个文件。

:command:`walk`
  递归地遍历文件系统（类似 find ）。


使用范围
========

**ceph-syn** 是 Ceph 的一部分，这是个伸缩力强、开源、分布式的存储系统，\
更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`ceph <ceph>`\(8),
:doc:`ceph-fuse <ceph-fuse>`\(8)
