.. _rados-crush-map:

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

.. note::

   #. 注意键（关键词）与顺序无关；
   #. 键名（ ``=`` 左边）必须是 CRUSH 内的合法 ``type`` ，默认情况下，
      它包含 ``root`` 、 ``datacenter`` 、 ``room`` 、 ``row`` 、
      ``pod`` 、 ``pdu`` 、 ``rack`` 、 ``chassis`` 、和 ``host`` 。
      这些预定义的类型能满足几乎所有集群，
      但可以修改 CRUSH 图自定义。

一个 OSD 的 CRUSH 位置可以在 ``ceph.conf`` 配置文件里加上
``crush_location`` 选项来设置，例如： ::

   crush_location = root=default row=a rack=a2 chassis=a2a host=a2a1

如果加了这个选项，以后 OSD 每次启动时，
都会验证它还在 CRUSH 图内的正确位置，
如果不是，就自己挪过去。
要禁用 CRUSH 图的这个自动管理功能，
把下面的加进 ``ceph.conf`` 配置文件的 ``[osd]`` 段下： ::

   osd_crush_update_on_start = false

注意，大多数情况下都没必要配置这个。

如果没有明确设置 ``crush_location`` ，
OSD 们的默认位置就是 ``root=default host=HOSTNAME`` ，
其中，主机名是 ``hostname -s`` 命令的输出结果。

.. note:: 如果将默认设置改为明确设置的 ``crush_location`` ，
   不要忘了包含 ``root=default`` ，因为现有的 CRUSH 规则会引用它。


定制位置挂钩
------------
.. Custom location hooks

定制的位置钩子可用于在启动时生成更完整的 crush 位置。

如果在编写 ``ceph.conf`` 时不知道某些位置字段（例如，
在多个数据中心部署单套配置时的机架 ``rack`` 或\
数据中心 ``datacenter`` 字段），那么这个钩子就非常有用。

如果配置、执行和解析成功，钩子的输出将取代之前设置的任何 CRUSH 位置。

可以在 ``ceph.conf`` 中提供可执行文件（通常是脚本）的路径\
来启用钩子，例如： ::

   crush_location_hook = /path/to/customized-ceph-crush-location

此挂钩应该接受几个参数（下述）。
并向标准输出 ``stdout`` 打印一行
CRUSH 位置描述。参数大概是下面的样子： ::

  --id ID --type TYPE

其中， ``id`` 是守护进程标识符或者
（如果是 OSD 的话）是 OSD 号，
守护进程类型是 ``osd`` 、 ``mds`` 、 ``mgr`` 或者 ``mon`` 。

例如，下面这个简单的钩子，在 ``/etc/rack`` （假设它没有空格）\
里额外指定了一个机架位，如下： ::

  #!/bin/sh
  echo "root=default rack=$(cat /etc/rack) host=$(hostname -s)"


CRUSH 结构
==========
.. CRUSH structure

CRUSH 图包括， (1) 描述集群物理拓扑的一个分级结构，和 (2) 定义数据归置策略的\
一系列规则。这个分级结构包括，叶子上挂的设备（ OSD ）、
内部节点对应其它物理功能或分组：主机、机架、排、数据中心，等等。
这些规则描述了副本们如何放置在分级结构里
（例如，“三个副本分别放到不同的机柜里”）。

设备
----
.. Devices

设备（ device ）都是用来存数据的单个 OSD （通常一个设备对应一个存储驱动器）。
设备是用一个 ``id`` （非负整数）和名字 ``name`` 标识的
（通常是 ``osd.N`` ，其中的 ``N`` 是设备的 ``id`` ）。

从 Luminous 版起，设备还能分配一个 *device class （设备类别）*
（例如 ``hdd`` 或 ``ssd`` 或 ``nvme`` ），方便让 CRUSH 规则引用。
主机内混搭多种设备类型时设备类别尤其有用。

.. _crush_map_default_types:

类型和桶
--------
.. Types and Buckets

桶（ bucket ），在 CRUSH 这里，是分级结构的内部节点：主机、机架、行等等。
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

大多数集群都只用了这些类型中的一小部分，其它的可以按需定义。

这个分级结构的构成是：设备（类型通常是 ``osd`` ）位于叶子上、
类型为非设备的内部节点、和一个类型为 ``root`` 的根节点。
例如，


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


