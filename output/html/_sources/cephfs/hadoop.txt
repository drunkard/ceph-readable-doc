=========================
 在 CephFS 上使用 Hadoop
=========================

Ceph 文件系统可作为 Hadoop 文件系统（ HDFS ）的落地式替代品，本章描述了 Ceph 用于 \
Hadoop 存储的安装和配置过程。


依赖关系
========

* CephFS 的 Java 接口
* Hadoop 的 CephFS 插件

.. important:: 当前要求 Hadoop 1.1.x 稳定版系列。


安装
====

在 CephFS 上使用 Hadoop 有三个必要条件。首先，必须有一个运行的 Ceph 。建设一个 \
Ceph 集群和文件系统不是本章的讨论范围，请参考 Ceph 安装文档。

另外两个必要条件是 Hadoop 安装和 Ceph 文件系统 Java 软件包，包括 Java 的 CephFS \
Hadoop 插件。后续步骤分别是把依赖添加到 Hadoop 的环境变量 ``CLASSPATH`` 、更改 \
Hadoop 配置让它使用 Ceph 文件系统。


CephFS Java 软件包
------------------

* CephFS Hadoop 插件 (`hadoop-cephfs.jar <http://ceph.com/download/hadoop-cephfs.jar>`_)

安装 Hadoop 时的依赖取决于你怎么部署的，一般来说，这些依赖必须安装到 Hadoop 集群的\
各节点、且必须能从 ``CLASSPATH`` 里找到。典型做法是把另加的 ``jar`` 文件放入 \
``hadoop/lib`` 目录、或编辑 ``hadoop-env.sh`` 中的 ``HADOOP_CLASSPATH`` 变量。

Hadoop 集群内的各参与节点都必须安装原生 Ceph 文件系统客户端。


Hadoop 配置
===========

本段描述了用于控制 Ceph 的 Hadoop 配置选项，这些选项应该写于 Hadoop 配置文件 \
`conf/core-site.xml` 。

+---------------------+--------------------------+----------------------------+
|Property             |Value                     |Notes                       |
|                     |                          |                            |
+=====================+==========================+============================+
|fs.default.name      |Ceph URI                  |ceph://[monaddr:port]/      |
|                     |                          |                            |
|                     |                          |                            |
+---------------------+--------------------------+----------------------------+
|ceph.conf.file       |Local path to ceph.conf   |/etc/ceph/ceph.conf         |
|                     |                          |                            |
|                     |                          |                            |
|                     |                          |                            |
+---------------------+--------------------------+----------------------------+
|ceph.conf.options    |Comma separated list of   |opt1=val1,opt2=val2         |
|                     |Ceph configuration        |                            |
|                     |key/value pairs           |                            |
|                     |                          |                            |
+---------------------+--------------------------+----------------------------+
|ceph.root.dir        |Mount root directory      |Default value: /            |
|                     |                          |                            |
|                     |                          |                            |
+---------------------+--------------------------+----------------------------+
|ceph.mon.address     |Monitor address           |host:port                   |
|                     |                          |                            |
|                     |                          |                            |
|                     |                          |                            |
+---------------------+--------------------------+----------------------------+
|ceph.auth.id         |Ceph user id              |Example: admin              |
|                     |                          |                            |
|                     |                          |                            |
|                     |                          |                            |
+---------------------+--------------------------+----------------------------+
|ceph.auth.keyfile    |Ceph key file             |                            |
|                     |                          |                            |
|                     |                          |                            |
|                     |                          |                            |
+---------------------+--------------------------+----------------------------+
|ceph.auth.keyring    |Ceph keyring file         |                            |
|                     |                          |                            |
|                     |                          |                            |
|                     |                          |                            |
+---------------------+--------------------------+----------------------------+
|ceph.object.size     |Default file object size  |Default value (64MB):       |
|                     |in bytes                  |67108864                    |
|                     |                          |                            |
|                     |                          |                            |
+---------------------+--------------------------+----------------------------+
|ceph.data.pools      |List of Ceph data pools   |Default value: default Ceph |
|                     |for storing file.         |pool.                       |
|                     |                          |                            |
|                     |                          |                            |
+---------------------+--------------------------+----------------------------+
|ceph.localize.reads  |Allow reading from file   |Default value: true         |
|                     |replica objects           |                            |
|                     |                          |                            |
|                     |                          |                            |
+---------------------+--------------------------+----------------------------+


对每文件定制副本数的支持
------------------------

Hadoop 文件系统接口允许用户在创建文件时定制复制因子（例如每块 3 个副本）。然而对象\
复制因子在 Ceph 里是以每存储池为单位进行控制的，并且 Ceph 文件系统默认会包含一个预\
配置的存储池。因此，在 Ceph 之上运行 Hadoop 时，为支持每文件复制策略，必须创建多个\
有非默认复制因子的存储池，且必须配置 Hadoop 让它自动选择合适存储池。

额外的数据存储池用 ``ceph.data.pools`` 指定，此选项的值是逗号分隔的一溜存储池名\
字。此选项被忽略或为空时将使用默认存储池。例如，下面的配置会使 Hadoop 在为文件选择\
存储池时考虑 ``pool1`` 、 ``pool2`` 、 ``pool5`` ： ::

	<property>
	  <name>ceph.data.pools</name>
	  <value>pool1,pool2,pool5</value>
	</property>

Hadoop 不会自动创建存储池。要创建一个指定复制因子的存储池，可用 \
``ceph osd pool create`` 命令、然后用 ``ceph osd pool set`` 设置存储池的 \
``size`` 属性。更多的创建、配置手册见 `RADOS 存储池文档`_\ 。

.. _RADOS 存储池文档: ../../rados/operations/pools

存储池创建、配置完毕后，必须把新存储池可用于数据存储的消息告知元数据服务，用 \
``ceph mds add_data_pool`` 命令告知，这样存储池就可存储文件系统数据了。

首先创建存储池。本例中，我们创建 ``hadoop1`` 存储池，其复制因子为 1 。 ::

	ceph osd pool create hadoop1 100
	ceph osd pool set hadoop1 size 1

下一步，找出存储池 ID ，命令为 ``ceph osd dump`` 。例如，找出刚创建的 \
``hadoop1`` 存储池： ::

	ceph osd dump | grep hadoop1

输出应该类似： ::

	pool 3 'hadoop1' rep size 1 min_size 1 crush_ruleset 0...

其中， ``3`` 是存储池 id 。下面我们用前述 ID 把存储池注册为数据存储池，用于存储文件\
系统数据。 ::

	ceph mds add_data_pool 3

最后配置 Hadoop ，让它在为新文件选择目标存储池时考虑此存储池。 ::

	<property>
		<name>ceph.data.pools</name>
		<value>hadoop1</value>
	</property>


存储池选择规则
~~~~~~~~~~~~~~

下面的规则描述了 Hadoop 如何根据期望复制因子从 ``ceph.data.pools`` 配置的一堆存储\
池中选择一个存储池的规则。

1. 未指定存储池时用 Ceph 的默认 ``data`` 存储池；
2. 复制因子相同时，定制存储池优先于 Ceph 的默认 ``data`` 存储池；
3. 复制因子和期望值相同的存储池会被选择；
4. 否则，选择复制因子和期望值最接近的存储池，或者复制因子最大的。


存储池选择调试
~~~~~~~~~~~~~~

Hadoop 不确定存储池复制因子时会产生日志（如它未被配置为数据存储池），日志消息长相如下： ::

	Error looking up replication of pool: <pool name>

未能选到复制数准确匹配的存储池时 Hadoop 也会产生日志，其长相如下： ::

	selectDataPool path=<path> pool:repl=<name>:<value> wanted=<value>
