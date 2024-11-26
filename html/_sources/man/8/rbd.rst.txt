:orphan:

=======================================
 rbd -- 管理 RADOS 块设备（ RBD ）映像
=======================================

.. program:: rbd


提纲
====

| **rbd** [ -c *ceph.conf* ] [ -m *monaddr* ] [--cluster *cluster-name*]
  [ -p | --pool *pool* ] [ *command* ... ]


描述
====

**rbd** 是个修改 rados 块设备（ RBD ）映像的工具，
QEMU/KVM 通过 Linux 内核驱动和 rbd 存储驱动使用 RBD 。
RBD 映像是简单的块设备，
它被条带化成小块对象后存储于 RADOS 对象存储集群，
条带化后的对象尺寸必须是 2 的幂。


选项
====

.. option:: -c ceph.conf, --conf ceph.conf

   指定 ceph.conf 配置文件，而不是用默认的 /etc/ceph/ceph.conf
   来确定启动时需要的监视器。

.. option:: -m monaddress[:port]

   连接到指定监视器，无需通过 ceph.conf 查找。

.. option:: --cluster cluster-name

   使用非默认的集群名字，即不是 *ceph* 的集群名。

.. option:: -p pool-name, --pool pool-name

   在指定存储池下操作，大多数命令都得指定。

.. option:: --namespace namespace-name

   使用存储池内一个预定义的映像命名空间。

.. option:: --no-progress

   不显示进度（有些命令会默认输出到标准输出）。


参数
====

.. option:: --image-format format-id

   选择用哪个对象布局，默认为 2 。

   * format 1 - （已废弃）新建 rbd 映像时使用最初的格式。
     此格式兼容所有版本的 librbd 和内核模块，
     但是不支持较新的功能，像克隆。

   * format 2 - 使用第二 rbd 格式， librbd 从 Bobtail 版起、\
     内核版本在 3.10 以上（ fancy 条带化功能除外，
     4.17 的内核才支持）的内核 rbd 模块才支持此格式。
     此格式增加了克隆支持，
     使得将来扩展新功能更容易。

.. option:: -s size-in-M/G/T, --size size-in-M/G/T

   指定新 rbd 映像、或是已有 rbd 映像的新尺寸，单位可以是
   M/G/T ，没加后缀的话默认为 M 。

.. option:: --object-size size-in-B/K/M

   指定对象尺寸，单位可以是 B/K/M 。对象尺寸将被对齐到最接近的
   2 的幂；如果不指定后缀，则认为单位是 B 。默认的对象尺寸是
   4MB ，最小允许 4K 、最大允许 32M 。

   这个默认值可以用配置选项 ``rbd_default_order`` 更改，
   它是 2 的幂次（默认对象尺寸是 ``2 ^ rbd_default_order`` ）。

.. option:: --stripe-unit size-in-B/K/M

   指定条带单元尺寸，单位可以是 B/K/M ，没加的话默认为 B 。详\
   情见下面的条带化一段。

.. option:: --stripe-count num

   条带化要至少跨越多少对象才能转回第一个。
   详情见条带化一节。

.. option:: --snap snap

   某些操作需要指定快照名。

.. option:: --id username

   指定 map 命令要用到的用户名（不含 ``client.`` 前缀）。

.. option:: --keyring filename

   因 map 命令所需，
   指定一个用户及其密钥文件。
   如果未指定，从默认密钥环里找。

.. option:: --keyfile filename

   因 map 命令所需，给 ``--id user`` 用户指定一个包含密钥的文件。
   如果同时指定了 ``--keyring`` 选项，本选项就会被覆盖。

.. option:: --shared lock-tag

   `lock add` 命令的选项，它允许使用同一标签的多个客户端\
   同时锁住同一映像。标签是任意字符串。
   当某映像必须从多个客户端同时打开时，
   此选项很有用，
   像迁移活动虚拟机时、
   或者在集群文件系统下使用时。

.. option:: --format format

   指定输出格式，默认： plain 、 json 、 xml 。

.. option:: --pretty-format

   使 json 或 xml 格式的输出更易读。

.. option:: -o krbd-options, --options krbd-options

   通过 rbd 内核驱动映射或取消映射某一映像时指定的选项。
   krbd-options 是逗号分隔的一系列选项
   （类似于 mount(8) 的挂载选项）。
   详情见下面的内核 rbd (krbd) 选项一段。

.. option:: --read-only

   以只读方式映射到映像，等价于 -o ro 。

.. option:: --image-feature feature-name

   创建格式 2 的 RBD 映像时，指定要启用哪些功能。
   想要启用多个功能的话，可以多次重复使用此选项。
   当前支持下列功能：

   * layering: 支持分层
   * striping: 支持条带化 v2
   * exclusive-lock: 支持独占锁
   * object-map: 支持对象映射（依赖 exclusive-lock ）
   * fast-diff: 快速计算差异（依赖 object-map ）
   * deep-flatten: 支持快照扁平化操作
   * journaling: 支持记录 IO 操作（依赖独占锁）
   * data-pool: 纠删码存储池支持

.. option:: --image-shared

   指定该映像将被多个客户端同时使用。
   此选项将禁用那些依赖于独占所有权的功能。

.. option:: --whole-object

   把 diff 操作范围限定在完整的对象条带级别，
   而非对象内差异。
   当某一映像启用了 object-map 功能时，
   把 diff 操作限定到对象条带会显著地提高性能，
   因为通过检查驻留于内存中的对象映射就可以计算出差异，
   而无需针对映像内的各个对象查询 RADOS 。

