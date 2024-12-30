===================
 CephFS 支持的能力
===================
.. Capabilities in CephFS

一个客户端想要操作索引节点时，它会向 MDS 发起多种查询，
MDS 会授予此客户端一系列\ *能力（ capabilities ）*\ ，有了这些能力，\
客户端就有权限操作索引节点了。与其它网络文件系统（如 NFS 或 SMB ）相比，
一个主要的区别在于这些授予的能力非常粗糙，
而且它允许多个客户端同时持有同一索引节点的不同能力。

能力的种类
----------
.. Types of Capabilities

有几种“常规”能力位，下面的表示这个能力位授予它哪种能力。

.. code-block:: cpp

        /* generic cap bits */
        #define CEPH_CAP_GSHARED     1  /* (metadata) client can reads (s) */
        #define CEPH_CAP_GEXCL       2  /* (metadata) client can read and update (x) */
        #define CEPH_CAP_GCACHE      4  /* (file) client can cache reads (c) */
        #define CEPH_CAP_GRD         8  /* (file) client can read (r) */
        #define CEPH_CAP_GWR        16  /* (file) client can write (w) */
        #define CEPH_CAP_GBUFFER    32  /* (file) client can buffer writes (b) */
        #define CEPH_CAP_GWREXTEND  64  /* (file) client can extend EOF (a) */
        #define CEPH_CAP_GLAZYIO   128  /* (file) client can perform lazy io (l) */

然后，给这些数字做一定数量的位移。这些表示能力是要授权给 inode 的数据或元数据的一部分：

.. code-block:: cpp

        /* per-lock shift */
        #define CEPH_CAP_SAUTH      2 /* A */
        #define CEPH_CAP_SLINK      4 /* L */
        #define CEPH_CAP_SXATTR     6 /* X */
        #define CEPH_CAP_SFILE      8 /* F */

.. note:: 译者注：下面这段没完全理解，也许翻译的有问题，先附上原文。

:译文: 然而，以前只有某些常规能力授予了其中的一些“位移”，特别是，只有
       FILE 位移多于前两位。
:原文: Only certain generic cap types are ever granted for some of those "shifts", however. In particular, only the FILE shift ever has more than the first two bits.

::

        | AUTH | LINK | XATTR | FILE
        2      4      6       8

从以上定义我们得到几个常数，它是用各个位值移位到相应的位（用变量表示）生成的：

.. code-block:: cpp

        #define CEPH_CAP_AUTH_SHARED  (CEPH_CAP_GSHARED  << CEPH_CAP_SAUTH)

然后，这些位就可以通过或操作产生位掩码（ bitmask ），用以表示一系列能力。

有一个例外的：

.. code-block:: cpp

        #define CEPH_CAP_PIN  1  /* no specific capabilities beyond the pin */

pin 只是把 inode 插入内存，不授予任何能力。

图形化就是： ::

    +---+---+---+---+---+---+---+---+
    | p | _ |As   x |Ls   x |Xs   x |
    +---+---+---+---+---+---+---+---+
    |Fs   x   c   r   w   b   a   l |
    +---+---+---+---+---+---+---+---+

当前尚未使用第二个 bit 。


各个 cap 授予的能力
-------------------
.. Abilities granted by each cap

这就解释了能力是如何授予的（和通讯的），重要的位是它允许客户端干什么：

* **PIN**: 只是把 inode 插入内存。这足以让客户端取得 inode 号，
  还有其它的不可变信息，如设备 inode 的主、次设备号、或符号链接内容。

* **AUTH**: 授予能力以获取与认证相关的元数据，
  特别是所有者、组和权限位。需要注意的是，
  完整的权限检查也许还要获取 ACL ，它是存在扩展属性（ xattrs ）里的。

* **LINK**: inode 的链接计数。

* **XATTR**: 访问或修改扩展属性的能力。另外，由于 ACL 定义存在扩展属性里，
  有时候检查权限还需访问它们。

