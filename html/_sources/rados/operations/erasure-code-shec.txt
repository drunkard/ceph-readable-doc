=================
 SHEC 纠删码插件
=================

*shec* 插件封装了 `multiple SHEC
<https://wiki.ceph.com/Planning/Blueprints/Hammer/Shingled_Erasure_Code_(SHEC)>`_
库。与 Reed Solomon 编码相比，它能使 Ceph 更高效地恢复数据。


创建 SHEC 配置
==============

要新建 *shec* 纠删码配置： ::

	ceph osd erasure-code-profile set {name} \
		plugin=shec \
		[k={data-chunks}] \
		[m={coding-chunks}] \
		[c={durability-estimator}] \
		[ruleset-root={root}] \
		[ruleset-failure-domain={bucket-type}] \
		[directory={directory}] \
		[--force]

其中：


``k={data-chunks}``

:描述: 各对象都被分割为 **data-chunks** 块，分别存储到不同的 OSD 上。
:类型: Integer
:是否必需: No.
:默认值: 4


``m={coding-chunks}``

:描述: 为各对象计算出 **coding-chunks** 个编码块，并存储到不同 OSD 上。 \
	**coding-chunks** 数值不一定等于宕机不丢数据所需的 OSD 数量。
:类型: Integer
:是否必需: No.
:默认值: 3


``c={durability-estimator}``

:描述: 奇偶校验块数量，它们在各自的计算范围内包含了各数据块。此数值被用作 \
	 **durability estimator** （持久性估值）。例如，假设 c=2 ，就是说\
	 不丟数据的情况下可损失 2 个 OSD 。
:类型: Integer
:是否必需: No.
:默认值: 2


``ruleset-root={root}``

:描述: 规则集第一步所用的 crush 桶名字。例如 **step take default** 。
:类型: String
:是否必需: No.
:默认值: default


``ruleset-failure-domain={bucket-type}``

:描述: 要确保同一故障域内不能把任意两个块放进一个桶内。例如，如果故障域是 \
	 **host** ，那就不能把任意两个块放到同一主机。它被用于创建规则集的\
	 一步，像 **step chooseleaf host** 。
:类型: String
:是否必需: No.
:默认值: host


``directory={directory}``

:描述: 设置纠删码插件所在目录（ **directory** ）。
:类型: String
:是否必需: No.
:默认值: /usr/lib/ceph/erasure-code


``--force``

:描述: 覆盖已存在的同名配置。
:类型: String
:是否必需: No.


SHEC 布局概述
=============


空间效率
--------

空间效率是数据块在所有对象内占的比率，并表示为 k/(k+m) 。为提升空间效率，\
你可以增加 k 值或降低 m 值。

::

	space efficiency of SHEC(4,3,2) = 4/(4+3) = 0.57
	SHEC(5,3,2) or SHEC(4,2,2) improves SHEC(4,3,2)'s space efficiency


持久性
------

SHEC 的第三个参数（ =c ）是一个持久性估值，它大致等于在不丢数据的前提下允\
许损失的 OSD 数量。

``durability estimator of SHEC(4,3,2) = 2``


恢复效率
--------

描述恢复效率的计算超出了本文档范畴，但是，提高 m 而不提高 c 确实能提高恢复\
效率。（然而，在这种情况下必须注意空间效率的牺牲）

``SHEC(4,2,2) -> SHEC(4,3,2) : 实现了恢复效率的提升``


纠删码配置实例
==============

::

	$ ceph osd erasure-code-profile set SHECprofile \
		plugin=shec \
		k=8 m=4 c=3 \
		ruleset-failure-domain=host
	$ ceph osd pool create shecpool 256 256 erasure SHECprofile