.. option:: --limit

   指定快照的数量上限。


命令
====

.. TODO rst "option" directive seems to require --foo style options, parsing breaks on subcommands.. the args show up as bold too

:command:`bench` --io-type <read | write | readwrite | rw> [--io-size *size-in-B/K/M/G/T*] [--io-threads *num-ios-in-flight*] [--io-total *size-in-B/K/M/G/T*] [--io-pattern seq | rand] [--rw-mix-read *read proportion in readwrite*] *image-spec*
  向指定映像生成一系列 IO 操作，以此衡量 IO 吞吐量和延时。如果\
  不加后缀， --io-size 和 --io-total 的单位就当是 B 。默认参数\
  为 --io-size 4096 、 --io-threads 16 、 --io-total 1G 、 \
  --io-pattern seq 、 --rw-mix-read 50 。

:command:`children` *snap-spec*
  列出此映像指定快照的克隆品。它会检查各存储池、并输出存储池\
  名/映像名。

  只适用于 format 2 。

:command:`clone` [--object-size *size-in-B/K/M*] [--stripe-unit *size-in-B/K/M* --stripe-count *num*] [--image-feature *feature-name*] [--image-shared] *parent-snap-spec* *child-image-spec*
  创建一个父快照的克隆品（写时复制子映像）。
  若不指定，对象尺寸将与父映像完全一样。
  尺寸和父快照一样。
  参数 --stripe-unit 和 --stripe-count 是可选的，但必须同时使用。

  父快照必须已被保护（见 `rbd snap protect` ）。
  format 2 格式的映像才支持。

:command:`config global get` *config-entity* *key*
  查看一条全局级的配置选项覆盖。

:command:`config global list` [--format plain | json | xml] [--pretty-format] *config-entity*
  罗列全局级的配置选项覆盖。

:command:`config global set` *config-entity* *key* *value*
  设置一条全局级的配置选项覆盖。

:command:`config global remove` *config-entity* *key*
  删除一条全局级的配置选项覆盖。

:command:`config image get` *image-spec* *key*
  查看一条映像级的配置选项覆盖。

:command:`config image list` [--format plain | json | xml] [--pretty-format] *image-spec*
  罗列映像级的配置选项覆盖。.

:command:`config image set` *image-spec* *key* *value*
  设置一条映像级的配置选项覆盖。

:command:`config image remove` *image-spec* *key*
  删除一条映像级的配置选项覆盖。

:command:`config pool get` *pool-name* *key*
  查看一条存储池级的配置选项覆盖。

:command:`config pool list` [--format plain | json | xml] [--pretty-format] *pool-name*
  罗列存储池级的配置选项覆盖。.

:command:`config pool set` *pool-name* *key* *value*
  配置一条存储池级的配置选项覆盖。

:command:`config pool remove` *pool-name* *key*
  删除一条存储池级的配置选项覆盖。

:command:`cp` (*src-image-spec* | *src-snap-spec*) *dest-image-spec*
  把源映像内容复制进新建的目标映像，
  目标映像和源映像将有相同的尺寸、对象尺寸和映像格式。
  注意：它的快照没有复制，
  用 `deep cp` 命令包含快照。

:command:`create` (-s | --size *size-in-M/G/T*) [--image-format *format-id*] [--object-size *size-in-B/K/M*] [--stripe-unit *size-in-B/K/M* --stripe-count *num*] [--thick-provision] [--no-progress] [--image-feature *feature-name*]... [--image-shared] *image-spec*
  新建一个 rbd 映像。还必须用 --size 指定尺寸。 --strip-unit 和
  --strip-count 参数是可选项，但必须一起用。如果加了
  --thick-provision 选项，它会在创建时就为映像分配所需的所有\
  存储空间，需要很长时间完成。注意：全配（ thick provisioning
  ）要求把整个映像的内容都清零。

:command:`deep cp` (*src-image-spec* | *src-snap-spec*) *dest-image-spec*
  把 src-image 的内容深复制到新建的 dest-image 。 dest-image
  将会有和 src-image 相同的尺寸、对象尺寸、映像格式、和快照。

:command:`device list` [-t | --device-type *device-type*] [--format plain | json | xml] --pretty-format
  展示通过 rbd 内核模块映射的 rbd 映像（默认的）或其它支持的\
  设备。

:command:`device map` [-t | --device-type *device-type*] [--cookie *device-cookie*] [--show-cookie] [--read-only] [--exclusive] [-o | --options *device-options*] *image-spec* | *snap-spec*
  把指定映像通过 rbd 内核模块映射成一个块设备（默认的）、
  或其它支持的设备
  （ Linux 上的 *nbd* 或 FreeBSD 上的 *ggate* ）。

  --options 参数是个逗号分隔的特定于某类型设备的一系列选项
  （ opt1,opt2=val,... ）。

:command:`device unmap` [-t | --device-type *device-type*] [-o | --options *device-options*] *image-spec* | *snap-spec* | *device-path*
  断开块设备映射，之前通过 rbd 内核模块映射的（默认的）、
  或其它支持的设备。

  --options 参数是个逗号分隔的特定于某类型设备的一系列选项
  （ opt1,opt2=val,... ）。

