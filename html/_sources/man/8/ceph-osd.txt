:orphan:

===================================
 ceph-osd -- ceph 对象存储守护进程
===================================

.. program:: ceph-osd

提纲
====

| **ceph-osd** -i *osdnum* [ --osd-data *datapath* ] [ --osd-journal
  *journal* ] [ --mkfs ] [ --mkjournal ] [ --mkkey ]


描述
====

**ceph-osd** 是 Ceph 分布式对象存储系统的对象存储守护进程。它负责把对象存储\
到本地文件系统，并使之通过网络可访问。

*datapath* 参数应该是 btrfs 文件系统上保存对象数据的一个目录。日志是可选的，\
只有它位于非数据盘的低延时设备上（理想中应该是 NVRAM ）时才会达到最佳性能。


选项
====

.. option:: -f, --foreground

   前台：启动后不要作为守护进程，仍在前台运行。不要生成 PID 文件。\
   通过 :doc:`ceph-run <ceph-run>`\(8) 运行时此选项有用。

.. option:: -d

   调试模式：类似 ``-f`` ，还会把所有日志发到了标准错误。

.. option:: --setuser userorgid

   启动后设置 UID 。如果指定的是用户名，会查询用户记录以获取 UID \
   及其 GID ，同时设置 GID ，除非还指定了 --setgroup 选项。

.. option:: --setgroup grouporgid

   启动后设置 GID 。如果指定的是组名，会查询组记录以获取 GID 。

.. option:: --osd-data osddata

   把对象存储在 *osddata* 。

.. option:: --osd-journal journal

   把日志更新到 *journal* 。

.. option:: --mkfs

   创建空的对象仓库。如果定义了日志，也同时初始化。

.. option:: --mkkey

   生成新的私钥。通常和 ``--mkfs`` 一起使用，因为与 \
   :doc:`ceph-authtool <ceph-authtool>`\(8) 生成密钥相比此选项更便捷。

.. option:: --mkjournal

   创建适用于已有对象仓库的新日志文件。常用于因硬盘或文件系统故障时导致的日\
   志设备或文件损坏。

.. option:: --flush-journal

   把日志刷回永久存储，它运行于前台，这样你就能知道它何时完成。适用于你想调\
   整日志尺寸或以其他方式销毁它时：此功能可保证不丢数据。

.. option:: --get-cluster-fsid

   打印集群的 fsid (uuid) 然后退出。

.. option:: --get-osd-fsid

   打印 OSD 的 fsid 然后退出。 OSD 的 UUID 是在创建文件系统（ --mkfs ）时生\
   成的，而且对这个特定的 OSD 例程来说是惟一的。

.. option:: --get-journal-fsid

   打印日志的 UUID 。在新建文件系统（ --mkfs ）时设置了日志 fsid 以与 OSD 相配。

.. option:: -c ceph.conf, --conf=ceph.conf

   用 *ceph.conf* 配置文件而非默认的 ``/etc/ceph/ceph.conf`` 来确定运行时配置。

.. option:: -m monaddress[:port]

   连接到指定监视器（而非到 ``ceph.conf`` 里找）。


使用范围
========

**ceph-osd** 是 Ceph 的一部分，这是个伸缩力强、开源、分布式的存储系统，\
更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`ceph <ceph>`\(8),
:doc:`ceph-mds <ceph-mds>`\(8),
:doc:`ceph-mon <ceph-mon>`\(8),
:doc:`ceph-authtool <ceph-authtool>`\(8)
