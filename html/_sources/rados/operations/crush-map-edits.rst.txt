手动编辑一个 CRUSH 图
=====================

.. note:: 手动编辑 CRUSH 图是一项高级管理任务。
   绝大多数集群所需的 CRUSH 变更基本都能通过\
   标准的 ceph CLI 实现，不需要手动编辑 CRUSH 图。
   如果你发现了一种使用案例，用最近的
   Ceph 版本还 *不得不* 手动编辑，
   试着联系一下 Ceph 开发者们，
   这样未来的 Ceph 版本可以排除你的边角案例。

要编辑现有的 CRUSH 图：

#. `获取 CRUSH 图`_\ ；
#. `反编译`_ CRUSH 图；
#. 至少编辑一个\ `设备`_\ 、\ `桶`_\ 、\ `规则`_\ ；
#. `重编译`_ CRUSH 图；
#. `注入 CRUSH 图`_\ 。

关于为指定存储池设置 CRUSH 图规则的细节，见 `调整存储池`_\ 。

.. _获取 CRUSH 图: #getcrushmap
.. _反编译: #decompilecrushmap
.. _设备: #crushmapdevices
.. _桶: #crushmapbuckets
.. _规则: #crushmaprules
.. _重编译: #compilecrushmap
.. _注入 CRUSH 图: #setcrushmap
.. _调整存储池: ../pools#setpoolvalues


.. _getcrushmap:

获取 CRUSH 图
-------------

要获取集群的 CRUSH 图，执行命令： ::

    ceph osd getcrushmap -o {compiled-crushmap-filename}

Ceph 将把 CRUSH 输出（ -o ）到你指定的文件，
由于 CRUSH 图是已编译的，所以编辑前必须先反编译。

.. _decompilecrushmap:

反编译 CRUSH 图
---------------
.. Decompile a CRUSH Map

要反编译 CRUSH 图，执行命令： ::

    crushtool -d {compiled-crushmap-filename} -o {decompiled-crushmap-filename}

.. _compilecrushmap:

重编译 CRUSH 图
---------------
.. Recompile a CRUSH Map

编译 CRUSH 图用下列命令： ::

    crushtool -c {decompiled-crushmap-filename} -o {compiled-crushmap-filename}

.. _setcrushmap:

设置 CRUSH 图
-------------
.. Set the CRUSH Map

用下列命令为你的集群注入 CRUSH 图： ::

    ceph osd setcrushmap -i {compiled-crushmap-filename}

Ceph 将从你指定的文件加载（ -i ）一份编译好的 CRUSH 图。

分段
----
.. Sections

CRUSH 图有 6 个主要段落。

#. **tunables:** 图顶的序言描述了所有 CRUSH 行为变化
   （与历史的、遗留的 CRUSH 行为相比）的 *tunables （可调值）* 。
   它们之中有对以前缺陷的纠正、优化、或者其他行为上的变化，
   都是多年来对 CRUSH 行为的持续优化。

#. **devices:** 设备是存储数据的单个 OSD 。

#. **types**: ``types`` 桶定义了 CRUSH 分级结构里要用的桶类型，
   桶由逐级汇聚的存储位置（如行、机柜、机箱、主机等等）
   及其权重组成。

#. **buckets:** 定义了桶类型之后，
   还必须定义分级结构里的各个节点、
   它们的类型、以及它包含的设备或其它节点。

#. **rules:** 规则定义的是数据在分级结构里的设备上\
   如何分配的策略。

#. **choose_args:** 它是与已经调整完毕的分级结构关联的另一种权重，
   用于优化数据归置。单个 choose_args 图可以用于整个集群、
   也可以为每个存储池单独创建一个。


.. _crushmapdevices:

CRUSH 图之设备
--------------
.. CRUSH Map Devices

这里的设备是存储数据的单个 OSD 。
通常，集群里的每个 OSD 守护进程都要在这里定义一条。
设备用一个 ``id`` （一个非负整数）和一个 ``name`` 标识，
一般 ``osd.N`` 里的 ``N`` 就是设备 id 。

.. _crush-map-device-class:

设备也可以与一个 *device class* （如 ``hdd`` 或者 ``ssd`` ）\
关联，这样就可以让 crush 规则方便地引用。

::

    # devices
    device {num} {osd.name} [class {class}]

