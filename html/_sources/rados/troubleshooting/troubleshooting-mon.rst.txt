.. _rados-troubleshooting-mon:

================
 监视器故障排除
================
.. Troubleshooting Monitors

.. index:: monitor, high availability

When a cluster encounters monitor-related troubles there's a tendency to
panic, and sometimes with good reason. Losing one or more monitors doesn't
necessarily mean that your cluster is down, so long as a majority are up,
running, and form a quorum.
Regardless of how bad the situation is, the first thing you should do is to
calm down, take a breath, and step through the below troubleshooting steps.


开始排障
========
.. Initial Troubleshooting


**监视器在运行吗？**

  First of all, we need to make sure the monitor (*mon*) daemon processes
  (``ceph-mon``) are running.  You would be amazed by how often Ceph admins
  forget to start the mons, or to restart them after an upgrade. There's no
  shame, but try to not lose a couple of hours looking for a deeper problem.
  When running Kraken or later releases also ensure that the manager
  daemons (``ceph-mgr``) are running, usually alongside each ``ceph-mon``.
 

**到各个监视器节点是否可达？**

  Doesn't happen often, but sometimes there are ``iptables`` rules that
  block accesse to mon nodes or TCP ports. These may be leftovers from
  prior stress-testing or rule development. Try SSHing into
  the server and, if that succeeds, try connecting to the monitor's ports
  (``tcp/3300`` and ``tcp/6789``) using a ``telnet``, ``nc``, or similar tools.

**ceph -s 是否能运行并收到集群回复？**

  如果答案是肯定的，那么你的集群已启动并运行着。可以向你保证\
  的是，监视器们必须形成法定人数之后才会响应 ``status`` 请求。
  还必须确定能联系到至少一个 ``mgr`` 守护进程，最好是全部都在。

  If ``ceph -s`` hangs without obtaining a reply from the cluster
  or showing ``fault`` messages, then it is likely that your monitors
  are either down completely or just a fraction are up -- a fraction
  insufficient to form a majority quorum.  This check will connect to an
  arbitrary mon; in rare cases it may be illuminating to bind to specific
  mons in sequence by adding e.g. ``-m mymon1`` to the command.

**ceph -s 没返回是什么情况？**

  如果你还没跑通所有步骤，先回头做完。

  你可以单独连线各监视器去询问其状态，不论是否达成法定人数。\
  用 ``ceph tell mon.ID mon_status`` 命令，其中 ID 是监视器的\
  标识符，它是针对集群内单个监视器的命令。在\
  `理解 mon_status`_ 小节中，我们将会解读此命令的输出。

  你还可以 ssh 登录到各监视器节点、并查看其守护进程的
  admin socket （管理套接字）。


使用监视器的管理套接字
======================
.. Using the monitor's admin socket

通过管理套接字，你可以用 Unix 套接字文件直接与指定守护进程交互。\
这个文件位于你监视器的 ``run`` 目录下，默认配置时它位于 \
``/var/run/ceph/ceph-mon.ID.asok`` ，但你要是改过就不一定在那里\
了。如果你在那里没找到它，请看看 ``ceph.conf`` 里是否配置了其它\
路径、或者用下面的命令获取： ::

	ceph-conf --name mon.ID --show-config-value admin_socket

请牢记，只有在监视器运行时管理套接字才可用。监视器正常关闭时，\
管理套接字会被删除；如果监视器不运行了、但管理套接字还存在，就\
说明监视器不是正常关闭的。不管怎样，监视器没在运行，你就不能使\
用管理套接字， ``ceph`` 命令会返回类似 \
``Error 111: Connection Refused`` 的错误消息。

Accessing the admin socket is as simple as running ``ceph tell`` on the daemon
you are interested in. For example::

  ceph tell mon.<id> mon_status

Under the hood, this passes the command ``help`` to the running MON daemon
``<id>`` via its "admin socket", which is a file ending in ``.asok``
somewhere under ``/var/run/ceph``. Once you know the full path to the file,
you can even do this yourself::

  ceph --admin-daemon <full_path_to_asok_file> <command>

``ceph`` 工具的 ``help`` 命令会显示管理套接字支持的其它命令。请\
仔细了解一下 ``config get`` 、 ``config show`` 、 ``mon_status`` \
和 ``quorum_status`` 命令，在排除监视器故障时它们能给你些启发。


