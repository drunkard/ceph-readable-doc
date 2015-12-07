========
 存储池
========

如果你开始部署集群时没有创建存储池， Ceph 会用默认存储池存数据。存储池提供的功能：

- **自恢复力：** 你可以设置在不丢数据的前提下允许多少 OSD 失效，对多副本存储\
  池来说，此值是一对象应达到的副本数。典型配置存储一个对象和它的一个副本（即 \
  ``size = 2`` ），但你可以更改副本数；对\ `纠删编码的存储池`_\ 来说，此值是\
  编码块数（即\ **纠删码配置**\ 里的 ``m=2`` ）。

- **归置组：** 你可以设置一个存储池的归置组数量。典型配置给每个 OSD 分配大约 100 \
  个归置组，这样，不用过多计算资源就能得到较优的均衡。配置了多个存储池时，要考虑到\
  这些存储池和整个集群的归置组数量要合理。

- **CRUSH 规则：** 当你在存储池里存数据的时候，与此存储池相关联的 CRUSH 规则集可控\
  制 CRUSH 算法，并以此操纵集群内对象及其副本的复制（或纠删码编码的存储池里的数据\
  块）。你可以自定义存储池的 CRUSH 规则。

- **快照：** 用 ``ceph osd pool mksnap`` 创建快照的时候，实际上创建了某一特定存储\
  池的快照。

- **设置所有者：** 你可以设置一个用户 ID 为一个存储池的所有者。

要把数据组织到存储池里，你可以列出、创建、删除存储池，也可以查看每个存储池的利用率。

.. _纠删编码的存储池: ../erasure-code


列出存储池
==========

要列出集群的存储池，命令如下： ::

	ceph osd lspools

在新安装好的集群上，只有一个 ``rbd`` 存储池。


.. _createpool:

创建存储池
==========

创建存储池前先看看\ `存储池、归置组和 CRUSH 配置参考`_\ 。你最好在配置文件\
里重置默认归置组数量，因为默认值并不理想。关于归置组数量请参考\
`设置归置组数量`_\ 。

例如： ::

	osd pool default pg num = 100
	osd pool default pgp num = 100

要创建一个存储池，执行： ::

	ceph osd pool create {pool-name} {pg-num} [{pgp-num}] [replicated] \
		[crush-ruleset-name] [expected-num-objects]
	ceph osd pool create {pool-name} {pg-num}  {pgp-num}   erasure \
		[erasure-code-profile] [crush-ruleset-name] [expected_num_objects]

参数含义如下：


``{pool-name}``

:描述: 存储池名称，必须唯一。
:类型: String
:是否必需: 必需。


``{pg-num}``

:描述: 存储池拥有的归置组总数。关于如何计算合适的数值，请参见\ `归置组`_\ 。\
       默认值 ``8`` 对大多数系统都不合适。

:类型: 整数
:是否必需: Yes
:默认值: 8


``{pgp-num}``

:描述: 用于归置的归置组总数。此值\ **应该等于归置组总数**\ ，归置组分割的情\
       况下除外。

:类型: 整数
:是否必需: 没指定的话读取默认值、或 Ceph 配置文件里的值。
:默认值: 8


``{replicated|erasure}``

:描述: 存储池类型，可以是\ **副本**\ （保存多份对象副本，以便从丢失的 OSD \
       恢复）或\ **纠删**\ （获得类似 `RAID5`_ 的功能）。多副本存储池需更多\
       原始存储空间，但已实现所有 Ceph 操作；\ **纠删**\ 存储池所需原始存储\
       空间较少，但目前仅实现了部分 Ceph 操作。

:类型: String
:是否必需: No.
:默认值: replicated

.. _RAID5: ../erasure-code


``[crush-ruleset-name]``

:描述: 此存储池所用的 CRUSH 规则集名字。指定的规则集必须存在。
:类型: String
:是否必需: No.
:默认值: 对于多副本（ **replicated** ）存储池来说，其默认规则集由 \
         ``osd pool default crush replicated ruleset`` 配置决定，此规则集\
         必须存在。
         对于用 ``erasure-code`` 编码的纠删码（ **erasure** ）存储池来说，\
         不同的 ``{pool-name}`` 所使用的默认（ ``default`` ）\ \
         `纠删码配置`_\ 是不同的，如果它不存在的话，会显式地创建它。


``[erasure-code-profile=profile]``