* **FILE**: 这是个大头，允许客户端访问和修改数据。
  也涵盖了与文件数据相关的元数据——
  特别是尺寸、 mtime 、 atime 、 ctime 。


简写
----
.. Shorthand

需要注意的是，客户端日志里会紧凑地表达各个能力，例如：

::

        pAsLsXsFs

其中， p 表示 pin ，各大写字母对应位移值，而位移值后面的小写\
字母是真正赋予此位置的的能力。

锁状态和能力之间的关系
----------------------
.. The relation between the lock states and the capabilities

在 MDS 中，每个节点有四种不同的锁，分别是 simplelock 、
scatterlock 、 filelock 和 locallock 。每种锁都有几种不同的锁状态，
MDS 会根据锁状态向客户端发放能力。

在每种状态下， MDS Locker 都会尝试向客户端发布所有允许的能力，
即使有些能力是客户端不需要或不想要的，
因为某些情况下，预先发放能力可以减少延时。

如果只有一个客户端，它通常是所有节点的独行客户端（ loner client ）。
而在有多个客户端的情况下， MDS 会尝试根据客户端（需要 | 想要）的能力
为每个节点计算出一个独行客户端，
但通常会失败。独行客户端将始终获得所有能力。

filelock 会控制文件的、部分元数据的、和文件内容的访问权限。
元数据包括 **mtime** 、 **atime** 、 **size** 等。

* **Fs**: 客户端一旦拥有它，其他所有客户端的 **Fw** 都将被拒绝。

* **Fx**: 只有独行客户端才可拥有此能力。
  一旦锁状态转换为 LOCK_EXCL ，独行客户端就会被授予此能力，
  以及其他所有除 **Fl** 以外的能力.

* **Fr**: 一旦某个客户端拥有了它，说明\
  其他所有客户端的 **Fb** 能力都已经被撤销。

  如果客户端只要求读取文件，锁状态将直接转为 LOCK_SYNC 稳定状态。
  所有客户端都可以从权威 MDS 获得 **Fscrl** 能力，
  从副本 MDS 获得 **Fscr** 能力。

  如果多个客户端读出和写入同一个文件，
  那么锁状态将最终转换到 LOCK_MIX 稳定状态，
  所有客户端都可以获得权威 MDS 的 **Frwl** 能力和副本 MDS 的 **Fr** 能力。
  **Fcb** 能力不会授予所有客户端，客户端们将做同步读/写操作。

* **Fw**: 如果没有独行客户端，并且一旦某个客户端获得此能力，
  **Fsxcb** 能力将不能授予其他客户端。

  如果多个客户端读出和写入同一个文件，
  那么锁状态将最终转换到 LOCK_MIX 稳定状态，
  所有客户端都可以获取权威 MDS 的 **Frwl** 能力和来自副本 MDS 的 **Fr** 能力。
  **Fcb** 能力不会授予所有客户端，
  客户端们将做同步读/写操作。

* **Fc**: 该能力意味着客户端们可以缓存文件读出操作，
  应与 **Fr** 能力一起发放，
  只有在这种用例中才有意义。

  实际上，在一些稳定或临时过渡状态下，
  即使没有授予 **Fr** 能力，也会允许使用 **Fc** ，
  因为这样可以避免强制客户端丢弃完整缓存，
  例如在简单的文件大小扩展或截断用例中。

* **Fb**: 该能力意味着客户端可以缓冲文件写入操作，
  应与 **Fw** 能力一起发放，
  只有在这种情境下才有意义。

  实际上，在某些稳定或临时过渡状态下，
  即使不授予 **Fw** 能力，也会保留 **Fc** 能力，
  因为这样可以避免强制客户端丢弃脏缓冲区，
  例如在简单的文件大小扩展或截断用例中。

* **Fl**: 这个能力意味着客户端可以施行懒惰 IO 。
  LazyIO 放宽了 POSIX 语义。即使一个文件同时\
  被多个客户端上的多个应用程序打开，也允许缓冲读/写。
  应用程序它们自行负责管理缓存一致性。
