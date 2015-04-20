===============
 Ceph 存储集群
===============

所有 Ceph 部署都始于 :term:`Ceph 存储集群`\ 。基于 :abbr:`RADOS (Reliable \
Autonomic Distributed Object Store ，可靠自主的分布式对象存储)` 的 Ceph 对象\
存储集群包括两类守护进程：term:`对象存储守护进程`\ （ OSD ）把存储节点上的数\
据存储为对象； term:`Ceph 监视器`\ （ MON ）维护集群运行图的主拷贝。一个 \
Ceph 集群可以包含数千个存储节点，最简系统至少需要一个监视器和两个 OSD 才能\
做到数据复制。

Ceph 文件系统、 Ceph 对象存储、和 Ceph 块设备从 Ceph 存储集群读出和写入数据。

.. raw:: html

	<style type="text/css">div.body h3{margin:5px 0px 0px 0px;}</style>
	<table cellpadding="10"><colgroup><col width="33%"><col width="33%"><col width="33%"></colgroup><tbody valign="top"><tr><td><h3>配置和部署</h3>

Ceph 存储集群的某些配置选项是必要的，但大多数都有默认值。典型部署是通过部署工具定义\
集群、并启动监视器的，关于 ``ceph-deploy`` 的详情见\ `部署`_\ 。

.. toctree::
	:maxdepth: 2

	配置 <configuration/index>
	部署 <deployment/index>

.. raw:: html 

	</td><td><h3>运维</h3>

部署后就可以开始操作 Ceph 集群了。

.. toctree::
	:maxdepth: 2


	运维 <operations/index>

.. toctree::
	:maxdepth: 1

	手册页 <man/index>


.. toctree:: 
	:hidden:

	troubleshooting/index

.. raw:: html 

	</td><td><h3>应用编程接口（ API ）</h3>

大多数 Ceph 部署都使用了 `Ceph 块设备`_\ 、 `Ceph 对象存储`_\ 和/或 `Ceph 文件系\
统`_\ 。也可以开发程序直接与 Ceph 对象存储对接。

.. toctree::
	:maxdepth: 2

	APIs <api/index>

.. raw:: html

	</td></tr></tbody></table>

.. _Ceph 块设备: ../rbd/rbd
.. _Ceph 文件系统: ../cephfs/
.. _Ceph 对象存储: ../radosgw/
.. _部署: ../rados/deployment/
