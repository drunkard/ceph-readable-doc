.. _python_swift:

=================
Python Swift 实例
=================


新建连接
========

这里先建立一个连接，这样你就能与服务器交互了：

.. code-block:: python

	import swiftclient
	user = 'account_name:username'
	key = 'your_api_key'

	conn = swiftclient.Connection(
		user=user,
		key=key,
		authurl='https://objects.dreamhost.com/auth',
	)


创建容器
========

下面创建一个名为 ``my-new-container`` 的新容器：

.. code-block:: python

	container_name = 'my-new-container'
	conn.put_container(container_name)


创建对象
========

下面从名为 ``my_hello.txt`` 的本地文件创建一个名为 ``hello.txt`` 的文件：

.. code-block:: python

	with open('my_hello.txt', 'r') as hello_file:
		conn.put_object(container_name, 'hello.txt',
				contents=hello_file.read(),
				content_type='text/plain')


罗列自己拥有的容器
==================

下面获取你拥有的容器列表，并打印容器名：

.. code-block:: python

	for container in conn.get_account()[1]:
		print container['name']

其输出大致如此： ::

   mahbuckat1
   mahbuckat2
   mahbuckat3


罗列一容器的内容
================

获取容器中对象的列表，并打印各对象的名字、文件尺寸、和最后修改时间：

.. code-block:: python

	for data in conn.get_container(container_name)[1]:
		print '{0}\t{1}\t{2}'.format(data['name'], data['bytes'], data['last_modified'])

其输出大致如此： ::

   myphoto1.jpg	251262	2011-08-08T21:35:48.000Z
   myphoto2.jpg	262518	2011-08-08T21:38:01.000Z


检索一个对象
============

下载对象 ``hello.txt`` 并保存为 ``./my_hello.txt`` ：

.. code-block:: python

	obj_tuple = conn.get_object(container_name, 'hello.txt')
	with open('my_hello.txt', 'w') as my_hello:
		my_hello.write(obj_tuple[1])


删除对象
========

删除对象 ``goodbye.txt`` ：

.. code-block:: python

	conn.delete_object(container_name, 'hello.txt')


删除一个容器
============

.. note::

   容器必须是空的！否则请求不会成功！

.. code-block:: python

	conn.delete_container(container_name)

