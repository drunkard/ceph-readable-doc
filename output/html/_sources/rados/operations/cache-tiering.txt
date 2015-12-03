==========
 分级缓存
==========

分级缓存可提升后端存储内某些（热点）数据的 I/O 性能。分级缓存需创建一个由高速而昂贵\
存储设备（如 SSD ）组成的存储池、作为缓存层，以及一个相对低速/廉价设备组成的后端存\
储池（或纠删码编码的）、作为经济存储层。 Ceph 的对象处理器决定往哪里存储对象，分级\
代理决定何时把缓存内的对象刷回后端存储层；所以缓存层和后端存储层对 Ceph 客户端来说\
是完全透明的。


.. ditaa::
           +-------------+
           | Ceph Client |
           +------+------+
                  ^
     Tiering is   |
    Transparent   |              Faster I/O
        to Ceph   |           +---------------+
     Client Ops   |           |               |
                  |    +----->+   Cache Tier  |
                  |    |      |               |
                  |    |      +-----+---+-----+
                  |    |            |   ^
                  v    v            |   |   Active Data in Cache Tier
           +------+----+--+         |   |
           |   Objecter   |         |   |
           +-----------+--+         |   |
                       ^            |   |   Inactive Data in Storage Tier
                       |            v   |
                       |      +-----+---+-----+
                       |      |               |
                       +----->|  Storage Tier |
                              |               |
                              +---------------+
                                 Slower I/O


缓存层代理自动处理缓存层和后端存储之间的数据迁移。然而，管理员仍可干预此迁移规则，主\
要有两种场景：

- **回写模式：** 管理员把缓存层配置为 ``writeback`` 模式时， Ceph 客户端们会把数据\
  写入缓存层、并收到缓存层发来的 ACK ；写入缓存层的数据会被迁移到存储层、然后从缓存\
  层刷掉。直观地看，缓存层位于后端存储层的“前面”，当 Ceph 客户端要读取的数据位于存\
  储层时，缓存层代理会把这些数据迁移到缓存层，然后再发往 Ceph 客户端。从此， Ceph \
  客户端将与缓存层进行 I/O 操作，直到数据不再被读写。此模式对于易变数据来说较理想\
  （如照片/视频编辑、事务数据等）。

- **只读模式：** 管理员把缓存层配置为 ``readonly`` 模式时， Ceph 直接把数据写入后\
  端。读取时， Ceph 把相应对象从后端复制到缓存层，根据已定义策略、脏对象会被缓存层\
  踢出。此模式适合不变数据（如社交网络上展示的图片/视频、 DNA 数据、 X-Ray 照片\
  等），因为从缓存层读出的数据可能包含过期数据，即一致性较差。对易变数据不要用 \
  ``readonly`` 模式。

正因为所有 Ceph 客户端都能用缓存层，所以才有提升块设备、 Ceph 对象存储、 Ceph 文件\
系统和原生绑定的 I/O 性能的潜力。


配置存储池
==========

要设置缓存层，你必须有两个存储池。一个作为后端存储、另一个作为缓存。


配置后端存储池
--------------

设置后端存储池通常会遇到两种场景：

- **标准存储：** 此时，Ceph存储集群内的存储池保存了一对象的多个副本；

- **纠删存储池：** 此时，存储池用纠删码高效地存储数据，性能稍有损失。

在标准存储场景中，你可以用 CRUSH 规则集来标识失败域（如 osd 、主机、机箱、机架、排\
等）。当规则集所涉及的所有驱动器规格、速度（转速和吞吐量）和类型相同时， OSD 守护进\
程运行得最优。创建规则集的详情见 `CRUSH 图`_\ 。创建好规则集后，再创建后端存储池。

在纠删码编码情景中，创建存储池时指定好参数就会自动生成合适的规则集，详情见\ \
`创建存储池`_\ 。

在后续例子中，我们把 ``cold-storage`` 当作后端存储池。


配置缓存池
----------

缓存存储池的设置步骤大致与标准存储情景相同，但仍有不同：缓存层所用的驱动器通常都是高\
性能的、且安装在专用服务器上、有自己的规则集。制定规则集时，要考虑到装有高性能驱动器\
的主机、并忽略没有的主机。详情见\ `给存储池指定 OSD`_ 。