:描述: 仅用于\ **纠删**\ 存储池。指定\ `纠删码配置`_\ 框架，此配置必须已由 \
       ``osd erasure-code-profile set`` 定义。

:类型: String
:是否必需: No.

.. _纠删码配置: ../erasure-code-profile

创建存储池时，要设置一个合理的归置组数量（如 ``100`` ）。也要考虑\
到每 OSD 的归置组总数，因为归置组很耗计算资源，所以很多存储池和很\
多归置组（如 50 个存储池，各包含 100 归置组）会导致性能下降。收益\
递减点取决于 OSD 主机的强大。

如何为存储池计算合适的归置组数量请参见\ `归置组`_\ 。

.. _归置组: ../placement-groups


``[expected-num-objects]``

:描述: 为这个存储池预估的对象数。设置此值（要同时把 \
       **filestore merge threshold** 设置为负数）后，在创建存储\
       池时就会拆分 PG 文件夹，以免运行时拆分文件夹导致延时增大。

:类型: Integer
:是否必需: No.
:默认值: 0 ，创建存储池时不拆分目录。


设置存储池配额
==============

存储池配额可设置最大字节数、和/或每存储池最大对象数。 ::

	ceph osd pool set-quota {pool-name} [max_objects {obj-count}] [max_bytes {bytes}]

例如： ::

	ceph osd pool set-quota data max_objects 10000

要取消配额，设置为 ``0`` 。


删除存储池
==========

要删除一存储池，执行： ::

	ceph osd pool delete {pool-name} [{pool-name} --yes-i-really-really-mean-it]

如果你给自建的存储池创建了定制的规则集，你不需要存储池时最好删除它。如果你曾严格地创\
建了用户及其权限给一个存储池，但存储池已不存在，最好也删除那些用户。


重命名存储池
============

要重命名一个存储池，执行： ::

	ceph osd pool rename {current-pool-name} {new-pool-name}

如果重命名了一个存储池，且认证用户有每存储池能力，那你必须用新存储池名字更新用户的能\
力（即 caps ）。

.. note:: 适用 ``0.48 Argonaut`` 及以上。


查看存储池统计信息
==================

要查看某存储池的使用统计信息，执行命令： ::

	rados df


拍下存储池快照
==============

要拍下某存储池的快照，执行命令： ::

	ceph osd pool mksnap {pool-name} {snap-name}

.. note:: 适用 ``0.48 Argonaut`` 及以上。


删除存储池快照
==============

要删除某存储池的一个快照，执行命令： ::

	ceph osd pool rmsnap {pool-name} {snap-name}

.. note:: 适用 ``0.48 Argonaut`` 及以上。


.. _setpoolvalues:

调整存储池选项值
================

要设置一个存储池的选项值，执行命令： ::

	ceph osd pool set {pool-name} {key} {value}

你可以设置下列键的值：


.. _size:

``size``

:描述: 设置存储池中的对象副本数，详情参见\ `设置对象副本数`_\ 。\
       仅适用于副本存储池。

:类型: 整数


.. _min_size:

``min_size``

:描述: 设置 I/O 需要的最小副本数，详情参见\ `设置对象副本数`_\ 。\
       仅适用于副本存储池。

:类型: 整数
:适用版本: ``0.54`` 及以上。


.. _crash_replay_interval:

``crash_replay_interval``

:描述: 允许客户端重放确认而未提交请求的秒数。
:类型: 整数


.. _pgp_num:

``pgp_num``

:描述: 计算数据归置时使用的有效归置组数量。
:类型: 整数
:有效范围: 等于或小于 ``pg_num`` 。


.. _crush_ruleset:

``crush_ruleset``

:描述: 集群内映射对象归置时使用的规则集。
:类型: 整数


.. _hashpspool:

``hashpspool``

:描述: 给指定存储池设置/取消 HASHPSPOOL 标志。
:类型: 整数
:有效范围: 1 开启， 0 取消
:适用版本: ``0.48`` 及以上。


.. _nodelete:

``nodelete``

:描述: 给指定存储池设置/取消 NODELETE 标志。
:类型: 整数
:有效范围: 1 开启， 0 取消
:适用版本: Version ``FIXME``


.. _nopgchange:

``nopgchange``

:描述: 给指定存储池设置/取消 NOPGCHANGE 标志。
:类型: 整数
:有效范围: 1 开启， 0 取消
:适用版本: Version ``FIXME``