分级结构里的每个节点（设备或桶）都有一个 *权重（ weight ）* ，
表示的是那个设备或分级结构子树会存储的总数据的相对比重。
权重是在叶子上设置的，表示的是设备的容量，
并且随树逐级累加，这样，
默认节点的权重就是它下面所有设备的总和。
通常，权重按 TB 数表示。

你可以用下面的命令查看集群的 CRUSH 分级结构，包括权重：

.. prompt:: bash $

   ceph osd tree

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

你可以这样查看集群定义了哪些规则：

.. prompt:: bash $

   ceph osd crush rule ls

你可以这样查看规则的内容：

.. prompt:: bash $

   ceph osd crush rule dump

.. _device_classes:

设备类别
--------
.. Device classes

每个设备都可以选择性地关联一个类别 *class* 。
默认情况下， OSD 们在启动时会根据其后端的设备类型\
自动将其类别设置为 `hdd` 、 `ssd` 或 `nvme` 。

以下命令可以设置一或多个 OSD 的设备类别：

.. prompt:: bash $

   ceph osd crush set-device-class <class> <osd-name> [...]

设备类别配置后就不能改成另一个类别，必须先取消其旧类别。
按下面的格式用命令删除一个或多个 OSD 的旧类别：

.. prompt:: bash $

   ceph osd crush rm-device-class <osd-name> [...]

如此一来，管理员配置设备类别后，
就不会被 OSD 重启或其它脚本误改。

指向某个特定设备类别的归置规则可以这样创建，
命令格式如下：

.. prompt:: bash $

   ceph osd crush rule create-replicated <rule-name> <root> <failure-domain> <class>

给指定存储池应用新规则，命令如下：

.. prompt:: bash $

  ceph osd pool set <pool-name> crush_rule <rule-name>

设备类别的实现方式是在现有类别之上再创建一个“影子” CRUSH 分级结构，
此类别中只包含了本类别下的设备。
这样，各 CRUSH 规则就可以通过对应的影子分级结构分发数据了。
这个实现方法的好处之一是，
它完全向后兼容老的 Ceph 客户端们。
CRUSH 分级结构的影子条目可以这样查看： ::

.. prompt:: bash #

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

#. **compat （兼容的）** 权重集是集群里各个设备和节点的一个单一的备选权重集。
   它不能完美纠正所有异常情况（例如，
   不同存储池的副本数不同、并且负载水平也不同，
   但均衡器却几乎公平地对待它们的归置组）。
   然而， compat 权重集的巨大优势是它与先前版本的 Ceph 有 *向后兼容性* ，
   这意味着，即便权重集是在 Luminous v12.2.z 首次引进的，
   在集群使用 compat 权重集来均衡数据时，
   较老的客户端们（如 firefly ）仍然能连接到这个集群。

#. **per-pool （基于存储池的）** 权重集更灵活，
   它允许单独优化各个数据存储池的归置。
   另外，可以针对归置的每个位置调整权重，
   这样优化程序通过微调其相对于互联节点的权重、
   进而纠正数据偏爱某些设备的情形
   （这种影响在非常巨大的集群上一般不明显，但却能导致均衡问题）。

启用了权重集、用命令查看时，分级结构里与各节点关联的权重显示在单独的一列里
（标题是 ``(compat)`` 或是存储池名字）：

.. prompt:: bash #

   ceph osd tree

*compat* 和 *per-pool* 两种权重集都在使用时，
如果某个存储池有它自己的 per-pool 权重集，那就使用它；
如果没有，但有 compat 权重集，那就用它；
如果都没有，就用普通的 CRUSH 权重。

虽然可以手动启用和修改权重集，运行 Luminous 及更高版本时，
还是建议启用 ``ceph-mgr`` 的 *balancer* 模块自动完成。


修改 CRUSH 图
=============
.. Modifying the CRUSH map

.. _addosd:

增加/移动 OSD
-------------

.. note:: 正常情况下， OSD 是在创建时自动加入 CRUSH 图的。
   这一段里的命令很少用到。


要增加或移动在线集群里 OSD 所对应的 CRUSH 图位置，执行命令：

.. prompt:: bash $

   ceph osd crush set {name} {weight} root={root} [{bucket-type}={bucket-name} ...]

命令参数的详细含义如下：

``name``
    :描述: OSD 的全名。
    :类型: String
    :是否必需: Yes
    :实例: ``osd.0``


``weight``
    :描述: OSD 的 CRUSH 权重，就是它的尺寸，是以 TB 计算的数值。
    :类型: Double
    :是否必需: Yes
    :实例: ``2.0``