在后续例子中， ``hot-storage`` 作为缓存存储池、 ``cold-storage`` 作为后端存储池。

关于缓存层的配置及其默认值的详细解释请参考\ `存储池——调整存储池`_\ 。


创建缓存层
==========

设置一缓存层需把缓存存储池挂接到后端存储池上： ::

	ceph osd tier add {storagepool} {cachepool}

例如： ::

	ceph osd tier add cold-storage hot-storage

用下列命令设置缓存模式： ::

	ceph osd tier cache-mode {cachepool} {cache-mode}

例如： ::

	ceph osd tier cache-mode hot-storage writeback

缓存层盖在后端存储层之上，所以要多一步：必须把所有客户端流量从存储\
池迁移到缓存存储池。用此命令把客户端流量指向缓存存储池： ::

	ceph osd tier set-overlay {storagepool} {cachepool}

例如： ::

	ceph osd tier set-overlay cold-storage hot-storage


配置缓存层
==========

缓存层支持几个配置选项，可按下列语法配置： ::

	ceph osd pool set {cachepool} {key} {value}

详情见\ `存储池——调整存储池`_\ 。


目标尺寸和类型
--------------

生产环境下，缓存层的 ``hit_set_type`` 还只能用 Bloom 过滤器： ::

	ceph osd pool set {cachepool} hit_set_type bloom

例如： ::

	ceph osd pool set hot-storage hit_set_type bloom

``hit_set_count`` 和 ``hit_set_period`` 选项分别定义了 HitSet 覆盖\
的时间区间、以及保留多少个这样的 HitSet 。 ::

	ceph osd pool set {cachepool} hit_set_count 1
	ceph osd pool set {cachepool} hit_set_period 3600
	ceph osd pool set {cachepool} target_max_bytes 1000000000000

保留一段时间以来的访问记录，这样 Ceph 就能判断一客户端在一段时间内\
访问了某对象一次、还是多次（存活期与热度）。

``min_read_recency_for_promote`` 定义了在处理一个对象的读操作时检\
查多少个 HitSet ，检查结果将用于决定是否异步地提升对象。它的取值应\
该在 0 和 ``hit_set_count`` 之间，如果设置为 0 ，对象会一直被提\
升；如果设置为 1 ，就只检查当前 HitSet ，如果此对象在当前 HitSet \
里就提升它，否则就不提升；设置为其它值时，就要挨个检查此数量的历\
史 HitSet ，如果此对象出现在 ``min_read_recency_for_promote`` 个 \
HitSet 里的任意一个，那就提升它。

还有一个相似的参数用于配置写操作，它是 \
``min_write_recency_for_promote`` 。 ::

	ceph osd pool set {cachepool} min_read_recency_for_promote 1
	ceph osd pool set {cachepool} min_write_recency_for_promote 1

.. note:: 统计时间越长、 ``min_read_recency_for_promote`` 或 \
   ``min_write_recency_for_promote`` 取值越高， ``ceph-osd`` 进程\
   消耗的内存就越多，特别是代理正忙着刷回或赶出对象时，此时所有 \
   ``hit_set_count`` 个 HitSet 都要载入内存。


缓存空间消长
------------

缓存分层代理有两个主要功能：

- **刷回：** 代理找出修改过（或脏）的对象、并把它们转发给存储池做长期存储。

- **赶出：** 代理找出未修改（或干净）的对象、并把最近未用过的赶出缓存。


相对空间消长
~~~~~~~~~~~~

缓存分层代理可根据缓存存储池相对大小刷回或赶出对象。当缓存池包含的已修\
改（或脏）对象达到一定比例时，缓存分层代理就把它们刷回到存储池。用下列\
命令设置 ``cache_target_dirty_ratio`` ： ::

	ceph osd pool set {cachepool} cache_target_dirty_ratio {0.0..1.0}

例如，设置为 ``0.4`` 时，脏对象达到缓存池容量的 40% 就开始刷回： ::

	ceph osd pool set hot-storage cache_target_dirty_ratio 0.4

当脏对象达到其容量的一定比例时，要更快地刷回脏对象。用下列命令设置 \
``cache_target_dirty_high_ratio``::

	ceph osd pool set {cachepool} cache_target_dirty_high_ratio {0.0..1.0}

