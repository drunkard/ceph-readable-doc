.. _Mount Ceph FS in your File Systems Table:

===============
 用 fstab 挂载
===============

如果你从文件系统表挂载， Ceph 文件系统将在启动时自动挂载。


.. _Kernel Driver:

内核驱动
========

要用内核驱动挂载 Ceph FS ，按下列格式添加到 ``/etc/fstab`` ： ::

	{ipaddress}:{port}:/ {mount}/{mountpoint} {filesystem-name}	[name=username,secret=secretkey|secretfile=/path/to/secretfile],[{mount.options}]

例如： ::

	10.10.10.10:6789:/     /mnt/ceph    ceph    name=admin,secretfile=/etc/ceph/secret.key,noatime,_netdev    0       2

.. important:: 启用了认证时， ``name`` 及 ``secret`` 或 ``secretfile`` 选项必须加。

详情见\ `用户管理`_\ 。


FUSE
====

要在用户空间挂载 Ceph 文件系统，按如下加入 ``/etc/fstab`` ： ::

        #DEVICE PATH       TYPE      OPTIONS
        none    /mnt/ceph  fuse.ceph ceph.id={user-ID}[,ceph.conf={path/to/conf.conf}],_netdev,defaults  0 0

例如： ::

        none    /mnt/ceph  fuse.ceph ceph.id=myuser,_netdev,defaults  0 0
        none    /mnt/ceph  fuse.ceph ceph.id=myuser,ceph.conf=/etc/ceph/foo.conf,_netdev,defaults  0 0

确保填上了 ID （如 ``admin`` ，不是 ``client.admin`` ）。你可\
以用此方法加上 ``ceph-fuse`` 的其它可用选项。

详情见\ `用户管理`_\ 。


.. _用户管理: ../../rados/operations/user-management/
