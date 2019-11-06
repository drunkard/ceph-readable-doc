.. _configuring-ceph:

===========
 配置 Ceph
===========

你启动 Ceph 服务时，初始化进程会把一系列守护进程放到后台运行。
:term:`Ceph 存储集群`\ 运行三种守护进程：

- :term:`Ceph 监视器` （ ``ceph-mon`` ）
- :term:`Ceph 管理器` （ ``ceph-mgr`` ）
- :term:`Ceph OSD 守护进程` （ ``ceph-osd`` ）

支持 :term:`Ceph 文件系统`\ 的 Ceph 存储集群至少运行着一个
:term:`Ceph 元数据服务器`\ （ ``ceph-mds`` ）。支持
:term:`Ceph 对象存储`\ 的集群需运行 Ceph 网关守护进程（
``radosgw`` ）。

各个守护进程都有一系列配置选项，各选项都有其默认值。你可以更改\
这些配置选项，以调整系统行为。


.. Option names

选项名
======

All Ceph configuration options have a unique name consisting of words
formed with lower-case characters and connected with underscore
(``_``) characters.

When option names are specified on the command line, either underscore
(``_``) or dash (``-``) characters can be used interchangeable (e.g.,
``--mon-host`` is equivalent to ``--mon_host``).

When option names appear in configuration files, spaces can also be
used in place of underscore or dash.


.. Config sources

配置来源
========

Each Ceph daemon, process, and library will pull its configuration
from several sources, listed below.  Sources later in the list will
override those earlier in the list when both are present.

- the compiled-in default value
- the monitor cluster's centralized configuration database
- a configuration file stored on the local host
- environment variables
- command line arguments
- runtime overrides set by an administrator

One of the first things a Ceph process does on startup is parse the
configuration options provided via the command line, environment, and
local configuration file.  The process will then contact the monitor
cluster to retrieve configuration stored centrally for the entire
cluster.  Once a complete view of the configuration is available, the
daemon or process startup will proceed.


.. Bootstrap options

自举引导选项
------------

Because some configuration options affect the process's ability to
contact the monitors, authenticate, and retrieve the cluster-stored
configuration, they may need to be stored locally on the node and set
in a local configuration file.  These options include:

  - ``mon_host``, the list of monitors for the cluster
  - ``mon_dns_serv_name`` (default: `ceph-mon`), the name of the DNS
    SRV record to check to identify the cluster monitors via DNS
  - ``mon_data``, ``osd_data``, ``mds_data``, ``mgr_data``, and
    similar options that define which local directory the daemon
    stores its data in.
  - ``keyring``, ``keyfile``, and/or ``key``, which can be used to
    specify the authentication credential to use to authenticate with
    the monitor.  Note that in most cases the default keyring location
    is in the data directory specified above.

In the vast majority of cases the default values of these are
appropriate, with the exception of the ``mon_host`` option that
identifies the addresses of the cluster's monitors.  When DNS is used
to identify monitors a local ceph configuration file can be avoided
entirely.


.. Skipping monitor config

跳过监视器配置
--------------

Any process may be passed the option ``--no-mon-config`` to skip the
step that retrieves configuration from the cluster monitors.  This is
useful in cases where configuration is managed entirely via
configuration files or where the monitor cluster is currently down but
some maintenance activity needs to be done.


.. Configuration sections
.. _ceph-conf-file:

配置段落
========

Any given process or daemon has a single value for each configuration
option.  However, values for an option may vary across different
daemon types even daemons of the same type.  Ceph options that are
stored in the monitor configuration database or in local configuration
files are grouped into sections to indicate which daemons or clients
they apply to.

These sections include:


``global``

:描述: ``global`` 下的配置影响 Ceph 存储集群里的所有守护进程和\
       客户端。
:实例: ``log_file = /var/log/ceph/$cluster-$type.$id.log``


``mon``

:描述: ``mon`` 下的配置影响 Ceph 集群里的所有 ``ceph-mon``
       守护进程，并且会覆盖 ``global`` 下的同一选项。
:实例: ``mon_cluster_log_to_syslog = true``


``mgr``

:描述: Settings in the ``mgr`` section affect all ``ceph-mgr`` daemons in
              the Ceph Storage Cluster, and override the same setting in 
              ``global``.
