==========
 CRUSH 图
==========

:abbr:`CRUSH (Controlled Replication Under Scalable Hashing)` 算法\
通过计算数据存储位置来确定如何存储和检索。
CRUSH 授权 Ceph 客户端直接连接 OSD ，
而非通过一个中央服务器或经纪人。
数据存储、检索算法的使用，使 Ceph 避免了\
单点故障、性能瓶颈、和伸缩的物理限制。

CRUSH 需要一张集群的地图（ CRUSH 图），
根据配置的复制策略和故障域把数据伪随机地映射到各个 OSD 。
关于 CRUSH 的详细讨论见 \
`CRUSH - 可控、可伸缩、分布式地归置多副本数据`_ 。

CRUSH 图包含一系列 :abbr:`OSD (对象存储设备，Object Storage Devices)` 、
汇聚设备和桶的“桶”分级结构、
和指示 CRUSH 在存储池内如何复制数据的规则。
由于反映了部署的底层物理组织方式，
CRUSH 能模型化、并因此定位到潜在的相关联设备故障，
典型的因素有机箱、机架、物理距离、共享电源、
和共享网络，把这些信息编码到集群运行图里，
CRUSH 归置策略可把对象副本分布到不同的故障域，
却仍能维持期望的分配情况。例如，
要定位同时失败的可能性，可能希望保证数据\
复制到的设备位于不同机架、不同托盘、不同电源、
不同控制器、甚至不同物理位置。

你在部署 OSD 时，它们自动被放入 CRUSH 图中的 ``host`` 节点下，\
其名字就是此 OSD 所在主机的主机名。这个，
加上默认的 CRUSH 故障域，
确保了副本或纠删码分片会跨主机散布，
而且单个主机的失效不会影响可用性。然而，
对于大一些的集群，管理员就应该谨慎地考虑故障域的选择；
例如，副本跨机柜散布，对很多中型到大型集群来说很常见。


CRUSH 位置
==========
.. CRUSH Location

用 CRUSH 图层次结构所表示的 OSD 位置被称为
“ crush 位置（crush location ）”，它用键/值对列表来表示。
例如，一 OSD 位于某特定行、机柜、机架、和主机，
且是 CRUSH 图里名为 default 树的一部分
（绝大多数集群都是这样的情形），
那么其 crush 位置可表示如下： ::

  root=default row=a rack=a2 chassis=a2a host=a2a1

注：

#. 注意键（关键词）与顺序无关；
#. 键名（ ``=`` 左边）必须是 CRUSH 内的合法 ``type`` ，默认情况下，
   它包含 ``root`` 、 ``datacenter`` 、 ``room`` 、 ``row`` 、 ``pod`` 、 ``pdu`` 、
   ``rack`` 、 ``chassis`` 、和 ``host`` 。
   这些预定义的类型能满足几乎所有集群，
   但可以修改 CRUSH 图自定义。
#. 并非所有键都需指定，例如，默认情况下 Ceph 会自动把
   ``ceph-osd`` 守护进程的位置设置为
   ``root=default host=HOSTNAME`` （即是 ``hostname -s`` ）。

一个 OSD 的 CRUSH 位置可以用 ``crush location`` 配置选项表示，\
写入 ``ceph.conf`` 文件。 OSD 每次启动时，
都会验证它还在 CRUSH 图内的正确位置，如果不是，就自己挪过去。
要禁用 CRUSH 图的这个自动管理功能，
把下面的加进配置文件的 ``[osd]`` 段下： ::

  osd crush update on start = false



定制位置挂钩
------------
.. Custom location hooks

定制的位置钩子可用于在启动时生成更完整的 crush 位置。
按先后顺序， CRUSH 位置是基于：

#. ``ceph.conf`` 内的 ``crush location`` 选项；
#. 默认的 ``root=default host=HOSTNAME`` ，其中主机名是用
   ``hostname -s`` 生成的。

可以写个脚本来提供额外的位置字段
（例如，机柜或数据中心），
然后通过配置选项启用这个钩子： ::

  crush location hook = /path/to/customized-ceph-crush-location

此挂钩应该接受几个参数（下述）并\
向标准输出打印一行 CRUSH 位置描述： ::

  --cluster CLUSTER --id ID --type TYPE

其中，集群名通常是 ``ceph`` ， ``id`` 是守护进程标识符
（例如， OSD 号或守护进程标识符），
守护进程类型是 ``osd`` 、 ``mds`` 之类的。

例如，下面这个简单的钩子，在 ``/etc/rack`` 里额外指定了一个机架位，
如下： ::

  #!/bin/sh
  echo "host=$(hostname -s) rack=$(cat /etc/rack) root=default"


CRUSH 结构
==========
.. CRUSH structure

