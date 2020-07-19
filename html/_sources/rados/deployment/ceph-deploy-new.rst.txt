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

``ceph-deploy`` 工具也创建了一个监视器密钥环并置于 ``[mon.]``
内，详情见 `Cephx 手册`_\ 。


用法
----

要用 ``ceph-deploy`` 创建集群，用 ``new`` 命令、并指定几个主机\
作为初始监视器法定人数。 ::

	ceph-deploy new {host [host], ...}

例如： ::

	ceph-deploy new mon1.foo.com
	ceph-deploy new mon{1,2,3}

``ceph-deploy`` 工具会用 DNS 把主机名解析为 IP 地址。监视器将\
被命名为域名的第一段（如前述的 ``mon1`` ），它会把指定主机名加\
入 Ceph 配置文件。其他用法见： ::

	ceph-deploy new -h



.. _监视器配置参考: ../../configuration/mon-config-ref
.. _Cephx 手册: ../../../dev/mon-bootstrap#secret-keys
