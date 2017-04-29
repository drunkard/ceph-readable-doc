==============
 安装（快速）
==============

.. raw:: html

	<style type="text/css">div.body h3{margin:5px 0px 0px 0px;}</style>
	<table cellpadding="10"><colgroup><col width="33%"><col width="33%"><col width="33%"></colgroup><tbody valign="top"><tr><td><h3>步骤一：飞前</h3>

:term:`Ceph 客户端`\ 和 :term:`Ceph 节点`\ 需要些基本预配置才能部署 Ceph 存储集\
群，你也可以加入 Ceph 社区以寻求帮助。

.. toctree::

   飞前 <quick-start-preflight>

.. raw:: html

	</td><td><h3>步骤二：存储集群</h3>

完成飞前检查表之后，应该可以开始部署 Ceph 存储集群了。

.. toctree::

	存储集群入门 <quick-ceph-deploy>


.. raw:: html

	</td><td><h3>步骤三： Ceph 客户端</h3>

大多数 Ceph 用户不会直接往 Ceph 存储集群里存储对象，他们通常会用 Ceph 块设备、 \
Ceph 文件系统、或 Ceph 对象存储三个功能之一或多个。

.. toctree::

   块设备入门 <quick-rbd>
   文件系统入门 <quick-cephfs>
   对象存储入门 <quick-rgw>

.. raw:: html

	</td></tr></tbody></table>