不严谨地说， CRUSH 图包括，描述集群物理拓扑的一个分级结构、
和一系列规则（定义了如何在那些设备上放置数据的策略）。
这个分级结构包括，把叶子上挂设备（ ``ceph-osd`` 守护进程）、
各内部节点对应其它物理功能或分组：
主机、机架、排、数据中心，等等。
这些规则描述了副本们如何放置在分级结构里
（例如，“三个副本分别放到不同的机柜里”）。

设备
----
.. Devices

设备都是单个的 OSD ，可以存储数据，通常每个存储设备对应一个。
设备是用一个 id （非负整数）和名字标识的，
通常 ``osd.N`` 中的 ``N`` 是设备 id 。

从 Luminous 版起，设备还能分配一个 *device class*
（例如 ``hdd`` 或 ``ssd`` 或 ``nvme`` ），
方便让 CRUSH 规则引用。主机内混搭多种设备类型时尤其有用。

.. _crush_map_default_types:

类型和桶
--------
.. Types and Buckets

桶是 CRUSH 术语，是分级结构的内部节点：主机、机架、行等等。
CRUSH 图定义了一系列的 *类型* ，用于描述这些节点。
默认类型有：

- ``osd`` (或 ``device``)
- ``host``
- ``chassis``
- ``rack``
- ``row``
- ``pdu``
- ``pod``
- ``room``
- ``datacenter``
- ``zone``
- ``region``
- ``root``

大多数集群都只用了这些类型中的一小部分，
其它的可以按需定义。

这个分级结构的构成是：设备（类型通常是 ``osd`` ）位于叶子上、
类型为非设备的内部节点、和一个类型为 ``root`` 的根节点。例如，

.. ditaa::

                        +-----------------+
                        |{o}root default  |
                        +--------+--------+
                                 |
                 +---------------+---------------+
                 |                               |
          +------+------+                 +------+------+
          |{o}host foo  |                 |{o}host bar  |
          +------+------+                 +------+------+
                 |                               |
         +-------+-------+               +-------+-------+
         |               |               |               |
   +-----+-----+   +-----+-----+   +-----+-----+   +-----+-----+
   |   osd.0   |   |   osd.1   |   |   osd.2   |   |   osd.3   |
   +-----------+   +-----------+   +-----------+   +-----------+

分级结构里的每个节点（设备或桶）都有一个 *权重* ，
表示的是那个设备或分级结构子树会存储的总数据的相对比重。
权重是在叶子上设置的，表示的是设备的容量，
并且随树逐级累加，这样，
默认节点的权重就是它下面所有设备的总和。
通常，权重按 TB 数表示。

你可以用下面的命令查看集群的 CRUSH 分级结构，包括权重： ::

  ceph osd crush tree

规则
----
.. Rules

CRUSH 规则可以定义策略，
让数据在分级结构里的设备上如何分布。
它们定义归置和复制策略（或分配策略），
让你决定如何指挥 CRUSH 归置数据副本。
例如，你可以创建一条规则，让它选取一对目标做双路镜像；
另一条规则每次可以选取三个位于不同数据中心的目标做三路镜像；
还有一条规则，可以跨 6 个存储设备做纠删编码（ EC ）。
想了解更多关于 CRUSH 规则的论述，见
`CRUSH - 可控、可伸缩、分布式地归置多副本数据`_ ，
特别是 **Section 3.2** 。

CRUSH 规则可以通过 CLI 创建，
需要指定用于什么样的 *存储池类型* 
（多副本的或是纠删码的）、 *故障域* 、和可选的 *设备类别* 。
在极少数情况下，只能通过手动编辑 CRUSH 图\
手写规则。

你可以这样查看集群定义了哪些规则： ::

  ceph osd crush rule ls

你可以这样查看规则的内容： ::

  ceph osd crush rule dump

设备类别
--------
.. Device classes

每个设备都可以选择性地关联一个类别 *class* 。
默认情况下， OSD 们在启动时会根据其后端的设备类型\
自动将其类别设置为 `hdd` 、 `ssd` 或 `nvme` 。

以下命令可以设置一或多个 OSD 的设备类别： ::

  ceph osd crush set-device-class <class> <osd-name> [...]

设备类别配置后就不能再更改为另一个类别，
得先删掉其旧类别，用命令： ::

  ceph osd crush rm-device-class <osd-name> [...]

如此一来，管理员配置设备类别后，
就不会被 OSD 重启或其它脚本误改。

指向某个特定设备类别的归置规则可以这样创建： ::

  ceph osd crush rule create-replicated <rule-name> <root> <failure-domain> <class>

然后，存储池就可以改用新规则了： ::

  ceph osd pool set <pool-name> crush_rule <rule-name>

