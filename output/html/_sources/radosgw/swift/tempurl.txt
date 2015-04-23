===============
 Temp URL 操作
===============

为做到无需共享凭据也能临时允许访问（比如用于 `GET` 请求）对象， radosgw \
末端的 swift 也支持临时 URL 功能。要用此功能，需设置 \
`X-Account-Meta-Temp-URL-Key` 或可选项 `X-Account-Meta-Temp-URL-Key-2` 的\
初始值。 Temp URL 功能需要这些密钥的 HMAC-SHA1 签名。


POST Temp-URL 密钥
==================

向 swift 帐户发送一个带有所需 Key 的 ``POST`` 请求，就能给此帐户设置临时 \
URL 私钥，这样就可向其他帐户提供临时 URL 访问了。最多支持两个密钥，两个密\
钥的签名都要检查，如果没问题，密钥就可以继续使用、临时 URL 也不会失效。


语法
~~~~

::

	POST /{api version}/{account} HTTP/1.1
	Host: {fqdn}
	X-Auth-Token: {auth-token}


请求头
~~~~~~


``X-Account-Meta-Temp-URL-Key``

:描述: 用户定义的密钥，可以是任意字符串。
:类型: String
:是否必需: Yes


``X-Account-Meta-Temp-URL-Key-2``

:描述: 用户定义的密钥，可以是任意字符串。
:类型: String
:是否必需: No


获取 Temp-URL 对象
==================

临时 URL 用密码学 HMAC-SHA1 签名，它包含下列要素：

#. 请求方法的值，如 "GET"
#. 过期时间，格式为纪元以来的秒数，即 Unix 时间
#. 从 "v1" 起的请求路径

上述条目将用新行格式化，并用之前上传的临时 URL 密钥通过 SHA-1 哈希算法生\
成一个 HMAC 。

下面的 Python 脚本演示了上述过程：

.. code-block:: python

	import hmac
	from hashlib import sha1
	from time import time

	method = 'GET'
	host = 'https://objectstore.example.com'
	duration_in_seconds = 300  # Duration for which the url is valid
	expires = int(time() + duration_in_seconds)
	path = '/v1/your-bucket/your-object'
	key = 'secret'
	hmac_body = '%s\n%s\n%s' % (method, expires, path)
	hmac_body = hmac.new(key, hmac_body, sha1).hexdigest()
	sig = hmac.new(key, hmac_body, sha1).hexdigest()
	rest_uri = "{host}{path}?temp_url_sig={sig}&temp_url_expires={expires}".format(
		host=host, path=path, sig=sig, expires=expires)
	print rest_uri

	# Example Output
	# https://objectstore.example.com/v1/your-bucket/your-object?temp_url_sig=ff4657876227fc6025f04fcf1e82818266d022c6&temp_url_expires=1423200992

