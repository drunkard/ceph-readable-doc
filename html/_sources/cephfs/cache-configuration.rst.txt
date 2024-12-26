==============
 MDS 缓存配置
==============
.. MDS Cache Configuration

元数据服务器协调着所有 MDS 和所有 CephFS 客户端之间的分布式缓存。
缓存的作用是改善元数据访问延迟，并允许客户端\
安全（连贯）地更改元数据状态（如通过 `chmod` ）。
MDS 会发布 **能力**\ 和\ **目录条目租约**\ ，
来指明客户端可以缓存哪些状态、以及客户端可以执行哪些操作（如写入文件）。

MDS 和客户端们都会尝试控制缓存大小。下文介绍了指定 MDS 缓存大小的机制。
注意， MDS 缓存大小不是硬性限制,
MDS 始终允许客户端查找加载到缓存中的新元数据。
这是一项必备策略，因为它能避免客户端请求中出现死锁
（某些请求可能依赖于持有的能力，是释放前持有的能力）。

当 MDS 缓存过大时， MDS 会\ **回收（ recall ）**\ 客户端状态，
这样缓存项就会被解除锁定，可以安全丢弃。
MDS 只能丢弃那些没有客户端引用的元数据状态。
下文还介绍了如何根据工作负载的需要配置 MDS 回收选项。
如果 MDS 回收的内部节流跟不上客户端工作负载的速度，
就有必要这样做。


MDS 缓存尺寸
------------
.. MDS Cache Size

你可以通过一个字节计数来限制元数据服务器（ MDS ）的缓存大小，
通过 `mds_cache_memory_limit` 配置选项：

.. confval:: mds_cache_memory_limit

另外，你可以用 `mds_cache_reservation` 配置一个缓存预留量，
用作 MDS 操作：

.. confval:: mds_cache_reservation

缓存预留量是用一个百分比来限制的，默认为 5% 。
设立此参数的初衷是让 MDS 额外保留一些内存，
以供新的元数据操作使用。因此，通常 MDS 在内存限额以内运行，
因为它会从客户端回收旧条目，
以便丢弃自己缓存里没在用的元数据。

如果 MDS 无法将缓存保持在限额以内， MDS 就会向监视器发送健康警报，
说缓存太大了。这个由 `mds_health_cache_threshold` 配置控制，
默认值是最大缓存尺寸的 150% ：

.. confval:: mds_health_cache_threshold

由于缓存限额不是硬限制， CephFS 客户端、 MDS 中潜在的软件缺陷或\
行为不正常的应用程序可能会导致 MDS 缓存尺寸超额。
健康警告是为了帮助运维人员检测这种情况，并做出必要的调整或调查有问题的客户端。


MDS 缓存修剪
------------
.. MDS Cache Trimming

在 MDS 中，有两个配置选项可以控制缓存修剪的速度：

.. confval:: mds_cache_trim_threshold

.. confval:: mds_cache_trim_decay_rate

节流的目的是防止 MDS 花过多时间修剪缓存。
这可能会影响它处理客户端请求或\
执行其他维护的能力。

修剪配置控制着一个内部\ **衰减计数器**\ 。
每次从缓存中删除元数据时，计数器都会递增。
阈值设定了计数器的最大尺寸，而衰减率则表示计数器的指数半衰期。
如果 MDS 不断从缓存中删除条目，它最终会达到每秒删除
``-ln(0.5)/rate*threshold`` 个条目的稳定状态。

.. note:: 增加配置选项 ``mds_cache_trim_decay_rate`` 的值\
   会让 MDS 花更少的时间修剪缓存。
   要提高缓存修剪速率，
   可以设置一个比较低的值。

默认值比较保守，缓存容量较大的生产集群 MDS
可能需要更改此值。


MDS 回收
--------
.. MDS Recall

MDS 约束着对客户端状态（能力/租约）的回收，
以避免在处理客户端释放消息时产生过多工作。
这个行为通过以下配置进行控制：

在一次给定的回收事件中，
从单个客户端回收能力的最大数量：

.. confval:: mds_recall_max_caps

一个会话中，衰减计数器的阈值和衰减率：

