.. _configuring-ceph:

===========
 配置 Ceph
===========

你启动 Ceph 服务时，
初始化进程会把一系列守护进程放到后台运行。
:term:`Ceph 存储集群`\ 运行三种守护进程：

- :term:`Ceph 监视器` （ ``ceph-mon`` ）
- :term:`Ceph 管理器` （ ``ceph-mgr`` ）
- :term:`Ceph OSD 守护进程` （ ``ceph-osd`` ）

支持 :term:`Ceph 文件系统`\ 的 Ceph 存储集群至少运行着一个
:term:`Ceph 元数据服务器`\ （ ``ceph-mds`` ）。支持
:term:`Ceph 对象存储`\ 的集群需运行 Ceph 网关守护进程（
``radosgw`` ）。

各个守护进程都有一系列配置选项，各选项都有其默认值。
你可以更改这些配置选项，以调整系统行为。
覆盖默认值前应该深入理解可能产生的后果，
因为有可能显著降低集群的性能和稳定性。
还要注意后续版本的默认值可能会变，所以，
最好回顾一下与你的 Ceph 版本相对应的文档。


选项名
======
.. Option names

All Ceph configuration options have a unique name consisting of words
formed with lower-case characters and connected with underscore
(``_``) characters.

When option names are specified on the command line, either underscore
(``_``) or dash (``-``) characters can be used interchangeable (e.g.,
``--mon-host`` is equivalent to ``--mon_host``).

When option names appear in configuration files, spaces can also be
used in place of underscore or dash.  We suggest, though, that for
clarity and convenience you consistently use underscores, as we do
throughout this documentation.


配置来源
========
.. Config sources

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


.. _bootstrap-options:

自举引导选项
------------

Some configuration options affect the process's ability to contact the
monitors, to authenticate, and to retrieve the cluster-stored configuration.
For this reason, these options might need to be stored locally on the node, and
set by means of a local configuration file. These options include the
following:

.. confval:: mon_host
.. confval:: mon_host_override

- :confval:`mon_dns_srv_name`
- ``mon_data``, ``osd_data``, ``mds_data``, ``mgr_data``, and
  similar options that define which local directory the daemon
  stores its data in.
- :confval:`keyring`, :confval:`keyfile`, and/or :confval:`key`, which can be used to
  specify the authentication credential to use to authenticate with
  the monitor.  Note that in most cases the default keyring location
  is in the data directory specified above.

In most cases, the default values of these options are suitable. There is one
exception to this: the :confval:`mon_host` option that identifies the addresses
of the cluster's monitors.  When DNS is used to identify monitors, a local Ceph
configuration file can be avoided entirely.


跳过监视器配置
--------------
.. Skipping monitor config

Pass the option ``--no-mon-config`` to any process to skip the step that
retrieves configuration information from the cluster monitors. This is useful
in cases where configuration is managed entirely via configuration files, or
when the monitor cluster is down and some maintenance activity needs to be
done.


.. _ceph-conf-file:

配置段落
========
.. Configuration sections

Any given process or daemon has a single value for each configuration
option.  However, values for an option may vary across different
daemon types even daemons of the same type.  Ceph options that are
stored in the monitor configuration database or in local configuration
files are grouped into sections to indicate which daemons or clients
they apply to.

These sections include:

.. confsec:: global

   ``global`` 下的配置选项影响 Ceph 存储集群里的所有守护进程和\
   客户端。

   :example: ``log_file = /var/log/ceph/$cluster-$type.$id.log``

.. confsec:: mon

   ``mon`` 下的配置影响 Ceph 集群里的所有 ``ceph-mon`` 守护进程，
   并且会覆盖 ``global`` 下的同一选项。

   :example: ``mon_cluster_log_to_syslog = true``

.. confsec:: mgr

   Settings in the ``mgr`` section affect all ``ceph-mgr`` daemons in
   the Ceph Storage Cluster, and override the same setting in
   ``global``.

   :example: ``mgr_stats_period = 10``

.. confsec:: osd

   ``osd`` 下的配置影响 Ceph 存储集群里的所有 ``ceph-osd`` 守护进程，
   并且会覆盖 ``global`` 下的同一选项。

   :example: ``osd_op_queue = wpq``

.. confsec:: mds

   ``mds`` 下的配置影响 Ceph 存储集群里的所有 ``ceph-mds``
   守护进程，并且会覆盖 ``global`` 下的同一选项。

   :example: ``mds_cache_memory_limit = 10G``


.. confsec:: client

   ``[client]`` 下的配置影响所有 Ceph 客户端（如挂载的 Ceph
       文件系统、挂载的块设备等等）、也影响 Rados 网关（ RGW ）\
       守护进程。

   :example: ``objecter_inflight_ops = 512``

Sections may also specify an individual daemon or client name.  For example,
``mon.foo``, ``osd.123``, and ``client.smith`` are all valid section names.

Any given daemon will draw its settings from the global section, the
daemon or client type section, and the section sharing its name.
Settings in the most-specific section take precedence, so for example
if the same option is specified in both :confsec:`global`, :confsec:`mon`, and
``mon.foo`` on the same source (i.e., in the same configurationfile),
the ``mon.foo`` value will be used.

If multiple values of the same configuration option are specified in the same
section, the last value wins.

Note that values from the local configuration file always take
precedence over values from the monitor configuration database,
regardless of which section they appear in.


.. _ceph-metavariables:

元变量
======
.. Metavariables

元变量大大简化了 Ceph 集群配置。如果在配置值里设置了元变量，
Ceph 会在使用此配置值时把相应的元变量展开为具体值。Ceph 元变量\
类似于 Bash shell 的变量展开。

Ceph 支持下列元变量：