理解 mon_status
===============
.. Understanding mon_status

``mon_status`` can always be obtained via the admin socket. This command will
output a multitude of information about the monitor, including the same output
you would get with ``quorum_status``.

Take the following example output of ``ceph tell mon.c mon_status``::

  
  { "name": "c",
    "rank": 2,
    "state": "peon",
    "election_epoch": 38,
    "quorum": [
          1,
          2],
    "outside_quorum": [],
    "extra_probe_peers": [],
    "sync_provider": [],
    "monmap": { "epoch": 3,
        "fsid": "5c4e9d53-e2e1-478a-8061-f543f8be4cf8",
        "modified": "2013-10-30 04:12:01.945629",
        "created": "2013-10-29 14:14:41.914786",
        "mons": [
              { "rank": 0,
                "name": "a",
                "addr": "127.0.0.1:6789\/0"},
              { "rank": 1,
                "name": "b",
                "addr": "127.0.0.1:6790\/0"},
              { "rank": 2,
                "name": "c",
                "addr": "127.0.0.1:6795\/0"}]}}

A couple of things are obvious: we have three monitors in the monmap (*a*, *b*
and *c*), the quorum is formed by only two monitors, and *c* is in the quorum
as a *peon*.

Which monitor is out of the quorum?

  The answer would be **a**.

Why?

  Take a look at the ``quorum`` set. We have two monitors in this set: *1*
  and *2*. These are not monitor names. These are monitor ranks, as established
  in the current monmap. We are missing the monitor with rank 0, and according
  to the monmap that would be ``mon.a``.

By the way, how are ranks established?

  Ranks are (re)calculated whenever you add or remove monitors and follow a
  simple rule: the **greater** the ``IP:PORT`` combination, the **lower** the
  rank is. In this case, considering that ``127.0.0.1:6789`` is lower than all
  the remaining ``IP:PORT`` combinations, ``mon.a`` has rank 0.


最常见的监视器问题
==================
.. Most Common Monitor Issues

存在法定人数但是挂了不止一个监视器
----------------------------------
.. Have Quorum but at least one Monitor is down

When this happens, depending on the version of Ceph you are running,
you should be seeing something similar to::

      $ ceph health detail
      [snip]
      mon.a (rank 0) addr 127.0.0.1:6789/0 is down (out of quorum)

How to troubleshoot this?

  First, make sure ``mon.a`` is running.

  Second, make sure you are able to connect to ``mon.a``'s node from the
  other mon nodes. Check the TCP ports as well. Check ``iptables`` and
  ``nf_conntrack`` on all nodes and ensure that you are not
  dropping/rejecting connections.

  If this initial troubleshooting doesn't solve your problems, then it's
  time to go deeper.

  First, check the problematic monitor's ``mon_status`` via the admin
  socket as explained in `使用监视器的管理套接字`_ and
  `理解 mon_status`_.

  If the monitor is out of the quorum, its state should be one of
  ``probing``, ``electing`` or ``synchronizing``. If it happens to be either
  ``leader`` or ``peon``, then the monitor believes to be in quorum, while
  the remaining cluster is sure it is not; or maybe it got into the quorum
  while we were troubleshooting the monitor, so check you ``ceph -s`` again
  just to make sure. Proceed if the monitor is not yet in the quorum.

What if the state is ``probing``?

  This means the monitor is still looking for the other monitors. Every time
  you start a monitor, the monitor will stay in this state for some time
  while trying to find the rest of the monitors specified in the ``monmap``.
  The time a monitor will spend in this state can vary. For instance, when on
  a single-monitor cluster, the monitor will pass through the probing state
  almost instantaneously, since there are no other monitors around. On a
  multi-monitor cluster, the monitors will stay in this state until they
  find enough monitors to form a quorum -- this means that if you have 2 out
  of 3 monitors down, the one remaining monitor will stay in this state
  indefinitively until you bring one of the other monitors up.

  If you have a quorum, however, the monitor should be able to find the
  remaining monitors pretty fast, as long as they can be reached. If your
  monitor is stuck probing and you've gone through with all the communication
  troubleshooting, then there is a fair chance that the monitor is trying
  to reach the other monitors on a wrong address. ``mon_status`` outputs the
  ``monmap`` known to the monitor: check if the other monitor's locations
  match reality. If they don't, jump to
  `修复监视器损坏的 monmap`_; if they do, then it may be related
  to severe clock skews amongst the monitor nodes and you should refer to
  `时钟偏移`_ first, but if that doesn't solve your problem then it is
  the time to prepare some logs and reach out to the community (please refer
  to `收集所需日志`_ on how to best prepare your logs).


