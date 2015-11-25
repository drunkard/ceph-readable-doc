================
 开发集群的部署
================

要从事 ceph 开发，可以用 *vstart.sh* 工具部署一个位于本地的伪集群，以便\
测试。


用法
====

用此工具可在本机部署伪集群以便开发，它可以启动 rgw 、 mon 、 osd 、和/或 \
mds ，不指定的话会启动所有类型。

要启动开发集群，执行此命令： ::

	vstart.sh [OPTIONS]... [mon] [osd] [mds]

要停止集群，可以执行： ::

	./stop.sh


选项
====

.. option:: -i ip_address

    Bind to the specified *ip_address* instead of guessing and resolve from hostname.

.. option:: -k

    Keep old configuration files instead of overwritting theses.

.. option:: -l, --localhost

    Use localhost instead of hostanme.

.. option:: -m ip[:port]

    Specifies monitor *ip* address and *port*.

.. option:: -n, --new

    Create a new cluster.

.. option:: -o config

    Add *config* to all sections in the ceph configuration.

.. option:: -r

    Start radosgw (ceph needs to be compiled with --radosgw), create an apache2 configuration file, and start apache2 with it (needs apache2 with mod_fastcgi) on port starting from 8000.

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

Environment variables
=====================

{OSD,MDS,MON,RGW}

Theses environment variables will contains the number of instances of the desired ceph process you want to start.

Example: ::

	OSD=3 MON=3 RGW=1 vstart.sh
