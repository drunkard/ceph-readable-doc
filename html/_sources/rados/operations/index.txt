==========
 集群运维
==========

.. raw:: html

	<table><colgroup><col width="50%"><col width="50%"></colgroup><tbody valign="top"><tr><td><h3>高级运维</h3>

高级集群操作主要包括用 ``ceph`` 服务管理脚本启动、停止、重启集群，和集群健康状态检\
查、监控和操作集群。

.. toctree::
	:maxdepth: 1 

	operating
	monitoring
	monitoring-osd-pg
	user-management

.. raw:: html 

	</td><td><h3>数据归置</h3>

你的集群开始运行后，就可以尝试数据归置了。 Ceph 是 PB 级数据存储集群，它用 CRUSH \
算法、靠存储池和归置组在集群内分布数据。

.. toctree::
	:maxdepth: 1

	data-placement
	pools
	erasure-code
	cache-tiering
	placement-groups
	crush-map



.. raw:: html

	</td></tr><tr><td><h3>低级运维</h3>

低级集群运维包括启动、停止、重启集群内的某个具体守护进程；更改某守护进程或子系统配\
置；增加或拆除守护进程。低级运维还经常遇到扩展、缩减 Ceph 集群，以及更换老旧、或损\
坏的硬件。

.. toctree::
	:maxdepth: 1

	add-or-rm-osds
	add-or-rm-mons
	命令参考 <control>



.. raw:: html

	</td><td><h3>故障排除</h3>

Ceph 仍在积极开发中，所以你可能碰到一些问题，需要评估 Ceph 配置文件、并修改日志和调\
试选项来纠正它。

.. toctree::
	:maxdepth: 1 

	../troubleshooting/community
	../troubleshooting/troubleshooting-mon
	../troubleshooting/troubleshooting-osd
	../troubleshooting/troubleshooting-pg
	../troubleshooting/log-and-debug
	../troubleshooting/cpu-profiling
	../troubleshooting/memory-profiling




.. raw:: html

	</td></tr></tbody></table>
