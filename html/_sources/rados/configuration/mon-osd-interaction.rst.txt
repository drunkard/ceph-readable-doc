=========================
 监视器与 OSD 交互的配置
=========================
.. Configuring Monitor/OSD Interaction

.. index:: heartbeat

完成基本配置后就可以部署、运行 Ceph 了。\
执行 ``ceph health`` 或 ``ceph -s`` 命令时，\
:term:`监视器`\ 会报告 :term:`Ceph 存储集群`\ 的当前状态。\
监视器通过让各 :term:`OSD` 自己报告、\
并接收 OSD 关于邻居状态的报告来掌握集群动态。\
如果监视器没收到报告，或者它只收到集群的变更报告，\
那它就要更新\ :term:`集群运行图`\ 。

对于监视器与 OSD 的交互 Ceph 提供了合理的默认值，\
然而你可以覆盖默认值。\
下面几段从集群监控角度描述了 Ceph 监视器与 OSD 如何交互。


.. index:: heartbeat interval

OSD 验证心跳
============
.. OSDs Check Heartbeats

各 OSD 每间隔 6 秒内的随机时间段会与其他 OSD 守护进程进行心跳检查，\
如果一个邻近的 OSD 在 20 秒的宽限期内都没有心跳，\
就把这个邻近 OSD 的状态标记为 ``down`` 、\
并上报给监视器，它会更新 Ceph 集群运行图。\
这个宽限期可以用 Ceph 配置文件的 ``[mon]`` 和 ``[osd]`` 段（同时配置）、\
或 ``[global]`` 段下的 ``osd heartbeat grace`` 选项更改、\
或者在运行时更改。


.. ditaa::

           +---------+          +---------+
           |  OSD 1  |          |  OSD 2  |
           +---------+          +---------+
                |                    |
                |----+ Heartbeat     |
                |    | Interval      |
                |<---+ Exceeded      |
                |                    |
                |       Check        |
                |     Heartbeat      |
                |------------------->|
                |                    |
                |<-------------------|
                |   Heart Beating    |
                |                    |
                |----+ Heartbeat     |
                |    | Interval      |
                |<---+ Exceeded      |
                |                    |
                |       Check        |
                |     Heartbeat      |
                |------------------->|
                |                    |
                |----+ Grace         |
                |    | Period        |
                |<---+ Exceeded      |
                |                    |
                |----+ Mark          |
                |    | OSD 2         |
                |<---+ Down          |


.. index:: OSD down report

OSD 报告死亡 OSD
================
.. OSDs Report Down OSDs

在默认配置下，必须有两个来自不同主机的 Ceph OSD 守护进程向\
监视器报告了另一个 OSD 守护进程倒下（ ``down`` ）的消息，此时\
监视器才会确认那个报告所指的 OSD 倒下了。但是有可能报告这个\
错误的所有 OSD 都位于同一机架上、连着一个有问题的交换机，\
导致它们与另一个 OSD 的连接有问题；为避免此类误报，我们把报告\
这个错误的互联点们当作一个代理点，代理这部分滞后情况差不多的\
嫌疑“子集群”（相对于整个集群）。很明显，它不可能百发百中，\
但是遇到了就能帮我们把只需轻微修正的控制在遇挫系统的一个子集内。
``mon osd reporter subtree level`` 选项可用于分组互联点，\
也就是按照它们在 CRUSH 图里的共同父级把这些节点分组为\
“子集群”；按默认配置，只需要有两个来自不同子树的报告就可以证明\
另一个 OSD 守护进程倒下了。你可以更改来自独立子树的报告者数量、\
以及要求的共同父级类型（向 Ceph 监视器报告某个 OSD 倒下时\
会被采纳），在 Ceph 配置文件的 ``[mon]`` 段下增加
``mon osd min down reporters`` 和
``mon osd reporter subtree level`` 即可，或者更改运行时配置。


.. ditaa::

           +---------+     +---------+      +---------+
           |  OSD 1  |     |  OSD 2  |      | Monitor |
           +---------+     +---------+      +---------+
                |               |                |
                | OSD 3 Is Down |                |
                |---------------+--------------->|
                |               |                |
                |               |                |
                |               | OSD 3 Is Down  |
                |               |--------------->|
                |               |                |
                |               |                |
                |               |                |---------+ Mark
                |               |                |         | OSD 3
                |               |                |<--------+ Down


