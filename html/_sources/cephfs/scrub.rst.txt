.. _mds-scrub:

===================
 Ceph 文件系统洗刷
===================
.. Ceph File System Scrub

CephFS 为集群管理员（操作员）提供了一系列洗刷命令，\
可以用以检查文件系统的一致性。洗刷可分类为两部分：

#. 正向洗刷（ Forward Scrub ）：此时，洗刷操作从文件系统的根目录\
   （或一个子目录）开始，检查层次结构中可以触及的所有内容，以确保一致性。

#. 逆向洗刷（ Backward Scrub ）：此时，洗刷操作会检查文件系统存储池中的\
   每个 RADOS 对象，并将其映射回文件系统层次结构。

本文档详细介绍了启动和控制前向洗刷（后文简称为洗刷）的命令。

.. warning::

   CephFS 正向洗刷是在 rank 0 上启动和操作的。所有洗刷命令都必须指向 rank 0 。


初始化文件系统洗刷
==================
.. Initiate File System Scrub

启动对一个目录树的洗刷操作，用下列命令： ::

   ceph tell mds.<fsname>:0 scrub start <path> [scrubopts] [tag]

其中， ``scrubopts`` 逗号分隔的列表，内容是 ``recursive`` 、 ``force`` 、或
``repair`` ，还有 ``tag`` ，它是一个可选的定制字符串标签（默认值是\
生成的 UUID ）。命令实例： ::

   ceph tell mds.cephfs:0 scrub start / recursive
   {
       "return_code": 0,
       "scrub_tag": "6f0d204c-6cfd-4300-9e02-73f382fd23c1",
       "mode": "asynchronous"
   }

递归洗刷是异步的（上面输出中的 `mode` 说了）。
异步洗刷必须用 ``scrub status`` 进行轮询，才能看到状态。

洗刷标签（ scrub tag ）用于区分洗刷，也用于在默认数据存储池
（回溯信息存储的位置）中标记每个 inode 的第一个数据对象，
并在 ``scrub_tag`` 扩展属性中写入该标签的值。
用 RADOS 工具查看扩展属性，
就可以验证一个 inode 是否已被洗刷。

洗刷操作可以同时在多个活跃 MDS（多个 rank ）上进行。
洗刷由 rank 0 管理，并根据情况分配到各个 MDS 。


监视（进行中的）文件系统洗刷操作
================================
.. Monitor (ongoing) File System Scrubs

可以用 ``scrub status`` 命令监视和轮询正在进行的洗刷操作。
该命令会列出正在进行的洗刷（从标签认出的），
以及启动洗刷的路径和选项： ::

   ceph tell mds.cephfs:0 scrub status
   {
       "status": "scrub active (85 inodes in the stack)",
       "scrubs": {
           "6f0d204c-6cfd-4300-9e02-73f382fd23c1": {
               "path": "/",
               "options": "recursive"
           }
       }
   }

`status` 显示的是按计划随时可进行洗刷的 inode 数量，
因此，随后用 `scrub status` 查看时会变化。此外，
洗刷操作的高级汇总（其中包括操作状态和开始洗刷的路径）
也会显示在 `ceph status` 中： ::

   ceph status
   [...]

   task status:
     scrub status:
         mds.0: active [paths:/]

   [...]

当不再显示在此列表中时，洗刷就完成了（不过在未来的版本中可能会有所变化）。
有损坏的话会在集群健康警告里报告。

控制（进行中的）文件系统洗刷
============================
.. Control (ongoing) File System Scrubs

- 暂停（ Pause ）：暂停进行中的洗刷操作会导致正在进行的 RADOS 操作
  （对应当前正被洗刷的 inode ）结束后，不再洗刷新的或待处理的 inode ： ::

   ceph tell mds.cephfs:0 scrub pause
   {
       "return_code": 0
   }

  暂停后的 ``scrub status`` 会反映暂停后的状态。此时，
  启动新的洗刷操作（通过 ``scrub start`` ）
  将只是把这些 inode 放入洗刷队列： ::

   ceph tell mds.cephfs:0 scrub status
   {
       "status": "PAUSED (66 inodes in the stack)",
       "scrubs": {
           "6f0d204c-6cfd-4300-9e02-73f382fd23c1": {
               "path": "/",
               "options": "recursive"
           }
       }
   }

- 恢复（ Resume ）：恢复启动已暂停的洗刷操作： ::

   ceph tell mds.cephfs:0 scrub resume
   {
       "return_code": 0
   }

- 中止（ Abort ）：在进行中的 RADOS 操作（对应当前正在洗刷的 inode ）结束后，
  中止正在进行的洗刷操作会从洗刷队列中删除待处理的 inode
  （从而中止洗刷）： ::

   ceph tell mds.cephfs:0 scrub abort
   {
       "return_code": 0
   }

损坏的
======
.. Damages

文件系统洗刷操作能够报告和修复的损坏类型有：

* DENTRY : Inode 的 dentry 丢失了。

* DIR_FRAG : Inode 的目录片段丢失了。

* BACKTRACE : Inode 放在数据存储池里的的回溯信息损坏了。

上面说的这些 MDS 损坏可以用下列命令修复： ::

    ceph tell mds.<fsname>:0 scrub start /path recursive, repair, force

如果洗刷能够修复损坏的，那么对应的条目会从损坏表自动删除。

用递归洗刷甄别流浪者
====================
.. Evaluate strays using recursive scrub

- 为了甄别流浪者，即清除 ``~mdsdir`` 中的流浪目录，用以下命令： ::

    ceph tell mds.<fsname>:0 scrub start ~mdsdir recursive

- 从 CephFS 根目录启动洗刷时， ``~mdsdir`` 默认不会进入队列。要在根目录下\
  执行流浪者甄别，加上 ``scrub_mdsdir`` 和 ``recursive`` 标志运行洗刷： ::

    ceph tell mds.<fsname>:0 scrub start / recursive,scrub_mdsdir
