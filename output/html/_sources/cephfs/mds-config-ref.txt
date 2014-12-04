==============
 MDS 配置参考
==============


``mon force standby active`` 

:Description: 如设为 ``true`` ，监视器会把 standby-replay 状态强设为 active 。设\
              置于 ``[mon]`` 或 ``[global]`` 下。

:Type: Boolean
:Default: ``true`` 


``max mds``

:Description: 集群创建时的 MDS 守护进程数量，设置于 ``[mon]`` 或 ``[global]`` 下。              
:Type:  32-bit Integer
:Default: ``1``


``mds max file size``

:Description: 创建一新文件系统时允许的最大文件尺寸。
:Type:  64-bit Integer Unsigned
:Default:  ``1ULL << 40``


``mds cache size``

:Description: 缓存的索引节点数。
:Type:  32-bit Integer
:Default: ``100000``


``mds cache mid``

:Description: 把新条目插入缓存 LRU 时的插入点（从顶端）。
:Type:  Float
:Default: ``0.7``


``mds dir commit ratio``

:Description: 目录的脏到什么程度时做完全更新（而不是部分更新）。
:Type:  Float
:Default: ``0.5``


``mds dir max commit size``

:Description: 一个目录更新超过多大时要拆分为较小的事务（ MB ）。              
:Type:  32-bit Integer
:Default: ``90``


``mds decay halflife``

:Description: MDS 缓存热度的半衰期。
:Type:  Float
:Default: ``5``


``mds beacon interval``

:Description: 标识消息发送给监视器的频率，秒。
:Type:  Float
:Default: ``4``


``mds beacon grace``

:Description: 多久没收到标识消息就认为 MDS 落后了（并可能替换它）。              
:Type:  Float
:Default: ``15``


``mds blacklist interval``

:Description: OSD 运行图里的 MDS 失败后，把它留在黑名单里的时间。
:Type:  Float
:Default: ``24.0*60.0``


``mds session timeout``

:Description: 客户端多久不活跃时， Ceph 就清除其能力和租期。
:Type:  Float
:Default: ``60``


``mds session autoclose``

:Description: 自动关闭空闲会话前的等待时间，秒。
:Type:  Float
:Default: ``300``


``mds reconnect timeout``

:Description: MDS 重启期间客户端等待时间，秒。
:Type:  Float
:Default: ``45``


``mds tick interval``

:Description: MDS 执行内部周期性任务的间隔。
:Type:  Float
:Default: ``5``


``mds dirstat min interval``

:Description: 在目录树中递归地查找目录信息的最小间隔，秒。              
:Type:  Float
:Default: ``1``


``mds scatter nudge interval``

:Description: dirstat 变更传播多快。
:Type:  Float
:Default: ``5``


``mds client prealloc inos``

:Description: 为每个客户端会话预分配的索引节点数。
:Type:  32-bit Integer
:Default: ``1000``


``mds early reply``

:Description: 请求结果成功提交到日志前是否应该先展示给客户端。
:Type:  Boolean
:Default: ``true``


``mds use tmap``

:Description: 把  trivialmap 用于目录更新。
:Type:  Boolean
:Default: ``true``


``mds default dir hash``

:Description: 用于哈希文件在目录中分布情况的函数。
:Type:  32-bit Integer
:Default: ``2`` (i.e., rjenkins)


``mds log``

:Description: 默认为 ``true`` ， MDS 是否要记录元数据更新（只可在性能评测时禁用）。              
:Type:  Boolean
:Default: ``true``


``mds log skip corrupt events``

:Description: 在日志重放时， MDS 是否应跳过损坏的日志事件。              
:Type:  Boolean
:Default:  ``false``


``mds log max events``

:Description: 日志里的事件达到多少时开始裁剪，设为 ``-1`` 取消限制。              
:Type:  32-bit Integer
:Default: ``-1``


``mds log max segments``