设备类别的实现是在现有类别之上再创建一个“影子” CRUSH 分级结构，
此类别中只包含了本类别下的设备。
这样，各规则就可以通过影子分级结构分发数据了。
这个实现方法的好处之一是，
它完全向后兼容老的 Ceph 客户端们。
CRUSH 分级结构的影子条目可以这样查看： ::

  ceph osd crush tree --show-shadow

对于 Luminous 之前比较老的集群，
它们靠手动更改的 CRUSH 图维护每个设备类型的分级结构，
有个 *reclassify （重分类）* 工具可以帮你转换成设备类，
还不会导致数据移动（见 :ref:`crush-reclassify` ）。


权重集
------
.. Weights sets

*权重集*\ 是计算数据归置时使用的另一种集。
常规权重与 CRUSH 图内各设备的尺寸相关联，
表明哪里\ *应该*\ 存储多少数据。
然而，由于 CRUSH 是“概率上的”伪随机归置过程，
总会各种变数干扰这种理想的分布，
道理和掷色子一样，掷 60 次不会正好是
10 个一点、和 10 个六点。
权重集可以让集群系统针对特定的集群
（层级、存储池、等等）做一些数字上的优化，
以实现更均衡的分布。

当前支持两种权重集：

 #. **compat （兼容的）** 权重集是集群里\
    各个设备和节点的一个单一的备选权重集。
    它不能完美纠正所有异常情况（例如，
    不同存储池的副本数不同、并且负载水平也不同，
    但均衡器却几乎公平地对待它们的归置组）。
    然而， compat 权重集的巨大优势是\
    它与先前版本的 Ceph 有 *向后兼容性* ，
    这意味着，即便权重集是在 Luminous v12.2.z 首次引进的，
    在集群使用 compat 权重集来均衡数据时，
    较老的客户端们（如 firefly ）仍然能连接到这个集群。
 #. **per-pool （基于存储池的）** 权重集更灵活，
    它允许单独优化各个数据存储池的归置。
    另外，可以针对归置的每个位置调整权重，
    这样优化程序通过微调其相对于互联节点的权重、
    进而纠正数据偏爱某些设备的情形
    （这种影响在非常巨大的集群上一般不明显，
    但却能导致均衡问题）。

启用了权重集、用命令查看时，
分级结构里与各节点关联的权重显示在单独的一列里
（标题是 ``(compat)`` 或是存储池名字）： ::

  ceph osd crush tree

*compat* 和 *per-pool* 两种权重集都在使用时，
如果某个存储池有它自己的 per-pool 权重集，那就使用它；
如果没有，但有 compat 权重集，那就用它；
如果都没有，就用普通的 CRUSH 权重。

虽然可以手动启用和修改权重集，
运行 Luminous 及更高版本时，
还是建议启用 ``ceph-mgr`` 的 *balancer* 模块自动完成。


修改 CRUSH 图
=============
.. Modifying the CRUSH map

.. _addosd:

增加/移动 OSD
-------------

.. note:: OSD 通常是在创建时自动被加进 CRUSH 图的。这个命令很少用到。

要增加或移动在线集群里 OSD 所对应的 CRUSH 图位置，执行： ::

  ceph osd crush set {name} {weight} root={root} [{bucket-type}={bucket-name} ...]

其中：


``name``

:描述: OSD 的全名。
:类型: String
:是否必需: Yes
:实例: ``osd.0``


``weight``

:描述: OSD 的 CRUSH 权重，通常是以 TB 计算的数值。
:类型: Double
:是否必需: Yes
:实例: ``2.0``


``root``

:描述: OSD 所在树的根节点（通常是 ``default`` ）。
:类型: Key/value pair.
:是否必需: Yes
:实例: ``root=default``


``bucket-type``

:描述: 定义 OSD 在 CRUSH 分级结构中的位置。
:类型: Key/value pairs.
:是否必需: No
:实例: ``datacenter=dc1 room=room1 row=foo rack=bar host=foo-bar-1``


下例把 ``osd.0`` 添加到分级结构里、或者说从前一个位置挪动一下。 ::

  ceph osd crush set osd.0 1.0 root=default datacenter=dc1 room=room1 row=foo rack=bar host=foo-bar-1


调整 OSD 的权重
---------------
.. Adjust OSD weight

.. note:: 通常在创建 OSD 时它们就会自动把自己加进 CRUSH 图、
   并且权重正确。这个命令很少用到。

要调整在线集群中一个 OSD 在 CRUSH 图中的 CRUSH 权重，执行命令： ::

  ceph osd crush reweight {name} {weight}

其中：


``name``

:描述: OSD 的全名。
:类型: String
:是否必需: Yes
:实例: ``osd.0``


``weight``

:描述: OSD 的 CRUSH权重。
:类型: Double
:是否必需: Yes
:实例: ``2.0``


