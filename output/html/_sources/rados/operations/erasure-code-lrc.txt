======================
 局部自修复纠删码插件
======================

用 *jerasure* 插件时，纠删码编码的对象存储在多个 OSD 上，丢失一个 OSD 的恢复\
过程需读取所有其他的 OSD 。比如 *jerasure* 的配置为 *k=8* 且 *m=4* ，丢失一\
个 OSD 后需读取其他 11 个 OSD 才能恢复。

*lrc* 纠删码插件创建的是局部校验块，这样只需较少的 OSD 即可恢复。比如 *lrc* \
的配置为 *k=8* 、 *m=4* 且 *l=4* ，它将为每四个 OSD 创建额外的校验块，当一个 \
OSD 丢失时，它只需四个 OSD 即可恢复，而不需要十一个。


纠删码配置实例
==============

降低主机间的恢复带宽
--------------------

虽然当所有主机都接入同一交换机时，这不会是诱人的用法，但是带宽利用率确实降低\
了。 ::

        $ ceph osd erasure-code-profile set LRCprofile \
             plugin=lrc \
             k=4 m=2 l=3 \
             ruleset-failure-domain=host
        $ ceph osd pool create lrcpool 12 12 erasure LRCprofile


降低机架间的恢复带宽
--------------------

在 Firefly 版中，只有主 OSD 与丢失块位于同一机架时所需带宽才能降低。 ::

        $ ceph osd erasure-code-profile set LRCprofile \
             plugin=lrc \
             k=4 m=2 l=3 \
             ruleset-locality=rack \
             ruleset-failure-domain=host
        $ ceph osd pool create lrcpool 12 12 erasure LRCprofile


创建 lrc 配置
=============

要新建 *lrc* 纠删码配置： ::

        ceph osd erasure-code-profile set {name} \
             plugin=lrc \
             k={data-chunks} \
             m={coding-chunks} \
             l={locality} \
             [ruleset-root={root}] \
             [ruleset-locality={bucket-type}] \
             [ruleset-failure-domain={bucket-type}] \
             [directory={directory}] \
             [--force]

其中：


``k={data chunks}``

:描述: 各对象都被分割为\ **数据块**\ ，分别存储于不同 OSD 。
:类型: Integer
:是否必需: Yes.
:实例: 4


``m={coding-chunks}``

:描述: 计算各对象的\ **编码块**\ 、并存储于不同 OSD 。编码块的数量等同于在\
       不丢数据的前提下允许同时失效的 OSD 数量。

:类型: Integer
:是否必需: Yes.
:实例: 2


``l={locality}``

:描述: 把编码块和数据块分组为大小为 **locality** 的集合。比如， **k=4** 且 \
       **m=2** 时，若设置 **locality=3** ，将会分组为大小为三的两组，这样各组\
       都能自行恢复，无需从另一组读数据块。

:类型: Integer
:是否必需: Yes.
:实例: 3


``ruleset-root={root}``

:描述: 规则集第一步所指向的 CRUSH 桶之名，如 **step take default** 。
:类型: String
:是否必需: No.
:默认值: default


``ruleset-locality={bucket-type}``

:描述: 由 **l** 定义的块集合将按哪种 crush 桶类型存储。比如，若设\
       置为 **rack** ，大小为 **l** 块的各组将被存入不同的机架，\
       此值会被用于创建类似 **step choose rack** 的规则集。如果没\
       设置，就不会这样分组。

:类型: String
:是否必需: No.


``ruleset-failure-domain={bucket-type}``

:描述: 确保两个编码块不会存在于同一故障域的桶里面。比如，假设故障\
       域是 **host** ，就不会有两个编码块存储到同一主机；此值用于\
       在规则集中创建类似 **step chooseleaf host** 的步骤。

:类型: String
:是否必需: No.
:默认值: host


``directory={directory}``

:描述: 设置纠删码插件的路径，需是\ **目录**\ 。
:类型: String
:是否必需: No.
:默认值: /usr/lib/ceph/erasure-code


``--force``

:描述: 覆盖同名配置。
:类型: String
:是否必需: No.


低级插件配置
============

**k** 与 **m** 之和必须是 **l** 参数的整数倍。低级配置参数没有强加\
这样的限制，并且在某些场合下更方便。因此有可能配置两个组，一组 4 \
块、另一组 3 块；也有可能递归地定义局部集合，如数据中心和机架再组\
合为数据中心。 **k/m/l** 可通过生成低级配置来实现。

*lrc* 纠删码插件递归地使用纠删码技术，这样一些块丢失的恢复大多只需\
少部分数据块的子集。

比如，三步编码描述为如下： ::

   chunk nr    01234567
   step 1      _cDD_cDD
   step 2      cDDD____
   step 3      ____cDDD

其中， *c* 是从数据块 *D* 计算出的编码块，块 *7* 丢失后能从后四个\
块恢复，块 *2* 丢失后能从前四个块恢复。


使用低级配置的纠删码配置实例
============================

最小测试
--------

此例其实等价于默认纠删码配置， *DD* 其实就是 *K=2* 、 *c* 就是 \
*m=1* 并且默认使用 *jerasure* 插件。 ::

        $ ceph osd erasure-code-profile set LRCprofile \
             plugin=lrc \
             mapping=DD_ \
             layers='[ [ "DDc", "" ] ]'
        $ ceph osd pool create lrcpool 12 12 erasure LRCprofile


降低主机间的恢复带宽
--------------------