例如： ::

    # devices
    device 0 osd.0 class ssd
    device 1 osd.1 class hdd
    device 2 osd.2
    device 3 osd.3

绝大多数情况下，每个设备都映射到单个 ``ceph-osd`` 守护进程。
它通常是单个存储设备、一对设备（例如，
一个用于数据、一个用于日志或元数据）、
或者某些情况下一个小的 RAID 设备。



CRUSH 图之桶类型
----------------
.. CRUSH Map Bucket Types

CRUSH 图里的第二个列表定义了 bucket （桶）类型，
桶简化了节点和叶子层次。
节点（或非叶子）桶在分级结构里一般表示物理位置，
节点汇聚了其它节点或叶子，
叶桶表示 ``ceph-osd`` 守护进程及其对应的存储媒体。

.. tip:: CRUSH 中用到的术语 bucket 表示分级结构中的一个节点，\
   也就是一个位置或一部分硬件。
   但是在 RADOS 网关接口的术语中，\
   它又是不同的概念。

要往 CRUSH 图中增加一种 bucket 类型，在现有桶类型列表下方新增\
一行，输入 ``type`` 、之后是惟一数字 ID 和一个桶名。按惯例，\
会有一个叶子桶为 ``type 0`` ，然而你可以指定任何名字（如 osd 、
disk 、 drive 、 storage 等等）： ::

    #types
    type {num} {bucket-name}

例如： ::

    # types
    type 0 osd
    type 1 host
    type 2 chassis
    type 3 rack
    type 4 row
    type 5 pdu
    type 6 pod
    type 7 room
    type 8 datacenter
    type 9 zone
    type 10 region
    type 11 root



.. _crushmapbuckets:

CRUSH 图之桶层次
----------------
.. CRUSH Map Bucket Hierarchy

CRUSH 算法根据各设备的权重、
大致统一的概率把数据对象分布到存储设备中。
CRUSH 根据你定义的集群运行图分布对象及其副本，
CRUSH 图表达了可用存储设备以及\
包含它们的逻辑单元。

要把归置组映射到跨故障域的 OSD ，
一个 CRUSH 图需定义一系列分级桶类型
（即现有 CRUSH 图的 ``#type`` 下）。
创建桶分级结构的目的是按故障域隔离叶节点，
像主机、机箱、机柜、电力分配单元、机群、行、房间、和数据中心。
除了表示叶节点的 OSD ，其它分级结构都是任意的，
你可以按需定义。

我们建议 CRUSH 图内的命名符合贵公司的硬件命名规则，
并且采用反映物理硬件的例程名。
良好的命名可简化集群管理和故障排除，
当 OSD 和/或其它硬件出问题时，
管理员可轻易找到对应物理硬件。

在下例中，桶分级结构有一个名为 ``osd`` 的分支、和两个节点分别\
名为 ``host`` 和 ``rack`` 。

.. ditaa::

                           +-----------+
                           | {o}rack   |
                           |   Bucket  |
                           +-----+-----+
                                 |
                 +---------------+---------------+
                 |                               |
           +-----+-----+                   +-----+-----+
           | {o}host   |                   | {o}host   |
           |   Bucket  |                   |   Bucket  |
           +-----+-----+                   +-----+-----+
                 |                               |
         +-------+-------+               +-------+-------+
         |               |               |               |
   +-----+-----+   +-----+-----+   +-----+-----+   +-----+-----+
   |    osd    |   |    osd    |   |    osd    |   |    osd    |
   |   Bucket  |   |   Bucket  |   |   Bucket  |   |   Bucket  |
   +-----------+   +-----------+   +-----------+   +-----------+

.. note:: 编号较高的 ``rack`` 桶类型汇聚编号较低的 ``host`` 桶类型。

位于 CRUSH 图起始部分、 ``#devices`` 列表内是表示叶节点的存储设备，
没必要声明为桶例程。
位于分级结构第二低层的桶一般用于汇聚设备
（即它通常是包含存储媒体的计算机，
你可以用自己喜欢的名字描述，如节点、计算机、服务器、主机、机器等等）。
在高密度环境下，经常出现一机框内安装多个主机/节点的情况，
因此还要考虑机框故障——比如，
某一节点故障后需要拉出机框维修，
这会影响多个主机/节点和其内的 OSD 。