.. describe:: $cluster

   展开为存储集群的名字，
   在同一套硬件上运行多个集群时有用。

   :example:``/etc/ceph/$cluster.keyring``
   :default:``ceph``

.. describe:: $type

   可展开为守护进程或进程类型，如 ``mds`` 、 ``osd`` 、 ``mon`` 。

   :example:``/var/lib/ceph/$type``

.. describe:: $id

   展开为守护进程或客户端标识符；
   ``osd.0`` 应为 ``0`` ， ``mds.a`` 是 ``a`` 。

   :example:``/var/lib/ceph/$type/$cluster-$id``

.. describe:: $host

   展开为当前守护进程的主机名。

.. describe:: $name

   展开为 ``$type.$id`` 。

   :example:``/var/run/ceph/$cluster-$name.asok``

.. describe:: $pid

   展开为守护进程的 pid 。

   :example:``/var/run/ceph/$cluster-$name-$pid.asok``



配置文件
========
.. The Configuration File

启动时， Ceph 的各进程会依次到下列位置搜索配置文件：

#. ``$CEPH_CONF`` （\ *就是* ``$CEPH_CONF`` 环境变量所\
   指示的路径）；
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


.. _ceph-conf-settings:

配置文件段落
============
.. Config file section names

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

Every configuration option is typed with one of the types below:

.. describe:: int

   64-bit signed integer, Some SI prefixes are supported, like "K", "M", "G",
   "T", "P", "E", meaning, respectively, 10\ :sup:`3`, 10\ :sup:`6`,
   10\ :sup:`9`, etc.  And "B" is the only supported unit. So, "1K", "1M", "128B" and "-1" are all valid
   option values. Some times, a negative value implies "unlimited" when it comes to
   an option for threshold or limit.

   :example: ``42``, ``-1``

.. describe:: uint

   It is almost identical to ``integer``. But a negative value will be rejected.

   :example: ``256``, ``0``

.. describe:: str

   Free style strings encoded in UTF-8, but some characters are not allowed. Please
   reference the above notes for the details.

   :example: ``"hello world"``, ``"i love \#"``, ``yet-another-name``

.. describe:: boolean

   one of the two values ``true`` or ``false``. But an integer is also accepted,
   where "0" implies ``false``, and any non-zero values imply ``true``.

   :example: ``true``, ``false``, ``1``, ``0``

.. describe:: addr

   a single address optionally prefixed with ``v1``, ``v2`` or ``any`` for the messenger
   protocol. If the prefix is not specified, ``v2`` protocol is used. Please see
   :ref:`address_formats` for more details.

   :example: ``v1:1.2.3.4:567``, ``v2:1.2.3.4:567``, ``1.2.3.4:567``, ``2409:8a1e:8fb6:aa20:1260:4bff:fe92:18f5::567``, ``[::1]:6789``

.. describe:: addrvec

   a set of addresses separated by ",". The addresses can be optionally quoted with ``[`` and ``]``.

   :example: ``[v1:1.2.3.4:567,v2:1.2.3.4:568]``, ``v1:1.2.3.4:567,v1:1.2.3.14:567``  ``[2409:8a1e:8fb6:aa20:1260:4bff:fe92:18f5::567], [2409:8a1e:8fb6:aa20:1260:4bff:fe92:18f5::568]``

.. describe:: uuid

   the string format of a uuid defined by `RFC4122 <https://www.ietf.org/rfc/rfc4122.txt>`_.
   And some variants are also supported, for more details, see
   `Boost document <https://www.boost.org/doc/libs/1_74_0/libs/uuid/doc/uuid.html#String%20Generator>`_.

   :example: ``f81d4fae-7dec-11d0-a765-00a0c91e6bf6``

.. describe:: size

   denotes a 64-bit unsigned integer. Both SI prefixes and IEC prefixes are
   supported. And "B" is the only supported unit. A negative value will be
   rejected.

   :example: ``1Ki``, ``1K``, ``1KiB`` and ``1B``.

.. describe:: secs

   denotes a duration of time. By default the unit is second if not specified.
   Following units of time are supported:

              * second: "s", "sec", "second", "seconds"
              * minute: "m", "min", "minute", "minutes"
              * hour: "hs", "hr", "hour", "hours"
              * day: "d", "day", "days"
              * week: "w", "wk", "week", "weeks"
              * month: "mo", "month", "months"
              * year: "y", "yr", "year", "years"

   :example: ``1 m``, ``1m`` and ``1 week``

.. _ceph-conf-database:

监视器配置数据库
================
.. Monitor configuration database

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

覆盖值
------
.. Override values

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


查看运行时配置
==============
.. Viewing runtime settings

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


nautilus 以来的变化
===================
.. Changes since nautilus

With the Octopus release We changed the way the configuration file is parsed.
These changes are as follows:

- Repeated configuration options are allowed, and no warnings will be printed.
  The value of the last one is used, which means that the setting last in the file
  is the one that takes effect. Before this change, we would print warning messages
  when lines with duplicated options were encountered, like::

    warning line 42: 'foo' in section 'bar' redefined

- Invalid UTF-8 options were ignored with warning messages. But since Octopus,
  they are treated as fatal errors.

- Backslash ``\`` is used as the line continuation marker to combine the next
  line with current one. Before Octopus, it was required to follow a backslash with
  a non-empty line. But in Octopus, an empty line following a backslash is now allowed.

- In the configuration file, each line specifies an individual configuration
  option. The option's name and its value are separated with ``=``, and the
  value may be quoted using single or double quotes. If an invalid
  configuration is specified, we will treat it as an invalid configuration
  file ::

    bad option ==== bad value

- Before Octopus, if no section name was specified in the configuration file,
  all options would be set as though they were within the :confsec:`global` section. This is
  now discouraged. Since Octopus, only a single option is allowed for
  configuration files without a section name.
