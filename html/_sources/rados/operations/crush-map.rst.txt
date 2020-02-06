==========
 CRUSH 图
==========

:abbr:`CRUSH (Controlled Replication Under Scalable Hashing)`
算法通过计算数据存储位置来确定如何存储和检索。 CRUSH 授权
Ceph 客户端直接连接 OSD ，而非通过一个中央服务器或经纪人。\
数据存储、检索算法的使用，使 Ceph 避免了单点故障、性能瓶颈、\
和伸缩的物理限制。

CRUSH 需要一张集群的地图，且使用 CRUSH 把数据伪随机地存储、\
检索于整个集群的 OSD 里。 CRUSH 的讨论详情参见 \
`CRUSH - 可控、可伸缩、分布式地归置多副本数据`_ 。

CRUSH 图包含 :abbr:`OSD (Object Storage Devices)` 列表、把\
设备汇聚为物理位置的“桶”列表、和指示 CRUSH 如何复制存储池里\
的数据的规则列表。由于对所安装底层物理组织的表达， CRUSH 能\
模型化、并因此定位到潜在的相关失败设备源头，典型的源头有物\
理距离、共享电源、和共享网络，把这些信息编码到集群运行图里，
CRUSH 归置策略可把对象副本分离到不同的失败域，却仍能保持期\
望的分布。例如，要定位同时失败的可能性，可能希望保证数据复\
制到的设备位于不同机架、不同托盘、不同电源、不同控制器、甚\
至不同物理位置。

你在部署 OSD 时，它们自动被放入 CRUSH 图中的 ``host`` 节点下，\
其名字就是此 OSD 所在主机的主机名。这个，加上默认的 CRUSH \
失效域，确保了副本或纠删码分片会跨主机散布，而且单个主机的失效\
不会影响可用性。然而，对于大一些的集群，管理员就应该仔细考虑\
失效域的选择；例如，副本跨机柜散布，对很多中型到大型集群来说\
很常见。


.. CRUSH Location

CRUSH 位置
==========

用 CRUSH 图层次结构所表示的 OSD 位置被称为“ crush 位置（
crush location ）”，它用键/值对列表来表示。例如，一 OSD 位于\
某特定行、机柜、机架、和主机，且是 CRUSH 图里名为 default 树\
的一部分（绝大多数集群都是这样的情形），那么其 crush 位置可\
表示如下： ::

  root=default row=a rack=a2 chassis=a2a host=a2a1

注：

#. 注意键（关键词）与顺序无关；
#. 键名（ ``=`` 左边）必须是 CRUSH 内的合法 ``type`` ，默认\
   情况下，它包含 root 、 datacenter 、 room 、 row 、 pod 、
   pdu 、 rack 、 chassis 、和 host ，但这些类型可修改 CRUSH
   图任意定义。
#. 并非所有键都需指定，例如，默认情况下 Ceph 会自动把
   ``ceph-osd`` 守护进程的位置设置为
   ``root=default host=HOSTNAME`` （即是 ``hostname -s`` ）。

一个 OSD 的 crush 位置可以用 ``crush location`` 配置选项表示，\
写入 ``ceph.conf`` 文件。 OSD 每次启动时，都会验证它还在
CRUSH 图内的正确位置，如果不是，就自己挪过去。要禁用 CRUSH 图\
的这个自动管理功能，把下面的加进配置文件的 ``[osd]`` 段下： ::

  osd crush update on start = false


.. Custom location hooks

定制位置挂钩
------------

定制的位置钩子可用于在启动时生成更完整的 crush 位置。按先后\
顺序， crush 位置是基于：

#. ceph.conf 内的 ``crush location`` 选项；
#. 默认的 ``root=default host=HOSTNAME`` ，其中主机名是用
   ``hostname -s`` 生成的。

这个用处不大，因为 OSD 自身有着完全相同的行为。然而，可以写个\
脚本来提供额外的位置字段（例如，机柜或数据中心），然后通过\
配置选项启用这个钩子： ::

  crush location hook = /path/to/customized-ceph-crush-location

