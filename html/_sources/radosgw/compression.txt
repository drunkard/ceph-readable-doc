.. _Compression:

======
 压缩
======

.. versionadded:: Kraken

Ceph 对象网关支持在服务器端压缩上传的对象，可以用任何已有的压\
缩插件实现。


配置
====

压缩功能可在域的归置目标那里启用，执行 ``radosgw-admin zone placement modify``
命令时加 ``--compression=<type>`` 选项即可。

压缩类型 ``type`` 应该换成写入新对象数据时要用的压缩插件名。每\
个压缩的对象都记录了使用的压缩插件，所以更改此配置不影响已有对\
象的解压缩，更不会强制再次压缩已有对象。

这个压缩选项对所有新上传到桶（桶使用了这个归置目标）内的对象起\
作用。 ``type`` 设置为空字符串或 ``none`` 时禁用压缩。

例如： ::

  $ radosgw-admin zone placement modify --rgw-zone=default --placement-id=default-placement --compression=zlib
  {
  ...
      "placement_pools": [
          {
              "key": "default-placement",
              "val": {
                  "index_pool": "default.rgw.buckets.index",
                  "data_pool": "default.rgw.buckets.data",
                  "data_extra_pool": "default.rgw.buckets.non-ec",
                  "index_type": 0,
                  "compression": "zlib"
              }
          }
      ],
  ...
  }

.. note:: 如果你没做过\ `多站配置`_\ ，那么此命令会创建默认域
   ``default`` 。


统计信息
========

当前，所有命令和 API 都是基于未压缩数据上报对象和桶的大小，某\
个桶的压缩统计信息可通过 ``bucket stats`` 命令查看： ::

  $ radosgw-admin bucket stats --bucket=<name>
  {
  ...
      "usage": {
          "rgw.main": {
              "size": 1075028,
              "size_actual": 1331200,
              "size_utilized": 592035,
              "size_kb": 1050,
              "size_kb_actual": 1300,
              "size_kb_utilized": 579,
              "num_objects": 104
          }
      },
  ...
  }

``size_utilized`` 和 ``size_kb_utilized`` 字段表示已压缩数据的\
总尺寸，单位分别是字节和千字节。


.. _`多站配置`: ../multisite
