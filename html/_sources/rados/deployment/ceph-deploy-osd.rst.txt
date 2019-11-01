===============
 增加/删除 OSD
===============

新增和拆除 Ceph 的 OSD 进程相比其它两种要多几步。 OSD 守护进程\
把数据写入磁盘和日志，所以你得相应地提供一 OSD 数据盘和日志分\
区路径（这是最常见的配置，但你可以按需调整）。

从 Ceph v0.60 起， Ceph 支持 ``dm-crypt`` 加密的硬盘，在准备
OSD 时你可以用 ``--dm-crypt`` 参数告诉 ``ceph-deploy`` 你想用\
加密功能。也可以用 ``--dmcrypt-key-dir`` 参数指定 ``dm-crypt``
加密密钥的位置。

在投建一个大型集群前，你应该测试各种驱动器配置来衡量其吞吐量。\
详情见\ `数据存储`_\ 。


.. List Disks

罗列磁盘
========

执行下列命令罗列一节点上的磁盘： ::

	ceph-deploy disk list {node-name [node-name]...}


.. Zap Disks

擦净磁盘
========

用下列命令擦净（删除分区表）磁盘，以用于 Ceph ： ::

	ceph-deploy disk zap {osd-server-name}:{disk-name}
	ceph-deploy disk zap osdserver1:sdb

.. important:: 这会删除所有数据。


.. Create OSDs

创建 OSD
========
创建集群、安装 Ceph 软件包、收集密钥完成后你就可以创建 OSD 并\
把它们部署到 OSD 节点了。如果你想敲定某磁盘或擦净它，以便用作
OSD ，参见\ `罗列磁盘`_\ 和\ `擦净磁盘`_\ 。 ::

	ceph-deploy osd create --data {data-disk} {node-name}

例如： ::

	ceph-deploy osd create --data /dev/ssd osd-server1

对于 bluestore （默认的）本例假设一个 Ceph OSD 守护进程使用\
一个磁盘。也支持 Filestore ，此时除了 ``--filestore`` 之外\
还得用 ``--journal`` 来定义远程主机上的日志设备。

.. note:: 在一个节点运行多个 OSD 守护进程、且多个 OSD 守护进程\
   共享一个日志分区时，你应该考虑整个节点的最小 CRUSH 故障域，\
   因为如果这个 SSD 坏了，所有用其做日志的 OSD 守护进程也会失效。


.. List OSDs

罗列 OSD
========
要罗列一节点上部署的所有 OSD ，用此命令： :: 

  ceph-deploy osd list {node-name}


.. Destroy OSDs

拆除 OSD
=========

.. note:: 稍后完成。手动过程见\ `删除 OSD`_ 。

.. 用以下命令拆除 OSD ：
.. ::

..	ceph-deploy osd destroy {node-name}:{path-to-disk}[:{path/to/journal}]

.. 拆除 OSD 将把它在集群中的状态改为 ``down`` 且 ``out`` 。

.. _数据存储: ../../../start/hardware-recommendations#data-storage
.. _删除 OSD: ../../operations/add-or-rm-osds#removing-osds-manual
