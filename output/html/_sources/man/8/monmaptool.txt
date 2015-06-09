:orphan:

=========================================
 monmaptool -- ceph 监视器运行图操作工具
=========================================

.. program:: monmaptool

提纲
====

| **monmaptool** *mapfilename* [ --clobber ] [ --print ] [ --create ]
  [ --add *ip*:*port* *...* ] [ --rm *ip*:*port* *...* ]


描述
====

**monmaptool** 工具用于创建、查看、修改用于 Ceph 分布式存储系统的监视器集群\
运行图。监视器图只是在 Ceph 分布式系统中定义了几个固定的地址，其他所有守护进\
程绑定到任意地址、并注册到监视器。

用 --create 选项创建新图时，会创建新的随机 UUID ，此选项后还应加一或多个监视\
器地址。

Ceph 监视器的默认端口是 6789 。


选项
====

.. option:: --print

   在所有修改完成后，打印一份监视器图的纯文本转储。

.. option:: --clobber

   更改时允许 monmaptool 覆盖 mapfilename 。

.. option:: --create

   新建一监视器图，它有新的 UUID （用它可创建个新的空 Ceph 文件系统）。

.. option:: --generate

   基于命令行参数或配置文件中的配置生成新 monmap ，配置来源优先级如下：

      #. ``--monmap filename`` 指定要载入的 monmap
      #. ``--mon-host 'host1,ip2'`` 指定一系列主机或 IP 地址
      #. 配置文件中包含 ``mon addr`` 选项的 ``[mon.foo]`` 段落

.. option:: --filter-initial-members

   用 ``mon initial members`` 选项的设置过滤初始 monmap ，不在此列表内的监视\
   器将被删除、没在图内的初始成员将用假地址加入。

.. option:: --add name ip:port

   把指定 ip:port 的监视器加入图中。

.. option:: --rm name

   从图中删除 ip:port 监视器。

.. option:: --fsid uuid

    把 fsid 设置为指定的 uuid ，如果 --create 时没指定，将会随机生成一个。


实例
====

新建一个有三个监视器的新图（为新的 Ceph 文件系统）： ::

        monmaptool  --create  --add  mon.a 192.168.0.10:6789 \
          --add mon.b 192.168.0.11:6789 \
          --add mon.c 192.168.0.12:6789 --clobber monmap

显示监视器图内容： ::

        monmaptool --print monmap

替换一个监视器： ::

        monmaptool --rm mon.a --add mon.a 192.168.0.9:6789 --clobber monmap


使用范围
========

**monmaptool** 是 Ceph 的一部分，这是个伸缩力强、开源、分布式的存储系统，\
更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`ceph <ceph>`\(8),
:doc:`crushtool <crushtool>`\(8),
