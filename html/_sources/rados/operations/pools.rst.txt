.. _rados_pools:

========
 存储池
========

存储池是存储对象时的逻辑分区。

存储池提供的功能：

- **自恢复力（ Resilience ）：** 你可以设置在不丢数据的前提下允许多少 OSD 失效，
  如果你的集群用的是多副本存储池，在不丢失数据的前提下，
  可以失效的 OSD 数量等于副本数。

  例如：典型配置是存储一个对象和两个 RADOS 对象副本
  （即 ``size = 3`` ），但你可以单独配置每个存储池的副本数；
  对\ `纠删码存储池 <../erasure-code>`_\ 来说，
  自恢复力的定义是编码块数
  （例如，默认\ **纠删码配置**\ 里的 ``m=2`` ）。

- **归置组：** :ref:`autoscaler <pg-autoscaler>`
  会设置存储池的归置组（ PG ）数量。
  典型配置中，目标 PG 数是每个 OSD 大约 150 个，
  这样，不用过多计算资源就能得到较优的均衡。
  配置了多个存储池时，要整体考虑到各个存储池和\
  整个集群的归置组数量都要合理。
  每个 PG 都属于一个特定的存储池，所以多个存储池共用一些相同的 OSD 时，
  要确保每个 OSD 上 PG 数的 **总和** 在每 OSD 的 PG 数（ PG-per-OSD ）目标范围内。
  想了解如何手动设置每个存储池的归置组数量，
  看 :ref:`设置归置组数量 <setting the number of placement groups>` ，
  （没有采用 autoscaler 时才需要这个步骤）。

- **CRUSH 规则：** 当你往存储池里存数据的时候，
  此集群内对象及其副本（或纠删码存储池里的数据块）的归置\
  由 CRUSH 规则控制。如果默认规则不符合你的使用情形，
  你可以给自己的存储池定制 CRUSH 规则。

- **快照：** 用 ``ceph osd pool mksnap`` 创建快照的时候，
  实际上创建了某一特定存储池的快照。

存储池命名
==========
.. Pool Names

以 ``.`` 打头的存储池名字是为 Ceph 内部操作保留的，
请不要创建或修改这样的存储池。


列出存储池
==========
.. List Pools

有多种方法获取集群存储池列表。

只罗列集群的存储池名字（写脚本方便），执行：

.. prompt:: bash $

   ceph osd pool ls

::

   .rgw.root
   default.rgw.log
   default.rgw.control
   default.rgw.meta

要罗列集群存储池时带上它的编号，执行下列命令：

.. prompt:: bash $

   ceph osd lspools

::

   1 .rgw.root
   2 default.rgw.log
   3 default.rgw.control
   4 default.rgw.meta

要罗列集群存储池的额外信息，执行：

.. prompt:: bash $

   ceph osd pool ls detail

::

   pool 1 '.rgw.root' replicated size 3 min_size 1 crush_rule 0 object_hash rjenkins pg_num 1 pgp_num 1 autoscale_mode on last_change 19 flags hashpspool stripe_width 0 application rgw read_balance_score 4.00
   pool 2 'default.rgw.log' replicated size 3 min_size 1 crush_rule 0 object_hash rjenkins pg_num 1 pgp_num 1 autoscale_mode on last_change 21 flags hashpspool stripe_width 0 application rgw read_balance_score 4.00
   pool 3 'default.rgw.control' replicated size 3 min_size 1 crush_rule 0 object_hash rjenkins pg_num 1 pgp_num 1 autoscale_mode on last_change 23 flags hashpspool stripe_width 0 application rgw read_balance_score 4.00
   pool 4 'default.rgw.meta' replicated size 3 min_size 1 crush_rule 0 object_hash rjenkins pg_num 1 pgp_num 1 autoscale_mode on last_change 25 flags hashpspool stripe_width 0 pg_autoscale_bias 4 application rgw read_balance_score 4.00

要查看更多信息，执行此命令时可以加上 ``--format`` （或 ``-f`` ）选项，
还有它的值 ``json`` 、 ``json-pretty`` 、 ``xml`` 或 ``xml-pretty`` 。


.. _createpool:

创建存储池
==========
.. Creating a Pool