.. _removeosd:

删除 OSD
--------
.. Remove an OSD

.. note:: 通常 OSD 会随 ``ceph osd purge`` 命令从 CRUSH 图中删除。
   这个命令很少用到。

要从在线集群里把一 OSD 踢出 CRUSH 图，执行命令： ::

  ceph osd crush remove {name}

其中：


``name``

:描述: OSD 全名。
:类型: String
:是否必需: Yes
:实例: ``osd.0``


增加桶
------
.. Add a Bucket

.. note:: 新加 OSD 时如果指定了 ``{bucket-type}={bucket-name}``
   这样的位置参数，而且没有那个名字的桶，就会隐式地创建这个桶。
   这个命令的典型用法是在 OSD 创建之后对分级结构进行手动调整。
   一种用途是把一系列主机移到一个新的、机架级的桶内；
   另一种用法是把新的 ``host`` 桶（ OSD 节点）挂到一个假的 ``root`` 下，
   它就不会接收数据，你准备好之后，
   就可以把它们移动到 ``default`` 或下文描述的其他根下面。

要在在线集群的 CRUSH 图中新建一个桶，用
``ceph osd crush add-bucket`` 命令： ::

  ceph osd crush add-bucket {bucket-name} {bucket-type}

其中：

``bucket-name``

:描述: 桶的全名。
:类型: String
:是否必需: Yes
:实例: ``rack12``


``bucket-type``

:描述: 桶的类型，它必须已存在于分级结构中。
:类型: String
:是否必需: Yes
:实例: ``rack``


下例把 ``rack12`` 桶加入了分级结构： ::

	ceph osd crush add-bucket rack12 rack

移动桶
------
.. Move a Bucket

要把一个桶挪动到 CRUSH 图里的不同位置，执行命令： ::

  ceph osd crush move {bucket-name} {bucket-type}={bucket-name}, [...]

其中：

``bucket-name``

:描述: 要移动或重新定位的桶名。
:类型: String
:是否必需: Yes
:实例: ``foo-bar-1``


``bucket-type``

:描述: 你可以指定桶在 CRUSH 分级结构里的位置。
:类型: Key/value pairs.
:是否必需: No
:实例: ``datacenter=dc1 room=room1 row=foo rack=bar host=foo-bar-1``

删除桶
------
.. Remove a Bucket

要把一个桶从 CRUSH 图的分级结构中删除，可用此命令： ::

  ceph osd crush remove {bucket-name}

.. note:: 从 CRUSH 分级结构里删除时必须是空桶。

其中：

``bucket-name``

:描述: 将要删除的桶的名字。
:类型: String
:是否必需: Yes
:实例: ``rack12``

下例从分级结构里删除了 ``rack12`` 。 ::

  ceph osd crush remove rack12

创建一个 compat 权重集
----------------------
.. Creating a compat weight set

.. note:: 启用 ``balancer`` 时，它通常会自动完成。

要创建一个 *compat* 权重集： ::

  ceph osd crush weight-set create-compat

compat 权重集的权重可以这样调整： ::

  ceph osd crush weight-set reweight-compat {name} {weight}

销毁 compat 权重集： ::

  ceph osd crush weight-set rm-compat

创建 per-pool 权重集
--------------------
.. Creating per-pool weight sets

为指定存储池创建权重集： ::

  ceph osd crush weight-set create {pool-name} {mode}

.. note:: Per-pool 权重集要求所有服务器和守护进程都运行
   Luminous v12.2.z 及以上版本。

其中：

``pool-name``

:描述: RADOS 存储池的名字。
:类型: String
:是否必需: Yes
:实例: ``rbd``

``mode``

:描述: 可以是 ``flat`` 或 ``positional`` 。
       *flat* 权重集为每个设备或桶都分配了单独的权重。
       *positional* 权重集可能给归置映射图里的各个位置分配不同的权重。
       例如，如果一个存储池的副本数是 3 ，
       那么它的 positional 权重集将是每个设备和桶都有 3 个权重。
:类型: String
:是否必需: Yes
:实例: ``flat``

调整一个权重集内一个条目的权重： ::

  ceph osd crush weight-set reweight {pool-name} {item-name} {weight [...]}

罗列现有的权重集： ::

  ceph osd crush weight-set ls

删除一个权重集： ::

  ceph osd crush weight-set rm {pool-name}


为多副本存储池创建规则
----------------------
.. Creating a rule for a replicated pool

对于多副本存储池，创建 CRUSH 规则时的主要关注点在于故障域是什么。
例如，如果选择 ``host`` 作为故障域，那么
CRUSH 就会确保将数据的各个副本存储到不同的主机上；
如果选择 ``rack`` ，那么各个副本将会存储到不同的机架上。
选择什么作为故障域主要取决于规模和集群的拓扑结构。

