====================
 开发者指南（快速）
====================

本指南将解说如何构建并测试用于开发的 Ceph 。

开发
----

``run-make-check.sh`` 脚本会安装 Ceph 依赖，一切都在调试模式下编译、并进\
行一系列测试，以验证结果正如所需。

.. code::

	$ ./run-make-check.sh


开发集群的部署
--------------

Ceph 包含一个名为 ``vstart.sh`` 的脚本（还有\ \
:doc:`/dev/dev_cluster_deployement`\ ），可以让开发者们在开发系统上用\
最简部署快速地测试代码。编译成功后，用下列命令开始部署：

.. code::

	$ cd src
	$ ./vstart.sh -d -n -x

You can also configure ``vstart.sh`` to use only one monitor and one metadata server by using the following:

.. code::

	$ MON=1 MDS=1 ./vstart.sh -d -n -x

The system creates three pools on startup: `cephfs_data`, `cephfs_metadata`, and `rbd`.  Let's get some stats on
the current pools:

.. code::

	$ ./ceph osd pool stats
	*** DEVELOPER MODE: setting PATH, PYTHONPATH and LD_LIBRARY_PATH ***
	pool rbd id 0
	  nothing is going on

	pool cephfs_data id 1
	  nothing is going on

	pool cephfs_metadata id 2
	  nothing is going on

	$ ./ceph osd pool stats cephfs_data
	*** DEVELOPER MODE: setting PATH, PYTHONPATH and LD_LIBRARY_PATH ***
	pool cephfs_data id 1
	  nothing is going on

	$ ./rados df
	pool name       category                 KB      objects       clones     degraded      unfound     rd        rd KB           wr        wr KB
	rbd             -                          0            0            0            0     0            0            0            0            0
	cephfs_data     -                          0            0            0            0     0            0            0            0            0
	cephfs_metadata -                          2           20            0           40     0            0            0           21            8
	  total used        12771536           20
	  total avail     3697045460
	  total space     3709816996


Make a pool and run some benchmarks against it:

.. code::

	$ ./rados mkpool mypool
	$ ./rados -p mypool bench 10 write -b 123

Place a file into the new pool:

.. code::

	$ ./rados -p mypool put objectone <somefile>
	$ ./rados -p mypool put objecttwo <anotherfile>

List the objects in the pool:

.. code::

	$ ./rados -p mypool ls

Once you are done, type the following to stop the development ceph deployment:

.. code::

	$ ./stop.sh


运行 RadosGW 开发环境
---------------------

Add the ``-r`` to vstart.sh to enable the RadosGW

.. code::

	$ cd src
	$ ./vstart.sh -d -n -x -r

You can now use the swift python client to communicate with the RadosGW.

.. code::

    $ swift -A http://localhost:8000/auth -U tester:testing -K asdf list
    $ swift -A http://localhost:8000/auth -U tester:testing -K asdf upload mycontainer ceph
    $ swift -A http://localhost:8000/auth -U tester:testing -K asdf list


运行单元测试
------------

The tests are located in `src/tests`.  To run them type:

.. code::

	$ make check