创建存储池前先看看\ `存储池、归置组和 CRUSH 配置参考`_\ 。
放在监视器集群里的 Ceph 中央配置数据库，里面有个配置选项（名为 ``pg_num`` ），
用于决定每个存储池的 PG 数量，在创建存储池、
却没给它单独指定 PG 数时就用这个值。这个值的默认数值可以更改。
关于每个存储池的归置组数量，见\ `设置归置组数量`_\ 。

.. note:: 从 Luminous 版起，所有存储池都需要与使用它们的应用程序关联。
   详情见\ `关联存储池与应用程序`_\ 。

要创建一个存储池，执行下列命令：

.. prompt:: bash $

    ceph osd pool create {pool-name} [{pg-num} [{pgp-num}]] [replicated] \
             [crush-rule-name] [expected-num-objects]

或者：

.. prompt:: bash $

    ceph osd pool create {pool-name} [{pg-num} [{pgp-num}]]   erasure \
             [erasure-code-profile] [crush-rule-name] [expected_num_objects] [--autoscale-mode=<on,off,warn>]

上述命令里各元素的简要描述，见下文：

.. describe:: {pool-name}

   存储池名称，必须唯一。

   :类型: String
   :是否必需: 必需。

.. describe:: {pg-num}

   存储池拥有的归置组总数。
   关于如何计算合适的数值，参见\ :ref:`归置组`\ 。
   默认值 ``8`` 对大多数系统都 **不合适** 。

   :类型: 整数
   :是否必需: Yes
   :默认值: 8

.. describe:: {pgp-num}

   用于归置的归置组总数。
   此值\ **应该等于归置组总数**\ ，
   归置组分割的情况下除外。

   :类型: 整数
   :是否必需: 没指定的话读取默认值、或 Ceph 配置文件里的值。
   :默认值: 8

.. describe:: {replicated|erasure}

   存储池类型，可以是\ **replicated （多副本）**
   （保存多份对象副本，以便从丢失的 OSD 恢复）或\ **erasure （纠删）**\
   （获得类似 `通用奇偶校验 RAID <../erasure-code>`_ 能力）。
   **replicated** 存储池需更多原始存储空间，但能实现所有 Ceph 操作；
   **erasure**\ 存储池所需原始存储空间较少，但只能执行部分 Ceph 任务，
   而且可能性能也不尽如人意。

   :类型: String
   :是否必需: No.
   :默认值: replicated

.. describe:: [crush-rule-name]

   此存储池所用的 CRUSH 规则名字。
   指定的规则必须存在。

   :类型: String
   :是否必需: No.
   :默认值: 对于多副本存储池（ **replicated pool** ）来说，其规则由
        :confval:`osd_pool_default_crush_rule` 配置指定，此规则必须存在。
        对于纠删码存储池（ **erasure pool** ）来说，
        如果用的是 ``default`` `纠删码配置`_\ 那就是 ``erasure-code`` ，
        否则就是 ``{pool-name}`` 。如果此规则不存在，会悄悄地创建。

.. describe:: [erasure-code-profile=profile]

   仅适用于\ **纠删**\ 存储池。指定\ `纠删码配置`_\ 框架，\
   此配置必须已经由 **osd erasure-code-profile set** 定义好了。

   :类型: String
   :是否必需: No.

.. _纠删码配置: ../erasure-code-profile

.. describe:: --autoscale-mode=<on,off,warn>

   - ``on``: Ceph 集群会根据存储池里 PG 数的实际使用情况自动调整、或给出更改建议。
   - ``warn``: Ceph 集群会根据存储池里 PG 数的实际使用情况自动调整、或给出更改建议。
   - ``off``: 详情参考 :ref:`归置组`\ 。

   :类型: String
   :是否必需: No.
   :默认值:  默认行为由 :confval:`osd_pool_default_pg_autoscale` 选项控制。

.. describe:: [expected-num-objects]

   为这个存储池预估的对象数。设置此值（要同时把 **filestore merge threshold**
   设置为负数）后，在创建存储池时就会拆分 PG 文件夹，
   以免运行时拆分文件夹导致延时增大。

   :类型: Integer
   :是否必需: No.
   :默认值: 0 ，创建存储池时不拆分目录。


.. _associate-pool-to-application:

关联存储池与应用程序
====================
.. Associating a Pool with an Application

存储池要先与应用程序关联才能使用。要用于 CephFS 的存储池、
或由 RGW 创建的存储池已经自动关联过了；计划用于 RBD 的存储池应该\
用 ``rbd`` 工具初始化（详情见\ `块设备命令`_\ ）。

对于其它案例，你可以手动关联存储池与应用程序名字（格式自由），执行下列命令：

.. prompt:: bash $

   ceph osd pool application enable {pool-name} {application-name}

.. note:: CephFS 的应用程序名字是 ``cephfs`` ； RBD 的应用程序\
   名字是 ``rbd`` ， RGW 的应用程序名字是 ``rgw`` 。


设置存储池配额
==============
.. Setting Pool Quotas

存储池配额可设置最大字节数、和/或每存储池最大对象数。

.. prompt:: bash $

   ceph osd pool set-quota {pool-name} [max_objects {obj-count}] [max_bytes {bytes}]

例如：

.. prompt:: bash $

   ceph osd pool set-quota data max_objects 10000

要取消配额，设置为 ``0`` 。


删除存储池
==========
.. Deleting a Pool

要删除一存储池，执行下列命令：

.. prompt:: bash $

   ceph osd pool delete {pool-name} [{pool-name} --yes-i-really-really-mean-it]

要删除存储池，监视器配置的 ``mon_allow_pool_delete`` 标志必须设置为 ``true`` ，
否则监视器们会拒绝删除存储池。

详情见\ `监视器配置`_\ 。

.. _监视器配置: ../../configuration/mon-config-ref

如果有给存储池创建的自定义规则，应该删掉它们。

.. prompt:: bash $

   ceph osd pool get {pool-name} crush_rule

例如，假设规则 id 为 123 ，检查所有存储池，看它们是否在用此规则，执行下列命令：

.. prompt:: bash $

    ceph osd dump | grep "^pool" | grep "crush_rule 123"

如果没有别的存储池使用这个定制规则，那就可以安全地从集群里删掉它。

同样地，如果有用户及其权限与存储池绑死了，但如今这些存储池已不存在，
最好也删除那些用户，执行下列命令：

.. prompt:: bash $

    ceph auth ls | grep -C 5 {pool-name}
    ceph auth del {user}


重命名存储池
============
.. Renaming a Pool

要重命名一个存储池，执行下列命令：

.. prompt:: bash $

   ceph osd pool rename {current-pool-name} {new-pool-name}

如果重命名了一个存储池，而它之前的认证用户有每存储池能力，
那你必须用新存储池名字更新这个用户的能力（即 caps ）。


查看存储池统计信息
==================
.. Showing Pool Statistics

要查看某存储池的利用率统计信息，执行下列命令:

.. prompt:: bash $

   rados df

要获取某个指定的或所有存储池的 I/O 信息，执行下列命令：

.. prompt:: bash $

   ceph osd pool stats [{pool-name}]


给指定存储池拍快照
==================
.. Making a Snapshot of a Pool

要拍下某存储池的快照，执行下列命令：

.. prompt:: bash $

   ceph osd pool mksnap {pool-name} {snap-name}

删除存储池快照
==============
.. Removing a Snapshot of a Pool

要删除某存储池的一个快照，执行下列命令：

.. prompt:: bash $

   ceph osd pool rmsnap {pool-name} {snap-name}

.. _setpoolvalues:

调整存储池选项值
================
.. Setting Pool Values

要设置一个存储池的配置选项值，执行下列命令：

.. prompt:: bash $

   ceph osd pool set {pool-name} {key} {value}

你可以设置下列键的值：

.. _compression_algorithm:

.. describe:: compression_algorithm

   :描述: 设置向底层 BlueStore 后端存储数据时所用的内联压缩算法。
          此选项会覆盖全局配置 :confval:`bluestore_compression_algorithm` 。
   :类型: String
   :有效选项: ``lz4``, ``snappy``, ``zlib``, ``zstd``

.. describe:: compression_mode

   :描述: 设置向底层 BlueStore 后端存储数据时所用内联压缩算法的策略。
          此选项会覆盖全局配置 :confval:`bluestore_compression_mode` 。
   :类型: String
   :有效选项: ``none``, ``passive``, ``aggressive``, ``force``

