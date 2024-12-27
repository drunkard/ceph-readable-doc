Mantle
======

.. warning::

    Mantle 的初衷是用于研究和开发元数据均衡算法，
    不是给 CephFS 生产集群用的。

Mantle 是内置于 MDS 的可编程元数据均衡器。

默认情况下（没有 Mantle 时），多个活跃 MDS 可以迁移目录，
以均衡元数据载荷。决定何时、何地、迁移多少的策略\
是硬编码在元数据均衡模块中的。

Mantle 工作方式是保护负载均衡的机制
（迁移、复制、分片），同时用 Lua 脚本抑制均衡策略。
Mantle 基于 [1] ，
但目前的实现还不具备该论文说的以下功能：

#. 均衡 API ：在该论文中，用户需要填写何时、何地、
   多少以及负载计算策略。目前，
   Mantle 只要求 Lua 策略返回一张目标负载表
   （例如，向每个 MDS 分配多少负载）。

#. “多少”钩子：在论文中，有一个钩子允许用户控制
   “分片选择器策略（ fragment selector policy ）”。
   目前， Mantle 没有这个钩子。

#. 作为指标的“ CPU 瞬时利用率”。

[1] Supercomputing '15 论文：
http://sc15.supercomputing.org/schedule/event_detail-evid=pap168.html

vstart 快速入门
---------------
.. Quickstart with vstart

.. warning::

   使用 vstart 开发均衡器很困难，因为在一个节点上运行\
   所有守护进程和客户端会使系统超载。让系统运行一段时间，
   即使可能会出现许多心跳丢失警告和许多 MDS 滞后警告。
   大多数情况下，本指南都有用，但有时在使用 vstart 开发时，
   所有 MDS 都会锁死，实际上看不到它们溢出。最好在多节点集群上运行。

作为前提条件，我们假设您已经安装了 `mdtest <https://sourceforge.net/projects/mdtest/>`_
或已经拉取了 `Docker 镜像 <https://hub.docker.com/r/michaelsevilla/mdtest/>`_ 。
我们使用 mdtest ，是因为需要产生足够的负载，才能超过均衡器中\
任意设置的 MIN_OFFLOAD 阈值。例如，下面的不会产生足够的元数据负载：

::

    while true; do
      touch "/cephfs/blah-`date`"
    done


Mantle 和 `vstart.sh` 调试
~~~~~~~~~~~~~~~~~~~~~~~~~~
.. Mantle with `vstart.sh`

1. 启动 Ceph 并调整日志记录，我们才能看到是否有迁移：

::

    cd build
    ../src/vstart.sh -n -l
    for i in a b c; do 
      bin/ceph --admin-daemon out/mds.$i.asok config set debug_ms 0
      bin/ceph --admin-daemon out/mds.$i.asok config set debug_mds 2
      bin/ceph --admin-daemon out/mds.$i.asok config set mds_beacon_grace 1500
    done


2. 把均衡器放进 RADOS:

::

    bin/rados put --pool=cephfs_metadata_a greedyspill.lua ../src/mds/balancers/greedyspill.lua


3. 激活 Mantle:

::

    bin/ceph fs set cephfs max_mds 5
    bin/ceph fs set cephfs_a balancer greedyspill.lua


4. 在另一个窗口，挂载 CephFS ：

::

     bin/ceph-fuse /cephfs -o allow_other &
     tail -f out/mds.a.log


.. note:: 注意，如果查看最后一个 MDS （可能是 a 、 b 或 c --
   它是随机的。你会看到索引零值的尝试。这是因为\
   最后一个 MDS 试图检查其邻居的负载，而邻居并不存在。

5. 运行个简单的压力测试。我们用的是 Docker mdtest 映像，用它产生负载：

::

    for i in 0 1 2; do
      docker run -d \
        --name=client$i \
        -v /cephfs:/cephfs \
        michaelsevilla/mdtest \
        -F -C -n 100000 -d "/cephfs/client-test$i"
    done


6. 结束后，用下列命令清理客户端：

::

    for i in 0 1 2 3; do docker rm -f client$i; done


输出解析
~~~~~~~~
.. Output

查看第一个 MDS （可能是 a 、 b 或 c ）的日志，看到大家都没有负载：