``root``
    :描述: OSD 所在 CRUSH 分级结构的根节点（通常是 ``default`` ）。
    :类型: Key/value pair.
    :是否必需: Yes
    :实例: ``root=default``


``bucket-type``
    :描述: 定义 OSD 在 CRUSH 分级结构中的位置。
    :类型: Key/value pairs.
    :是否必需: No
    :实例: ``datacenter=dc1 room=room1 row=foo rack=bar host=foo-bar-1``


下例把 ``osd.0`` 添加到分级结构里、或者说从前一个位置挪动一下。

.. prompt:: bash $

   ceph osd crush set osd.0 1.0 root=default datacenter=dc1 room=room1 row=foo rack=bar host=foo-bar-1


调整 OSD 的权重
---------------
.. Adjusting OSD weight

.. note:: 正常情况下， OSD 是在创建时自动加入 CRUSH 图的。
   这一段里的命令很少用到。

要调整在线集群中一个 OSD 的 CRUSH 权重，执行命令：

.. prompt:: bash $

   ceph osd crush reweight {name} {weight}

命令参数的详细含义如下：

``name``
    :描述: OSD 的全名。
    :类型: String
    :是否必需: Yes
    :实例: ``osd.0``


``weight``
    :描述: OSD 的 CRUSH 权重。
    :类型: Double
    :是否必需: Yes
    :实例: ``2.0``


.. _removeosd:

删除 OSD
--------
.. Removing an OSD

.. note:: 通常 OSD 会随 ``ceph osd purge`` 命令从 CRUSH 图中删除。
   这个命令很少用到。

要从在线集群里把一 OSD 踢出 CRUSH 图，执行命令： ::

.. prompt:: bash $

   ceph osd crush remove {name}

命令参数的详细含义如下：

``name``
    :描述: OSD 全名。
    :类型: String
    :是否必需: Yes
    :实例: ``osd.0``


增加桶
------
.. Adding a CRUSH Bucket

.. note:: 新加 OSD 时如果指定了 ``{bucket-type}={bucket-name}``
   这样的位置参数，而且没有那个名字的桶，就会隐式地创建这个桶。
   这个命令的典型用法是在 OSD 创建之后对分级结构进行手动调整。
   一种用途是把一系列主机移到一个新的、机架级的桶内；
   另一种用法是把新的 ``host`` 桶（ OSD 节点）挂到一个假的 ``root`` 下，
   它就不会接收数据，你准备好之后，
   就可以把它们移动到 ``default`` 或下文描述的其他根下面。

要在在线集群的 CRUSH 图中新建一个桶，用命令：

.. prompt:: bash $

   ceph osd crush add-bucket {bucket-name} {bucket-type}

命令参数的详细含义如下：

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

下例把 ``rack12`` 桶加入了分级结构：

.. prompt:: bash $

   ceph osd crush add-bucket rack12 rack

移动桶
------
.. Moving a Bucket

要把一个桶挪动到 CRUSH 图里的不同位置，执行命令：

.. prompt:: bash $

   ceph osd crush move {bucket-name} {bucket-type}={bucket-name}, [...]

命令参数的详细含义如下：

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

重命名桶
--------
.. Renaming a bucket

要重新命名一个桶，同时保持其在 CRUSH 图层次结构中的位置不变，
用命令：

.. prompt:: bash #

   ceph osd crush rename-bucket {oldname} {newname}

删除桶
------
.. Removing a Bucket

要把一个桶从 CRUSH 图分级结构中删除，可用此命令：

.. prompt:: bash $

   ceph osd crush remove {bucket-name}

.. note:: 从 CRUSH 分级结构里删除时必须是空桶，
   换句话说，里面一定不能有 OSD 或者其它的 CRUSH 桶。

命令参数的详细含义如下：

``bucket-name``
    :描述: 将要删除的桶的名字。
    :类型: String
    :是否必需: Yes
    :实例: ``rack12``

下例从分级结构里删除了 ``rack12`` ：

.. prompt:: bash $

   ceph osd crush remove rack12


创建一个 compat 权重集
----------------------
.. Creating a compat weight set

.. note:: 通常， ``balancer`` 模块认为有必要时会自动完成此动作
   （前提是此模块已启用）。

要创建一个 *compat* 权重集：

.. prompt:: bash $

   ceph osd crush weight-set create-compat

