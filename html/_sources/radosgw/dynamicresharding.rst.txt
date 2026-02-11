.. _rgw_dynamic_bucket_index_resharding:

========================
 RGW 动态的桶索引重分片
========================

.. versionadded:: Luminous

桶索引对象内条目太多时会导致性能问题，
此问题可以通过对桶索引进行重分片（ resharding ）来解决。
到 Luminous 版为止，更改桶的分片数量（重分片， resharding ）
必须离线操作，还要同时禁用 RGW 服务。
从 Luminous 起 Ceph 支持桶的在线重分片。

Each bucket index shard can handle its entries efficiently up until
reaching a certain threshold number. If this threshold is exceeded the
system can suffer from performance issues. The dynamic resharding
feature detects this situation and automatically increases the number
of shards used by a bucket's index, resulting in a reduction of the
number of entries in each shard. This process is transparent to the
user. Writes to the target bucket can be blocked briefly during
resharding process, but reads are not.

By default dynamic bucket index resharding can only increase the
number of bucket index shards to 1999, although this upper-bound is a
configuration parameter (see `Configuration`_ below). When
possible, the process chooses a prime number of shards in order to
spread the number of entries across the bucket index
shards more evenly.

Detection of resharding opportunities runs as a background process
that periodically scans all buckets. A bucket that requires resharding
is added to a queue. A thread runs in the background and processes the
queueued resharding tasks one at a time.

Starting with Tentacle, dynamic resharding has the ability to reduce
the number of shards. Once the condition allowing reduction is noted,
there is a time delay before it will actually be executed, in case the
number of objects increases in the near future. The goal of the delay
is to avoid thrashing where resharding keeps getting re-invoked on
buckets that fluctuate in numbers of objects.


多站
====
.. Multisite

在 Ceph 的 Reef 版本之前， Ceph 对象网关（RGW）不支持在多站环境下的动态重分片。
关于动态重分片，见 RGW 多站文档里的\ :ref:`重分片 <feature_resharding>`\ 。


配置选项
========
.. Configuration

.. confval:: rgw_dynamic_resharding
.. confval:: rgw_max_objs_per_shard
.. confval:: rgw_max_dynamic_shards
.. confval:: rgw_dynamic_resharding_may_reduce
.. confval:: rgw_dynamic_resharding_reduction_wait
.. confval:: rgw_reshard_bucket_lock_duration
.. confval:: rgw_reshard_thread_interval
.. confval:: rgw_reshard_num_logs
.. confval:: rgw_reshard_progress_judge_interval
.. confval:: rgw_reshard_progress_judge_ratio


管理命令
========
.. Admin commands

把一个桶加进重分片队列
----------------------
.. Add a bucket to the resharding queue

.. prompt:: bash #

   radosgw-admin reshard add --bucket <bucket_name> --num-shards <new number of shards>


罗列重分片队列
--------------
.. List resharding queue

.. prompt:: bash #

   radosgw-admin reshard list


处理重分片队列里的任务
----------------------
.. Process tasks on the resharding queue

.. prompt:: bash #

   radosgw-admin reshard process


桶重分片状态
------------
.. Bucket resharding status

.. prompt:: bash #

   radosgw-admin reshard status --bucket <bucket_name>

以下输出是每个分片里有 3 个属性（ ``reshard_status``, ``new_bucket_instance_id``, ``num_shards`` ）的 JSON 数组。

例如，在动态重分片的每个阶段，输出分别如下：

#. 重分片之前：

   ::

     [
       {
           "reshard_status": "not-resharding",
           "new_bucket_instance_id": "",
           "num_shards": -1
       }
     ]

#. 重分片期间：

   ::

     [
       {
           "reshard_status": "in-progress",
           "new_bucket_instance_id": "1179f470-2ebf-4630-8ec3-c9922da887fd.8652.1",
           "num_shards": 2
       },
       {
           "reshard_status": "in-progress",
           "new_bucket_instance_id": "1179f470-2ebf-4630-8ec3-c9922da887fd.8652.1",
           "num_shards": 2
       }
     ]