.. _nosizechange:

``nosizechange``

:描述: 给指定存储池设置/取消 NOSIZECHANGE 标志。
:类型: 整数
:有效范围: 1 开启， 0 取消
:适用版本: Version ``FIXME``


.. _write_fadvise_dontneed:

``write_fadvise_dontneed``

:描述: 设置或取消指定存储池的 WRITE_FADVISE_DONTNEED 标志。
:类型: Integer
:有效范围: 1 开启， 0 取消


.. _noscrub:

``noscrub``

:描述: 设置或取消指定存储池的 NOSCRUB 标志。
:类型: Integer
:有效范围: 1 设置， 0 取消


.. _nodeep-scrub:

``nodeep-scrub``

:描述: 设置或取消指定存储池的 NODEEP_SCRUB 标志。
:类型: Integer
:有效范围: 1 开启， 0 取消


.. _hit_set_type:

``hit_set_type``

:描述: 启用缓存存储池的命中集跟踪，详情见 `Bloom 过滤器`_\ 。
:类型: String
:有效值: ``bloom``, ``explicit_hash``, ``explicit_object``
:默认值: ``bloom`` ，其它是用于测试的。


.. _hit_set_count:

``hit_set_count``

:描述: 为缓存存储池保留的命中集数量。此值越高， ``ceph-osd`` \
       守护进程消耗的内存越多。
:类型: 整数
:有效范围: ``1``. Agent doesn't handle > 1 yet.


.. _hit_set_period:

``hit_set_period``

:描述: 为缓存存储池保留的命中集有效期。此值越高， ``ceph-osd`` \
       消耗的内存越多。
:类型: 整数
:实例: ``3600`` 1hr


.. _hit_set_fpp:

``hit_set_fpp``

:描述: ``bloom`` 命中集类型的假阳性概率。详情见 `Bloom 过滤器`_\ 。
:类型: Double
:有效范围: 0.0 - 1.0
:默认值: ``0.05``


.. _cache_target_dirty_ratio:

``cache_target_dirty_ratio``

:描述: 缓存存储池包含的脏对象达到多少比例时就把它们回写到后端的\
       存储池。
:类型: Double
:默认值: ``.4``


.. _cache_target_dirty_high_ratio:

``cache_target_dirty_high_ratio``

:描述: 缓存存储池内包含的已修改（脏的）对象达到此比例时，缓存层\
       代理就会更快地把脏对象刷回到后端存储池。

:类型: Double
:默认值: ``.6``


.. _cache_target_full_ratio:

``cache_target_full_ratio``

:描述: 缓存存储池包含的干净对象达到多少比例时，缓存代理就把它们\
       赶出缓存存储池。
:类型: Double
:默认值: ``.8``


.. _target_max_bytes:

``target_max_bytes``

:描述: 达到 ``max_bytes`` 阀值时 Ceph 就回写或赶出对象。
:类型: 整数
:实例: ``1000000000000``  #1-TB


.. _target_max_objects:

``target_max_objects``

:描述: 达到 ``max_objects`` 阀值时 Ceph 就回写或赶出对象。
:类型: 整数
:实例: ``1000000`` #1M objects


``hit_set_grade_decay_rate``

:描述: 在两个连续 hit_sets 间的热度衰退速率。
:类型: Integer
:有效范围: 0 - 100
:默认值: ``20``


``hit_set_grade_search_last_n``

:描述: 计算热度时，在 hit_sets 里最多计数 N 次。
:类型: Integer
:有效范围: 0 - hit_set_count
:默认值: ``1``


.. _cache_min_flush_age:

``cache_min_flush_age``

:描述: 达到此时间（单位为秒）时，缓存代理就把某些对象从缓存存储池\
       刷回到存储池。

:类型: 整数
:实例: ``600`` 10min


.. _cache_min_evict_age:

``cache_min_evict_age``

:描述: 达到此时间（单位为秒）时，缓存代理就把某些对象从缓存存储池\
       赶出。

:类型: 整数
:实例: ``1800`` 30min


.. _fast_read:

``fast_read``

:描述: 在纠删码存储池上，如果打开了这个标志，读请求会向所有分片\
       发送子操作读，然后等着，直到收到的分片足以解码给客户端。\
       对 jerasure 和 isa 纠删码插件来说，只要前 K 个请求返回，\
       就能立即解码、并先把这些数据发给客户端。这样有助于资源折\
       衷，以提升性能。当前，这些标志还只能用于纠删码存储池。