调整 compat 权重集的权重，用下列命令：

.. prompt:: bash $

   ceph osd crush weight-set reweight-compat {name} {weight}

销毁 compat 权重集，用下列命令：

.. prompt:: bash $

   ceph osd crush weight-set rm-compat


创建 per-pool 权重集
--------------------
.. Creating per-pool weight sets

为指定存储池创建权重集，用下列命令：

.. prompt:: bash $

   ceph osd crush weight-set create {pool-name} {mode}

.. note:: Per-pool 权重集要求所有服务器和守护进程都运行
   Luminous v12.2.z 及以上版本。

命令参数的详细含义如下：

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

调整一个权重集内一个条目的权重，用下列命令：

.. prompt:: bash $

   ceph osd crush weight-set reweight {pool-name} {item-name} {weight [...]}

罗列现有的权重集，用下列命令：

.. prompt:: bash $

  ceph osd crush weight-set ls

删除一个权重集，用下列命令：

.. prompt:: bash $

  ceph osd crush weight-set rm {pool-name}


为多副本存储池创建规则
----------------------
.. Creating a rule for a replicated pool

对于多副本存储池，创建 CRUSH 规则时重要的关注点在于：故障域是什么。
例如，如果选择 ``host`` 作为故障域，
那么 CRUSH 就会确保将数据的各个副本存储到不同的主机上；
或者，如果选择 ``rack`` ，
那么各个数据副本将会存储到不同的机架上。
选择什么作为故障域主要取决于集群规模和它的 CRUSH 拓扑结构。

大多数情况下，整个集群的分级结构都嵌套在一个名为 ``default`` 的根节点下。
如果你自定义过分级结构，你也许想要创建一条规则，
嵌套在分级结构里的某个节点上。
在自定义的分级结构里创建规则时，那个节点的类型是什么无关紧要，
而且，此规则不一定非得嵌套在 ``root`` 节点下。

还能创建一种规则，它只负责 *一类（ class ）* 设备的数据归置。
默认情况下， Ceph OSD 们会根据在用设备的类型\
把它自己归类为 ``hdd`` 或 ``ssd`` 。
这些设备类别也可以自定义。
我们可以将 OSD 的设备类别 ``device class`` 设置为 ``nvme`` ，
以区别于 SATA SSD ；或者，将其随便设置为诸如
``ssd-testing`` 或 ``ssd-ethel`` 这样的类别，
这样就可以根据特定要求灵活地限制规则和存储池，
以使用（或避免使用）特定的 OSD 子集。

要给多副本存储池创建一条规则，用下列命令：

.. prompt:: bash $

   ceph osd crush rule create-replicated {name} {root} {failure-domain-type} [{class}]

命令参数的详细含义如下：

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

罗列纠删码配置信息，用下列命令：

.. prompt:: bash $

   ceph osd erasure-code-profile ls

查看一个现有配置信息，用下列命令：

.. prompt:: bash $

   ceph osd erasure-code-profile get {profile-name}

正常情况下不要去修改配置信息；反之，
应该新建存储池时新建一份配置信息并应用它，
或者为现有存储池创建一条新规则。

一份纠删码配置信息由一系列 key=value 对组成。
其中的大多数都是为了控制纠删码的行为，让它在存储池中编码数据。
而那些以 ``crush-`` 打头的，
能够影响将要创建的这条 CRUSH 规则。

纠删码配置信息的众多属性里比较重要的有：

 * **crush-root**: 容纳数据的 CRUSH 节点的名字
   [默认值: ``default``] ；
 * **crush-failure-domain**: 分配纠删编码的数据分片的 CRUSH 桶类型
   [默认值: ``host``] ；
 * **crush-osds-per-failure-domain**: 每个故障域里最多几个 OSD ，
   默认是 1 。取值大于 1 的话，
   会导致创建一个 CRUSH MSR 规则，见下文。
   如果配置了 ``crush-num-failure-domains`` ，那就必须配置这个选项。
 * **crush-num-failure-domains**: 要映射的故障域数量。
   如果配置了 ``crush-osds-per-failure-domain`` ，那就必须配置这个选项。
   会导致创建一个 CRUSH MSR 规则。
 * **crush-device-class**: 可以放置数据的设备类
   [默认值: none, 表示使用所有设备] ；
 * **k** 和 **m** (对于 ``lrc`` 插件还有 **l**): 这些确定了纠删码分片的数量，
   影响着生成的 CRUSH 规则；