::

    2016-08-21 06:44:01.763930 7fd03aaf7700  0 lua.balancer MDS0: < auth.meta_load=0.0 all.meta_load=0.0 req_rate=1.0 queue_len=0.0 cpu_load_avg=1.35 > load=0.0
    2016-08-21 06:44:01.763966 7fd03aaf7700  0 lua.balancer MDS1: < auth.meta_load=0.0 all.meta_load=0.0 req_rate=0.0 queue_len=0.0 cpu_load_avg=1.35 > load=0.0
    2016-08-21 06:44:01.763982 7fd03aaf7700  0 lua.balancer MDS2: < auth.meta_load=0.0 all.meta_load=0.0 req_rate=0.0 queue_len=0.0 cpu_load_avg=1.35 > load=0.0
    2016-08-21 06:44:01.764010 7fd03aaf7700  2 lua.balancer when: not migrating! my_load=0.0 hisload=0.0
    2016-08-21 06:44:01.764033 7fd03aaf7700  2 mds.0.bal  mantle decided that new targets={}


作业开始后， MDS0 有大概 1953 个单位的负载。
激进的分配均衡器决定，将一半的负载分配给邻居 MDS ，
因此我们看到 Mantle 尝试将 1953 个负载单位发送到 MDS1。

::

    2016-08-21 06:45:21.869994 7fd03aaf7700  0 lua.balancer MDS0: < auth.meta_load=5834.188908912 all.meta_load=1953.3492228857 req_rate=12591.0 queue_len=1075.0 cpu_load_avg=3.05 > load=1953.3492228857
    2016-08-21 06:45:21.870017 7fd03aaf7700  0 lua.balancer MDS1: < auth.meta_load=0.0 all.meta_load=0.0 req_rate=0.0 queue_len=0.0 cpu_load_avg=3.05 > load=0.0
    2016-08-21 06:45:21.870027 7fd03aaf7700  0 lua.balancer MDS2: < auth.meta_load=0.0 all.meta_load=0.0 req_rate=0.0 queue_len=0.0 cpu_load_avg=3.05 > load=0.0
    2016-08-21 06:45:21.870034 7fd03aaf7700  2 lua.balancer when: migrating! my_load=1953.3492228857 hisload=0.0
    2016-08-21 06:45:21.870050 7fd03aaf7700  2 mds.0.bal  mantle decided that new targets={0=0,1=976.675,2=0}
    2016-08-21 06:45:21.870094 7fd03aaf7700  0 mds.0.bal    - exporting [0,0.52287 1.04574] 1030.88 to mds.1 [dir 100000006ab /client-test2/ [2,head] auth pv=33 v=32 cv=32/0 ap=2+3+4 state=1610612802|complete f(v0 m2016-08-21 06:44:20.366935 1=0+1) n(v2 rc2016-08-21 06:44:30.946816 3790=3788+2) hs=1+0,ss=0+0 dirty=1 | child=1 dirty=1 authpin=1 0x55d2762fd690]
    2016-08-21 06:45:21.870151 7fd03aaf7700  0 mds.0.migrator nicely exporting to mds.1 [dir 100000006ab /client-test2/ [2,head] auth pv=33 v=32 cv=32/0 ap=2+3+4 state=1610612802|complete f(v0 m2016-08-21 06:44:20.366935 1=0+1) n(v2 rc2016-08-21 06:44:30.946816 3790=3788+2) hs=1+0,ss=0+0 dirty=1 | child=1 dirty=1 authpin=1 0x55d2762fd690]


最终，负载只是到处移动：

::

    2016-08-21 06:47:10.210253 7fd03aaf7700  0 lua.balancer MDS0: < auth.meta_load=415.77414300449 all.meta_load=415.79000078186 req_rate=82813.0 queue_len=0.0 cpu_load_avg=11.97 > load=415.79000078186
    2016-08-21 06:47:10.210277 7fd03aaf7700  0 lua.balancer MDS1: < auth.meta_load=228.72023977691 all.meta_load=186.5606496623 req_rate=28580.0 queue_len=0.0 cpu_load_avg=11.97 > load=186.5606496623
    2016-08-21 06:47:10.210290 7fd03aaf7700  0 lua.balancer MDS2: < auth.meta_load=0.0 all.meta_load=0.0 req_rate=1.0 queue_len=0.0 cpu_load_avg=11.97 > load=0.0
    2016-08-21 06:47:10.210298 7fd03aaf7700  2 lua.balancer when: not migrating! my_load=415.79000078186 hisload=186.5606496623
    2016-08-21 06:47:10.210311 7fd03aaf7700  2 mds.0.bal  mantle decided that new targets={}


实现细节
--------
.. Implementation Details

大部分实现都在 MDBalancer 里面。度量指标通过 Lua 栈传递给均衡策略，
负载列表则返回给 MDBalancer 。它与当前的均衡器实现并存，可通过 Ceph CLI 命令
（ ``ceph fs set cephfs balancer mybalancer.lua`` ）启用。如果 Lua 策略失效
（无论出于何种原因），将回退到最初的元数据负载均衡器。这个均衡器存储在
RADOS 元数据存储池中， MDSMap 中的字符串会告诉 MDS 使用哪个均衡器。