大多数情况下，整个集群的分级结构都嵌套在一个名为 ``default`` 的根节点下。
如果你自定义过分级结构，你也许想要创建一条规则，
嵌套在分级结构里的某个节点上。那个节点的类型是什么无关紧要
（不一定非得是 ``root`` 节点）。

还能创建一种规则，它只负责 *一类（ class ）* 设备的数据归置。
默认情况下， Ceph OSD 们会根据在用设备的类型把它自己归类为 ``hdd`` 或 ``ssd`` 。
这些类别也可以自定义。

创建一条多副本规则： ::

  ceph osd crush rule create-replicated {name} {root} {failure-domain-type} [{class}]

其中：

``name``

:描述: 规则的名字
:类型: String
:是否必需: Yes
:实例: ``rbd-rule``

``root``

:描述: 节点的名字，数据应该放置到这个节点之下。
:类型: String
:是否必需: Yes
:实例: ``default``

``failure-domain-type``

:描述: CRUSH 节点的类型，数据副本会跨过这些节点分布。
:类型: String
:是否必需: Yes
:实例: ``rack``

``class``

:描述: 数据应该放置到这种类型的设备上。
:类型: String
:是否必需: No
:实例: ``ssd``


为纠删码存储池创建规则
----------------------
.. Creating a rule for an erasure coded pool

对于纠删码（ EC ）存储池，有同样的基本关注点：
故障域是什么、数据放置到分级结构里的哪个节点下（通常是 ``default`` ）、
归置动作是否要限定在某一类设备上。
然而，纠删码存储池的创建有点不同，
因为要根据所用的纠删码小心地构建。
正因为这样， *纠删码配置信息* 里必须包含这个信息。
创建存储池时需要用到配置信息，
此时就会根据这些信息显式或自动地创建一条 CRUSH 规则。

纠删码配置信息可以这样罗列： ::

  ceph osd erasure-code-profile ls

现有配置信息可以这样查看： ::

  ceph osd erasure-code-profile get {profile-name}

正常情况下不要去修改配置信息；反之，
应该新建存储池时新建一份配置信息并应用它，
或者为现有存储池创建一条新规则。

一份纠删码配置信息由一系列 key=value 对组成。
其中的大多数都是为了控制纠删码的行为，让它在存储池中编码数据。
而那些以 ``crush-`` 打头的，
能够影响将要创建的这条 CRUSH 规则。

纠删码配置信息的众多属性里比较重要的有：

 * **crush-root**: 容纳数据的 CRUSH 节点的名字 [默认值: ``default``] ；
 * **crush-failure-domain**: 分配纠删编码的数据分片的 CRUSH 桶类型 [默认值: ``host``] ；
 * **crush-device-class**: 可以放置数据的设备类 [默认值: none, 表示使用所有设备] ；
 * **k** 和 **m** (对于 ``lrc`` 插件还有 **l**): 这些确定了纠删码分片的数量，
   影响着生成的 CRUSH 规则；

配置信息定义好之后，就可以用下列命令创建 CRUSH 规则： ::

  ceph osd crush rule create-erasure {name} {profile-name}

.. note:: 创建新存储池时，并非一定要显式地创建规则。
   如果只是指定了纠删码配置信息，
   而具体的规则参数却是空的， Ceph 会自动创建 CRUSH 规则。


规则的删除
----------
.. Deleting rules

没有存储池在用的规则可以这样删除： ::

  ceph osd crush rule rm {rule-name}


.. _crush-map-tunables:

可调选项
========
.. Tunables

时光荏苒，我们已经改进（并将继续改进）了用于计算数据位置的 CRUSH 算法。
为了体现（算法的）行为变化，我们引进了一系列可调选项，
以控制是使用老的、还是改进的算法。

要使用较新的可调选项，客户端和服务器都得支持较新版的 CRUSH 。
正因为如此，我们建立了一系列以 Ceph 版本命名的配置集
（ ``profiles`` ），比如，
``firefly`` 可调选项是在 firefly 版首次支持的，
而且不支持更老的（如 dumpling ）客户端。
一旦配置的可调选项生效、改变了旧有的默认行为，
``ceph-mon`` 和 ``ceph-osd`` 就不再允许那些老的、
不支持这些新 CRUSH 功能的客户端连接集群。


argonaut (遗老)
-----------------

argonaut 和更老版本的 CRUSH 工作方式对大多数集群来说都没问题，
也没有太多 OSD 被标记为 out 。


bobtail (CRUSH_TUNABLES2)
-------------------------

