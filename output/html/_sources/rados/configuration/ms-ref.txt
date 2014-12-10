==========
 消息传递
==========


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

:描述: 如果想让守护进程绑定到 IPv6 地址而非 IPv4 就得启用（如果你指定了守护\
       进程或集群 IP 就不必要了）
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