配置信息定义好之后，就可以创建 CRUSH 规则了，
用下列命令：

.. prompt:: bash $

   ceph osd crush rule create-erasure {name} {profile-name}

.. note:: 创建新存储池时，没必要明确地创建规则。
   如果只是指定了纠删码配置信息，
   而具体的规则参数却是空的， Ceph 会自动创建 CRUSH 规则。


CRUSH MSR 规则
--------------
.. CRUSH MSR Rules

创建纠删码配置文件时，如果 ``crush-osds-per-failure-domain`` 值大于 1 ，
会导致创建 CRUSH MSR 类型的规则，而不是普通的 CRUSH 规则。
当遇到 OSD 故障时，普通 CRUSH 规则无法重试之前的步骤，
只能依靠 CHOOSELEAF 步骤将 OSD 转移到新主机。
然而， CHOOSELEAF 规则却不允许每个故障域超过一个 OSD 。
在 squid 版中新加入了 MSR 规则，
它允许一个故障域里有多个 OSD ，
此规则允许在遇到 OSD 掉线时重试所有先前的步骤。
使用 MSR 规则要求所有 OSD 和客户端们都支持 CRUSH_MSR 功能位
（在 squid 以及更高版本上）。


规则的删除
----------
.. Deleting rules

要删除没有存储池在用的规则，用下列命令：

.. prompt:: bash $

   ceph osd crush rule rm {rule-name}


.. _crush-map-tunables:

可调选项
========
.. Tunables

时光荏苒，我们已经改进（并将继续改进）了用于计算数据位置的 CRUSH 算法。
为了体现（算法的）行为变化，我们引进了一系列可调选项，
以控制是使用老的、还是改进的算法。

要使用较新的可调选项，所有 Ceph 客户端和守护进程\
都得支持较新版的 CRUSH 。正因为如此，
我们建立了一系列以 Ceph 版本命名的配置集（ ``profiles`` ）。
比如，``firefly`` 可调选项是在 firefly 版首次支持的，
而且不支持较老的客户端（如 dumpling 版的客户端）。
一套集群的可调选项配置更改后，比如从较老的升级到较新的\
或更优化的（ ``optimal`` ）配置集，
``ceph-mon`` 和 ``ceph-osd`` 会阻止那些老的、
不支持这些新 CRUSH 功能的客户端连接集群。


argonaut (遗老)
-----------------

argonaut 和更老版本的 CRUSH 工作方式对大多数集群来说都没问题，
也没有太多 OSD 被标记为 ``out`` 。


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

chooseleaf_vary_r
~~~~~~~~~~~~~~~~~

``firefly`` 可调选项对 ``chooseleaf`` 这条 CRUSH 规则的行为有所修正。
在太多 OSD 被标记为 ``out`` 状态时这个问题会浮现，
会导致 PG 和极少的几个 OSD 映射。

此配置是在 Firefly 版引进的，新增了如下这个可调选项：

 * ``chooseleaf_vary_r``: 根据父节点已做过多少尝试，
   递归选叶是否应该以非零值 ``r`` 开始。
   原先的默认值是 ``0`` ，但是用此值的话
   CRUSH 有时候会找不到映射关系；
   较优的值（计算代价和正确性合理）是 ``1`` 。

对数据迁移的影响：

 * 对于已经在运行、里面已经有了大量数据的集群，
   从 ``0`` 改为 ``1`` 会导致大量的数据迁移；
   ``4`` 或 ``5`` 时 CRUSH 也能正确找到映射，而且数据迁移少很多。


straw_calc_version 可调选项
---------------------------

以前，给 ``straw`` 算法桶计算出、并存储在 CRUSH 图里的内部权重有些问题。
当有些条目的 CRUSH 权重为 ``0`` 或者混合了不同的唯一权重时，
CRUSH 就不能正确地分布数据
（也就不按照权重比例分配数据）。

这个可调选项是 Firefly 版引进的，如下：

 * ``straw_calc_version``: 值为 ``0`` 时保留老的、
   有问题的内部权重算法；值为 ``1`` 时修正此行为。

对数据迁移的影响：

 * 这个可调选项改为 ``1`` 之后，
   *假如*\ 此集群触碰了某个雷区（ problematic condition ），
   调整 straw 桶（新增、删除、更改某一条目的权重、
   或用 reweight-all 命令更改所有权重）
   时就有可能引起少量或少部分数据迁移。

