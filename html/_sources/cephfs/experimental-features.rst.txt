==========
实验性功能
==========
.. Experimental Features

CephFS includes a number of experimental features which are not fully
stabilized or qualified for users to turn on in real deployments. We generally
do our best to clearly demarcate these and fence them off so they cannot be
used by mistake.

Some of these features are closer to being done than others, though. We
describe each of them with an approximation of how risky they are and briefly
describe what is required to enable them. Note that doing so will
*irrevocably* flag maps in the monitor as having once enabled this flag to
improve debugging and support processes.


内联数据
--------
.. Inline data

By default, all CephFS file data is stored in RADOS objects. The inline data
feature enables small files (generally <2KB) to be stored in the inode
and served out of the MDS. This may improve small-file performance but increases
load on the MDS. It is not sufficiently tested to support at this time, although
failures within it are unlikely to make non-inlined data inaccessible

Inline data has always been off by default and requires setting
the ``inline_data`` flag.

Inline data has been declared deprecated for the Octopus release, and will
likely be removed altogether in the Q release.


Mantle: 可编程的元数据负载均衡器
--------------------------------
.. Mantle: Programmable Metadata Load Balancer

Mantle is a programmable metadata balancer built into the MDS. The idea is to
protect the mechanisms for balancing load (migration, replication,
fragmentation) but stub out the balancing policies using Lua. For details, see
:doc:`/cephfs/mantle`.


LazyIO
------
LazyIO 放松了 POSIX 语义的要求，允许读、写缓冲的存在，即使有\
多个客户端上的多个应用程序同时打开一个文件时也允许。应用程序\
自行负责缓存一致性的管理。