.. confval:: mds_recall_max_decay_threshold

.. confval:: mds_recall_max_decay_rate

会话衰减计数器控制着单个会话的回收率。
计数器的工作原理与上述缓存修剪相同。
每回收一次能力，计数器就会增加一次。

另外还有一个全局衰减计数器，用于节制所有会话的回收：

.. confval:: mds_recall_global_max_decay_threshold

其衰减率与 ``mds_recall_max_decay_rate`` 相同。
所有会话的每一次回收能力都会使这个计数器递增。

如果客户端释放状态的速度较慢，就会出现警告： "failing to respond to cache pressure
（未能及时响应缓存压力）" 或 ``MDS_HEALTH_CLIENT_RECALL`` 。
每个会话的释放速率由另一个衰减计数器监视，用下面的选项配置：

.. confval:: mds_recall_warning_threshold

.. confval:: mds_recall_warning_decay_rate

每次释放能力后，计数器都会递增。
如果客户端释放能力的速度不够快，并且存在缓存压力，
计数器就会显示客户端释放状态的速度是否很慢。

某些载荷和客户端行为可能需要更快地回收客户端状态，
才能跟上获取能力的速度。建议根据需要增加上述计数器，
以消除集群健康状态中、关于回收慢的警告。


MDS 能力获取节制
----------------
.. MDS Cap Acquisition Throttle

在大型目录树上执行一个微不足道的 find 命令，
就会导致客户端接收能力的速度大大超过它释放的速度。
MDS 会尝试让客户端把它的能力降低到 ``mds_max_caps_per_client`` 限额以下，
但回收节流机制会阻止它赶上获取能力的速度。
因此，为了控制能力获取速度， readdir 就被节流了，
配置选项如下：


readdir 能力获取衰减计数器的阈值和衰减率：

.. confval:: mds_session_cap_acquisition_throttle

.. confval:: mds_session_cap_acquisition_decay_rate

能力获取衰减计数器控制着 readdir 获取能力的速率。
衰减计数器的作用与缓存修剪或能力回收相同。
每次调用 readdir 都会增加计数器，
增加的数量就是结果中的文件数。

.. confval:: mds_session_max_caps_throttle_ratio

.. confval:: mds_cap_acquisition_throttle_retry_request_timeout

如果客户端每个会话获取的能力数量大于 ``mds_session_max_caps_throttle_ratio`` ，
且能力获取衰减计数器大于 ``mds_session_cap_acquisition_throttle`` ，
就对 readdir 进行节流。 readdir 请求会在
``mds_cap_acquisition_throttle_retry_request_timeout`` 秒后重试。



会话存活性
----------
.. Session Liveness

MDS 还会持续跟踪会话是否静默。
如果客户端会话没有使用其能力或静默着，
即使缓存没有压力， MDS 也会开始从会话中回收空间。
这有助于 MDS 避免以后在集群载荷较高\
且迫于缓存压力才回收状态。
我们的预期是，未使用其能力的客户端\
在近期内也不太可能使用这些能力。

判断指定的会话是否静默，
由以下配置选项控制：

.. confval:: mds_session_cache_liveness_magnitude

.. confval:: mds_session_cache_liveness_decay_rate

配置选项 ``mds_session_cache_liveness_decay_rate`` 表示\
客户端跟踪能力使用情况的衰减计数器的半衰期。
每次客户端操作或获取能力时， MDS 都会递增计数器。
这是监控客户端缓存使用情况的一种粗略但有效的方法。

``mds_session_cache_liveness_magnitude`` 是一个以 2 为底的\
存活性衰减计数器与会话持有的能力数量之差。
因此，假设客户端有 ``1*2^20`` (1M) 个持有的能力，
但只使用了\ **少于** ``1*2^(20-mds_session_cache_liveness_magnitude)``
（默认时为 1K ）， MDS 就认为客户端处于静默状态\
并开始回收。


能力限额
--------
.. Capability Limit

MDS 还会尽量避免单个客户端获得过多的能力。
这有助于防止某些情况下，恢复进程花费太长时间。
一般来说，客户端没有必要拥有如此大的缓存。
该限额配置选项如下：

.. confval:: mds_max_caps_per_client

