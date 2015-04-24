============
 纠删码配置
============

纠删码由\ **配置**\ 定义，在创建纠删码存储池及其相关的 CRUSH 规则集时用到。

创建 Ceph 集群时初始化的、名为 **default** 的纠删码配置可提供与两副本相同的冗\
余水平，却能节省 25% 的磁盘空间。在此配置中 **k=2** 和 **m=1** ，其含义为数据\
分布于 3 个 OSD （ k+m==3 ）且允许一个失效。

为了在不增加原始存储空间需求的前提下提升冗余性，你可以新建配置。例如，一个 \
**k=10** 且 **m=4** 的配置可容忍 4 个 OSD 失效，它会把一对象分布到 14 个\
（ k+m=14 ） OSD 上。此对象先被分割为 **10** 块（若对象为 10MB ，那每块就是 \
1MB ）、并计算出 **4** 个用于恢复的编码块（各编码块尺寸等于数据块，即 1MB ）；\
这样，原始空间仅多占用 10% 就可容忍 4 个 OSD 同时失效、且不丢失数据。

.. _可用插件列表:

.. toctree::
	:maxdepth: 1

	erasure-code-jerasure
	erasure-code-isa
	erasure-code-lrc
	erasure-code-shec


osd erasure-code-profile set
============================

要新建纠删码配置： ::

	ceph osd erasure-code-profile set {name} \
             [{directory=directory}] \
             [{plugin=plugin}] \
             [{key=value} ...] \
             [--force]

其中：


``{directory=directory}``

:描述: 设置纠删码插件的路径，需是\ **目录**\ 。
:类型: String
:是否必需: No.
:默认值: /usr/lib/ceph/erasure-code


``{plugin=plugin}``

:描述: 指定纠删码\ **插件**\ 来计算编码块、及恢复丢失块。详见\ `可用插件列表`_\ 。
:类型: String
:是否必需: No.
:默认值: jerasure


``{key=value}``

:描述: 纠删码插件所定义的键/值对含义。
:类型: String
:是否必需: No.


``--force``

:描述: 覆盖同名配置。
:类型: String
:是否必需: No.


osd erasure-code-profile rm
============================

要删除纠删码配置： ::

	ceph osd erasure-code-profile rm {name}

若此配置还被某个存储池使用着，则删除会失败。


osd erasure-code-profile get
============================

要查看一纠删码配置： ::

	ceph osd erasure-code-profile get {name}


osd erasure-code-profile ls
===========================

列出所有纠删码配置： ::

	ceph osd erasure-code-profile ls