这个可调选项有些特殊，因为\
它对客户端的内核版本没任何要求。


hammer (CRUSH_V4)
-----------------

仅仅是更改配置的话， ``hammer`` 版的可调配置不会影响\
已有 CRUSH 图的映射关系。然而：

 * 支持了新的桶算法 ``straw2`` 。
   这种新算法解决了原来 ``straw`` 桶的几个局限性。
   具体来说，老的 ``straw`` 桶在\
   有权重变化时会改变一些本来不应该改变的映射关系，
   而 ``straw2`` 可实现预定目标，
   即只改变权重发生变化的那些桶的映射关系。

 * ``straw2`` 是新建桶的默认类型。

对数据迁移的影响：

 * 把桶类型从 ``straw`` 改为 ``straw2`` 会导致少量数据迁移，
   这取决于桶内各条目的权重有多大落差。
   它们的权重都相同时就不会有数据迁移，
   各条目的权重落差巨大时就会有更多迁移。


jewel (CRUSH_TUNABLES5)
-----------------------

``jewel`` 可调配置提升了 CRUSH 的整体性能，这样，
在 OSD 被标记到集群外（ ``out`` ）时可显著减少映射变化。
因此导致的数据移动少得多。

这个新可调参数是 Jewel 版引进的：

 * ``chooseleaf_stable``: 决定递归选叶是否使用更好的内循环值，
   该值可在 OSD 被标记为 ``out`` 时大大减少映射更改的数量。
   以前的数值是 ``0`` ，而新值 ``1`` 表示用新方法。

对数据迁移的影响：

 * 在已有集群上更改此值会导致海量数据迁移，
   因为几乎每个 PG 映射都可能改变。

哪些客户端版本支持 CRUSH_TUNABLES2
----------------------------------
.. Client versions that support CRUSH_TUNABLES2

 * v0.55 或更高版，包括 bobtail 系列 (v0.56.x)
 * Linux 内核版本大于等于 v3.9 （对文件系统和 RBD 客户端都一样）

哪些客户端版本支持 CRUSH_TUNABLES3
----------------------------------
.. Client versions that support CRUSH_TUNABLES3

 * v0.78 (firefly) 或更高版
 * Linux 内核版本大于等于 v3.15 （对文件系统和 RBD 内核客户端来说）

哪些客户端版本支持 CRUSH_V4
---------------------------
.. Client versions that support CRUSH_V4

 * v0.94 (hammer) 或更高版
 * Linux 内核版本大于等于 v4.1 （对文件系统和 RBD 内核客户端来说）

哪些客户端版本支持 CRUSH_TUNABLES5
----------------------------------
.. Client versions that support CRUSH_TUNABLES5

 * v10.0.2 (jewel) 或更高版
 * Linux 内核版本大于等于 v4.5 （对文件系统和 RBD 内核客户端来说）

可调选项非最优时发出警告
------------------------
.. "Non-optimal tunables" warning

从 v0.74 版起，如果当前的 CRUSH 可调选项没囊括
:ref:` ``default`` 配置<rados_operations_crush_map_default_profile_definition>`\
里的所有最优值， Ceph 就会发出健康警告
（ "HEALTH_WARN crush map has non-optimal tunables" ）。
有两种方法可消除这些警告：

#. 调整现有集群上的可调选项，使其达到最佳状态。
   做这个调整可能会导致一些数据迁移（可能有 10% 之多）。
   这个方法比别的方法更好，但是在数据迁移会对性能带来影响时要特别小心，
   比如在生产集群上。此命令可启用最佳可调选项：

   .. prompt:: bash $

      ceph osd crush tunables optimal

   有几个潜在问题可能会迫使你回退到以前的可调选项值。
   新值可能会对集群产生过大的负载、新值可能导致集群龟速运行、
   或者可能存在客户端兼容性问题。
   使用较老的 CephFS 内核驱动或 RBD 客户端、
   或早于 Bobtail 的 ``librados`` 客户端，会遇到这样的客户端兼任性问题。
   可调选项可以这样回退到以前的取值，用下列命令：

   .. prompt:: bash $

	  ceph osd crush tunables legacy

#. 不对 CRUSH 做任何更改也能消除报警，把下列配置加入
   ``ceph.conf`` 文件的 ``[mon]`` 段下： ::

      mon_warn_on_legacy_crush_tunables = false

   要让这些变更生效，需重启所有监视器，
   或者在监视器上执行下列命令来应用此选项：

   .. prompt:: bash $

      ceph tell mon.\* config set mon_warn_on_legacy_crush_tunables false


