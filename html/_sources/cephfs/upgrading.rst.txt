.. _upgrade-mds-cluster:

升级 MDS 集群
=============
.. Upgrading the MDS Cluster

当前， MDS 集群还没有内置的版本管理或者文件系统标识，
可以让你避免由不兼容的消息或其它功能变化引起的断言或其它错误，
就能完成 MDS 的无缝升级，正因为如此，在所有集群升级期间，
都有必要首先把一个文件系统的活跃 MDS 数量减少成一个，
以免两个版本不同的活跃 MDS 进行通讯。

升级 MDS 集群的合理顺序是：

1. 给每个文件系统禁用、并停止热备（ standby-replay ）守护进程。

::

    ceph fs set <fs_name> allow_standby_replay false

在 Pacific 版中，运行这个命令就会停止热备守护进程。
更老版本的 Ceph 需要手动停止这些守护进程。

::

    ceph fs dump # find standby-replay daemons
    ceph mds fail mds.<X>


2. 把所有文件系统的 rank 都减少到 1:

::

    ceph fs set <fs_name> max_mds 1

3. 等着集群停掉非 0 的 rank ，也就是只有 rank 0 是活跃的，其余都是备用的。

::

    ceph status # wait for MDS to finish stopping

4. 在每一台 MDS 上，升级软件包并重启。注意：为了避免失败，
   建议这样做 -- 但不是必须的 -- 首先升级备用守护进程。

::

    # 用软件包管理器升级集群
    systemctl restart ceph-mds.target

5. 对每一个文件系统，恢复集群之前的 max_mds 和 allow_standby_replay 配置：

::

    ceph fs set <fs_name> max_mds <old_max_mds>
    ceph fs set <fs_name> allow_standby_replay <old_allow_standby_replay>


升级比 Firefly 老的文件系统，需过 Jewel 这个槛
==============================================
.. Upgrading pre-Firefly file systems past Jewel

.. tip::

    本建议是给那些用 *Firefly* (0.80) 之前的
    Ceph 创建了文件系统的用户们。
    打算创建新文件系统的用户可以忽略此建议。

Ceph 版本小于 Firefly 的，使用了现已废弃的格式\
来存储 CephFS 目录对象，也就是 TMAP 。
到 Jewel 版以后， RADOS 就不再支持此格式的读取了，
所以，对于升级 CephFS 的用户来说，
非常重要的事就是转换旧目录对象格式。

在所有的 MDS 和 OSD 服务器上安装 Jewel 版、并重启服务后，\
运行下面的命令：

::
    
    cephfs-data-scan tmap_upgrade <metadata pool name>

此命令只需运行一次，且运行期间没必要中止其它服务。
这个命令要运行不少时间，因为它得\
遍历元数据存储池内的所有对象。
在此命令运行期间，你可以放心地、正常使用文件系统。
如果此命令因故中止，可以放心地再次运行。

如果你想把一个比 Firefly 还老的 CephFS 文件系统升级到比
Jewel 更高的版本，必须先升级到 Jewel 版，然后运行
``tmap_upgrade`` 命令，之后才能升级到最新版本。