:Description: 日志里的片段（对象）达到多少时开始裁剪，设为 ``-1`` 取消限制。
:Type:  32-bit Integer
:Default: ``30``


``mds log max expiring``

:Description: 可同时过期的片段数。
:Type:  32-bit Integer
:Default: ``20``


``mds log eopen size``

:Description: 在一个 EOpon 事件中最大索引节点数。
:Type:  32-bit Integer
:Default: ``100``


``mds bal sample interval``

:Description: 对目录热度取样的频率（碎片粒度）。              
:Type:  Float
:Default: ``3``


``mds bal replicate threshold``

:Description: 达到多大热度时 Ceph 就把元数据复制到其它节点。              
:Type:  Float
:Default: ``8000``


``mds bal unreplicate threshold``

:Description: 热度低到多少时 Ceph 就不再把元数据复制到其它节点。              
:Type:  Float
:Default: ``0``


``mds bal frag``

:Description: MDS 是否应该给目录分片。
:Type:  Boolean
:Default:  ``false``


``mds bal split size``

:Description: 目录尺寸大到多少时 MDS 就把片段拆分成更小的片段。              
:Type:  32-bit Integer
:Default: ``10000``


``mds bal split rd``

:Description: 目录的最大读取热度达到多大时 Ceph 将拆分此片段。              
:Type:  Float
:Default: ``25000``


``mds bal split wr``

:Description: 目录的最大写热度达到多大时 Ceph 将拆分此片段。              
:Type:  Float
:Default: ``10000``


``mds bal split bits``

:Description: 把一个目录片段再分割成多大。
:Type:  32-bit Integer
:Default: ``3``


``mds bal merge size``

:Description: 目录尺寸小到多少时 Ceph 就把它合并到邻近目录片段。              
:Type:  32-bit Integer
:Default: ``50``


``mds bal merge rd``

:Description: 读热度低于此值时 Ceph 将合并邻近目录片段。
:Type:  Float
:Default: ``1000``


``mds bal merge wr``

:Description: 写热度低于此值时 Ceph 就合并邻近目录片段。              
:Type:  Float
:Default: ``1000``


``mds bal interval``

:Description: MDS 服务器负荷交换频率，秒。
:Type:  32-bit Integer
:Default: ``10``


``mds bal fragment interval``

:Description: 邻近目录分片频率，秒。
:Type:  32-bit Integer
:Default: ``5``


``mds bal idle threshold``

:Description: 热度低于此值时 Ceph 把子树迁移回父节点。              
:Type:  Float
:Default: ``0``


``mds bal max``

:Description: 均衡器迭代到此数量时 Ceph 就停止（仅适用于测试）。
:Type:  32-bit Integer
:Default: ``-1``


``mds bal max until``

:Description: 均衡器运行多久就停止（仅适用于测试）。
:Type:  32-bit Integer
:Default: ``-1``


``mds bal mode``

:Description: 计算 MDS 负载的方法。

              - ``1`` = 混合；
              - ``2`` = 请求速率和延时；
              - ``3`` = CPU 负载。
              
:Type:  32-bit Integer
:Default: ``0``


``mds bal min rebalance``

:Description: 子树热度最小多少时开始迁移。
:Type:  Float
:Default: ``0.1``


``mds bal min start``

:Description: 子树热度最小多少时 Ceph 才去搜索。
:Type:  Float
:Default: ``0.2``


``mds bal need min``

:Description: 接受的最小目标子树片段。
:Type:  Float
:Default: ``0.8``


``mds bal need max``

:Description: 目标子树片段的最大尺寸。
:Type:  Float
:Default: ``1.2``


``mds bal midchunk``

:Description: 尺寸大于目标子树片段的子树， Ceph 将迁移它。              
:Type:  Float
:Default: ``0.3``


``mds bal minchunk``

:Description: 尺寸小于目标子树片段的子树， Ceph 将忽略它。              
:Type:  Float
:Default: ``0.001``


``mds bal target removal min``

