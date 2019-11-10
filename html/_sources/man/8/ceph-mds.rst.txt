:orphan:

=======================================
 ceph-mds -- Ceph 元数据服务器守护进程
=======================================

.. program:: ceph-mds

提纲
====

| **ceph-mds** -i <*ID*> [flags]


描述
====

**ceph-mds** 是 Ceph 分布式文件系统的元数据服务器守护进程。一\
或多个 ceph-mds 例程协作着管理文件系统的命名空间、协调到共享
OSD 集群的访问。

各 ceph-mds 守护进程例程都应该有惟一的名字，此名用于在
ceph.conf 里标识例程。

一旦守护进程启动，监视器集群会给它分配一个逻辑机架，或把它放入\
一个候补存储池用于接替其它崩溃的进程。一些特定选项会导致其它行\
为。


选项
====

.. option:: -f, --foreground

   前台：启动后不要进入后台（在前台运行），不生成 pid 文件。通\
   过 :doc:`ceph-run <ceph-run>`\(8) 调用时有用。

.. option:: -d

   调试模式：像 ``-f`` ，但它也把所有输出日志发到了标准错误。

.. option:: --setuser userorgid

   启动后设置 UID 。如果指定的是用户名，会查询用户记录以获取 UID \
   及其 GID ，同时设置 GID ，除非还指定了 --setgroup 选项。

.. option:: --setgroup grouporgid

   启动后设置 GID 。如果指定的是组名，会查询组记录以获取 GID 。

.. option:: -c ceph.conf, --conf=ceph.conf

   在启动时用 *ceph.conf* 而非默认的 ``/etc/ceph/ceph.conf`` 来确\
   定监视器地址。

.. option:: -m monaddress[:port]

   连接到指定监视器（而非通过 ``ceph.conf`` 查找）。

.. option:: --id/-i ID

   指定 MDS 名字的 ID 部分。


使用范围
========

**ceph-mds** 是 Ceph 的一部分，这是个伸缩力强、开源、分布式的\
存储系统，更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`ceph <ceph>`\(8),
:doc:`ceph-mon <ceph-mon>`\(8),
:doc:`ceph-osd <ceph-osd>`\(8)
