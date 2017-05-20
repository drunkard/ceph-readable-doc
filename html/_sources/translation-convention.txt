==============
 词语翻译惯例
==============

标记符号含义：

- ``=>`` 表示正在更换翻译口径，如果您遇到请反馈给我。
- ``?``  表示尚未决定翻译还是不翻译。

.. glossary::
    access key
        访问密钥

    active-active
        (RGW) 多活

    active-passive
        (RGW) 主从

    ancestor type
        父级类型

    array
        JSON 相关的为数组；

    backfill
        回填

    bitrot
        位翻转

    bootstrap
        自举引导

    bucket
    bucket policy
        桶、桶策略

    clock drift
        时钟漂移

    clog
    cluster log
        集群日志（基础设施？）

    cluster map
        集群运行图

        释义：集群处于动态的运行中，配置会变更、 OSD 会 up/down ，所以把它理解\
        为静态的图是不对的；尤其对大型集群来说，当机、硬件故障是常态。但是在理\
        解、分析时，提取的片段都可以当作静态的，就像拍下的照片。

    config-key
        <不翻译>，来自代码的名词

    corruption
        （数据）损坏

    delta
        (pg) 增量

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

        epoch 原意是“新纪元，时代，时期，时间上的一点”，我想作者的意思大概就是\
        每隔一段时间总结一下，汇报下某段时间的事件。大概类似于朝代更迭，只是时\
        间短点而以。

        *last epoch start:*
        the last epoch at which all nodes in the acting set for a particular
        placement group agreed on an authoritative history. At this point,
        peering is deemed to have been successful.

        *last epoch clean:*
        the last epoch at which all nodes in the acting set for a particular
        placement group were completely up to date (both PG logs and object
        contents). At this point, recovery is deemed to have been completed.

    erasure coding
    erasure coded pool
        纠删码存储池

        Erasure coding (EC) is a method of data protection in which data is broken into fragments, expanded and encoded with redundant data pieces and stored across a set of different locations, such as disks, storage nodes or geographic locations.

        The goal of erasure coding is to enable data that becomes corrupted at some point in the disk storage process to be reconstructed by using information about the data that's stored elsewhere in the array.

        Erasure coding creates a mathematical function to describe a set of numbers so they can be checked for accuracy and recovered if one is lost. Referred to as polynomial interpolation or oversampling, this is the key concept behind erasure codes. In mathematical terms, the protection offered by erasure coding can be represented in simple form by the following equation: n = k + m. The variable “k” is the original amount of data or symbols. The variable “m” stands for the extra or redundant symbols that are added to provide protection from failures. The variable “n” is the total number of symbols created after the erasure coding process.

        For instance, in a 10 of 16 configuration, or EC 10/16, six extra symbols (m) would be added to the 10 base symbols (k). The 16 data fragments (n) would be spread across 16 drives, nodes or geographic locations. The original file could be reconstructed from 10 verified fragments.

        Erasure codes, also known as forward error correction (FEC) codes, were developed more than 50 years ago. Different types have emerged since that time. In one of the earliest and most common types, Reed-Solomon, the data can be reconstructed using any combination of “k” symbols, or pieces of data, even if “m” symbols are lost or unavailable. For example, in EC 10/16, six drives, nodes or geographic locations could be lost or unavailable, and the original file would still be recoverable.

        Erasure coding can be useful with large quantities of data and any applications or systems that need to tolerate failures, such as disk array systems, data grids, distributed storage applications, object stores and archival storage. One common current use case for erasure coding is object-based cloud storage

    exclusive lock
        互斥锁

    export pin
        (CephFS) 导出销

        释义：默认情况下， MDS 会动态地做负载均衡；而此功能可让目录绑死到一个
        rank ，就像用“销子”固定住了，不能再随便动。

    fast read
        （EC 存储池的）速读（功能）

    full ratio
        占满率

    get ... (eg: get user quota)
        查看... (如：查看用户配额)

    grace period
    grace time
        宽限期；宽限时间；

    intent log
        意图日志

        *From src/rgw/rgw_rados.h:*
        to notify upper layer that we need to do some operation on an object,
        and it's up to the upper layer to schedule this operation.
        e.g., log intent in intent log

    keystone
        <不翻译>

        Keystone 是 OpenStack 项目的子项目，提供身份识别、令牌、目录和策略服\
        务。实现了 OpenStack 的身份识别 API 。

    laggy (osd)
    laggy estimation
        滞后的；滞后量；

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
        <不翻译>，因为它是专有名词，来自代码、JSON 输出。

    (osd) reporter
        报告者 => 报信的?

    out
        <不翻译> => 出列、出局?

    peer
    peering
        （归置组、 OSD ）互联、互联点、正在互联；

    period
        界期 => <不翻译>

        界期保存着组界当前状态的配置数据结构。每个界期都包含一个唯一标识符和一\
        个时间结（ epoch )，每个提交操作都会使界期的时间结递增。

    pin, pinning
        销子，插入

    placement group
    pg
    PG
        归置组

        placement 意思是放置、配置的意思，是静态的；而归置含有整理、放好的意\
        思，是动态过程。但纵观全文，每次用 CRUSH 算法计算出的结果都是静态的，\
        经常变的只是 CRUSH 计算时的输入，所以从整体来说是“归置”，而从局部来说\
        都是“放置”。

        *pg log:*
        a list of recent updates made to objects in a PG. Note that these logs
        can be truncated after all OSDs in the acting set have acknowledged up
        to a certain point.

        *primary:*
        the (by convention first) member of the acting set, who is responsible
        for coordination peering, and is the only OSD that will accept client
        initiated writes to objects in a placement group.

        *recovery:*
        ensuring that copies of all of the objects in a PG are on all of the
        OSDs in the acting set. Once peering has been performed, the primary
        can start accepting write operations, and recovery can proceed in the
        background.

    placement target
        归置目标 => 归置靶

    pool
        存储池

    promote (an image to primary)
    promote (zone)
        晋级...

    proposal
    proposer
        (PAXOS) 提议、提案

    quorum
        法定人数

    quota scope
        配额作用域

    rank
        (CephFS) <不翻译> => 座席、销槽?

    realm
        组界 => <不翻译>

        组界，是域组的容器，有了它就能跨集群划分域组。系统允许创建多个组界，这\
        样就能轻易地在同一集群内跑多个不同的配置。

    region
        <不翻译> => 辖区?

        **此概念已废弃，取而代之的是 zonegroup 。**

        region 是地理空间的逻辑划分，它包含一个或多个 zone 。一个包含多个
        region 的集群必须指定一个主 region 。

    replica
        副本

        a non-primary OSD in the acting set for a placement group (and who has
        been recognized as such and activated by the primary).

    replicated pool
        副本存储池

    sanity check
        健全性检查

    scrub
        洗刷、洗刷操作

    secondary zone
    secondary zone group
        次域、次域组 => 副域、副域组

    secret key
        私钥

    \* set
        *acting set:*
        一个归置组的数据同时分布于多个 OSD ，也就是说这些 OSD 负责这个归置组，\
        这些 OSD 就称为 acting set 。也是个变化的集合。

        *hit set:*
        在 cache tering 中译为：命中集

        *missing set:*
        Each OSD notes update log entries and if they imply updates to the
        contents of an object, adds that object to a list of needed updates.
        This list is called the missing set for that <OSD,PG>.

        *up set:*
        是 acting set 中处于 up 状态的那部分 OSD 。

    shard
        分片

    snap trim
        快照修剪

    snapset
        *未翻译*

    spread metadata load
        散布元数据负荷

    stale pg
        掉队、落伍的归置组

    standby
        灾备、备用

    standby-replay
    standby-replay daemon
        灾备重放、灾备重放守护进程； => 热备？

    stray
        an OSD who is not a member of the current acting set, but has not yet
        been told that it can delete its copies of a particular placement group.

    string interpolation
        字符串插值， https://en.wikipedia.org/wiki/String_interpolation

        即把字符串替换成同名变量的值。

    subuser
        (Swift API) 子用户

    tenant
        (OpenStack) 租户

    token
        (OpenStack) 令牌

    trim
    trimming
        裁剪、清理；
        裁截 => 清理?

    unlink bucket
        断开、切断桶链接、解绑桶，视具体语境采用。

    zone
        域，是一或多个 Ceph 对象网关例程的逻辑分组。每个域组应该指定一个域为主\
        域，由它负责所有桶和用户的创建。

    zonegroup
    zone group
        域组，由多个域组成，此概念大致相当于Jewel 版以前联盟部署中的辖区（
        region ）。应该有一个主域组，负责处理系统配置变更。

    zonegroup map
    zone group map
        域组映射图，是个配置的数据结构，它保存着整个系统的映射图，也就是哪个域\
        组是主的、各个域组间的关系、以及其它可配置信息，如存储策略。


.. vim: set ts=4 sw=4 expandtab colorcolumn=80:
