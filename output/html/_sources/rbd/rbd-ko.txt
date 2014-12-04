==============
 内核模块操作
==============

.. index:: Ceph Block Device; kernel module

.. important:: 要用内核模块操作，必须有一个运行着的 Ceph 集群。


获取映像列表
============

要挂载映像设备，先罗列映像。
::

	rbd list


映射块设备
==========

用 ``rbd`` 把映像名映射为内核模块。必须指定映像名、存储池名、和用户名。若 RBD 内核\
模块尚未加载， ``rbd`` 命令会自动加载。
::

	sudo rbd map {image-name} --pool {pool-name} --id {user-name}

例如：
::

	sudo rbd map --pool rbd myimage --id admin

如果你启用了 `cephx`_ 认证，你必须提供密钥，可以从密钥环或文件获取。
::

	sudo rbd map --pool rbd myimage --id admin --keyring /path/to/keyring
	sudo rbd map --pool rbd myimage --id admin --keyfile /path/to/file


查看已映射块设备
================

可以用 ``rbd`` 命令的 ``showmapped`` 选项查看映射为内核模块的块设备映像。
::

	rbd showmapped


取消块设备映射
==============

要取消块设备映射，用 ``rbd`` 命令、指定 ``unmap`` 选项和设备名（即为方便起见使用的\
同名块设备映像）。
::

	sudo rbd unmap /dev/rbd/{poolname}/{imagename}

例如：
::

	sudo rbd unmap /dev/rbd/rbd/foo


.. _cephx: ../../rados/operations/authentication/