:实例: ``mgr_stats_period = 10``


``osd``

:描述: ``osd`` 下的配置影响 Ceph 存储集群里的所有 ``ceph-osd``
         守护进程，并且会覆盖 ``global`` 下的同一选项。
:实例: ``osd_op_queue = wpq``


``mds``

:描述: ``mds`` 下的配置影响 Ceph 存储集群里的所有 ``ceph-mds``
       守护进程，并且会覆盖 ``global`` 下的同一选项。
:实例: ``mds_cache_size = 10G``


``client``

:描述: ``[client]`` 下的配置影响所有 Ceph 客户端（如挂载的 Ceph
       文件系统、挂载的块设备等等）、也影响 Rados 网关（ RGW ）\
       守护进程。
:实例: ``objecter_inflight_ops = 512``

Sections may also specify an individual daemon or client name.  For example,
``mon.foo``, ``osd.123``, and ``client.smith`` are all valid section names.


Any given daemon will draw its settings from the global section, the
daemon or client type section, and the section sharing its name.
Settings in the most-specific section take precedence, so for example
if the same option is specified in both ``global``, ``mon``, and
``mon.foo`` on the same source (i.e., in the same configurationfile),
the ``mon.foo`` value will be used.

If multiple values of the same configuration option are specified in the same
section, the last value wins.

Note that values from the local configuration file always take
precedence over values from the monitor configuration database,
regardless of which section they appear in.


.. Metavariables
.. _ceph-metavariables:

元变量
======

元变量大大简化了 Ceph 集群配置。如果在配置值里设置了元变量，
Ceph 会在使用此配置值时把相应的元变量展开为具体值。Ceph 元变量\
类似于 Bash shell 的变量展开。

Ceph 支持下列元变量：


``$cluster``

:描述: 展开为存储集群名字，在同一套硬件上运行多个集群时有用。
:实例: ``/etc/ceph/$cluster.keyring``
:默认值: ``ceph``


``$type``

:描述: 可展开为 ``mds`` 、 ``osd`` 、 ``mon`` 中的一个，有赖于\
       当前守护进程的类型。
:实例: ``/var/lib/ceph/$type``


``$id``

:描述: 展开为守护进程标识符； ``osd.0`` 应为 ``0`` ， ``mds.a``
       是 ``a`` 。
:实例: ``/var/lib/ceph/$type/$cluster-$id``


``$host``

:描述: 展开为当前守护进程的主机名。


``$name``

:描述: 展开为 ``$type.$id`` 。
:实例: ``/var/run/ceph/$cluster-$name.asok``


``$pid``

:描述: 展开为守护进程的 pid 。
:实例: ``/var/run/ceph/$cluster-$name-$pid.asok``


.. The Configuration File

配置文件
========

启动时， Ceph 的各进程会依次到下列位置搜索配置文件：

#. ``$CEPH_CONF`` （\ *就是* ``$CEPH_CONF`` 环境变量所指示的路径）；
#. ``-c path/path``  （\ *就是* ``-c`` 命令行参数）；
#. ``/etc/ceph/$cluster.conf``
#. ``~/.ceph/$cluster.conf``
#. ``./$cluster.conf`` （\ *就是*\ 当前的工作路径）。
#. 对 FreeBSD 而言， ``/usr/local/etc/ceph/$cluster.conf``

其中， ``$cluster`` 代表集群名（默认为 ``ceph`` ）。