.. index:: peering failure

OSD 报告互联失败
================
.. OSDs Report Peering Failure

如果一 OSD 守护进程不能和配置文件中定义的任何 OSD 建立连接，\
它会每 30 秒向监视器索要一次最新集群运行图，你可以在 ``[osd]``
下设置 ``osd mon heartbeat interval`` 来更改这个心跳间隔，或\
者运行时更改。

.. ditaa::

           +---------+     +---------+     +-------+     +---------+
           |  OSD 1  |     |  OSD 2  |     | OSD 3 |     | Monitor |
           +---------+     +---------+     +-------+     +---------+
                |               |              |              |
                |  Request To   |              |              |
                |     Peer      |              |              |
                |-------------->|              |              |
                |<--------------|              |              |
                |    Peering                   |              |
                |                              |              |
                |  Request To                  |              |
                |     Peer                     |              |
                |----------------------------->|              |
                |                                             |
                |----+ OSD Monitor                            |
                |    | Heartbeat                              |
                |<---+ Interval Exceeded                      |
                |                                             |
                |         Failed to Peer with OSD 3           |
                |-------------------------------------------->|
                |<--------------------------------------------|
                |          Receive New Cluster Map            |


.. index:: OSD status

OSD 报告自己的状态
==================
.. OSDs Report Their Status

如果一 OSD 在 ``mon osd report timeout`` 时间内没向监视器报告\
过，监视器就认为它 ``down`` 了。在 OSD 守护进程会向监视器报告\
某些事件，如某次操作失败、归置组状态变更、 ``up_thru`` 变更、\
或它将在 5 秒内启动。你可以设置 ``[osd]`` 下的 \
``osd mon report interval`` 来更改最小报告间隔，或在运行时\
更改。 OSD 守护进程每 120 秒会向监视器报告其状态，不论是否有值\
得报告的事件。在 ``[osd]`` 段下设置 ``osd mon report interval max``
可更改 OSD 报告间隔，或运行时更改。


.. ditaa::

           +---------+          +---------+
           |  OSD 1  |          | Monitor |
           +---------+          +---------+
                |                    |
                |----+ Report Min    |
                |    | Interval      |
                |<---+ Exceeded      |
                |                    |
                |----+ Reportable    |
                |    | Event         |
                |<---+ Occurs        |
                |                    |
                |     Report To      |
                |      Monitor       |
                |------------------->|
                |                    |
                |----+ Report Max    |
                |    | Interval      |
                |<---+ Exceeded      |
                |                    |
                |     Report To      |
                |      Monitor       |
                |------------------->|
                |                    |
                |----+ Monitor       |
                |    | Fails         |
                |<---+               |
                                     +----+ Monitor OSD
                                     |    | Report Timeout
                                     |<---+ Exceeded
                                     |
                                     +----+ Mark
                                     |    | OSD 1
                                     |<---+ Down




配置选项
========
.. Configuration Settings

心跳选项应该置于配置文件的 ``[global]`` 段下。


.. index:: monitor heartbeat

监视器选项
----------
.. Monitor Settings

.. confval:: mon_osd_min_up_ratio
.. confval:: mon_osd_min_in_ratio
.. confval:: mon_osd_laggy_halflife
.. confval:: mon_osd_laggy_weight
.. confval:: mon_osd_laggy_max_interval
.. confval:: mon_osd_adjust_heartbeat_grace
.. confval:: mon_osd_adjust_down_out_interval
.. confval:: mon_osd_auto_mark_in
.. confval:: mon_osd_auto_mark_auto_out_in
.. confval:: mon_osd_auto_mark_new_in
.. confval:: mon_osd_down_out_interval
.. confval:: mon_osd_down_out_subtree_limit
.. confval:: mon_osd_report_timeout
.. confval:: mon_osd_min_down_reporters
.. confval:: mon_osd_reporter_subtree_level


.. index:: OSD hearbeat

OSD 选项
--------
.. OSD Settings

.. confval:: osd_heartbeat_interval
.. confval:: osd_heartbeat_grace
.. confval:: osd_mon_heartbeat_interval
.. confval:: osd_mon_heartbeat_stat_stale
.. confval:: osd_mon_report_interval
