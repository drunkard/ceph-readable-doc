======
 配置
======

Ceph 作为集群时可以包含数千个对象存储设备（ OSD ）。最简系统至少需要二个 OSD 做数\
据复制。要配置 OSD 集群，你得把配置写入配置文件。 Ceph 对很多选项提供了默认值，你可\
以在配置文件里覆盖它们；另外，你可以使用命令行工具修改运行时配置。

Ceph 启动时要激活三类守护进程：

- ``ceph-mon`` （必备）
- ``ceph-osd`` （必备）
- ``ceph-mds`` （ cephfs 必备）

各进程、守护进程或工具都会读取配置文件。一个进程可能需要不止一个守护进程例程的信息\
（\ *即*\ 多个上下文）；一个守护进程或工具只有关于单个守护进程例程的信息（单上下文）。

.. note:: 试用时 Ceph 可运行在单机上。


.. raw:: html

	<table cellpadding="10"><colgroup><col width="50%"><col width="50%"></colgroup><tbody valign="top"><tr><td><h3>配置对象存储</h3>

下列是通用对象存储的配置：

.. toctree::
   :maxdepth: 1

   硬盘和文件系统 <filesystem-recommendations>
   ceph-conf


.. raw:: html 

	</td><td><h3>参考</h3>

参考下列文档优化集群性能：

.. toctree::
	:maxdepth: 1

	网络选项 <network-config-ref>
	认证选项 <auth-config-ref>
	监视器选项 <mon-config-ref>
	心跳选项 <mon-osd-interaction>
	OSD 选项 <osd-config-ref>
	filestore 选项 <filestore-config-ref>
	键/值存储选项 <keyvaluestore-config-ref>
	日志选项 <journal-ref>
	存储池、归置组和 CRUSH 选项 <pool-pg-config-ref.rst>
	消息传递选项 <ms-ref>
	常规选项 <general-config-ref>

   
.. raw:: html

	</td></tr></tbody></table>