把指标暴露给 Lua
~~~~~~~~~~~~~~~~
.. Exposing Metrics to Lua

度量指标作为全局变量直接暴露给 Lua 代码，而不是用明确定义的函数符号。
有一个全局 "mds" 表，其中每个索引都是一个 MDS 编号（例如 0），
每个值都是指标和值组成的字典。 Lua 代码可以用类似下面的方法获取指标：

::

    mds[0]["queue_len"]


这与 OSD 中的 cls-lua 形成鲜明对比，后者有明确定义的参数
（例如输入/输出缓冲列表）。直接暴露指标可以更方便地增加新指标，
而无需更改 Lua 这边的 API ；我们想让 API 能随着我们对指标的探索而自如地扩展和收缩。
这种方法的缺点是，做 Lua 均衡策略编程的人员必须查看 Ceph 源代码，
才能知道暴露了哪些指标。我们认为，
Mantle 开发人员无论如何都得接触 MDS 的内部结构。

暴露给 Lua 策略的指标与已存储在 mds_load_t 中的指标相同： auth.meta_load() 、
all.meta_load() 、 req_rate 、 queue_length 、 cpu_load_avg。


编译、执行均衡器
~~~~~~~~~~~~~~~~
.. Compile/Execute the Balancer

这里我们使用 `lua_pcall` 而不是 `lua_call` ，是因为我们想在 MDBalancer 中\
处理错误。我们不希望错误沿着调用链传播出去。 cls_lua 类希望自己处理错误，
因为它必须可控地失败。对于 Mantle 而言，我们不在乎 Lua 错误是否会\
导致均衡器崩溃，如果遇到了，我们就回退到原来的均衡器。

使用 `lua_call` 而不是 `lua_pcall` 带来的性能提升没多大影响，
因为均衡器默认每 10 秒调用一次。


把策略的决策返回给 C++
~~~~~~~~~~~~~~~~~~~~~~
.. Returning Policy Decision to C++

我们强制 Lua 策略引擎返回一个张表的数值，
这些值对应发送到每个 MDS 的负载量。
这些负载会直接插入 MDBalancer 的 my_targets 向量中。
我们不允许 MDS 返回 MDS 和指标表，
因为我们希望完全由 Lua 做出决策。

遍历 Lua 返回的表是通过堆栈完成的。
用 Lua 的行话说就是：一个假值被推到堆栈上，
下一个迭代器用 (k, v) 对替换堆栈顶部。读取每个值后，
弹出该值，但保留键，用于下一次调用 `lua_next` 。


从 RADOS 读取
~~~~~~~~~~~~~
.. Reading from RADOS

当 MDS 运行图中的均衡器版本变化时，所有 MDS 都将从 RADOS 读取均衡代码。
均衡器从 RADOS 同步地读取 Lua 代码。这个操作带超时限制：
如果异步读取没有在半个均衡时间间隔内返回，操作就会被取消，
并返回 Connection Timeout （连接超时）错误。默认情况下，
均衡时间间隔为 10 秒，这样 Mantle 的超时时间是 5 秒。
这种设计会让 Mantle 在任何与 RADOS 相关的操作出错时立即返回错误信息。

我们用这种实现方式，是因为我们不想在全局 MDS 锁内进行阻塞式 OSD 读取。
如果有哪个 OSD 没有响应，这样做会导致 MDS 集群宕机 --
这在 ceph-qa-suite 中测试过了，把所有 OSD 设置为 down/out ，并确保 MDS 集群保持活跃。

一种方法是在处理 MDS 运行图时异步触发读取，并在后台装入 Lua 代码。
我们不能这样做，因为 MDS 不支持守护进程本地回退，
而且均衡器假定所有 MDS 都在同一时间做出相同的决策（例如，导入器、导出器等）。


调试
~~~~

Lua 策略中的日志将显示在 MDS 日志中。语法与 cls 日志接口相同：

::

    BAL_LOG(0, "this is a log message")


它是通过传递一个函数来实现的，该函数使用 `lua_register()` 原语\
将 `dout` 日志框架( `dout_wrapper` ) 封装到 Lua 中。
Lua 代码实际上是在调用 C++ 中的 `dout` 函数。

Warning 和 Info 消息通过 clog/Beacon 集中处理。
成功消息仅在第一个 MDS 进行版本变更时发送，
以避免对 ``ceph -w`` 工具产生垃圾信息。这些消息用于集成测试。


测试
~~~~

测试使用 ceph-qa-suite (tasks.cephfs.test_mantle) 完成。
我们不测试无效的平衡器日志记录和实际的 Lua VM 加载。
