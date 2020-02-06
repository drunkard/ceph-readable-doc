=============
 Ceph 块设备
=============

.. index:: Ceph Block Device; introduction

块是一个字节序列（例如，一个 512 字节的一块数据），基于块的存储\
接口是最常见的存储数据方法，它们基于旋转媒体，像硬盘、 CD 、软\
盘、甚至传统的 9 磁道磁带。无处不在的块设备接口使虚拟块设备成为\
与 Ceph 这样的海量存储系统交互的理想之选。

Ceph 块设备是瘦接口、大小可调且数据条带化到集群内的多个 OSD 。 \
Ceph 块设备均衡多个 \
:abbr:`RADOS (Reliable Autonomic Distributed Object Store)`
能力，如快照、复制和一致性， Ceph 的 \
:abbr:`RADOS (Reliable Autonomic Distributed Object Store)`
块设备（ RBD ）用内核模块或 librbd 库与各个 OSD 交互。

.. ditaa::  +------------------------+ +------------------------+
            |     Kernel Module      | |        librbd          |
            +------------------------+-+------------------------+
            |                   RADOS Protocol                  |
            +------------------------+-+------------------------+
            |          OSDs          | |        Monitors        |
            +------------------------+ +------------------------+

.. note:: 内核模块可使用 Linux 页缓存。对基于 ``librbd`` 的应用\
   程序， Ceph 可提供 `RBD 缓存`_\ 。

Ceph 块设备靠无限伸缩性提供了高性能，如向\ `内核模块`_\ 、或向
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
.. _CloudStack: ./rbd-cloudstack
