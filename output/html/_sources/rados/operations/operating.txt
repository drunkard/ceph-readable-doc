==========
 操纵集群
==========

.. index:: Upstart; operating a cluster

用 upstart 控制 Ceph
====================

用 ``ceph-deploy`` 把 Ceph Cuttlefish 及更高版部署到 Ubuntu 之后，你可以用基于事\
件的 `Upstart`_ 来启动、关闭 :term:`Ceph 节点`\ 上的守护进程。 Upstart 不要求你在\
配置文件里定义守护进程例程。

用下列命令列出 Ceph 作业和例程： ::

	sudo initctl list | grep ceph

详情参见 `initctl`_ 。


启动所有守护进程
----------------

要启动一 Ceph 节点（任何类型）上的所有守护进程，用下列命令： ::

	sudo start ceph-all


停止所有守护进程
----------------

要停止一 Ceph 节点（任何类型）上的所有守护进程，用下列命令： ::

	sudo stop ceph-all


按类型启动所有守护进程
----------------------

要启动一节点上的某一类守护进程，用下列命令： ::

	sudo start ceph-osd-all
	sudo start ceph-mon-all
	sudo start ceph-mds-all


按类型停止所有守护进程
----------------------

要停止一节点上的某一类守护进程，用下列命令：

	sudo stop ceph-osd-all
	sudo stop ceph-mon-all
	sudo stop ceph-mds-all


启动单个进程
------------

要启动某节点上一指定守护进程例程，用下列命令之一： ::

	sudo start ceph-osd id={id}
	sudo start ceph-mon id={hostname}
	sudo start ceph-mds id={hostname}

例如： ::

	sudo start ceph-osd id=1
	sudo start ceph-mon id=ceph-server
	sudo start ceph-mds id=ceph-server


停止单个进程
------------

要停止某节点上一指定守护进程例程，用下列命令之一： ::

	sudo stop ceph-osd id={id}
	sudo stop ceph-mon id={hostname}
	sudo stop ceph-mds id={hostname}

例如： ::

	sudo stop ceph-osd id=1
	sudo start ceph-mon id=ceph-server
	sudo start ceph-mds id=ceph-server


.. index:: Ceph service; sysvinit; operating a cluster


运行 Ceph
=========

每次用命令\ **启动**\ 、\ **重启**\ 、\ **停止**\ Ceph 守护进程（或整个集群）时，\
你必须指定至少一个选项和一个命令，还可能要指定守护进程类型或具体例程。 ::

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

至于子系统操作， ``ceph`` 服务能指定守护进程类型，在 ``[daemons]`` 处指定守护进程\
类型就行了，守护进程类型包括：

- ``mon``
- ``osd``
- ``mds``



通过 sysvinit 机制运行 Ceph
---------------------------

在 CentOS 、 Redhat 、 Fedora 和 SLES 发行版上可以通过传统的 ``sysvinit`` 运行 \
Ceph ， Debian/Ubuntu 的较老的版本也可以用此方法。


启动所有守护进程
~~~~~~~~~~~~~~~~

要启动 Ceph 集群，执行 ``ceph`` 时加上 ``start`` 命令，语法如下： ::

	sudo /etc/init.d/ceph [options] [start|restart] [daemonType|daemonID]

下面是个典型实例： ::

	sudo /etc/init.d/ceph -a start

加 ``-a`` （即在所有节点上执行）执行完成后 Ceph 应该开始运行了。


停止所有守护进程
~~~~~~~~~~~~~~~~

要停止 Ceph 集群，执行 ``ceph`` 时加上 ``stop`` 命令，语法如下： ::

	sudo /etc/init.d/ceph [options] stop [daemonType|daemonID]

下面是个典型实例： ::

	sudo /etc/init.d/ceph -a stop

执行命令时一旦加了 ``-a`` （即在所有节点执行），整个 Ceph 集群都会关闭。


启动一类守护进程
~~~~~~~~~~~~~~~~

要启动本节点上某一类的所有 Ceph 守护进程，按此语法： ::

	sudo /etc/init.d/ceph start {daemon-type}
	sudo /etc/init.d/ceph start osd

要启动非本机节点上某一类的所有 Ceph 守护进程，按此语法： ::

	sudo /etc/init.d/ceph -a start {daemon-type}
	sudo /etc/init.d/ceph -a start osd


