====================
 欢迎来到 Ceph 世界
====================

Ceph 在一个统一的系统中同时提供了\ **对象、块、和文件存储功能**\ 。

.. warning::

   :ref:`如果这是你第一次使用 Ceph ，请阅读《 Ceph 开发者指南》中的\
   “基本工作流程”页面，了解如何向 Ceph 项目贡献。
   （点击本段任意位置，可阅读《Ceph 开发者指南》里的
   “基本工作流程”页面。） <basic workflow dev guide>`.

.. note::

   :ref:`如果您想提交文档变更，但不知道如何开始，
   请阅读“为 Ceph 写作文档”页面。
   (点击本段中的任意位置可阅读“为 Ceph 写作文档”页面。） <documenting_ceph>`

.. container:: columns-3

   .. container:: column

      .. raw:: html

          <h3>Ceph 对象存储</h3>

      - REST 风格的接口
      - 与 S3 和 Swift 兼容的 API
      - S3 风格的子域
      - 统一的 S3/Swift 命名空间
      - 用户管理
      - 利用率跟踪
      - 条带化对象
      - 云解决方案集成
      - 多站点部署
      - 多站点复制

   .. container:: column

      .. raw:: html

          <h3>Ceph Block Device</h3>

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
      - 灾难恢复（多站点异步复制）

   .. container:: column

      .. raw:: html

          <h3>Ceph 文件系统</h3>

      - 与 POSIX 兼容的语义
      - 元数据独立于数据
      - 动态重均衡
      - 子目录快照
      - 可配置的条带化
      - 有内核驱动支持
      - 有用户空间驱动支持
      - 可作为 NFS/CIFS 部署
      - 可用于 Hadoop （取代 HDFS ）

.. container:: columns-3

   .. container:: column

      详情见 `Ceph 对象存储`_\ 。

   .. container:: column

      详情见 `Ceph 块设备`_\ 。

   .. container:: column

      详情见 `Ceph 文件系统`_\ 。

它可靠性高、管理简单，并且是开源软件。
Ceph 的强大可以改变您公司的 IT 基础架构和海量数据管理能力。
想试试 Ceph 的话看\ `入门`_\ 手册；
想深入理解可以看\ `体系结构`_\ 一节。



.. _Ceph 对象存储: radosgw
.. _Ceph 块设备: rbd
.. _Ceph 文件系统: cephfs
.. _入门: install
.. _体系结构: architecture

.. toctree::
   :maxdepth: 3
   :hidden:

   start/index
   install/index
   cephadm/index
   rados/index
   cephfs/index
   rbd/index
   radosgw/index
   mgr/index
   mgr/dashboard
   monitoring/index
   api/index
   architecture
   开发者指南 <dev/developer_guide/index>
   dev/internals
   governance
   foundation
   ceph-volume/index
   crimson/crimson
   releases/general
   releases/index
   security/index
   hardware-monitoring/index
   Ceph 术语 <glossary>
   Tracing <jaegertracing/index>
   中文版翻译资源 <translation_cn/index>
