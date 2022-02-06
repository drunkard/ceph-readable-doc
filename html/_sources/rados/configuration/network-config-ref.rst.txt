==============
 网络配置参考
==============
.. Network Configuration Reference

网络配置对构建高性能 :term:`Ceph 存储集群`\ 来说相当重要。
Ceph 存储集群不会代表 :term:`Ceph 客户端`\ 执行请求路由或调度，\
相反， Ceph 客户端（如块设备、 CephFS 、 REST 网关）直接向 OSD
请求，然后OSD为客户端执行数据复制，也就是说复制和其它因素会额\
外增加集群网的负载。

我们的快速入门配置提供了一个简陋的 Ceph 配置文件，其中\
只设置了监视器 IP 地址和守护进程所在的主机名。如果没有配置\
集群网，那么 Ceph 假设你只有一个“公共网”。只用一个网可以运行
Ceph ，但是在大型集群里用单独的“集群”网可显著地提升性能。

Ceph 存储集群可以运行在两个网络上：
一个公共网（客户端、前端）和一个集群网（私有的、用于复制、后端）。
然而，此方法把网络配置（硬件和软件都是）复杂化了，
而且通常对整体性能也没有太大的影响。
故此，考虑到系统弹性和容量，
我们建议采用双网卡系统，用 active/active 模式\
绑定起来或者做 3 层多路径策略，如 FRR 。

如果你不怕复杂，还想分两个网络，
各 :term:`Ceph 节点`\ 就得配备多个网卡或 VLAN ，
更多细节见\ `硬件推荐——网络`\ 。

.. ditaa::

                               +-------------+
                               | Ceph Client |
                               +----*--*-----+
                                    |  ^
                            Request |  : Response
                                    v  |
    /----------------------------------*--*-------------------------------------\
    |                              Public Network                               |
    \---*--*------------*--*-------------*--*------------*--*------------*--*---/
     ^  ^            ^  ^             ^  ^            ^  ^            ^  ^
     |  |            |  |             |  |            |  |            |  |
     |  :            |  :             |  :            |  :            |  :
     v  v            v  v             v  v            v  v            v  v
    +---*--*---+    +---*--*---+     +---*--*---+    +---*--*---+    +---*--*---+
    | Ceph MON |    | Ceph MDS |     | Ceph OSD |    | Ceph OSD |    | Ceph OSD |
    +----------+    +----------+     +---*--*---+    +---*--*---+    +---*--*---+
                                      ^  ^            ^  ^            ^  ^
     The cluster network relieves     |  |            |  |            |  |
     OSD replication and heartbeat    |  :            |  :            |  :
     traffic from the public network. v  v            v  v            v  v
    /------------------------------------*--*------------*--*------------*--*---\
    |   cCCC                      Cluster Network                               |
    \---------------------------------------------------------------------------/


防火墙
======
.. IP Tables

默认情况下，守护进程会\ `绑定`_\ 到 ``6800:7300`` 间的端口，\
你可以更改此范围。更改防火墙配置前先检查下 ``iptables`` 配置。 ::

	sudo iptables -L

一些 Linux 发行版的规则拒绝除 SSH 之外的所有网卡的所有入栈\
连接，例如： ::

	REJECT all -- anywhere anywhere reject-with icmp-host-prohibited

你得先删掉公共网和集群网对应的这些规则，然后再增加安全保护规则。


监视器防火墙
------------
.. Monitor IP Tables

监视器默认监听 ``6789`` 端口，而且监视器总是运行在公共网。
按下例增加规则时，要把 ``{iface}`` 替换为公共网接口
（如 ``eth0`` 、 ``eth1`` 等等）、
``{ip-address}`` 替换为公共网 IP 、
``{netmask}`` 替换为公共网掩码。 ::

   sudo iptables -A INPUT -i {iface} -p tcp -s {ip-address}/{netmask} --dport 6789 -j ACCEPT


MDS 和管理器防火墙
------------------
.. MDS and Manager IP Tables

:term:`Ceph 元数据服务器`\ 或 :term:`Ceph 管理器`\
会监听公共网 6800 以上的第一个可用端口。
需要注意的是，这种行为是不确定的，
所以如果你在同一主机上运行多个 OSD 或 MDS 、
或者在很短的时间内重启了多个守护进程，它们会绑定更高的端口号；
所以说你应该预先打开整个 6800-7300 端口区间。
按下例增加规则时，要把 ``{iface}`` 替换为\
公共网接口（如 ``eth0`` 、 ``eth1`` 等等）、
``{ip-address}`` 替换为公共网 IP 、 ``{netmask}`` 替换为公共网掩码。

例如： ::

	sudo iptables -A INPUT -i {iface} -m multiport -p tcp -s {ip-address}/{netmask} --dports 6800:7300 -j ACCEPT


OSD 防火墙
----------
.. OSD IP Tables

OSD 守护进程默认\ `绑定`_ 从 6800 起的第一个可用端口，需要注意\
的是，这种行为是不确定的，所以如果你在同一主机上运行多个 OSD
或 MDS 、或者在很短的时间内重启了多个守护进程，它们会绑定更高\
的端口号。一主机上的各个 OSD 最多会用到 4 个端口：

#. 一个用于和客户端、监视器通讯；
#. 一个用于发送数据到其他 OSD ；
#. 两个用于各个网卡上的心跳；

.. ditaa::

              /---------------\
              |      OSD      |
              |           +---+----------------+-----------+
              |           | Clients & Monitors | Heartbeat |
              |           +---+----------------+-----------+
              |               |
              |           +---+----------------+-----------+
              |           | Data Replication   | Heartbeat |
              |           +---+----------------+-----------+
              | cCCC          |
              \---------------/

当某个守护进程失败并重启时没释放端口，重启后的进程就会监听\
新端口。你应该打开整个 6800-7300 端口区间，以应对这种可能性。

如果你分开了公共网和集群网，必须分别为之设置防火墙，
因为客户端会通过公共网连接、而其他 OSD 会通过集群网连接。
按下例增加规则时，要把 ``{iface}`` 替换为网口
（如 ``eth0`` 、 ``eth1`` 等等）、
``{ip-address}`` 替换为公共网或集群网 IP 、
``{netmask}`` 替换为公共网或集群网掩码。例如： ::

	sudo iptables -A INPUT -i {iface}  -m multiport -p tcp -s {ip-address}/{netmask} --dports 6800:7300 -j ACCEPT

.. tip:: 如果你的元数据服务器和 OSD 在同一节点上，可以合并\
   公共网配置。


Ceph 网络
=========
.. Ceph Networks

Ceph 的网络配置要放到 ``[global]`` 段下。
前述的 5 分钟快速入门提供了一个简陋的 Ceph 配置文件，
它假设服务器和客户端都位于同一网段，
Ceph 可以很好地适应这种情形。
然而 Ceph 允许配置更精细的公共网，包括多 IP 和多掩码；
也能用单独的集群网处理 OSD 心跳、对象复制、和恢复流量。
不要混淆你配置的 IP 地址和客户端用来访问存储服务的公共网地址。
典型的内网常常是 ``192.168.0.0`` 或 ``10.0.0.0`` 。

.. tip:: 如果你给公共网或集群网配置了\
   多个 IP 地址及子网掩码，\
   这些子网必须能互通。
   另外要确保在防火墙上为各 IP
   和子网开放了必要的端口。

.. note:: Ceph 用 CIDR 法表示子网，如 ``10.0.0.0/24`` 。

配置完几个网络后，可以重启集群或挨个重启守护进程。
Ceph 守护进程动态地绑定端口，
所以更改网络配置后无需重启整个集群。


公共网
------
.. Public Network

要配置一个公共网，把下列选项加到配置文件的 ``[global]`` 段下。

.. code-block:: ini

	[global]
		# ... elided configuration
		public network = {public-network/netmask}


.. _cluster-network:

集群网
------
.. Cluster Network

如果你声明了集群网， OSD 将把心跳、对象复制和恢复流量路由到\
集群网，与单个网络相比这会提升性能。要配置集群网，把下列选项\
加进配置文件的 ``[global]`` 段。

.. code-block:: ini

	[global]
		# ... elided configuration
		cluster network = {cluster-network/netmask}

为安全起见，从公共网或互联网到集群网应该是\ **不可达**\ 的。


IPv4/IPv6 双栈模式
------------------
.. IPv4/IPv6 Dual Stack Mode

如果你想在 IPv4/IPv6 双栈模式下运行集群、而且想定义公共网和/或集群网，
那么你需要分别配置 IPv4 和 IPv6 网络的地址段：

.. code-block:: ini

	[global]
		# ... elided configuration
		public_network = {IPv4 public-network/netmask}, {IPv6 public-network/netmask}

