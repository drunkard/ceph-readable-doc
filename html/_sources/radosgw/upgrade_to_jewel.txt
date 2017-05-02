===============================================================
 把 RGW 升级到 Jewel 版本 10.2.0 、 10.2.1 、 10.2.2 和 10.2.3
===============================================================

.. versionadded:: Jewel

把 :term:`Ceph 对象网关`\ 升级到 jewel 的早期版本（包括 10.2.3
）会有问题，本文档给出了必要的恢复步骤。

:term:`Ceph 对象网关`\ 不支持多版本混合部署。


备份旧配置文件
==============

rados mkpool .rgw.root.backup
rados cppool .rgw.root .rgw.root.backup


`rgw region root pool` 选项不是默认值
=====================================

如果已有的多站配置的 ``rgw region root pool`` 选项不是默认值，\
那么新的存储池选项 ``rgw zonegroup root pool`` 、
``rgw period root pool`` 和 ``rgw realm root pool`` 也应该一并\
更改。


.. _Fix confgiuration after upgrade:

升级后更正配置
==============

停止集群内运行的所有 :term:`Ceph 对象网关`\ 。

运行下面这些命令： ::

    $ rados rmpool .rgw.root

    $ radosgw-admin zonegroup get --rgw-zonegroup=default | sed 's/"id":.*/"id": "default",/g' | sed 's/"master_zone.*/"master_zone":"default",/g' > default-zg.json

    $ raodsgw-admin zone get --zone-id=default > default-zone.json

    $ radosgw-admin realm create --rgw-realm=myrealm

    $ radosgw-admin zonegroup set --rgw-zonegroup=default --default < default-zg.json

    $ radosgw-admin zone set --rgw-zone=default --default < default-zone.json

    $ radosgw-admin period update --commit

启动集群内的所有 :term:`Ceph 对象网关`\ 。

