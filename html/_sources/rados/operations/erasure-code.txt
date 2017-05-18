.. Erasure code

========
 纠删码
========

A Ceph pool is associated to a type to sustain the loss of an OSD
(i.e. a disk since most of the time there is one OSD per disk). The
default choice when `creating a pool <../pools>`_ is *replicated*,
meaning every object is copied on multiple disks. The `Erasure Code
<https://en.wikipedia.org/wiki/Erasure_code>`_ pool type can be used
instead to save space.


创建样板纠删码存储池
--------------------

The simplest erasure coded pool is equivalent to `RAID5
<https://en.wikipedia.org/wiki/Standard_RAID_levels#RAID_5>`_ and
requires at least three hosts::

    $ ceph osd pool create ecpool 12 12 erasure
    pool 'ecpool' created
    $ echo ABCDEFGHI | rados --pool ecpool put NYAN -
    $ rados --pool ecpool get NYAN -
    ABCDEFGHI

.. note:: the 12 in *pool create* stands for 
          `the number of placement groups <../pools>`_.


纠删码配置
----------

默认的纠删码配置可容忍单个 OSD 的损失，相当于副本数为二的副本\
存储池，但只需 1.5TB 的空间即可存储 1TB 数据，而无需 2TB 。默\
认配置可这样查看： ::

    $ ceph osd erasure-code-profile get default
    k=2
    m=1
    plugin=jerasure
    ruleset-failure-domain=host
    technique=reed_sol_van

Choosing the right profile is important because it cannot be modified
after the pool is created: a new pool with a different profile needs
to be created and all objects from the previous pool moved to the new.

The most important parameters of the profile are *K*, *M* and
*ruleset-failure-domain* because they define the storage overhead and
the data durability. For instance, if the desired architecture must
sustain the loss of two racks with a storage overhead of 40% overhead,
the following profile can be defined::

    $ ceph osd erasure-code-profile set myprofile \
       k=3 \
       m=2 \
       ruleset-failure-domain=rack
    $ ceph osd pool create ecpool 12 12 erasure myprofile
    $ echo ABCDEFGHI | rados --pool ecpool put NYAN -
    $ rados --pool ecpool get NYAN -
    ABCDEFGHI

The *NYAN* object will be divided in three (*K=3*) and two additional
*chunks* will be created (*M=2*). The value of *M* defines how many
OSD can be lost simultaneously without losing any data. The
*ruleset-failure-domain=rack* will create a CRUSH ruleset that ensures
no two *chunks* are stored in the same rack.

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


.. _Erasure Coding with Overwrites:

在纠删码存储池上启用重写功能
----------------------------

默认情况下，纠删码存储池只适用于像 RGW 这样进行完整对象写入和\
追加的场景。

从 Luminous 起，在纠删码存储池里进行部分写入的功能可以在存储池\
配置里启用，这样 RBD 和 CephFS 就可以使用纠删码存储池存储数据\
了： ::

    ceph osd pool set ec_pool allow_ec_overwrites true

只有当存储池座落于 bluestore 格式的 OSD 上时才能启用此功能，因\
为 bluestore 的校验和功能在深度洗刷时能探测到位翻转或者其它的\
损坏。除了不安全外，在 filestore 上使用 EC 重写的性能也比
bluestore 上差得多。

纠删码存储池不支持 omap ，所以用于 RBD 和 CephFS 时你必须让它\
们把数据存到 EC 存储池里、而元数据存到副本存储池里。对 RBD 而\
言，这意味着在创建映像时要用纠删码存储池作 ``--data-pool`` （\
数据存储池）的参数： ::

    rbd create --size 1G --data-pool ec_pool replicated_pool/image_name

对 CephFS 而言，要想使用纠删码存储池，可在\
`文件布局 </cephfs/file-layouts>`_\ 里设置成那个存储池。


.. Erasure coded pool and cache tiering

纠删码存储池与缓存分级
----------------------

纠删码存储池与副本存储池相比需要的计算资源更多，而且还缺少一些\
功能，像 omap 。要消除这些局限性，可以在纠删码存储池前加一个\
`缓存层 <../cache-tiering>`_\ 。

例如，假设存储池 *hot-storage* 性能比较好： ::

    $ ceph osd tier add ecpool hot-storage
    $ ceph osd tier cache-mode hot-storage writeback
    $ ceph osd tier set-overlay ecpool hot-storage

这样就把 *hot-storage* 存储池配置成了 *ecpool* 的缓存层，模式为
*writeback* 。如此一来，到 *ecpool* 的每个写和读实际上用的是
*hot-storage* ，而且还受益于其灵活性和速度。

更详细的文档请参阅\ `分级缓存 <../cache-tiering>`_\ 。


.. Glossary

术语
----

*chunk*
   when the encoding function is called, it returns chunks of the same
   size. Data chunks which can be concatenated to reconstruct the original
   object and coding chunks which can be used to rebuild a lost chunk.

*K*
   the number of data *chunks*, i.e. the number of *chunks* in which the
   original object is divided. For instance if *K* = 2 a 10KB object
   will be divided into *K* objects of 5KB each.

*M*
   the number of coding *chunks*, i.e. the number of additional *chunks*
   computed by the encoding functions. If there are 2 coding *chunks*,
   it means 2 OSDs can be out without losing data.


内容列表
--------

.. toctree::
	:maxdepth: 1

	erasure-code-profile
	erasure-code-jerasure
	erasure-code-isa
	erasure-code-lrc
	erasure-code-shec