虽然当所有主机都接入同一交换机时，这不会是诱人的用法，但是带宽利用率确实降低\
了。它等价于 **k=4** 、 **m=2** 且 **l=3** ，尽管数据块的布局不同： ::

        $ ceph osd erasure-code-profile set LRCprofile \
             plugin=lrc \
             mapping=__DD__DD \
             layers='[
                       [ "_cDD_cDD", "" ],
                       [ "cDDD____", "" ],
                       [ "____cDDD", "" ],
                     ]'
        $ ceph osd pool create lrcpool 12 12 erasure LRCprofile


降低机架间的恢复带宽
--------------------

在 Firefly 版中，只有主 OSD 与丢失块位于同一机架时所需带宽才能降低。 ::

        $ ceph osd erasure-code-profile set LRCprofile \
             plugin=lrc \
             mapping=__DD__DD \
             layers='[
                       [ "_cDD_cDD", "" ],
                       [ "cDDD____", "" ],
                       [ "____cDDD", "" ],
                     ]' \
             ruleset-steps='[
                             [ "choose", "rack", 2 ],
                             [ "chooseleaf", "host", 4 ],
                            ]'
        $ ceph osd pool create lrcpool 12 12 erasure LRCprofile


不同纠删码后端测试
------------------

LRC 当前用 jerasure 作为默认 EC 后端。使用低级配置时，你可以为每一级分别指定 \
EC 后端、算法。 layers='[ [ "DDc", "" ] ]' 里的第二个参数其实是用于本级的纠删\
码配置。下面的例子为 lrcpool 存储池配置了 cauchy 技术的 ISA 后端。 ::

        $ ceph osd erasure-code-profile set LRCprofile \
             plugin=lrc \
             mapping=DD_ \
             layers='[ [ "DDc", "plugin=isa technique=cauchy" ] ]'
        $ ceph osd pool create lrcpool 12 12 erasure LRCprofile

你也可以为各级分别使用不同的纠删码配置。 ::

        $ ceph osd erasure-code-profile set LRCprofile \
             plugin=lrc \
             mapping=__DD__DD \
             layers='[
                       [ "_cDD_cDD", "plugin=isa technique=cauchy" ],
                       [ "cDDD____", "plugin=isa" ],
                       [ "____cDDD", "plugin=jerasure" ],
                     ]'
        $ ceph osd pool create lrcpool 12 12 erasure LRCprofile


纠删编码和解码算法
==================

在层描述中找出的步骤： ::

   chunk nr    01234567

   step 1      _cDD_cDD
   step 2      cDDD____
   step 3      ____cDDD

将被依次应用。比如一个 4K 的对象要被编码，它要先通过 **step 1** \
被分割为四个 1K 的块（四个大写的 D ），分别依次存储于 2 、 3 、 \
6 和 7 。这些数据产生了两个编码块（两个小写 c ），它们分别存储于\
\ 1 和 5 。

*step 2* 以相似的方式重用 *step 1* 创建的内容，并把单个编码块 *c* \
存储于位置 0 。最后四个下划线（ *_* ）标记是为提高可读性的，被忽\
略了。

*step 3* 把单个编码块存储到了位置 4 ， *step 1* 创建的三个块被用\
于计算此编码块，也就是 *step 1* 产生的编码块成了 *step 3* 的数据\
块。

如果 *2* 块丢失了： ::

   chunk nr    01234567

   step 1      _c D_cDD
   step 2      cD D____
   step 3      __ _cDDD

将通过解码来恢复它，反向依次执行： *step 3* 然后 *step 2* 最后是 \
*step 1* 。

*step 3* 对 *2* 一无所知（即它是下划线），所以跳过此步。

*step 2* 里的编码块存储在 *0* 块中，可用来恢复 *2* 块的内容。没有\
需要恢复的数据块了，不再考虑 *step 1* ，进程终止。

恢复块 *2* 需读取块 *0, 1, 3* 并写回块 *2* 。

如果块 *2, 3, 6* 丢失： ::

   chunk nr    01234567

   step 1      _c  _c D
   step 2      cD  __ _
   step 3      __  cD D

*step 3* 可恢复块 *6* 的内容： ::

   chunk nr    01234567

   step 1      _c  _cDD
   step 2      cD  ____
   step 3      __  cDDD

*step 2* 未能恢复被跳过了，因为丢失了两块（ *2, 3* ），它只能恢复一个块的丢失。

*step 1* 中的编码块位于块 *1, 5* ，因此能恢复块 *2, 3* 的内容。 ::

   chunk nr    01234567

   step 1      _cDD_cDD
   step 2      cDDD____
   step 3      ____cDDD


CRUSH 归置的控制
================

默认的 CRUSH 规则集会选择位于不同主机的 OSD ，例如： ::

   chunk nr    01234567

   step 1      _cDD_cDD
   step 2      cDDD____
   step 3      ____cDDD

需要整整 8 个 OSD ，分别存储 8 个块。如果这些主机分别位于相邻的机架，前四块可\
放到第一个机架，后四块可放到第二个机架，这样丢失单个 OSD 恢复时就不会用到机架\
间的带宽。

例如： ::

   ruleset-steps='[ [ "choose", "rack", 2 ], [ "chooseleaf", "host", 4 ] ]'

此配置会创建这样的规则集，选定类型为 *rack* 的两个 crush 桶、并在\
各桶中再选四个 OSD ，这几个 OSD 分别位于类型为 *host* 的不同桶中。

此规则集还可以手工雕琢一下，使其更精细。
