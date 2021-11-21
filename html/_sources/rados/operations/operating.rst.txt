.. Operating a Cluster

==========
 操纵集群
==========

.. index:: systemd; operating a cluster

.. Running Ceph with systemd

用 systemd 控制 Ceph
====================
现在，所有支持 systemd 的发行版（ CentOS 7 、 Fedora 、Debian
Jessie 以及更高版、 SUSE ）都用原生的 systemd 文件来管理 ceph
守护进程，不再使用原来的 sysvinit 脚本了。例如： ::

        sudo systemctl start ceph.target       # start all daemons
        sudo systemctl status ceph-osd@12      # check status of osd.12

罗列节点上 Ceph 的 systemd unit 用此命令： ::

        sudo systemctl status ceph\*.service ceph\*.target


启动所有守护进程
----------------

要启动一 Ceph 节点（任何类型）上的所有守护进程，用下列命令： ::

        sudo systemctl start ceph.target


停止所有守护进程
----------------

要停止一 Ceph 节点（任何类型）上的所有守护进程，用下列命令： ::

        sudo systemctl stop ceph\*.service ceph\*.target


按类型启动所有守护进程
----------------------

要启动一节点上的某一类守护进程，用下列命令： ::

        sudo systemctl start ceph-osd.target
        sudo systemctl start ceph-mon.target
        sudo systemctl start ceph-mds.target


按类型停止所有守护进程
----------------------

要停止一节点上的某一类守护进程，用下列命令： ::

        sudo systemctl stop ceph-mon\*.service ceph-mon.target
        sudo systemctl stop ceph-osd\*.service ceph-osd.target
        sudo systemctl stop ceph-mds\*.service ceph-mds.target


启动单个进程
------------

要启动某节点上的某个守护进程例程，用下列命令之一： ::

        sudo systemctl start ceph-osd@{id}
        sudo systemctl start ceph-mon@{hostname}
        sudo systemctl start ceph-mds@{hostname}

例如： ::

        sudo systemctl start ceph-osd@1
        sudo systemctl start ceph-mon@ceph-server
        sudo systemctl start ceph-mds@ceph-server


停止单个进程
------------

要停止某节点上的某个守护进程例程，用下列命令之一： ::

        sudo systemctl stop ceph-osd@{id}
        sudo systemctl stop ceph-mon@{hostname}
        sudo systemctl stop ceph-mds@{hostname}

例如： ::

        sudo systemctl stop ceph-osd@1
        sudo systemctl stop ceph-mon@ceph-server
        sudo systemctl stop ceph-mds@ceph-server


.. index:: sysvinit; operating a cluster

用 sysvinit 运行 Ceph
=====================
.. Running Ceph

每次用命令\ **启动**\ 、\ **重启**\ 、\ **停止** Ceph 守护进程\
（或整个集群）时，你必须指定至少一个选项和一个命令，还可能要指\
定守护进程类型或具体例程。 ::

        {commandline} [options] [commands] [daemons]


``ceph`` 命令的选项包括：

+-----------------+----------+-------------------------------------------------+
| 选项            | 简写     | 描述                                            |
+=================+==========+=================================================+
| ``--verbose``   |  ``-v``  | 详细的日志。                                    |
+-----------------+----------+-------------------------------------------------+
| ``--valgrind``  | ``N/A``  | （只适合开发者和质检人员）用 `Valgrind`_ 调试。 |
+-----------------+----------+-------------------------------------------------+
| ``--allhosts``  |  ``-a``  | 在 ``ceph.conf`` 里配置的所有主机上执行，否     |
|                 |          | 则它只在本机执行。                              |
+-----------------+----------+-------------------------------------------------+
| ``--restart``   | ``N/A``  | 核心转储后自动重启。                            |
+-----------------+----------+-------------------------------------------------+
| ``--norestart`` | ``N/A``  | 核心转储后不自动重启。                          |
+-----------------+----------+-------------------------------------------------+
| ``--conf``      |  ``-c``  | 使用另外一个配置文件。                          |
+-----------------+----------+-------------------------------------------------+

Ceph 子命令包括：

+------------------+-----------------------------------------------------------+
| 命令             | 描述                                                      |
+==================+===========================================================+
|    ``start``     | 启动守护进程。                                            |
+------------------+-----------------------------------------------------------+
|    ``stop``      | 停止守护进程。                                            |
+------------------+-----------------------------------------------------------+
|  ``forcestop``   | 暴力停止守护进程，等价于 ``kill -9``                      |
+------------------+-----------------------------------------------------------+
|   ``killall``    | 杀死某一类守护进程。                                      |
+------------------+-----------------------------------------------------------+
|  ``cleanlogs``   | 清理掉日志目录。                                          |
+------------------+-----------------------------------------------------------+
| ``cleanalllogs`` | 清理掉日志目录内的\ **所有**\ 文件。                      |
+------------------+-----------------------------------------------------------+

至于子系统操作， ``ceph`` 服务能指定守护进程类型，在
``[daemons]`` 处指定守护进程类型就行了，守护进程类型包括：

- ``mon``
- ``osd``
- ``mds``


.. _Valgrind: http://www.valgrind.org/
.. _initctl: http://manpages.ubuntu.com/manpages/raring/en/man8/initctl.8.html
