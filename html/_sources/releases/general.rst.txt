.. Ceph Releases (general)
.. _ceph-releases-general:

====================
 Ceph 版本（总目录）
====================

.. toctree::
    :maxdepth: 1

.. Active stable releases

活跃的稳定版
------------

.. ceph_releases:: releases.yml

.. Understanding the release cycle

理解发布周期
------------

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

.. Development releases (x.0.z)

开发版（ x.0.z ）
^^^^^^^^^^^^^^^^^

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

.. Release candidates (x.1.z)

候选发布版（ x.1.z ）
^^^^^^^^^^^^^^^^^^^^^

从起初计划的稳定版起，差不多每 6 周就有一个功能版，之后的工作\
重心将转移，仅限于稳定性和缺陷修正。

* 候选发布版每 1-2 周发布一个
* 用于最终测试，以及验证即将到来的稳定版

.. Stable releases (x.2.z)

稳定版（ x.2.z ）
^^^^^^^^^^^^^^^^^

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

.. Lifetime of stable releases

稳定版的生命周期
----------------

The lifetime of a stable release series is calculated to be approximately 24
months (i.e., two 12 month release cycles) after the month of the first release.
For example, Mimic (13.2.z) will reach end of life (EOL) shortly after Octopus
(15.2.0) is released. The lifetime of a release may vary because it depends on
how quickly the stable releases are published.

Jewel 和 Kraken 版的寿命稍有不同。在 Luminous 之前，每两个稳\
定版只有一个会成为 LTS 版，因此，

* 这样升级 Jewel -> Kraken -> Luminous 或 Jewel -> Luminous 是\
  没问题的；
* 从 Jewel 或 Kraken 开始升级的话，必须先升级到 Luminous 才能\
  再往前（如 Kraken -> Luminous -> Mimic 但不能是 Kraken ->
  Mimic ）。
* Jewel 一直维护到 Mimic 发布为止（2018年6月）；
* Kraken 就不再维护了。

Detailed information on all releases, past and present, can be found at :ref:`ceph-releases-index`


.. Release timeline

发布时间表
----------

.. ceph_timeline:: releases.yml development nautilus mimic luminous kraken jewel infernalis hammer giant firefly emperor

.. _Octopus: ../octopus
.. _15.2.4: ../octopus#v15-2-4-octopus
.. _15.2.3: ../octopus#v15-2-3-octopus
.. _15.2.2: ../octopus#v15-2-2-octopus
.. _15.2.1: ../octopus#v15-2-1-octopus
.. _15.2.0: ../octopus#v15-2-0-octopus

.. _Nautilus: ../nautilus
.. _14.2.10: ../nautilus#v14-2-10-nautilus
.. _14.2.9: ../nautilus#v14-2-9-nautilus
.. _14.2.8: ../nautilus#v14-2-8-nautilus
.. _14.2.7: ../nautilus#v14-2-7-nautilus
.. _14.2.6: ../nautilus#v14-2-6-nautilus
.. _14.2.5: ../nautilus#v14-2-5-nautilus
.. _14.2.4: ../nautilus#v14-2-4-nautilus
.. _14.2.3: ../nautilus#v14-2-3-nautilus
.. _14.2.2: ../nautilus#v14-2-2-nautilus
.. _14.2.1: ../nautilus#v14-2-1-nautilus
.. _14.2.0: ../nautilus#v14-2-0-nautilus

.. _Mimic: ../mimic
.. _13.2.10: ../mimic#v13-2-10-mimic
.. _13.2.9: ../mimic#v13-2-9-mimic
.. _13.2.8: ../mimic#v13-2-8-mimic
.. _13.2.7: ../mimic#v13-2-7-mimic
.. _13.2.6: ../mimic#v13-2-6-mimic
.. _13.2.5: ../mimic#v13-2-5-mimic
.. _13.2.4: ../mimic#v13-2-4-mimic
.. _13.2.3: ../mimic#v13-2-3-mimic
.. _13.2.2: ../mimic#v13-2-2-mimic
.. _13.2.1: ../mimic#v13-2-1-mimic
.. _13.2.0: ../mimic#v13-2-0-mimic

.. _Luminous: ../luminous
.. _12.2.13: ../luminous#v12-2-13-luminous
.. _12.2.12: ../luminous#v12-2-12-luminous
.. _12.2.11: ../luminous#v12-2-11-luminous
.. _12.2.10: ../luminous#v12-2-10-luminous
.. _12.2.9: ../luminous#v12-2-9-luminous
.. _12.2.8: ../luminous#v12-2-8-luminous
.. _12.2.7: ../luminous#v12-2-7-luminous
.. _12.2.6: ../luminous#v12-2-6-luminous
.. _12.2.5: ../luminous#v12-2-5-luminous
.. _12.2.4: ../luminous#v12-2-4-luminous
.. _12.2.3: ../luminous#v12-2-3-luminous
.. _12.2.2: ../luminous#v12-2-2-luminous
.. _12.2.1: ../luminous#v12-2-1-luminous
.. _12.2.0: ../luminous#v12-2-0-luminous

.. _11.2.1: ../kraken#v11-2-1-kraken
.. _11.2.0: ../kraken#v11-2-0-kraken
.. _Kraken: ../kraken#v11-2-0-kraken

.. _11.0.2: ../kraken#v11-0-2-kraken

