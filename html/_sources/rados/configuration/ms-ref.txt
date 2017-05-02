==========
 消息传递
==========

常规选项
========


``ms tcp nodelay``

:描述: 在信差的 TCP 会话上禁用 nagle 算法。
:类型: Boolean
:是否必需: No
:默认值: ``true``


``ms initial backoff``

:描述: 出错时重连的初始等待时间。
:类型: Double
:是否必需: No
:默认值: ``.2``


``ms max backoff``

:描述: 出错重连时等待的最大时间。
:类型: Double
:是否必需: No
:默认值: ``15.0``


``ms nocrc``

:描述: 禁用网络消息的 crc 校验， CPU 不足时可提升性能。
:类型: Boolean
:是否必需: No
:默认值: ``false``


``ms die on bad msg``

:描述: 调试选项，不要配置。
:类型: Boolean
:是否必需: No
:默认值: ``false``


``ms dispatch throttle bytes``

:描述: 等着传送的消息尺寸阀值。
:类型: 64-bit Unsigned Integer
:是否必需: No
:默认值: ``100 << 20``


``ms bind ipv6``

:描述: 如果想让守护进程绑定到 IPv6 地址而非 IPv4 就得启用（如\
       果你指定了守护进程或集群 IP 就不必要了）
:类型: Boolean
:是否必需: No
:默认值: ``false``


``ms rwthread stack bytes``

:描述: 堆栈尺寸调试选项，不要配置。
:类型: 64-bit Unsigned Integer
:是否必需: No
:默认值: ``1024 << 10``


``ms tcp read timeout``

:描述: 控制信差关闭空闲连接前的等待秒数。
:类型: 64-bit Unsigned Integer
:是否必需: No
:默认值: ``900``


``ms inject socket failures``

:描述: 调试选项，别配置。
:类型: 64-bit Unsigned Integer
:是否必需: No
:默认值: ``0``


.. _Async messenger options:

Async Messenger （异步信使）选项
================================


``ms async transport type``

:描述: Async Messenger 所用的传输类型，可以是 ``posix`` 、
       ``dpdk`` 或者 ``rdma`` 。 POSIX 用标准 TCP/IP 网络，是\
       默认的；其它传输类型还是实验性的，支持尚不完整。
:类型: String
:是否必需: No
:默认值: ``posix``


``ms async op threads``

:描述: 各 Async Messenger 例程所用工作线程的初始数量。至少也得\
       等于副本数的最大值，但是，如果 CPU 核心数比较少、或者单\
       台服务器上的 OSD 很多，那你可以适当降低些。
:类型: 64-bit Unsigned Integer
:是否必需: No
:默认值: ``3``


``ms async max op threads``

:描述: 各 Async Messenger 例程所用工作线程的最大数量。 CPU 核\
       心数少时可以设置得小些， CPU 利用率低时可以大些（也就是\
       在 I/O 操作期间会有一或多个 CPU 核心的利用率持续在 100% ）。
:类型: 64-bit Unsigned Integer
:是否必需: No
:默认值: ``5``


``ms async set affinity``

:描述: 设置为 true 时， Async Messenger 工作线程会绑到特定的
       CPU 核心。
:类型: Boolean
:是否必需: No
:默认值: ``true``


``ms async affinity cores``

:描述: ``ms async set affinity`` 为 true 时，此处的字符串可配置
       Async Messenger 工作线程如何绑到 CPU 核心上。例如， "0,2"
       表示把工作线程 #1 和 #2 分别绑到 CPU 核心 #0 和 #2 上。
       **注意：** 手动设置亲和性时，千万别把工作线程绑到超线程\
       或类似技术所虚拟出的 CPU 上，因为它们比一般 CPU 慢。
:类型: String
:是否必需: No
:默认值: ``(empty)``


``ms async send inline``

:描述: 让生成这些消息的线程直接发送出去，而不是放入队列、再让
       Async Messenger 线程发送。现已知晓，在 CPU 核心很多的系\
       统上，启用此选项会降低性能，所以默认禁用了。
:类型: Boolean
:是否必需: No
:默认值: ``false``


