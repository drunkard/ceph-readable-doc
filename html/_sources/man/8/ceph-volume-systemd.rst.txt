:orphan:

=====================================================
 ceph-volume-systemd -- systemd ceph-volume 辅助工具
=====================================================

.. program:: ceph-volume-systemd

提纲
====

| **ceph-volume-systemd** *systemd instance name*


描述
====

:program:`ceph-volume-systemd` 是一个 systemd 辅助工具，它会\
持续接收来自 systemd 单元的输入（动态创建的），这样才能激活
OSD 。

它会把输入翻译为一条指向 ceph-volume 的系统调用，只用于激活\
目的。


实例
====

其输入是 ``systemd instance name`` （在 systemd 单元里用 ``%i``
表示），其格式如下： ::

    <ceph-volume subcommand>-<extra metadata>

对于 ``lvm`` ，一个调用大概是： ::

    /usr/bin/ceph-volume-systemd lvm-0-8715BEB4-15C5-49DE-BA6F-401086EC7B41

它会按如下方法依次调用 ``ceph-volume``::

    ceph-volume lvm trigger  0-8715BEB4-15C5-49DE-BA6F-401086EC7B41

任何其它子命令都需要实现一个 ``trigger`` 命令，以处理这种格式\
的其余元数据。


使用范围
========

:program:`ceph-volume-systemd` 是 Ceph 的一部分，这是个伸缩力\
强、开源、分布式的存储系统，更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`ceph-osd <ceph-osd>`\(8),
