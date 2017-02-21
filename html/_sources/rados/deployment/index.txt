===========
 Ceph 部署
===========

``ceph-deploy`` 工具是一种部署 Ceph 的方法，它只依赖到服务器的 SSH 访问、 \
``sudo`` 和 Python 。它可在你的工作站上运行，不需要服务器、数据库、或其它工具。如\
果你安装、拆卸过很多 Ceph 集群，不想要额外的工具，那 ``ceph-deploy`` 是理想之选。\
它不是个通用部署系统，只为 Ceph 用户设计，用它可以快速地设置并运行一个默认值较合理\
的集群，而无需头疼 `Chef`_ 、 Puppet 或 Juju 的安装。如果您想全面控制安全设置、分\
区、或目录位置，可以试试类似 Juju 、 Chef 、 Crowbar 的工具。

用 ``ceph-deploy`` 你可以自己开发脚本往远程主机上安装 Ceph 软件包、创建集群、增加\
监视器、收集（或销毁）密钥、增加 OSD 和元数据服务器、配置管理主机，甚至拆除集群。

.. raw:: html

	<table cellpadding="10"><tbody valign="top"><tr><td>

.. toctree:: 

	飞前检查 <preflight-checklist>
	安装 Ceph <ceph-deploy-install>

.. raw:: html

	</td><td>

.. toctree::

	创建集群 <ceph-deploy-new>
	增加/删除监视器 <ceph-deploy-mon>
	密钥管理 <ceph-deploy-keys>
	增加/删除 OSD <ceph-deploy-osd>
	增加/删除 MDS <ceph-deploy-mds>


.. raw:: html

	</td><td>

.. toctree::

	删减主机 <ceph-deploy-purge>
	管理任务 <ceph-deploy-admin>


.. raw:: html

	</td></tr></tbody></table>


.. _Chef: http://wiki.ceph.com/02Guides/Deploying_Ceph_with_Chef