:command:`device attach` [-t | --device-type *device-type*] --device *device-path* [--cookie *device-cookie*] [--show-cookie] [--read-only] [--exclusive] [--force] [-o | --options *device-options*] *image-spec* | *snap-spec*
  把指定映像捆绑到指定块设备（当前仅适用于 Linux 上的 `nbd` ）。
  此操作不安全，平常不应该使用。
  特别是，指定了错误的映像或错误的块设备可能会导致数据损坏，
  因为 `nbd` 内核驱动不会进行核实。

  --options 参数是一个逗号分隔的、特定于设备类型的选项
  （ opt1,opt2=val,... ）列表。

:command:`device detach` [-t | --device-type *device-type*] [-o | --options *device-options*] *image-spec* | *snap-spec* | *device-path*
  解绑之前映射或绑定（当前仅适用于 Linux 上的 `nbd` ）的块设备。
  此操作不安全，平常不应该使用。

  --options 参数是一个逗号分隔的、特定于设备类型的选项
  （ opt1,opt2=val,... ）列表。

:command:`diff` [--from-snap *snap-name*] [--whole-object] *image-spec* | *snap-spec*
  打印出从指定快照点起、或从映像创建点起，映像内的变动区域。
  输出的各行都包含起始偏移量（按字节）、
  数据块长度（按字节）、还有 zero 或 data ，
  用来指示此范围以前是 0 还是其它数据。

:command:`du` [-p | --pool *pool-name*] [*image-spec* | *snap-spec*] [--merge-snapshots]
  会计算指定存储池内所有映像及其相关快照的磁盘使用量，
  包括分配的和实际使用的。
  此命令也可用于单个映像和快照。

  如果 RBD 映像的 fast-diff 特性没启用，本操作就需要向各个 OSD
  挨个查询此映像涉及的每个潜在对象。

  --merge-snapshots 会把快照占用的空间算到它的父映像头上。

:command:`encryption format` *image-spec* *format* *passphrase-file* [--cipher-alg *alg*]
  把映像格式化成加密格式。
  之前写入此映像的所有数据都将不可读。
  虽然加密的映像可以被克隆，但是克隆的映像不能被格式化。
  支持的格式有： *luks1* 、 *luks2* 。
  支持的加密算法： *aes-128* 、 *aes-256* （默认）。

:command:`export` [--export-format *format (1 or 2)*] (*image-spec* | *snap-spec*) [*dest-path*]
  把映像导出到目的路径，用 - （短线）输出到标准输出。
  --export-format 现在只认 '1' 或 '2' 。格式 2 不仅允许我们导\
  出映像内容，还可以导出快照和其它属性，如 image_order 、功能标志。

:command:`export-diff` [--from-snap *snap-name*] [--whole-object] (*image-spec* | *snap-spec*) *dest-path*
  导出一映像的增量差异，用-导出到标准输出。
  若给了起始快照，就只包含与此快照的差异部分；
  否则包含映像的所有数据部分；
  结束快照用 --snap 选项或 @snap （见下文）指定。
  此映像的差异格式包含了映像尺寸变更的元数据、起始和结束快照，
  它高效地表达了被忽略或映像内的全 0 区域。

:command:`feature disable` *image-spec* *feature-name*...
  禁用指定镜像的某些功能，
  可以一次指定多个功能。

:command:`feature enable` *image-spec* *feature-name*...
  启用指定镜像的某些功能，
  可以一次指定多个功能。

:command:`flatten` *image-spec*
  如果映像是个克隆品，就从父快照拷贝所有共享块，
  并使子快照独立于父快照、切断父子快照间的链接。
  如果没有克隆品引用此父快照了，
  就可以取消保护并删除。

  只适用于 format 2 。

:command:`group create` *group-spec*
  创建一个组。

:command:`group image add` *group-spec* *image-spec*
  把一个映像加入某一组。

:command:`group image list` *group-spec*
  罗列一个组内的映像。

:command:`group image remove` *group-spec* *image-spec*
  删除一个组内的对象。

:command:`group ls` [-p | --pool *pool-name*]
  罗列所有 rbd 组。

:command:`group rename` *src-group-spec* *dest-group-spec*
  重命名一个组。注意：不支持跨存储池重命名。

:command:`group rm` *group-spec*
  删除一个组。

:command:`group snap create` *group-snap-spec*
  创建一个组的快照。

:command:`group snap list` *group-spec*
  罗列一个组的快照。

:command:`group snap rm` *group-snap-spec*
  删除一个组的某一快照。

:command:`group snap rename` *group-snap-spec* *snap-name*
  重命名组的快照。

:command:`group snap rollback` *group-snap-spec*
  把组回滚到某快照。

:command:`image-meta get` *image-spec* *key*
  获取关键字对应的元数据值。

:command:`image-meta list` *image-spec*
  显示此映像持有的元数据。第一列是关键字、第二列是值。

:command:`image-meta remove` *image-spec* *key*
  删除元数据关键字及其值。

:command:`image-meta set` *image-spec* *key* *value*
  设置指定元数据关键字的值，会显示在 `metadata-list` 中。

