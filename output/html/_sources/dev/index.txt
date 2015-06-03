================
 内部开发者文档
================

.. note:: 如果你要找的是如何对接 Ceph 和你自己的软件，那么你应该去看这个文档： \
   :doc:`/api/index` 。

编译完之后，你可以用以下命令启动开发者模式的 Ceph 集群： ::

	cd src
	install -d -m0755 out dev/osd0
	./vstart.sh -n -x -l
	# check that it's there
	./ceph health

.. todo:: vstart is woefully undocumented and full of sharp sticks to poke yourself with.


.. _mailing-list:

.. rubric:: 邮件列表

官方的开发邮件列表是 ``ceph-devel@vger.kernel.org`` ，把下面这行内容发送到 \
``majordomo@vger.kernel.org`` 可订阅： ::

	subscribe ceph-devel

要作为邮件正文发出。


.. rubric:: 内容目录

.. toctree::
   :glob:

   *
   osd_internals/index*
   mds_internals/index*
   radosgw/index*