bobtail 可调选项修正了一些关键的错误行为：

 * 如果分级结构树的叶子桶内只有少量设备，
   某些 PG 映射的副本数小于期望值。
   这种情形通常出现在分级结构树中、
   某些 host 节点下面只挂少量（1-3个） OSD 时。

 * 在大型集群里，小部分 PG 映射到的 OSD 数目小于期望值，\
   有多层结构（如： ``row``, ``rack``, ``host``, ``osd`` ）时\
   这种情况更普遍。

 * 当一些 OSD 标记为 out 时，数据倾向于重分布到附近的 OSD
   而非整个分级结构树。

新的可调选项有：

 * ``choose_local_tries``: 本地重试次数。以前是 2 ，
   最优值是 0 。

 * ``choose_local_fallback_tries``: 以前 5 ，
   最优值是 0 。

 * ``choose_total_tries``: 选择一个条目的最大尝试次数。
   以前是 19 ，后来的测试表明，
   对典型的集群来说 50 更合适。
   最相当大的集群来说，也许有必要用更大的值。

 * ``chooseleaf_descend_once``: 是否重试递归选叶尝试，
   或只试一次、并允许最初的归置重试。
   以前默认为 0 ，最优为 1 。

对数据迁移的影响：

 * 可调选项从 ``argonaut`` 改为 ``bobtail`` 会引起一定量的数据迁移。
   在已经有数据的集群上需谨慎点。


firefly (CRUSH_TUNABLES3)
-------------------------

``firefly`` 可调选项对 ``chooseleaf`` 这条 CRUSH 规则的行为有所修正，
在太多 OSD 被标记为 out 状态时，用非常少的结果生成 PG
映射关系会有问题。

新的可调选项是：

 * ``chooseleaf_vary_r``: 根据父节点已做过多少尝试，
   递归选叶是否应该以非零值 ``r`` 开始。
   原先的默认值是 ``0`` ，但是用此值的话
   CRUSH 有时候会找不到映射关系；
   较优的值（计算代价和正确性合理）是 ``1`` 。

对数据迁移的影响：

 * 对于已经在运行、里面已经有了大量数据的集群，
   从 ``0`` 改为 ``1`` 会导致大量的数据迁移； ``4`` 或 ``5`` 时
   CRUSH 也能正确找到映射，而且数据迁移少的多。


straw_calc_version 可调选项（也是在 Firefly 引进的）
----------------------------------------------------

以前，给 ``straw`` 桶计算、并存储在 CRUSH 图里的内部权重有些问题。
特别是，如果有些条目 CRUSH 权重为 ``0`` 或者配置了多个权重、
重复配置了权重时， CRUSH 就不能正确地分布数据
（也就是与权重配比失衡）。

这个新可调参数是：

 * ``straw_calc_version``: 值为 ``0`` 时保留老的、有问题的内部\
   权重算法；值为 ``1`` 时修正此行为。

对数据迁移的影响：

 * straw_calc_version 改为 ``1`` 之后，
   *假如*\ 此集群触碰了某个雷区（ problematic condition ），
   调整 straw 桶（新增、删除、更改某一条目的权重、
   或用 reweight-all 命令更改所有权重）
   时就有可能引起少量或少部分数据迁移。

这个可调选项有些特殊，因为\
它对客户端的内核版本没任何要求。


hammer (CRUSH_V4)
-----------------

仅仅是更改配置的话， ``hammer`` 版的可调配置不会影响已有 CRUSH
图的映射关系。然而：

 * 增加了新型桶（ ``straw2`` ）的支持。
   这种新型的 ``straw2`` 桶解决了原来 ``straw`` 桶的几个局限性。
   具体来说，老的 ``straw`` 桶在\
   有权重变化时会改变一些映射关系，
   新的 ``straw2`` 桶实现了预定目标，
   只有在这些桶的权重发生变化时\
   才更改相应的映射关系。

 * ``straw2`` 是新建桶的默认类型。

对数据迁移的影响：

 * 把桶类型从 ``straw`` 改为 ``straw2`` 会导致少量数据迁移，
   这取决于桶内各条目的权重有多大落差。
   它们的权重都相同时就不会有数据迁移，
   各条目的权重落差巨大时就会有更多迁移。


jewel (CRUSH_TUNABLES5)
-----------------------

``jewel`` 可调配置提升了 CRUSH 的整体行为，
这样，在 OSD 被标记到集群外时\
可显著减少映射变化。
因此导致的数据移动少得多。

这个新可调参数是：

 * ``chooseleaf_stable``: 当某一 OSD 被标记为 out 时，
   递归选叶尝试是否把更优值代入内层递归、
   对于减少重映射具有重大影响。
   以前的数值是 ``0`` ，而新值 ``1`` 表示用新方法。

对数据迁移的影响：

 * 在已有集群上更改此值会导致海量数据迁移，
   因为几乎每个 PG 映射都可能改变。