不建议把此值设置在 5M 以上，
但也许有的载荷有必要。


处理消息 "clients failing to respond to cache pressure"
-------------------------------------------------------
.. Dealing with "clients failing to respond to cache pressure" messages

每隔一秒钟（或 ``mds_cache_trim_interval`` 配置选项设置的间隔），
MDS 就会运行一次“缓存修剪（ cache trim ）”程序。
其中一个步骤是“回收客户端状态（ recall client state ）”。
在此步骤中， MDS 会检查每个客户端（会话），以确定是否需要回收能力。
如果有以下情况，则 MDS 需要回收能力：

#. 缓存已满（已超过 ``mds_cache_memory_limit`` ），需要释放一些 inode
#. 客户端超过了 ``mds_max_caps_per_client`` （默认为 1M）
#. 客户端处于非活动状态

要确定一个客户端（一个会话）是否处于非活动状态，
需要检查会话的 ``cache_liveness`` 参数，并与下面的值比较： ::

   (num_caps >> mds_session_cache_liveness_magnitude)

其中 ``mds_session_cache_liveness_magnitude`` 是一个配置选项（默认为 ``10`` ）。
如果 ``cache_liveness`` 小于这个计算出的值，
此会话将被视为不活动， MDS 会发送 "recall caps （回收能力）" 请求\
以回收所有缓存的能力（实际的回收值为
``num_caps - mds_min_caps_per_client(100)`` ）。

在某些情况下，许多 "recall caps" 请求发送得太快，
以至于产生了健康警告： "clients failing to respond to cache pressure
（客户端无法响应缓存压力）" 。如果客户端释放能力的速度不够快，
MDS 会在一秒后重新发送 "recall caps" 请求。这意味着
MDS 将会反复发送 "recall caps" 请求。会话中 "recall caps" 的\
“total （总）”计数器会不断增加，最终会超过“监视器警告限额”。

由 ``mds_recall_max_decay_threshold`` 参数（默认为 126K ）控制的\
节流机制可以降低 "recall caps" 计数器的增长速度，
但有时还是不足以减缓 "recall caps" 计数器的增长速度。
如果更改 ``mds_recall_max_decay_threshold`` 值仍不足以降低
"recall caps" 计数器的增长速度，那么可以逐步降低 ``mds_recall_max_caps`` ，
直到日志中不再出现 "clients failing to respond to cache pressure"
（客户端无法响应缓存压力）信息为止。

场景实例
~~~~~~~~
.. Example Scenario

这是一个例子。一个客户端缓存了 20k 个能力。在某个时刻，
服务器认为客户端处于非活动状态（因为会话的 ``cache_liveness`` 值很低）。
服务器开始要求客户端将能力释放到
``mds_min_caps_per_client`` 这么多（默认为 100）。
每秒钟，它都会发送 recall_caps ，要求释放 ``caps_num - mds_min_caps_per_client`` 个能力
（但不能超过 ``mds_recall_max_caps`` ，默认为 30k ）。
客户端开始释放，但释放速度（例如）仅为每秒 100 个能力。

因此，在第一秒内， mds 发送的数量是 recall_caps = 20k - 100 ，
第二秒 recall_caps = (20k - 100) - 100 ，
第三秒 recall_caps = (20k - 200) - 100 ，
以此类推。每次发送 recall_caps 时，都会更新会话的 recall_caps 值，
该值是根据上一分钟发送的 recall_caps 数量计算得出的。也就是说，
计数器会快速增长，最终超过 ``mds_recall_warning_threshold`` （默认值为 128K），
然后 ceph 就会开始在状态中报告 "failing to respond to cache pressure"
（未能响应缓存压力）警告。现在，当我们将 mds_recall_max_caps 设置为 3K 后，
在这种情况下， mds 服务器每秒只发送 3K 个 recall_caps ，
会话的 recall_caps 最大值应该是（如果 mds 每秒发送 3K 个 recall_caps ，
持续至少一分钟） 60 * 3K = 180K 。这意味着仍有可能达到
``mds_recall_warning_threshold`` ，但前提是客户端长时间不“响应”，
而实践证明情况并非如此。
