==============
 词语翻译惯例
==============

标记符号含义：

- ``=>`` 表示正在更换翻译口径，如果您遇到请反馈给我。
- ``?``  表示尚未决定翻译还是不翻译。

.. glossary::
    active-active
        (RGW) 多活

    active-passive
        (RGW) 主从

    array
        JSON 相关的为数组；

    backfill
        回填

    bucket
        桶

    demote (a image to non-primary)
        降级

    down / up
        倒下、倒下了；起来了，活过来了；

    dump
        转储、倒出

    endpoint
        终结点

    epoch
        时间结 => <不翻译> ?

    layout
        （ CephFS 的）布局

    link (bucket)
        链接（桶到用户）

    manpage
        手册页

    master zone
    master zone group
        主域、主域组

    non-master zone
    non-master zone group
        副域、副域组

    object-info
        *未翻译*\ ，因为它是专有名词，来自代码、JSON 输出。

    peer
    peering
        （归置组）互联

    period
        界期 => <不翻译>

        界期保存着组界当前状态的配置数据结构。每个界期都包含一个唯一标识符和一个时间结（ epoch )，每个提交操作都会使界期的时间结递增。

    placement group
    pg
    PG
        归置组

    placement target
        归置目标 => 归置靶

    pool
        存储池

    promote (an image to primary)
    promote (zone)
        晋级...

    realm
        当前未翻译。

        组界，是域组的容器，有了它就能跨集群划分域组。系统允许创建多个组界，这样就能轻易地在同一集群内跑多个不同的配置。

    region
        未翻译。辖区

    replica
        副本

    scrub
        洗刷、洗刷操作

    secondary zone
    secondary zone group
        次域、次域组 => 副域、副域组

    shard
        分片

    snap trim
        快照修剪

    snapset
        *未翻译*

    standby
        灾备、备用

    standby-replay
    standby-replay daemon
        灾备重放、灾备重放守护进程； => 热备？

    tenant
        (OpenStack) 租户

    token
        (OpenStack) 令牌

    unlink (bucket)
        断开、切断

    zone
        域，是一或多个 Ceph 对象网关例程的逻辑分组。每个域组应该指定一个域为主域，由它负责所有桶和用户的创建。

    zonegroup
    zone group
        域组，由多个域组成，此概念大致相当于Jewel 版以前联盟部署中的辖区（ region ）。应该有一个主域组，负责处理系统配置变更。

    zonegroup map
    zone group map
        域组映射图，是个配置的数据结构，它保存着整个系统的映射图，也就是哪个域组是主的、各个域组间的关系、以及其它可配置信息，如存储策略。


.. vim: set ts=4 sw=4 expandtab:
