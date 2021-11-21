====================
 开发者指南（快速）
====================

本指南将解说如何构建并测试用于开发的 Ceph 。

开发
----

``run-make-check.sh`` 脚本会安装 Ceph 依赖，一切都在调试模式下\
编译、并进行一系列测试，以验证结果正如所需。

.. prompt:: bash $

   ./run-make-check.sh

Optionally if you want to work on a specific component of Ceph,
install the dependencies and build Ceph in debug mode with required cmake flags.

Example:

.. prompt:: bash $

   ./install-deps.sh
   ./do_cmake.sh -DWITH_MANPAGE=OFF -DWITH_BABELTRACE=OFF -DWITH_MGR_DASHBOARD_FRONTEND=OFF

You can also turn off building of some core components that are not relevant to
your development:

.. prompt:: bash $

   ./do_cmake.sh ... -DWITH_RBD=OFF -DWITH_KRBD=OFF -DWITH_RADOSGW=OFF

Finally, build ceph:

.. prompt:: bash $

   cmake --build build [--target <target>...]

Omit ``--target...`` if you want to do a full build.


开发集群的部署
--------------
.. Running a development deployment

Ceph 包含一个名为 ``vstart.sh`` 的脚本（还有\ \
:doc:`/dev/dev_cluster_deployement`\ ），可以让开发者们在开发\
系统上用最简部署快速地测试代码。编译成功后，用下列命令开始部署：

.. prompt:: bash $

   cd build
   ../src/vstart.sh -d -n

你也可以让 ``vstart.sh`` 只用一个监视器和一个元数据服务器，用\
下列命令：

.. prompt:: bash $

   env MON=1 MDS=1 ../src/vstart.sh -d -n -x

Most logs from the cluster can be found in ``build/out``.

这个系统启动时创建了两个存储池： `cephfs_data_a` 和
`cephfs_metadata_a` ，我们看看当前存储池的统计信息：

.. code-block:: console

  $ bin/ceph osd pool stats
  *** DEVELOPER MODE: setting PATH, PYTHONPATH and LD_LIBRARY_PATH ***
  pool cephfs_data_a id 1
    nothing is going on
	
  pool cephfs_metadata_a id 2
    nothing is going on
	
  $ bin/ceph osd pool stats cephfs_data_a
  *** DEVELOPER MODE: setting PATH, PYTHONPATH and LD_LIBRARY_PATH ***
  pool cephfs_data_a id 1
    nothing is going on

  $ bin/rados df
  POOL_NAME         USED OBJECTS CLONES COPIES MISSING_ON_PRIMARY UNFOUND DEGRADED RD_OPS RD WR_OPS WR
  cephfs_data_a        0       0      0      0                  0       0        0      0  0      0    0
  cephfs_metadata_a 2246      21      0     63                  0       0        0      0  0     42 8192

  total_objects    21
  total_used       244G
  total_space      1180G


创建个存储池，并给它做个压力测试：

.. prompt:: bash $

   bin/ceph osd pool create mypool
   bin/rados -p mypool bench 10 write -b 123

放一个文件到新存储池里：

.. prompt:: bash $

   bin/rados -p mypool put objectone <somefile>
   bin/rados -p mypool put objecttwo <anotherfile>

罗列存储池内的对象：

.. prompt:: bash $

   bin/rados -p mypool ls

工作结束后，可以用下面的停止 Ceph 开发环境：

.. prompt:: bash $

   ../src/stop.sh

vstart 环境的重置
-----------------
.. Resetting your vstart environment

vstart 脚本会创建 out/ 和 dev/ 目录，集群的状态就保存在里面。\
如果你想快速重置环境，也许可以这样：

.. prompt:: bash [build]$

   ../src/stop.sh
   rm -rf out dev
   env MDS=1 MON=1 OSD=3 ../src/vstart.sh -n -d

部署 RadosGW 开发环境
---------------------
.. Running a RadosGW development environment

运行 vstart.sh 时设置 ``RGW`` 变量即可启用 RadosGW 。

.. prompt:: bash $

   cd build
   RGW=1 ../src/vstart.sh -d -n -x

现在你可以用 swift python 客户端与 RadosGW 通讯了。

.. prompt:: bash $

   swift -A http://localhost:8000/auth -U test:tester -K testing list
   swift -A http://localhost:8000/auth -U test:tester -K testing upload mycontainer ceph
   swift -A http://localhost:8000/auth -U test:tester -K testing list


运行单元测试
------------

The tests are located in `src/tests`.  To run them type:

.. prompt:: bash $

   (cd build && ninja check)