此挂钩应该接受几个参数（下述）并向标准输出打印一行 CRUSH 位\
置描述： ::

  --cluster CLUSTER --id ID --type TYPE

其中，集群名通常是 "ceph" ， id 是守护进程标识符（ OSD 号或\
守护进程标识符），守护进程类型是 ``osd`` 、 ``mds`` 之类的。

例如，下面这个简单的钩子，基于一个假设的文件 ``/etc/rack`` 、\
它额外指定了一个机架位，如下： ::

  #!/bin/sh
  echo "host=$(hostname -s) rack=$(cat /etc/rack) root=default"


.. CRUSH structure

CRUSH 结构
==========

The CRUSH map consists of, loosely speaking, a hierarchy describing
the physical topology of the cluster, and a set of rules defining
policy about how we place data on those devices.  The hierarchy has
devices (``ceph-osd`` daemons) at the leaves, and internal nodes
corresponding to other physical features or groupings: hosts, racks,
rows, datacenters, and so on.  The rules describe how replicas are
placed in terms of that hierarchy (e.g., 'three replicas in different
racks').

Devices
-------

Devices are individual ``ceph-osd`` daemons that can store data.  You
will normally have one defined here for each OSD daemon in your
cluster.  Devices are identified by an id (a non-negative integer) and
a name, normally ``osd.N`` where ``N`` is the device id.

Devices may also have a *device class* associated with them (e.g.,
``hdd`` or ``ssd``), allowing them to be conveniently targeted by a
crush rule.

Types and Buckets
-----------------

A bucket is the CRUSH term for internal nodes in the hierarchy: hosts,
racks, rows, etc.  The CRUSH map defines a series of *types* that are
used to describe these nodes.  By default, these types include:

- osd (or device)
- host
- chassis
- rack
- row
- pdu
- pod
- room
- datacenter
- region
- root

Most clusters make use of only a handful of these types, and others
can be defined as needed.

The hierarchy is built with devices (normally type ``osd``) at the
leaves, interior nodes with non-device types, and a root node of type
``root``.  For example,

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

Each node (device or bucket) in the hierarchy has a *weight*
associated with it, indicating the relative proportion of the total
data that device or hierarchy subtree should store.  Weights are set
at the leaves, indicating the size of the device, and automatically
sum up the tree from there, such that the weight of the default node
will be the total of all devices contained beneath it.  Normally
weights are in units of terabytes (TB).

You can get a simple view the CRUSH hierarchy for your cluster,
including the weights, with::

  ceph osd crush tree

Rules
-----

Rules define policy about how data is distributed across the devices
in the hierarchy.

CRUSH rules define placement and replication strategies or
distribution policies that allow you to specify exactly how CRUSH
places object replicas. For example, you might create a rule selecting
a pair of targets for 2-way mirroring, another rule for selecting
three targets in two different data centers for 3-way mirroring, and
yet another rule for erasure coding over six storage devices. For a
detailed discussion of CRUSH rules, refer to `CRUSH - 可控、可伸缩、分布式地归置多副本数据`_,
and more
specifically to **Section 3.2**.

In almost all cases, CRUSH rules can be created via the CLI by
specifying the *pool type* they will be used for (replicated or
erasure coded), the *failure domain*, and optionally a *device class*.
In rare cases rules must be written by hand by manually editing the
CRUSH map.

You can see what rules are defined for your cluster with::

  ceph osd crush rule ls

You can view the contents of the rules with::

  ceph osd crush rule dump

  
.. Device classes

设备类别
--------

每个设备都可以选择性地关联一个类别 *class* 。默认情况下， OSD
们在启动时会根据其后端的设备类型自动将其类别设置为 `hdd` 、
`ssd` 或 `nvme` 。

以下命令可以设置一或多个 OSD 的设备类别： ::
  ceph osd crush set-device-class <class> <osd-name> [...]

设备类别配置后就不能再更改为另一个类别，得先删掉其旧类别，用\
命令： ::

  ceph osd crush rm-device-class <osd-name> [...]

如此一来，管理员配置设备类别后，就不会被 OSD 重启或其它脚本\
误改。

指向某个特定设备类别的归置规则可以这样创建： ::

  ceph osd crush rule create-replicated <rule-name> <root> <failure-domain> <class>

然后，存储池就可以改用新规则了： ::

  ceph osd pool set <pool-name> crush_rule <rule-name>

设备类别的实现是在现有类别之上再创建一个“影子” CRUSH
分级结构，此类别中只包含了本类别下的设备。这样，各规则就可以\
通过影子分级结构分发数据了。这个实现方法的好处之一是，它与\
老的Ceph 客户端们完全向后兼容。 CRUSH 分级结构的影子条目可以\
这样查看： ::

  ceph osd crush tree --show-shadow

For older clusters created before Luminous that relied on manually
crafted CRUSH maps to maintain per-device-type hierarchies, there is a
*reclassify* tool available to help transition to device classes
without triggering data movement (see :ref:`crush-reclassify`).


.. Weights sets

权重集
------

*权重集*\ 是计算数据归置时使用的另一种集。常规权重与 CRUSH 图\
内各设备的尺寸相关联，表明哪里\ *应该*\ 存储多少数据。然而，\
由于 CRUSH 是基于伪随机归置过程，总会各种变数干扰这种理想的\
分布，道理和掷色子一样，掷 60 次不会正好是 10 个一点、和 10 个\
六点。权重集可以让集群系统针对特定的集群（层级、存储池、等等）\
做一些数字上的优化，以实现更均衡的分布。

当前支持两种权重集：

 #. A **compat** weight set is a single alternative set of weights for
    each device and node in the cluster.  This is not well-suited for
    correcting for all anomalies (for example, placement groups for
    different pools may be different sizes and have different load
    levels, but will be mostly treated the same by the balancer).
    However, compat weight sets have the huge advantage that they are
    *backward compatible* with previous versions of Ceph, which means
    that even though weight sets were first introduced in Luminous
    v12.2.z, older clients (e.g., firefly) can still connect to the
    cluster when a compat weight set is being used to balance data.
 #. A **per-pool** weight set is more flexible in that it allows
    placement to be optimized for each data pool.  Additionally,
    weights can be adjusted for each position of placement, allowing
    the optimizer to correct for a subtle skew of data toward devices
    with small weights relative to their peers (and effect that is
    usually only apparently in very large clusters but which can cause
    balancing problems).

When weight sets are in use, the weights associated with each node in
the hierarchy is visible as a separate column (labeled either
``(compat)`` or the pool name) from the command::

  ceph osd crush tree

When both *compat* and *per-pool* weight sets are in use, data
placement for a particular pool will use its own per-pool weight set
if present.  If not, it will use the compat weight set if present.  If
neither are present, it will use the normal CRUSH weights.

Although weight sets can be set up and manipulated by hand, it is
recommended that the *balancer* module be enabled to do so
automatically.


.. Modifying the CRUSH map

修改 CRUSH 图
=============

.. _addosd:

增加/移动 OSD
-------------

.. note: OSDs are normally automatically added to the CRUSH map when
         the OSD is created.  This command is rarely needed.

要增加或删除在线集群里 OSD 所对应的 CRUSH 图条目，执行： ::

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


下例把 ``osd.0`` 添加到分级结构里、或者说从前一个位置挪动\
一下。 ::

  ceph osd crush set osd.0 1.0 root=default datacenter=dc1 room=room1 row=foo rack=bar host=foo-bar-1


.. Adjust OSD weight

调整 OSD 的权重
---------------

.. note: Normally OSDs automatically add themselves to the CRUSH map
         with the correct weight when they are created. This command
         is rarely needed.

要调整在线集群中一个 OSD 在 CRUSH 图中的 CRUSH 权重，执行\
命令： ::

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


.. Remove an OSD
.. _removeosd:

删除 OSD
--------

.. note: OSDs are normally removed from the CRUSH as part of the
   ``ceph osd purge`` command.  This command is rarely needed.

要从在线集群里把一 OSD 踢出 CRUSH 图，执行命令： ::

  ceph osd crush remove {name}

其中：


``name``

:描述: OSD 全名。
:类型: String
:是否必需: Yes
:实例: ``osd.0``


.. Add a Bucket

增加桶
------

.. note: Buckets are normally implicitly created when an OSD is added
   that specifies a ``{bucket-type}={bucket-name}`` as part of its
   location and a bucket with that name does not already exist.  This
   command is typically used when manually adjusting the structure of the
   hierarchy after OSDs have been created (for example, to move a
   series of hosts underneath a new rack-level bucket).

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


.. Move a Bucket

移动桶
------

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


.. Remove a Bucket

删除桶
------

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


Creating a compat weight set
----------------------------

.. note: This step is normally done automatically by the ``balancer``
   module when enabled.

To create a *compat* weight set::

  ceph osd crush weight-set create-compat

Weights for the compat weight set can be adjusted with::

  ceph osd crush weight-set reweight-compat {name} {weight}

The compat weight set can be destroyed with::

  ceph osd crush weight-set rm-compat


Creating per-pool weight sets
-----------------------------

To create a weight set for a specific pool,::

  ceph osd crush weight-set create {pool-name} {mode}

.. note:: Per-pool weight sets require that all servers and daemons
          run Luminous v12.2.z or later.

Where:

``pool-name``

:Description: The name of a RADOS pool
:Type: String
:Required: Yes
:Example: ``rbd``

``mode``

:Description: Either ``flat`` or ``positional``.  A *flat* weight set
	      has a single weight for each device or bucket.  A
	      *positional* weight set has a potentially different
	      weight for each position in the resulting placement
	      mapping.  For example, if a pool has a replica count of
	      3, then a positional weight set will have three weights
	      for each device and bucket.
:Type: String
:Required: Yes
:Example: ``flat``

To adjust the weight of an item in a weight set::

  ceph osd crush weight-set reweight {pool-name} {item-name} {weight [...]}

To list existing weight sets,::

  ceph osd crush weight-set ls

To remove a weight set,::

  ceph osd crush weight-set rm {pool-name}


.. Creating a rule for a replicated pool

为多副本存储池创建规则
----------------------

For a replicated pool, the primary decision when creating the CRUSH
rule is what the failure domain is going to be.  For example, if a
failure domain of ``host`` is selected, then CRUSH will ensure that
each replica of the data is stored on a different host.  If ``rack``
is selected, then each replica will be stored in a different rack.
What failure domain you choose primarily depends on the size of your
cluster and how your hierarchy is structured.

Normally, the entire cluster hierarchy is nested beneath a root node
named ``default``.  If you have customized your hierarchy, you may
want to create a rule nested at some other node in the hierarchy.  It
doesn't matter what type is associated with that node (it doesn't have
to be a ``root`` node).

