===================
 Ceph 对象存储速读
===================

要使用 :term:`Ceph 对象存储`\ 快速入门，你必须先完成\ `存储集群快速入门`_\ \
指南，并且至少有一个 :term:`RGW` 在线。


配置新的 RGW 例程
=================

按照\ `存储集群快速入门`_\ 创建的 :term:`RGW` 例程会用嵌入式的 CivetWeb \
网页服务器运行。 ``ceph-deploy`` 会创建此例程、并用默认参数自动启动它。

关于 :term:`RGW` 例程的管理，请参考 `RGW 管理手册`_\ 。

其他细节可以在 `Ceph 对象网关的配置`_\ 手册里找到，但是特定于 Apache 的步\
骤不再需要了。

.. note:: 用 ``ceph-deploy`` 部署 RGW 、并 CivetWeb 网页服务器替代 Apache \
   是从 **Hammer** 版起才有的新功能。


.. _存储集群快速入门: ../quick-ceph-deploy
.. _RGW 管理手册: ../../radosgw/admin
.. _Ceph 对象网关的配置: ../../radosgw/config
