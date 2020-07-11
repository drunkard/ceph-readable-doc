.. Manually editing a CRUSH Map

手动编辑一个 CRUSH 图
=====================

.. note:: Manually editing the CRUSH map is considered an advanced
	  administrator operation.  All CRUSH changes that are
	  necessary for the overwhelming majority of installations are
	  possible via the standard ceph CLI and do not require manual
	  CRUSH map edits.  If you have identified a use case where
	  manual edits *are* necessary, consider contacting the Ceph
	  developers so that future versions of Ceph can make this
	  unnecessary.

要编辑现有的 CRUSH 图：

#. `获取 CRUSH 图`_\ ；
#. `反编译`_ CRUSH 图；
#. 至少编辑一个\ `设备`_\ 、\ `桶`_\ 、\ `规则`_\ ；
#. `重编译`_ CRUSH 图；
#. `注入 CRUSH 图`_\ 。

关于为指定存储池设置 CRUSH 图规则的细节，见\ `调整存储池`_\ 。

.. _获取 CRUSH 图: #getcrushmap
.. _反编译: #decompilecrushmap
.. _设备: #crushmapdevices
.. _桶: #crushmapbuckets
.. _规则: #crushmaprules
.. _重编译: #compilecrushmap
.. _注入 CRUSH 图: #setcrushmap
.. _调整存储池: ../pools#setpoolvalues


.. Get a CRUSH Map
.. _getcrushmap:

获取 CRUSH 图
-------------

要获取集群的 CRUSH 图，执行命令： ::

	ceph osd getcrushmap -o {compiled-crushmap-filename}

Ceph 将把 CRUSH 输出（ -o ）到你指定的文件，由于 CRUSH 图是\
已编译的，所以编辑前必须先反编译。


.. Decompile a CRUSH Map
.. _decompilecrushmap:

反编译 CRUSH 图
---------------

要反编译 CRUSH 图，执行命令： ::

	crushtool -d {compiled-crushmap-filename} -o {decompiled-crushmap-filename}


.. Recompile a CRUSH Map
.. _compilecrushmap:

重编译 CRUSH 图
---------------

To compile a CRUSH map, execute the following::

	crushtool -c {decompiled-crushmap-filename} -o {compiled-crushmap-filename}


.. Set the CRUSH Map
.. _setcrushmap:

设置 CRUSH 图
-------------

To set the CRUSH map for your cluster, execute the following::

	ceph osd setcrushmap -i {compiled-crushmap-filename}

Ceph will load (-i) a compiled CRUSH map from the filename you specified.


.. Sections

分段
----

CRUSH 图有 6 个主要段落。

#. **tunables:** The preamble at the top of the map described any *tunables*
   for CRUSH behavior that vary from the historical/legacy CRUSH behavior. These
   correct for old bugs, optimizations, or other changes in behavior that have
   been made over the years to improve CRUSH's behavior.

#. **devices:** Devices are individual ``ceph-osd`` daemons that can
   store data.

#. **types**: 定义了 CRUSH 分级结构里要用的桶类型（
   ``types`` ），桶由逐级汇聚的存储位置（如行、机柜、机箱、\
   主机等等）及其权重组成。

#. **buckets:** Once you define bucket types, you must define each node
   in the hierarchy, its type, and which devices or other nodes it
   contains.

#. **rules:** Rules define policy about how data is distributed across
   devices in the hierarchy.

#. **choose_args:** Choose_args are alternative weights associated with
   the hierarchy that have been adjusted to optimize data placement.  A single
   choose_args map can be used for the entire cluster, or one can be
   created for each individual pool.


.. CRUSH Map Devices
.. _crushmapdevices:

CRUSH 图之设备
--------------

Devices are individual ``ceph-osd`` daemons that can store data.  You
will normally have one defined here for each OSD daemon in your
cluster.  Devices are identified by an id (a non-negative integer) and
a name, normally ``osd.N`` where ``N`` is the device id.

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

绝大多数情况下，各设备都映射到单个 ``ceph-osd`` 守护进程。它\
通常是单个存储设备、一对设备（例如，一个用于数据、一个用于日志\
或元数据）、或者某些情况下一个小的 RAID 设备。


