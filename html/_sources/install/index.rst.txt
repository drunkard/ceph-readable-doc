.. _install-overview:

===========
 安装 Ceph
===========

有好几种方法安装 Ceph 。请选择最适合你的。


建议的方法
~~~~~~~~~~

:ref:`Cephadm <cephadm>` 用容器和 systemd 安装和管理 Ceph 集群，
带有松耦合的 CLI 和 GUI 仪表盘。

* cephadm 只支持 Octopus 和更新的版本。
* cephadm 已经和新的编排 API 完整对接了、
  并且完整支持新的 CLI 和仪表盘功能，
  以此来管理集群部署。
* cephadm 需要容器功能的支持（ podman 或 docker ）、还有 Python 3。

`Rook <https://rook.io/>`_ 可以部署和管理 Ceph 集群，让它们在 Kubernetes 里运行，
同时还能管理存储资源、并通过 Kubernetes API 提供服务。
我们建议通过用 Rook 让 Ceph 在 Kubernetes 里运行、或者\
把现有的 Ceph 存储集群连接到 Kubernetes 。

* Rook 只支持 Nautilus 和更新版本的 Ceph。
* Rook 是在 Kubernetes 上运行 Ceph 、
  或者把 Kubernetes 集群连接到一个\
  已有（外部的） Ceph 集群的首选方式。
* Rook 支持新的编排器 API ，也完全支持 CLI 和仪表盘里的新管理功能。


其他方法
~~~~~~~~
.. Other methods

`ceph-ansible <https://docs.ceph.com/ceph-ansible/>`_ 使用
Ansible 来部署和管理 Ceph 集群。

* ceph-ansible 已经广泛部署了。
* ceph-ansible 还没有和新的编排器 API 对接好，
  它是在 Nautilus 和 Octopus 才引进的，这就意味着\
  较新的管理功能和仪表盘对接还不能用。


`ceph-deploy <https://docs.ceph.com/projects/ceph-deploy/en/latest/>`_ 是个工具，用于快速部署集群。

  .. IMPORTANT::
     ceph-deploy 已经不再维护了。它在 Nautilus 之后的 Ceph 上没有测试过。
     它不支持 RHEL8 、 CentOS 8 、或者较新的操作系统。

`ceph-salt <https://github.com/ceph/ceph-salt>`_ 使用 Salt 和 cephadm 按照 Ceph 。

`jaas.ai/ceph-mon <https://jaas.ai/ceph-mon>`_ 用 Juju 安装 Ceph 。

`github.com/openstack/puppet-ceph <https://github.com/openstack/puppet-ceph>`_  通过 Puppet 安装 Ceph 。

Ceph 还可以 :ref:`手动安装 <install-manual>`.

.. toctree::
   :hidden:

   index_manual


Windows
~~~~~~~

在 Windows 上的安装方式，请参考这份文档：
`Windows 安装指南`_.

.. _Windows 安装指南: ./windows-install