.. describe:: compression_min_blob_size

   :描述: 设置可压缩数据块的最小尺寸：小于此值的数据块不会被压缩。
          此设置会覆盖下列全局配置：

   * :confval:`bluestore_compression_min_blob_size`
   * :confval:`bluestore_compression_min_blob_size_hdd`
   * :confval:`bluestore_compression_min_blob_size_ssd`

   :类型: Unsigned Integer

.. describe:: compression_max_blob_size

   :描述: 设置数据块的最大尺寸：大于此数值的数据块在压缩前会破碎成尺寸为
          ``compression_max_blob_size`` 的较小二进制块（ blob ）。
   :类型: Unsigned Integer

.. _size:

.. describe:: size

   :描述: 设置存储池中的对象副本数，详情见\ `设置 RADOS 对象副本数`_\ 。
          仅适用于多副本存储池。
   :类型: 整数

.. _min_size:

.. describe:: min_size

   :描述: 设置 I/O 要求的最小副本数，详情参见\ `设置 RADOS 对象副本数`_ 。对于纠删码存储池，
          这个值应该设置成大于 k 的值，因为，如果我们让这个值等于 k ，那就没有冗余，
          在遇到永久的 OSD 故障这样的事件时，数据会丢失。更多信息见\ `纠删码 <../erasure-code>`_ 。
   :类型: 整数
   :适用版本: ``0.54`` 及以上。

.. _pg_num:

.. describe:: pg_num

   :描述: 计算数据归置时使用的有效归置组数量。
   :类型: 整数
   :有效范围: ``0`` 到 ``mon_max_pool_pg_num`` 。如果设置为 ``0`` ，将采用 ``osd_pool_default_pg_autoscale`` 的值。

.. _pgp_num:

.. describe:: pgp_num

   :描述: 计算数据归置时使用的、用于归置的有效归置组数量。
   :类型: 整数
   :有效范围: 在 ``1`` 和 ``pg_num`` 的当前数值之间。

.. _crush_rule:

.. describe:: crush_rule

   :描述: Ceph 用以在存储池内映射对象归置时使用的 CRUSH 规则。
   :类型: String

.. _allow_ec_overwrites:

.. describe:: allow_ec_overwrites

   :描述: 写入一个纠删码存储池时是否允许只更新 RADOS 对象的部分数据，打开后
          CephFS 和 RBD 才能用 EC （erasure-coded ，纠删码）存储池存储用户数据
          （但不能用于元数据）。详情见\ `带重写功能的纠删码编码`_\ 。
   :类型: Boolean

   .. versionadded:: 12.2.0

.. describe:: hashpspool

   :描述: 给指定存储池设置/取消 HASHPSPOOL 标志。
   :类型: 整数
   :有效范围: 1 开启， 0 取消

.. _nodelete:

.. describe:: nodelete

   :描述: 给指定存储池设置/取消 NODELETE 标志。
   :类型: 整数
   :有效范围: 1 开启， 0 取消
   :适用版本: Version ``FIXME``

.. _nopgchange:

.. describe:: nopgchange

   :描述: 给指定存储池设置/取消 NOPGCHANGE 标志。
   :类型: 整数
   :有效范围: 1 开启， 0 取消
   :适用版本: Version ``FIXME``

.. _nosizechange:

.. describe:: nosizechange

   :描述: 给指定存储池设置/取消 NOSIZECHANGE 标志。
   :类型: 整数
   :有效范围: 1 开启， 0 取消
   :适用版本: Version ``FIXME``

.. _bulk:

.. describe:: bulk

   :描述: 设置、取消指定存储池的 bulk 标志。
   :类型: Boolean
   :有效范围: true/1 设置标志， false/0 取消标志

.. _write_fadvise_dontneed:

.. describe:: write_fadvise_dontneed

   :描述: 设置或取消指定存储池的 WRITE_FADVISE_DONTNEED 标志。
   :类型: Integer
   :有效范围: 1 开启， 0 取消

.. _noscrub:

.. describe:: noscrub

   :描述: 设置或取消指定存储池的 NOSCRUB 标志。
   :类型: Integer
   :有效范围: ``1`` 设置， ``0`` 取消

