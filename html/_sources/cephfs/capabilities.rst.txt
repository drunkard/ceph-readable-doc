.. Capabilities in CephFS

===================
 CephFS 支持的能力
===================

一个客户端想要操作索引节点时，它会向 MDS 发起多种查询， MDS 会\
授予此客户端一系列\ *能力（ capabilities ）*\ ，有了这些能力，\
客户端就有权限操作索引节点了。与其它网络文件系统（如 NFS 或 SMB
）相比，一个主要的区别在于这些授予的能力非常粗糙，而且它允许多\
个客户端同时持有同一索引节点的不同能力。


.. Types of Capabilities

能力的种类
----------

有几种“常规”能力位，以下是定义。

.. code-block:: cpp

        /* generic cap bits */
        #define CEPH_CAP_GSHARED     1  /* client can reads (s) */
        #define CEPH_CAP_GEXCL       2  /* client can read and update (x) */
        #define CEPH_CAP_GCACHE      4  /* (file) client can cache reads (c) */
        #define CEPH_CAP_GRD         8  /* (file) client can read (r) */
        #define CEPH_CAP_GWR        16  /* (file) client can write (w) */
        #define CEPH_CAP_GBUFFER    32  /* (file) client can buffer writes (b) */
        #define CEPH_CAP_GWREXTEND  64  /* (file) client can extend EOF (a) */
        #define CEPH_CAP_GLAZYIO   128  /* (file) client can perform lazy io (l) */

这些能力是通过二进制数字的位翻转定义的，用以表示与某一索引节点\
的数据或元数据相关的能力：

.. code-block:: cpp

        /* per-lock shift */
        #define CEPH_CAP_SAUTH      2 /* A */
        #define CEPH_CAP_SLINK      4 /* L */
        #define CEPH_CAP_SXATTR     6 /* X */
        #define CEPH_CAP_SFILE      8 /* F */

.. note:: 译者注：下面这段没完全理解，也许翻译的有问题，先附上原文。

:译文: 然而，以前只有某些常规能力授予了其中的一些“位移”，特别是，只有
       FILE 位移多于前两位。
:原文: Only certain generic cap types are ever granted for some of those "shifts",
       however. In particular, only the FILE shift ever has more than the first two
       bits.

::

        | AUTH | LINK | XATTR | FILE
        2      4      6       8

从以上定义我们得到一个常数，它是用各个位值移位到相应的位（用\
变量表示）生成的：

.. code-block:: cpp

        #define CEPH_CAP_AUTH_SHARED  (CEPH_CAP_GSHARED  << CEPH_CAP_SAUTH)

然后，这些位就可以通过或操作产生位掩码（ bitmask ），用以表示\
一系列能力。

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


.. Abilities granted by each cap

各个 cap 授予的能力
-------------------

这就解释完了能力是如何授予的（和通讯的），以下重要的位说明了允\
许客户端干什么：

* PIN: 只是把 inode 插入内存。这足以让客户端取得 inode 号，还\
  有其它的不可变信息，如设备 inode 的主、次设备号、或符号链接\
  内容。

* AUTH: 授予能力以获取与认证相关的元数据，特别是所有者、组和权\
  限位。需要注意的是，完整的权限检查也许还要获取 ACL ，它是存\
  在扩展属性（ xattrs ）里的。

* LINK: inode 的链接计数。

* XATTR: 访问或修改扩展属性的能力。另外，由于 ACL 定义存在扩展\
  属性里，有时候检查权限还需访问它们。

* FILE: 这是个大头，允许客户端访问和修改数据。也涵盖了与文件数\
  据相关的元数据——特别是尺寸、 mtime 、 atime 、 ctime 。


.. Shorthand

简写
----

需要注意的是，客户端日志里会紧凑地表达各个能力，例如： ::

        pAsLsXsFs

其中， p 表示 pin ，各大写字母对应位移值，而位移值后面的小写\
字母是真正赋予此位置的的能力。