声明一个桶例程时，你必须指定其类型、
惟一名称（字符串）、惟一负整数 ID （可选）、
指定和各条目总容量/能力相关的权重、\
指定桶算法（通常是 ``straw2`` ）、和哈希（通常为 ``0`` ，
表示散列算法 ``rjenkins1`` ）。一个桶可以包含一到多条，
这些条目可以由节点桶或叶子组成，
它们可以有个权重用来反映条目的相对权重。

你可以按下列语法声明一个节点桶： ::

    [bucket-type] [bucket-name] {
        id [a unique negative numeric ID]
        weight [the relative capacity/capability of the item(s)]
        alg [the bucket type: uniform | list | tree | straw ]
        hash [the hash type: 0 by default]
        item [item-name] weight [weight]
    }

例如，用上面的图表，我们可以定义两个主机桶和一个机柜桶，
OSD 被声明为主机桶内的条目： ::

    host node1 {
        id -1
        alg straw2
        hash 0
        item osd.0 weight 1.00
        item osd.1 weight 1.00
    }

    host node2 {
        id -2
        alg straw2
        hash 0
        item osd.2 weight 1.00
        item osd.3 weight 1.00
    }

    rack rack1 {
        id -3
        alg straw2
        hash 0
        item node1 weight 2.00
        item node2 weight 2.00
    }

.. note:: 在前述示例中，机柜桶不包含任何 OSD ，
   它只包含低一级的主机桶、
   以及其内条目的权重之和。

.. topic:: 桶类型

   Ceph 支持五种桶类型，每种都是性能和组织简易间的折衷。\
   如果你不确定用哪种桶，我们建议 ``straw2`` ，\
   关于桶类型的详细讨论见
   `CRUSH - 可控、可伸缩、分布式地归置多副本数据`_\ ，\
   特别是 **Section 3.4** 。桶的类型有：

    #. **uniform**: 这种桶用\ **完全**\ 相同的权重汇聚设备。\
        例如，公司采购或淘汰硬件时，\
        一般都有相同的物理配置（如批量采购）。\
        当存储设备权重都相同时，\
        你就可以用 ``uniform`` 桶类型，\
        这样 CRUSH 就始终把副本映射到 uniform 桶。\
        权重不统一时，你应该采用其它桶算法。

    #. **list**: 这种桶把它们的内容汇聚为链表。\
        对于一个\ **持续扩展的集群** 来说，基于
        :abbr:`RUSH (基于可伸缩哈希算法的复制， Replication Under Scalable Hashing)` :sub:`P`
        算法的列表是一个自然、直观的选择，\
        不管是对象按一定概率被重定位到最新的设备、\
        或者像从前一样仍保留在较老的设备上，其结果都是，\
        有新条目加入桶时，产生的数据迁移都是最优的。\
        然而，如果从链表的中间或末尾删除掉一些条目，\
        将会导致大量没必要的挪动，因此，\
        这种桶适合\ **永不或极少缩减**\ 的场景。

    #. **tree**: 它用一种二进制搜索树，\
       在桶包含大量条目时比 list 桶更高效。它基于 \
       :abbr:`RUSH (Replication Under Scalable Hashing)` :sub:`R` 算法，
       tree 桶把归置时间减少到了 O(log :sub:`n`) ，\
       这使得它们更适合管理更大规模的设备或嵌套桶。

    #. **straw**: list 和 tree 桶用分而治之策略，\
       或者给特定条目一定优先级（如位于链表开头的条目）、\
       或者根本无需考虑整个子树上所有的条目。\
       这样提升了副本归置进程的性能，但是也导致了重新组织时的次优结果，\
       如增加、拆除、或调整某条目的权重。\
       straw 桶类型允许所有条目像抽签一样相互公平“竞争”\
       副本归置。

        #. **straw2**: straw2 桶是对 straw 的改进，\
           在邻居权重改变时可正确地避免条目间的数据迁移。

           例如，条目 A 的权重再次增大或完全删除，都仅会有数据\
           迁移进或移出条目 A 。

.. topic:: Hash

   各个桶都用了一种散列算法，当前 Ceph 仅支持 ``rjenkins1`` ，\
   输入 ``0`` 表示散列算法设置为 ``rjenkins1`` 。


.. _weightingbucketitems:

