====================
 开发者指南（快速）
====================

本指南将解说如何构建并测试用于开发的 Ceph 。

开发
----

``run-make-check.sh`` 脚本会安装 Ceph 依赖，一切都在调试模式下\
编译、并进行一系列测试，以验证结果正如所需。

.. code::

	$ ./run-make-check.sh


开发集群的部署
--------------

Ceph 包含一个名为 ``vstart.sh`` 的脚本（还有\ \
:doc:`/dev/dev_cluster_deployement`\ ），可以让开发者们在开发\
系统上用最简部署快速地测试代码。编译成功后，用下列命令开始部署：

.. code::

	$ cd ceph/build  # 假设这是你运行 cmake 的地方
	$ make vstart
	$ ../src/vstart.sh -d -n -x

你也可以让 ``vstart.sh`` 只用一个监视器和一个元数据服务器，用\
下列命令：

.. code::

	$ MON=1 MDS=1 ../src/vstart.sh -d -n -x

这个系统启动时创建了三个存储池： `cephfs_data` 、 `cephfs_metadata`
和 `rbd` ，我们看看当前存储池的统计信息：

.. code::

	$ bin/ceph osd pool stats
	*** DEVELOPER MODE: setting PATH, PYTHONPATH and LD_LIBRARY_PATH ***
	pool rbd id 0
	  nothing is going on

	pool cephfs_data id 1
	  nothing is going on

	pool cephfs_metadata id 2
	  nothing is going on

	$ bin/ceph osd pool stats cephfs_data
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

	$ bin/rados mkpool mypool
	$ bin/rados -p mypool bench 10 write -b 123

放一个文件到新存储池里：

.. code::

	$ bin/rados -p mypool put objectone <somefile>
	$ bin/rados -p mypool put objecttwo <anotherfile>

罗列存储池内的对象：

.. code::

	$ bin/rados -p mypool ls

工作结束后，可以用下面的停止 Ceph 开发环境：

.. code::

	$ ../src/stop.sh


vstart 环境的重置
-----------------

vstart 脚本会创建 out/ 和 dev/ 目录，集群的状态就保存在里面。\
如果你想快速重置环境，也许可以这样：

.. code::

    [build]$ ../src/stop.sh
    [build]$ rm -rf out dev
    [build]$ MDS=1 MON=1 OSD=3 ../src/vstart.sh -n -d


运行 RadosGW 开发环境
---------------------

执行 vstart.sh 时加 ``-r`` 可启用 RadosGW 。

.. code::

	$ cd build
	$ ../src/vstart.sh -d -n -x -r

现在你可以用 swift python 客户端与 RadosGW 通讯了。

.. code::

    $ swift -A http://localhost:8000/auth -U test:tester -K testing list
    $ swift -A http://localhost:8000/auth -U test:tester -K testing upload mycontainer ceph
    $ swift -A http://localhost:8000/auth -U test:tester -K testing list


运行单元测试
------------

The tests are located in `src/tests`.  To run them type:

.. code::

	$ make check

