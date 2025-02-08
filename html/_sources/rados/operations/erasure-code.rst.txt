.. _ecpool:

========
 纠删码
========
.. Erasure code

By default, Ceph `pools <../pools>`_ are created with the type "replicated". In
replicated-type pools, every object is copied to multiple disks. This
multiple copying is the method of data protection known as "replication".

By contrast, `erasure-coded <https://en.wikipedia.org/wiki/Erasure_code>`_
pools use a method of data protection that is different from replication. In
erasure coding, data is broken into fragments of two kinds: data blocks and
parity blocks. If a drive fails or becomes corrupted, the parity blocks are
used to rebuild the data. At scale, erasure coding saves space relative to
replication.

In this documentation, data blocks are referred to as "data chunks"
and parity blocks are referred to as "coding chunks".

Erasure codes are also called "forward error correction codes". The
first forward error correction code was developed in 1950 by Richard
Hamming at Bell Laboratories.


创建纠删码存储池样板
--------------------
.. Creating a sample erasure coded pool

The simplest erasure coded pool is equivalent to `RAID5
<https://en.wikipedia.org/wiki/Standard_RAID_levels#RAID_5>`_ and
requires at least three hosts:

.. prompt:: bash $

   ceph osd pool create ecpool erasure

::

   pool 'ecpool' created

.. prompt:: bash $

   echo ABCDEFGHI | rados --pool ecpool put NYAN -
   rados --pool ecpool get NYAN -
   
::

   ABCDEFGHI


纠删码配置
----------
.. Erasure-code profiles

默认的纠删码配置可容忍连续两个 OSD 的损失，而不丢失数据。
这个纠删码配置相当于副本数为三的多副本存储池，但存储需求却不同：
它只需 2TB 的空间即可存储 1TB 数据，而不是 3TB 空间存储 1TB 数据。\
默认配置可这样查看：

.. prompt:: bash $

   ceph osd erasure-code-profile get default

::

   k=2
   m=2
   plugin=jerasure
   crush-failure-domain=host
   technique=reed_sol_van

.. note::
  The profile just displayed is for the *default* erasure-coded pool, not the
  *simplest* erasure-coded pool. These two pools are not the same:

   The default erasure-coded pool has two data chunks (K) and two coding chunks
   (M). The profile of the default erasure-coded pool is "k=2 m=2".

   The simplest erasure-coded pool has two data chunks (K) and one coding chunk
   (M). The profile of the simplest erasure-coded pool is "k=2 m=1".

确定合理的配置很重要，因为存储池创建后就不能再修改了。
If you find that you need an erasure-coded pool with
a profile different than the one you have created, you must create a new pool
with a different (and presumably more carefully considered) profile. When the
new pool is created, all objects from the wrongly configured pool must be moved
to the newly created pool. There is no way to alter the profile of a pool after
the pool has been created.

配置文件里最重要的参数是 *K* 、 *M* 和
*crush-failure-domain* ，因为它们决定了存储的开销和数据的持久性。
例如，假设期望的系统架构必须能承受\
两个故障机架和 67% 的开销，
可以用下列配置文件：

.. prompt:: bash $

   ceph osd erasure-code-profile set myprofile \
       k=3 \
       m=2 \
       crush-failure-domain=rack
   ceph osd pool create ecpool erasure myprofile
   echo ABCDEFGHI | rados --pool ecpool put NYAN -
   rados --pool ecpool get NYAN -

::

    ABCDEFGHI

对象 *NYAN* 将被分割成三块（ *K=3* ）、
并额外创建两个 *校验块*\ （ *M=2* ）。
*M* 值决定了在不丢数据的前提下可以同时失去多少 OSD 。
*crush-failure-domain=rack* 能使创建的 CRUSH 规则\
可确保两个\ *校验块*\ 不会存储在同一机架上。