.. _nodeep-scrub:

.. describe:: nodeep-scrub

   :描述: 设置或取消指定存储池的 NODEEP_SCRUB 标志。
   :类型: Integer
   :有效范围: ``1`` 开启， ``0`` 取消

.. _target_max_bytes:

.. describe:: target_max_bytes

   :描述: 达到 ``max_bytes`` 阀值时 Ceph 就回写或赶出对象。
   :类型: 整数
   :实例: ``1000000000000``  #1-TB

.. _target_max_objects:

.. describe:: target_max_objects

   :描述: 达到 ``max_objects`` 阀值时 Ceph 就回写或赶出对象。
   :类型: 整数
   :实例: ``1000000`` #1M objects

.. _fast_read:

.. describe:: fast_read

   :描述: 在纠删码存储池上，如果打开了这个标志，读请求会向所有分片发送子操作读（ sub reads ），
          然后等着，直到收到的分片足以解码给客户端。对 *jerasure* 或 *isa* 纠删码插件来说，
          只要前 *K* 个请求返回，就能立即解码、并先把这些数据发给客户端。
          这样有助于资源折衷，以提升性能。此标志只支持纠删码存储池。
   :类型: Boolean
   :默认值: ``0``

.. _scrub_min_interval:

.. describe:: scrub_min_interval

   :描述: 在负载低时，洗刷存储池 PG 的最小间隔秒数。如果是 ``0`` ，
          就按照中央配置库里的 ``osd_scrub_min_interval`` 执行。
   :类型: Double
   :默认值: ``0``

.. _scrub_max_interval:

.. describe:: scrub_max_interval

   :描述: 不管集群负载如何，都要洗刷存储池 PG 的最大间隔秒数。如果 ``scrub_max_interval``
          的值是 ``0`` ，就采用中央配置库里的 ``osd_scrub_max_interval`` 数值。
   :类型: Double
   :默认值: ``0``

.. _deep_scrub_interval:

.. describe:: deep_scrub_interval

   :描述: “深度”洗刷存储池 PG 的间隔秒数。如果 ``deep_scrub_interval`` 的数值是 ``0`` ，
          就采用中央配置库里的 ``osd_deep_scrub_interval`` 。
   :类型: Double
   :默认值: ``0``

.. _recovery_priority:

.. describe:: recovery_priority

   :描述: 设置此值可调整存储池计算出的保留优先级。此值必须介于 ``-10`` 到 ``10`` 之间。
          所有分配了负值的存储池，其优先级都会低于所有新存储池，所以用户应该给低优先级存储池分配负值。
   :类型: Integer
   :默认值: ``0``

.. _recovery_op_priority:

.. describe:: recovery_op_priority

   :描述: 设置指定存储池 PG 的恢复操作优先级。本选项会覆盖
          :confval:`osd_recovery_op_priority` 设置的通用优先级。
   :类型: Integer
   :默认值: ``0``


获取存储池选项值
================
.. Getting Pool Values

要获取一个存储池选项（ key ）的值，执行下列命令：

.. prompt:: bash $

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


``target_max_bytes``

:描述: 见 target_max_bytes_

:类型: 整数


``target_max_objects``

:描述: 见 target_max_objects_
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


设置 RADOS 对象副本数
=====================
.. Setting the Number of RADOS Object Replicas

要设置多副本存储池的对象副本数，执行下列命令：

.. prompt:: bash $

   ceph osd pool set {poolname} size {num-replicas}

.. important:: ``{num-replicas}`` 参数包括主对象自身。
   例如，如果你想要对象自身及其两份副本共计三份，执行下列命令指定 ``3`` ：

.. prompt:: bash $

   ceph osd pool set data size 3

你可以在每个存储池上执行这个命令。

.. note:: 一个处于降级模式的对象，其副本数小于规定值 ``pool size`` ，
   但仍可接受 I/O 请求。为保证 I/O 正常，
   可用 ``min_size`` 选项为其设置个最低副本数。例如，执行下列命令：

.. prompt:: bash $

   ceph osd pool set data min_size 2

此命令可确保数据存储池里任何副本数小于 ``min_size`` 的对象都不会收到 I/O 了。


获取对象副本数
==============
.. Getting the Number of Object Replicas