What if state is ``electing``?

  This means the monitor is in the middle of an election. With recent Ceph
  releases these typically complete quickly, but at times the monitors can
  get stuck in what is known as an *election storm*. This can indicate
  clock skew among the monitor nodes; jump to
  `时钟偏移`_ for more information. If all your clocks are properly
  synchronized, you should search the mailing lists and tracker.
  This is not a state that is likely to persist and aside from
  (*really*) old bugs there is not an obvious reason besides clock skews on
  why this would happen.  Worst case, if there are enough surviving mons,
  down the problematic one while you investigate.

What if state is ``synchronizing``?

  This means the monitor is synchronizing with the rest of the cluster in
  order to join the quorum. The synchronization process is as faster as
  smaller your monitor store is, so if you have a big store it may
  take a while. Don't worry, it should be finished soon enough.

  However, if you notice that the monitor jumps from ``synchronizing`` to
  ``electing`` and then back to ``synchronizing``, then you do have a
  problem: the cluster state is advancing (i.e., generating new maps) way
  too fast for the synchronization process to keep up. This used to be a
  thing in early Cuttlefish, but since then the synchronization process was
  quite refactored and enhanced to avoid just this sort of behavior. If this
  happens in later versions let us know. And bring some logs
  (see `收集所需日志`_).

What if state is ``leader`` or ``peon``?

  This should not happen. There is a chance this might happen however, and
  it has a lot to do with clock skews -- see `时钟偏移`_. If you're not
  suffering from clock skews, then please prepare your logs (see
  `收集所需日志`_) and reach out to us.


修复监视器损坏的 monmap
-----------------------
.. Recovering a Monitor's Broken monmap

This is how a ``monmap`` usually looks, depending on the number of
monitors::


      epoch 3
      fsid 5c4e9d53-e2e1-478a-8061-f543f8be4cf8
      last_changed 2013-10-30 04:12:01.945629
      created 2013-10-29 14:14:41.914786
      0: 127.0.0.1:6789/0 mon.a
      1: 127.0.0.1:6790/0 mon.b
      2: 127.0.0.1:6795/0 mon.c
      
This may not be what you have however. For instance, in some versions of
early Cuttlefish there was a bug that could cause your ``monmap``
to be nullified.  Completely filled with zeros. This means that not even
``monmaptool`` would be able to make sense of cold, hard, inscrutable zeros.
It's also possible to end up with a monitor with a severely outdated monmap,
notably if the node has been down for months while you fight with your vendor's
TAC.  The subject ``ceph-mon`` daemon might be unable to find the surviving
monitors (e.g., say ``mon.c`` is down; you add a new monitor ``mon.d``,
then remove ``mon.a``, then add a new monitor ``mon.e`` and remove
``mon.b``; you will end up with a totally different monmap from the one
``mon.c`` knows).

In this situation you have two possible solutions:

Scrap the monitor and redeploy

  You should only take this route if you are positive that you won't
  lose the information kept by that monitor; that you have other monitors
  and that they are running just fine so that your new monitor is able
  to synchronize from the remaining monitors. Keep in mind that destroying
  a monitor, if there are no other copies of its contents, may lead to
  loss of data.

Inject a monmap into the monitor

  Usually the safest path. You should grab the monmap from the remaining
  monitors and inject it into the monitor with the corrupted/lost monmap.

  These are the basic steps:

  1. Is there a formed quorum? If so, grab the monmap from the quorum::

      $ ceph mon getmap -o /tmp/monmap

  2. No quorum? Grab the monmap directly from another monitor (this
     assumes the monitor you're grabbing the monmap from has id ID-FOO
     and has been stopped)::

      $ ceph-mon -i ID-FOO --extract-monmap /tmp/monmap

  3. Stop the monitor you're going to inject the monmap into.

  4. Inject the monmap::

      $ ceph-mon -i ID --inject-monmap /tmp/monmap

  5. Start the monitor

  Please keep in mind that the ability to inject monmaps is a powerful
  feature that can cause havoc with your monitors if misused as it will
  overwrite the latest, existing monmap kept by the monitor.