#. 重分片完成之后：

   ::

     [
       {
           "reshard_status": "not-resharding",
           "new_bucket_instance_id": "",
           "num_shards": -1
       },
       {
           "reshard_status": "not-resharding",
           "new_bucket_instance_id": "",
           "num_shards": -1
       }
     ]


取消挂着的桶重分片操作
----------------------
.. Cancel pending bucket resharding

.. note:: 一旦桶重分片任务从开始的 ``not-resharding`` 状态\
   转变为 ``in-progress`` 状态后，此操作就无法取消了。

.. prompt:: bash #

   radosgw-admin reshard cancel --bucket <bucket_name>


手动执行即时桶重分片
--------------------
.. Manual immediate bucket resharding

.. prompt:: bash #

   radosgw-admin bucket reshard --bucket <bucket_name> --num-shards <new number of shards>

When choosing a number of shards, the administrator should keep a
number of items in mind. Ideally the administrator is aiming for no
more than 100000 entries per shard, now and through some future point
in time.

Additionally, bucket index shards that are prime numbers tend to work
better in evenly distributing bucket index entries across the
shards. For example, 7001 bucket index shards is better than 7000
since the former is prime. A variety of web sites have lists of prime
numbers; search for "list of prime numbers" withy your favorite web
search engine to locate some web sites.


设置桶的最少分片数
------------------
.. Setting a Bucket's Minimum Number of Shards

.. prompt:: bash #

   radosgw-admin bucket set-min-shards --bucket <bucket_name> --num-shards <min number of shards>

Since dynamic resharding can now reduce the number of shards,
administrators may want to prevent the number of shards from becoming
too low, for example if the expect the number of objects to increase
in the future. This command allows administrators to set a per-bucket
minimum. This does not, however, prevent administrators from manually
resharding to a lower number of shards.


故障排除
========
.. Troubleshooting

Clusters prior to Luminous 12.2.11 and Mimic 13.2.5 left behind stale bucket
instance entries, which were not automatically cleaned up. The issue also affected
LifeCycle policies, which were not applied to resharded buckets anymore. Both of
these issues can be worked around using a couple of radosgw-admin commands.


掉队例程管理
------------
.. Stale instance management

罗列出集群里已经准备好、可以被清理掉的过时例程。

.. prompt:: bash #

   radosgw-admin reshard stale-instances list

清理掉集群里的过时例程。

.. prompt:: bash #

   radosgw-admin reshard stale-instances rm

.. note:: 不应该在多站集群上清理过时例程。


生命周期修复
------------
.. Lifecycle fixes

For clusters that had resharded instances, it is highly likely that the old
lifecycle processes would have flagged and deleted lifecycle processing as the
bucket instance changed during a reshard. While this is fixed for newer clusters
(from Mimic 13.2.6 and Luminous 12.2.12), older buckets that had lifecycle policies and
that have undergone resharding will have to be manually fixed.

完成修复的命令是：

.. prompt:: bash #

    radosgw-admin lc reshard fix --bucket {bucketname}

As a convenience wrapper, if the ``--bucket`` argument is dropped then this
command will try and fix lifecycle policies for all the buckets in the cluster.


对象逾期管理器修复
------------------
.. Object Expirer fixes

Objects subject to Swift object expiration on older clusters may have
been dropped from the log pool and never deleted after the bucket was
resharded. This would happen if their expiration time was before the
cluster was upgraded, but if their expiration was after the upgrade
the objects would be correctly handled. To manage these expire-stale
objects, radosgw-admin provides two subcommands.

罗列：

.. prompt:: bash #

   radosgw-admin objects expire-stale list --bucket {bucketname}

Displays a list of object names and expiration times in JSON format.

删除：

.. prompt:: bash #

   radosgw-admin objects expire-stale rm --bucket {bucketname}

它会初始化这些对象的删除、以 JSON 格式显示对象名列表、过期时间、和删除状态。