.. CRUSH Map Bucket Types

CRUSH 图之桶类型
----------------

CRUSH 图里的第二个列表定义了 bucket （桶）类型，桶简化了节点和\
叶子层次。节点（或非叶子）桶在分级结构里一般表示物理位置，节点\
汇聚了其它节点或叶子，叶桶表示 ``ceph-osd`` 守护进程及其对应的\
存储媒体。

.. tip:: CRUSH 中用到的 bucket 意思是分级结构中的一个节点，\
   也就是一个位置或一部分硬件。但是在 RADOS 网关接口的术语中，\
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
	type 9 region
	type 10 root



.. CRUSH Map Bucket Hierarchy
.. _crushmapbuckets:

CRUSH 图之桶层次
----------------

CRUSH 算法根据各设备的权重、大致统一的概率把数据对象分布到存储\
设备中。 CRUSH 根据你定义的集群运行图分布对象及其副本， CRUSH \
图表达了可用存储设备以及包含它们的逻辑单元。

要把归置组映射到跨故障域的 OSD ，一个 CRUSH 图需定义一系列分级\
桶类型（即现有 CRUSH 图的 ``#type`` 下）。创建桶分级结构的目的\
是按故障域隔离叶节点，像主机、机箱、机柜、电力分配单元、机群、\
行、房间、和数据中心。除了表示叶节点的 OSD ，其它分级结构都是\
任意的，你可以按需定义。

我们建议 CRUSH 图内的命名符合贵公司的硬件命名规则，并且采用反\
映物理硬件的例程名。良好的命名可简化集群管理和故障排除，当 OSD \
和/或其它硬件出问题时，管理员可轻易找到对应物理硬件。

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

.. note:: 编号较高的 ``rack`` 桶类型汇聚编号较低的 ``host`` 桶\
   类型。

位于 CRUSH 图起始部分、 ``#devices`` 列表内是表示叶节点的存\
储设备，没必要声明为桶例程。位于分级结构第二低层的桶一般用\
于汇聚设备（即它通常是包含存储媒体的计算机，你可以用自己喜\
欢的名字描述，如节点、计算机、服务器、主机、机器等等）。在\
高密度环境下，经常出现一机框内安装多个主机/节点的情况，因此\
还要考虑机框故障——比如，某一节点故障后需要拉出机框维修，这\
会影响多个主机/节点和其内的 OSD 。

声明一个桶例程时，你必须指定其类型、惟一名称（字符串）、惟\
一负整数 ID （可选）、指定和各条目总容量/能力相关的权重、指\
定桶算法（通常是 ``straw`` ）、和哈希（通常为 ``0`` ，表示\
散列算法 ``rjenkins1`` ）。一个桶可以包含一到多条，这些条目\
可以由节点桶或叶子组成，它们可以有个权重用来反映条目的相对\
权重。

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
		alg straw
		hash 0
		item osd.0 weight 1.00
		item osd.1 weight 1.00
	}

	host node2 {
		id -2
		alg straw
		hash 0
		item osd.2 weight 1.00
		item osd.3 weight 1.00
	}

	rack rack1 {
		id -3
		alg straw
		hash 0
		item node1 weight 2.00
		item node2 weight 2.00
	}

.. note:: 在前述示例中，机柜桶不包含任何 OSD ，它只包含低一\
   级的主机桶、以及其内条目的权重之和。

