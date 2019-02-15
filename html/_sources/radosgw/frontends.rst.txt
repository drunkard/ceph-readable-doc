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


选项
----

``port`` and ``ssl_port``

:描述: Sets the listening port number. Can be specified multiple
              times as in ``port=80 port=8000``.

:类型: Integer
:默认值: ``80``


``endpoint`` and ``ssl_endpoint``

:描述: Sets the listening address in the form ``address[:port]``,
              where the address is an IPv4 address string in dotted decimal
              form, or an IPv6 address in hexadecimal notation surrounded
              by square brackets. The optional port defaults to 80 for
              ``endpoint`` and 443 for ``ssl_endpoint``. Can be specified
              multiple times as in ``endpoint=[::1] endpoint=192.168.0.100:8000``.

:类型: Integer
:默认值: None


``ssl_certificate``

:描述: Path to the SSL certificate file used for SSL-enabled endpoints.

:类型: String
:默认值: None


``ssl_private_key``

:描述: Optional path to the private key file used for SSL-enabled
              endpoints. If one is not given, the ``ssl_certificate`` file
              is used as the private key.

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

:描述: Sets the listening port number. For SSL-enabled ports, add an
              ``s`` suffix like ``443s``. To bind a specific IPv4 or IPv6
              address, use the form ``address:port``. Multiple endpoints
              can either be separated by ``+`` as in ``127.0.0.1:8000+443s``,
              or by providing multiple options as in ``port=8000 port=443s``.

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

:描述: Path to a file for access logs. Either full path, or relative
			  to the current working directory. If absent (default), then
			  accesses are not logged.

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

Some frontend options are generic and supported by all frontends:

``prefix``

:描述: A prefix string that is inserted into the URI of all
              requests. For example, a swift-only frontend could supply
              a uri prefix of ``/swift``.

:类型: String
:默认值: None


.. _Civetweb User Manual: https://civetweb.github.io/civetweb/UserManual.html
