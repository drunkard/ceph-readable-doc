=====================
 通过 DNS 查询监视器
=====================
.. Looking up Monitors through DNS

从 11.0.0 版起， RADOS 支持通过 DNS 查询监视器。

这样的话，守护进程和客户端们的 ceph.conf 配置文件里就不需要
*mon host* 配置了。

客户端可以通过 DNS SRV TCP 记录来查询监视器。

这样可减少客户端和监视器的配置。因为通过更新 DNS ，客户端和守\
护进程们就可以发现监视器的拓扑变化。

默认情况下，客户端和守护进程会查询名为 *ceph-mon* （可通过
*mon_dns_srv_name* 配置）的 TCP 服务。

.. confval:: mon_dns_srv_name


实例
----

假设 DNS 域名是 *example.com* ，域文件内可加上如下配置。

首先，创建监视器的相关记录，可以是 IPv4 (A) 或 IPv6 (AAAA) 的。

::

    mon1.example.com. AAAA 2001:db8::100
    mon2.example.com. AAAA 2001:db8::200
    mon3.example.com. AAAA 2001:db8::300

::

    mon1.example.com. A 192.168.0.1
    mon2.example.com. A 192.168.0.2
    mon3.example.com. A 192.168.0.3

这些记录配置好后，我们用 *ceph-mon* 名字创建 SRV TCP 记录，分\
别指向三个监视器。 ::

    _ceph-mon._tcp.example.com. 60 IN SRV 10 20 6789 mon1.example.com.
    _ceph-mon._tcp.example.com. 60 IN SRV 10 30 6789 mon2.example.com.
    _ceph-mon._tcp.example.com. 60 IN SRV 20 50 6789 mon3.example.com.

此时，所有监视器都运行在 *6789* 端口上，它们的优先级分别是
10 、 10 、 20 ，权重分别是 20 、 30 、 50 。

Monitor clients choose monitor by referencing the SRV records. If a cluster has multiple Monitor SRV records
with the same priority value, clients and daemons will load balance the connections to Monitors in proportion
to the values of the SRV weight fields.

For the above example, this will result in approximate 40% of the clients and daemons connecting to mon1,
60% of them connecting to mon2. However, if neither of them is reachable, then mon3 will be reconsidered as a fallback.
