===============
 用 fstab 挂载
===============

如果你从文件系统表挂载， Ceph 文件系统将在启动时自动挂载。


内核驱动
========

要用内核驱动挂载 Ceph FS ，按下列格式添加到 ``/etc/fstab`` ： ::

	{ipaddress}:{port}:/ {mount}/{mountpoint} {filesystem-name}	[name=username,secret=secretkey|secretfile=/path/to/secretfile],[{mount.options}]

例如： ::

	10.10.10.10:6789:/     /mnt/ceph    ceph    name=admin,secretfile=/etc/ceph/secret.key,noatime    0       2

.. important:: 启用了认证时， ``name`` 及 ``secret`` 或 ``secretfile`` 选项必须加。

详情见\ `认证`_\ 。


FUSE
====

要在用户空间挂载 Ceph 文件系统，按如下加入 ``/etc/fstab`` ： ::

	#DEVICE                                  PATH         TYPE      OPTIONS
	id={user-ID}[,conf={path/to/conf.conf}] /mount/path  fuse.ceph defaults 0 0

例如： ::

	id=admin  /mnt/ceph  fuse.ceph defaults 0 0
	id=myuser,conf=/etc/ceph/cluster.conf  /mnt/ceph2  fuse.ceph defaults 0 0

``DEVICE`` 字段是逗号分隔的一系列选项，确保填上了 ID （如 ``admin`` ，不是 \
``client.admin`` ）。你可以用此方法把任何 ``ceph-fuse`` 接受的选项加上。

详情见\ `认证`_\ 。


.. _认证: ../../rados/operations/authentication/
