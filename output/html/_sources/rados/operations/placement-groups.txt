========
 归置组
========


.. _preselection:

预定义 pg_num
=============

When creating a new pool with::

        ceph osd pool set {pool-name} pg_num

it is mandatory to choose the value of ``pg_num`` because it cannot be
calculated automatically. Here are a few values commonly used:

- Less than 5 OSDs set ``pg_num`` to 128

- Between 5 and 10 OSDs set ``pg_num`` to 512

- Between 10 and 50 OSDs set ``pg_num`` to 4096

- If you have more than 50 OSDs, you need to understand the tradeoffs
  and how to calculate the ``pg_num`` value by yourself

As the number of OSDs increases, chosing the right value for pg_num
becomes more important because it has a significant influence on the
behavior of the cluster as well as the durability of the data when
something goes wrong (i.e. the probability that a catastrophic event
leads to data loss).


归置组是如何使用的？
====================

存储池内的归置组（ PG ）把对象汇聚在一起，因为跟踪每一个对象的位置及其元数据需要大\
量计算——即一个拥有数百万对象的系统，不可能在对象这一级追踪位置。

.. ditaa::
           /-----\  /-----\  /-----\  /-----\  /-----\
           | obj |  | obj |  | obj |  | obj |  | obj |
           \-----/  \-----/  \-----/  \-----/  \-----/
              |        |        |        |        |
              +--------+--------+        +---+----+
              |                              |
              v                              v
   +-----------------------+      +-----------------------+
   |  Placement Group #1   |      |  Placement Group #2   |
   |                       |      |                       |
   +-----------------------+      +-----------------------+
               |                              |
               +------------------------------+
                             |
                             v
                  +-----------------------+
                  |        Pool           |
                  |                       |
                  +-----------------------+

Placement groups are invisible to the Ceph user: the CRUSH algorithm
determines in which placement group the object will be
placed. Although CRUSH is a deterministic function using the object
name as a parameter, there is no way to force an object into a given
placement group.

The object's contents within a placement group are stored in a set of
OSDs. For instance, in a replicated pool of size two, each placement
group will store objects on two OSDs, as shown below.

.. ditaa::

   +-----------------------+      +-----------------------+
   |  Placement Group #1   |      |  Placement Group #2   |
   |                       |      |                       |
   +-----------------------+      +-----------------------+
        |             |               |             |
        v             v               v             v
   /----------\  /----------\    /----------\  /----------\
   |          |  |          |    |          |  |          |
   |  OSD #1  |  |  OSD #2  |    |  OSD #2  |  |  OSD #3  |
   |          |  |          |    |          |  |          |
   \----------/  \----------/    \----------/  \----------/


Should OSD #2 fail, another will be assigned to Placement Group #1 and
will be filled with copies of all objects in OSD #1. If the pool size
is changed from two to three, an additional OSD will be assigned to
the placement group and will receive copies of all objects in the
placement group.

Placement groups do not own the OSD, they share it with other
placement groups from the same pool or even other pools. If OSD #2
fails, the Placement Group #2 will also have to restore copies of
objects, using OSD #3.

When the number of placement groups increases, the new placement
groups will be assigned OSDs. The result of the CRUSH function will
also change and some objects from the former placement groups will be
copied over to the new Placement Groups and removed from the old ones.


Placement Groups Tradeoffs
==========================

Data durability and even distribution among all OSDs call for more
placement groups but their number should be reduced to the minimum to
save CPU and memory.


.. _data durability:

Data durability
---------------

After an OSD fails, the risk of data loss increases until the data it
contained is fully recovered. Let's imagine a scenario that causes
permanent data loss in a single placement group:

- The OSD fails and all copies of the object it contains are lost.
  For all objects within the placement group the number of replica
  suddently drops from three to two.

- Ceph starts recovery for this placement group by chosing a new OSD
  to re-create the third copy of all objects.

- Another OSD, within the same placement group, fails before the new
  OSD is fully populated with the third copy. Some objects will then
  only have one surviving copies.

- Ceph picks yet another OSD and keeps copying objects to restore the
  desired number of copies.

- A third OSD, within the same placement group, fails before recovery
  is complete. If this OSD contained the only remaining copy of an
  object, it is permanently lost.

In a cluster containing 10 OSDs with 512 placement groups in a three
replica pool, CRUSH will give each placement groups three OSDs. In the
end, each OSDs will end up hosting (512 * 3) / 10 = ~150 Placement
Groups. When the first OSD fails, the above scenario will therefore
start recovery for all 150 placement groups at the same time.

The 150 placement groups being recovered are likely to be
homogeneously spread over the 9 remaining OSDs. Each remaining OSD is
therefore likely to send copies of objects to all others and also
receive some new objects to be stored because they became part of a
new placement group.

