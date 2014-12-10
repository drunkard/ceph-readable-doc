===================
 Ceph 文件系统入门
===================

开始实践 :term:`Ceph 文件系统`\ 入门前，必须先完成\ `存储集群入门`_\ 。在管理\
节点上完成此入门。


准备工作
========

#. 确认下你的内核版本没问题，详情见\ `操作系统推荐`_\ 。 ::

	lsb_release -a
	uname -r

#. 在管理节点上，用 ``ceph-deploy`` 把 Ceph 安装到 ``ceph-client`` 节点上。 ::

	ceph-deploy install ceph-client

#. 确保 :term:`Ceph 存储集群`\ 在运行，且已进入 ``active + clean`` 状态。同\
   时，确保至少有一个 :term:`Ceph 元数据服务器`\ 在运行。 ::

	ceph -s [-m {monitor-ip-address}] [-k {path/to/ceph.client.admin.keyring}]


创建文件系统
============

你虽然已创建元数据服务器（\ `存储集群入门`_\ ），但如果你没有创建存储池和文件\
系统的话它是不会活动的，参见 :doc:`/cephfs/createfs` 。 ::

    ceph osd pool create cephfs_data <pg_num>
    ceph osd pool create cephfs_metadata <pg_num>
    ceph fs new <fs_name> cephfs_metadata cephfs_data


创建密钥文件
============

Ceph 存储集群默认启用认证，你应该有个包含密钥的配置文件（但不是密钥环本身）。\
用下述方法为某一用户申请密钥：

#. 找到与某用户对应的密钥环文件，例如： ::

	cat ceph.client.admin.keyring

#. 复制其密钥，随后将用于挂载 Ceph 文件系统。长相大概如下： ::

	[client.admin]
	   key = AQCj2YpRiAe6CxAA7/ETt7Hcl9IyxyYciVs47w==

#. 打开文本编辑器。

#. 把密钥粘帖进去，大概如此： ::

	AQCj2YpRiAe6CxAA7/ETt7Hcl9IyxyYciVs47w==

#. 保存文件，并把其用户名 ``name`` 作为一个属性（如 ``admin.secret`` ）。

#. 确保文件权限位对此用户可读，但对别人不可读。


内核驱动
========

把 CephFS 挂载为内核驱动。 ::

	sudo mkdir /mnt/mycephfs
	sudo mount -t ceph {ip-address-of-monitor}:6789:/ /mnt/mycephfs

Ceph 存储集群默认需要认证，所以挂载时需要指定用户名和\ `创建密钥文件`_\ 一节创\
建的密钥文件 ``secretfile`` ，例如： ::

	sudo mount -t ceph 192.168.0.1:6789:/ /mnt/mycephfs -o name=admin,secretfile=admin.secret

.. note:: 从管理节点而非服务器节点挂载 Ceph FS 文件系统，详情见 `FAQ`_ 。


用户空间文件系统
================

在用户空间（ FUSE ）下挂载 Ceph FS 文件系统。 ::

	sudo mkdir ~/mycephfs
	sudo ceph-fuse -m {ip-address-of-monitor}:6789 ~/mycephfs

Ceph 存储集群默认要求认证，需指定相应的密钥环文件，除非它在默认位置（即 \
``/etc/ceph`` ）： ::

	sudo ceph-fuse -k ./ceph.client.admin.keyring -m 192.168.0.1:6789 ~/mycephfs


额外信息
========

额外信息见 `Ceph FS`_ 。 Ceph FS 还不像 Ceph 块设备和 Ceph 对象存储那么稳定，\
如果遇到问题请参考\ `故障排除`_\ 。

.. _存储集群入门: ../quick-ceph-deploy
.. _Ceph FS: ../../cephfs/
.. _FAQ: http://wiki.ceph.com/03FAQs/01General_FAQ#How_Can_I_Give_Ceph_a_Try.3F
.. _故障排除: ../../cephfs/troubleshooting
.. _操作系统推荐: ../os-recommendations
