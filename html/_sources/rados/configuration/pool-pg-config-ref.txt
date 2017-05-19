.. _Pool, PG and CRUSH Config Reference:

=================================
 存储池、归置组和 CRUSH 配置参考
=================================

.. index:: pools; configuration

当你创建存储池并给它设置归置组数量时，如果你没指定 Ceph 就\
用默认值。\ **我们建议**\ 更改某些默认值，特别是存储池的副\
本数和默认归置组数量，可以在运行 `pool`_ 命令的时候设置这些\
值。你也可以把配置写入 Ceph 配置文件的 ``[global]`` 段来覆\
盖默认值。


.. literalinclude:: pool-pg.conf
   :language: ini


``mon max pool pg num``

:描述: 每个存储的最大归置组数量。
:类型: Integer
:默认值: ``65536``


``mon pg create interval``

:描述: 在同一个 OSD 里创建 PG 的间隔秒数。
:类型: Float
:默认值: ``30.0``


``mon pg stuck threshold``

:描述: 多长时间无响应的 PG 才认为它卡住了。
:类型: 32-bit Integer
:默认值: ``300``


``mon pg min inactive``

:描述: 如果处于不活跃状态的 PG 数量超过 ``mon_pg_stuck_threshold``
       选项配置的值，就向集群日志发出一个 ``HEALTH_ERR`` 。负\
       数表示禁用，永远别进入 ERR 状态。
:类型: Integer
:默认值: ``1``


``mon pg warn min per osd``

:描述: 如果每个 OSD 上的 PG 数量平均值低于此数值，就向集群日志\
       发出一个 ``HEALTH_WARN`` 。负数禁用此功能。
:类型: Integer
:默认值: ``30``


``mon pg warn max per osd``

:描述: 如果每个 OSD 上的 PG 数量平均值超过此数值，就向集群日志\
       发出一个 ``HEALTH_WARN`` 。负数禁用此功能。
:类型: Integer
:默认值: ``300``


``mon pg warn min objects``

:描述: 集群内的对象总数小于此数值时不发出警告。
:类型: Integer
:默认值: ``1000``


``mon pg warn min pool objects``

:描述: 存储池内对象数小于此数值时，不发出有关此存储池的警告。
:类型: Integer
:默认值: ``1000``


``mon pg check down all threshold``

:描述: 倒下的 OSD 百分比阈值，超过此值我们会检查所有 PG ，看有\
       没有掉队的。
:类型: Float
:默认值: ``0.5``


``mon pg warn max object skew``

:描述: 如果某一个存储池的平均对象数大于全部存储池的 \
       ``mon pg warn max object skew`` 倍，就向集群日志发出一个
       ``HEALTH_WARN`` 。负数禁用此功能。
:类型: Float
:默认值: ``10``


``mon delta reset interval``

:描述: 多少秒没活动我们就把 pg 增量重置为 0 。我们会跟踪各存储\
       池已用空间的增量，借此，我们可以更容易地理解恢复进度或\
       者缓存层的性能；但是，如果没收到某个存储池的活动情况报\
       告，我们会简单粗暴地重置与它相关的增量历史。
:类型: Integer
:默认值: ``10``


``mon osd max op age``

:描述: 最大操作时长（最好设置为 2 的幂），一个请求被阻塞的时间\
       超过此值就会发出 ``HEALTH_WARN`` 告警。
:类型: Float
:默认值: ``32.0``


``osd pg bits``

:描述: 每个 OSD 的归置组位数。
:类型: 32-bit Integer
:默认值: ``6``


``osd pgp bits``

:描述: 每个 OSD 为 PGP 留的位数。
:类型: 32-bit Integer
:默认值: ``6``


``osd crush chooseleaf type``

:描述: 在一个 CRUSH 规则内用于 ``chooseleaf`` 的桶类型。用\
       序列号而不是名字。
:类型: 32-bit Integer
:默认值: ``1`` ，通常一台主机包含一或多个 OSD 。


``osd crush initial weight``

:描述: 新加入 crushmap 的 OSD 应该分配的初始 CRUSH 权重。

:类型: Double
:默认值: ``按 TB 计算的新增 OSD 的容量``\ 。默认情况下，给\
         新增 OSD 分配的 CRUSH 权重是其以 TB 为单位的容量\
         数值，详情见\ `调整桶条目的权重`_\ 。


``osd pool default crush replicated ruleset``

:描述: 创建多副本存储池时用哪个默认 CRUSH 规则集。
:类型: 8-bit Integer
:默认值: ``CEPH_DEFAULT_CRUSH_REPLICATED_RULESET`` ，也就是\
         说，“挑选数字 ID 最小的规则集”。这样，没有规则集 0
         时也能成功创建存储池。


``osd pool erasure code stripe unit``

:描述: 设置纠删码存储池内对象条带的默认尺寸，单位为字节。每个\
       尺寸为 S 的对象都将存储为单个数据块为 ``stripe unit`` \
       字节的 N 个条带，每个尺寸为 ``N * stripe unit`` 字节的\
       条带将分别独立地编码、解码。此选项可被纠删码配置中的
       ``stripe_unit`` 选项覆盖。

:类型: Unsigned 32-bit Integer
:默认值: ``4096``


``osd pool default size``

:描述: 设置一存储池的对象副本数，默认值等同于 \
       ``ceph osd pool set {pool-name} size {size}`` 。

:类型: 32-bit Integer
:默认值: ``3``


``osd pool default min size``

:描述: 设置存储池中已写副本的最小数量，以向客户端确认写操作。\
       如果未达到配置的最小值， Ceph 就不会向客户端反馈已写\
       确认。此选项可确保降级（ ``degraded`` ）模式下的最小\
       副本数。

:类型: 32-bit Integer
:默认值: ``0`` ，意思是没有最小值。如果为 ``0`` ，最小值是
         ``size - (size / 2)`` 。


``osd pool default pg num``

:描述: 一个存储池的默认归置组数量，默认值即是 ``mkpool`` 的
       ``pg_num`` 参数。
:类型: 32-bit Integer
:默认值: ``8``


``osd pool default pgp num``

:描述: 一个存储池里，为归置使用的归置组数量，默认值等同于
       ``mkpool`` 的 ``pgp_num`` 参数。当前 PG 和 PGP 应\
       该相同。

:类型: 32-bit Integer
:默认值: ``8``


``osd pool default flags``

:描述: 新存储池的默认标志。
:类型: 32-bit Integer
:默认值: ``0``


``osd max pgls``

:描述: 将列出的最大归置组数量，一客户端请求量大时会影响 OSD 。
:类型: Unsigned 64-bit Integer
:默认值: ``1024``
:Note: 默认值应该没问题。


``osd min pg log entries``

:描述: 清理日志文件的时候保留的归置组日志量。
:类型: 32-bit Int Unsigned
:默认值: ``1000``


``osd default data pool replay window``

:描述: 一 OSD 等待客户端重播请求的时间，秒。
:类型: 32-bit Integer
:默认值: ``45``


.. _pool: ../../operations/pools
.. _监控 OSD 和归置组: ../../operations/monitoring-osd-pg#peering
.. _调整桶条目的权重: ../../operations/crush-map#weightingbucketitems
