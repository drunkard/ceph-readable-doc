=======================================
 rbd -- 管理 RADOS 块设备（ RBD ）映像
=======================================

.. program:: rbd

提纲
====

| **rbd** [ -c *ceph.conf* ] [ -m *monaddr* ] [ -p | --pool *pool* ] [
  --size *size* ] [ --order *bits* ] [ *command* ... ]


描述
====

**rbd** 是个修改 RBD 映像的工具， QEMU/KVM 通过 Linux 内核驱动和 rbd 存储驱动使\
用 RBD 。 RBD 块设备是很简单的映像，它被分块后存储于 RADOS 对象存储集群，对象的尺\
寸必须是 2 的幂。


选项
====

.. option:: -c ceph.conf, --conf ceph.conf

   指定 ceph.conf 配置文件，而不是用默认的 /etc/ceph/ceph.conf 来确定启动时需要的\
   监视器。

.. option:: -m monaddress[:port]

   连接到指定监视器，无需通过 ceph.conf 查找。

.. option:: -p pool, --pool pool

   在指定存储池下操作，大多数命令都得指定。

.. option:: --no-progress

   不显示进度（有些命令会默认输出到标准输出）。


参数
====

.. option:: --image-format format

   选择用哪个对象布局，默认为 1 。

   * format 1 - 新建 rbd 映像时使用最初的格式。此格式兼容所有版本的 librbd 和内核\
     模块，但是不支持较新的功能，像克隆。

   * format 2 - 使用第二版 rbd 格式， librbd 和 3.11 版以上内核模块支持（除非是分\
     拆的模块）。此格式增加了克隆支持，使得扩展更容易，还允许以后增加新功能。

.. option:: --size size-in-mb

   指定新 rbd 映像的尺寸， MB 。

.. option:: --order bits

   指定对象尺寸。用位数表示，即对象大小为 ``1 << order`` ，默认为 22 （ 4MB ）。

.. option:: --stripe-unit size-in-bytes

   指定条带单元尺寸，字节数。详情见下面的条带化一段。

.. option:: --stripe-count num

   条带化要至少跨越多少对象才能转回第一个。详情见条带化一节。

.. option:: --snap snap

   某些操作需要指定快照名。

.. option:: --id username

   指定 map 命令要用到的用户名（不含 ``client.`` 前缀）。

.. option:: --keyfile filename

   为 map 命令指定一个包含密钥的文件。如果没指定，默认使用 ``client.admin`` 。

.. option:: --keyring filename

   因 map 命令所需，指定一个用户及其密钥文件。如果未指定，从默认密钥环里找。

.. option:: --shared tag

   `lock add` 命令的选项，它允许使用同一标签的多个客户端同时锁住同一映像。标签是任\
   意字符串。当某映像必须从多个客户端同时打开时，此选项很有用，像迁移活动虚拟机时、\
   或者在集群文件系统下使用时。

.. option:: --format format

   指定输出格式，默认： plain 、 json 、 xml 。

.. option:: --pretty-format

   使 json 或 xml 格式的输出更易读。

.. option:: -o map-options, --options map-options

   映射到映像时所用的选项。格式为逗号分隔的字符串选项（类似于 mount(8) 的挂载选\
   项）。详情见下一段的 map 选项。

.. option:: --read-only

   以只读方式映射到映像，等价于 -o ro 。


命令
====

.. TODO rst "option" directive seems to require --foo style options, parsing breaks on subcommands.. the args show up as bold too

:command:`ls` [-l | --long] [pool-name]
  列出 rbd_directory 对象中的所有 rbd 映像。加 -l 选项后也显示快照，并用长格式输\
  出，包括大小、父映像（若是克隆品）、格式等等。

:command:`info` [*image-name*]
  显示指定 rbd 映像的信息（如大小和顺序）。若映像是克隆品，会显示相关父快照；若指定\
  了快照，会显示是否被保护。

:command:`create` [*image-name*]
  如要新建 rbd 映像，必须用 --size 指定尺寸。 --strip-unit 和 --strip-count 参数\
  是可选项，但必须一起用。

:command:`clone` [*parent-snapname*] [*image-name*]
  创建一个父快照的克隆品（写时复制子映像）。若不指定，对象顺序将与父映像完全一样。尺\
  寸和父快照一样。

  父快照必须已被保护（见 `rbd snap protect` ）。 format 2 格式的映像才支持。