哪些客户端版本支持 CRUSH_TUNABLES
---------------------------------

 * argonaut 系列， v0.48.1 或更高版
 * v0.49 或更高版
 * Linux 内核版本大于等于 v3.6 （对文件系统和 RBD 客户端都一样）

哪些客户端版本支持 CRUSH_TUNABLES2
----------------------------------

 * v0.55 或更高版，包括 bobtail 系列 (v0.56.x)
 * Linux 内核版本大于等于 v3.9 （对文件系统和 RBD 客户端都一样）

哪些客户端版本支持 CRUSH_TUNABLES3
----------------------------------

 * v0.78 (firefly) 或更高版
 * Linux 内核版本大于等于 v3.15 （对文件系统和 RBD 内核客户端来说）

哪些客户端版本支持 CRUSH_V4
---------------------------

 * v0.94 (hammer) 或更高版
 * Linux 内核版本大于等于 v4.1 （对文件系统和 RBD 内核客户端来说）

哪些客户端版本支持 CRUSH_TUNABLES5
----------------------------------

 * v10.0.2 (jewel) 或更高版
 * Linux 内核版本大于等于 v4.5 （对文件系统和 RBD 内核客户端来说）


可调选项非最优时发出警告
------------------------
.. Warning when tunables are non-optimal

从 v0.74 起，如果当前的 CRUSH 可调选项没囊括 ``default`` 配置集里的\
所有最优值（ ``default`` 配置集的含义见下文），
Ceph 就会发出健康警告，有两种方法可消除这些警告：

#. 调整现有集群上的可调选项。注意，
   这可能会导致一些数据迁移（可能有 10% 之多）。
   这是推荐的办法，但是在生产集群上要注意\
   此调整对性能带来的影响。
   此命令可启用较优可调选项： ::

     ceph osd crush tunables optimal

   如果切换得不太顺利（如负载太高）且切换才不久，
   或者有客户端兼容问题（较老的 cephfs 内核驱动或 rbd 客户端、
   或早于 bobtail 的 ``librados`` 客户端），
   你可以这样切回： ::

	ceph osd crush tunables legacy

#. 不对 CRUSH 做任何更改也能消除报警，把下列配置加入
   ``ceph.conf`` 的 ``[mon]`` 段下： ::

     mon warn on legacy crush tunables = false

   为使变更生效需重启所有监视器，
   或者执行下列命令： ::

     ceph tell mon.\* config set mon_warn_on_legacy_crush_tunables false


一些要点
--------

 * 调整这些值将使一些 PG 在存储节点间移位，
   如果 Ceph 集群已经存储了大量数据，
   做好移动一部分数据的准备。
 * 一旦更新运行图， ``ceph-osd`` 和 ``ceph-mon``
   就会开始向新建连接要求功能位，
   然而，之前已经连接的客户端如果\
   不支持新功能将行为失常。
 * 如果 CRUSH 可调值更改过、然后又改回了默认值，
   ``ceph-osd`` 守护进程将不要求支持此功能，
   然而， OSD 连接建立进程要能检查和理解旧地图。
   因此，集群如果用过非默认 CRUSH 值就不应该\
   再运行版本小于 0.48.1 的 ``ceph-osd`` ，
   即使最新版地图已经回滚到了\
   遗留默认值。


调整 CRUSH
----------
.. Tuning CRUSH

更改 CRUSH 可调值的最简方法就是应用一个配置集，也就是 *profile* ，
到 Octopus 为止有：

 * ``legacy``: 采用 argonaut 及更低版本的行为；
 * ``argonaut``: 采用 argonaut 版最初的配置；
 * ``bobtail``: 采用 bobtail 版的配置；
 * ``firefly``: 采用 firefly 版的配置；
 * ``hammer``: hammer 版支持的值
 * ``jewel``: jewel 版支持的值
 * ``optimal``: 当前 Ceph 版本的最佳（即最优的）值；
 * ``default``: 从头安装的新集群的默认值。
   这些值依附于当前的 Ceph 版本，
   是写死的（ hard coded ），
   而且通常是最优值和遗留值的混合体。
   这些值通常对应于前一个 LTS 版本的 ``optimal`` 配置集，
   或者是最近的、我们预计大多数用户都不会升级过来的版本。

你可以在运行着的集群上选择一个配置： ::

	ceph osd crush tunables {PROFILE}

要注意，这可能产生数据迁移，可能还不少。\
在运行着的集群上更改此配置前，请仔细研究发布说明和文档，\
并试着压制一下恢复、回填参数，以降低一大波回填造成的影响。


.. _CRUSH - 可控、可伸缩、分布式地归置多副本数据: https://ceph.com/wp-content/uploads/2016/08/weil-crush-sc06.pdf


主亲和性
========
.. Primary Affinity

