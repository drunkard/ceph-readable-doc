==============================
 用内核驱动挂载 Ceph 文件系统
==============================

要挂载 Ceph 文件系统，如果你知道监视器 IP 地址可以用 ``mount``
命令、或者用 ``mount.ceph`` 工具来自动解析监视器 IP 地址。例\
如： ::

	sudo mkdir /mnt/mycephfs
	sudo mount -t ceph 192.168.0.1:6789:/ /mnt/mycephfs

要挂载启用了 ``cephx`` 认证的 Ceph 文件系统，必须指定用户名、\
密钥。 ::

	sudo mount -t ceph 192.168.0.1:6789:/ /mnt/mycephfs -o name=admin,secret=AQATSKdNGBnwLhAAnNDKnH65FmVKpXZJVasUeQ==

前述用法会把密码遗留在 Bash 历史里，更安全的途径是从文件读密\
码。例如： ::

	sudo mount -t ceph 192.168.0.1:6789:/ /mnt/mycephfs -o name=admin,secretfile=/etc/ceph/admin.secret

如果你的文件系统不止一个，还需用 ``mds_namespace`` 选项指定挂\
载哪一个，例如 ``-o mds_namespace=myfs`` 。

关于 cephx 参见\ `用户管理`_\ 。

要卸载 Ceph 文件系统，可以用 ``unmount`` 命令，例如： ::

	sudo umount /mnt/mycephfs

.. tip:: 执行此命令前确保你不在此文件系统目录下。

详情见 `mount.ceph`_ 。


.. _mount.ceph: ../../man/8/mount.ceph/
.. _用户管理: ../../rados/operations/user-management/