:command:`flatten` [*image-name*]
  如果映像是个克隆品，从父快照拷贝所有共享块，并使子快照独立于父快照、切断父子快照间\
  的链接。如果没有克隆品引用此父快照了，就可以取消保护并删除。

  只适用于 format 2 。

:command:`children` [*image-name*]
  列出此映像指定快照的克隆品。它会检查各存储池、并输出存储池名/映像名。

  只适用于 format 2 。

:command:`resize` [*image-name*] [--allow-shrink]
  rbd 大小调整。尺寸参数必须指定； --allow-shrink 选项允许缩小。

:command:`rm` [*image-name*]
  删除一 rbd 映像，包括所有数据块。如果映像有快照，此命令会失效。

:command:`export` [*image-name*] [*dest-path*]
  把映像导出到目的路径，用 - （短线）输出到标准输出。

:command:`import` [*path*] [*dest-image*]
  创建一映像，并从目的路径导入数据，用 - （短线）从标准输入导入。如果可能的话，导入\
  操作会试着创建稀疏映像。如果从标准输入导入，稀疏化单位将是目标映像的数据块尺寸\
  （即 1<<order ）。

:command:`export-diff` [*image-name*] [*dest-path*] [--from-snap *snapname*]
  导出一映像的增量差异，用-导出到标准输出。若给了起始快照，就只包含与此快照的差异部\
  分；否则包含映像的所有数据部分；结束快照用 --snap 选项或 @snap （见下文）指定。此\
  映像的差异格式包含了映像尺寸变更的元数据、起始和结束快照，它高效地表达了被忽略或映\
  像内的全 0 区域。

:command:`import-diff` [*src-path*] [*image-name*]
  导入一映像的增量差异并应用到当前映像。如果此差异是在起始快照基础上生成的，我们会先\
  校验那个已存在快照再继续；如果指定了结束快照，我们先检查它是否存在、再应用变更，结\
  束后再创建结束快照。

:command:`diff` [*image-name*] [--from-snap *snapname*]
  打印出从指定快照点起、或从映像创建点起，映像内的变动区域。输出的各行都包含起始偏移\
  量（按字节）、数据块长度（按字节）、还有 zero 或 data ，用来指示此范围以前是 0 还\
  是其它数据。

:command:`cp` [*src-image*] [*dest-image*]
  把源映像内容复制进新建的目标映像，目标映像和源映像将有相同的尺寸、顺序和格式。

:command:`mv` [*src-image*] [*dest-image*]
  映像改名。注：不支持跨存储池。

:command:`snap` ls [*image-name*]
  列出一映像内的快照。

:command:`snap` create [*image-name*]
  新建一快照。需指定快照名。

:command:`snap` rollback [*image-name*]
  把指定映像回滚到快照。此动作会递归整个块阵列，并把数据头内容更新到快照版本。

:command:`snap` rm [*image-name*]
  删除指定快照。

:command:`snap` purge [*image-name*]
  删除一映像的所有快照。

:command:`snap` protect [*image-name*]
  保护快照，防删除，这样才能从它克隆（见 `rbd clone` ）。做克隆前必须先保护快照，保\
  护意味着克隆出的子快照依赖于此快照。 `rbd clone` 不能在未保护的快照上操作。

  只适用于 format 2 。

:command:`snap` unprotect [*image-name*]
  取消对快照的保护（撤销 `snap protect` ）。如果还有克隆出的子快照尚在， \
  `snap unprotect` 命令会失效。（注意克隆品可能位于不同于父快照的存储池。）

  只适用于 format 2 。

:command:`map` [*image-name*] [-o | --options *map-options* ] [--read-only]
  通过内核 rbd 模块把指定映像映射到某一块设备。

:command:`unmap` [*device-path*]
  取消通过内核 rbd 模块的映射。

:command:`showmapped`
  显示通过内核 rbd 模块映射过的 rbd 映像。

:command:`lock` list [*image-name*]
  显示锁着映像的锁，第一列是 `lock remove` 可以使用的锁名。

:command:`lock` add [*image-name*] [*lock-id*]
  为映像加锁，锁标识是用户一己所好的任意名字。默认加的是互斥锁，也就是说如果已经加过\
  锁的话此命令会失败； --shared 选项会改变此行为。注意，加锁操作本身不影响除加锁之\
  外的任何操作，也不会保护对象、防止它被删除。

:command:`lock` remove [*image-name*] [*lock-id*] [*locker*]
  释放映像上的锁。锁标识和其持有者来自 lock ls 。

