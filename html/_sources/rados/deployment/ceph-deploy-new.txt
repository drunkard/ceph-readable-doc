============
 创建一集群
============

使用 ``ceph-deploy`` 的第一步就是新建一个集群，新集群具备：

- 一个 Ceph 配置文件，以及
- 一个监视器密钥环。

Ceph 配置文件至少要包含：

- 它自己的文件系统 ID （ ``fsid`` ）
- 最初的监视器（们）及其主机名（们），以及
- 最初的监视器及其 IP 地址。

详情见\ `监视器配置参考`_\ 。

``ceph-deploy`` 工具也创建了一个监视器密钥环并置于 ``[mon.]`` 内，详情见 \
`Cephx 手册`_\ 。


用法
----

要用 ``ceph-deploy`` 创建集群，用 ``new`` 命令、并指定几个主机作为初始监视器法定人数。 ::

	ceph-deploy new {host [host], ...}

例如： ::

	ceph-deploy new mon1.foo.com
	ceph-deploy new mon{1,2,3}

``ceph-deploy`` 工具会用 DNS 把主机名解析为 IP 地址。监视器将被命名为域名的第一段\
（如前述的 ``mon1`` ），它会把指定主机名加入 Ceph 配置文件。其他用法见： ::

	ceph-deploy new -h


命名集群
--------

Ceph 集群的默认名字为 ``ceph`` ，如果你想在同一套硬件上运行多套集群可以指定其他名\
字。比如，如果想优化一个集群用于块设备，另一个用作网关，你可以在同一套硬件上运行两\
个不同的集群，但是它们要配置不同的 ``fsid`` 和集群名。

	ceph-deploy --cluster {cluster-name} new {host [host], ...}

例如： ::

	ceph-deploy --cluster rbdcluster new ceph-mon1
	ceph-deploy --cluster rbdcluster new ceph-mon{1,2,3}

.. note:: 如果你运行多个集群，必须修改默认端口选项并为其打开端口，这样两个不同的集\
   群网才不会相互冲突。


.. _监视器配置参考: ../../configuration/mon-config-ref
.. _Cephx 手册: ../../operations/authentication#monitor-keyrings