.. ditaa::

                            +-------------------+
                       name |       NYAN        |
                            +-------------------+
                    content |     ABCDEFGHI     |
                            +--------+----------+
                                     |
                                     |
                                     v
                              +------+------+
              +---------------+ encode(3,2) +-----------+
              |               +--+--+---+---+           |
              |                  |  |   |               |
              |          +-------+  |   +-----+         |
              |          |          |         |         |
           +--v---+   +--v---+   +--v---+  +--v---+  +--v---+
     name  | NYAN |   | NYAN |   | NYAN |  | NYAN |  | NYAN |
           +------+   +------+   +------+  +------+  +------+
    shard  |  1   |   |  2   |   |  3   |  |  4   |  |  5   |
           +------+   +------+   +------+  +------+  +------+
  content  | ABC  |   | DEF  |   | GHI  |  | YXY  |  | QGC  |
           +--+---+   +--+---+   +--+---+  +--+---+  +--+---+
              |          |          |         |         |
              |          |          v         |         |
              |          |       +--+---+     |         |
              |          |       | OSD1 |     |         |
              |          |       +------+     |         |
              |          |                    |         |
              |          |       +------+     |         |
              |          +------>| OSD2 |     |         |
              |                  +------+     |         |
              |                               |         |
              |                  +------+     |         |
              |                  | OSD3 |<----+         |
              |                  +------+               |
              |                                         |
              |                  +------+               |
              |                  | OSD4 |<--------------+
              |                  +------+
              |
              |                  +------+
              +----------------->| OSD5 |
                                 +------+

 
More information can be found in the `erasure code profiles
<../erasure-code-profile>`_ documentation.


在纠删码存储池上启用重写功能
----------------------------
.. Erasure Coding with Overwrites

默认情况下，纠删码存储池只适用于像 RGW 这样进行完整对象写入和\
追加的场景。

从 Luminous 起，在纠删码存储池里进行部分写入的功能可以在存储池\
配置里启用，这样 RBD 和 CephFS 就可以使用纠删码存储池存储数据\
了：

.. prompt:: bash $

    ceph osd pool set ec_pool allow_ec_overwrites true

只有当存储池座落于 bluestore 格式的 OSD 上时才能启用此功能，因\
为 bluestore 的校验和功能在深度洗刷时能探测到位翻转或者其它的\
损坏。除了不安全外，在 filestore 上使用 EC 重写的性能也比
bluestore 上差得多。

纠删码存储池不支持 omap ，所以用于 RBD 和 CephFS 时你必须让它\
们把数据存到 EC 存储池里、而元数据存到副本存储池里。对 RBD 而\
言，这意味着在创建映像时要用纠删码存储池作 ``--data-pool`` （\
数据存储池）的参数：

.. prompt:: bash $

    rbd create --size 1G --data-pool ec_pool replicated_pool/image_name

对 CephFS 而言，可在文件系统创建期间、或通过\
`文件布局 <../../../cephfs/file-layouts>`_\ 把一个纠删码存储池\
设置为默认的数据存储池。


Erasure-coded pool overhead
---------------------------

The overhead factor (space amplification) of an erasure-coded pool
is `(k+m) / k`.  For a 4,2 profile, the overhead is
thus 1.5, which means that 1.5 GiB of underlying storage are used to store
1 GiB of user data.  Contrast with default three-way replication, with
which the overhead factor is 3.0.  Do not mistake erasure coding for a free
lunch: there is a significant performance tradeoff, especially when using HDDs
and when performing cluster recovery or backfill.

Below is a table showing the overhead factors for various values of `k` and `m`.
As `m` increases above 2, the incremental capacity overhead gain quickly
experiences diminishing returns but the performance impact grows proportionally.
We recommend that you do not choose a profile with `k` > 4 or `m` > 2 until
and unless you fully understand the ramifications, including the number of
failure domains your cluster topology must contain.  If  you choose `m=1`,
expect data unavailability during maintenance and data loss if component
failures overlap.