:command:`bench-write` [*image-name*] --io-size [*io-size-in-bytes*] --io-threads [*num-ios-in-flight*] --io-total [*total-bytes-to-write*]
  生成一系列顺序写来衡量写吞吐量和延时。默认参数为 --io-size 4096 、 --io-threads \
  16 、 --io-total 1GB 。


映像名
======

除了 --pool 和 --snap 选项之外，映像名还能包含存储池名和快照名，其格式如下： ::

	[pool/]image-name[@snap]

因此包含斜杠（ / ）的映像名显式地指定了存储池名。


条带化
======

RBD 映像被条带化为很多对象，然后存储到 Ceph 分布式对象存储（ RADOS ）集群中。因此，\
到此映像的读和写请求会被分布到集群内的很多节点，也因此避免了映像巨大或繁忙时可能出现\
的单节点瓶颈。

条带化由三个参数控制：

.. option:: order

   条带化产生的对象尺寸是 2 的幂，即 2^[*order*] 字节。默认为 22 ，或 4 MB 。

.. option:: stripe_unit

   各条带单位是连续的字节，相邻地存储于同一对象，用满再去下一个对象。

.. option:: stripe_count

   我们把 [*stripe_unit*] 个字节写够 [*stripe_count*] 个对象后，再转回到第一个对\
   象写另一轮条带，直到达到对象的最大尺寸（由 [*order*] 影响）。此时，我们再用下一\
   轮 [*stripe_count*] 个对象。

默认情况下， [*stripe_unit*] 和对象尺寸相同、且 [*stripe_count*] 为 1 ；另外指定 \
[*stripe_unit*] 需 STRIPINGV2 功能（ Ceph 0.53 起加入）并使用 format 2 格式的映像。


Map 选项
========

这里的大多数选项主要适用于调试和压力测试。默认值设置于内核中，因此还与所用内核的版本\
有关。

* fsid=aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee - 应该由客户端提供的 FSID 。

* ip=a.b.c.d[:p] - IP 还有客户端可选的端口。

* share - 允许与其它映射共享客户端例程（默认）。

* noshare - 禁止与其它映射共享客户端例程。

* crc - 启用在写入数据时计算 CRC32C 校验值（默认）。

* nocrc - 在写入数据时不计算 CRC32C 校验值。

* osdkeepalive=x - OSD 保持连接的期限（默认为 5 秒）。

* osd_idle_ttl=x - OSD 闲置 TTL （默认为 60 秒）。

* rw - 以读写方式映射映像（默认）。

* ro - 以只读方式映射映像，等价于 --read-only 。


实例
====

要新建一 100GB 的 rbd 映像： ::

	rbd -p mypool create myimage --size 102400

或者这样： ::

	rbd create mypool/myimage --size 102400

用个非默认对象尺寸，8 MB： ::

	rbd create mypool/myimage --size 102400 --order 23

删除一 rbd 映像（谨慎啊！）： ::

	rbd rm mypool/myimage

新建快照： ::

	rbd snap create mypool/myimage@mysnap

创建已保护快照的写时复制克隆： ::

	rbd clone mypool/myimage@mysnap otherpool/cloneimage

查看快照有哪些克隆品： ::

	rbd children mypool/myimage@mysnap

删除快照： ::

	rbd snap rm mypool/myimage@mysnap

启用 cephx 时通过内核映射一映像： ::

	rbd map mypool/myimage --id admin --keyfile secretfile

取消映像映射： ::

	rbd unmap /dev/rbd0

创建一映像及其克隆品： ::

	rbd import --image-format 2 image mypool/parent
	rbd snap create --snap snapname mypool/parent
	rbd snap protect mypool/parent@snap
	rbd clone mypool/parent@snap otherpool/child

新建一 stripe_unit 较小的映像（在某些情况下可更好地分布少量写）： ::

	rbd -p mypool create myimage --size 102400 --stripe-unit 65536 --stripe-count 16

更改一映像的格式，先导出、再导入为期望格式： ::

	rbd export mypool/myimage@snap /tmp/img
	rbd import --image-format 2 /tmp/img mypool/myimage2

互斥地锁住一映像： ::

	rbd lock add mypool/myimage mylockid

释放锁： ::

	rbd lock remove mypool/myimage mylockid client.2485


使用范围
========

**rbd** 是 Ceph 分布式文件系统的一部分，更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`ceph <ceph>`\(8),
:doc:`rados <rados>`\(8)