.. topic:: 调整桶的权重

   Ceph 用双整形表示桶权重。
   权重和设备容量不同，
   我们建议用 ``1.00`` 作为 1TB 存储设备的相对权重，
   这样 ``0.5`` 的权重大概代表 500GB 、
   ``3.00`` 大概代表 3TB 。
   较高级桶的权重是所有枝叶桶的\
   权重之和。

   一个桶的权重是一维的，
   你也可以计算条目权重来反映存储设备性能。
   例如，如果你有很多 1TB 的硬盘，
   其中一些数据传输速率相对低、其他的数据传输率相对高，
   即使它们容量相同，也应该设置不同的权重
   （如给吞吐量较低的硬盘设置权重 0.8 ，\
   较高的设置 1.20 ）。



.. _crushmaprules:

CRUSH 图之规则
--------------
.. CRUSH Map Rules

CRUSH 图支持“ CRUSH 规则”概念，用以决定一个存储池里的数据如何\
归置。默认的 CRUSH 图有一条规则适用于各存储池。对大型集群\
来说，你可能创建很多存储池，且每个存储池都有它自己的、非默认的
CRUSH 规则。

.. note:: 大多数情况下，你都不需要修改默认规则。默认情况下，\
   新创建的存储池其规则会设置为 ``0`` 。

CRUSH 规则定义了归置和复制策略、或分布策略，
用它可以规定 CRUSH 如何放置对象副本。
例如，你也许想创建一条规则用以选择一对目的地做双路镜像；
另一条规则用以选择位于两个数据中心的三个目的地做三路镜像；
又一条规则用 6 个设备做纠删编码。
关于 CRUSH 规则的详细研究见
`CRUSH - 可控、可伸缩、分布式地归置多副本数据`_\ ，
主要是 **Section 3.2** 。

规则格式如下： ::

    rule <rulename> {

        id [a unique whole numeric ID]
        type [ replicated | erasure ]
        step take <bucket-name> [class <device-class>]
        step [choose|chooseleaf] [firstn|indep] <N> type <bucket-type>
        step emit
    }


``id``

:描述: 全局唯一的数字，用于标识此规则。
:目的: 规则掩码的一个组件。
:类型: Integer
:是否必需: Yes
:默认值: 0


``type``

:描述: 为一个驱动器（多副本的）或 RAID 确定\
       一条规则。

:目的: 规则掩码的一个组件。
:类型: String
:是否必需: Yes
:默认值: ``replicated``
:有效值: 当前仅支持 ``replicated`` 和 ``erasure``


``step take <bucket-name> [class <device-class>]``

:描述: 选取一个桶名，并沿树往下迭代。如果指定了
       ``device-class`` ，它必须与前面定义设备时的分类名一致，\
       不属于此类的设备都会被排除在外。

:目的: 规则的一个组件。
:是否必需: Yes
:实例: ``step take data``


``step choose firstn {num} type {bucket-type}``

:描述: 在当前可用桶中选取指定类型桶的数量，这个数字通常是所在\
       存储池的副本数（即 pool size ）。

       - 如果 ``{num} == 0`` 选择 ``pool-num-replicas`` 个桶\
         （所有可用的）；
       - 如果 ``{num} > 0 && < pool-num-replicas`` 就选择那么\
         多的桶；
       - 如果 ``{num} < 0`` 它意为 ``pool-num-replicas - {num}`` 。

:目的: 规则的一个组件。
:先决条件: 跟在 ``step take`` 或 ``step choose`` 之后。
:实例: ``step choose firstn 1 type row``


``step chooseleaf firstn {num} type {bucket-type}``

:描述: 选择 ``{bucket-type}`` 类型的一堆桶，并从各桶的子树里\
       选择一个叶子节点（即一个 OSD ）。集合内桶的数量通常是\
       所在存储池的副本数（即 pool size ）。

       - 如果 ``{num} == 0`` 选择 ``pool-num-replicas`` 个桶\
         （所有可用的）；
       - 如果 ``{num} > 0 && < pool-num-replicas`` 就选择那么\
         多的桶；
       - 如果 ``{num} < 0`` 意为 ``pool-num-replicas - {num}`` 。

:目的: 规则的一个组件。使用它之后就没必要分两步来选择一设备。
:先决条件: 跟在 ``step take`` 或 ``step choose`` 之后。
:实例: ``step chooseleaf firstn 0 type row``


``step emit``

