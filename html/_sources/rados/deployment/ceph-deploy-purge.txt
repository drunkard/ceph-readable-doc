============
 清除一主机
============

拆除 Ceph 守护进程并卸载软件包后，此主机上可能还有集群中的数据， ``purge`` 和 \
``purgedata`` 命令提供了一种清理主机的简便方法。


purgedata
=========

如果只想清除 ``/var/lib/ceph`` 下的数据、并保留 Ceph 安装包，可以用 \
``purgedata`` 命令： ::

	ceph-deploy purgedata {hostname} [{hostname} ...]


purge
=====

要清理掉 ``/var/lib/ceph`` 下的所有数据、并卸载 Ceph 软件包，用 ``purge`` 命\
令。 ::

	ceph-deploy purge {hostname} [{hostname} ...]