要获取对象副本数，执行下列命令：

.. prompt:: bash $

   ceph osd dump | grep 'replicated size'

Ceph 会列出存储池，且高亮 ``replicated size`` 属性。默认情况下，
Ceph 会创建一对象的两个副本（一共三个副本，或 size 值为 ``3`` ）。


管理加了 ``--bulk`` 标记的存储池
================================
.. Managing pools that are flagged with ``--bulk``

见 :ref:`managing_bulk_flagged_pools` 。


配置 stretch 存储池的值
=======================
.. Setting values for a stretch pool

要配置 stretch 存储池的值，执行下列命令：

.. prompt:: bash $

   ceph osd pool stretch set {pool-name} {peering_crush_bucket_count} {peering_crush_bucket_target} {peering_crush_bucket_barrier} {crush_rule} {size} {min_size} [--yes-i-really-mean-it]

以下是参数解析：

.. describe:: {pool-name}

   此存储池的名字。它必须是存在的存储池，此命令不会创建一个新存储池。

   :类型: String
   :是否必需: Yes.

.. describe:: {peering_crush_bucket_count}

   此值与 peering_crush_bucket_barrier 一起使用，可以决定选定的 acting set 内\
   的一组 OSD 是否能相互建立对等互联，根据的是 actine set 中不同桶的数量。

   :类型: Integer
   :是否必需: Yes.

.. describe:: {peering_crush_bucket_target}

   此值与 peering_crush_bucket_barrier 和 size 一起用于计算 bucket_max 值，
   这个值限制着同一个桶内被选入 PG actine set 的 OSD 数量。

   :类型: Integer
   :是否必需: Yes.

.. describe:: {peering_crush_bucket_barrier}

   一个存储池能跨越的桶类型，例如 rack 、 row 、 datacenter 。

   :类型: String
   :是否必需: Yes.

.. describe:: {crush_rule}

   stretch 存储池所用的 crush 规则。存储池类型和 crush_rule 类型必须相匹配（replicated 或 erasure ）。

   :类型: String
   :是否必需: Yes.

.. describe:: {size}

   stretch 存储池内的对象副本数。
   
   :类型: Integer
   :是否必需: Yes.

.. describe:: {min_size}

   stretch 存储池维持正常 I/O 操作所需的最低副本数。

   :类型: Integer
   :是否必需: Yes.

.. describe:: {--yes-i-really-mean-it}

   需要使用该标记来确认您是否真的想绕过\安全检查并为某个 stretch 存储池设置值，
   例如，当你试图把 ``peering_crush_bucket_count`` 或 ``peering_crush_bucket_target``
   设置为大于 crush 图中的桶数量时。

   :类型: Flag
   :是否必需: No.

.. _setting_values_for_a_stretch_pool:

取消给 stretch 存储池设置的值
=============================
.. Unsetting values for a stretch pool

要把存储池恢复回非 stretch 状态时，执行下列命令：

.. prompt:: bash $

   ceph osd pool stretch unset {pool-name}

以下是参数解析：

.. describe:: {pool-name}

   存储池的名字。它必须是存在的存储池且为 stretch 模式，
   就是说，它必须是已经用 `ceph osd pool stretch set` 命令设置过的。

   :类型: String
   :是否必需: Yes.

显示一个 stretch 存储池设置的值
===============================
.. Showing values of a stretch pool

要显示一个 stretch 存储池的值，执行下列命令：

.. prompt:: bash $

   ceph osd pool stretch show {pool-name}

以下是参数解析：

.. describe:: {pool-name}

   存储池的名字。它必须是存在的存储池且为 stretch 模式，
   就是说，它必须是已经用 `ceph osd pool stretch set` 命令设置过的。

   :类型: String
   :是否必需: Yes.


.. _存储池、归置组和 CRUSH 配置参考: ../../configuration/pool-pg-config-ref
.. _Bloom 过滤器: https://en.wikipedia.org/wiki/Bloom_filter
.. _设置归置组数量: ../placement-groups#set-the-number-of-placement-groups
.. _带重写功能的纠删码编码: ../erasure-code#erasure-coding-with-overwrites
.. _块设备命令: ../../../rbd/rados-rbd-cmds/#create-a-block-device-pool
