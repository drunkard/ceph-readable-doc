==========
 故障排除
==========


Mount 5 Error
=============

mount 5 错误通常是 MDS 服务器滞后或崩溃导致的。要确保至少有一个 MDS 是启动且运行\
的，集群也要处于 ``active+healthy`` 状态。


Mount 12 Error
==============

mount 12 错误显示 ``cannot allocate memory`` ，常见于 :term:`Ceph 客户端`\ 和 \
:term:`Ceph 存储集群`\ 版本不匹配。用以下命令检查版本： ::

	ceph -v

如果 Ceph 客户端版本落后于集群，试着升级它： ::

	sudo apt-get update && sudo apt-get install ceph-common 

你也许得卸载、清理和删除 ``ceph-common`` ，然后再重新安装，以确保安装的是最新版。
