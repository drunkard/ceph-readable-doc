.. _rgw_dynamic_bucket_index_resharding:

========================
 RGW 动态的桶索引重分片
========================

.. versionadded:: Luminous

巨大的桶索引会导致性能问题，为了解决此问题，我们引进了桶索引分片（机制）。
到 Luminous 版为止，更改桶的分片数量（重分片， resharding ）需要离线操作。
从 Luminous 起才支持桶的在线重分片。

Each bucket index shard can handle its entries efficiently up until
reaching a certain threshold number of entries. If this threshold is
exceeded the system can suffer from performance issues. The dynamic
resharding feature detects this situation and automatically increases
the number of shards used by the bucket index, resulting in a
reduction of the number of entries in each bucket index shard. This
process is transparent to the user. Write I/Os to the target bucket
are blocked and read I/Os are not during resharding process.

By default dynamic bucket index resharding can only increase the
number of bucket index shards to 1999, although this upper-bound is a
configuration parameter (see Configuration below). When
possible, the process chooses a prime number of bucket index shards to
spread the number of bucket index entries across the bucket index
shards more evenly.

The detection process runs in a background process that periodically
scans all the buckets. A bucket that requires resharding is added to
the resharding queue and will be scheduled to be resharded later. The
reshard thread runs in the background and execute the scheduled
resharding tasks, one at a time.


多站
====
.. Multisite

动态重分片功能还不能在多站环境下使用。


配置选项
========
.. Configuration

启用、禁用动态的桶索引重分片功能：

- ``rgw_dynamic_resharding``:  true/false, 默认： true.

用于控制重分片进程的配置选项：

- ``rgw_max_objs_per_shard``: 每个桶索引分片内的对象数最大多少时就触发重分片，默认： 100000 对象。

- ``rgw_max_dynamic_shards``: 动态桶索引重分片可以达到的最大分片数，默认： 1999

- ``rgw_reshard_bucket_lock_duration``: 重分片时锁住桶对象的时长，按秒算，默认： 360 秒（即6分钟）

- ``rgw_reshard_thread_interval``: 两轮重分片队列处理的最大间隔时间，按秒算，默认： 600 秒（即10分钟）

- ``rgw_reshard_num_logs``: 重分片队列的分片数量，默认： 16


管理命令
========
.. Admin commands

把一个桶加进重分片队列
----------------------
.. Add a bucket to the resharding queue

::

   # radosgw-admin reshard add --bucket <bucket_name> --num-shards <new number of shards>


罗列重分片队列
--------------
.. List resharding queue

::

   # radosgw-admin reshard list


处理重分片队列里的任务
----------------------
.. Process tasks on the resharding queue

::

   # radosgw-admin reshard process


桶重分片状态
------------
.. Bucket resharding status

::

   # radosgw-admin reshard status --bucket <bucket_name>

以下输出是每个分片里有 3 个对象的 json 数组（reshard_status, new_bucket_instance_id, num_shards）。

例如，在动态重分片的不同阶段，输出分别如下：

``1. 重分片之前：``
::

  [
    {
        "reshard_status": "not-resharding",
        "new_bucket_instance_id": "",
        "num_shards": -1
    }
  ]

``2. 重分片期间：``
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

``3, 重分片完成之后：``
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

注意：正在进行着的桶重分片操作无法取消。 ::

   # radosgw-admin reshard cancel --bucket <bucket_name>


手动执行即时桶重分片
--------------------
.. Manual immediate bucket resharding

::

   # radosgw-admin bucket reshard --bucket <bucket_name> --num-shards <new number of shards>

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

罗列出集群里已经准备好、可以被清理掉的过时例程。 ::

   # radosgw-admin reshard stale-instances list

清理掉集群里的过时例程。注意：
只能在单站点集群上进行这种例程清理。 ::

   # radosgw-admin reshard stale-instances rm


生命周期修复
------------
.. Lifecycle fixes

For clusters that had resharded instances, it is highly likely that the old
lifecycle processes would have flagged and deleted lifecycle processing as the
bucket instance changed during a reshard. While this is fixed for newer clusters
(from Mimic 13.2.6 and Luminous 12.2.12), older buckets that had lifecycle policies and
that have undergone resharding will have to be manually fixed.

完成修复的命令是： ::

    # radosgw-admin lc reshard fix --bucket {bucketname}

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

罗列： ::

   # radosgw-admin objects expire-stale list --bucket {bucketname}

Displays a list of object names and expiration times in JSON format.

删除：

::

   # radosgw-admin objects expire-stale rm --bucket {bucketname}

它会初始化这些对象的删除、以 JSON 格式显示对象名列表、过期时间、和删除状态。