It is also possible to create a rule that restricts data placement to
a specific *class* of device.  By default, Ceph OSDs automatically
classify themselves as either ``hdd`` or ``ssd``, depending on the
underlying type of device being used.  These classes can also be
customized.

To create a replicated rule,::

  ceph osd crush rule create-replicated {name} {root} {failure-domain-type} [{class}]

Where:

``name``

:Description: The name of the rule
:Type: String
:Required: Yes
:Example: ``rbd-rule``

``root``

:Description: The name of the node under which data should be placed.
:Type: String
:Required: Yes
:Example: ``default``

``failure-domain-type``

:Description: The type of CRUSH nodes across which we should separate replicas.
:Type: String
:Required: Yes
:Example: ``rack``

``class``

:Description: The device class data should be placed on.
:Type: String
:Required: No
:Example: ``ssd``


.. Creating a rule for an erasure coded pool

为纠删码存储池创建规则
----------------------

For an erasure-coded pool, the same basic decisions need to be made as
with a replicated pool: what is the failure domain, what node in the
hierarchy will data be placed under (usually ``default``), and will
placement be restricted to a specific device class.  Erasure code
pools are created a bit differently, however, because they need to be
constructed carefully based on the erasure code being used.  For this reason,
you must include this information in the *erasure code profile*.  A CRUSH
rule will then be created from that either explicitly or automatically when
the profile is used to create a pool.

