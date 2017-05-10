============================
 用户空间挂载 Ceph 文件系统
============================

从用户空间（ FUSE ）挂载一 Ceph 文件系统前，确保客户端主机有一\
份 Ceph 配置文件副本、和具备 Ceph 元数据服务器能力的密钥环。

#. 在客户端主机上，把监视器主机上的 Ceph 配置文件拷贝到
   ``/etc/ceph/`` 目录下。 ::

	sudo mkdir -p /etc/ceph
	sudo scp {user}@{server-machine}:/etc/ceph/ceph.conf /etc/ceph/ceph.conf

#. 在客户端主机上，把监视器主机上的 Ceph 密钥环拷贝到
   ``/etc/ceph`` 目录下。 ::

	sudo scp {user}@{server-machine}:/etc/ceph/ceph.keyring /etc/ceph/ceph.keyring

#. 确保客户端机器上的 Ceph 配置文件和密钥环都有合适的权限位，如
   ``chmod 644`` 。

``cephx`` 如何配置请参考 `CEPHX 配置参考`_\ 。

要把 Ceph 文件系统挂载为用户空间文件系统，可以用 ``ceph-fuse``
命令，例如： ::

	sudo mkdir /home/usernname/cephfs
	sudo ceph-fuse -m 192.168.0.1:6789 /home/username/cephfs

如果你的文件系统不止一个，挂载时可用 ``--client_mds_namespace``
参数指定要挂载哪个文件系统，或者在 ``ceph.conf`` 里配置
``client_mds_namespace`` 选项。

详情见 `ceph-fuse`_ 。

要自动挂载 ceph-fuse ，你可以把它写入系统的 fstab_ 文件。另外，\
还得有 systemd 的单元文件 ``ceph-fuse@.service`` 和
``ceph-fuse.target`` ，这些单元文件声明了 ``ceph-fuse`` 所需的\
默认依赖和推荐的执行上下文。用 ceph-fuse 挂载到 ``/mnt`` 的实\
例如下： ::

	sudo systemctl start ceph-fuse@/mnt.service

永久挂载点可以这样配置： ::

	sudo systemctl enable ceph-fuse@/mnt.service


.. _ceph-fuse: ../../man/8/ceph-fuse/
.. _fstab: ./fstab
.. _CEPHX 配置参考: ../../rados/configuration/auth-config-ref
