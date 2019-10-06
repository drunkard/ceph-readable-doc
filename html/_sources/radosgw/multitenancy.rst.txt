.. RGW Multi-tenancy
.. _rgw-multitenancy:

============
 RGW 多租户
============

.. versionadded:: Jewel

The multi-tenancy feature allows to use buckets and users of the same
name simultaneously by segregating them under so-called ``tenants``.
This may be useful, for instance, to permit users of Swift API to
create buckets with easily conflicting names such as "test" or "trove".

From the Jewel release onward, each user and bucket lies under a tenant.
For compatibility, a "legacy" tenant with an empty name is provided.
Whenever a bucket is referred without an explicit tenant, an implicit
tenant is used, taken from the user performing the operation. Since
the pre-existing users are under the legacy tenant, they continue
to create and access buckets as before. The layout of objects in RADOS
is extended in a compatible way, ensuring a smooth upgrade to Jewel.


.. Administering Users With Explicit Tenants

指定租户内的用户管理
====================

这样的租户本身没有任何操作。在进行用户管理时，他们都是按需\
出现和消失。要想创建、修改、和删除带有租户的用户，使用
``radosgw-admin`` 命令时必须额外加参数 ``--tenant`` 、或者用
``<tenant>$<user>`` 语法指定用户。

实例
----

创建用户 testx$tester ，用于 S3 接口的访问： ::

  # radosgw-admin --tenant testx --uid tester --display-name "Test User" --access_key TESTER --secret test123 user create

创建用户 testx$tester ，用于 Swift 接口的访问： ::

  # radosgw-admin --tenant testx --uid tester --display-name "Test User" --subuser tester:test --key-type swift --access full user create
  # radosgw-admin --subuser 'testx$tester:test' --key-type swift --secret test123

.. note:: 在 shell 里，要指定带租户的子用户时必须加引号。

   租户名只能包含字母、数字、和下划线。


.. Accessing Buckets with Explicit Tenants

访问指定租户的桶
================

When a client application accesses buckets, it always operates with
credentials of a particular user. As mentioned above, every user belongs
to a tenant. Therefore, every operation has an implicit tenant in its
context, to be used if no tenant is specified explicitly. Thus a complete
compatibility is maintained with previous releases, as long as the
referred buckets and referring user belong to the same tenant.
In other words, anything unusual occurs when accessing another tenant's
buckets *only*.

Extensions employed to specify an explicit tenant differ according
to the protocol and authentication system used.

S3
--

In case of S3, a colon character is used to separate tenant and bucket.
Thus a sample URL would be::

  https://ep.host.dom/tenant:bucket

Here's a simple Python sample:

.. code-block:: python
   :linenos:

	from boto.s3.connection import S3Connection, OrdinaryCallingFormat
	c = S3Connection(
		aws_access_key_id="TESTER",
		aws_secret_access_key="test123",
		host="ep.host.dom",
		calling_format = OrdinaryCallingFormat())
	bucket = c.get_bucket("test5b:testbucket")

Note that it's not possible to supply an explicit tenant using
a hostname. Hostnames cannot contain colons, or any other separators
that are not already valid in bucket names. Using a period creates an
ambiguous syntax. Therefore, the bucket-in-URL-path format has to be
used.


.. _Swift with built-in authenticator:

用内置认证器的 Swift 接口
-------------------------

TBD -- not in test_multen.py yet


.. _Swift with Keystone:

用 Keystone 认证的 Swift 接口
-----------------------------

TBD -- don't forget to explain the function of
       rgw keystone implicit tenants = true
       in commit e9259486decab52a362443d3fd3dec33b0ec654f


.. _Notes and known issues:

注意事项和已知问题
------------------

Just to be clear, it is not possible to create buckets in other
tenants at present. The owner of newly created bucket is extracted
from authentication information.

This document needs examples of administration of Keystone users.
The keystone.rst may need to be updated.
