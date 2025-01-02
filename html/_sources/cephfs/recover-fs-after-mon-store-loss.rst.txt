监视器存储灾难性丢失后恢复文件系统
==================================
.. Recovering the file system after catastrophic Monitor store loss

在极少数情况下，集群的所有监视器存储可能被损坏或丢失。
要在这种情况下恢复集群，需要使用 OSD 重建监视器存储
（请参阅 :ref:`mon-store-recovery-using-osds` ），
并完整地恢复存储池（状态变为 active+clean ）。
然而，重建的监视器存储不能恢复文件系统映射（ FSMap ）。
要恢复文件系统，还需要额外的步骤。
恢复多活 MDS 文件系统或多个文件系统的步骤尚未建立。目前，
只完成并测试了恢复\ **单活 MDS** 文件系统的步骤，
而且还是集群只有一个文件系统的情况。
这些步骤大致是：使用基本的默认值重新创建 FSMap ；
然后让 MDS 自己根据文件系统存储池中的日志/元数据恢复。
下面将详细介绍这些步骤。

首先，用恢复的文件系统存储池重新创建文件系统。
新 FSMap 将具备文件系统的默认设置。但是，用户自定义的文件系统设置，
比如 ``standby_count_wanted`` 、 ``required_client_features`` 、
额外的数据存储池、等等，这些配置丢失了，之后需要重新应用。

::

    ceph fs new <fs_name> <metadata_pool> <data_pool> --force --recover

``recover`` 标志会将文件系统 rank 0 的状态设置为“存在但故障”。
因此，当一个 MDS 守护进程捡起 rank 0 时，它会读取现有的、
RADOS 内的元数据，而且不会覆盖它。
该标志还能防止备用 MDS 守护进程激活文件系统。

文件系统的文件系统集群 ID ，就是 fscid 将不会保留。
有些应用程序（如 Ceph CSI ）希望文件系统在恢复后保持不变，
那么这样的恢复结果就不太符合期望。要解决这个问题，
可以在上述命令中（选择性地）设置 ``fscid`` 选项
（参阅 :ref:`advanced-cephfs-admin-settings` ）。

允许备用 MDS 守护进程加入文件系统。

::

    ceph fs set <fs_name> joinable true


检查下，文件系统应该不是降级状态、且有一个活跃 MDS 。

::

    ceph fs status

重新应用其它自定义的文件系统配置。
