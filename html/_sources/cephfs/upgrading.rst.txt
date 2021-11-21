.. _upgrade-mds-cluster:

升级 MDS 集群
=============
.. Upgrading the MDS Cluster

Currently the MDS cluster does not have built-in versioning or file system
flags to support seamless upgrades of the MDSs without potentially causing
assertions or other faults due to incompatible messages or other functional
differences. For this reason, it's necessary during any cluster upgrade to
reduce the number of active MDS for a file system to one first so that two
active MDS do not communicate with different versions.

The proper sequence for upgrading the MDS cluster is:

1. For each file system, disable and stop standby-replay daemons.

::

    ceph fs set <fs_name> allow_standby_replay false

In Pacific, the standby-replay daemons are stopped for you after running this
command. Older versions of Ceph require you to stop these daemons manually.

::

    ceph fs dump # find standby-replay daemons
    ceph mds fail mds.<X>


2. For each file system, reduce the number of ranks to 1:

::

    ceph fs set <fs_name> max_mds 1

3. Wait for cluster to stop non-zero ranks where only rank 0 is active and the rest are standbys.

::

    ceph status # wait for MDS to finish stopping

4. For each MDS, upgrade packages and restart. Note: to reduce failovers, it is
   recommended -- but not strictly necessary -- to first upgrade standby daemons.

::

    # use package manager to update cluster
    systemctl restart ceph-mds.target

5. For each file system, restore the previous max_mds and allow_standby_replay settings for your cluster:

::

    ceph fs set <fs_name> max_mds <old_max_mds>
    ceph fs set <fs_name> allow_standby_replay <old_allow_standby_replay>


升级比 Firefly 老的文件系统，需过 Jewel 这个槛
==============================================
.. Upgrading pre-Firefly file systems past Jewel

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