:command:`import` [--export-format *format (1 or 2)*] [--image-format *format-id*] [--object-size *size-in-B/K/M*] [--stripe-unit *size-in-B/K/M* --stripe-count *num*] [--image-feature *feature-name*]... [--image-shared] *src-path* [*image-spec*]
  创建一映像，并从目的路径导入数据，用 - （短线）从标准输入导入。
  如果可能的话，导入操作会试着创建稀疏映像。
  如果从标准输入导入，稀疏化单位将是目标映像的数据块尺寸
  （即对象尺寸）。

  参数 --stripe-unit 和 --stripe-count 是可选的，
  但必须同时使用。

  --export-format 现在只认 '1' 或 '2' 。格式 2 不仅允许我们导\
  出映像内容，还可以导出快照和其它属性，如 image_order 、功能标志。

:command:`import-diff` *src-path* *image-spec*
  导入一映像的增量差异并应用到当前映像。
  如果此差异是在起始快照基础上生成的，我们会先校验那个已存在快照再继续；
  如果指定了结束快照，我们先检查它是否存在、再应用变更，
  结束后再创建结束快照。

:command:`info` *image-spec* | *snap-spec*
  显示指定 rbd 映像的信息（如大小和对象尺寸）。
  若映像是克隆品，会显示相关父快照；
  若指定了快照，会显示是否被保护。

:command:`journal client disconnect` *journal-spec*
  把映像日志客户端标记为连接已断。

:command:`journal export` [--verbose] [--no-error] *src-journal-spec* *path-name*
  把映像日志导出到指定路径（ ``-`` 导出到标准输出 stdout ）。\
  它可以作为映像日志的备份手段，特别是打算做危险的操作前。

  注意，如果日志损坏严重，此命令有可能失效。

:command:`journal import` [--verbose] [--no-error] *path-name* *dest-journal-spec*
  从指定路径导入映像日志（ ``-`` 从标准输入 stdin 导入）。

:command:`journal info` *journal-spec*
  展示映像日志的信息。

:command:`journal inspect` [--verbose] *journal-spec*
  检查并报告映像日志的结构性错误。

:command:`journal reset` *journal-spec*
  重置映像日志。

:command:`journal status` *journal-spec*
  展示映像日志的状态。

:command:`lock add` [--shared *lock-tag*] *image-spec* *lock-id*
  为映像加锁，锁标识是用户一己所好的任意名字。
  默认加的是互斥锁，也就是说如果已经加过锁的话此命令会失败；
  --shared 选项会改变此行为。
  注意，加锁操作本身不影响除加锁之外的任何操作，
  也不会保护对象、
  防止它被删除。

:command:`lock ls` *image-spec*
  显示锁着映像的锁，
  第一列是 `lock remove` 可以使用的锁名。

:command:`lock rm` *image-spec* *lock-id* *locker*
  释放映像上的锁。
  锁标识和其持有者来自 lock ls 。

:command:`ls` [-l | --long] [*pool-name*]
  列出 rbd_directory 对象中的所有 rbd 映像。\
  加 -l 选项后也会列出快照，并用长格式输出，包括大小、\
  父映像（若是克隆品）、格式等等。

:command:`merge-diff` *first-diff-path* *second-diff-path* *merged-diff-path*
  把两个连续的增量差异合并为单个差异。前一个差异的末尾快照必须\
  与后一个差异的起始快照相同。前一个差异可以是标准输入 - ，合\
  并后的差异可以是标准输出 - ；这样就可以合并多个差异文件，像\
  这样： 'rbd merge-diff first second - | \
  rbd merge-diff - third result' 。\
  注意，当前此命令只支持 stripe_count == 1 这样的源增量差异。

:command:`migration abort` *image-spec*
  取消映像迁移。这个步骤在成功或失败的迁移准备、
  各个迁移执行步骤之后运行，
  并让映像回到它最初（迁移前）的状态。
  目的映像的所有更改都将丢失。

:command:`migration commit` *image-spec*
  提交映像迁移。这个步骤在成功的迁移准备、
  迁移执行的各个步骤、并删除源映像数据后运行。

:command:`migration execute` *image-spec*
  执行映像迁移。这个步骤在成功的迁移准备步骤、
  并把映像数据复制到目的地之后运行。

:command:`migration prepare` [--order *order*] [--object-size *object-size*] [--image-feature *image-feature*] [--image-shared] [--stripe-unit *stripe-unit*] [--stripe-count *stripe-count*] [--data-pool *data-pool*] [--import-only] [--source-spec *json*] [--source-spec-path *path*] *src-image-spec* [*dest-image-spec*]
  准备映像迁移。这是迁移一个映像的第一步，
  即改变映像的位置，格式或其它参数不能动态更改。
  在目标和源的规格一致时，可以忽略 *dest-image-spec* 。
  本步骤之后，源映像将被设置成目标映像的父映像，
  这个映像将可以按照目标规格、
  通过写时复制模式访问。

  映像也可以从一个只读导入源迁移，
  加可选的 *--import-only* 、
  并加上 JSON 编码的 *--source-spec* 或\
  用 *--source-spec-path* 指定一个 JSON 编码的源规格文件路径。


:command:`mirror image demote` *image-spec*
  把 RBD 映像中的主映像降级成非主映像。

:command:`mirror image disable` [--force] *image-spec*
  禁用一个映像的 RBD 镜像服务。
  如果镜像服务是在 ``image`` 模式下\
  在镜像存储池上配置的，
  那该存储池内各个映像的镜像服务就可以显式地禁用。