时钟偏移
--------
.. Clock Skews

Monitor operation can be severely affected by clock skew among the quorum's
mons, as the PAXOS consensus algorithm requires tight time alignment.
Skew can result in weird behavior with no obvious
cause. To avoid such issues, you must run a clock synchronization tool
on your monitor nodes:  ``Chrony`` or the legacy ``ntpd``.  Be sure to
configure the mon nodes with the `iburst` option and multiple peers:

* Each other
* Internal ``NTP`` servers
* Multiple external, public pool servers

For good measure, *all* nodes in your cluster should also sync against
internal and external servers, and perhaps even your mons.  ``NTP`` servers
should run on bare metal; VM virtualized clocks are not suitable for steady
timekeeping.  Visit `https://www.ntp.org <https://www.ntp.org>`_ for more info.  Your
organization may already have quality internal ``NTP`` servers you can use.  
Sources for ``NTP`` server appliances include:

* Microsemi (formerly Symmetricom) `https://microsemi.com <https://www.microsemi.com/product-directory/3425-timing-synchronization>`_
* EndRun `https://endruntechnologies.com <https://endruntechnologies.com/products/ntp-time-servers>`_
* Netburner `https://www.netburner.com <https://www.netburner.com/products/network-time-server/pk70-ex-ntp-network-time-server>`_


What's the maximum tolerated clock skew?

  By default the monitors will allow clocks to drift up to 0.05 seconds (50 ms).


Can I increase the maximum tolerated clock skew?

  The maximum tolerated clock skew is configurable via the
  ``mon-clock-drift-allowed`` option, and
  although you *CAN* you almost certainly *SHOULDN'T*. The clock skew mechanism
  is in place because clock-skewed monitors are liely to misbehave. We, as
  developers and QA aficionados, are comfortable with the current default
  value, as it will alert the user before the monitors get out hand. Changing
  this value may cause unforeseen effects on the
  stability of the monitors and overall cluster health.

How do I know there's a clock skew?

  The monitors will warn you via the cluster status ``HEALTH_WARN``. ``ceph health
  detail`` or ``ceph status`` should show something like::

      mon.c addr 10.10.0.1:6789/0 clock skew 0.08235s > max 0.05s (latency 0.0045s)

  That means that ``mon.c`` has been flagged as suffering from a clock skew.

  On releases beginning with Luminous you can issue the
  ``ceph time-sync-status`` command to check status.  Note that the lead mon
  is typically the one with the numerically lowest IP address.  It will always
  show ``0``: the reported offsets of other mons are relative to
  the lead mon, not to any external reference source.


What should I do if there's a clock skew?

  Synchronize your clocks. Running an NTP client may help. If you are already
  using one and you hit this sort of issues, check if you are using some NTP
  server remote to your network and consider hosting your own NTP server on
  your network.  This last option tends to reduce the amount of issues with
  monitor clock skews.


客户端不能连接或挂载
--------------------
.. Client Can't Connect or Mount

检查防火墙配置。有些系统安装工具把 ``REJECT`` 规则加入了
``iptables`` ，它会拒绝除 ``ssh`` 以外的所有入栈连接。如果你的\
监视器主机有这样的 ``REJECT`` 规则，别的客户端进来的连接将遇到\
超时错误而不能挂载。得先找到这条拒绝客户端连入的 ``iptables`` \
规则，例如，你要找到形似以下的规则： ::

	REJECT all -- anywhere anywhere reject-with icmp-host-prohibited

你也许还要在 Ceph 主机上增加 iptables 规则来放通 Ceph 监视器\
端口（即默认的 6789 端口）、和 OSD 端口（默认从 6800 到 7300
）。例如： ::

	iptables -A INPUT -m multiport -p tcp -s {ip-address}/{netmask} --dports 6789,6800:7300 -j ACCEPT


监视器存储故障
==============
.. Monitor Store Failures

