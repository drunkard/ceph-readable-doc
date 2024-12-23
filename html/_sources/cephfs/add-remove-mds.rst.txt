.. _cephfs_add_remote_mds:

.. warning:: 本资料仅用于手动配置起 Ceph 集群。
   如果您打算使用 :doc:`/cephadm/index`
   这样的自动化工具来配置 Ceph 集群，
   请勿使用本页面上的指导。

.. note:: 如果您确信知道自己在做什么，
   并打算手动部署 MDS 守护进程，
   请在动手之前参阅 :doc:`/cephadm/services/mds/` 。

==================
 部署元数据服务器
==================
.. Deploying Metadata Servers

每个 CephFS 文件系统至少需要一个 MDS 。
集群操作员通常会根据需要、使用自动部署工具来启动所需的 MDS 服务器。
建议使用 Rook 和 ansible （通过 ceph-ansible playbooks ）工具来完成此操作。
为清楚起见，我们还展示了 systemd 命令，如果在裸机上执行，部署工具可能会运行这些命令。

关于元数据服务器的配置，见 `MDS 配置参考`_\ 。


为 MDS 准备硬件
===============
.. Provisioning Hardware for an MDS

目前版本的 MDS 是单线程的，大多数活动
（包括响应客户请求）都绑在一个 CPU 上。
在客户端负载最大的情况下， MDS 大约使用 2 到 3 个 CPU 核心。
这是因为其他杂项维护线程在协同工作。

尽管如此，我们还是建议为 MDS 服务器配备有足够核心数的高端 CPU 。
目前正在进行开发，以更好地利用 MDS 中的闲置 CPU 核心；
希望在未来的 Ceph 版本中， MDS 服务器能够利用更多核心来提高性能。

影响 MDS 性能的另一个因素是可用于缓存的 RAM 。
MDS 必须管理所有客户端和其他活跃 MDS 之间的分布式、
且相互协作的元数据缓存。
因此，有必要给 MDS 提供足够的 RAM ，
才能让元数据的访问和变更快一些。
默认的 MDS 缓存大小（另请参阅 :doc:`/cephfs/cache-configuration` ）为 4GB 。
建议为 MDS 提供至少 8GB 内存，作为这样的缓存。

一般来说，为大型客户端集群（ 1000 个以上）提供服务的 MDS
将使用至少 64GB 的缓存。在已知的最大集群中，
使用更大缓存的 MDS 还没有深入探索过；可能会出现收益降低的情况，
即管理如此大的缓存会以意想不到的方式对性能产生负面影响。
最好在预想的载荷下进行分析，
以确定配置更多内存是否可行。

在裸机集群中，最佳做法是为 MDS 服务器超额配置硬件。
即使单个 MDS 守护进程不能充分利用硬件，
以后也可以根据需要在同一节点上启动更多活跃 MDS 守护进程，
以充分利用可用核心和内存。此外，
随着集群上载荷的增加，可能会发现在同一节点上\
启用多个活跃 MDS 比单个满载的 MDS 性能好。

最后，还要注意 CephFS 是一种高可用文件系统，支持热备 MDS
（另请参阅 :ref:`mds-standby` ），可以实现快速故障切换。
要让部署的热备 MDS 真能发挥作用，集群中的热备 MDS 守护进程\
通常有必要部署到两个以上的不同节点上。否则，
单个节点的硬件故障就可能导致文件系统不可用。

把 MDS 和其他 Ceph 守护进程（超融合）一起部署在同一台机器上是推荐做法，
只要硬件能满足所有守护进程的条件就行。
对于 MDS 而言，就意味着限制它的缓存尺寸。


增加一个 MDS
============
.. Adding an MDS

#. 创建 mds 目录 ``/var/lib/ceph/mds/ceph-${id}`` 。
   守护进程只用这个目录存储它的密钥。

#. 如果启用 CephX 的话，创建认证密钥： ::

	$ sudo ceph auth get-or-create mds.${id} mon 'profile mds' mgr 'profile mds' mds 'allow *' osd 'allow *' > /var/lib/ceph/mds/ceph-${id}/keyring

#. 启动服务： ::

	$ sudo systemctl start ceph-mds@${id}

#. 集群状态应该是： ::

	mds: ${id}:1 {0=${id}=up:active} 2 up:standby

#. 可选配置，让 MDS 加入哪个文件系统（ :ref:`mds-join-fs` ）： ::

    $ ceph config set mds.${id} mds_join_fs ${fs}


删除一个 MDS
============
.. Removing an MDS

如果你有想要删除的元数据服务器，可以用以下方法。

#. (可选操作：）创建元数据服务器的新替代。
   如果删除 MDS 后没有替代的 MDS 来接管，
   客户端将无法使用文件系统。如果不希望出现这种情况，
   应该在拆除元数据服务器之前先添加一个元数据服务器。

#. 关闭要删除的 MDS 。 ::

	$ sudo systemctl stop ceph-mds@${id}

   MDS 会自动通知所有 Ceph 监视器它将停机。
   这样，监视器就能即时把业务切换到可用的热备（如果有的话）。
   无需使用管理命令做故障切换，
   比如用 ``ceph mds fail mds.${id}`` 。

#. 删除 MDS 服务器的 ``/var/lib/ceph/mds/ceph-${id}`` 目录。 ::

	$ sudo rm -rf /var/lib/ceph/mds/ceph-${id}


.. note:: 如果活跃 MDS 有 MDS_TRIM 或 MDS_CACHE_OVERSIZED 健康警告时，
   需要加上确认标志（ ``--yes-i-really-mean-it`` ），
   否则命令会失败。不建议重启有这些警告的 MDS ，
   因为重启时恢复缓慢，可能会导致更多问题。


.. _MDS 配置参考: ../mds-config-ref
