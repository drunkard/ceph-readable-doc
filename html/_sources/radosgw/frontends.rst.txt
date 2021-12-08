.. _rgw_frontends:

===========
 HTTP 前端
===========
.. HTTP Frontends

.. contents::

Ceph 对象网关支持两个嵌入式的 HTTP 前端库，
可以用 ``rgw_frontends`` 配置。语法请参考\ `配置参考`_\ 。

Beast
=====

.. versionadded:: Mimic

``beast`` 前端用 Boost.Beast 库解析 HTTP 、
用 Boost.Asio 库做异步网络 I/O 。

选项
----
.. Options

``port`` 和 ``ssl_port``

:描述: 设置 IPv4 和 IPv6 的监听端口号。可以指定多次，
       如 ``port=80 port=8000`` 。
:类型: Integer
:默认值: ``80``


``endpoint`` and ``ssl_endpoint``

:描述: 设置监听地址，格式为 ``address[:port]`` ，
       其中， IPv4 地址应该是点分十进制格式、
       IPv6 地址是用方括号括起来的十六进制表示法。
       指定了 IPv6 端点就只会监听 IPv6 地址。
       端口号是可选的， ``endpoint`` 的默认为 80 、
       ``ssl_endpoint`` 的默认为 443 。
       可以设置多次，比如 ``endpoint=[::1] endpoint=192.168.0.100:8000`` 。
:类型: Integer
:默认值: None


``ssl_certificate``

:描述: SSL 证书文件的路径，用于启用了 SSL 的终结点。
       如果路径前缀是 ``config://`` ，证书将从 ceph 监视器的
       ``config-key`` 数据库拉取。
:类型: String
:默认值: None


``ssl_private_key``

:描述: 可选配置，私钥文件的路径，用于启用了 SSL 的终结点。
       如果没配置此路径，就把 ``ssl_certificate`` 文件\
       当作私钥用。
       如果路径前缀是 ``config://`` ，证书将从 ceph 监视器的
       ``config-key`` 数据库拉取。
:类型: String
:默认值: None


``ssl_options``

:描述: 可选的、冒号分隔的 ssl 内容选项列表：

       ``default_workarounds`` 实现了各种缺陷弥补；

       ``no_compression`` 禁用压缩；

       ``no_sslv2`` 禁用 SSL v2.

       ``no_sslv3`` 禁用 SSL v3.

       ``no_tlsv1`` 禁用 TLS v1.

       ``no_tlsv1_1`` 禁用 TLS v1.1.

       ``no_tlsv1_2`` 禁用 TLS v1.2.

       ``single_dh_use`` 用 tmp_dh 参数的时候总是新建一个密钥。

:类型: String
:默认值: ``no_sslv2:no_sslv3:no_tlsv1:no_tlsv1_1``


``ssl_ciphers``

:描述: 一个或多个由冒号分隔的密码字符串的可选列表。
       字符串的格式在 openssl 的 ciphers(1) 手册里有详细介绍。

:类型: String
:默认值: None


``tcp_nodelay``

:描述: 如果设置了这个选项，会在网络连接上禁用 Nagel 算法，
       意味着数据包会被尽快发送，
       而不是等到完整的缓冲或超时发生。

       ``1`` 对所有套接字禁用 Nagel 算法。

       ``0`` 保持默认值： Nagel 算法仍是启用的。
:类型: Integer (0 or 1)
:默认值: 0


``max_connection_backlog``

:描述: 可选的，用于配置等待被处理的连接队列的最大尺寸。
       如果没配置，就用
       ``boost::asio::socket_base::max_connections`` 的默认值。
:类型: Integer
:默认值: None


``request_timeout_ms``

:描述: Beast 等待后续进栈数据或出栈数据多少毫秒后才放弃。
       把此选项设置为 0 会禁用超时。

:类型: Integer
:默认值: ``65000``


通用选项
========
.. Generic Options

有些前端选项是通用的，所有前端都支持：

``prefix``

:描述: 插入所有请求 URI 里的一个前缀字符串。
       例如，仅支持 swift 的前端可以配置\
       个 ``/swift`` URI 前缀。

:类型: String
:默认值: None


.. _配置参考: ../config-ref