.. list-table:: Erasure coding overhead
   :widths: 4 4 4 4 4 4 4 4 4 4 4 4
   :header-rows: 1
   :stub-columns: 1

   * -
     - m=1
     - m=2
     - m=3
     - m=4
     - m=5
     - m=6
     - m=7
     - m=8
     - m=9
     - m=10
     - m=11
   * - k=1
     - 2.00
     - 3.00
     - 4.00
     - 5.00
     - 6.00
     - 7.00
     - 8.00
     - 9.00
     - 10.00
     - 11.00
     - 12.00
   * - k=2
     - 1.50
     - 2.00
     - 2.50
     - 3.00
     - 3.50
     - 4.00
     - 4.50
     - 5.00
     - 5.50
     - 6.00
     - 6.50
   * - k=3
     - 1.33
     - 1.67
     - 2.00
     - 2.33
     - 2.67
     - 3.00
     - 3.33
     - 3.67
     - 4.00
     - 4.33
     - 4.67
   * - k=4
     - 1.25
     - 1.50
     - 1.75
     - 2.00
     - 2.25
     - 2.50
     - 2.75
     - 3.00
     - 3.25
     - 3.50
     - 3.75
   * - k=5
     - 1.20
     - 1.40
     - 1.60
     - 1.80
     - 2.00
     - 2.20
     - 2.40
     - 2.60
     - 2.80
     - 3.00
     - 3.20
   * - k=6
     - 1.16
     - 1.33
     - 1.50
     - 1.66
     - 1.83
     - 2.00
     - 2.17
     - 2.33
     - 2.50
     - 2.66
     - 2.83
   * - k=7
     - 1.14
     - 1.29
     - 1.43
     - 1.58
     - 1.71
     - 1.86
     - 2.00
     - 2.14
     - 2.29
     - 2.43
     - 2.58
   * - k=8
     - 1.13
     - 1.25
     - 1.38
     - 1.50
     - 1.63
     - 1.75
     - 1.88
     - 2.00
     - 2.13
     - 2.25
     - 2.38
   * - k=9
     - 1.11
     - 1.22
     - 1.33
     - 1.44
     - 1.56
     - 1.67
     - 1.78
     - 1.88
     - 2.00
     - 2.11
     - 2.22
   * - k=10
     - 1.10
     - 1.20
     - 1.30
     - 1.40
     - 1.50
     - 1.60
     - 1.70
     - 1.80
     - 1.90
     - 2.00
     - 2.10
   * - k=11
     - 1.09
     - 1.18
     - 1.27
     - 1.36
     - 1.45
     - 1.54
     - 1.63
     - 1.72
     - 1.82
     - 1.91
     - 2.00


纠删码存储池与缓存分级
----------------------
.. Erasure-coded pools and cache tiering

.. note:: Cache tiering is deprecated in Reef.

纠删码存储池与副本存储池相比需要的计算资源更多，而且还缺少一些\
功能，像 omap 。要消除这些局限性，可以在纠删码存储池前加一个\
`缓存层 <../cache-tiering>`_\ 。

For example, if the pool *hot-storage* is made of fast storage, the following commands
will place the *hot-storage* pool as a tier of *ecpool* in *writeback*
mode:

.. prompt:: bash $

   ceph osd tier add ecpool hot-storage
   ceph osd tier cache-mode hot-storage writeback
   ceph osd tier set-overlay ecpool hot-storage

如此一来，到 *ecpool* 的每个写和读实际上用的是
*hot-storage* ，而且还受益于其灵活性和速度。

更详细的文档请参阅\ `分级缓存 <../cache-tiering>`_\ 。
注意：缓存分级功能已经废弃，以后的版本会完全删除。


纠删码存储池的恢复
------------------
.. Erasure-coded pool recovery

If an erasure-coded pool loses any data shards, it must recover them from others.
This recovery involves reading from the remaining shards, reconstructing the data, and
writing new shards.

In Octopus and later releases, erasure-coded pools can recover as long as there are at least *K* shards
available. (With fewer than *K* shards, you have actually lost data!)

Prior to Octopus, erasure-coded pools required that at least ``min_size`` shards be
available, even if ``min_size`` was greater than ``K``. This was a conservative
decision made out of an abundance of caution when designing the new pool
mode. As a result, however, pools with lost OSDs but without complete data loss were
unable to recover and go active without manual intervention to temporarily change
the ``min_size`` setting.

We recommend that ``min_size`` be ``K+1`` or greater to prevent loss of writes and
loss of data.


术语
----
.. Glossary

*chunk*
   When the encoding function is called, it returns chunks of the same size as each other. There are two
   kinds of chunks: (1) *data chunks*, which can be concatenated to reconstruct the original object, and
   (2) *coding chunks*, which can be used to rebuild a lost chunk.

*K*
   The number of data chunks into which an object is divided. For example, if *K* = 2, then a 10KB object
   is divided into two objects of 5KB each.

*M*
   The number of coding chunks computed by the encoding function. *M* is equal to the number of OSDs that can
   be missing from the cluster without the cluster suffering data loss. For example, if there are two coding
   chunks, then two OSDs can be missing without data loss.


内容列表
--------

.. toctree::
	:maxdepth: 1

	erasure-code-profile
	erasure-code-jerasure
	erasure-code-isa
	erasure-code-lrc
	erasure-code-shec
	erasure-code-clay