The amount of time it takes for this recovery to complete entirely
depends on the architecture of the Ceph cluster. Let say each OSD is
hosted by a 1TB SSD on a single machine and all of them are connected
to a 10Gb/s switch and the recovery for a single OSD completes within
M minutes. If there are two OSDs per machine using spinners with no
SSD journal and a 1Gb/s switch, it will at least be an order of
magnitude slower.

In a cluster of this size, the number of placement groups has almost
no influence on data durability. It could be 128 or 8192 and the
recovery would not be slower or faster.

However, growing the same Ceph cluster to 20 OSDs instead of 10 OSDs
is likely to speed up recovery and therefore improve data durability
significantly. Each OSD now participates in only ~75 placement groups
instead of ~150 when there were only 10 OSDs and it will still require
all 19 remaining OSDs to perform the same amount of object copies in
order to recover. But where 10 OSDs had to copy approximately 100GB
each, they now have to copy 50GB each instead. If the network was the
bottleneck, recovery will happen twice as fast. In other words,
recovery goes faster when the number of OSDs increases.

If this cluster grows to 40 OSDs, each of them will only host ~35
placement groups. If an OSD dies, recovery will keep going faster
unless it is blocked by another bottleneck. However, if this cluster
grows to 200 OSDs, each of them will only host ~7 placement groups. If
an OSD dies, recovery will happen between at most of ~21 (7 * 3) OSDs
in these placement groups: recovery will take longer than when there
were 40 OSDs, meaning the number of placement groups should be
increased.

No matter how short the recovery time is, there is a chance for a
second OSD to fail while it is in progress. In the 10 OSDs cluster
described above, if any of them fail, then ~8 placement groups
(i.e. ~75 / 9 placement groups being recovered) will only have one
surviving copy. And if any of the 8 remaining OSD fail, the last
objects of one placement group are likely to be lost (i.e. ~8 / 8
placement groups with only one remaining copy being recovered).

When the size of the cluster grows to 20 OSDs, the number of Placement
Groups damaged by the loss of three OSDs drops. The second OSD lost
will degrade ~2 (i.e. ~35 / 19 placement groups being recovered)
instead of ~8 and the third OSD lost will only lose data if it is one
of the two OSDs containing the surviving copy. In other words, if the
probability of losing one OSD is 0.0001% during the recovery time
frame, it goes from 8 * 0.0001% in the cluster with 10 OSDs to 2 *
0.0001% in the cluster with 20 OSDs.

In a nutshell, more OSDs mean faster recovery and a lower risk of
cascading failures leading to the permanent loss of a Placement
Group. Having 512 or 4096 Placement Groups is roughly equivalent in a
cluster with less than 50 OSDs as far as data durability is concerned.

Note: It may take a long time for a new OSD added to the cluster to be
populated with placement groups that were assigned to it. However
there is no degradation of any object and it has no impact on the
durability of the data contained in the Cluster.


.. _object distribution:

Object distribution within a pool
---------------------------------

Ideally objects are evenly distributed in each placement group. Since
CRUSH computes the placement group for each object, but does not
actually know how much data is stored in each OSD within this
placement group, the ratio between the number of placement groups and
the number of OSDs may influence the distribution of the data
significantly.

For instance, if there was single a placement group for ten OSDs in a
three replica pool, only three OSD would be used because CRUSH would
have no other choice. When more placement groups are available,
objects are more likely to be evenly spread among them. CRUSH also
makes every effort to evenly spread OSDs among all existing Placement
Groups.

As long as there are one or two orders of magnitude more Placement
Groups than OSDs, the distribution should be even. For instance, 300
placement groups for 3 OSDs, 1000 placement groups for 10 OSDs etc.

Uneven data distribution can be caused by factors other than the ratio
between OSDs and placement groups. Since CRUSH does not take into
account the size of the objects, a few very large objects may create
an imbalance. Let say one million 4K objects totaling 4GB are evenly
spread among 1000 placement groups on 10 OSDs. They will use 4GB / 10
= 400MB on each OSD. If one 400MB object is added to the pool, the
three OSDs supporting the placement group in which the object has been
placed will be filled with 400MB + 400MB = 800MB while the seven
others will remain occupied with only 400MB.

.. _resource usage:

内存、处理器和网络使用情况
--------------------------

各个归置组、 OSD 和监视器都一直需要内存、网络、处理器，在恢复期间需求更大。为消除\
过载而把对象聚集成簇是归置组存在的主要原因。

最小化归置组数量可节省不少资源。


确定归置组数量
==============

If you have more than 50 OSDs, we recommend approximately 50-100
placement groups per OSD to balance out resource usage, data
durability and distribution. If you have less than 50 OSDs, chosing
among the `preselection`_ above is best. For a single pool of objects,
you can use the following formula to get a baseline::

                (OSDs * 100)
   Total PGs =  ------------
                 pool size

