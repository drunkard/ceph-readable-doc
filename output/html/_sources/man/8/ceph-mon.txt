:orphan:

=================================
 ceph-mon -- ceph 监视器守护进程
=================================

.. program:: ceph-mon

提纲
====

| **ceph-mon** -i *monid* [ --mon-data *mondatapath* ]


描述
====

**ceph-mon** 是 Ceph 分布式存储集群的监视器守护进程。一或多个 **ceph-mon** \
例程可形成 Paxos 兼职议员集群，它们能为集群成员、配置和状态提供非常可靠、坚\
实的存储。

*mondatapath* 是个本地文件系统上的目录，存储着监视器数据。通常可用配置文件中\
的 ``mon data`` 选项指定。


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

.. option:: -c ceph.conf, --conf=ceph.conf

   启动时用 *ceph.conf* 配置文件而非默认的 ``/etc/ceph/ceph.conf`` \
   来确定启动时所需的监视器地址。

.. option:: --mkfs

   用种子信息初始化 ``mon data`` 目录，以形成并初始化 Ceph 文件系统或加入现\
   有监视器集群。以下三个信息是必需的：

   - 集群的 fsid 。可从 monmap （ ``--monmap <path>`` ）中获得或直接用 \
     ``--fsid <uuid>`` 指定。
   - 一连串监视器及其地址。这些监视器可来源于 monmap （ ``--monmap <path>`` \
     ）、 ``mon host`` 配置（在 *ceph.conf* 里或通过 ``-m host1,host2,...`` \
     ）或 *ceph.conf* 中的 ``mon addr`` 行。如果这个监视器将作为新 Ceph 集群\
     监视器法定人数的一部分，还必须放进初始列表中。匹配地址时，可以用 \
     ``public addr`` 或 ``public subnet`` 。
   - 监视器私钥 ``mon.`` 。必须包含在 ``--keyring <path>`` 所提供的密钥环内。

.. option:: --keyring

   指定 ``--mkfs`` 所需的密钥环。


使用范围
========

**ceph-mon** 是 Ceph 的一部分，这是个伸缩力强、开源、分布式的存储\
系统，更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`ceph <ceph>`\(8),
:doc:`ceph-mds <ceph-mds>`\(8),
:doc:`ceph-osd <ceph-osd>`\(8)