:command:`mirror image enable` *image-spec* *mode*
  启用一个映像的 RBD 镜像服务。
  如果镜像服务是在 ``image`` 模式下\
  在镜像存储池上配置的，
  那该存储池内各个映像的镜像服务就可以显式地启用。

  镜像映像的模式还可以是 ``journal`` （默认的）
  或 ``snapshot`` 。 ``journal`` 模式依赖
  RBD 的日志记录功能。

:command:`mirror image promote` [--force] *image-spec*
  为 RBD 镜像服务把一个非主映像晋级成主的。

:command:`mirror image resync` *image-spec*
  在 RBD 镜像中，强制重新同步到主映像。

:command:`mirror image status` *image-spec*
  显示一个映像的 RBD 镜像状态。

:command:`mirror pool demote` [*pool-name*]
  把一个存储池内的所有主映像都降级成非主的。
  存储池内每个启用了镜像服务的映像都会被降级。

:command:`mirror pool disable` [*pool-name*]
  默认禁用一个存储池的 RBD 镜像服务。
  某个存储池的镜像服务以这种方式禁用后，
  此存储池内的所有映像的镜像、包括那些\
  显式地启用了镜像服务的也会禁用。

:command:`mirror pool enable` [*pool-name*] *mode*
  启用一个存储池的默认镜像。
  镜像模式可以是 ``pool`` 或 ``image`` 。
  如果配置成了 ``pool`` 模式，存储池内所有\
  开启了日志功能的映像都会被镜像；
  如果配置成了 ``image`` 模式，
  每个映像的镜像功能都需要显式地启用
  （用 ``mirror image enable`` 命令）。

:command:`mirror pool info` [*pool-name*]
  显示出存储池的镜像服务配置信息。
  它包括镜像模式、互联的 UUID 、
  远程集群名字和远程客户端名字。

:command:`mirror pool peer add` [*pool-name*] *remote-cluster-spec*
  给存储池增加一个镜像互联点。
  *remote-cluster-spec* 是 [*远程客户端名*\ @\ ]\ *远程集群名*.

  *远程客户端名* 默认是 client.admin 。

  此命令要求先启用镜像模式。

:command:`mirror pool peer remove` [*pool-name*] *uuid*
  删除一个存储池的镜像互联点，此互联点的 uuid 可以\
  通过 ``mirror pool info`` 获取。

:command:`mirror pool peer set` [*pool-name*] *uuid* *key* *value*
  更新镜像互联点配置信息。
  键（ key ）是 ``client`` 或 ``cluster`` ，
  值对应远程客户端名或远程集群名。

:command:`mirror pool promote` [--force] [*pool-name*]
  把一个存储池内的所有非主映像晋级成主的。
  此存储池里所有启用了镜像服务的映像都会晋级。

:command:`mirror pool status` [--verbose] [*pool-name*]
  显示此存储池内所有被镜像的映像的状态。
  加 --verbose 选项后，还会显示存储池内、
  所有启用了镜像的映像的额外状态细节。

:command:`mirror snapshot schedule add` [-p | --pool *pool*] [--namespace *namespace*] [--image *image*] *interval* [*start-time*]
  预定镜像快照。

:command:`mirror snapshot schedule list` [-R | --recursive] [--format *format*] [--pretty-format] [-p | --pool *pool*] [--namespace *namespace*] [--image *image*]
  罗列镜像快照的预定情况。

:command:`mirror snapshot schedule remove` [-p | --pool *pool*] [--namespace *namespace*] [--image *image*] *interval* [*start-time*]
  删除镜像快照的预定任务。

:command:`mirror snapshot schedule status` [-p | --pool *pool*] [--format *format*] [--pretty-format] [--namespace *namespace*] [--image *image*]
  显示镜像快照的预定状态。

:command:`mv` *src-image-spec* *dest-image-spec*
  映像改名。注：不支持跨存储池。

:command:`namespace create` *pool-name*/*namespace-name*
  在存储池内新建一个映像命名空间。

:command:`namespace list` *pool-name*
  罗列存储池内定义的映像命名空间。

:command:`namespace remove` *pool-name*/*namespace-name*
  从存储池删除一个空的映像命名空间。

:command:`object-map check` *image-spec* | *snap-spec*
  核验对象映射图是否正确。

:command:`object-map rebuild` *image-spec* | *snap-spec*
  为指定映像重建无效的对象映射关系。指定映像快照时，
  将为此快照重建无效的对象映射关系。

:command:`pool init` [*pool-name*] [--force]
  初始化用于 RBD 的存储池。
  新建的存储池必须先初始化才能使用。

:command:`resize` (-s | --size *size-in-M/G/T*) [--allow-shrink] *image-spec*
  rbd 大小调整。尺寸参数必须指定；
  --allow-shrink 选项允许缩小。

:command:`rm` *image-spec*
  删除一 rbd 映像，包括所有数据块。如果此映像有快照，\
  此命令会失效，什么也不会删除。

:command:`snap create` *snap-spec*
  新建一快照。需指定快照名。

:command:`snap limit clear` *image-spec*
  清除先前设置的映像所允许的\
  快照数量上限。

:command:`snap limit set` [--limit] *limit* *image-spec*
  设置一个映像所允许的快照数量上限。

:command:`snap ls` *image-spec*
  列出一映像内的快照。

:command:`snap protect` *snap-spec*
  保护快照，防删除，这样才能从它克隆（见 `rbd clone` ）。
  做克隆前必须先保护快照，
  保护意味着克隆出的子快照依赖于此快照。
  `rbd clone` 不能在未保护的快照上操作。

  只适用于 format 2 。

