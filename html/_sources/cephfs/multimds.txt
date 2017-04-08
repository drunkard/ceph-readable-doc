:orphan:

.. _Configuring multiple active MDS daemons:

多活 MDS 守护进程的配置
-----------------------

*也叫： multi-mds 、 active-active MDS*

每个 CephFS 文件系统默认情况下都只配置一个活跃 MDS 守护进程。\
在大型系统中，为了扩展元数据性能你可以配置多个活跃的 MDS 守护\
进程，它们会共同承担元数据负载。


.. _When should I use multiple active MDS daemons?:

什么情况下我需要多个活跃的 MDS 守护进程？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

当元数据默认的单个 MDS 成为瓶颈时，你应该配置多个活跃的 MDS 守\
护进程。

增加守护进程不一定都能提升性能，要看负载类型。典型地，单个客户\
端上的单个应用程序就不会受益于 MDS 守护进程的增加，除非这个应\
用程序是在并行地操作元数据。

通常，有很多客户端（操作着很多不同的目录时更好）时，大量活跃的
MDS 守护进程有利于性能提升。


.. _Increasing the MDS active cluster size:

MDS 活跃集群的扩容
~~~~~~~~~~~~~~~~~~

每一个 CephFS 文件系统都有自己的 *max_mds* 配置，它控制着会创\
建多少 rank 。有空闲守护进程可接管新 rank 时，文件系统 rank 的\
实际数量才会增加，比如只有一个 MDS 守护进程运行着、 max_mds 被\
设置成了 2 ，此时不会创建第二个 rank 。

把 ``max_mds`` 设置为想要的 rank 数量。在下面的例子里，
``ceph status`` 输出的 fsmap 行是此命令可能输出的结果。 ::

    # fsmap e5: 1/1/1 up {0=a=up:active}, 2 up:standby

    ceph fs set max_mds 2

    # fsmap e8: 2/2/2 up {0=a=up:active,1=c=up:creating}, 1 up:standby
    # fsmap e9: 2/2/2 up {0=a=up:active,1=c=up:active}, 1 up:standby

新创建的 rank (1) 会从 creating 状态过渡到 active 状态。


.. _Standby daemons:

灾备守护进程
~~~~~~~~~~~~

即使拥有多活 MDS 守护进程，一个高可用系统\ *仍然需要灾备守护进\
程*\ 来顶替失效的活跃守护进程。

因此，高可用系统的 ``max_mds`` 实际最大值比系统中 MDS 服务器的\
总数小一。

为了在多个服务器失效时仍能保持可用，需增加系统中的灾备守护进\
程，以弥补你能承受的服务器失效数量。


.. _Decreasing the number of ranks:

减少 rank 数量
~~~~~~~~~~~~~~

所有的 rank ，包括你要删除的 rank(s) ，必须先是活跃的。也就是\
说，你必须有至少 max_mds 个 MDS 守护进程可用。

首先，把 max_mds 设置为较小的数字，比如我们想回到单活 MDS ： ::
    
    # fsmap e9: 2/2/2 up {0=a=up:active,1=c=up:active}, 1 up:standby
    ceph fs set max_mds 1
    # fsmap e10: 2/2/1 up {0=a=up:active,1=c=up:active}, 1 up:standby

注意，此时我们仍然有两个活跃 MDS ，即使降低了 max_mds 两个 rank
还都在，因为 max_mds 只能限制新 rank 的创建。

接下来，用 ``ceph mds deactivate <rank>`` 命令删除不需要的
rank ： ::

    ceph mds deactivate cephfs_a:1
    telling mds.1:1 172.21.9.34:6806/837679928 to deactivate

    # fsmap e11: 2/2/1 up {0=a=up:active,1=c=up:stopping}, 1 up:standby
    # fsmap e12: 1/1/1 up {0=a=up:active}, 1 up:standby
    # fsmap e13: 1/1/1 up {0=a=up:active}, 2 up:standby

被取消活跃状态的 rank 会先进入 stopping 状态，并持续一段时间，\
在此期间它要把它分享的那部分元数据让出给活跃 MDS 守护进程，这\
个过程可能要持续数秒到数分钟。如果这个 MDS 卡在了 stopping 状\
态，那可能是触发了软件缺陷。

如果处于 stopping 状态的 MDS 守护进程崩溃、或是被杀死了，就会\
有一个灾备顶替它，这个 rank 会回到 active 状态。等它回来后，你\
可以再次实施去激活操作。

守护进程完成 stopping 状态后，它会自己重生并成为灾备。


