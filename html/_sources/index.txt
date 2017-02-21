====================
 欢迎来到 Ceph 世界
====================

Ceph 独一无二地在一个统一的系统中同时提供了\ **对象、块、和文件存储功能**\ 。

.. raw:: html

	<style type="text/css">div.body h3{margin:5px 0px 0px 0px;}</style>
	<table cellpadding="10"><colgroup><col width="33%"><col width="33%"><col width="33%"></colgroup><tbody valign="top"><tr><td><h3>CEPH 对象存储</h3>

- REST 风格的接口
- 与 S3 和 Swift 兼容的 API
- S3 风格的子域
- 统一的 S3/Swift 命名空间
- 用户管理
- 利用率跟踪
- 条带化对象
- 云解决方案集成
- 多站点部署
- 灾难恢复

.. raw:: html 

	</td><td><h3>Ceph 块设备</h3>


- 瘦接口支持
- 映像尺寸最大 16EB
- 条带化可定制
- 内存缓存
- 快照
- 写时复制克隆
- 支持内核级驱动
- 支持 KVM 和 libvirt
- 可作为云解决方案的后端
- 增量备份

.. raw:: html 

	</td><td><h3>Ceph 文件系统</h3>

- 与 POSIX 兼容的语义
- 元数据独立于数据
- 动态重均衡
- 子目录快照
- 可配置的条带化
- 有内核驱动支持
- 有用户空间驱动支持
- 可作为 NFS/CIFS 部署
- 可用于 Hadoop （取代 HDFS ）

.. raw:: html

	</td></tr><tr><td>

详情见 `Ceph 对象存储`_\ 。

.. raw:: html

	</td><td>

详情见 `Ceph 块设备`_\ 。

.. raw:: html

	</td><td>

详情见 `Ceph 文件系统`_\ 。

.. raw::	html 

	</td></tr></tbody></table>

它可靠性高、管理简单，并且是开源软件。 Ceph 的强大可以改变您公司的 IT 基础架\
构和海量数据管理能力。想试试 Ceph 的话看\ `入门`_\ 手册；想深入理解可以看\ \
`体系结构`_\ 一节。


.. _Ceph 对象存储: radosgw
.. _Ceph 块设备: rbd/rbd
.. _Ceph 文件系统: cephfs
.. _入门: start
.. _体系结构: architecture

.. toctree::
   :maxdepth: 1
   :hidden:

   start/intro
   start/index
   install/index
   rados/index
   cephfs/index
   rbd/rbd
   radosgw/index
   api/index
   architecture
   开发文档 <dev/index>
   release-notes
   releases
   Ceph 术语 <glossary>