例如，设置为 ``0.6`` 表示：脏对象达到缓存存储池容量的 60% 时，将开始更\
激进地刷回脏对象。显然，其值最好在 dirty_ratio 和 full_ratio 之间： ::

	ceph osd pool set hot-storage cache_target_dirty_high_ratio 0.6

当缓存池利用率达到总容量的一定比例时，缓存分层代理会赶出部分对象以维持\
空闲空间。执行此命令设置 ``cache_target_full_ratio`` ： ::

	ceph osd pool set {cachepool} cache_target_full_ratio {0.0..1.0}

例如，设置为 ``0.8`` 时，干净对象占到总容量的 80% 就开始赶出缓存池： ::

	ceph osd pool set hot-storage cache_target_full_ratio 0.8


绝对空间消长
~~~~~~~~~~~~

缓存分层代理可根据总字节数或对象数量来刷回或赶出对象，用下列命令可指定最大字节数： ::

	ceph osd pool set {cachepool} target_max_bytes {#bytes}

例如，用下列命令配置在达到 1TB 时刷回或赶出： ::

	ceph osd pool set hot-storage target_max_bytes 1000000000000


用下列命令指定缓存对象的最大数量： ::

	ceph osd pool set {cachepool} target_max_objects {#objects}

例如，用下列命令配置对象数量达到 1M 时开始刷回或赶出： ::

	ceph osd pool set hot-storage target_max_objects 1000000

.. note:: 如果两个都配置了，缓存分层代理会按先到的阀值执行刷回或赶出。


缓存时长
--------

你可以规定缓存层代理必须延迟多久才能把某个已修改（脏）对象刷回后端存储池： ::

	ceph osd pool set {cachepool} cache_min_flush_age {#seconds}

例如，让已修改（或脏）对象需至少延迟 10 分钟才能刷回，执行此命令： ::

	ceph osd pool set hot-storage cache_min_flush_age 600

你可以指定某对象在缓存层至少放置多长时间才能被赶出： ::

	ceph osd pool {cache-tier} cache_min_evict_age {#seconds}

例如，要规定 30 分钟后才赶出对象，执行此命令： ::

	ceph osd pool set hot-storage cache_min_evict_age 1800


拆除缓存层
==========

回写缓存和只读缓存的去除过程不太一样。


拆除只读缓存
------------

只读缓存不含变更数据，所以禁用它不会导致任何近期更改的数据丢失。

#. 把缓存模式改为 ``none`` 即可禁用。 ::

	ceph osd tier cache-mode {cachepool} none

   例如： ::

	ceph osd tier cache-mode hot-storage none

#. 去除后端存储池的缓存池。 ::

	ceph osd tier remove {storagepool} {cachepool}

   例如： ::

	ceph osd tier remove cold-storage hot-storage



拆除回写缓存
------------

回写缓存可能含有更改过的数据，所以在禁用并去除前，必须采取些手段以免丢失缓存内近期更\
改的对象。


#. 把缓存模式改为 ``forward`` ，这样新的和更改过的对象将直接刷回到后端存储池。 ::

	ceph osd tier cache-mode {cachepool} forward

   例如： ::

	ceph osd tier cache-mode hot-storage forward


#. 确保缓存池已刷回，可能要等数分钟： ::

	rados -p {cachepool} ls

   如果缓存池还有对象，你可以手动刷回，例如： ::

	rados -p {cachepool} cache-flush-evict-all


#. 去除此盖子，这样客户端就不会被指到缓存了。 ::

	ceph osd tier remove-overlay {storagetier}

   例如： ::

	ceph osd tier remove-overlay cold-storage


#. 最后，从后端存储池剥离缓存层存储池。 ::

	ceph osd tier remove {storagepool} {cachepool}

   例如： ::

	ceph osd tier remove cold-storage hot-storage


.. _创建存储池: ../pools#create-a-pool
.. _存储池——调整存储池: ../pools#set-pool-values
.. _给存储池指定 OSD: ../crush-map/#placing-different-pools-on-different-osds
.. _Bloom 过滤器: http://en.wikipedia.org/wiki/Bloom_filter
.. _CRUSH 图: ../crush-map