Ceph 配置文件使用 *ini* 风格的语法，以分号 (;) 和井号 (#) 开始\
的行是注释，如下：

.. code-block:: ini

	# <--A number (#) sign precedes a comment.
	; A comment may be anything.
	# Comments always follow a semi-colon (;) or a pound (#) on each line.
	# The end of the line terminates a comment.
	# We recommend that you provide comments in your configuration file(s).


.. Config file section names
.. _ceph-conf-settings:

配置文件段落
============

The configuration file is divided into sections. Each section must begin with a
valid configuration section name (see `配置段落`_, above)
surrounded by square brackets. For example,

.. code-block:: ini

	[global]
	debug ms = 0
	
	[osd]
	debug ms = 1

	[osd.1]
	debug ms = 10

	[osd.2]
	debug ms = 10


Config file option values
-------------------------

The value of a configuration option is a string. If it is too long to
fit in a single line, you can put a backslash (``\``) at the end of line
as the line continuation marker, so the value of the option will be
the string after ``=`` in current line combined with the string in the next
line::

  [global]
  foo = long long ago\
  long ago

In the example above, the value of "``foo``" would be "``long long ago long ago``".

Normally, the option value ends with a new line, or a comment, like

.. code-block:: ini

    [global]
    obscure one = difficult to explain # I will try harder in next release
    simpler one = nothing to explain

In the example above, the value of "``obscure one``" would be "``difficult to explain``";
and the value of "``simpler one`` would be "``nothing to explain``".

If an option value contains spaces, and we want to make it explicit, we
could quote the value using single or double quotes, like

.. code-block:: ini

    [global]
    line = "to be, or not to be"

Certain characters are not allowed to be present in the option values directly.
They are ``=``, ``#``, ``;`` and ``[``. If we have to, we need to escape them,
like

.. code-block:: ini

    [global]
    secret = "i love \# and \["


.. Monitor configuration database

监视器配置数据库
================

The monitor cluster manages a database of configuration options that
can be consumed by the entire cluster, enabling streamlined central
configuration management for the entire system.  The vast majority of
configuration options can and should be stored here for ease of
administration and transparency.

A handful of settings may still need to be stored in local
configuration files because they affect the ability to connect to the
monitors, authenticate, and fetch configuration information.  In most
cases this is limited to the ``mon_host`` option, although this can
also be avoided through the use of DNS SRV records.

Sections and masks
------------------

Configuration options stored by the monitor can live in a global
section, daemon type section, or specific daemon section, just like
options in a configuration file can.

In addition, options may also have a *mask* associated with them to
further restrict which daemons or clients the option applies to.
Masks take two forms:

#. ``type:location`` where *type* is a CRUSH property like `rack` or
   `host`, and *location* is a value for that property.  For example,
   ``host:foo`` would limit the option only to daemons or clients
   running on a particular host.
#. ``class:device-class`` where *device-class* is the name of a CRUSH
   device class (e.g., ``hdd`` or ``ssd``).  For example,
   ``class:ssd`` would limit the option only to OSDs backed by SSDs.
   (This mask has no effect for non-OSD daemons or clients.)

When setting a configuration option, the `who` may be a section name,
a mask, or a combination of both separated by a slash (``/``)
character.  For example, ``osd/rack:foo`` would mean all OSD daemons
in the ``foo`` rack.

When viewing configuration options, the section name and mask are
generally separated out into separate fields or columns to ease readability.


Commands
--------

The following CLI commands are used to configure the cluster:

* ``ceph config dump`` will dump the entire configuration database for
  the cluster.

* ``ceph config get <who>`` will dump the configuration for a specific
  daemon or client (e.g., ``mds.a``), as stored in the monitors'
  configuration database.

* ``ceph config set <who> <option> <value>`` will set a configuration
  option in the monitors' configuration database.

* ``ceph config show <who>`` will show the reported running
  configuration for a running daemon.  These settings may differ from
  those stored by the monitors if there are also local configuration
  files in use or options have been overridden on the command line or
  at run time.  The source of the option values is reported as part
  of the output.

* ``ceph config assimilate-conf -i <input file> -o <output file>``
  will ingest a configuration file from *input file* and move any
  valid options into the monitors' configuration database.  Any
  settings that are unrecognized, invalid, or cannot be controlled by
  the monitor will be returned in an abbreviated config file stored in
  *output file*.  This command is useful for transitioning from legacy
  configuration files to centralized monitor-based configuration.


Help
====

You can get help for a particular option with::

  ceph config help <option>

Note that this will use the configuration schema that is compiled into the running monitors.  If you have a mixed-version cluster (e.g., during an upgrade), you might also want to query the option schema from a specific running daemon::

  ceph daemon <name> config help [option]

For example,::

  $ ceph config help log_file
  log_file - path to log file
    (std::string, basic)
    Default (non-daemon):
    Default (daemon): /var/log/ceph/$cluster-$name.log
    Can update at runtime: false
    See also: [log_to_stderr,err_to_stderr,log_to_syslog,err_to_syslog]

or::

  $ ceph config help log_file -f json-pretty
  {
      "name": "log_file",
      "type": "std::string",
      "level": "basic",
      "desc": "path to log file",
      "long_desc": "",
      "default": "",
      "daemon_default": "/var/log/ceph/$cluster-$name.log",
      "tags": [],
      "services": [],
      "see_also": [
          "log_to_stderr",
          "err_to_stderr",
          "log_to_syslog",
          "err_to_syslog"
      ],
      "enum_values": [],
      "min": "",
      "max": "",
      "can_update_at_runtime": false
  }

The ``level`` property can be any of `basic`, `advanced`, or `dev`.
The `dev` options are intended for use by developers, generally for
testing purposes, and are not recommended for use by operators.


Runtime Changes
===============

大多数情况下， Ceph 都允许在运行时更改守护进程的配置。此功能在\
增加/降低日志输出、启用/禁用调试设置、甚至是运行时优化的时候\
非常有用。

Generally speaking, configuration options can be updated in the usual
way via the ``ceph config set`` command.  For example, do enable the debug log level on a specific OSD,::

  ceph config set osd.123 debug_ms 20

Note that if the same option is also customized in a local
configuration file, the monitor setting will be ignored (it has a
lower priority than the local config file).


.. Override values

覆盖值
------

You can also temporarily set an option using the `tell` or `daemon`
interfaces on the Ceph CLI.  These *override* values are ephemeral in
that they only affect the running process and are discarded/lost if
the daemon or process restarts.

Override values can be set in two ways:

#. From any host, we can send a message to a daemon over the network with::

     ceph tell <name> config set <option> <value>

   For example,::

     ceph tell osd.123 config set debug_osd 20

   The `tell` command can also accept a wildcard for the daemon
   identifier.  For example, to adjust the debug level on all OSD
   daemons,::

     ceph tell osd.* config set debug_osd 20

#. From the host the process is running on, we can connect directly to
   the process via a socket in ``/var/run/ceph`` with::

     ceph daemon <name> config set <option> <value>

   For example,::

     ceph daemon osd.4 config set debug_osd 20

Note that in the ``ceph config show`` command output these temporary
values will be shown with a source of ``override``.


.. Viewing runtime settings

查看运行时配置
==============

You can see the current options set for a running daemon with the ``ceph config show`` command.  For example,::

  ceph config show osd.0

will show you the (non-default) options for that daemon.  You can also look at a specific option with::

  ceph config show osd.0 debug_osd

or view all options (even those with default values) with::

  ceph config show-with-defaults osd.0

You can also observe settings for a running daemon by connecting to it from the local host via the admin socket.  For example,::

  ceph daemon osd.0 config show

will dump all current settings,::

  ceph daemon osd.0 config diff

will show only non-default settings (as well as where the value came from: a config file, the monitor, an override, etc.), and::

  ceph daemon osd.0 config get debug_osd

will report the value of a single option.


.. Changes since nautilus

nautilus 以来的变化
===================

We changed the way the configuration file is parsed in Octopus. The changes are
listed as follows:

- repeated configuration options are allowed, and no warnings will be printed.
  The value of the last one wins. Before this change, we would print out warning
  messages at seeing lines with duplicated values, like::

    warning line 42: 'foo' in section 'bar' redefined
- invalid UTF-8 options are ignored with warning messages. But since Octopus,
  they are treated as fatal errors.
- backslash ``\`` is used as the line continuation marker to combine the next
  line with current one. Before Octopus, it was required to follow backslash with
  non-empty line. But in Octopus, empty line following backslash is now allowed.
- In the configuration file, each line specifies an individual configuration
  option. The option's name and its value are separated with ``=``. And the
  value can be quoted using single or double quotes. But if an invalid
  configuration is specified, we will treat it as an invalid configuration
  file ::

    bad option ==== bad value
- Before Octopus, if no section name was specified in the configuration file,
  all options would be grouped into the section of ``global``. But this is
  discouraged now. Since Octopus, only a single option is allowed for
  configuration files without a section name.
