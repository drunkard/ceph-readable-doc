.. _rgw_frontends:

===========
 HTTP 前端
===========
.. HTTP Frontends

.. contents::

Ceph 对象网关支持两个嵌入式的 HTTP 前端库，可以用
``rgw_frontends`` 配置。语法请参考\ `配置参考`_\ 。

Beast
=====

.. versionadded:: Mimic

The ``beast`` frontend uses the Boost.Beast library for HTTP parsing
and the Boost.Asio library for asynchronous network i/o.


选项
----
.. Options

``port`` 和 ``ssl_port``

:描述: 设置 IPv4 和 IPv6 的监听端口号。可以指定多次，如
       ``port=80 port=8000`` 。
:类型: Integer
:默认值: ``80``


``endpoint`` and ``ssl_endpoint``

:描述: Sets the listening address in the form ``address[:port]``, where
              the address is an IPv4 address string in dotted decimal form, or
              an IPv6 address in hexadecimal notation surrounded by square
              brackets. Specifying a IPv6 endpoint would listen to v6 only. The
              optional port defaults to 80 for ``endpoint`` and 443 for
              ``ssl_endpoint``. Can be specified multiple times as in
              ``endpoint=[::1] endpoint=192.168.0.100:8000``.
:类型: Integer
:默认值: None


``ssl_certificate``

:描述: SSL 证书文件的路径，用于启用了 SSL 的终结点。
       如果路径前缀是 ``config://`` ，证书将从 ceph 监视器的
       ``config-key`` 数据库拉取。
:类型: String
:默认值: None


``ssl_private_key``

:描述: 可选配置，私钥文件的路径，用于启用了 SSL 的终结点。如果\
       没配置此路径，就把 ``ssl_certificate`` 文件当作私钥用。
       如果路径前缀是 ``config://`` ，证书将从 ceph 监视器的
       ``config-key`` 数据库拉取。
:类型: String
:默认值: None


``ssl_options``

:描述: Optional colon separated list of ssl context options:

              ``default_workarounds`` Implement various bug workarounds.

              ``no_compression`` Disable compression.

              ``no_sslv2`` Disable SSL v2.

              ``no_sslv3`` Disable SSL v3.

              ``no_tlsv1`` Disable TLS v1.

              ``no_tlsv1_1`` Disable TLS v1.1.

              ``no_tlsv1_2`` Disable TLS v1.2.

              ``single_dh_use`` Always create a new key when using tmp_dh parameters.

:Type: String
:Default: ``no_sslv2:no_sslv3:no_tlsv1:no_tlsv1_1``


``ssl_ciphers``

:Description: Optional list of one or more cipher strings separated by colons.
              The format of the string is described in openssl's ciphers(1)
              manual.

:Type: String
:Default: None


``tcp_nodelay``

:描述: If set the socket option will disable Nagle's algorithm on 
              the connection which means that packets will be sent as soon 
              as possible instead of waiting for a full buffer or timeout to occur.

              ``1`` Disable Nagel's algorithm for all sockets.

              ``0`` Keep the default: Nagel's algorithm enabled.
:类型: Integer (0 or 1)
:默认值: 0


``max_connection_backlog``

:描述: Optional value to define the maximum size for the queue of
              connections waiting to be accepted. If not configured, the value
              from ``boost::asio::socket_base::max_connections`` will be used.
:类型: Integer
:默认值: None


``request_timeout_ms``

:Description: The amount of time in milliseconds that Beast will wait
              for more incoming data or outgoing data before giving up.
              Setting this value to 0 will disable timeout.

:Type: Integer
:Default: ``65000``


通用选项
========
.. Generic Options

有些前端选项是通用的，所有前端都支持：

``prefix``

:描述: A prefix string that is inserted into the URI of all
              requests. For example, a swift-only frontend could supply
              a uri prefix of ``/swift``.

:类型: String
:默认值: None


.. _配置参考: ../config-ref