:描述: 输出当前值并清空堆栈。通常用于规则末尾，但也能用于在\
       同一规则内选取别的树。

:目的: 规则的一个组件。
:先决条件: 在 ``step choose`` 之后。
:实例: ``step emit``

.. important:: 一条 CRUSH 规则可以分配给多个存储池使用，
   但是一个存储池不能同时配置多条 CRUSH 规则。


``firstn`` 对比 ``indep``

:描述: 控制着 CRUSH 图里的条目（ OSD ）被标记为 down 时 CRUSH 要用的替代策略。
       如果这条规则用于多副本存储池那就用 ``firstn`` ，
       如果用于纠删码存储池那就是 ``indep`` 。

       原因与前面选定的设备失败后、它们的反应行为有关。我们假设，
       你有一个 PG 存储在 OSD 1、2、3、4、5 上，然后 3 挂了。

       在 firstn 模式下， CRUSH 只是简单地调整一下\
       它选定 1 和 2 的计算方式，然后选择 3 但发现它挂了，
       所以它需要重试，选定了 4 和 5 ，然后继续选定新的 OSD 6 ，
       所以最终的 CRUSH 映射变更是 1, 2, 3, 4, 5 -> 1, 2, 4, 5, 6 。

       但是如果你用 EC 存储池存储的话，
       就意味着你只是把数据重新映射到 OSD 4、5 和 6 !
       所以 indep 模式尝试不那样做，或者说期待这样的结果，
       当它选择失败的 OSD 3 时，重试了一次并选定了 6 ，
       所以最终转换是 1, 2, 3, 4, 5 -> 1, 2, 6, 4, 5 。


.. _crush-reclassify:

从老式的 SSD 规则迁移到设备类
-----------------------------
.. Migrating from a legacy SSD rule to device classes

以前，为了给每种专门的设备类型（如 SSD ）维护一套并行的分级结构，
必须手动编辑 CRUSH 图，然后才能写规则应用这些设备。
从 Luminous 版起，通过 *device class* 功能可以透明地实现此需求了。

然而，从常规途径把现有的、手工定制的\
单设备图迁移到新的设备类规则，
将导致系统里的所有数据重新洗牌。

``crushtool`` 有一些命令可以转换旧的规则和分级结构，
这样你就能应用新的基于类的规则了。
有三种可能的转换方式：

#. ``--reclassify-root <root-name> <device-class>``

   这种方式会处理分级结构里根名字之下的所有内容，
   并校正引用这个根的所有规则，把 ``take <root-name>``
   改为 ``take <root-name> class <device-class>`` 。
   它会以这种方式对桶重新编号，
   用旧的 ID 作为指定类的“影子树（ shadow tree ）”，
   这样就不会发生数据迁移。

   例如，假设你现在的规则是这样的： ::

     rule replicated_rule {
        id 0
        type replicated
        step take default
        step chooseleaf firstn 0 type rack
        step emit
     }

   如果你把 root `default` 重新分类为 `hdd` 类，
   这条规则就变成了： ::

     rule replicated_rule {
        id 0
        type replicated
        step take default class hdd
        step chooseleaf firstn 0 type rack
        step emit
     }

#. ``--set-subtree-class <bucket-name> <device-class>``

   它会把以 *bucket-name* 为根的子树内的所有设备标记为\
   指定的设备类。

   通常和 ``--reclassify-root`` 选项一起使用，
   以确保那个根下的所有设备都被标记成正确的类。
   然而，有的时候，那些设备中的一些属于不同的类（正确的），
   我们不想重新标记它们；这时，
   你可以排除 ``--set-subtree-class`` 选项。
   这意味着重映射过程是不完美的，
   因为先前的规则跨越了多种类型的设备，
   而校正过的规则只映射到指定设备类 *device-class* 的设备上，
   这种异常设备数量不大的时候，由此导致的数据迁移处于可接受的水平。