这样，两种地址下 Ceph 都能找到可用的 IP 地址。

如果你只想要一个 IPv4 或 IPv6 单栈环境，
正确地配置 `ms bind` 选项即可。

.. note::
   绑定到 IPv4 地址默认就是启用的，所以，如果你想加上一个选项绑定到 IPv6 ，
   你其实已经进入了双栈模式。如果你只想要 IPv6 ，可以禁用 IPv4 并启用 IPv6 。
   见下文的 `绑定`_ 。


Ceph 守护进程
=============
.. Ceph Daemons

监视器守护进程都分别配置成了绑定到某一个特定的 IP 地址。
这些地址通常都是由部署工具配置的。
Ceph 集群里的其它组件通过 ``mon host`` 配置选项来发现监视器，
通常写在 ``ceph.conf`` 文件的 ``[global]`` 段下。

.. code-block:: ini

     [global]
         mon host = 10.0.0.2, 10.0.0.3, 10.0.0.4

``mon_host`` 的值可以是 IP 地址列表、或者域名，
可以通过 DNS 查找。如果是域名，有多个 A 或者 AAAA 记录的话，
会按顺序探测所有的地址，以发现监视器。
只要有一个可达，就能发现当前所有的其他监视器，
所以， ``mon host`` 配置选项只要够新，
客户端能找到一个当前在线的监视器就可以。

MGR 、 OSD 、和 MDS 守护进程会绑定到所有可用地址上，
不需要任何特殊的配置。即便如此，
还是可以用 ``public addr`` 配置选项（和/或，
对于 OSD 守护进程是 ``cluster addr`` ）
指定一个让它们绑定的特定 IP 地址。例如，


.. code-block:: ini

	[osd.0]
		public addr = {host-public-ip-address}
		cluster addr = {host-cluster-ip-address}

.. topic:: 单网卡OSD、双网络集群

   一般来说，我们不建议用单网卡 OSD 主机部署两个网络。然而这事\
   可以实现，把 ``public addr`` 选项配在 ``[osd.n]`` 段下即可\
   强制 OSD 主机运行在公共网，其中 ``n`` 是其 OSD 号。另外，\
   公共网和集群网必须互通，考虑到安全因素我们不建议这样做。


网络配置选项
============
.. Network Config Settings

网络配置选项不是必需的， Ceph 假设所有主机都运行于公共网，
除非你特意配置了一个集群网。


公共网
------
.. Public Network

公共网配置用于明确地为公共网定义 IP 地址和子网。你可以分配\
静态 IP 或用 ``public addr`` 覆盖 ``public network`` 选项。

.. confval:: public_network
.. confval:: public_addr


集群网
------
.. Cluster Network

集群网配置用来声明一个集群网，并明确地定义其 IP 地址和子网。\
你可以配置静态 IP 或为某 OSD 守护进程配置 ``cluster addr`` 以\
覆盖 ``cluster network`` 选项。

.. confval:: cluster_network
.. confval:: cluster_addr


绑定
----
.. Bind

绑定选项用于设置 OSD 和 MDS 默认使用的端口范围，默认范围是
``6800:7300`` 。确保\ `防火墙`_\ 开放了对应端口范围。

你也可以让 Ceph 守护进程绑定到 IPv6 地址而非 IPv4 地址。

.. confval:: ms_bind_port_min
.. confval:: ms_bind_port_max
.. confval:: ms_bind_ipv4
.. confval:: ms_bind_ipv6
.. confval:: public_bind_addr


TCP
---

Ceph 默认禁用 TCP 缓冲。

.. confval:: ms_tcp_nodelay
.. confval:: ms_tcp_rcvbuf


常规选项
--------
.. General Settings

.. confval:: ms_type
.. confval:: ms_async_op_threads
.. confval:: ms_initial_backoff
.. confval:: ms_max_backoff
.. confval:: ms_die_on_bad_msg
.. confval:: ms_dispatch_throttle_bytes
.. confval:: ms_inject_socket_failures


.. _伸缩性和高可用性: ../../../architecture#scalability-and-high-availability
.. _硬件推荐——网络: ../../../start/hardware-recommendations#networks
.. _硬件推荐: ../../../start/hardware-recommendations
.. _监视器与 OSD 的交互: ../mon-osd-interaction
.. _消息签名: ../auth-config-ref#signatures
.. _CIDR: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
.. _Nagle 算法: https://en.wikipedia.org/wiki/Nagle's_algorithm
