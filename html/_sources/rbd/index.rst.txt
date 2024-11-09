=============
 Ceph 块设备
=============

.. index:: Ceph Block Device; introduction

块是一个字节序列（通常是 512 字节）。
在媒体（像硬盘、SSD 、 CD 、软盘、甚至磁带）上存入数据时，
基于块的存储接口是一种成熟且通用的方式。
块设备接口的普遍性使它完美适配海量存储系统，包括 Ceph 。

Ceph 块设备是瘦接口、大小可调且数据\
条带化到 Ceph 集群内的多个 OSD 。
Ceph 块设备能够均衡多种 :abbr:`RADOS
(Reliable Autonomic Distributed Object Store, 可靠自主的分布式对象存储库)`
能力，如快照、复制和强一致性， Ceph 的 \
块存储客户端通过内核模块或 librbd 库与 Ceph 集群通讯。

.. ditaa::

            +------------------------+ +------------------------+
            |     Kernel Module      | |        librbd          |
            +------------------------+-+------------------------+
            |                   RADOS Protocol                  |
            +------------------------+-+------------------------+
            |          OSDs          | |        Monitors        |
            +------------------------+ +------------------------+

.. note:: 内核模块可使用 Linux 页缓存。对基于 ``librbd`` 的\
   应用程序， Ceph 可提供 `RBD 缓存`_\ 。

Ceph 块设备靠超强的可扩展性提供了高性能，如向\ `内核模块`_\ 、或向
abbr:`KVM (kernel virtual machines)` （如 `QEMU`_ 、依赖
libvirt 和 QEMU 的 `OpenStack`_ 和 `CloudStack`_ 云计算系统都\
可与 Ceph 块设备集成）。你可以用同一个集群同时运营
:ref:`Ceph RADOS 网关 <object-gateway>`\ 、
:ref:`CephFS 文件系统 <ceph-file-system>`\ 、和 Ceph 块设备。

.. important:: 要使用 Ceph 块设备，你必须有一个在运行的
   Ceph 集群。

.. toctree::
	:maxdepth: 1

	基本命令 <rados-rbd-cmds>

.. toctree::
        :maxdepth: 2

        运维 <rbd-operations>

.. toctree::
	:maxdepth: 2

        对接 <rbd-integrations>

.. toctree::
	:maxdepth: 2

	手册页 <man/index>

.. toctree::
	:maxdepth: 2

	APIs <api/index>


.. _RBD 缓存: ./rbd-config-ref/
.. _内核模块: ./rbd-ko/
.. _QEMU: ./qemu-rbd/
.. _OpenStack: ./rbd-openstack
.. _OpenNebula: https://docs.opennebula.io/stable/open_cluster_deployment/storage_setup/ceph_ds.html
.. _CloudStack: ./rbd-cloudstack
