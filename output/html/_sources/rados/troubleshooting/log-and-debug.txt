================
 日志记录和调试
================

一般来说，你应该在运行时增加调试选项来调试问题；也可以把调试选项添加到 Ceph 配置文\
件里来调试启动问题，然后查看 ``/var/log/ceph`` （默认位置）下的日志文件。

.. tip:: 调试输出会拖慢系统，这种延时有可能掩盖竞争条件。

日志记录是资源密集任务。如果你碰到的问题在集群的某个特定区域，只启用那个区域对应的日\
志功能即可。例如，你的 OSD 运行良好、元数据服务器却不行，这时应该先打开那个可疑元数\
据服务器例程的调试日志；如果不行再打开各子系统的日志。

.. important:: 详尽的日志每小时可能超过 1GB ，如果你的系统盘满了，这个节点就会停止\
   工作。

如果你要打开或增加 Ceph 日志级别，确保系统盘空间足够。滚动日志文件的方法见\ \
`加快日志更迭`_\ 。集群稳定运行后，可以关闭不必要的调试选项以更好地运行。在运营中记\
录调试输出会拖慢系统、且浪费资源。

可用选项参见\ `子系统、日志和调试选项`_\ 。


运行时
======

如果你想查看一进程的运行时配置，必须先登录对应主机，然后执行命令：
::

	ceph --admin-daemon {/path/to/admin/socket} config show | less
	ceph --admin-daemon /var/run/ceph/ceph-osd.0.asok config show | less

要在运行时激活 Ceph 的调试输出（即 ``dout()`` ），用 ``ceph tell`` 命令把参数注入\
运行时配置：
::

	ceph tell {daemon-type}.{daemon id or *} injectargs --{name} {value} [--{name} {value}]

用 ``osd`` 、 ``mon`` 或 ``mds`` 替代 ``{daemon-type}`` 。你可以用星号（ ``*`` ）\
把配置应用到同类型的所有守护进程，或者指定具体守护进程的标识号（即其名字或字母）。例\
如，要给名为 ``ods.0`` 的 ``ceph-osd`` 守护进程提高调试级别，用下列命令：
::

	ceph tell osd.0 injectargs --debug-osd 0/5

``ceph tell`` 命令会贯穿所有监视器。如果你不能绑定监视器，还可以登录你要改的那台主\
机用 ``ceph --admin-daemon`` 来更改。例如：
::

	sudo ceph --admin-daemon /var/run/ceph/ceph-osd.0.asok config set debug_osd 0/5

可用选项参见\ `子系统、日志和调试选项`_\ 。


启动时
======

要在启动时激活调试输出（\ *即* ``dout()`` ），你得把选项加入配置文件。各进程共有配\
置可写在配置文件的 ``[global]`` 下，某类进程的配置可写在守护进程段下（\ *如* \
``[mon]`` 、 ``[osd]`` 、 ``[mds]`` ）。例如：
::

	[global]
		debug ms = 1/5

	[mon]
		debug mon = 20
		debug paxos = 1/5
		debug auth = 2
		 
 	[osd]
 		debug osd = 1/5
 		debug filestore = 1/5
 		debug journal = 1
 		debug monc = 5/20
 
	[mds]
		debug mds = 1
		debug mds balancer = 1
		debug mds log = 1
		debug mds migrator = 1


可用选项参见\ `子系统、日志和调试选项`_\ 。


加快日志更迭
============

如果你的系统盘比较满，可以修改 ``/etc/logrotate.d/ceph`` 内的日志滚动配置以加快滚\
动。在滚动频率后增加一个尺寸选项（达到此尺寸就滚动）来加快滚动（通过 cronjob ）。例\
如默认配置大致如此：
::

	rotate 7
  	weekly
  	compress
  	sharedscripts

增加一个 ``size`` 选项。
::

	rotate 7
	weekly
	size 500M
	compress
	sharedscripts

然后，打开 crontab 编辑器。
::
   
  	crontab -e

最后，增加一条用以检查 ``/etc/logrorate.d/ceph`` 文件。
::

  	30 * * * * /usr/sbin/logrotate /etc/logrotate.d/ceph >/dev/null 2>&1

本例中每 30 分钟检查一次 ``/etc/logrorate.d/ceph`` 文件。


Valgrind
========

你也许还得追踪内存和线程问题，可以在 Valgrind 中运行一个守护进程、一类进程、或整个\
集群。 Valgrind 是计算密集型程序，应该只用于开发或调试，否则会拖慢系统。其消息记录\
到 ``stderr`` 。


子系统、日志和调试选项
======================

大多数情况下你可以通过子系统打开调试。

Ceph 子系统概览
---------------

