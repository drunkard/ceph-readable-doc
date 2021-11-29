.. _mgr-administrator-guide:

ceph-mgr 管理员指南
===================

手动设置
--------
.. Manual setup

平时，你可以用 ceph-ansible 之类的工具配置 ceph-mgr 守护进程。\
下面说明了如何手动配置好一个 ceph-mgr 守护进程。

首先，给守护进程创建认证密钥： ::

    ceph auth get-or-create mgr.$name mon 'allow profile mgr' osd 'allow *' mds 'allow *'

把创建的密钥放入 ``mgr data`` 所指路径的 ``keyring`` 文件内，
对于名为 ceph 的集群、 mgr 的 $name 为 foo 的路径应该是
``/var/lib/ceph/mgr/ceph-foo`` 内的文件 ``/var/lib/ceph/mgr/ceph-foo/keyring`` 。

启动 ceph-mgr 守护进程： ::

    ceph-mgr -i $name

通过 ``ceph status`` 的输出可检查 mgr 是否起来了，起来的话应该\
会包含一行 mgr 状态： ::

    mgr active: $name

客户端认证
----------
.. Client authentication

管理器是一种新的守护进程，需要新的 CephX 能力。如果你是从旧版
Ceph 升级集群、或者使用默认的安装/部署工具，那么管理客户端应该\
可以自动获取到这个能力。如果你用了非标准工具，在调用某些 ceph
集群命令时可能会遇到 EACCES 错误。要修正此问题，需\
`更改用户能力`_\ ，在客户端的 cephx 能力里加上 ``mgr allow \*``
声明。

高可用性
--------
.. High availability

通常来说，你应该在每台运行 ceph-mon 守护进程的主机上都配置一个
ceph-mgr ，以实现相同级别的可用性。

默认情况下，监视器会把任意一个最先启动的
ceph-mgr 例程当作活跃的，其它的作为备用。
ceph-mgr 守护进程无需形成法定人数。

如果活跃的守护进程在 :confval:`mon_mgr_beacon_grace` 这么长的时间内\
都没向监视器们发送信标，那它就会被备用顶替。

如果你想提前做故障切换，可以用 ``ceph mgr fail <mgr name>`` 把
ceph-mgr 明确地标记为已失效。

模块的使用
----------
.. Using modules

用 ``ceph mgr module ls`` 命令可查看有哪些模块可用、
哪些是当前已经启用的；用 ``ceph mgr module ls --format=json-pretty`` 命令\
可以查看已禁用模块的详细元数据。
启用或禁用模块分别使用命令 ``ceph mgr module enable <module>``
和 ``ceph mgr module disable <module>`` 。

If a module is *enabled* then the active ceph-mgr daemon will load
and execute it.  In the case of modules that provide a service,
such as an HTTP server, the module may publish its address when it
is loaded.  To see the addresses of such modules, use the command 
``ceph mgr services``.

Some modules may also implement a special standby mode which runs on
standby ceph-mgr daemons as well as the active daemon.  This enables
modules that provide services to redirect their clients to the active
daemon, if the client tries to connect to a standby.

Consult the documentation pages for individual manager modules for more
information about what functionality each module provides.

Here is an example of enabling the :term:`Dashboard` module:

.. code-block:: console

	$ ceph mgr module ls
	{
		"enabled_modules": [
			"restful",
			"status"
		],
		"disabled_modules": [
			"dashboard"
		]
	}

	$ ceph mgr module enable dashboard
	$ ceph mgr module ls
	{
		"enabled_modules": [
			"restful",
			"status",
			"dashboard"
		],
		"disabled_modules": [
		]
	}

	$ ceph mgr services
	{
		"dashboard": "http://myserver.com:7789/",
		"restful": "https://myserver.com:8789/"
	}


The first time the cluster starts, it uses the :confval:`mgr_initial_modules`
setting to override which modules to enable.  However, this setting
is ignored through the rest of the lifetime of the cluster: only
use it for bootstrapping.  For example, before starting your
monitor daemons for the first time, you might add a section like
this to your ``ceph.conf``:

.. code-block:: ini

    [mon]
        mgr_initial_modules = dashboard balancer

Module Pool
-----------

The manager creates a pool for use by its module to store state. The name of
this pool is ``.mgr`` (with the leading ``.`` indicating a reserved pool
name).

.. note::

   Prior to Quincy, the ``devicehealth`` module created a
   ``device_health_metrics`` pool to store device SMART statistics. With
   Quincy, this pool is automatically renamed to be the common manager module
   pool.


调用模块命令
------------
.. Calling module commands

对于实现了命令行钩子的模块，其实现的命令可以像一般的 Ceph 命令\
那样调用。 Ceph 会自动把模块命令整合进标准 CLI 接口，并正确地\
路由到那个模块。 ::

    ceph <command | help>

配置选项
--------
.. Configuration

.. confval:: mgr_module_path
.. confval:: mgr_initial_modules
.. confval:: mgr_disabled_modules
.. confval:: mgr_standby_modules
.. confval:: mgr_data
.. confval:: mgr_tick_period
.. confval:: mon_mgr_beacon_grace


.. _更改用户能力: ../../rados/operations/user-management/#modify-user-capabilities
