===========
 配置 Ceph
===========

你启动 Ceph 服务时，初始化进程会把一系列守护进程放到后台运行。 \
:term:`Ceph 存储集群`\ 运行两种守护进程：

- :term:`Ceph 监视器` （ ``ceph-mon`` ）
- :term:`Ceph OSD 守护进程` （ ``ceph-osd`` ）

要支持 :term:`Ceph 文件系统`\ 功能，它还需要运行至少一个 \
:term:`Ceph 元数据服务器`\ （ ``ceph-mds`` ）；支持 :term:`Ceph 对象存储`\ 功能\
的集群还要运行网关守护进程（ ``radosgw`` ）。为方便起见，各类守护进程都有一系列默认\
值（很多由 ``ceph/src/common/config_opts.h`` 配置），你可以用 Ceph 配置文件覆盖这\
些默认值。


.. _ceph-conf-file:

配置文件
========

启动 Ceph 存储集群时，各守护进程都从同一个配置文件（即默认的 ``ceph.conf`` ）里查\
找它自己的配置。手动部署时，你需要创建此配置文件；用部署工具（\ *如* \
``ceph-deploy`` 、 ``chef`` 等）创建配置文件时可以参考下面的信息。配置文件定义了：

- 集群身份
- 认证配置
- 集群成员
- 主机名
- 主机 IP 地址
- 密钥环路径
- 日志路径
- 数据路径
- 其它运行时选项

默认的 Ceph 配置文件位置相继排列如下：

#. ``$CEPH_CONF`` （\ *就是* ``$CEPH_CONF`` 环境变量所指示的路径）；
#. ``-c path/path``  （\ *就是* ``-c`` 命令行参数）；
#. ``/etc/ceph/ceph.conf``
#. ``~/.ceph/config``
#. ``./ceph.conf`` （\ *就是*\ 当前所在的工作路径。


