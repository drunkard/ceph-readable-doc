============
 块设备入门
============

要实践本手册，你必须先完成\ `存储集群入门`_ ，并确保 :term:`Ceph 存储集群`\ 处\
于 ``active + clean`` 状态，这样才能使用 :term:`Ceph 块设备`\ 。

.. note:: Ceph 块设备也叫 :term:`RBD` 或 :term:`RADOS` 块设备。


.. ditaa::
           /------------------\         /----------------\
           |    Admin Node    |         |   ceph–client  |
           |                  +-------->+ cCCC           |
           |    ceph–deploy   |         |      ceph      |
           \------------------/         \----------------/


你可以在虚拟机上运行 ``ceph-client`` 节点，但是不能在与 Ceph 存储集群（除非它\
们也用 VM ）相同的物理节点上执行下列步骤。详情见 `FAQ`_ 。


安装 Ceph
=========

#. 确认下你的内核版本没问题，详情见\ `操作系统推荐`_\ 。
   ::

	lsb_release -a
	uname -r

#. 在管理节点上，通过 ``ceph-deploy`` 把 Ceph 安装到 ``ceph-client`` 节点。
   ::

	ceph-deploy install ceph-client

#. 在管理节点上，用 ``ceph-deploy`` 把 Ceph 配置文件和 \
   ``ceph.client.admin.keyring`` 拷贝到 ``ceph-client`` 。
   ::

	ceph-deploy admin ceph-client

   ``ceph-deploy`` 工具会把密钥环复制到 ``/etc/ceph`` 目录，要确保此密钥环文件\
   可读（如 ``sudo chmod +r /etc/ceph/ceph.client.admin.keyring`` ）。


配置块设备
==========

#. 在 ``ceph-client`` 节点上创建一个块设备映像。
   ::

	rbd create foo --size 4096 [-m {mon-IP}] [-k /path/to/ceph.client.admin.keyring]

#. 在 ``ceph-client`` 节点上，把映像映射为块设备。
   ::

	sudo rbd map foo --name client.admin [-m {mon-IP}] [-k /path/to/ceph.client.admin.keyring]

#. 在 ``ceph-client`` 节点上，创建文件系统后就可以使用了。
   ::

	sudo mkfs.ext4 -m0 /dev/rbd/rbd/foo

   此命令可能耗时较长。

#. 在 ``ceph-client`` 节点上挂载此文件系统。
   ::

	sudo mkdir /mnt/ceph-block-device
	sudo mount /dev/rbd/rbd/foo /mnt/ceph-block-device
	cd /mnt/ceph-block-device


详情见\ `块设备`_ 。

.. _存储集群入门: ../quick-ceph-deploy
.. _块设备: ../../rbd/rbd
.. _FAQ: http://wiki.ceph.com/FAQs/How_Can_I_Give_Ceph_a_Try%3F
.. _操作系统推荐: ../os-recommendations