.. _10.2.11: ../jewel#v10-2-11-jewel
.. _10.2.10: ../jewel#v10-2-10-jewel
.. _10.2.9: ../jewel#v10-2-9-jewel
.. _10.2.8: ../jewel#v10-2-8-jewel
.. _10.2.7: ../jewel#v10-2-7-jewel
.. _10.2.6: ../jewel#v10-2-6-jewel
.. _10.2.5: ../jewel#v10-2-5-jewel
.. _10.2.4: ../jewel#v10-2-4-jewel
.. _10.2.3: ../jewel#v10-2-3-jewel
.. _10.2.2: ../jewel#v10-2-2-jewel
.. _10.2.1: ../jewel#v10-2-1-jewel
.. _10.2.0: ../jewel#v10-2-0-jewel
.. _Jewel: ../jewel#v10-2-0-jewel

.. _10.1.2: ../jewel#v10-1-2-jewel-release-candidate
.. _10.1.1: ../jewel#v10-1-1-jewel-release-candidate
.. _10.1.0: ../jewel#v10-1-0-jewel-release-candidate
.. _10.0.5: ../jewel#v10-0-5
.. _10.0.3: ../jewel#v10-0-3
.. _10.0.2: ../jewel#v10-0-2
.. _10.0.1: ../jewel#v10-0-1
.. _10.0.0: ../jewel#v10-0-0

.. _9.2.1: ../infernalis#v9-2-1-infernalis
.. _9.2.0: ../infernalis#v9-2-0-infernalis
.. _Infernalis: ../infernalis#v9-2-0-infernalis

.. _9.1.0: ../infernalis#v9-1-0
.. _9.0.3: ../infernalis#v9-0-3
.. _9.0.2: ../infernalis#v9-0-2
.. _9.0.1: ../infernalis#v9-0-1
.. _9.0.0: ../infernalis#v9-0-0

.. _0.94.10: ../hammer#v0-94-10-hammer
.. _0.94.9: ../hammer#v0-94-9-hammer
.. _0.94.8: ../hammer#v0-94-8-hammer
.. _0.94.7: ../hammer#v0-94-7-hammer
.. _0.94.6: ../hammer#v0-94-6-hammer
.. _0.94.5: ../hammer#v0-94-5-hammer
.. _0.94.4: ../hammer#v0-94-4-hammer
.. _0.94.3: ../hammer#v0-94-3-hammer
.. _0.94.2: ../hammer#v0-94-2-hammer
.. _0.94.1: ../hammer#v0-94-1-hammer
.. _0.94: ../hammer#v0-94-hammer
.. _Hammer: ../hammer#v0-94-hammer

.. _0.93: ../hammer#v0-93
.. _0.92: ../hammer#v0-92
.. _0.91: ../hammer#v0-91
.. _0.90: ../hammer#v0-90
.. _0.89: ../hammer#v0-89
.. _0.88: ../hammer#v0-88

.. _0.87.2: ../giant#v0-87-2-giant
.. _0.87.1: ../giant#v0-87-1-giant
.. _0.87: ../giant#v0-87-giant
.. _Giant: ../giant#v0-87-giant

.. _0.86: ../giant#v0-86
.. _0.85: ../giant#v0-85
.. _0.84: ../giant#v0-84
.. _0.83: ../giant#v0-83
.. _0.82: ../giant#v0-82
.. _0.81: ../giant#v0-81

.. _0.80.11: ../firefly#v0-80-11-firefly
.. _0.80.10: ../firefly#v0-80-10-firefly
.. _0.80.9: ../firefly#v0-80-9-firefly
.. _0.80.8: ../firefly#v0-80-8-firefly
.. _0.80.7: ../firefly#v0-80-7-firefly
.. _0.80.6: ../firefly#v0-80-6-firefly
.. _0.80.5: ../firefly#v0-80-5-firefly
.. _0.80.4: ../firefly#v0-80-4-firefly
.. _0.80.3: ../firefly#v0-80-3-firefly
.. _0.80.2: ../firefly#v0-80-2-firefly
.. _0.80.1: ../firefly#v0-80-1-firefly
.. _0.80: ../firefly#v0-80-firefly
.. _Firefly: ../firefly#v0-80-firefly

.. _0.79: ../firefly#v0-79
.. _0.78: ../firefly#v0-78
.. _0.77: ../firefly#v0-77
.. _0.76: ../firefly#v0-76
.. _0.75: ../firefly#v0-75
.. _0.74: ../firefly#v0-74
.. _0.73: ../firefly#v0-73

.. _0.72.2: ../emperor#v0-72-2-emperor
.. _0.72.1: ../emperor#v0-72-1-emperor
.. _0.72: ../emperor#v0-72-emperor
.. _Emperor: ../emperor#v0-72-emperor

.. _0.71: ../dumpling#v0-71
.. _0.70: ../dumpling#v0-70
.. _0.69: ../dumpling#v0-69
.. _0.68: ../dumpling#v0-68

.. _0.67.11: ../dumpling#v0-67-11-dumpling
.. _0.67.10: ../dumpling#v0-67-10-dumpling
.. _0.67.9: ../dumpling#v0-67-9-dumpling
.. _0.67.8: ../dumpling#v0-67-8-dumpling
.. _0.67.7: ../dumpling#v0-67-7-dumpling
.. _0.67.6: ../dumpling#v0-67-6-dumpling
.. _0.67.5: ../dumpling#v0-67-5-dumpling
.. _0.67.4: ../dumpling#v0-67-4-dumpling
.. _0.67.3: ../dumpling#v0-67-3-dumpling
.. _0.67.2: ../dumpling#v0-67-2-dumpling
.. _0.67.1: ../dumpling#v0-67-1-dumpling
.. _0.67: ../dumpling#v0-67-dumpling
.. _Dumpling:  ../dumpling#v0-67-dumpling