The erasure code profiles can be listed with::

  ceph osd erasure-code-profile ls

An existing profile can be viewed with::

  ceph osd erasure-code-profile get {profile-name}

Normally profiles should never be modified; instead, a new profile
should be created and used when creating a new pool or creating a new
rule for an existing pool.

An erasure code profile consists of a set of key=value pairs.  Most of
these control the behavior of the erasure code that is encoding data
in the pool.  Those that begin with ``crush-``, however, affect the
CRUSH rule that is created.

The erasure code profile properties of interest are:

 * **crush-root**: the name of the CRUSH node to place data under [default: ``default``].
 * **crush-failure-domain**: the CRUSH type to separate erasure-coded shards across [default: ``host``].
 * **crush-device-class**: the device class to place data on [default: none, meaning all devices are used].
 * **k** and **m** (and, for the ``lrc`` plugin, **l**): these determine the number of erasure code shards, affecting the resulting CRUSH rule.

Once a profile is defined, you can create a CRUSH rule with::

  ceph osd crush rule create-erasure {name} {profile-name}

.. note: When creating a new pool, it is not actually necessary to
   explicitly create the rule.  If the erasure code profile alone is
   specified and the rule argument is left off then Ceph will create
   the CRUSH rule automatically.


Deleting rules
--------------

