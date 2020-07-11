.. HTTP Frontends
.. _rgw_frontends:

===========
 HTTP 前端
===========

.. contents::

Ceph 对象网关支持两个嵌入式的 HTTP 前端库，可以用
``rgw_frontends`` 配置。


Beast
=====

.. versionadded:: Mimic

The ``beast`` frontend uses the Boost.Beast library for HTTP parsing
and the Boost.Asio library for asynchronous network i/o.


.. Options

选项
----

``port`` 和 ``ssl_port``

:描述: 设置监听端口号。可以指定多次，如 ``port=80 port=8000`` 。
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
:类型: String
:默认值: None


``ssl_private_key``

:描述: 可选配置，私钥文件的路径，用于启用了 SSL 的终结点。如果\
       没配置此路径，就把 ``ssl_certificate`` 文件当作私钥用。
:类型: String
:默认值: None


Civetweb
========

.. versionadded:: Firefly

The ``civetweb`` frontend uses the Civetweb HTTP library, which is a
fork of Mongoose.


选项
----

``port``

:描述: 设置监听端口号。对于启用了 SSL 的端口，加个 ``s``
       后缀，如 ``443s`` 。要绑定某个特定的 IPv4 或 IPv6
       地址，按照 ``address:port`` 格式；多个终结点可以用 ``+``
       分隔（如 ``127.0.0.1:8000+443s`` ）或写多个选项（如
       ``port=8000 port=443s`` ）。
:类型: String
:默认值: ``7480``


``num_threads``

:描述: Sets the number of threads spawned by Civetweb to handle
              incoming HTTP connections. This effectively limits the number
              of concurrent connections that the frontend can service.

:类型: Integer
:默认值: ``rgw_thread_pool_size``


``request_timeout_ms``

:描述: The amount of time in milliseconds that Civetweb will wait
              for more incoming data before giving up.

:类型: Integer
:默认值: ``30000``


``ssl_certificate``

:描述: Path to the SSL certificate file used for SSL-enabled ports.

:类型: String
:默认值: None

``access_log_file``

:描述: 访问日志的文件路径。可以是完整路径、或当前工作目录的\
       相对路径。如果未设置（默认的），就不会记录访问日志。

:类型: String
:默认值: ``EMPTY``


``error_log_file``

:描述: Path to a file for error logs. Either full path, or relative
			  to the current working directory. If absent (default), then
			  errors are not logged.

:类型: String
:默认值: ``EMPTY``


The following is an example of the ``/etc/ceph/ceph.conf`` file with some of these options set: ::
 
 [client.rgw.gateway-node1]
 rgw_frontends = civetweb 
 request_timeout_ms = 30000 
 error_log_file = /var/log/radosgw/civetweb.error.log 
 access_log_file = /var/log/radosgw/civetweb.access.log

A complete list of supported options can be found in the `Civetweb User Manual`_.


.. Generic Options

通用选项
========

有些前端选项是通用的，所有前端都支持：

``prefix``

:描述: A prefix string that is inserted into the URI of all
              requests. For example, a swift-only frontend could supply
              a uri prefix of ``/swift``.

:类型: String
:默认值: None


.. _Civetweb User Manual: https://civetweb.github.io/civetweb/UserManual.html