各子系统都有日志级别用于分别控制其输出日志、和暂存日志，你可以分别为这些子系统设置不\
同的记录级别。 Ceph 的日志级别从 ``1`` 到 ``20`` ， ``1`` 是简洁、 ``20`` 是详尽。

调试选项允许用单个数字同时设置日志级别和内存级别，会设置为一样。比如，如果你指定 \
``debug ms = 5`` ， Ceph 会把日志级别和内存级别都设置为 ``5`` 。也可以分别设置，第\
一个选项是日志级别、后一个是内存级别，二者必须用斜线（ ``/`` ）分隔。假如你想把 \
``ms`` 子系统的调试日志级别设为 ``1`` 、内存级别设为 ``5`` ，可以写为 \
``debug ms = 1/5`` ，如下：

.. code-block:: ini 

	debug {subsystem} = {log-level}/{memory-level}
	#for example
	debug mds log = 1/20


下表列出了 Ceph 子系统及其默认日志和内存级别。一旦你完成调试，应该恢复默认值、或一\
个适合平常运营的级别。


+--------------------+-----------+--------------+
| 子系统             | 日志级别  | 内存日志级别 |
+====================+===========+==============+
| ``default``        |     0     |      5       |
+--------------------+-----------+--------------+
| ``lockdep``        |     0     |      5       |
+--------------------+-----------+--------------+
| ``context``        |     0     |      5       |
+--------------------+-----------+--------------+
| ``crush``          |     1     |      5       |
+--------------------+-----------+--------------+
| ``mds``            |     1     |      5       |
+--------------------+-----------+--------------+
| ``mds balancer``   |     1     |      5       |
+--------------------+-----------+--------------+
| ``mds locker``     |     1     |      5       |
+--------------------+-----------+--------------+
| ``mds log``        |     1     |      5       |
+--------------------+-----------+--------------+
| ``mds log expire`` |     1     |      5       |
+--------------------+-----------+--------------+
| ``mds migrator``   |     1     |      5       |
+--------------------+-----------+--------------+
| ``buffer``         |     0     |      0       |
+--------------------+-----------+--------------+
| ``timer``          |     0     |      5       |
+--------------------+-----------+--------------+
| ``filer``          |     0     |      5       |
+--------------------+-----------+--------------+
| ``objecter``       |     0     |      0       |
+--------------------+-----------+--------------+
| ``rados``          |     0     |      5       |
+--------------------+-----------+--------------+
| ``rbd``            |     0     |      5       |
+--------------------+-----------+--------------+
| ``journaler``      |     0     |      5       |
+--------------------+-----------+--------------+
| ``objectcacher``   |     0     |      5       |
+--------------------+-----------+--------------+
| ``client``         |     0     |      5       |
+--------------------+-----------+--------------+
| ``osd``            |     0     |      5       |
+--------------------+-----------+--------------+
| ``optracker``      |     0     |      5       |
+--------------------+-----------+--------------+
| ``objclass``       |     0     |      5       |
+--------------------+-----------+--------------+
| ``filestore``      |     1     |      5       |
+--------------------+-----------+--------------+
| ``journal``        |     1     |      5       |
+--------------------+-----------+--------------+
| ``ms``             |     0     |      5       |
+--------------------+-----------+--------------+
| ``mon``            |     1     |      5       |
+--------------------+-----------+--------------+
| ``monc``           |     0     |      5       |
+--------------------+-----------+--------------+
| ``paxos``          |     0     |      5       |
+--------------------+-----------+--------------+
| ``tp``             |     0     |      5       |
+--------------------+-----------+--------------+
| ``auth``           |     1     |      5       |
+--------------------+-----------+--------------+
| ``finisher``       |     1     |      5       |
+--------------------+-----------+--------------+
| ``heartbeatmap``   |     1     |      5       |
+--------------------+-----------+--------------+
| ``perfcounter``    |     1     |      5       |
+--------------------+-----------+--------------+
| ``rgw``            |     1     |      5       |
+--------------------+-----------+--------------+
| ``javaclient``     |     1     |      5       |
+--------------------+-----------+--------------+
| ``asok``           |     1     |      5       |
+--------------------+-----------+--------------+
| ``throttle``       |     1     |      5       |
+--------------------+-----------+--------------+


日志记录选项
------------

日志和调试选项不是必需配置，但你可以按需覆盖默认值。 Ceph 支持如下配置：


``log file``

:Description: 集群日志文件的位置。
:Type: String
:Required: No
:Default: ``/var/log/ceph/$cluster-$name.log``


``log max new``

:Description: 新日志文件的最大数量。
:Type: Integer
:Required: No
:Default: ``1000``


``log max recent``