#. ``--reclassify-bucket <match-pattern> <device-class> <default-parent>``

   此选项可以让你把一个并行的、特定类型的分级结构合并成普通的分级结构。
   例如，很多用户都有这样的图： ::

     host node1 {
        id -2           # do not change unnecessarily
        # weight 109.152
        alg straw2
        hash 0  # rjenkins1
        item osd.0 weight 9.096
        item osd.1 weight 9.096
        item osd.2 weight 9.096
        item osd.3 weight 9.096
        item osd.4 weight 9.096
        item osd.5 weight 9.096
        ...
     }

     host node1-ssd {
        id -10          # do not change unnecessarily
        # weight 2.000
        alg straw2
        hash 0  # rjenkins1
        item osd.80 weight 2.000
        ...
     }

     root default {
        id -1           # do not change unnecessarily
        alg straw2
        hash 0  # rjenkins1
        item node1 weight 110.967
        ...
     }

     root ssd {
        id -18          # do not change unnecessarily
        # weight 16.000
        alg straw2
        hash 0  # rjenkins1
        item node1-ssd weight 2.000
        ...
     }

   这个函数会重分类每个匹配到的桶。
   匹配的模式诸如 ``%suffix`` 或者 ``prefix%`` 。
   比如，在上面的例子中，我们可以用模式 ``%-ssd`` 。
   每一个匹配到的桶，其名字（匹配 ``%`` 通配符的）
   的其余部分表明了它的 *基础桶（ base bucket ）* 。
   匹配到的桶里的所有设备都被标记为指定的设备类，
   然后移进基础桶。如果基础桶不存在
   （例如 ``node12-ssd`` 存在但 ``node12`` 不存在），
   那么它会被创建并链接到指定的 *默认父节点* 下。
   不管哪种情形，我们都会为新的影子桶小心地保留旧的桶 ID ，以防数据移动。
   所有引用旧的桶、包含 ``take`` 的语句都会被校正。

#. ``--reclassify-bucket <bucket-name> <device-class> <base-bucket>``

   同样的命令，不用通配符可以映射单个桶。
   例如，在上述例子中，我们要把 ``ssd`` 桶映射到 ``default`` 桶。

综上所述，最终的转换命令可以这样写： ::

  $ ceph osd getcrushmap -o original
  $ crushtool -i original --reclassify \
      --set-subtree-class default hdd \
      --reclassify-root default hdd \
      --reclassify-bucket %-ssd ssd default \
      --reclassify-bucket ssd ssd default \
      -o adjusted

为了确保转换正确无误，可以用 ``--compare`` 命令来测试，
它可以向 CRUSH 图模拟输入大量样本，以确保会返回相同的结果。
输入选项和 ``--test`` 命令的一样。以上面的为例： ::

  $ crushtool -i original --compare adjusted
  rule 0 had 0/10240 mismatched mappings (0)
  rule 1 had 0/10240 mismatched mappings (0)
  maps appear equivalent

如果有不同的地方，你得看看输入中有多大比例被重映射了
（圆括号里）。

如果你对调整好的图满意了，就可以应用到集群，用： ::

  ceph osd setcrushmap -i adjusted


调整 CRUSH ，强硬方法
---------------------
.. Tuning CRUSH, the hard way

如果你能保证所有客户端都运行最新代码，你可以这样调整可调值：
从集群抽取 CRUSH 图、修改它们的值、重注入。

* 提抽取最新 CRUSH 图： ::

    ceph osd getcrushmap -o /tmp/crush

* 调整可调参数。这些值在我们测试过的大、小型集群上都有最佳表现。
  在极端情况下，你需要给 ``crushtool`` 额外指定 \
  ``--enable-unsafe-tunables`` 参数才行，
  用这个选项时需要万分谨慎。 ::

    crushtool -i /tmp/crush --set-choose-local-tries 0 --set-choose-local-fallback-tries 0 --set-choose-total-tries 50 -o /tmp/crush.new

* 重注入修改的图。 ::

    ceph osd setcrushmap -i /tmp/crush.new


遗留值
------
.. Legacy values

供参考，CRUSH 可调参数的遗留值可以用下面命令设置： ::

   crushtool -i /tmp/crush --set-choose-local-tries 2 --set-choose-local-fallback-tries 5 --set-choose-total-tries 19 --set-chooseleaf-descend-once 0 --set-chooseleaf-vary-r 0 -o /tmp/crush.legacy

再次申明， ``--enable-unsafe-tunables`` 是必需的，
而且前面也提到了，回退到遗留值后慎用旧版 ``ceph-osd`` 进程，
因为此功能位不是完全强制的。


.. _CRUSH - 可控、可伸缩、分布式地归置多副本数据: https://ceph.io/assets/pdfs/weil-crush-sc06.pdf