调整 CRUSH
----------
.. Tuning CRUSH

在调整 CRUSH 可调选项时，时刻注意以下几点：

 * 调整 CRUSH 可调选项的值会导致一个或更多 PG 在存储节点间迁移。
   如果 Ceph 集群已经存储了大量数据，
   做好迁移大量数据的准备。
 * ``ceph-osd`` 和 ``ceph-mon`` 守护进程们收到新的运行图后，
   它们会立即拒绝不支持新功能的客户端建立新连接。
   然而，之前已经连接的客户端实际上仍能继续使用，
   而这些客户端里不支持新功能的将会出现故障。
 * 如果 CRUSH 可调选项设置成了较新的值（不是旧的 legacy 值），
   然后又改回了旧的值， ``ceph-osd`` 守护进程将不要求支持\
   与这个较新值（不是旧的 legacy 值）对应的 CRUSH 功能。
   然而， OSD 互联进程要能检查和理解旧的运行图。
   因此，\ **如果集群之前用过非默认（ non-legacy ） CRUSH 值，
   就不应该再运行** ``ceph-osd`` **守护进程**\ ，
   —— 即使为了使用默认的旧值已经回滚了最新版运行图。

更改 CRUSH 可调值的最简方法就是应用一个配置集，
也就是 *profile* ，到 Octopus 为止，
Ceph 支持下列配置集：

 * ``legacy``: 采用 argonaut 及更低版本的行为；
 * ``argonaut``: 采用 argonaut 版最初的配置；
 * ``bobtail``: 采用 bobtail 版的配置；
 * ``firefly``: 采用 firefly 版的配置；
 * ``hammer``: hammer 版支持的值
 * ``jewel``: jewel 版支持的值
 * ``optimal``: 当前 Ceph 版本的最佳值；
   .. _rados_operations_crush_map_default_profile_definition:
 * ``default``: 从头安装的新集群的默认值。
   这些值依附于当前的 Ceph 版本，
   是写死的（ hard coded ），
   而且通常是最优值和遗留值的混合体。
   这些值通常对应于前一个 LTS （长期服务）版本或者是\
   最新版本的 ``optimal`` 配置集，
   而大多数用户都拥有最新客户端。

你可以在运行着的集群上选择一个配置：

.. prompt:: bash $

   ceph osd crush tunables {PROFILE}

这个操作可能导致大量数据迁移。
在运行着的集群上更改此配置前，请仔细研究发布说明和文档，\
并试着压制一下恢复、回填参数，以降低一大波回填造成的影响。


.. _CRUSH - 可控、可伸缩、分布式地归置多副本数据: https://ceph.io/assets/pdfs/weil-crush-sc06.pdf


主 OSD 选择的调整
=================
.. Tuning Primary OSD Selection

当 Ceph 客户端读出或写入数据时，它会首先联系\
每个相关 PG 的 acting set 中的第一个 OSD ，默认情况下，
acting set 中的第一个 OSD 是主 OSD （也称为“主导 OSD ”， "lead OSD" ）。
例如，在 acting set ``[2, 3, 4]`` 中， ``osd.2`` 位列第一，因此是主 OSD 。
但有时，很明显，与其他 OSD 相比，某个 OSD 不适合充当主 OSD
（例如，如果该 OSD 的驱动器较慢或控制器速度较慢）。
为了防止出现性能瓶颈（尤其是读出操作），同时最大限度地提高硬件利用率，
可以调整“主亲和性（ primary affinity ）”值或\
定制 CRUSH 规则来影响主 OSD 的选定，
从而内定一批 OSD 充当主导 OSD ，而不会选定其他 OSD 。

要确定调整 Ceph 的主 OSD 选择是否能提高集群性能，
还必须考虑存储池冗余策略。
对于多副本存储池，这种调整可能特别有用，因为默认情况下，
读出操作是由每个 PG 的主 OSD 提供服务的。
不过，对于纠删码存储池，可以通过启用\ **快速读（ fast read ）**\
来提高读取操作的速度（参阅 :ref:`pool-settings` ）。


.. _rados_ops_primary_affinity:

主亲和性
========
.. Primary Affinity

**主亲和性（ Primary affinity ）** 是 OSD 的一个特性，
它决定了一个指定 OSD 在一个指定 acting set 中\
被选为主 OSD （或“主导 OSD ”）的可能性。
主亲和性的值可以是 ``0`` 至 ``1`` （含 ``1`` ）之间的任何实数。

