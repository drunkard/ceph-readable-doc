.. _governance:

==========
 项目管理
==========

Ceph 开源社区由几个不同的团体引导着。

项目负责人
----------
.. Project Leader

Ceph 项目现在是由 Sage Weil <sage@redhat.com> 领导的。
项目领导人负责指引整个项目的总体方向，并维持开发者和用户社区的健康发展。


提交者
------
.. Committers

提交者是有 Ceph 中央源码仓库写权限的项目贡献者，当前托管在 GitHub 上。
这一组开发者都被授权了、可以更改 Ceph 源代码。

一般来说，个人不应该单独做更改：
所有代码的贡献都通过一个协作式的审核流程
（并经过测试）才能并入。
这一流程的具体实施是动态的、并随时间变迁。

新的提交者会加入项目（或者提交者被移出项目）都是由
Ceph 领导团队（下述）谨慎地决定的。
成为贡献者的条件包括水平相当的质量、
和对项目的长期投入。


.. _clt:

Ceph 领导团队
-------------
.. Ceph Leadership Team

Ceph 领导团队 (CLT) 是一组组件领导和其他核心开发者们，
他们共同为项目做技术决策。
这些决策通常都是协商一致后做出的，
但必要时也会投票决定。

CLT 每周通过视频连线来讨论还没定论的问题或者决策。
CLT 会议记录发布在
`https://pad.ceph.com/p/clt-weekly-minutes <https://pad.ceph.com/p/clt-weekly-minutes>`_ 。

提交者的加入或移出都由 CLT 自己小心决定。

当前的 CLT 成员有：

 * Casey Bodley <cbodley@redhat.com>
 * Dan van der Ster <daniel.vanderster@cern.ch>
 * David Galloway <dgallowa@redhat.com>
 * David Orman <ormandj@iland.com>
 * Ernesto Puerta <epuerta@redhat.com>
 * Gregory Farnum <gfarnum@redhat.com>
 * Haomai Wang <haomai@xsky.com>
 * Ilya Dryomov <idryomov@redhat.com>
 * Igor Fedotov <igor.fedotov@croit.io>
 * Jeff Layton <jlayton@redhat.com>
 * Josh Durgin <jdurgin@redhat.com>
 * João Eduardo Luis <joao@suse.de>
 * Ken Dreyer <kdreyer@redhat.com>
 * Mark Nelson <mnelson@redhat.com>
 * Matt Benjamin <mbenjami@redhat.com>
 * Mike Perez <miperez@redhat.com>
 * Myoungwon Oh <myoungwon.oh@samsung.com>
 * Neha Ojha <nojha@redhat.com>
 * Patrick Donnelly <pdonnell@redhat.com>
 * Sage Weil <sage@redhat.com>
 * Sam Just <sjust@redhat.com>
 * Sebastian Wagner <sewagner@redhat.com>
 * Xie Xingguo <xie.xingguo@zte.com.cn>
 * Yehuda Sadeh <yehuda@redhat.com>
 * Yuri Weinstein <yweinste@redhat.com>
 * Zac Dover <zac.dover@gmail.com>


组件负责人
----------
.. Component Leads

Ceph 项目的每个主要的子组件都有一名主管工程师，他负责指导和协调开发进程。
这些主管由 CLT 的项目领导谨慎地提名或任命。
主管的责任有：

 * 通过视频聊天的方式指导每天（通常是）的“紧急（ stand-up ）”协调电话；
 * 为每个版本周期描绘开发路线图；
 * 协调贡献者之间的开发活动；
 * 确保贡献得到审查；
 * 确保不同的修改建议没有冲突；
 * 确保测试能顺利通过（新功能，包括测试、更改的测试能通过，等等）

所有的组件主管聚集在 CLT 周围。
他们应该向领导团队的其他成员报告进度和状态更新，
并帮助实现开发时的跨组件协调。


Ceph 基金会
-----------
.. The Ceph Foundation

Ceph 基金会是 Linux 基金会下的定向基金会，
主要任务是支持 Ceph 项目的社区和生态。
它对 Ceph 开源项目除了对协作式开发进程提供回馈和输入之外，
对技术方向没有直接的控制。

更多信息见 :ref:`foundation` 。