:Description: 一个日志文件包含的最新事件的最大数量。
:Type: Integer
:Required:  No
:Default: ``1000000``


``log to stderr``

:Description: 设置日志消息是否输出到标准错误（ ``stderr`` ）。
:Type: Boolean
:Required: No
:Default: ``true``


``err to stderr``

:Description: 设置错误消息是否输出到标准错误（ ``stderr`` ）。
:Type: Boolean
:Required: No
:Default: ``true``


``log to syslog``

:Description: 设置日志消息是否输出到 ``syslog`` 。
:Type: Boolean
:Required: No
:Default: ``false``


``err to syslog``

:Description: 设置错误消息是否输出到 ``syslog`` 。
:Type: Boolean
:Required: No
:Default: ``false``


``log flush on exit``

:Description: 设置 Ceph 退出后是否回写日志文件。
:Type: Boolean
:Required: No
:Default: ``true``


``clog to monitors``

:Description: 设置是否把 ``clog`` 消息发送给监视器。
:Type: Boolean
:Required: No
:Default: ``true``


``clog to syslog``

:Description: 设置是否把 ``clog`` 输出到 syslog 。
:Type: Boolean
:Required: No
:Default: ``false``


``mon cluster log to syslog``

:Description: 设置集群日志是否输出到 syslog 。
:Type: Boolean
:Required: No
:Default: ``false``


``mon cluster log file``

:Description: 集群日志位置。
:Type: String
:Required: No
:Default: ``/var/log/ceph/$cluster.log``



OSD
---

``osd debug drop ping probability``

:Description: ?
:Type: Double
:Required: No
:Default: 0


``osd debug drop ping duration``

:Description: 
:Type: Integer
:Required: No
:Default: 0


``osd debug drop pg create probability``

:Description: 
:Type: Integer
:Required: No
:Default: 0


``osd debug drop pg create duration``

:Description: ?
:Type: Double
:Required: No
:Default: 1


``osd preserve trimmed log``

:Description: 裁减后保留剩余日志。
:Type: Boolean
:Required: No
:Default: ``false``


``osd tmapput sets uses tmap``

:Description: 使用 ``tmap`` ，仅用于调试。
:Type: Boolean
:Required: No
:Default: ``false``


``osd min pg log entries``

:Description: 归置组日志最小条数。
:Type: 32-bit Unsigned Integer
:Required: No
:Default: 1000


``osd op log threshold``

:Description: 一次发送多少操作日志消息。
:Type: Integer
:Required: No
:Default: 5



Filestore
---------

``filestore debug omap check``

:Description: 调试同步检查，这是昂贵的操作。
:Type: Boolean
:Required: No
:Default: 0


MDS
---

``mds debug scatterstat``

:Description: Ceph 将把各种回归状态常量设置为真（谨为开发者）。
:Type: Boolean
:Required: No
:Default: ``false``


``mds debug frag``

:Description: Ceph 将在方便时校验目录碎片（谨为开发者）。
:Type: Boolean
:Required: No
:Default: ``false``


``mds debug auth pins``

:Description: debug auth pin 开关（谨为开发者）。
:Type: Boolean
:Required: No
:Default: ``false``


``mds debug subtrees``

:Description: debug subtree 开关（谨为开发者）。
:Type: Boolean
:Required: No
:Default: ``false``


RADOS 网关
----------

``rgw log nonexistent bucket``

:Description: 记录不存在的桶？
:Type: Boolean
:Required: No
:Default: ``false``


``rgw log object name``

:Description: 是否记录对象名称。注：关于格式参考 ``man date`` ，子集也支持。
:Type: String
:Required: No
:Default: ``%Y-%m-%d-%H-%i-%n``


``rgw log object name utc``

:Description: 对象日志名称包含 UTC ？
:Type: Boolean
:Required: No
:Default: ``false``


``rgw enable ops log``

:Description: 允许记录 RGW 的每一个操作。
:Type: Boolean
:Required: No
:Default: ``true``


``rgw enable usage log``

:Description: 允许记录 RGW 的带宽使用。
:Type: Boolean
:Required: No
:Default: ``true``


``rgw usage log flush threshold``

:Description: 回写未决的日志数据阀值。
:Type: Integer
:Required: No
:Default: ``1024``


``rgw usage log tick interval``

:Description: 每隔 ``s`` 回写一次未决日志。
:Type: Integer
:Required: No
:Default: 30


``rgw intent log object name``

:Description: 
:Type: String
:Required: No
:Default: ``%Y-%m-%d-%i-%n``


``rgw intent log object name utc``

:Description: 日志对象名字里包含 UTC 时间戳。
:Type: Boolean
:Required: No
:Default: ``false``
