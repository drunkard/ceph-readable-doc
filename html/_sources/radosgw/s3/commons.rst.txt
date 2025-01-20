==========
 常见问题
==========
.. Common Entities

.. toctree::
   :maxdepth: -1

桶和主机名
----------
.. Bucket and Host Name

访问桶有两种不同的方法。第一种（首选的）方法把桶当作 URI 中的顶极目录。 ::

	GET /mybucket HTTP/1.1
	Host: cname.domain.com

第二种方法把桶当作虚拟主机名，例如： ::

	GET / HTTP/1.1
	Host: mybucket.cname.domain.com

The second method is deprecated by AWS. See the `Amazon S3 Path Deprecation
Plan`_ for more information.

要配置支持虚拟主机的桶，你可以在 ceph.conf 里设置 ``rgw_dns_name = cname.domain.com`` ，
或者把 ``cname.domain.com`` 加进域组配置的 ``hostnames`` 里面。
域组的配置请参考 `Ceph 对象网关——多站配置`_\ 。

Here is an example of a ``ceph config set`` comamnd that sets ``rgw_dns_name``
to ``cname.domain.com``:

.. prompt:: bash $

   ceph config set client.rgw.<ceph authx client for rgw> rgw_dns_name cname.domain.dom

.. tip:: You can define multiple hostnames directly with the
   :confval:`rgw_dns_name` parameter.

.. tip:: When SSL is enabled, the certificates must use a wildcard in the
   domain name in order to match the bucket subdomains.

.. note:: When Ceph Object Gateways are behind a proxy, use the proxy's DNS
   name instead. Then you can use ``ceph config set client.rgw`` to set the DNS
   name for all instances.

.. note:: The static website view for the `s3website` API must be served under
   a different domain name. This is configured separately from
   :confval:`rgw_dns_name`, in :confval:`rgw_dns_s3website_name`.


常见请求头
----------
.. Common Request Headers

+--------------------+------------------------------------------+
| 请求头             | 描述                                     |
+====================+==========================================+
| ``CONTENT_LENGTH`` | Length of the request body.              |
+--------------------+------------------------------------------+
| ``DATE``           | Request time and date (in UTC).          |
+--------------------+------------------------------------------+
| ``HOST``           | The name of the host server.             |
+--------------------+------------------------------------------+
| ``AUTHORIZATION``  | Authorization token.                     |
+--------------------+------------------------------------------+

常见响应状态
------------
.. Common Response Status

+---------------+-----------------------------------+
| HTTP 状态     | 响应代码                          |
+===============+===================================+
| ``100``       | Continue                          |
+---------------+-----------------------------------+
| ``200``       | Success                           |
+---------------+-----------------------------------+
| ``201``       | Created                           |
+---------------+-----------------------------------+
| ``202``       | Accepted                          |
+---------------+-----------------------------------+
| ``204``       | NoContent                         |
+---------------+-----------------------------------+
| ``206``       | Partial content                   |
+---------------+-----------------------------------+
| ``304``       | NotModified                       |
+---------------+-----------------------------------+
| ``400``       | InvalidArgument                   |
+---------------+-----------------------------------+
| ``400``       | InvalidDigest                     |
+---------------+-----------------------------------+
| ``400``       | BadDigest                         |
+---------------+-----------------------------------+
| ``400``       | InvalidBucketName                 |
+---------------+-----------------------------------+
| ``400``       | InvalidObjectName                 |
+---------------+-----------------------------------+
| ``400``       | UnresolvableGrantByEmailAddress   |
+---------------+-----------------------------------+
| ``400``       | InvalidPart                       |
+---------------+-----------------------------------+
| ``400``       | InvalidPartOrder                  |
+---------------+-----------------------------------+
| ``400``       | RequestTimeout                    |
+---------------+-----------------------------------+
| ``400``       | EntityTooLarge                    |
+---------------+-----------------------------------+
| ``403``       | AccessDenied                      |
+---------------+-----------------------------------+
| ``403``       | UserSuspended                     |
+---------------+-----------------------------------+
| ``403``       | RequestTimeTooSkewed              |
+---------------+-----------------------------------+
| ``404``       | NoSuchKey                         |
+---------------+-----------------------------------+
| ``404``       | NoSuchBucket                      |
+---------------+-----------------------------------+
| ``404``       | NoSuchUpload                      |
+---------------+-----------------------------------+
| ``405``       | MethodNotAllowed                  |
+---------------+-----------------------------------+
| ``408``       | RequestTimeout                    |
+---------------+-----------------------------------+
| ``409``       | BucketAlreadyExists               |
+---------------+-----------------------------------+
| ``409``       | BucketNotEmpty                    |
+---------------+-----------------------------------+
| ``411``       | MissingContentLength              |
+---------------+-----------------------------------+
| ``412``       | PreconditionFailed                |
+---------------+-----------------------------------+
| ``416``       | InvalidRange                      |
+---------------+-----------------------------------+
| ``422``       | UnprocessableEntity               |
+---------------+-----------------------------------+
| ``500``       | InternalError                     |
+---------------+-----------------------------------+

.. _`Ceph 对象网关——多站配置`: ../../multisite
.. _`Amazon S3 Path Deprecation Plan`: https://aws.amazon.com/blogs/aws/amazon-s3-path-deprecation-plan-the-rest-of-the-story/
