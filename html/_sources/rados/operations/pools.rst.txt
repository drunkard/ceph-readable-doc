========
 存储池
========

如果你开始部署集群时没有创建存储池， Ceph 会用默认存储池存数\
据。存储池提供的功能：

- **自恢复力：** 你可以设置在不丢数据的前提下允许多少 OSD 失\
  效，对多副本存储池来说，此值是一对象应达到的副本数。典型配置\
  存储一个对象和它的一个副本（即 ``size = 2`` ），但你可以更改\
  副本数；对\ `纠删编码的存储池`_\ 来说，此值是编码块数（即\
  **纠删码配置**\ 里的 ``m=2`` ）。

- **归置组：** 你可以设置一个存储池的归置组数量。典型配置给每\
  个 OSD 分配大约 100 个归置组，这样，不用过多计算资源就能得到\
  较优的均衡。配置了多个存储池时，要考虑到这些存储池和整个集群\
  的归置组数量要合理。

- **CRUSH 规则：** 当你往存储池里存数据的时候，此集群内对象\
  及其副本（或纠删码存储池里的数据块）的归置由 CRUSH 规则控制。\
  如果默认规则不符合你的使用情形，你可以给自己的存储池定制
  CRUSH 规则。

- **快照：** 用 ``ceph osd pool mksnap`` 创建快照的时候，实际\
  上创建了某一特定存储池的快照。

要把数据组织到存储池里，你可以列出、创建、删除存储池，也可以查\
看每个存储池的利用率。

.. _纠删编码的存储池: ../erasure-code


.. List Pools

列出存储池
==========

要列出集群的存储池，命令如下： ::

	ceph osd lspools


.. Create a Pool
.. _createpool:

创建存储池
==========

创建存储池前先看看\ `存储池、归置组和 CRUSH 配置参考`_\ 。你\
最好在配置文件里重置默认归置组数量，因为默认值并不理想。关于\
归置组数量请参考\ `设置归置组数量`_\ 。

.. note:: 从 Luminous 版起，所有存储池都需要与使用它们的\
   应用程序关联。详情见\ `关联存储池与应用程序`_\ 。

例如： ::

	osd pool default pg num = 100
	osd pool default pgp num = 100

要创建一个存储池，执行： ::

	ceph osd pool create {pool-name} {pg-num} [{pgp-num}] [replicated] \
		[crush-crush-name] [expected-num-objects]
	ceph osd pool create {pool-name} {pg-num}  {pgp-num}   erasure \
		[erasure-code-profile] [crush-rule-name] [expected_num_objects]

参数含义如下：


``{pool-name}``

:描述: 存储池名称，必须唯一。
:类型: String
:是否必需: 必需。


``{pg-num}``

:描述: 存储池拥有的归置组总数。关于如何计算合适的数值，请参见\
       `归置组`_\ 。默认值 ``8`` 对大多数系统都不合适。

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


``[crush-rule-name]``

:描述: 此存储池所用的 CRUSH 规则名字。指定的规则必须存在。
:类型: String
:是否必需: No.
:默认值: 对于多副本存储池（ **replicated pool** ）来说，其规则\
         由 ``osd pool default crush rule`` 配置决定，此规则\
         必须存在。对于纠删码存储池（ **erasure pool** ）来说，
         如果用的是 ``default`` `纠删码配置`_\ 那就是
         ``erasure-code`` ，否则就是 ``{pool-name}`` 。如果此\
         规则不存在，会悄悄地创建。


``[erasure-code-profile=profile]``

:描述: 仅用于\ **纠删**\ 存储池。指定\ `纠删码配置`_\ 框架，\
       此配置必须已由 ``osd erasure-code-profile set`` 定义。

:类型: String
:是否必需: No.

.. _纠删码配置: ../erasure-code-profile

创建存储池时，要设置一个合理的归置组数量（如 ``100`` ）。也要\
考虑到每 OSD 的归置组总数，因为归置组很耗计算资源，所以很多\
存储池和很多归置组（如 50 个存储池，各包含 100 归置组）会导致\
性能下降。收益递减点取决于 OSD 主机的强大。

如何为存储池计算合适的归置组数量请参见\ `归置组`_\ 。

.. _归置组: ../placement-groups


``[expected-num-objects]``