Ceph 配置文件使用 *ini* 风格的语法，以分号 (;) 和井号 (#) 开始的行是注释，如下：

.. code-block:: ini

	# <--A number (#) sign precedes a comment.
	; A comment may be anything.
	# Comments always follow a semi-colon (;) or a pound (#) on each line.
	# The end of the line terminates a comment.
	# We recommend that you provide comments in your configuration file(s).


.. _ceph-conf-settings:

配置段落
========

Ceph 配置文件可用于配置存储集群内的所有守护进程、或者某一类型的所有守护进程。要配置\
一系列守护进程，这些配置必须位于能收到配置的段落之下，比如：


``[global]``

:描述: ``[global]`` 下的配置影响 Ceph 集群里的所有守护进程。
:实例: ``auth supported = cephx``


``[osd]``

:描述: ``[osd]`` 下的配置影响存储集群里的所有 ``ceph-osd`` 进程，并且会覆盖 \
       ``[global]`` 下的同一选项。

:实例: ``osd journal size = 1000``


``[mon]``

:描述: ``[mon]`` 下的配置影响集群里的所有 ``ceph-mon`` 进程，并且会覆盖 \
       ``[global]`` 下的同一选项。

:实例: ``mon addr = 10.0.0.101:6789``


``[mds]``

:描述: ``[mds]`` 下的配置影响集群里的所有 ``ceph-mds`` 进程，并且会覆盖 \
       ``[global]`` 下的同一选项。

:实例: ``host = myserver01``


``[client]``

:描述: ``[client]`` 下的配置影响所有客户端（如挂载的 Ceph 文件系统、挂载的\
       块设备等等）。

:实例: ``log file = /var/log/ceph/radosgw.log``


全局设置影响集群内所有守护进程的例程，所以 ``[global]`` 可用于设置适用所有守护进程\
的选项。但可以用这些覆盖 ``[global]`` 设置：

#. 在 ``[osd]`` 、 ``[mon]`` 、 ``[mds]`` 下更改某一类进程的配置。

#. 更改特定进程的设置，如 ``[osd.1]`` 。

覆盖全局设置会影响所有子进程，明确剔除的例外。

典型的全局设置包括激活认证，例如：

.. code-block:: ini

	[global]
	#Enable authentication between hosts within the cluster.
	#v 0.54 and earlier
	auth supported = cephx

	#v 0.55 and after
	auth cluster required = cephx
	auth service required = cephx
	auth client required = cephx


你可以统一配置一类守护进程。配置写到 ``[osd]`` 、 ``[mon]`` 、 ``[mds]`` 下时，无\
须再指定某个特定例程，即可分别影响所有 OSD 、监视器、元数据进程。

典型的类范畴配置包括日志尺寸、 filestore 选项等，如：

.. code-block:: ini

	[osd]
	osd journal size = 1000


你也可以配置某个特定例程。一个例程由类型和及其例程 ID 确定， OSD 的例程 ID 只能是\
数字，但监视器和元数据服务器的 ID 可包含字母和数字。

.. code-block:: ini

	[osd.1]
	# settings affect osd.1 only.

	[mon.a]
	# settings affect mon.a only.

	[mds.b]
	# settings affect mds.b only.


如果你想配置某个 Ceph 网关客户端，可以用点（ . ）分隔的守护进程和例程来指定，例如：

.. code-block:: ini

	[client.radosgw.instance-name]
	# settings affect client.radosgw.instance-name only.



.. _ceph-metavariables:

元变量
======

元变量大大简化了集群配置。 Ceph 会把配置的元变量展开为具体值；元变量功能很强大，可\
以用在配置文件的 ``[global]`` 、 ``[osd]`` 、 ``[mon]`` 、 ``[mds]`` 段里，类似\
于 Bash 的 shell 扩展。

Ceph 支持下列元变量：


``$cluster``

:描述: 展开为存储集群名字，在同一套硬件上运行多个集群时有用。
:实例: ``/etc/ceph/$cluster.keyring``
:默认值: ``ceph``


``$type``

:描述: 可展开为 ``mds`` 、 ``osd`` 、 ``mon`` 中的一个，有赖于当前守护进程\
       的类型。

:实例: ``/var/lib/ceph/$type``


``$id``

:描述: 展开为守护进程标识符； ``osd.0`` 应为 ``0`` ， ``mds.a`` 是 ``a`` 。
:实例: ``/var/lib/ceph/$type/$cluster-$id``


``$host``

:描述: 展开为当前守护进程的主机名。


``$name``

:描述: 展开为 ``$type.$id`` 。
:实例: ``/var/run/ceph/$cluster-$name.asok``


.. _ceph-conf-common-settings:

共有选项
========

`硬件推荐`_\ 段提供了一些配置 Ceph 存储集群的硬件指导。一个 :term:`Ceph 节点`\ 可\
以运行多个进程，例如一个节点有多个硬盘，可以为每个硬盘配置一个 ``ceph-osd`` 守护进\
程。理想情况下一台主机应该只运行一类进程，例如：一台主机运行着 ``ceph-osd`` 进程，\
另一台主机运行着 ``ceph-mds`` 进程， ``ceph-mon`` 进程又在另外一台主机上。

各节点都用 ``host`` 选项指定主机名字，监视器还需要用 ``addr`` 选项指定网络地址和端\
口（即域名或 IP 地址）。基本配置文件可以只指定最小配置。例如：

.. code-block:: ini

	[global]
	mon_initial_members = ceph1
	mon_host = 10.0.0.1


.. important:: ``host`` 选项是此节点的短名字，不是全资域名（ FQDN ），也\ \
   **不是** IP 地址；执行 ``hostname -s`` 即可得到短名字。不要给初始监视器之外的例\
   程设置 ``host`` ，除非你想手动部署；\ **一定不能**\ 用于 ``chef`` 或 \
   ``ceph-deploy`` ，这些工具会自动获取正确结果。


.. _ceph-network-config:

网络
====

关于 Ceph 网络配置的讨论见\ `网络配置参考`_\ 。


监视器
======

典型的 Ceph 生产集群至少部署 3 个\ :term:`监视器`\ 来确保高可靠性，它允许一个\
监视器例程崩溃。奇数个监视器（ 3 个）确保 PAXOS 算法能确定一批监视器里哪个版本的\ \
:term:`集群运行图`\ 是最新的。

.. note:: 一个 Ceph 集群可以只有一个监视器，但是如果它失败了，因没有监视器数据服务\
   就会中断。

Ceph 监视器默认监听 ``6789`` 端口，例如：

.. code-block:: ini

	[mon.a]
	host = hostName
	mon addr = 150.140.130.120:6789

默认情况下， Ceph 会在下面的路径存储监视器数据： ::

	/var/lib/ceph/mon/$cluster-$id

你必须手动或通过部署工具（如 ``ceph-deploy`` ）创建对应目录。前述元变量必须先全部展\
开，名为 “ceph” 的集群将展开为： ::

	/var/lib/ceph/mon/ceph-a

其它细节见\ `监视器配置参考`_\ 。

.. _监视器配置参考: ../mon-config-ref


.. _ceph-osd-config:


认证
====

.. versionadded:: Bobtail 0.56

对于 v0.56 及后来版本，要在配置文件的 ``[global]`` 中明确启用或禁用认证。 ::

	auth cluster required = cephx
	auth service required = cephx
	auth client required = cephx

另外，你应该启用消息签名，详情见 `Cephx 配置参考`_\ 和 `Cephx 认证`_\ 。

.. important:: 我们建议，升级时先明确地关闭认证，再进行升级。等升级完成后再重新启用\
   认证。

.. _Cephx 认证: ../../operations/authentication
.. _Cephx 配置参考: ../auth-config-ref


.. _ceph-monitor-config:


OSDs
====

通常， Ceph 生产集群在一个节点上只运行一个 :term:`Ceph OSD 守护进程`\ ，\
此守护进程在一个存储驱动器上只运行一个 filestore ；典型部署需指定日志\
尺寸。例如：

.. code-block:: ini

	[osd]
	osd journal size = 10000

	[osd.0]
	host = {hostname} #manual deployments only.


默认情况下， Ceph 认为你把 OSD 数据存储到了以下路径： ::

	/var/lib/ceph/osd/$cluster-$id

你必须手动或通过部署工具（如 ``ceph-deploy`` ）创建对应目录，名为 “ceph” 的集群其\
元变量完全展开后，前述的目录将是： ::

	/var/lib/ceph/osd/ceph-0

你可以用 ``osd data`` 选项更改默认值，但我们不建议修改。用下面的命令在新 OSD 主机\
上创建默认目录： ::

	ssh {osd-host}
	sudo mkdir /var/lib/ceph/osd/ceph-{osd-number}

``osd data`` 路径应该指向一个独立硬盘的挂载点，这个硬盘应该独立于操作系统和守护进程\
所在硬盘。按下列步骤准备好并挂载： ::

	ssh {new-osd-host}
	sudo mkfs -t {fstype} /dev/{disk}
	sudo mount -o user_xattr /dev/{hdd} /var/lib/ceph/osd/ceph-{osd-number}

我们推荐用 ``xfs`` 或 ``btrfs`` 文件系统，命令是 :command:mkfs 。

配置详细步骤见 `OSD 配置参考`_\ 。


心跳
====

在运行时， OSD 守护进程会相互检查邻居 OSD 、并把结果报告给 Ceph 监视器，一般不需要\
更改默认配置。但如果你的网络延时比较大，也许需要更改某些选项。

其它细节部分见\ `监视器与 OSD 交互的配置`_\ 。


.. _ceph-logging-and-debugging:

日志、调试
==========

有时候你可能遇到一些麻烦，需要修改日志或调试选项，请参考\ `调试和日志记录`_\ 。

.. _调试和日志记录: ../../troubleshooting/log-and-debug


ceph.conf 实例
==============

.. literalinclude:: demo-ceph.conf
   :language: ini


.. _ceph-runtime-config:

运行时更改
==========

Ceph 可以在运行时更改 ``ceph-osd`` 、 ``ceph-mon`` 、 ``ceph-mds`` 守护进程的配\
置，此功能在增加/降低日志输出、启用/禁用调试设置、甚至是运行时优化的时候非常有用，\
下面是运行时配置方法： ::

	ceph tell {daemon-type}.{id or *} injectargs --{name} {value} [--{name} {value}]

用 ``osd`` 、 ``mon`` 、 ``mds`` 中的一个替代 ``{daemon-type}`` ，你可以用星号\
（ ``*`` ）更改一类进程的所有例程配置、或者更改某一具体进程 ID （即数字或字母）的配\
置。例如提高名为 ``osd.0`` 的 ``ceph-osd`` 进程之调试级别的命令如下： ::

	ceph tell osd.0 injectargs --debug-osd 20 --debug-ms 1

在 ``ceph.conf`` 文件里配置时用空格分隔关键词；但在命令行使用的时候要用下划线或连字\
符（ ``_`` 或 ``-`` ）分隔，例如 ``debug osd`` 变成 ``debug-osd`` 。


查看运行时配置
==============

如果你的 Ceph 存储集群在运行，而你想看一个在运行进程的配置，用下面的命令： ::

	ceph daemon {daemon-type}.{id} config show | less

如果你现在位于 osd.0 所在的主机，命令将是： ::

	ceph daemon osd.0 config show | less


运行多个集群
============

用 Ceph 可以实现在同一套硬件上运行多个集群，运行多个集群与同一集群内用 CRUSH 规则控\
制多个存储池相比提供了更高水平的隔离。独立的集群需配备独立的监视器、 OSD 和元数据服\
务器进程。默认配置下集群名字是 “ceph” ，这意味着你得把配置文件保存为 ``/etc/ceph`` \
下的 ``ceph.conf`` 。

详情见 `ceph-deploy new`_ 。
.. _ceph-deploy new: ../ceph-deploy-new

运行多个集群时，你必须为集群命名并用这个名字保存配置文件。例如，名为 ``openstack`` \
的集群其配置文件将是 ``/etc/ceph`` 下的 ``openstack.conf`` 。

.. important:: 集群名字里只能包含字母 a-z 和数字 0-9 。

独立的集群意味着独立数据盘和日志，它们不能在集群间共享。根据前面的\ `元变量`_\ ， \
``$cluster`` 元变量对应集群名字（就是前面的 ``openstack`` ）。多处设置都用到 \
``$cluster`` 元变量，包括：

- ``keyring``
- ``admin socket``
- ``log file``
- ``pid file``
- ``mon data``
- ``mon cluster log file``
- ``osd data``
- ``osd journal``
- ``mds data``
- ``rgw data``

和 ``$cluster`` 元变量相关的默认路径见\ `常规选项`_\ 、 `OSD 选项`_\ 、\ \
`监视器选项`_\ 、 `MDS 选项`_\ 、 `RGW 选项`_\ 和\ `日志选项`_\ 。

.. _常规选项: ../general-config-ref
.. _OSD 选项: ../osd-config-ref
.. _监视器选项: ../mon-config-ref
.. _MDS 选项: ../../../cephfs/mds-config-ref
.. _RGW 选项: ../../../radosgw/config-ref/
.. _日志选项: ../../troubleshooting/log-and-debug


创建默认目录和文件时，路径里该用集群名的地方要注意，例如： ::

	sudo mkdir /var/lib/ceph/osd/openstack-0
	sudo mkdir /var/lib/ceph/mon/openstack-a

.. important:: 在一台主机上运行多个监视器时，你得指定不同端口。监视器默认使用 6789 \
   端口，如果它已经被占，其它集群得指定其它端口。

要调动一个名字不是 ``ceph`` 的集群，要给 ``ceph`` 命令加 ``-c {filename}.conf`` \
选项，例如： ::

	ceph -c {cluster-name}.conf health
	ceph -c openstack.conf health


.. _硬件推荐: ../../../install/hardware-recommendations
.. _硬件推荐: ../../../install/hardware-recommendations
.. _网络配置参考: ../network-config-ref
.. _OSD 配置参考: ../osd-config-ref
.. _监视器与 OSD 交互的配置: ../mon-osd-interaction
.. _ceph-deploy new: ../../deployment/ceph-deploy-new#naming-a-cluster