:command:`snap purge` *image-spec*
  删除一映像的所有未保护快照。

:command:`snap rename` *src-snap-spec* *dest-snap-spec*
  重命名一个快照。注意：不支持跨存储池和跨映像重命名。

:command:`snap rm` [--force] *snap-spec*
  删除指定快照。

:command:`snap rollback` *snap-spec*
  把指定映像回滚到快照。此动作会递归整个块阵列，
  并把数据头内容更新到快照版本。

:command:`snap unprotect` *snap-spec*
  取消对快照的保护（撤销 `snap protect` ）。如果还有克隆出的\
  子快照尚在， `snap unprotect` 命令会失效。（注意克隆品可能\
  位于不同于父快照的存储池。）

  只适用于 format 2 。

:command:`sparsify` [--sparse-size *sparse-size*] *image-spec*
  回收已被清零的映像条带所占的空间。
  默认的稀疏尺寸为 4096 字节，可用 --sparse-size 选项更改，
  但有这些限制条件：它应该是 2 幂、不小于 4096 、
  且不大于映像的对象尺寸。

:command:`status` *image-spec*
  显示映像状态，包括哪个客户端打开着它。

:command:`trash ls` [*pool-name*]
  罗列垃圾桶内的所有条目。

:command:`trash mv` *image-spec*
  把映像移入垃圾桶。所有映像，包括正被克隆件引用的，
  都能被移入垃圾桶，而后删除。

:command:`trash purge` [*pool-name*]
  删除垃圾桶内所有过期的映像。

:command:`trash restore` *image-id*  
  从垃圾桶恢复一个映像。

:command:`trash rm` *image-id* 
  从垃圾桶删除一个映像。如果映像的延期时间尚未满，
  那就不能删除，除非强删。但是正被克隆件引用的、
  或还有快照的删不掉。

:command:`trash purge schedule add` [-p | --pool *pool*] [--namespace *namespace*] *interval* [*start-time*]
  预定一次垃圾清理。

:command:`trash purge schedule list` [-R | --recursive] [--format *format*] [--pretty-format] [-p | --pool *pool*] [--namespace *namespace*]
  罗列垃圾清理的预定情况。

:command:`trash purge schedule remove` [-p | --pool *pool*] [--namespace *namespace*] *interval* [*start-time*]
  删除垃圾清理的预定任务。

:command:`trash purge schedule status` [-p | --pool *pool*] [--format *format*] [--pretty-format] [--namespace *namespace*]
  显示垃圾清理的预定状态。

:command:`watch` *image-spec*
  盯着有关此映像的事件。


映像、快照、组和日志的名称规范
==============================
.. Image, snap, group and journal specs

| *image-spec*      is [*pool-name*/[*namespace-name*/]]\ *image-name*
| *snap-spec*       is [*pool-name*/[*namespace-name*/]]\ *image-name*\ @\ *snap-name*
| *group-spec*      is [*pool-name*/[*namespace-name*/]]\ *group-name*
| *group-snap-spec* is [*pool-name*/[*namespace-name*/]]\ *group-name*\ @\ *snap-name*
| *journal-spec*    is [*pool-name*/[*namespace-name*/]]\ *journal-name*

*pool-name* 的默认值是 rbd 、 *namespace-name* 默认是 "" （为空）。
如果某个映像名包含斜杠字符（ / ），那么还必须指定 *pool-name* 。

*journal-name* 是 *image-id* 。

你可以用 --pool 、 --namespace 、 --image 和 --snap 选项分别\
指定各个名字，但是不鼓励这样用，大家还是倾向于上面的规范语法。


条带化
======
.. Striping

RBD 映像被条带化到很多对象，然后存储到 Ceph 分布式对象存储（ RADOS ）集群中。
因此，到此映像的读和写请求会被分布到集群内的很多节点，
也因此避免了映像巨大或繁忙时可能出现的单节点瓶颈。

条带化由三个参数控制：

.. option:: object-size

   条带化产生的对象尺寸是 2 的幂，它会被对齐到最接近的 2 的幂。
   默认对象尺寸是 4MB ，最小是 4K 、最大 32 M 。

.. option:: stripe_unit

   各条带单位是连续的字节，相邻地存储于同一对象，用满再去下一个对象。

.. option:: stripe_count

   我们把 [*stripe_unit*] 个字节写够 [*stripe_count*] 个对象\
   后，再转回到第一个对象写另一轮条带，直到达到对象的最大尺\
   寸。此时，我们继续写下一轮 [*stripe_count*] 个对象。

默认情况下， [*stripe_unit*] 和对象尺寸相同、且 [*stripe_count*] 为 1 ；
另外指定 [*stripe_unit*] 和/或 [*stripe_count*] 通常出现在使用 fancy 条带化时、
而且必须是 format 2 格式的映像。


内核 rbd (krbd) 选项
====================
.. Kernel rbd (krbd) options

这里的大多数选项主要适用于调试和压力测试。默认值设置于内核中，\
因此还与所用内核的版本有关。

每个客户端例程的 `rbd device map` 选项：

* fsid=aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee - 应该由客户端提供\
  的 FSID 。

* ip=a.b.c.d[:p] - IP 还有客户端可选的端口。

* share - 允许与其它映射共享客户端例程（默认）。

* noshare - 禁止与其它映射共享客户端例程。

