=================================
 存储池、归置组和 CRUSH 配置参考
=================================

.. index:: pools; configuration

当你创建存储池并给它设置归置组数量时，如果你没指定 Ceph 就用默认值。\ \
**我们建议**\ 更改某些默认值，特别是存储池的副本数和默认归置组数量，可以在运\
行 `pool`_ 命令的时候设置这些值。你也可以把配置写入 Ceph 配置文件的 \
``[global]`` 段来覆盖默认值。


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


``osd pg bits``

:描述: 每个 OSD 的归置组位数。
:类型: 32-bit Integer
:默认值: ``6``


``osd pgp bits``

:描述: 每个 OSD 为 PGP 留的位数。
:类型: 32-bit Integer
:默认值: ``6``


``osd crush chooseleaf type``

:描述: 在一个 CRUSH 规则内用于 ``chooseleaf`` 的桶类型。用序列号而不是名字。
:类型: 32-bit Integer
:默认值: ``1`` ，通常一台主机包含一或多个 OSD 。


``osd pool default crush replicated ruleset``

:描述: 创建多副本存储池时用哪个默认 CRUSH 规则集。
:类型: 8-bit Integer
:默认值: ``CEPH_DEFAULT_CRUSH_REPLICATED_RULESET`` ，也就是说，“挑选数\
	 字 ID 最小的规则集”。这样，没有规则集 0 时也能成功创建存储池。


``osd pool erasure code stripe width``

:描述: 设置每个已编码池内的对象条带尺寸（单位为字节）。尺寸为 S 的各对象将存\
       储为 N 个条带，且各条带将分别编码/解码。

:类型: Unsigned 32-bit Integer
:默认值: ``4096``


``osd pool default size``

:描述: 设置一存储池的对象副本数，默认值等同于 \
       ``ceph osd pool set {pool-name} size {size}`` 。

:类型: 32-bit Integer
:默认值: ``3``


``osd pool default min size``

:描述: 设置存储池中已写副本的最小数量，以向客户端确认写操作。如果未达到最小\
       值， Ceph 就不会向客户端回复已写确认。此选项可确保降级（ \
       ``degraded`` ）模式下的最小副本数。

:类型: 32-bit Integer
:默认值: ``0`` ，意思是没有最小值。如果为 ``0`` ，最小值是 ``size - (size / 2)`` 。


``osd pool default pg num``

:描述: 一个存储池的默认归置组数量，默认值即是 ``mkpool`` 的 ``pg_num`` 参数。
:类型: 32-bit Integer
:默认值: ``8``


``osd pool default pgp num``

:描述: 一个存储池里，为归置使用的归置组数量，默认值等同于 ``mkpool`` 的 \
       ``pgp_num`` 参数。当前 PG 和 PGP 应该相同。

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
