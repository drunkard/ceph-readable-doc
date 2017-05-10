================
 开发集群的部署
================

要从事 ceph 开发，可以用 *vstart.sh* 工具部署一个位于本地的\
伪集群，以便测试。


用法
====

用此工具可在本机部署伪集群以便开发，它可以启动 rgw 、 mon 、
osd 、和/或 mds ，不指定的话会启动所有类型。

要启动开发集群，执行此命令： ::

	vstart.sh [OPTIONS]...

要停止集群，可以执行： ::

	./stop.sh


选项
====

.. option:: -i ip_address

    绑定到指定的 *ip_address* （ IP 地址），而不是从主机名解析。

.. option:: -k

    保留旧的配置文件，而非覆盖它们。

.. option:: -l, --localhost

    Use localhost instead of hostanme.

.. option:: -m ip[:port]

    Specifies monitor *ip* address and *port*.

.. option:: -n, --new

    Create a new cluster.

.. option:: -o config

    Add *config* to all sections in the ceph configuration.

.. option:: -r

    启动 radosgw ，端口从 8000 起递增。

.. option:: --nodaemon

    Use ceph-run as wrapper for mon/osd/mds.

.. option:: --smallmds

    Configure mds with small limit cache size.

.. option:: -x

    Enable Cephx (on by default).

.. option:: -X

    Disable Cephx.

.. option:: -d, --debug

    Launch in debug mode

.. option:: --valgrind[_{osd,mds,mon}] 'valgrind_toolname [args...]'

    Launch the osd/mds/mon/all the ceph binaries using valgrind with the specified tool and arguments.

.. option:: --{mon,osd,mds}_num

    设置 mon/osd/mds 守护进程的数量。

.. option:: --bluestore

    指定 bluestore 作为 OSD 的对象存储后端。

.. option:: --memstore

    指定 memstore 作为 OSD 的对象存储后端。

.. option:: --cache <pool>

    给指定存储池配置一个缓存层。


环境变量
========

{OSD,MDS,MON,RGW}

这些环境变量取值的数值表示你想启动的 ceph 例程数量。

例如： ::

	OSD=3 MON=3 RGW=1 vstart.sh


==============================
 在同一机器上部署多套开发集群
==============================

要在同一机器上启动多套 ceph 集群，可用 *mstart.sh* 脚本，它是\
前述 *vstart* 的简单包装。


用法
=====

要启动多套集群，你可以用 mstart 分别部署各集群，它会在不同端\
口上启动各个集群的监视器、 rgw 进程们，这样你就能在同一集群\
内运行多个监视器、 rgw 等进程了。调用方式如下： ::

        mstart.sh <cluster-name> <vstart options>

例如： ::

        ./mstart.sh cluster1 -n -r

关闭集群可以用： ::

        ./mstop.sh <cluster-name>
