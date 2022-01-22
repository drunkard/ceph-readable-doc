.. _rados-index:

===============
 Ceph 存储集群
===============

所有 Ceph 部署都始于 :term:`Ceph 存储集群`\ 。基于
:abbr:`RADOS (Reliable Autonomic Distributed Object Store ，\
可靠自主的分布式对象存储)` 的 Ceph 存储集群包括几类守护进程：

 1. :term:`Ceph OSD 守护进程`\ （ OSD ）把存储节点上的数据存储为对象；
 2. :term:`Ceph 监视器`\ （ MON ）维护集群运行图的主副本；
 3. :term:`Ceph 管理器` 守护进程

一个 Ceph 集群可以包含数千个存储节点，最简系统至少\
需要一个监视器和两个 OSD 守护进程才能做到数据复制。

Ceph 文件系统、 Ceph 对象存储、和 Ceph 块设备从 Ceph 存储集群\
读出和写入数据。


.. container:: columns-3

   .. container:: column

      .. raw:: html

          <h3>配置和部署</h3>

      Ceph 存储集群有少量配置选项是必要的，但大多数都有默认值。
      典型部署是通过部署工具定义集群、并启动监视器的。
      详情见\ :ref:`cephadm`\ 。

      .. toctree::
         :maxdepth: 2

         配置 <configuration/index>

   .. container:: column

      .. raw:: html

          <h3>运维、操作</h3>

      部署完一个 Ceph 存储集群后就可以操作它了。

      .. toctree::
         :maxdepth: 2

         运维 <operations/index>

      .. toctree::
         :maxdepth: 1

	     手册页 <man/index>

      .. toctree::
         :hidden:

         troubleshooting/index

   .. container:: column

      .. raw:: html

          <h3>应用编程接口（ API ）</h3>

      大多数 Ceph 部署都使用了 `Ceph 块设备`_\ 、
      `Ceph 对象存储`_\ 和/或 `Ceph 文件系统`_\ 。也可以\
      开发程序直接与 Ceph 对象存储对接。

      .. toctree::
         :maxdepth: 2

         APIs <api/index>


.. _Ceph 块设备: ../rbd/
.. _Ceph 文件系统: ../cephfs/
.. _Ceph 对象存储: ../radosgw/
.. _部署: ../cephadm