Rules that are not in use by pools can be deleted with::

  ceph osd crush rule rm {rule-name}


.. Tunables
.. _crush-map-tunables:

可调选项
========

时光荏苒，我们已经改进（并将继续改进）了用于计算数据位置的
CRUSH 算法。为了体现（算法的）行为变化，我们引进了一系列可\
调选项，以控制是使用老的、还是改进算法。

要使用较新的可调选项，客户端和服务器都得支持较新版的
CRUSH 。正因为如此，我们建立了一系列以 Ceph 版本命名的配置\
集（ ``profiles`` ），比如， ``firefly`` 可调选项是在
firefly 版首次支持的，而且不支持更老的（如 dumpling ）客户\
端。一旦配置的可调选项生效、改变了旧有的默认行为，
``ceph-mon`` 和 ``ceph-osd`` 就不再允许那些老的、不支持这\
些新 CRUSH 功能的客户端连接集群。


argonaut (遗老)
-----------------

argonaut 和更老版本的 CRUSH 工作方式对大多数集群来说都没问\
题，也没有太多 OSD 被标记为 out 。


bobtail (CRUSH_TUNABLES2)
-------------------------

bobtail 可调选项修正了一些关键的错误行为：

 * 如果分级结构树的叶子桶内只有少量设备，某些 PG 映射的副\
   本数小于期望值。这种情形通常出现在分级结构树中、某些
   host 节点下面只挂少量（1-3个） OSD 时。

 * 在大型集群里，小部分 PG 映射到的 OSD 数目小于期望值，\
   有多层结构（如：机架行、机架、主机、 OSD ）时这种情况\
   更普遍。

 * 当一些 OSD 标记为 out 时，数据倾向于重分布到附近 OSD
   而非整个分级结构树。

新的可调选项有：

 * ``choose_local_tries``: 本地重试次数。以前是 2 ，最优值\
   是 0 。

 * ``choose_local_fallback_tries``: 以前 5 ，最优值是 0 。

 * ``choose_total_tries``: 选择一个条目的最大尝试次数。以前\
   是 19 ，后来的测试表明，对典型的集群来说 50 更合适。最相\
   当大的集群来说，也许有必要用更大的值。

 * ``chooseleaf_descend_once``: 是否重试递归选叶尝试，或只\
   试一次、并允许最初的归置重试。以前默认为 0 ，最优为 1 。