:描述: 为这个存储池预估的对象数。设置此值（要同时把 \
       **filestore merge threshold** 设置为负数）后，在创建\
       存储池时就会拆分 PG 文件夹，以免运行时拆分文件夹导致\
       延时增大。

:类型: Integer
:是否必需: No.
:默认值: 0 ，创建存储池时不拆分目录。


.. Associate Pool to Application
.. _associate-pool-to-application:

关联存储池与应用程序
====================

存储池要先与应用程序关联才能使用。要用于 CephFS 的存储池、或由
RGW 创建的存储池已经自动关联过了；计划用于 RBD 的存储池应该用
``rbd`` 工具初始化（详情见\ `块设备命令`_\ ）。

对于其它案例，你可以手动关联存储池与应用程序名字。 ::

        ceph osd pool application enable {pool-name} {application-name}

.. note:: CephFS 的应用程序名字是 ``cephfs`` ； RBD 的应用程序\
   名字是 ``rbd`` ， RGW 的应用程序名字是 ``rgw`` 。


.. Set Pool Quotas

设置存储池配额
==============

存储池配额可设置最大字节数、和/或每存储池最大对象数。 ::

	ceph osd pool set-quota {pool-name} [max_objects {obj-count}] [max_bytes {bytes}]

例如： ::

	ceph osd pool set-quota data max_objects 10000

要取消配额，设置为 ``0`` 。


.. Delete a Pool

删除存储池
==========

要删除一存储池，执行： ::

	ceph osd pool delete {pool-name} [{pool-name} --yes-i-really-really-mean-it]

要删除存储池，监视器配置的 mon_allow_pool_delete 标志必须设置为
true ，否则它会拒绝删除存储池。

详情见\ `监视器配置`_\ 。

.. _监视器配置: ../../configuration/mon-config-ref

如果你给自建的存储池创建了定制的规则，那么没有存储池在\
用它时你应该删掉它： ::

	ceph osd pool get {pool-name} crush_rule

假设规则 id 为 123 ，你可以这样找出还在用它的其它存储池： ::

	ceph osd dump | grep "^pool" | grep "crush_rule 123"

如果没有别的存储池使用这个定制规则，那就可以安全地从集群里删掉\
它。

如果你曾创建过一些用户及其权限、并与存储池绑死了，但如今这些\
存储池已不存在，最好也删除那些用户： ::

	ceph auth ls | grep -C 5 {pool-name}
	ceph auth del {user}


.. Rename a Pool

重命名存储池
============

要重命名一个存储池，执行： ::

	ceph osd pool rename {current-pool-name} {new-pool-name}

如果重命名了一个存储池，且认证用户有每存储池能力，那你必须用新\
存储池名字更新用户的能力（即 caps ）。


.. Show Pool Statistics

查看存储池统计信息
==================

要查看某存储池的使用统计信息，执行命令： ::

	rados df

另外，要获取某个或所有存储池的 I/O 信息，用命令： ::

        ceph osd pool stats [{pool-name}]


.. Make a Snapshot of a Pool

拍下存储池快照
==============

要拍下某存储池的快照，执行命令： ::

	ceph osd pool mksnap {pool-name} {snap-name}


.. Remove a Snapshot of a Pool

删除存储池快照
==============

要删除某存储池的一个快照，执行命令： ::

	ceph osd pool rmsnap {pool-name} {snap-name}


.. Set Pool Values
.. _setpoolvalues:

调整存储池选项值
================

要设置一个存储池的选项值，执行命令： ::

	ceph osd pool set {pool-name} {key} {value}

你可以设置下列键的值：


.. _compression_algorithm:

``compression_algorithm``

:描述: 设置底层 BlueStore 所用的内联压缩算法。此选项会覆盖
       ``bluestore compression algorithm`` 的\
       `全局配置 <http://docs.ceph.com/docs/master/rados/configuration/bluestore-config-ref/#inline-compression>`_ 。
:类型: String
:有效选项: ``lz4``, ``snappy``, ``zlib``, ``zstd``


``compression_mode``

:描述: 设置底层 BlueStore 所用压缩算法的策略。此选项会覆盖
       ``bluestore compression mode`` 的\
       `全局配置 <http://docs.ceph.com/docs/master/rados/configuration/bluestore-config-ref/#inline-compression>`_ 。
:类型: String
:有效选项: ``none``, ``passive``, ``aggressive``, ``force``


``compression_min_blob_size``