存储损坏的症状
--------------
.. Symptoms of store corruption

Ceph 监视器把\ :term:`集群运行图`\ 存储在键值数据库里，像
LevelDB 。如果某个监视器由于键值存储损坏而失败，监视器日志里\
可能出现如下错误消息： ::

  Corruption: error in middle of record

或者： ::

  Corruption: 1 missing files; e.g.: /var/lib/ceph/mon/mon.foo/store.db/1234567.ldb


用健康的监视器恢复
------------------
.. Recovery using healthy monitor(s)

只要有幸存的，我们就可以用新的\ :ref:`替换掉 <adding-and-removing-monitors>`\
损坏的；而且新加入的监视器启动后会与健康节点同步，完全同步后就\
可以服务于客户端了。


.. _mon-store-recovery-using-osds:

用 OSD 恢复
-----------
.. Recovery using OSDs

但是，所有监视器同时失效怎么办呢？我们建议用户在一个 Ceph 集群\
内至少部署三个监视器（最好是五个），所以同时失效的可能性\
非常低。但是，计划外的数据中心掉电、加上配置不当的磁盘和\
文件系统可能致使底层文件系统损坏，并因此损坏所有监视器。在\
这种情况下，我们可以用存储在 OSD 上的信息恢复监视器存储。

.. code-block:: bash

  ms=/root/mon-store
  mkdir $ms

  # 从已关停的 OSD 收集集群运行图
  for host in $hosts; do
    rsync -avz $ms/. user@$host:$ms.remote
    rm -rf $ms
    ssh user@$host <<EOF
      for osd in /var/lib/ceph/osd/ceph-*; do
        ceph-objectstore-tool --data-path \$osd --no-mon-config --op update-mon-db --mon-store-path $ms.remote
      done
  EOF
    rsync -avz user@$host:$ms.remote/. $ms
  done

  # 用收集来的运行图重建监视器存储，如果集群没用 cephx 认证，\
  # 我们可以跳过更新密钥环的步骤，也不用加 --keyring 选项了，\
  # 就是说可以直接运行 ``ceph-monstore-tool $ms rebuild``
  ceph-authtool /path/to/admin.keyring -n mon. \
    --cap mon 'allow *'
  ceph-authtool /path/to/admin.keyring -n client.admin \
    --cap mon 'allow *' --cap osd 'allow *' --cap mds 'allow *'
  # add one or more ceph-mgr's key to the keyring. in this case, an encoded key
  # for mgr.x is added, you can find the encoded key in
  # /etc/ceph/${cluster}.${mgr_name}.keyring on the machine where ceph-mgr is
  # deployed
  ceph-authtool /path/to/admin.keyring --add-key 'AQDN8kBe9PLWARAAZwxXMr+n85SBYbSlLcZnMA==' -n mgr.x \
    --cap mon 'allow profile mgr' --cap osd 'allow *' --cap mds 'allow *'
  # If your monitors' ids are not sorted by ip address, please specify them in order.
  # For example. if mon 'a' is 10.0.0.3, mon 'b' is 10.0.0.2, and mon 'c' is  10.0.0.4,
  # please passing "--mon-ids b a c".
  # In addition, if your monitors' ids are not single characters like 'a', 'b', 'c', please
  # specify them in the command line by passing them as arguments of the "--mon-ids"
  # option. if you are not sure, please check your ceph.conf to see if there is any
  # sections named like '[mon.foo]'. don't pass the "--mon-ids" option, if you are
  # using DNS SRV for looking up monitors.
  ceph-monstore-tool $ms rebuild -- --keyring /path/to/admin.keyring --mon-ids alpha beta gamma

  # 备份一下损坏的 store.db 以防万一！所有监视器上都要备份一下。
  mv /var/lib/ceph/mon/mon.foo/store.db /var/lib/ceph/mon/mon.foo/store.db.corrupted

  # move rebuild store.db into place.  repeat for all monitors.
  mv $ms/store.db /var/lib/ceph/mon/mon.foo/store.db
  chown -R ceph:ceph /var/lib/ceph/mon/mon.foo/store.db

上面的步骤

#. 从所有 OSD 收集映射图
#. 然后重建监视器存储
#. 把各项目加进密钥环文件，并分配相应的能力
#. 用恢复好的副本替换 ``mon.foo`` 上损坏的存储。