对数据迁移的影响：

 * 可调选项从 argonaut 改为 bobtail 会引起一定量的数据\
   迁移。在已经有数据的集群上需谨慎点。


firefly (CRUSH_TUNABLES3)
-------------------------

firefly 可调选项对 ``chooseleaf`` 这条 CRUSH 规则的行为有所\
修正，在太多 OSD 被标记为 out 状态时，用非常少的结果生成 PG
映射关系会有问题。

新的可调选项是：

 * ``chooseleaf_vary_r``: 根据父节点已做过多少尝试，递归选叶\
   是否应该以非零值 r 开始。原先的默认值是 0 ，但是用此值的话
   CRUSH 有时候会找不到映射关系；较优的值（计算代价和正确性\
   合理）是 1 。

对数据迁移的影响：

 * 对于已经在运行、里面已经有了大量数据的集群，从 0 改为 1
   会导致大量的数据迁移； 4 或 5 时 CRUSH 也能正确找到映射，\
   而且数据迁移少的多。


straw_calc_version 可调选项（也是在 Firefly 引进的）
----------------------------------------------------

以前，给 ``straw`` 桶计算、并存储在 CRUSH 图里的内部权重有\
些问题。特别是，如果有些条目 CRUSH 权重为 0 或者配置了多个\
权重、重复配置了权重时， CRUSH 就不能正确地分布数据（也就\
是与权重配比失衡）。

这个新可调参数是：

 * ``straw_calc_version``: 值为 0 时保留老的、有问题的内部\
   权重算法；值为 1 时修正此行为。

对数据迁移的影响：

 * straw_calc_version 改为 1 之后，\ *假如*\ 此集群触碰了某\
   个雷区（ problematic condition ），调整 straw 桶（新增、\
   删除、更改某一条目的权重、或用 reweight-all 命令更改所有\
   权重）时就有可能引起少量或少部分数据迁移。

这个可调选项有些特殊，因为它对客户端的内核版本没任何要求。


hammer (CRUSH_V4)
-----------------

仅仅是更改配置的话， hammer 版的可调配置不会影响已有 CRUSH
图的映射关系。然而：

 * 增加了新型桶（ ``straw2`` ）的支持。这种新型的 ``straw2``
   桶解决了原来 ``straw`` 桶的几个局限性。具体来说，老的
   ``straw`` 桶在有权重变化时会改变一些映射关系，新的
   ``straw2`` 桶实现了预定目标，只有在这些桶的权重发生变化\
   时才更改相应的映射关系。

 * ``straw2`` 是新建桶的默认类型。

对数据迁移的影响：

 * 把桶类型从 ``straw`` 改为 ``straw2`` 会导致少量数据迁移，\
   这取决于桶内各条目的权重有多大落差。它们的权重都相同时就\
   不会有数据迁移，各条目的权重落差巨大时就会有更多迁移。


jewel (CRUSH_TUNABLES5)
-----------------------

jewel 版的可调配置能够提升 CRUSH 的整体行为，这样，在 OSD
被标记到集群外时可显著减少重映射。

这个新可调参数是：

 * ``chooseleaf_stable``: 当某一 OSD 被标记为 out 时，递归\
   选叶尝试是否把更优值代入内层递归、对于减少重映射具有重大\
   影响。以前的数值是 0 ，而新值 1 表示用新方法。

对数据迁移的影响：

 * 在已有集群上更改此值会导致海量数据迁移，因为几乎每个 PG
   映射都可能改变。




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


.. _Warning when tunables are non-optimal:

可调选项非最优时发出警告
------------------------

从 v0.74 起，如果当前的 CRUSH 可调选项没囊括 ``default`` 配置\
集里的所有最优值（ ``default`` 配置集的含义见下文）， Ceph 就\
会发出健康警告，有两种方法可消除这些警告：