* crc - 对 msgr1 线上协议来说，启用 CRC32C 校验和（默认）；
  对 msgr2.1 协议来说，会忽略此选项： crc 模式下总是完全开启\
  校验和、 secure 模式下总是关闭。

* nocrc - 对 msgr1 线上协议禁用 CRC32C 校验。注意，
  只是禁用了载荷校验，头校验一直开启。
  msgr2.1 协议忽略此选项。

* cephx_require_signatures - 要求对 msgr1 消息签名（从 3.19 起\
  默认开启）。此选项已废弃，且未来会删除，
  因为此功能从 Bobtail 版起就支持了。

* nocephx_require_signatures - 不要求对 msgr1 消息签名
  （从 3.19 起）。此选项已废弃，且未来会删除，

* tcp_nodelay - 在客户端禁用 Nagle's 算法
  （从 4.0 起默认开启）。

* notcp_nodelay - 在客户端启用 Nagle's 算法（从 4.0 起）。

* cephx_sign_messages - 为 msgr1 协议启用消息签名
  （从 4.4 起默认开启）。使用 msgr2 协议时会忽略此选项：
  消息签名功能包含在 'secure' 模式里、而 'crc' 模式里没有。

* nocephx_sign_messages - 为 msgr1 线路协议禁用消息签名
  （从 4.4 起）。使用 msgr2 协议时会忽略此选项。

* mount_timeout=x - 执行 `rbd device map` 和 `rbd device unmap`
  时所涉及的各操作步骤的超时值（默认为 60 秒）。
  特别是从 4.2 起，与集群间没有连接时，
  即认为 `rbd device unmap` 操作超时了。

* osdkeepalive=x - OSD 保持连接的期限（默认为 5 秒）。

* osd_idle_ttl=x - OSD 闲置 TTL （默认为 60 秒）。

每个映射（块设备）的 `rbd device map` 选项：

* rw - 以读写方式映射映像（默认）。会被 --read-only 覆盖。

* ro - 以只读方式映射映像，等价于 --read-only 。

* queue_depth=x - 队列深度（从 4.2 起默认为 128 个请求）。

* lock_on_read - 除写入和 discard 操作外，读取时也要获取独占锁\
  （从 4.9 起）。

* exclusive - 禁止自动转换互斥锁（从 4.12 起）。
  等价于 --exclusive 。

* lock_timeout=x - 获取互斥锁的超时时长
  （ 4.17 起支持，默认是 0 秒，意味着没有超时）。

* notrim - 关闭 discard 、和填 0 功能，
  以免全配映像的空间被收回（从 4.17 起支持）。
  启用后， discard 请求会以 -EOPNOTSUPP 代码失败，
  填 0 请求会回退成手动填 0 。

* abort_on_full - 在集群空间用尽或数据存储池用完配额时\
  让写请求以 -ENOSPC 代码失败（从 5.0 起支持）。
  默认行为是阻塞着，直到占满条件释放。

* alloc_size - OSD 底层对象存储后端的最小分配单元
  （从 5.1 起支持，默认为 64KB ）。
  这是用于对齐数据块和丢弃太小的 discard 操作。对于 bluestore ，
  推荐的配置是 bluestore_min_alloc_size （一般来说，硬盘是 64K 、 SSD 是 16K ）；
  filestore 用 filestore_punch_hole = false 配置，
  推荐的配置是映像对象尺寸（一般是 4M ）。

* crush_location=x - 指定客户端在 CRUSH 分级结构（从 5.8 起）里的位置。
  这是用 '|' 分隔的一系列键值对，键名和值之间用 ':' 分隔。
  注意， '|' 可能得用引号包起来、或者转义（即 '\|' ），
  以免被 shell 解释为管道。键名是桶类型的名字
  （比如 rack 、 datacenter 、或者 region ，
  它是默认桶类型）、而值对应桶名字。
  例如，要表面客户端是本地的， rack 为 myrack 、
  数据中心为 mydc 、 region 是 myregion::

    crush_location=rack:myrack|datacenter:mydc|region:myregion

  每一个键值对都是独立的： myrack 不一定\
  位于 mydc 内、因此也不一定位于 myregion 内。
  这个位置不是到分级结构根（ root ）的路径，而是一系列独立匹配到的节点，
  多亏了桶名在 CRUSH 图里唯一这个特性。
  多路径（ multipath ）位置也可以，因此\
  可以为多个并行的分级结构定义位置： ::

    crush_location=rack:myrack1|rack:myrack2|datacenter:mydc

* read_from_replica=no - 禁用副本读取，总是选取主 OSD
  （从 5.8 起是默认行为）。

* read_from_replica=balance - 向多副本存储池发出读取请求时，
  随机选一个 OSD 提供服务（从 5.8 开始支持）。

  只有在 Octopus 之后（即 ``ceph osd require-osd-release octopus`` 之后）的版本上，
  这个模式才可以安全地作为一般用途。否则，就应该仅限于只读载荷，
  比如只读地映射出去的映像、或快照。

* read_from_replica=localize - 向多副本存储池发出读取请求时，
  选最近的 OSD 提供服务（从 5.8 开始支持）。
  位置指标根据客户端的 crush_location 位置计算出来；
  匹配到的级别最低的桶类型胜出。例如，在默认的桶类型中，
  一个匹配到 rack 的 OSD 比匹配到数据中心的 OSD 更近，
  自然也比匹配到 region 的 OSD 近。

  只有在 Octopus 之后（即 ``ceph osd require-osd-release octopus`` 之后）的版本上，
  这个模式才可以安全地作为一般用途。否则，就应该仅限于只读载荷，
  比如只读地映射出去的映像、或快照。