:Description: Ceph 从 MDS 运行图中剔除旧数据前，均衡器至少递归多少次。              
:Type:  32-bit Integer
:Default: ``5``


``mds bal target removal max``

:Description: Ceph 从MDS运行图中剔除旧数据前，均衡器最多递归多少次。
:Type:  32-bit Integer
:Default: ``10``


``mds replay interval``

:Description: MDS 处于 standby-replay 模式（热备）下时的日志滚动间隔。
:Type:  Float
:Default: ``1``


``mds shutdown check``

:Description: MDS 关闭期间缓存更新间隔。
:Type:  32-bit Integer
:Default: ``0``


``mds thrash exports``

:Description: Ceph 会把子树随机地在节点间迁移。（仅用于测试）
:Type:  32-bit Integer
:Default: ``0``


``mds thrash fragments``

:Description: Ceph 会随机地分片或合并目录。
:Type:  32-bit Integer
:Default: ``0``


``mds dump cache on map``

:Description: Ceph 会把各 MDSMap 的 MDS 缓存内容转储到一文件。
:Type:  Boolean
:Default:  ``false``


``mds dump cache after rejoin``

:Description: Ceph 重新加入缓存（恢复期间）后会把 MDS 缓存内容转储到一文件。              
:Type:  Boolean
:Default:  ``false``


``mds verify scatter``

:Description: Ceph 将把各种传播/聚集常量声明为true（仅适合开发者）。
:Type:  Boolean
:Default:  ``false``


``mds debug scatterstat``

:Description: Ceph 将把各种递归统计常量声明为 ``true`` （仅适合开发者）。
:Type:  Boolean
:Default:  ``false``


``mds debug frag``

:Description: Ceph 将在方便时校验目录分段（仅适合开发者）。
:Type:  Boolean
:Default:  ``false``


``mds debug auth pins``

:Description: 常量 debug auth pin （仅适合开发者）。
:Type:  Boolean
:Default:  ``false``


``mds debug subtrees``

:Description: 常量 debug subtree （仅适合开发者）。
:Type:  Boolean
:Default:  ``false``


``mds kill mdstable at``

:Description: Ceph 将向 MDSTable 代码注入 MDS 失败事件（仅适合开发者）。
:Type:  32-bit Integer
:Default: ``0``


``mds kill export at``

:Description: Ceph 将向子树出口代码注入 MDS 失败事件（仅适合开发者）。
:Type:  32-bit Integer
:Default: ``0``


``mds kill import at``

:Description: Ceph 将向子树入口代码注入 MDS 失败事件（仅适合开发者）。
:Type:  32-bit Integer
:Default: ``0``


``mds kill link at``

:Description: Ceph 将向硬链接代码注入 MDS 失败事件（仅适合开发者）。
:Type:  32-bit Integer
:Default: ``0``


``mds kill rename at``

:Description: Ceph 将向重命名代码注入 MDS 失败事件（仅适合开发者）。
:Type:  32-bit Integer
:Default: ``0``


``mds wipe sessions``

:Description: Ceph 将在启动时删除所有客户端会话（仅适合测试）。              
:Type:  Boolean
:Default: ``0``


``mds wipe ino prealloc``

:Description: Ceph 将在启动时删除索引节点号预分配元数据（仅适合测试）。
:Type:  Boolean
:Default: ``0``


``mds skip ino``

:Description: 启动时要跳过的索引节点号数量（仅适合测试）。
:Type:  32-bit Integer
:Default: ``0``


``mds standby for name``

:Description: 指定一 MDS 守护进程的名字，此进程将作为它的候补。
:Type:  String
:Default: N/A


``mds standby for rank``

:Description: 此 MDS 将作为本机架上 MDS 守护进程的候补。
:Type:  32-bit Integer
:Default: ``-1``


``mds standby replay``

:Description: 决定一 ``ceph-mds`` 守护进程是否应该滚动并重放活跃 MDS 的日志（热备）。
:Type:  Boolean
:Default:  ``false``
