===============================
Looking op Monitors through DNS
===============================

从 11.0.0 版起， RADOS 支持通过 DNS 查询监视器。

这样的话，守护进程和客户端们的 ceph.conf 配置文件里就不需要
*mon host* 配置了。

客户端可以通过 DNS SRV TCP 记录来查询监视器。

这样可减少客户端和监视器的配置。因为通过更新 DNS ，客户端和守\
护进程们就可以发现监视器的拓扑变化。

默认情况下，客户端和守护进程会查询名为 *ceph-mon* （可通过
*mon_dns_srv_name* 配置）的 TCP 服务。


``mon dns srv name``

:描述: 查询监视器主机、地址时，要向 DNS 查询的服务名称。
:类型: String
:默认值: ``ceph-mon``


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

    _ceph-mon._tcp.example.com. 60 IN SRV 10 60 6789 mon1.example.com.
    _ceph-mon._tcp.example.com. 60 IN SRV 10 60 6789 mon2.example.com.
    _ceph-mon._tcp.example.com. 60 IN SRV 10 60 6789 mon3.example.com.

这里，我们配置的监视器在 *6789* 端口上。

当前的客户端和守护进程还\ *不*\ 遵守或理睬 SRV 记录中的权重或\
优先级配置。

返回的所有记录都会按轮询方式平等对待。