停止一类守护进程
~~~~~~~~~~~~~~~~

要停止本节点上某一类的所有 Ceph 守护进程，按此语法：

	sudo /etc/init.d/ceph stop {daemon-type}
	sudo /etc/init.d/ceph stop osd

要启动非本机节点上某一类的所有 Ceph 守护进程，按此语法： ::

	sudo /etc/init.d/ceph -a stop {daemon-type}
	sudo /etc/init.d/ceph -a stop osd


启动单个守护进程
~~~~~~~~~~~~~~~~

要启动本节点上某个 Ceph 守护进程，按此语法： ::

	sudo /etc/init.d/ceph start {daemon-type}.{instance}
	sudo /etc/init.d/ceph start osd.0

要启动另一节点上某个 Ceph 守护进程，按此语法： ::

	sudo /etc/init.d/ceph -a start {daemon-type}.{instance}
	sudo /etc/init.d/ceph -a start osd.0


停止单个守护进程
~~~~~~~~~~~~~~~~

要停止本节点上某个 Ceph 守护进程，按此语法： ::

	sudo /etc/init.d/ceph stop {daemon-type}.{instance}
	sudo /etc/init.d/ceph stop osd.0

要停止另一节点上某个 Ceph 守护进程，按此语法： ::

	sudo /etc/init.d/ceph -a stop {daemon-type}.{instance}
	sudo /etc/init.d/ceph -a stop osd.0


把 Ceph 当服务运行
------------------

如果你是用 ``ceph-deploy`` 部署 Argonaut 或 Bobtail 的，那么 Ceph 可以作为服务运\
行（还可以用 sysvinit ）。


启动所有守护进程
~~~~~~~~~~~~~~~~

要启动你的 Ceph 集群，执行 ``ceph`` 时加上 ``start`` 命令，按此语法： ::

	sudo service ceph [options] [start|restart] [daemonType|daemonID]

下面是个典型实例： ::

	sudo service ceph -a start

加 ``-a`` （即在所有节点上执行）执行完成后 Ceph 应该开始运行了。


停止所有守护进程
~~~~~~~~~~~~~~~~

要停止你的 Ceph 集群，执行 ``ceph`` 时加上 ``stop`` 命令，按此语法： ::

	sudo service ceph [options] stop [daemonType|daemonID]

例如： ::

	sudo service ceph -a stop

加 ``-a`` （即在所有节点上执行）执行完成后 Ceph 应该已停止。


启动一类守护进程
~~~~~~~~~~~~~~~~

要启动本节点上某一类的所有 Ceph 守护进程，按此语法： ::

	sudo service ceph start {daemon-type}
	sudo service ceph start osd

要启动所有节点上某一类的所有 Ceph 守护进程，按此语法： ::

	sudo service ceph -a start {daemon-type}
	sudo service ceph -a start osd


停止一类守护进程
~~~~~~~~~~~~~~~~

要停止本节点上某一类的所有 Ceph 守护进程，按此语法： ::

	sudo service ceph stop {daemon-type}
	sudo service ceph stop osd

要停止所有节点上某一类的所有 Ceph 守护进程，按此语法： ::

	sudo service ceph -a stop {daemon-type}
	sudo service ceph -a stop osd


启动单个守护进程
~~~~~~~~~~~~~~~~

要启动本节点上某个 Ceph 守护进程，按此语法： ::

	sudo service ceph start {daemon-type}.{instance}
	sudo service ceph start osd.0

要启动另一节点上某个 Ceph 守护进程，按此语法： ::

	sudo service ceph -a start {daemon-type}.{instance}
	sudo service ceph -a start osd.0


停止单个守护进程
~~~~~~~~~~~~~~~~

要停止本节点上某个 Ceph 守护进程，按此语法： ::

	sudo service ceph stop {daemon-type}.{instance}
	sudo service ceph stop osd.0

要停止另一节点上某个 Ceph 守护进程，按此语法： ::

	sudo service ceph -a stop {daemon-type}.{instance}
	sudo service ceph -a stop osd.0




.. _Valgrind: http://www.valgrind.org/
.. _Upstart: http://upstart.ubuntu.com/index.html
.. _initctl: http://manpages.ubuntu.com/manpages/raring/en/man8/initctl.8.html