已知的局限性
~~~~~~~~~~~~
.. Known limitations

通过上面的步骤无法恢复以下信息：

- **一些加过的密钥环**\ ：所有用 ``ceph auth add`` 命令加上的 OSD 密\
  钥环都从 OSD 副本中恢复了； ``client.admin`` 密钥环也用
  ``ceph-monstore-tool`` 导入了。但是 MDS 密钥环和其它密钥环却\
  丢失了，你也许得手动重加。

- **正在创建的存储池**: If any RADOS pools were in the process of being creating, that state is lost.  The recovery tool assumes that all pools have been created.  If there are PGs that are stuck in the 'unknown' after the recovery for a partially created pool, you can force creation of the *empty* PG with the ``ceph osd force-create-pg`` command.  Note that this will create an *empty* PG, so only do this if you know the pool is empty.

- **MDS 映射图**\ ： MDS 的各种映射图会丢失。


所有尝试都失败了，怎么办？
==========================
.. Everything Failed! Now What?

到外面寻求帮助
--------------
.. Reaching out for help

You can find us on IRC at #ceph and #ceph-devel at OFTC (server irc.oftc.net)
and on ``ceph-devel@vger.kernel.org`` and ``ceph-users@lists.ceph.com``. Make
sure you have grabbed your logs and have them ready if someone asks: the faster
the interaction and lower the latency in response, the better chances everyone's
time is optimized.


收集所需日志
------------
.. Preparing your logs

Monitor logs are, by default, kept in ``/var/log/ceph/ceph-mon.FOO.log*``. We
may want them. However, your logs may not have the necessary information. If
you don't find your monitor logs at their default location, you can check
where they should be by running::

	ceph-conf --name mon.FOO --show-config-value log_file

The amount of information in the logs are subject to the debug levels being
enforced by your configuration files. If you have not enforced a specific
debug level then Ceph is using the default levels and your logs may not
contain important information to track down you issue.
A first step in getting relevant information into your logs will be to raise
debug levels. In this case we will be interested in the information from the
monitor.
Similarly to what happens on other components, different parts of the monitor
will output their debug information on different subsystems.

You will have to raise the debug levels of those subsystems more closely
related to your issue. This may not be an easy task for someone unfamiliar
with troubleshooting Ceph. For most situations, setting the following options
on your monitors will be enough to pinpoint a potential source of the issue::

	debug mon = 10
	debug ms = 1

If we find that these debug levels are not enough, there's a chance we may
ask you to raise them or even define other debug subsystems to obtain infos
from -- but at least we started off with some useful information, instead
of a massively empty log without much to go on with.


我需要重启监视器来更改调试级别吗？
----------------------------------
.. Do I need to restart a monitor to adjust debug levels?

No. You may do it in one of two ways:

You have quorum

  Either inject the debug option into the monitor you want to debug::

        ceph tell mon.FOO config set debug_mon 10/10

  or into all monitors at once::

        ceph tell mon.* config set debug_mon 10/10

No quourm

  Use the monitor's admin socket and directly adjust the configuration
  options::

      ceph daemon mon.FOO config set debug_mon 10/10


Going back to default values is as easy as rerunning the above commands
using the debug level ``1/10`` instead.  You can check your current
values using the admin socket and the following commands::

      ceph daemon mon.FOO config show

or::

      ceph daemon mon.FOO config get 'OPTION_NAME'


在某个调试级别下重现了问题，然后呢？
------------------------------------
.. Reproduced the problem with appropriate debug levels. Now what?

Ideally you would send us only the relevant portions of your logs.
We realise that figuring out the corresponding portion may not be the
easiest of tasks. Therefore, we won't hold it to you if you provide the
full log, but common sense should be employed. If your log has hundreds
of thousands of lines, it may get tricky to go through the whole thing,
specially if we are not aware at which point, whatever your issue is,
happened. For instance, when reproducing, keep in mind to write down
current time and date and to extract the relevant portions of your logs
based on that.

Finally, you should reach out to us on the mailing lists, on IRC or file
a new issue on the `tracker`_.

.. _tracker: http://tracker.ceph.com/projects/ceph/issues/new