#. 调整现有集群上的可调选项。注意，这可能会导致一些数据迁移（\
   可能有 10% 之多）。这是推荐的办法，但是在生产集群上要注意此\
   调整对性能带来的影响。此命令可启用较优可调选项： ::

	ceph osd crush tunables optimal

   如果切换得不太顺利（如负载太高）且切换才不久，或者有客户端\
   兼容问题（较老的 cephfs 内核驱动或 rbd 客户端、或早于
   bobtail 的 librados 客户端），你可以这样切回： ::

	ceph osd crush tunables legacy

#. 不对 CRUSH 做任何更改也能消除报警，把下列配置加入
   ``ceph.conf`` 的 ``[mon]`` 段下： ::

	mon warn on legacy crush tunables = false

   为使变更生效需重启所有监视器，或者执行下列命令： ::

        ceph tell mon.\* config set mon_warn_on_legacy_crush_tunables false


.. A few important points

一些要点
--------

 * 调整这些值将使一些 PG 在存储节点间移位，如果 Ceph 集群已经\
   存储了大量数据，做好移动一部分数据的准备。
 * 一旦更新运行图， ``ceph-osd`` 和 ``ceph-mon`` 就会开始向新\
   建连接要求功能位，然而，之前已经连接的客户端如果不支持新功\
   能将行为失常。
 * 如果 CRUSH 可调值更改过、然后又改回了默认值， ``ceph-osd``
   守护进程将不要求支持此功能，然而， OSD 连接建立进程要能检查\
   和理解旧地图。因此，集群如果用过非默认 CRUSH 值就不应该再运\
   行版本小于 0.48.1 的 ``ceph-osd`` ，即使最新版地图已经回滚\
   到了遗留默认值。


.. _Tuning CRUSH:

调整 CRUSH
----------

更改 crush 可调值的最简方法就是改到一个已知配置，它们有：

 * ``legacy``: 采用 argonaut 及更低版本的行为；
 * ``argonaut``: 采用 argonaut 版最初的配置；
 * ``bobtail``: 采用 bobtail 版的配置；
 * ``firefly``: 采用 firefly 版的配置；
 * ``hammer``: hammer 版支持的值
 * ``jewel``: jewel 版支持的值
 * ``optimal``: 当前 Ceph 版本的最佳（即最优的）值；
 * ``default``: 从头安装的新集群的默认值。这些值依附于当前的
   Ceph 版本，是写死的（ hard coded ），而且通常是最优值和遗留\
   值的混合体。这些值通常对应于前一个 LTS 版本的 ``optimal``
   配置集，或者是最近的、我们预计大多数用户都不会升级过来的版\
   本。

你可以在运行着的集群上选择一个配置： ::

	ceph osd crush tunables {PROFILE}

要注意，这可能产生一些数据迁移。


.. Primary Affinity

主亲和性
========

一 Ceph 客户端读写数据时，总是连接 acting set 里的主 OSD （如
``[2, 3, 4]`` 中， ``osd.2`` 是主的）。有时候某个 OSD 与其它\
的相比并不适合做主 OSD （比如其硬盘慢、或控制器慢），最大化\
硬件利用率时为防止性能瓶颈（特别是读操作），你可以调整 OSD 的\
主亲和性，这样 CRUSH 就尽量不把它用作 acting set 里的主 OSD
了。 ::

	ceph osd primary-affinity <osd-id> <weight>

主亲和性默认为 ``1`` （\ **就是说**\ 此 OSD 可作为主 OSD ）。\
此值合法范围为 ``0-1`` ，其中 ``0`` 意为此 OSD 不能用作主的，
``1`` 意为 OSD 可用作主的；此权重小于 ``1`` 时， CRUSH 选择\
主 OSD 时选中它的可能性低。


.. _CRUSH - 可控、可伸缩、分布式地归置多副本数据: https://ceph.com/wp-content/uploads/2016/08/weil-crush-sc06.pdf
