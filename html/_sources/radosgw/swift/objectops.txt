==========
 对象操作
==========

对象是个用于存储数据和元数据的容器。容器可以包含很多对象，但是对象的名字必\
须唯一。这个 API 允许客户端创建对象、设置访问权限及元数据、读取对象的数据\
和元数据、以及删除对象。因为此 API 发出的请求是与用户帐户信息相关的，所以\
此 API 内的所有请求都必须认证，除非容器或对象的访问控制权限被故意设置成了\
可公开访问（即允许匿名请求）。


创建或更新对象
==============

要创建新对象，需发送带有 API 版本、帐户、容器名和新对象名的 ``PUT`` 请求。\
还必须有对应容器的写权限才能创建或更新对象。容器内的对象名字必须唯一。 \
``PUT`` 请求不会预先检测，所以你要是没用唯一的名字，此请求就会更新对象。然\
而你可以在对象名中用伪分级语法来区分同名、但位于不同伪分级目录的对象。你可\
以在请求头里加上访问控制和元数据头。


语法
~~~~

::

	PUT /{api version}/{account}/{container}/{object} HTTP/1.1
	Host: {fqdn}
	X-Auth-Token: {auth-token}


请求头
~~~~~~


``ETag``

:描述: 对象内容的 MD5 哈希值，建议设置。
:类型: 字符串
:是否必需: 否


``Content-Type``

:描述: 对象所包含内容的类型。
:类型: 字符串
:是否必需: 否


``Transfer-Encoding``

:描述: 用于标识此对象是否是更大的聚合对象的一部分。
:类型: 字符串
:有效取值: ``chunked``
:是否必需: 否


复制对象
========

对象复制方法允许你在服务器端创建对象副本，这样你就不必先下载、再上传到另一\
个容器或名字了。要把一对象的内容复制到另一对象，你可以发送 ``PUT`` 请求或 \
``COPY`` 请求，要携带 API 版本、帐户、和容器名。发送 ``PUT`` 请求时，请求内\
容为目标容器和对象名，源容器和对象在请求头里设置；对于 ``COPY`` 请求，请求\
内容为源容器和对象，目标容器和对象在请求头里。要复制对象，你必须有写权限；\
目标对象名在其容器内也必须唯一。此请求不会预先检查，所以如果名字不唯一，它\
就会更新目标对象。然而你可以在对象名中用伪分级语法来区分同名、但位于不同伪\
分级目录的对象。你可以在请求头里加上访问控制和元数据头。


语法
~~~~

::

	PUT /{api version}/{account}/{dest-container}/{dest-object} HTTP/1.1
	X-Copy-From: {source-container}/{source-object}
	Host: {fqdn}
	X-Auth-Token: {auth-token}


或者这样：

::

	COPY /{api version}/{account}/{source-container}/{source-object} HTTP/1.1
	Destination: {dest-container}/{dest-object}


请求头
~~~~~~

``X-Copy-From``

:描述: 用于在 ``PUT`` 请求中定义源容器或对象的路径。
:类型: 字符串
:是否必需: 用 ``PUT`` 方法时必需


``Destination``

:描述: 用于在 ``COPY`` 请求中定义目标容器或对象的路径。
:类型: 字符串
:是否必需: 用 ``COPY`` 方法时必需


``If-Modified-Since``

:描述: 如果从源对象的 ``last_modified`` 属性记录的时间起修改过，那就复制。
:类型: 日期
:是否必需: 否


``If-Unmodified-Since``

:描述: 如果从源对象的 ``last_modified`` 属性记录的时间起没有修改过，那就复制。
:类型: 日期
:是否必需: 否


``Copy-If-Match``

:描述: 请求中的 ETag 与源对象的 ETag 属性相同时才复制。
:类型: ETag.
:是否必需: 否


``Copy-If-None-Match``

:描述: 请求中的 ETag 与源对象的 ETag 属性不同时才复制。
:类型: ETag.
:是否必需: 否


删除对象
========

要删除对象，可发送带有 API 版本、帐户、容器和对象名的 ``DELETE`` 请求。此\
帐户必须有容器的写权限，才能删除其内的对象。成功删除对象后，你就能重用对象\
名了。


语法
~~~~

::

	DELETE /{api version}/{account}/{container}/{object} HTTP/1.1
	Host: {fqdn}
	X-Auth-Token: {auth-token}


获取一对象
==========

要获取一对象，需发出带有 API 版本、帐户、容器和对象名的 ``GET`` 请求，而且\
必须有此容器的读权限，才能读取其内的对象。


语法
~~~~

::

	GET /{api version}/{account}/{container}/{object} HTTP/1.1
	Host: {fqdn}
	X-Auth-Token: {auth-token}


请求头
~~~~~~

``range``

:描述: 要获取某一对象内容的一部分，你可以指定字节范围。
:类型: 日期（译者：应为整数？）
:是否必需: 否


``If-Modified-Since``

:描述: 如果从源对象的 ``last_modified`` 属性记录的时间起修改过，那就下载。
:类型: 日期
:是否必需: 否


``If-Unmodified-Since``

:描述: 如果从源对象的 ``last_modified`` 属性记录的时间起没有修改过，那就下载。
:类型: 日期
:是否必需: 否


``Copy-If-Match``

:描述: 请求中的 ETag 与源对象的 ETag 属性相同时才下载。
:类型: ETag.
:是否必需: 否


``Copy-If-None-Match``

:描述: 请求中的 ETag 与源对象的 ETag 属性不同时才下载。
:类型: ETag.
:是否必需: No


响应头
~~~~~~


``Content-Range``

:描述: 此区间表示对象内容的子集。只有在请求头中有 range 字段时才会返回此字段。


获取对象元数据
==============

要查看一对象的元数据，可发送带有 API 版本、帐户、容器和对象名的 ``HEAD`` \
头。你还必须有此容器的读权限才能其内对象的元数据。此请求会返回和获取对象本\
身时相同的头信息，只是不返回对象的数据而已。


语法
~~~~

::

	HEAD /{api version}/{account}/{container}/{object} HTTP/1.1
	Host: {fqdn}
	X-Auth-Token: {auth-token}



增加或更新对象元数据
====================

要给对象增加元数据需发送 ``POST`` 请求，要带上 API 版本、帐户、容器和对象\
名。你还必须有父容器的写权限才能增加或更新元数据。


语法
~~~~

::

	POST /{api version}/{account}/{container}/{object} HTTP/1.1
	Host: {fqdn}
	X-Auth-Token: {auth-token}


请求头
~~~~~~

``X-Object-Meta-{key}``

:描述: 一个用户定义的元数据关键字，其值为任意字符串。
:类型: 字符串
:是否必需: 否