一个 Ceph 客户端读取或写入数据时，它首先连接相关 PG 的
acting set 里的主 OSD 。默认情况下，acting set 里的第一个 OSD 是主的。\
比如，在 acting set ``[2, 3, 4]`` 中， ``osd.2`` 位列第一，\
所以它就是是主 OSD （又名 lead ）。\
有时候我们知道某个 OSD 作为 lead 时不如其余的\
（比如它的硬盘慢、或控制器慢），\
为了防止出现性能瓶颈（特别是读操作），同时又能最大化地利用硬件，\
你可以调整 OSD 的主亲和性参数值、\
或定制一条 CRUSH 规则优先选择喜欢的的 OSD 们，以此来影响主 OSD 的选举。

调整主 OSD 的选择主要对多副本存储池有用，
因为默认情况下，读操作服务由每个 PG 的主 OSD 提供。
对于纠删码（ EC ）存储池，提速读操作的一种途径是启用 *快速读（ fast read ）* ，
会在 :ref:`pool-settings` 里详述。

主亲和性的一个常见应用场景是集群包含的驱动器容量大小不一时，
例如，较老的机架配备的是 1.9 TB 的 SATA SSD 、而较新的机架是 3.84TB SATA SSD 。
平均下来，后者将会分配到双倍的 PG 、
因而将需要提供双倍的读写操作，
因此它们会比前者忙碌得多。粗略地按照 OSD 容量的反比例\
分配主亲和性也未必是 100% 最优的，
但它能够很轻松地实现 15% 的整体读吞吐量提升，
因为更平均地利用了 SATA 接口带宽和 CPU 周期。

默认情况下， Ceph 所有 OSD 的主亲和性都是 ``1`` ，
表示所有 OSD 成为主 OSD 的几率都是相等的。

你可以降低 Ceph OSD 的主亲和性，这样 CRUSH 就不太会\
把这样的 OSD 选为 PG 的 acting set 里的主 OSD 了： ::

	ceph osd primary-affinity <osd-id> <weight>

你可以把某一 OSD 的主亲和性设置为 ``[0-1]`` 范围内的实数，\
其中， ``0`` 表示此 OSD **不能** 用作主的，\
而 ``1`` 表示这个 OSD 可以用作主的。这个权重介于二者之间时，
CRUSH 把那个 OSD 选为主 OSD 的可能性较低。\
选中一个 lead OSD 的过程不仅仅是个简单的、\
基于相对亲和值的概率问题，\
而是一系列可测量的结果，进而取得期望值的一阶近似值。


自定义 CRUSH 规则
-----------------
.. Custom CRUSH Rules

有的集群为了均衡成本和性能，会在同一个多副本存储池里混搭 SSD 和 HDD 。
然后把 HDD OSD 们的主亲和性设置为 ``0`` ，就可以把操作都引导到\
各个 acting set 里的 SSD 上。另一个办法是定义一条 CRUSH 规则，
只选用 SSD OSD 作为第一个 OSD 、然后其余 OSD 选用 HDD 。这样，
每个 PG 的 acting set 就会只包含一个 SSD OSD 作为主的，以此实现了和 HDD 的平衡。

以下面的 CRUSH 规则为例： ::

	rule mixed_replicated_rule {
	        id 11
	        type replicated
	        min_size 1
	        max_size 10
	        step take default class ssd
	        step chooseleaf firstn 1 type host
	        step emit
	        step take default class hdd
	        step chooseleaf firstn 0 type host
	        step emit
	}

它选了一个 SSD 作为第一个 OSD 。注意，对于 ``N`` 个副本的存储池，
本规则会选用 ``N+1`` 个 OSD 来保证 ``N`` 个副本位于不同的主机上，
因为第一个 SSD OSD 可能和 ``N`` 个 HDD OSD 中的某一个位于\
同一台主机上。

额外的存储空间需求也可以避免，需要折衷一下，
把 SSD 和 HDD 分别放到不同的主机上，
持有 SSD 的主机需要接收所有客户端的请求。为此，
你可以为 SSD 主机配备更快的 CPU 、为 HDD 节点配备更稳定的，
因为后者通常只有恢复服务的操作。这里的 CRUSH 根
``ssd_hosts`` 和 ``hdd_hosts`` 必须严格地不包含相同的服务器： ::

        rule mixed_replicated_rule_two {
               id 1
               type replicated
               min_size 1
               max_size 10
               step take ssd_hosts class ssd
               step chooseleaf firstn 1 type host
               step emit
               step take hdd_hosts class hdd
               step chooseleaf firstn -1 type host
               step emit
        }



还需注意，在 SSD 故障时，
到 PG 的请求临时会由（较慢的） HDD OSD 接替，
直到 PG 的数据复制到接替它的主 SSD OSD 上。

