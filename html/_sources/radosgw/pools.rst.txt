.. Pools

========
 存储池
========

Ceph 对象网关要用多个存储池来满足多种存储需求，它们罗列在 Zone
对象内（见 ``radosgw-admin zone get`` ）。系统会自动创建名为
``default`` 的单个域，相应地存储池名字以 ``default.rgw.`` 打\
头；然而\ `多站配置`_\ 会有多个域。


.. Tuning

调整
====

``radosgw`` 的首次操作涉及一个不存在的域存储池时，它会用
``osd pool default pg num`` 和 ``osd pool default pgp num`` 的\
默认值创建那个存储池。这些默认值对于某些存储池来说足够了，但是\
其它的（特别是 ``placement_pools`` 内罗列的、用于桶索引和数据\
的存储池）就需要额外调整了。我们建议用
`Ceph 归置组的单存储池计算器 <http://ceph.com/pgcalc/>`__\ 给\
这些存储池计算个合适的归置组数量。存储池创建细节请参考\ `存储池
<http://docs.ceph.com/docs/master/rados/operations/pools/#pools>`__\ 。


.. Pool Namespaces

存储池命名空间
==============

.. versionadded:: Luminous

一个域的存储池名字应该遵循命名惯例 ``{zone-name}.pool-name`` ，\
例如，名为 ``us-east`` 的域需要创建如下存储池：

-  ``.rgw.root``

-  ``us-east.rgw.control``

-  ``us-east.rgw.meta``

-  ``us-east.rgw.log``

-  ``us-east.rgw.buckets.index``

-  ``us-east.rgw.buckets.data``

域定义还多罗列了几个存储池，但是其中很多通过 rados 命名空间功\
能合并了。例如，下面所有的存储池条目共用了 ``us-east.rgw.meta``
存储池的命名空间： ::

    "user_keys_pool": "us-east.rgw.meta:users.keys",
    "user_email_pool": "us-east.rgw.meta:users.email",
    "user_swift_pool": "us-east.rgw.meta:users.swift",
    "user_uid_pool": "us-east.rgw.meta:users.uid",

.. _`多站配置`: ../multisite
