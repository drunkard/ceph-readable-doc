===============================
 ceph -- Ceph 文件系统控制工具
===============================

.. program:: ceph

提纲
====

| **ceph** [ -m *monaddr* ] [ -w | *command* ... ]


描述
====

**ceph** 是与在线 Ceph 分布式存储系统的监视器集群通讯的控制工具。

有三种基本操作模式。

交互模式
--------

要进入交互模式，不加参数运行即可。 Control-d 或 'quit' 可退出。

观察模式
--------

观察模式会显示实时发生的集群状态变更，例如： ::

       ceph -w

命令行模式
----------

最后，要向监视器集群下达单个指令（并等待响应），可在命令行下输入子命令。


选项
====

.. option:: -i infile

   指定一个输入文件，它将作为载荷与命令一起传递给监视器集群。仅用于某些特定\
   的监视器命令。

.. option:: -o outfile

   把响应中监视器集群返回的载荷写入 outfile 文件。只有某些特定的监视器命令\
   （如 psd getmap ）会返回载荷。

.. option:: -c ceph.conf, --conf=ceph.conf

   用 ceph.conf 配置文件而非默认的 /etc/ceph/ceph.conf 来确定启动时所用的监\
   视器地址。

.. option:: -m monaddress[:port]

   连接到指定监视器（而不是通过 ceph.conf 查找）。


实例
====

获取当前 OSD 运行图的副本： ::

       ceph -m 1.2.3.4:6789 osd getmap -o osdmap

转储一份归置组（ PG ）状态： ::

       ceph pg dump -o pg.txt


监视器命令
==========

监视器集群能理解的命令汇总到了在线文档中：

       http://ceph.com/docs/master/rados/operations/control


使用范围
========

**ceph** 是 Ceph 分布式文件系统的一部分，更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`ceph <ceph>`\(8),
