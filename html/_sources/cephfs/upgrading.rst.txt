.. Upgrading the MDS Cluster

升级 MDS 集群
=============

Currently the MDS cluster does not have built-in versioning or file system
flags to support seamless upgrades of the MDSs without potentially causing
assertions or other faults due to incompatible messages or other functional
differences. For this reason, it's necessary during any cluster upgrade to
reduce the number of active MDS for a file system to one first so that two
active MDS do not communicate with different versions.  Further, it's also
necessary to take standbys offline as any new CompatSet flags will propagate
via the MDSMap to all MDS and cause older MDS to suicide.

The proper sequence for upgrading the MDS cluster is:

1. Reduce the number of ranks to 1:

::

    ceph fs set <fs_name> max_mds 1

2. Wait for cluster to stop non-zero ranks where only rank 0 is active and the rest are standbys.

::

    ceph status # wait for MDS to finish stopping

3. Take all standbys offline, e.g. using systemctl:

::

    systemctl stop ceph-mds.target

4. Confirm only one MDS is online and is rank 0 for your FS:

::

    ceph status

5. Upgrade the single active MDS, e.g. using systemctl:

::

    # use package manager to update cluster
    systemctl restart ceph-mds.target

6. Upgrade/start the standby daemons.

::

    # use package manager to update cluster
    systemctl restart ceph-mds.target

7. Restore the previous max_mds for your cluster:

::

    ceph fs set <fs_name> max_mds <old_max_mds>


.. Upgrading pre-Firefly file systems past Jewel

升级比 Firefly 老的文件系统，需过 Jewel 这个槛
==============================================

.. tip::

    本建议是给那些用 *Firefly* (0.80) 之前的 Ceph 创建了文\
    件系统的用户们。打算创建新文件系统的用户可以忽略此建议。

Ceph 版本小于 Firefly 的，使用了现已废弃的格式来存储 CephFS
目录对象，也就是 TMAP 。到 Jewel 版以后， RADOS 就不再支持\
此格式的读取了，所以，对于升级 CephFS 的用户来说，非常重要\
的事就是转换旧目录对象格式。

在所有的 MDS 和 OSD 服务器上安装 Jewel 版、并重启服务后，\
运行下面的命令：

::
    
    cephfs-data-scan tmap_upgrade <metadata pool name>

此命令只需运行一次，且运行期间没必要中止其它服务。这个命令\
要运行不少时间，因为它得遍历元数据存储池内的所有对象。在此\
命令运行期间，你可以放心地、正常使用文件系统。如果此命令因\
故中止，可以放心地再次运行。

如果你想把一个比 Firefly 还老的 CephFS 文件系统升级到比
Jewel 更高的版本，必须先升级到 Jewel 版，然后运行
``tmap_upgrade`` 命令，之后才能升级到最新版本。