.. topic:: 桶类型

   Ceph 支持四种桶，每种都是性能和组织简易间的折衷。如果你\
   不确定用哪种桶，我们建议 ``straw`` ，关于桶类型的详细讨\
   论见 `CRUSH - 可控、可伸缩、分布式地归置多副本数据`_\ ，\
   特别是 **Section 3.4** 。支持的桶类型有：

	#. **Uniform**: 这种桶用\ **完全**\ 相同的权重汇聚\
           设备。例如，公司采购或淘汰硬件时，一般都有相同的\
           物理配置（如批发）。当存储设备权重都相同时，你可以\
           用 ``uniform`` 桶类型，它允许 CRUSH 按常数把副本\
           映射到 uniform 桶。权重不统一时，你应该采用其它\
           算法。

	#. **List**: 这种桶把它们的内容汇聚为链表。它基于 \
	   :abbr:`RUSH (Replication Under Scalable Hashing)` \
	   :sub:`P` 算法，一个列表就是一个自然、直观的\ \
	   **扩展集群**\ ：对象会按一定概率被重定位到最新的\
           设备、或者像从前一样仍保留在较老的设备上。结果是\
           优化了新条目加入桶时的数据迁移。然而，如果从链表\
           的中间或末尾删除了一些条目，将会导致大量没必要的\
           挪动。所以这种桶适合\ **永不或极少缩减**\ 的场景。

	#. **Tree**: 它用一种二进制搜索树，在桶包含大量条目时\
           比 list 桶更高效。它基于 \
	   :abbr:`RUSH (Replication Under Scalable Hashing)` \
	   :sub:`R` 算法， tree 桶把归置时间减少到了 \
	   O(log :sub:`n`) ，这使得它们更适合管理更大规模的\
           设备或嵌套桶。

	#. **Straw**: list 和 tree 桶用分而治之策略，给特定\
           条目一定优先级（如位于链表开头的条目）、或避开对\
           整个子树上所有条目的考虑。这样提升了副本归置进程\
           的性能，但是也导致了重新组织时的次优结果，如增加、\
           拆除、或重设某条目的权重。 straw 桶类型允许所有\
           条目模拟拉稻草的过程公平地相互“竞争”副本归置。

        #. **Straw2**: Straw2 桶是对 straw 的改进，在邻居权重\
           改变时可正确地避免条目间的数据迁移。

           例如，条目 A 的权重再次增大或完全删除，都仅会有数据\
           迁移进或移出条目 A 。

.. topic:: Hash

   各个桶都用了一种散列算法，当前 Ceph 仅支持 ``rjenkins1`` ，\
   输入 ``0`` 表示散列算法设置为 ``rjenkins1`` 。


.. _weightingbucketitems:

.. topic:: 调整桶的权重

   Ceph 用双整形表示桶权重。权重和设备容量不同，我们建议用
   ``1.00`` 作为 1TB 存储设备的相对权重，这样 ``0.5`` 的权\
   重大概代表 500GB 、 ``3.00`` 大概代表 3TB 。较高级桶的\
   权重是所有枝叶桶的权重之和。

   一个桶的权重是一维的，你也可以计算条目权重来反映存储设\
   备性能。例如，如果你有很多 1TB 的硬盘，其中一些数据传输\
   速率相对低、其他的数据传输率相对高，即使它们容量相同，\
   也应该设置不同的权重（如给吞吐量较低的硬盘设置权重 0.8 ，\
   较高的设置 1.20 ）。


.. CRUSH Map Rules
.. _crushmaprules:

CRUSH 图之规则
--------------

CRUSH 图支持“ CRUSH 规则”概念，用以决定一个存储池里的数据如何\
归置。默认的 CRUSH 图有一条规则适用于各存储池。对大型集群\
来说，你可能创建很多存储池，且每个存储池都有它自己的、非默认的
CRUSH 规则。

.. note:: 大多数情况下，你都不需要修改默认规则。默认情况下，\
   新创建的存储池其规则会设置为 ``0`` 。

CRUSH 规则定义了归置和复制策略、或分布策略，用它可以规定
CRUSH 如何放置对象副本。例如，你也许想创建一条规则用以选择\
一对目的地做双路镜像；另一条规则用以选择位于两个数据中心的\
三个目的地做三路镜像；又一条规则用 6 个设备做纠删编码。关于
CRUSH 规则的详细研究见
`CRUSH - 可控、可伸缩、分布式地归置多副本数据`_\ ，主要是 \
**Section 3.2** 。

