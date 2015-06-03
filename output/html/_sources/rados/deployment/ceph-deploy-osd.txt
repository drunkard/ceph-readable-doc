===============
 增加/删除 OSD
===============

新增和拆除 Ceph 的 OSD 进程相比其它两种要多几步。 OSD 守护进程把数据写入磁盘和日\
志，所以你得相应地提供一 OSD 数据盘和日志分区路径（这是最常见的配置，但你可以按需\
调整）。

从 Ceph v0.60 起， Ceph 支持 ``dm-crypt`` 加密的硬盘，在准备 OSD 时你可以用 \
``--dm-crypt`` 参数告诉 ``ceph-deploy`` 你想用加密功能。也可以用 \
``--dmcrypt-key-dir`` 参数指定 ``dm-crypt`` 加密密钥的位置。

在投建一个大型集群前，你应该测试各种驱动器配置来衡量其吞吐量。详情见\ `数据存储`_\ 。


列举磁盘
========

执行下列命令列举一节点上的磁盘： ::

	ceph-deploy disk list {node-name [node-name]...}


擦净磁盘
========

用下列命令擦净（删除分区表）磁盘，以用于 Ceph ： ::

	ceph-deploy disk zap {osd-server-name}:{disk-name}
	ceph-deploy disk zap osdserver1:sdb

.. important:: 这会删除所有数据。


准备 OSD
=========

创建集群、安装 Ceph 软件包、收集密钥完成后你就可以准备 OSD 并把它们部署到 OSD 节点\
了。如果你想确认某磁盘或擦净它，参见\ `列举磁盘`_\ 和\ `擦净磁盘`_\ 。 ::

	ceph-deploy osd prepare {node-name}:{data-disk}[:{journal-disk}]
	ceph-deploy osd prepare osdserver1:sdb:/dev/ssd
	ceph-deploy osd prepare osdserver1:sdc:/dev/ssd

``prepare`` 命令只准备 OSD 。在大多数操作系统中，硬盘分区创建后，不用 \
``activate`` 命令也会自动执行 ``activate`` 阶段（通过 Ceph 的 ``udev`` 规\
则）。详情见\ `激活 OSD`_\ 。

前例假定一个硬盘只会用于一个 OSD 守护进程，以及一个到 SSD 日志分区的路径。我们建议\
把日志存储于另外的驱动器以最优化性能；你也可以指定一单独的驱动器用于日志（也许比较昂\
贵）、或者把日志放到 OSD 数据盘（不建议，因为它有损性能）。前例中我们把日志存储于分\
好区的固态硬盘。

.. note:: 在一个节点运行多个 OSD 守护进程、且多个 OSD 守护进程共享一个日志分区时，\
   你应该考虑整个节点的最小 CRUSH 故障域，因为如果这个 SSD 坏了，所有用其做日志的 \
   OSD 守护进程也会失效。


激活 OSD
========

准备好 OSD 后，可以用下列命令激活它。 ::

	ceph-deploy osd activate {node-name}:{data-disk-partition}[:{journal-disk-partition}]
	ceph-deploy osd activate osdserver1:/dev/sdb1:/dev/ssd1
	ceph-deploy osd activate osdserver1:/dev/sdc1:/dev/ssd2

``activate`` 命令会让 OSD 进入 ``up`` 且 ``in`` 状态，此命令所用路径和 \
``prepare`` 相同。


创建 OSD
=========

你可以用 ``create`` 命令一次完成准备 OSD 、部署到 OSD 节点、并激活它。 ``create`` \
命令是依次执行 ``prepare`` 和 ``activate`` 命令的捷径。 ::

	ceph-deploy osd create {node-name}:{disk}[:{path/to/journal}]
	ceph-deploy osd create osdserver1:sdb:/dev/ssd1

.. 列举 OSD
.. ========

.. 要列举一节点上的所有 OSD，用此命令：
.. :: 

..	ceph-deploy osd list {node-name}


拆除 OSD
=========

.. note:: 稍后完成。手动过程见\ `删除 OSD`_ 。

.. 用以下命令拆除 OSD ：
.. ::

..	ceph-deploy osd destroy {node-name}:{path-to-disk}[:{path/to/journal}]

.. 拆除 OSD 将把它在集群中的状态改为 ``down`` 且 ``out`` 。

.. _数据存储: ../../../start/hardware-recommendations#data-storage
.. _删除 OSD: ../../operations/add-or-rm-osds#removing-osds-manual