* compression_hint=none - 不设置压缩提示（从 5.8 起支持，默认的）。

* compression_hint=compressible - 提示底层的 OSD 对象存储后端，
  告诉它数据可以压缩，在被动模式下启用了压缩功能
  （从 5.8 起支持）。

* compression_hint=incompressible - 提示底层的 OSD 对象存储后端，
  告诉它数据不可压缩，在激进模式下禁用了压缩功能
  （从 5.8 起支持）。

* ms_mode=legacy - 采用 msgr1 线路协议（从 5.11 起支持，默认的）。

* ms_mode=crc - 采用 msgr2.1 线路协议，选择 crc 模式，
  也叫明文模式（从 5.11 起支持）。如果守护进程拒绝 crc 模式，
  连接会失败。

* ms_mode=secure - 采用 msgr2.1 线路协议，选择 secure 模式
  （从 5.11 起支持）。 secure 模式提供了完整的传输加密，
  可以同时确保保密性和真实性。如果守护进程拒绝 secure 模式，
  连接会失败。

* ms_mode=prefer-crc - 采用 msgr2.1 线路协议，选择 crc 模式
  （从 5.11 起支持）。如果守护进程拒绝了 crc 模式而选择 secure 模式，
  那就用 secure 模式。

* ms_mode=prefer-secure - 采用 msgr2.1 线路协议，选择 secure 模式
  （从 5.11 起支持）。如果守护进程拒绝了 secure 模式而选择 crc 模式，
  那就用 crc 模式。

* rxbounce - 接收数据时使用回弹缓冲区（自 5.17 版起）。
  默认行为是直接读入目标缓冲区。
  如果目标缓冲区不能保证稳定（即读出时保持不变），
  就需要使用回弹缓冲区。尤其是在 Windows 系统中，
  为了产生一次单个的大 I/O，一个系统范围内的“虚拟（ dummy ）”\
  （一次性的）页面可能会被映射到目标缓冲区中。
  否则，就会出现 "libceph: ... bad crc/signature" 或者
  "libceph: ... integrity error, bad crc" 错误，进而导致性能下降。

* udev - 等着 udev 设备管理器，让它执行完所有\
  能匹配 "add" 的规则、并在退出前释放设备（默认的）。
  这个选项不是传递给内核的。

* noudev - 不要等待 udev 设备管理器。打开此选项后，
  退出之后，设备可能不会立即恢复完全可用的状态。

`rbd device unmap` 选项：

* force - 让某一已打开的块设备强制取消映射（从 4.9 起支持）。
  其驱动会等待当前的请求完成之后再 unmap ；
  在 unmap 初始化之后再发给驱动的请求会失败。

* udev - 等着 udev 设备管理器，让它执行完所有\
  能匹配 "add" 的规则、并在退出前释放设备（默认的）。
  这个选项不是传递给内核的。

* noudev - 不要等待 udev 设备管理器。


实例
====

要新建一 100GB 的 rbd 映像： ::

	rbd create mypool/myimage --size 102400

用个非默认对象尺寸，8 MB： ::

	rbd create mypool/myimage --size 102400 --object-size 8M

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

	rbd device map mypool/myimage --id admin --keyfile secretfile

要通过内核把某一映像映射到没用默认名字 *ceph* 的集群： ::

	rbd device map mypool/myimage --cluster cluster-name

取消映像映射： ::

	rbd device unmap /dev/rbd0

创建一映像及其克隆品： ::

	rbd import --image-format 2 image mypool/parent
	rbd snap create mypool/parent@snap
	rbd snap protect mypool/parent@snap
	rbd clone mypool/parent@snap otherpool/child

新建一 stripe_unit 较小的映像（在某些情况下可更好地分布少量写）： ::

	rbd create mypool/myimage --size 102400 --stripe-unit 65536B --stripe-count 16

要改变某一映像的格式，\
先导出它、然后再导入成期望的映像格式::

	rbd export mypool/myimage@snap /tmp/img
	rbd import --image-format 2 /tmp/img mypool/myimage2

互斥地锁住一映像： ::

	rbd lock add mypool/myimage mylockid

释放锁： ::

	rbd lock remove mypool/myimage mylockid client.2485

罗列垃圾桶里的映像： ::

       rbd trash ls mypool

推迟删除一个映像（用 *--expires-at* 设置一个过期时间，默认是现在）： ::

       rbd trash mv mypool/myimage --expires-at "tomorrow"

从垃圾桶删除一个映像（谨慎啊！）： ::

       rbd trash rm mypool/myimage-id

从垃圾桶强行删除一个映像（谨慎啊！）： ::

       rbd trash rm mypool/myimage-id  --force

从垃圾桶恢复一个映像： ::

       rbd trash restore mypool/myimage-id

从垃圾桶恢复一个映像、并给它改个名字： ::

       rbd trash restore mypool/myimage-id --image mynewimage


使用范围
========

**rbd** 是 Ceph 的一部分，这是个伸缩力强、开源、
分布式的存储系统，更多信息参见 https://docs.ceph.com 。


参考
====

:doc:`ceph <ceph>`\(8),
:doc:`rados <rados>`\(8)
