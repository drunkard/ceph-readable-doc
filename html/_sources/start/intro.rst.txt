==========
 Ceph 简介
==========

不管你是想让\ :term:`云平台`\ 提供 :term:`Ceph 对象存储`\
和/或 :term:`Ceph 块设备`\ 服务、还是部署一个
:term:`Ceph 文件系统`\ 、或者用 Ceph 干别的事，所有
:term:`Ceph 存储集群`\ 的部署都始于各 :term:`Ceph 节点`\ 、\
网络和 Ceph 存储集群的配置。最简的 Ceph 存储集群至少要一个
Ceph 监视器、 Ceph 管理器、和 Ceph OSD （对象存储守护进程）。\
要跑 Ceph 文件系统客户端的话，还必须有 Ceph 元数据服务器。

.. ditaa::  +---------------+ +------------+ +------------+ +---------------+
            |      OSDs     | | Monitors   | |  Managers  | |      MDSs     |
            +---------------+ +------------+ +------------+ +---------------+

- **Monitors**: :term:`Ceph 监视器`\ (``ceph-mon``) 维护着集群\
  状态的各种运行图，包括监视器运行图、管理器运行图、
  OSD 运行图、 CRUSH 图，这些运行图都是很要紧的集群状态，对于\
  各种 Ceph 守护进程的相互协作必不可少。监视器还负责管理守护\
  进程和客户端之间的认证。考虑到冗余性和高可用性，一般都要求\
  至少有三个监视器。

- **Managers**: :term:`Ceph 管理器`\ 守护进程（ ``ceph-mgr``
  ）负责持续跟踪运行时指标和 Ceph 当前的状态，包括存储利用率、\
  当前的性能指标、和系统负载。 Ceph 管理器守护进程还托管着基于
  python 的插件，用于管理和展示 Ceph 集群信息，包括一个基于\
  网页的 :ref:`mgr-dashboard` 和 `REST API`_ 。为保障\
  高可用性，一般要求至少有两个管理器。

- **Ceph OSDs**: :term:`Ceph OSD` （对象存储守护进程，
  ``ceph-osd`` ）负责存储数据、处理数据复制、恢复、重均衡、\
  以及向 Ceph 监视器和管理器提供些监控信息，如检查其它
  Ceph OSD 守护进程的心跳。为保障冗余性和高可用性，一般需要\
  至少 3 个 Ceph OSD 。

- **MDSs**: :term:`Ceph 元数据服务器`\ （ MDS ， ``ceph-mds``
  ）为 :term:`Ceph 文件系统`\ 存储元数据（也就是说， Ceph
  块设备和 Ceph 对象存储不使用 MDS ）。元数据服务器有益于
  POSIX 文件系统用户执行基本命令，（像 ``ls`` 、 ``find``
  等等），避免了给 Ceph 存储集群增加过重的负担。

Ceph 把数据保存为逻辑存储池内的对象。根据 :term:`CRUSH` 算法，
Ceph 可计算出哪个归置组应该持有指定对象，然后进一步计算出哪个
OSD 守护进程持有归置组，正因为有了 CRUSH 算法， Ceph 存储集群\
才具备动态伸缩、重均衡和动态恢复功能。

.. _REST API: ../../mgr/restful


.. raw:: html

	<style type="text/css">div.body h3{margin:5px 0px 0px 0px;}</style>
	<table cellpadding="10"><colgroup><col width="50%"><col width="50%"></colgroup><tbody valign="top"><tr><td><h3>建议</h3>

开始把 Ceph 用于生产环境前，您应该看看我们的硬件建议和操作系统\
建议。

.. toctree::
   :maxdepth: 2

   硬件推荐 <hardware-recommendations>
   操作系统推荐 <os-recommendations>


.. raw:: html

	</td><td><h3>参与</h3>

   欢迎您加入社区，贡献文档、代码，或发现软件缺陷。

.. toctree::
   :maxdepth: 2

   get-involved
   documenting-ceph

.. raw:: html

	</td></tr></tbody></table>