:描述: 小于这个的数据块不会被压缩。此选项会覆盖
       ``bluestore compression min blob *`` 的\
       `全局配置 <http://docs.ceph.com/docs/master/rados/configuration/bluestore-config-ref/#inline-compression>`_ 。
:类型: Unsigned Integer


``compression_max_blob_size``

:描述: 大于此数值的数据块在压缩前会破碎成尺寸为
       ``compression_max_blob_size`` 的较小二进制块。
:类型: Unsigned Integer


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


.. _pg_num:

``pg_num``

:描述: 计算数据归置时使用的有效归置组数量。
:类型: 整数
:有效范围: 不高于 ``pg_num`` 的当前值。


.. _pgp_num:

``pgp_num``

:描述: 计算数据归置时使用的、用于归置的有效归置组数量。
:类型: 整数
:有效范围: 等于或小于 ``pg_num`` 。


.. _crush_rule:

``crush_rule``

:描述: 集群内映射对象归置时使用的规则。
:类型: 整数


.. _allow_ec_overwrites:

``allow_ec_overwrites``

:描述: 写入一个纠删码存储池时是否允许更新对象的部分数据，允许后
       CephFS 和 RBD 才能用这个存储池，详情见\
       `在纠删码存储池上启用重写功能`_\ 。
:类型: Boolean
:版本要求: ``12.2.0`` 以及更高版


.. _hashpspool:

``hashpspool``

:描述: 给指定存储池设置/取消 HASHPSPOOL 标志。
:类型: 整数
:有效范围: 1 开启， 0 取消


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


``hit_set_search_last_n``

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

:描述: 在负载低时，洗刷存储池的最小间隔秒数。如果是 0 ，就按照\
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


.. _recovery_priority:

``recovery_priority``

:描述: 设置此值后，它会提高或降低计算出的保留优先级，此值必须\
       介于 -10 到 10 之间。给不太重要的存储池分配负值，其\
       优先级就低于其它新存储池。
:类型: Integer
:默认值: ``0``


.. _recovery_op_priority:

``recovery_op_priority``

:描述: 指定此存储池的恢复操作优先级，而非
       ``osd_recovery_op_priority`` 。
:类型: Integer
:默认值: ``0``


.. Get Pool Values

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


``pg_num``

:描述: 见 pg_num_
:类型: 整数


``pgp_num``

:描述: 见 pgp_num_
:类型: 整数
:有效范围: 小于等于 ``pg_num`` 。


``crush_rule``

:描述: 见 crush_rule_
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


``allow_ec_overwrites``

:描述: 见 allow_ec_overwrites_

:类型: Boolean


``recovery_priority``

:描述: 见 recovery_priority_

:类型: Integer


``recovery_op_priority``

:描述: 见 recovery_op_priority_

:类型: Integer


.. Set the Number of Object Replicas

设置对象副本数
==============

要设置多副本存储池的对象副本数，执行命令： ::

	ceph osd pool set {poolname} size {num-replicas}

.. important:: ``{num-replicas}`` 包括对象自身，如果你想要对象\
   自身及其两份拷贝共计三份，指定 3 。

例如： ::

	ceph osd pool set data size 3

你可以在每个存储池上执行这个命令。\ **注意**\ ，一个处于降级\
模式的对象其副本数小于规定值 ``pool size`` ，但仍可接受 I/O
请求。为保证 I/O 正常，可用 ``min_size`` 选项为其设置个最低\
副本数。例如： ::

	ceph osd pool set data min_size 2

这确保数据存储池里任何副本数小于 ``min_size`` 的对象都不会收\
到 I/O 了。


获取对象副本数
==============

要获取对象副本数，执行命令： ::

	ceph osd dump | grep 'replicated size'

Ceph 会列出存储池，且高亮 ``replicated size`` 属性。默认情况\
下， Ceph 会创建一对象的两个副本（一共三个副本，或 size 值为
3 ）。


.. _存储池、归置组和 CRUSH 配置参考: ../../configuration/pool-pg-config-ref
.. _Bloom 过滤器: https://en.wikipedia.org/wiki/Bloom_filter
.. _设置归置组数量: ../placement-groups#set-the-number-of-placement-groups
.. _在纠删码存储池上启用重写功能: ../erasure-code#erasure-coding-with-overwrites
.. _块设备命令: ../../../rbd/rados-rbd-cmds/#create-a-block-device-pool
