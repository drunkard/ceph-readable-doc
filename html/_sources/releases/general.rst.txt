.. _ceph-releases-general:

====================
 Ceph 版本（总目录）
====================

.. toctree::
    :maxdepth: 1

理解发布周期
------------
.. Understanding the release cycle

从 Nautilus 版（ 14.2.0 ）之后，每年是一个新的稳定版发布周期。\
各系列的稳定版都会取个名字（如 Mimic ）、和一个主版本号（如
Mimic 的是 13 ，因为 M 是字母表的第十三个）。

Releases are named after a species of cephalopod (usually the common
name, since the latin names are harder to remember or pronounce).

版本号包含三部分， *x.y.z* 。 *x* 标识发布周期（如 Mimic 的是
13 ）； *y* 标识发布类型：

* x.0.z - 开发版（适合早期测试人员、和胆肥的人）
* x.1.z - 候选发布版（适合测试集群、胆肥的用户）
* x.2.z - 稳定、缺陷修正版（适合普通用户）

这个版本命名惯例始于 9.y.z Infernalis 期间。在此之前，开发版\
定为 0.y 、稳定版是 0.y.z 。

开发版（ x.0.z ）
^^^^^^^^^^^^^^^^^
.. Development releases (x.0.z)

主开发分支冻结后形成开发版（ x.0.z ）、而且发布前还要进行\
`集成和升级测试
<https://github.com/ceph/ceph/tree/master/qa/suites/>`_\ 。\
一旦发布，就不再移植修正补丁；开发者们将致力于下一个开发版，\
一般也就几周时间。

* 每 8 到 12 周一个开发版
* 意在用于测试，不是生产环境部署
* 完整的集成测试
* 从上一个稳定版升级的测试
* 我们尽最大努力，以确保从前一个开发版可以成功地\ *离线*\ 升级\
  （就是说你可以停掉所有守护进程、升级、然后重启）；开发版\
  期间不会刻意保证在线滚动升级。这样便于在非生产的测试集群上\
  部署开发版，而不用重新注入数据。


候选发布版（ x.1.z ）
^^^^^^^^^^^^^^^^^^^^^
.. Release candidates (x.1.z)

从起初计划的稳定版起，差不多每 6 周就有一个功能版，之后的工作\
重心将转移，仅限于稳定性和缺陷修正。

* 候选发布版每 1-2 周发布一个
* 用于最终测试，以及验证即将到来的稳定版


稳定版（ x.2.z ）
^^^^^^^^^^^^^^^^^
.. Stable releases (x.2.z)

一旦初始稳定版确定（ x.2.0 ），之后会有不定时的修正版，包含了\
缺陷修正和（偶尔的）小功能移植。缺陷修正会暂存起来，并包含在下\
一个小版本中。

* 每 4 到 6 周一个稳定小版本
* 用于生产环境部署
* 在两个完整的发布周期间持续移植缺陷修正
* 可以从前两个稳定版（从 Luminous 起）在线升级、滚动升级，并\
  通过了测试；
* 可以从前一个稳定的小版本在线升级、滚动升级，并通过了测试；

对于各稳定版：

* 定期进行\ `集成和升级测试
  <https://github.com/ceph/ceph/tree/master/qa/suites/>`_\ ，\
  而且 Ceph 会分析\ `其结果 <http://pulpito.ceph.com/>`_\ 。
* 在开发分支（ ``master`` ）中已经修正的\ `问题
  <http://tracker.ceph.com/projects/ceph/issues?query_id=27>`_\
  也会按计划移植；
* 如果稳定版里发现了个问题、并\
  `报告 <http://tracker.ceph.com/projects/ceph/issues/new>`_\
  到了上游， Ceph 开发者会分类处理。
* `稳定版和移植小组 <http://tracker.ceph.com/projects/ceph-releases/wiki>`_\
  负责发布\ ``小版本``\ ，其中包含已经移植到稳定版的修正补丁。


稳定版的生命周期
----------------
.. Lifetime of stable releases

The lifetime of a stable release series is calculated to be approximately 24
months (i.e., two 12 month release cycles) after the month of the first release.
For example, Mimic (13.2.z) will reach end of life (EOL) shortly after Octopus
(15.2.0) is released. The lifetime of a release may vary because it depends on
how quickly the stable releases are published.

Jewel 和 Kraken 版的生命周期和前面说的不太一样。在 Luminous 之前，每两个稳\
定版只有一个会成为 LTS 版。

Detailed information on all releases, past and present, can be found at :ref:`ceph-releases-index`