主亲和性的一个常见应用场景是调整主亲和性值，
我们假设一个集群内的驱动器容量大小不一，
例如，较老的机架配备的是 1.9 TB 的 SATA SSD 、而较新的机架是 3.84TB SATA SSD 。
平均下来，后者将会分配到数量上双倍的 PG 、
因而将需要提供双倍的读写操作，
因此它们会比前者忙碌得多。在这样的场景下，
粗略地按照 OSD 容量的反比例分配主亲和性，
这样的分配未必是 100% 最优的，
但它能够很轻松地实现 15% 的整体读吞吐量提升，
因为更平均地利用了 SATA 接口带宽和 CPU 周期。
这个例子不仅仅是一个思想实验，
本意是为了在理论上说明调整主亲和性值的好处；
这百分之十五的改进也是在一个真实的 Ceph 集群上实测过的。

默认情况下，所有 Ceph OSD 的主亲和性都是 ``1`` 。
集群里所有 OSD 的主亲和性取值都是默认值时，
所有 OSD 成为主 OSD 的几率是相等的。

你可以降低 Ceph OSD 的主亲和性取值，
这样 CRUSH 就不太可能把这样的 OSD 选为 PG 的 acting set 里的主 OSD 了。
要更改一个指定 OSD 的主亲和性相关联的权重值，用下列命令：

.. prompt:: bash $

   ceph osd primary-affinity <osd-id> <weight>

你可以把某一个 OSD 的主亲和性设置为 ``[0-1]`` 范围内的实数，\
其中， ``0`` 表示此 OSD **不能** 用作主的，\
而 ``1`` 表示这个 OSD 可以用作主的，且可能性最大。
这个权重介于这两个极端之间时，
这个取值就大致是被 CRUSH 选定为主 OSD 的可能性。

CRUSH 选中一个 lead OSD 的过程不仅仅是个简单的、\
由相对亲和值决定的概率函数。
而是通过理想的主亲和性取值的一阶近似值，
取得一系列可测量的结果。


自定义 CRUSH 规则
-----------------
.. Custom CRUSH Rules

有的集群为了均衡成本和性能，会在同一个多副本存储池里混搭 SSD 和 HDD 。
然后把 HDD OSD 们的主亲和性设置为 ``0`` ，
就可以把操作都引导到各个 acting set 里的 SSD OSD 上。
另一个办法是定义一条 CRUSH 规则，
只选用 SSD OSD 作为主 OSD 、然后其余 OSD 选用 HDD 。
有了这个规则，每个 PG 的 acting set
就只会用 SSD OSD 作为主的，其余 OSD 在 HDD 上。

以下面的 CRUSH 规则为例： ::

    rule mixed_replicated_rule {
            id 11
            type replicated
            step take default class ssd
            step chooseleaf firstn 1 type host
            step emit
            step take default class hdd
            step chooseleaf firstn 0 type host
            step emit
    }

这个规则选了一个 SSD 作为第一个 OSD 。对于 ``N`` 个副本的存储池，
本规则会选用 ``N+1`` 个 OSD 来保证 ``N`` 个副本位于不同的主机上，
因为第一个 SSD OSD 可能和 ``N`` 个 HDD OSD 中的某一个位于\
同一台主机上。

为了避免额外的存储空间需求，需要折衷一下，
把 SSD 和 HDD 分别放到不同的主机上，
然而这就意味着持有 SSD 的主机需要接收所有客户端的请求。为此，
你可以为 SSD OSD 们配备更快的 CPU 、为 HDD OSD 节点配备差一点的，
因为后者只需要应对常规情况，执行恢复操作。
这里的 CRUSH 根 ``ssd_hosts`` 和 ``hdd_hosts`` 有严格要求，
不能包含相同的服务器，如下 CRUSH 规则所示： ::

        rule mixed_replicated_rule_two {
               id 1
               type replicated
               step take ssd_hosts class ssd
               step chooseleaf firstn 1 type host
               step emit
               step take hdd_hosts class hdd
               step chooseleaf firstn -1 type host
               step emit
        }

.. note:: 如果一个主的 SSD OSD 故障了，
   到相关联 PG 的请求临时会由较慢的 HDD OSD 接替，
   直到 PG 的数据复制到接替它的主 SSD OSD 上。


