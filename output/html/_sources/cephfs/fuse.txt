============================
 用户空间挂载 Ceph 文件系统
============================

Ceph v0.55 及后续版本默认开启了 ``cephx`` 认证。从用户空间（ FUSE ）挂载一 Ceph \
文件系统前，确保客户端主机有一份 Ceph 配置副本、和具备 Ceph 元数据服务器能力的密钥\
环。

#. 在客户端主机上，把监视器主机上的 Ceph 配置文件拷贝到 ``/etc/ceph/`` 目录\
   下。 ::

	sudo mkdir -p /etc/ceph
	sudo scp {user}@{server-machine}:/etc/ceph/ceph.conf /etc/ceph/ceph.conf

#. 在客户端主机上，把监视器主机上的 Ceph 密钥环拷贝到 ``/etc/ceph`` 目录下。 ::

	sudo scp {user}@{server-machine}:/etc/ceph/ceph.keyring /etc/ceph/ceph.keyring

#. 确保客户端机器上的 Ceph 配置文件和密钥环都有合适的权限位，如 ``chmod 644`` 。

``cephx`` 如何配置请参考 `CEPHX 配置参考`_\ 。

要把 Ceph 文件系统挂载为用户空间文件系统，可以用 ``ceph-fuse`` 命令，例如： ::

	sudo mkdir /home/usernname/cephfs
	sudo ceph-fuse -m 192.168.0.1:6789 /home/username/cephfs

详情见 `ceph-fuse`_ 。


.. _ceph-fuse: ../../man/8/ceph-fuse/
.. _CEPHX 配置参考: ../../rados/configuration/auth-config-ref