:类型: Boolean
:默认值: ``0``


.. _scrub_min_interval:

``scrub_min_interval``

:描述: 在负载低时，洗刷存储池的最大间隔秒数。如果是 0 ，就按照\
       配置文件里的 osd_scrub_min_interval 。

:类型: Double
:默认值: ``0``


.. _scrub_max_interval:

``scrub_max_interval``

:描述: 不管集群负载如何，都要洗刷存储池的最大间隔秒数。如果是 \
       0 ，就按照配置文件里的 osd_scrub_max_interval 。

:类型: Double
:默认值: ``0``


.. _deep_scrub_interval:

``deep_scrub_interval``

:描述: “深度”洗刷存储池的间隔秒数。如果是 0 ，就按照配置文件里\
       的 osd_deep_scrub_interval 。

:类型: Double
:默认值: ``0``


获取存储池选项值
================

要获取一个存储池的选项值，执行命令： ::

	ceph osd pool get {pool-name} {key}

你可以获取到下列选项的值：


``size``

:描述: 见 size_

:类型: 整数


``min_size``

:描述: 见 min_size_

:类型: 整数
:适用版本: ``0.54`` 及以上


``crash_replay_interval``

:描述: 见 crash_replay_interval_

:类型: 整数


``pgp_num``

:描述: 见 pgp_num_

:类型: 整数
:有效范围: 小于等于 ``pg_num`` 。


``crush_ruleset``

:描述: 见 crush_ruleset_
:类型: 整数


``hit_set_type``

:描述: 见 hit_set_type_

:类型: String
:有效选项: ``bloom`` 、 ``explicit_hash`` 、 ``explicit_object``


``hit_set_count``

:描述: 见 hit_set_count_

:类型: 整数


``hit_set_period``

:描述: 见 hit_set_period_

:类型: 整数


``hit_set_fpp``

:描述: 见 hit_set_fpp_

:类型: Double


``cache_target_dirty_ratio``

:描述: 见 cache_target_dirty_ratio_

:类型: Double


``cache_target_dirty_high_ratio``

:描述: 见 cache_target_dirty_high_ratio_

:类型: Double


``cache_target_full_ratio``

:描述: 见 cache_target_full_ratio_

:类型: Double


``target_max_bytes``

:描述: 见 target_max_bytes_

:类型: 整数


``target_max_objects``

:描述: 见 target_max_objects_

:类型: 整数


``cache_min_flush_age``

:描述: 见 cache_min_flush_age_

:类型: 整数


``cache_min_evict_age``

:描述: 见 cache_min_evict_age_

:类型: 整数


``fast_read``

:描述: 见 fast_read_

:类型: Boolean


``scrub_min_interval``

:描述: 见 scrub_min_interval_

:类型: Double


``scrub_max_interval``

:描述: 见 scrub_max_interval_

:类型: Double


``deep_scrub_interval``

:描述: 见 deep_scrub_interval_

:类型: Double


设置对象副本数
==============

要设置多副本存储池的对象副本数，执行命令： ::

	ceph osd pool set {poolname} size {num-replicas}

.. important:: ``{num-replicas}`` 包括对象自身，如果你想要对象\
   自身及其两份拷贝共计三份，指定 3 。

例如： ::

	ceph osd pool set data size 3

你可以在每个存储池上执行这个命令。\ **注意**\ ，一个处于降级模式的对象其副本数小于\
规定值 ``pool size`` ，但仍可接受 I/O 请求。为保证 I/O 正常，可用 ``min_size`` 选\
项为其设置个最低副本数。例如： ::

	ceph osd pool set data min_size 2

这确保数据存储池里任何副本数小于 ``min_size`` 的对象都不会收到 I/O 了。


获取对象副本数
==============

要获取对象副本数，执行命令： ::

	ceph osd dump | grep 'replicated size'

Ceph 会列出存储池，且高亮 ``replicated size`` 属性。默认情况下， Ceph 会创建一对象\
的两个副本（一共三个副本，或 size 值为 3 ）。


.. _存储池、归置组和 CRUSH 配置参考: ../../configuration/pool-pg-config-ref
.. _Bloom 过滤器: http://en.wikipedia.org/wiki/Bloom_filter
.. _设置归置组数量: ../placement-groups#set-the-number-of-placement-groups
