:orphan:

========================================
ceph-create-keys -- ceph 密钥环生成工具
========================================

.. program:: ceph-create-keys

提纲
====

| **ceph-create-keys** [-h] [-v] [--cluster *name*] --id *id*


描述
====

:program:`ceph-create-keys` 工具用于在指定监视器可用时、生成自举引导密钥环。

它可以创建以下认证项目（或用户）：

``client.admin``

    以及用于客户端的密钥

``client.bootstrap-{osd, rgw, mds}``

    以及用于自举引导相应服务的密钥。

要罗列此集群内的所有用户： ::

	ceph auth list


选项
====

.. option:: --cluster

   集群名字，默认为 'ceph' 。

.. option:: -i, --id

   将会起来的 ceph-mon 进程的唯一标识符，在它加入法定人数前， \
   **ceph-create-keys** 会一直等着。

.. option:: -v, --verbose

   显示得更详细些。


使用范围
========

:program:`ceph-create-keys` 是 Ceph 的一部分，这是个伸缩力强、开源、分布式的\
存储系统，更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`ceph <ceph>`\(8)