规则格式如下： ::

	rule <rulename> {

		id [a unique whole numeric ID]
		type [ replicated | erasure ]
		min_size <min-size>
		max_size <max-size>
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

:描述: 为一个驱动器（多副本的）或 RAID 确定一条规则。
:目的: 规则掩码的一个组件。
:类型: String
:是否必需: Yes
:默认值: ``replicated``
:有效值: 当前仅支持 ``replicated`` 和 ``erasure``


``min_size``

:描述: 如果一个归置组副本数小于此数， CRUSH 将\ **不**\ 应用此\
       规则。

:类型: Integer
:目的: 规则掩码的一个组件。
:是否必需: Yes
:默认值: ``1``


``max_size``

:描述: 如果一个归置组副本数大于此数， CRUSH 将\ **不**\ 应用此\
       规则。

:类型: Integer
:目的: 规则掩码的一个组件。
:是否必需: Yes
:默认值: 10


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

.. important:: 一条 CRUSH 规则可以分配给多个存储池使用，但是\
   一个存储池不能同时配置多条 CRUSH 规则。


``firstn`` versus ``indep``

:描述: Controls the replacement strategy CRUSH uses when items (OSDs)
	      are marked down in the CRUSH map. If this rule is to be used with
	      replicated pools it should be ``firstn`` and if it's for
	      erasure-coded pools it should be ``indep``.

	      The reason has to do with how they behave when a
	      previously-selected device fails. Let's say you have a PG stored
	      on OSDs 1, 2, 3, 4, 5. Then 3 goes down.
	      
	      With the "firstn" mode, CRUSH simply adjusts its calculation to
	      select 1 and 2, then selects 3 but discovers it's down, so it
	      retries and selects 4 and 5, and then goes on to select a new
	      OSD 6. So the final CRUSH mapping change is
	      1, 2, 3, 4, 5 -> 1, 2, 4, 5, 6.

	      But if you're storing an EC pool, that means you just changed the
	      data mapped to OSDs 4, 5, and 6! So the "indep" mode attempts to
	      not do that. You can instead expect it, when it selects the failed
	      OSD 3, to try again and pick out 6, for a final transformation of:
	      1, 2, 3, 4, 5 -> 1, 2, 6, 4, 5


.. Migrating from a legacy SSD rule to device classes
.. _crush-reclassify:

从老式的 SSD 规则迁移到设备类
-----------------------------

It used to be necessary to manually edit your CRUSH map and maintain a
parallel hierarchy for each specialized device type (e.g., SSD) in order to
write rules that apply to those devices.  Since the Luminous release,
the *device class* feature has enabled this transparently.

However, migrating from an existing, manually customized per-device map to
the new device class rules in the trivial way will cause all data in the
system to be reshuffled.

The ``crushtool`` has a few commands that can transform a legacy rule
and hierarchy so that you can start using the new class-based rules.
There are three types of transformations possible:

#. ``--reclassify-root <root-name> <device-class>``

   This will take everything in the hierarchy beneath root-name and
   adjust any rules that reference that root via a ``take
   <root-name>`` to instead ``take <root-name> class <device-class>``.
   It renumbers the buckets in such a way that the old IDs are instead
   used for the specified class's "shadow tree" so that no data
   movement takes place.

   For example, imagine you have an existing rule like::

     rule replicated_ruleset {
        id 0
        type replicated
        min_size 1
        max_size 10
        step take default
        step chooseleaf firstn 0 type rack
        step emit
     }

   If you reclassify the root `default` as class `hdd`, the rule will
   become::

     rule replicated_ruleset {
        id 0
        type replicated
        min_size 1
        max_size 10
        step take default class hdd
        step chooseleaf firstn 0 type rack
        step emit
     }

#. ``--set-subtree-class <bucket-name> <device-class>``

   This will mark every device in the subtree rooted at *bucket-name*
   with the specified device class.

   This is normally used in conjunction with the ``--reclassify-root``
   option to ensure that all devices in that root are labeled with the
   correct class.  In some situations, however, some of those devices
   (correctly) have a different class and we do not want to relabel
   them.  In such cases, one can exclude the ``--set-subtree-class``
   option.  This means that the remapping process will not be perfect,
   since the previous rule distributed across devices of multiple
   classes but the adjusted rules will only map to devices of the
   specified *device-class*, but that often is an accepted level of
   data movement when the nubmer of outlier devices is small.

#. ``--reclassify-bucket <match-pattern> <device-class> <default-parent>``

   This will allow you to merge a parallel type-specific hiearchy with the normal hierarchy.  For example, many users have maps like::

     host node1 {
        id -2           # do not change unnecessarily
        # weight 109.152
        alg straw
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
        alg straw
        hash 0  # rjenkins1
        item osd.80 weight 2.000
	...
     }

     root default {
        id -1           # do not change unnecessarily
        alg straw
        hash 0  # rjenkins1
        item node1 weight 110.967
        ...
     }

     root ssd {
        id -18          # do not change unnecessarily
        # weight 16.000
        alg straw
        hash 0  # rjenkins1
        item node1-ssd weight 2.000
	...
     }

   This function will reclassify each bucket that matches a
   pattern.  The pattern can look like ``%suffix`` or ``prefix%``.
   For example, in the above example, we would use the pattern
   ``%-ssd``.  For each matched bucket, the remaining portion of the
   name (that matches the ``%`` wildcard) specifies the *base bucket*.
   All devices in the matched bucket are labeled with the specified
   device class and then moved to the base bucket.  If the base bucket
   does not exist (e.g., ``node12-ssd`` exists but ``node12`` does
   not), then it is created and linked underneath the specified
   *default parent* bucket.  In each case, we are careful to preserve
   the old bucket IDs for the new shadow buckets to prevent data
   movement.  Any rules with ``take`` steps referencing the old
   buckets are adjusted.

#. ``--reclassify-bucket <bucket-name> <device-class> <base-bucket>``

   The same command can also be used without a wildcard to map a
   single bucket.  For example, in the previous example, we want the
   ``ssd`` bucket to be mapped to the ``default`` bucket.

The final command to convert the map comprised of the above fragments would be something like::

  $ ceph osd getcrushmap -o original
  $ crushtool -i original --reclassify \
      --set-subtree-class default hdd \
      --reclassify-root default hdd \
      --reclassify-bucket %-ssd ssd default \
      --reclassify-bucket ssd ssd default \
      -o adjusted

In order to ensure that the conversion is correct, there is a ``--compare`` command that will test a large sample of inputs to the CRUSH map and ensure that the same result comes back out.  These inputs are controlled by the same options that apply to the ``--test`` command.  For the above example,::

  $ crushtool -i original --compare adjusted
  rule 0 had 0/10240 mismatched mappings (0)
  rule 1 had 0/10240 mismatched mappings (0)
  maps appear equivalent

If there were difference, you'd see what ratio of inputs are remapped
in the parentheses.

If you are satisfied with the adjusted map, you can apply it to the cluster with something like::

  ceph osd setcrushmap -i adjusted


.. Tuning CRUSH, the hard way

调整 CRUSH ，强硬方法
---------------------

如果你能保证所有客户端都运行最新代码，你可以这样调整可调值：从\
集群抽取 CRUSH 图、修改值、重注入。

 * 提抽取最新 CRUSH 图： ::

        ceph osd getcrushmap -o /tmp/crush

 * 调整可调参数。这些值在我们测试过的大、小型集群上都有最佳表现。\
   在极端情况下，你需要给 ``crushtool`` 额外指定 \
   ``--enable-unsafe-tunables`` 参数才行： ::

        crushtool -i /tmp/crush --set-choose-local-tries 0 --set-choose-local-fallback-tries 0 --set-choose-total-tries 50 -o /tmp/crush.new

 * 重注入修改的图。 ::

        ceph osd setcrushmap -i /tmp/crush.new


.. Legacy values

遗留值
------

供参考，CRUSH 可调参数的遗留值可以用下面命令设置： ::

   crushtool -i /tmp/crush --set-choose-local-tries 2 --set-choose-local-fallback-tries 5 --set-choose-total-tries 19 --set-chooseleaf-descend-once 0 --set-chooseleaf-vary-r 0 -o /tmp/crush.legacy

再次申明， ``--enable-unsafe-tunables`` 是必需的，而且前面也\
提到了，回退到遗留值后慎用旧版 ``ceph-osd`` 进程，因为此功能位\
不是完全强制的。

.. _CRUSH - 可控、可伸缩、分布式地归置多副本数据: https://ceph.com/wp-content/uploads/2016/08/weil-crush-sc06.pdf