Where **pool size** is either the number of replicas for replicated
pools or the K+M sum for erasure coded pools (as returned by **ceph
osd erasure-code-profile get**).

You should then check if the result makes sense with the way you
designed your Ceph cluster to maximize `data durability`_,
`object distribution`_ and minimize `resource usage`_.

其结果\ **汇总后应该接近 2 的幂**\ 。汇总并非强制的，如果你想确保所有归置组内的对象\
数大致相等，最好检查下。

比如，一个配置了 200 个 OSD 且副本数为 3 的集群，你可以这样估算归置组数量： ::

   (200 * 100)
   ----------- = 6667. Nearest power of 2: 8192
        3

当用了多个数据存储池来存储数据时，你得确保均衡每个存储池的归置组数量、且归置组数量分\
摊到每个 OSD ，这样才能达到较合理的归置组总量，并因此使得每个 OSD 无需耗费过多系统\
资源或拖慢连接进程就能实现较小变迁。

For instance a cluster of 10 pools each with 512 placement groups on
ten OSDs is a total of 5,120 placement groups spread over ten OSDs,
that is 512 placement groups per OSD. That does not use too many
resources. However, if 1,000 pools were created with 512 placement
groups each, the OSDs will handle ~50,000 placement groups each and it
would require significantly more resources and time for peering.


.. _setting the number of placement groups:

设置归置组数量
==============

要设置某存储池的归置组数量，你必须在创建它时就指定好，详情见\ `创建存储池`_\ 。一存\
储池的归置组数量设置好之后，还可以增加（但不可以减少），下列命令可增加归置组数量： ::

	ceph osd pool set {pool-name} pg_num {pg_num}

你增加归置组数量后、还必须增加用于归置的归置组（ ``pgp_num`` ）数量，这样才会开始重\
均衡。 ``pgp_num`` 应等于 ``pg_num`` ，可用下列命令增加用于归置的归置组数量： ::

	ceph osd pool set {pool-name} pgp_num {pgp_num}


获取归置组数量
==============

要获取一个存储池的归置组数量，执行命令： ::

        ceph osd pool get {pool-name} pg_num


获取归置组统计信息
==================

要获取集群里归置组的统计信息，执行命令： ::

        ceph pg dump [--format {format}]

可用格式有纯文本 ``plain`` （默认）和 ``json`` 。


获取卡住的归置组统计信息
========================

要获取所有卡在某状态的归置组统计信息，执行命令： ::

        ceph pg dump_stuck inactive|unclean|stale|undersized|degraded [--format <format>] [-t|--threshold <seconds>]

**Inactive** （不活跃）归置组不能处理读写，因为它们在等待一个有最新数据的 OSD 复活\
且进入集群。

**Unclean** （不干净）归置组含有复制数未达到期望数量的对象，它们应该在恢复中。

**Stale** （不新鲜）归置组处于未知状态：存储它们的 OSD 有段时间没向监视器报告了\
（由  ``mon_osd_report_timeout`` 配置）。

可用格式有 ``plain`` （默认）和 ``json`` 。阀值定义的是，归置组被认为卡住前等待的\
最小时间（默认 300 秒）。


获取一归置组运行图
==================

要获取一个具体归置组的归置组图，执行命令： ::

        ceph pg map {pg-id}

例如： ::

        ceph pg map 1.6c

Ceph 将返回归置组图、归置组、和 OSD 状态： ::

        osdmap e13 pg 1.6c (1.6c) -> up [1,0] acting [1,0]


获取一 PG 的统计信息
====================

要查看一个具体归置组的统计信息，执行命令： ::

        ceph pg {pg-id} query


洗刷归置组
==========

要洗刷一个归置组，执行命令： ::

        ceph pg scrub {pg-id}

Ceph 检查原始的和任何复制节点，生成归置组里所有对象的目录，然后再对比，确保没有对象\
丢失或不匹配，并且它们的内容一致。


恢复丢失的
==========

如果集群丢了一或多个对象，而且必须放弃搜索这些数据，你就要把未找到的对象标记为丢失\
（ ``lost`` ）。

如果所有可能的位置都查询过了，而仍找不到这些对象，你也许得放弃它们了。这可能是罕见的\
失败组合导致的，集群在写入完成前，未能得知写入是否已执行。

当前只支持 revert 选项，它使得回滚到对象的前一个版本（如果它是新对象）或完全忽略\
它。要把 unfound 对象标记为 lost ，执行命令： ::

        ceph pg {pg-id} mark_unfound_lost revert|delete

.. important:: 要谨慎使用，它可能迷惑那些期望对象存在的应用程序。


.. toctree::
        :hidden:

        pg-states
        pg-concepts


.. _创建存储池: ../pools#createpool
