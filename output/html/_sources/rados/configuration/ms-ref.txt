==========
 消息传递
==========


``ms tcp nodelay``

:Description: 在信差的 TCP 会话上禁用 nagle 算法。
:Type: Boolean
:Required: No
:Default: ``true``


``ms initial backoff``

:Description: 出错时重连的初始等待时间。
:Type: Double
:Required: No
:Default: ``.2``


``ms max backoff``

:Description: 出错重连时等待的最大时间。
:Type: Double
:Required: No
:Default: ``15.0``


``ms nocrc``

:Description: 禁用网络消息的 crc 校验， CPU 不足时可提升性能。
:Type: Boolean
:Required: No
:Default: ``false``


``ms die on bad msg``

:Description: 调试选项，不要配置。
:Type: Boolean
:Required: No
:Default: ``false``


``ms dispatch throttle bytes``

:Description: 等着传送的消息尺寸阀值。
:Type: 64-bit Unsigned Integer
:Required: No
:Default: ``100 << 20``


``ms bind ipv6``

:Description: 如果想让守护进程绑定到 IPv6 地址而非 IPv4 就得启用（如果你指定了守\
              护进程或集群 IP 就不必要了）
:Type: Boolean
:Required: No
:Default: ``false``


``ms rwthread stack bytes``

:Description: 堆栈尺寸调试选项，不要配置。
:Type: 64-bit Unsigned Integer
:Required: No
:Default: ``1024 << 10``


``ms tcp read timeout``

:Description: 控制信差关闭空闲连接前的等待秒数。
:Type: 64-bit Unsigned Integer
:Required: No
:Default: ``900``


``ms inject socket failures``

:Description: 调试选项，别配置。
:Type: 64-bit Unsigned Integer
:Required: No
:Default: ``0``
