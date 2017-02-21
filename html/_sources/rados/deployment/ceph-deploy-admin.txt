==========
 管理任务
==========

用 ``ceph-deploy`` 建立一个集群后，你可以把客户端管理密钥和 Ceph 配置文件发给其他\
管理员，以便让他用 ``ceph`` 命令管理集群。


创建一管理主机
==============

要允许一主机以管理员权限执行 Ceph 命令，用 ``admin`` 命令： ::

	ceph-deploy admin {host-name [host-name]...}


分发配置文件
============

要把改过的配置文件分发给集群内各主机，可用 ``config push`` 命令。 ::

	ceph-deploy config push {host-name [host-name]...}

.. tip:: 用相同前缀和递增数字命名的话，部署可以通过一个命令轻易完成，如 \
   ``ceph-deploy config hostname{1,2,3,4,5}`` 。

检索配置文件
============

要从集群内的一主机检索配置文件，用 ``config pull`` 命令。 ::

	ceph-deploy config pull {host-name [host-name]...}
