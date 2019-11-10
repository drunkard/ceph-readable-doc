================
 开发集群的部署
================

要从事 ceph 开发，可以用 *vstart.sh* 工具部署一个位于本地的\
伪集群，以便测试。


.. Usage

用法
====
用此工具可在本机部署伪集群以便开发，它可以启动 rgw 、 mon 、
osd 、和/或 mds ，不指定的话会启动所有类型。

要启动开发集群，执行此命令： ::

	vstart.sh [OPTIONS]...

要停止集群，可以执行： ::

	./stop.sh


.. Options

选项
====

.. option:: --bluestore

    指定 bluestore 作为 OSD 的对象存储后端。

.. option:: --cache <pool>

    给指定存储池配置一个缓存层。

.. option:: -d, --debug

    在调试模式下启动。

.. option:: -e

    创建一个纠删码存储池。

.. option:: -f, --filestore

    指定 filestore 作为 OSD 的对象存储后端。

.. option:: --hitset <pool> <hit_set_type>

    启用 hitset 追踪。

.. option:: -i ip_address

    绑定到指定的 *ip_address* （ IP 地址），而不是从主机名解析。

.. option:: -k

    保留旧的配置文件，而非覆盖它们。

.. option:: -K, --kstore

    指定 kstore 作为 OSD 的对象存储后端。

.. option:: -l, --localhost

    用 localhost 作为主机名。

.. option:: -m ip[:port]

    指定监视器的 *ip* 地址和端口 *port* 。

.. option:: --memstore

    指定 memstore 作为 OSD 的对象存储后端。

.. option:: --multimds <count>

    启用多 MDS 功能、且指定最大活跃数。

.. option:: -n, --new

    创建一个新集群。

.. option:: -N, --not-new

    重用现有的集群配置（默认行为）。

.. option:: --nodaemon

    用 ceph-run 作为 mon/osd/mds 的包装。

.. option:: --nolockdep

    禁用 lockdep 。

.. option:: -o config

    把配置 *config* 加进 ceph 配置的所有段落下。

.. option:: --rgw_port <port>

    指定 ceph rgw 的 HTTP 监听端口。

.. option:: --rgw_frontend <frontend>

    指定 rgw 前端（默认是 civetweb ）。

.. option:: --rgw_compression <compression_type>

    指定 rgw 压缩插件（默认是禁用的）。

.. option:: --smallmds

    给 MDS 配置小缓存尺寸。

.. option:: --short

    只能用短对象名，对 ext4 开发版有必要指定。

.. option:: --valgrind[_{osd,mds,mon}] 'valgrind_toolname [args...]'

    在 valgrind 环境下、用指定的工具和参数，启动 ceph 的 \
    osd/mds/mon/all 二进制。

.. option:: --without-dashboard

    不要用 mgr 的仪表盘运行。

.. option:: -x

    启用 Cephx （默认开启）。

.. option:: -X

    禁用 Cephx 。


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

        ./mstart.sh cluster1 -n

关闭集群可以用： ::

        ./mstop.sh <cluster-name>
