=======================
 Librbd 的 Python 接口
=======================

.. highlight:: python

`rbd` 的 python 模块为 RBD 映像提供了类似文件的访问方法。


实例：创建一映像并写入
======================

要使用 `rbd` ，必须先连接 RADOS 并打开 IO 上下文： ::

    cluster = rados.Rados(conffile='my_ceph.conf')
    cluster.connect()
    ioctx = cluster.open_ioctx('mypool')

然后实例化 :class:rbd.RBD 对象，用它来创建映像： ::

    rbd_inst = rbd.RBD()
    size = 4 * 1024**3  # 4 GiB
    rbd_inst.create(ioctx, 'myimage', size)

要在映像上进行 I/O 操作，需实例化 :class:rbd.Image 对象： ::

    image = rbd.Image(ioctx, 'myimage')
    data = 'foo' * 200
    image.write(data, 0)

上面的代码向映像前面写入了 600 字节的 foo 字符串。注意数据不能是 :type:unicode ， \
librbd 不能如何处理大于 :c:type:char 的字符串。

最后，关闭映像、 IO 上下文、和到 RADOS 的连接。 ::

    image.close()
    ioctx.close()
    cluster.shutdown()

安全起见，每个调用都应该封装到单独的 :finally 块内。 ::

    cluster = rados.Rados(conffile='my_ceph_conf')
    try:
        ioctx = cluster.open_ioctx('my_pool')
        try:
            rbd_inst = rbd.RBD()
            size = 4 * 1024**3  # 4 GiB
            rbd_inst.create(ioctx, 'myimage', size)
            image = rbd.Image(ioctx, 'myimage')
            try:
                data = 'foo' * 200
                image.write(data, 0)
            finally:
                image.close()
        finally:
            ioctx.close()
    finally:
        cluster.shutdown()

这样做有些繁琐，所以 :class:`Rados` 、 :class:`Ioctx` 和 :class:`Image` 类可以当\
上下文管理器来用，它能自动关闭（见 :pep:`343` ）。当上下文管理器用时，上面的实例可\
以写成： ::

    with rados.Rados(conffile='my_ceph.conf') as cluster:
        with cluster.open_ioctx('mypool') as ioctx:
            rbd_inst = rbd.RBD()
            size = 4 * 1024**3  # 4 GiB
            rbd_inst.create(ioctx, 'myimage', size)
            with rbd.Image(ioctx, 'myimage') as image:
                data = 'foo' * 200
                image.write(data, 0)

API 参考
========

.. automodule:: rbd
    :members: RBD, Image, SnapIterator
