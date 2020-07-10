:orphan:

=======================
 ceph -- Ceph 管理工具
=======================

.. program:: ceph

提纲
====

| **ceph** **auth** [ *add* \| *caps* \| *del* \| *export* \| *get* \| *get-key* \| *get-or-create* \| *get-or-create-key* \| *import* \| *list* \| *print-key* \| *print_key* ] ...

| **ceph** **compact**

| **ceph** **config** [ *dump* | *ls* | *help* | *get* | *show* | *show-with-defaults* | *set* | *rm* | *log* | *reset* | *assimilate-conf* | *generate-minimal-conf* ] ...

| **ceph** **config-key** [ *rm* | *exists* | *get* | *ls* | *dump* | *set* ] ...

| **ceph** **daemon** *<name>* \| *<path>* *<command>* ...

| **ceph** **daemonperf** *<name>* \| *<path>* [ *interval* [ *count* ] ]

| **ceph** **df** *{detail}*

| **ceph** **fs** [ *ls* \| *new* \| *reset* \| *rm* ] ...

| **ceph** **fsid**

| **ceph** **health** *{detail}*

| **ceph** **injectargs** *<injectedargs>* [ *<injectedargs>*... ]

| **ceph** **log** *<logtext>* [ *<logtext>*... ]

| **ceph** **mds** [ *compat* \| *fail* \| *rm* \| *rmfailed* \| *set_state* \| *stat* \| *repaired* ] ...

| **ceph** **mon** [ *add* \| *dump* \| *getmap* \| *remove* \| *stat* ] ...

| **ceph** **osd** [ *blacklist* \| *blocked-by* \| *create* \| *new* \| *deep-scrub* \| *df* \| *down* \| *dump* \| *erasure-code-profile* \| *find* \| *getcrushmap* \| *getmap* \| *getmaxosd* \| *in* \| *ls* \| *lspools* \| *map* \| *metadata* \| *ok-to-stop* \| *out* \| *pause* \| *perf* \| *pg-temp* \| *force-create-pg* \| *primary-affinity* \| *primary-temp* \| *repair* \| *reweight* \| *reweight-by-pg* \| *rm* \| *destroy* \| *purge* \| *safe-to-destroy* \| *scrub* \| *set* \| *setcrushmap* \| *setmaxosd*  \| *stat* \| *tree* \| *unpause* \| *unset* ] ...

| **ceph** **osd** **crush** [ *add* \| *add-bucket* \| *create-or-move* \| *dump* \| *get-tunable* \| *link* \| *move* \| *remove* \| *rename-bucket* \| *reweight* \| *reweight-all* \| *reweight-subtree* \| *rm* \| *rule* \| *set* \| *set-tunable* \| *show-tunables* \| *tunables* \| *unlink* ] ...

| **ceph** **osd** **pool** [ *create* \| *delete* \| *get* \| *get-quota* \| *ls* \| *mksnap* \| *rename* \| *rmsnap* \| *set* \| *set-quota* \| *stats* ] ...

| **ceph** **osd** **pool** **application** [ *disable* \| *enable* \| *get* \| *rm* \| *set* ] ...

| **ceph** **osd** **tier** [ *add* \| *add-cache* \| *cache-mode* \| *remove* \| *remove-overlay* \| *set-overlay* ] ...

| **ceph** **pg** [ *debug* \| *deep-scrub* \| *dump* \| *dump_json* \| *dump_pools_json* \| *dump_stuck* \| *getmap* \| *ls* \| *ls-by-osd* \| *ls-by-pool* \| *ls-by-primary* \| *map* \| *repair* \| *scrub* \| *stat* ] ...

| **ceph** **quorum_status**

| **ceph** **report** { *<tags>* [ *<tags>...* ] }

| **ceph** **status**

| **ceph** **sync** **force** {--yes-i-really-mean-it} {--i-know-what-i-am-doing}

| **ceph** **tell** *<name (type.id)> <command> [options...]*

| **ceph** **version**


描述
====

:program:`ceph` 是个控制工具，可用于手动部署和维护 Ceph 集群。\
它提供的多种工具可用于部署监视器、 OSD 、归置组、 MDS 和维护、\
管理整个集群。


命令
====

auth
----

管理认证密钥。用于给某个具体实体（如监视器或 OSD ）增加、\
删除、导出或更新认证密钥。还需额外加子命令。

子命令 ``add`` 用于为特定实体增加认证信息，这些信息可从文件\
读入，若未在命令行指定密钥（和、或此密钥的能力）将生成随机密钥。

用法： ::

	ceph auth add <entity> {<caps> [<caps>...]}

子命令 ``caps`` 把 **name** 的能力更新为命令行中指定的。

用法： ::

	ceph auth caps <entity> <caps> [<caps>...]

子命令 ``del`` 删除 ``name`` 的所有能力。

用法： ::

	ceph auth del <entity>

子命令 ``export`` 把指定实体写入密钥环，若未指定则写入主密钥环。

用法： ::

	ceph auth export {<entity>}

子命令 ``get`` 把请求到的密钥写入密钥环文件。

用法： ::

	ceph auth get <entity>

子命令 ``get-key`` 显示所请求的密钥。

用法： ::

	ceph auth get-key <entity>

子命令 ``get-or-create`` 用于为特定实体增加认证信息，这些信息\
可从文件读入，若未在命令行指定密钥（和、或此密钥的能力）将生成\
随机密钥。

用法： ::

	ceph auth get-or-create <entity> {<caps> [<caps>...]}

子命令 ``get-or-create-key`` 根据命令行指定的系统、能力对，为
``name`` 获取或创建密钥。若密钥已存在，任何指定的能力必须与\
当前已有能力一致。

用法： ::

	ceph auth get-or-create-key <entity> {<caps> [<caps>...]}

子命令 ``import`` 从输入文件读入密钥环。

用法： ::

	ceph auth import

子命令 ``ls`` 罗列认证状态。

用法： ::

	ceph auth ls

子命令 ``print-key`` 显示请求的密钥。

用法： ::

	ceph auth print-key <entity>

子命令 ``print_key`` 显示请求的密钥。

用法： ::

	ceph auth print_key <entity>


compact
-------

让监视器压缩其 leveldb 存储。

用法： ::

	ceph compact


config-key
----------

管理配置密钥。需额外指定子命令。

子命令 ``del`` 用于删除配置密钥。

用法： ::

	ceph config-key del <key>

子命令 ``exists`` 用于检查配置密钥是否存在。

用法： ::

	ceph config-key exists <key>

子命令 ``get`` 用于获取配置密钥。

用法： ::

	ceph config-key get <key>

子命令 ``list`` 罗列配置密钥。

用法： ::

	ceph config-key ls

子命令 ``dump`` 倒出配置中的所有键及其值。

用法： ::

	ceph config-key dump

子命令 ``set`` 上传配置密钥及其内容。

用法： ::

	ceph config-key set <key> {<val>}


df
--

显示集群空闲空间状态。

用法： ::

	ceph df {detail}


.. _ceph features:

features
--------

查看所有已连接守护进程、以及连入集群的各客户端的版本号及其\
功能，还有各功能、版本号集合对应的（守护进程、客户端）数量。
Ceph 的各个版本都有不同的功能集，以功能位掩码表示。新集群功能\
要求客户端也支持这些功能，否则不允许它们连接这些新功能。因为\
新功能或能力是系统升级后才启用的，（新集群）会阻止老客户端\
连接。

用法： ::

    ceph features


fs
--

用于管理 cephfs 文件系统，需额外加子命令。

子命令 ``ls`` 用于罗列文件系统。

用法： ::

	ceph fs ls

子命令 ``new`` 用指定的存储池 <metadata> 和 <data> 创建新文件系统。

用法： ::

	ceph fs new <fs_name> <metadata> <data>

子命令 ``reset`` 仅适用于灾难恢复：重置成单 MDS 运行图。

用法： ::

	ceph fs reset <fs_name> {--yes-i-really-mean-it}

子命令 ``rm`` 用于禁用指定文件系统。

用法： ::

	ceph fs rm <fs_name> {--yes-i-really-mean-it}


fsid
----

显示集群的 FSID/UUID 。

用法： ::

	ceph fsid


health
------

显示集群健康状况。

用法： ::

	ceph health {detail}


heap
----

显示堆栈使用信息（编译时启用了 tcmalloc 支持才可用）

用法： ::

	ceph heap dump|start_profiler|stop_profiler|release|stats


injectargs
----------

向监视器注入配置参数。

用法： ::

	ceph injectargs <injected_args> [<injected_args>...]


log
---

把指定文本记录到监视器日志中。

用法： ::

	ceph log <logtext> [<logtext>...]


mds
---

用于元数据服务器的配置和管理，需额外指定子命令。

子命令 ``compat`` 管理兼容功能，需额外指定子命令。

子命令 ``rm_compat`` 可删除兼容功能。

用法： ::

	ceph mds compat rm_compat <int[0-]>

子命令 ``rm_incompat`` 可删除不兼容的功能。

用法： ::

	ceph mds compat rm_incompat <int[0-]>

子命令 ``show`` 可查看 mds 的兼容性选项。

用法： ::

	ceph mds compat show

子命令 ``fail`` 强制把 mds 状态设置为失效。

用法： ::

	ceph mds fail <role|gid>

子命令 ``rm`` 用于删除不活跃的 mds 。

用法： ::

	ceph mds rm <int[0-]> <name> (type.id)>

子命令 ``rmfailed`` 用于删除失效的 mds 。

用法： ::

	ceph mds rmfailed <int[0-]>

子命令 ``set_state`` 把 mds 状态从 <gid> 改为 <numeric-state> 。

用法： ::

	ceph mds set_state <int[0-]> <int[0-20]>

子命令 ``stat`` 显示 MDS 状态。

用法： ::

	ceph mds stat

子命令 ``repaired`` 把损坏的 MDS rank 标记为不再是损坏的。

用法： ::

	ceph mds repaired <role>


mon
---

用于监视器的配置和管理，需额外指定子命令。

子命令 ``add`` 新增名为 <name> 的监视器，地址为 <addr> 。

用法： ::

	ceph mon add <name> <IPaddr[:port]>

子命令 ``dump`` 转储格式化的 monmap ， epoch 号可选。

用法： ::

	ceph mon dump {<int[0-]>}

子命令 ``getmap`` 用于获取 monmap 。

用法： ::

	ceph mon getmap {<int[0-]>}

子命令 ``remove`` 用于删除名为 <name> 的监视器。

用法： ::

	ceph mon remove <name>

子命令 ``stat`` 汇总监视器状态。

用法： ::

	ceph mon stat


mon_status
----------

报告监视器状态。

用法： ::

	ceph mon_status


mgr
---

Ceph 管理器守护进程的配置和管理。

子命令 ``dump`` 转储最新的 MgrMap ，其中有活跃的和备用的管理器\
守护进程。

用法： ::

  ceph mgr dump

子命令 ``fail`` 可把一个管理器守护进程标记为已失效，并把它从\
管理器运行图中删掉。如果它是活跃管理器，将会有一个备机顶替它。

用法： ::

  ceph mgr fail <name>

子命令 ``module ls`` 可罗列当前已启用的管理器模块（插件）。

用法： ::

  ceph mgr module ls

子命令 ``module enable`` 可启用一个管理器模块。可用模块在
MgrMap 内，可以用 ``mgr dump`` 查看。

用法： ::

  ceph mgr module enable <module>

子命令 ``module disable`` 可禁用当前活跃的管理器模块。

用法： ::

  ceph mgr module disable <module>

子命令 ``metadata`` 可显示所有管理器守护进程的元数据；如果指定\
了名字，就只显示它的。

用法： ::

  ceph mgr metadata [name]

子命令 ``versions`` 可显示所有在运行守护进程的版本个数。

用法： ::

  ceph mgr versions

子命令 ``count-metadata`` 可显示任意守护进程的元数据字段个数。

用法： ::

  ceph mgr count-metadata <field>


.. _ceph-admin-osd:

osd
---

用于配置和管理 OSD ，需额外指定子命令。

子命令 ``blacklist`` 用于管理客户端黑名单，需额外加子命令。

子命令 ``add`` 用于把 <addr> 加入黑名单（可指定时间，从现在起 <expire> 秒）。

用法： ::

	ceph osd blacklist add <EntityAddr> {<float[0.0-]>}

子命令 ``ls`` 列出进黑名单的客户端。

用法： ::

	ceph osd blacklist ls

子命令 ``rm`` 从黑名单里删除 <addr> 。

用法： ::

	ceph osd blacklist rm <EntityAddr>

子命令 ``blocked-by`` 用于罗列哪些 OSD 在阻塞互联。

用法： ::

	ceph osd blocked-by


子命令 ``create`` 用于新建 OSD ， UUID 和 ID 是可选的。

从 Luminous 版起，此命令已\ **废弃**\ ，未来会删除。

请改用 ``new`` 子命令。

用法： ::

	ceph osd create {<uuid>} {<id>}


子命令 ``new`` 可用来创建新 OSD 或者重新创建之前销毁的已经\
分配过 *id* 的 OSD ；这个新 OSD 会用指定的 *uuid* ，此命令\
还需指定一个 JSON 文件，其内有认证实体 *client.osd.<id>* 的
base64 编码 cephx 密钥；还有些可选项，如访问 dm-crypt 密码箱的
base64 编码 cephx 密钥、和一个 dm-crypt 密钥。指定 dm-crypt
密钥时，还必须同时指定密码箱的 cephx 密钥。

用法： ::

    ceph osd new {<uuid>} {<id>} -i {<params.json>}

JSON 文件内的参数是可选的，但是如果设置了，就必须遵守下面的\
几种格式之一： ::

    {
        "cephx_secret": "AQBWtwhZdBO5ExAAIDyjK2Bh16ZXylmzgYYEjg==",
        "crush_device_class": "myclass"
    }

或者： ::

    {
        "cephx_secret": "AQBWtwhZdBO5ExAAIDyjK2Bh16ZXylmzgYYEjg==",
        "cephx_lockbox_secret": "AQDNCglZuaeVCRAAYr76PzR1Anh7A0jswkODIQ==",
        "dmcrypt_key": "<dm-crypt key>",
        "crush_device_class": "myclass"
    }

或者： ::

    {
        "crush_device_class": "myclass"
    }

``crush_device_class`` 属性是可选的；指定后，它将是新 OSD 的初\
始 CRUSH 设备类。


子命令 ``crush`` 用于 CRUSH 管理，需额外指定子命令。

子命令 ``add`` 可用于新增或更新 <name> 的 crushmap 位置及权重，权重改为 \
<weight> 、位置为 <args> 。

用法： ::

	ceph osd crush add <osdname (id|osd.id)> <float[0.0-]> <args> [<args>...]

子命令 ``add-bucket`` 可新增没有父级（可能是 root ）、类型为 <type> 、名为 \
<name> 的 crush 桶。

用法： ::

	ceph osd crush add-bucket <name> <type>

子命令 ``create-or-move`` 用于创建名为 <name> 、权重为 <weight> 的条目并放置\
到 <args> ，若已存在则移动到指定位置 <args> 。

用法： ::

	ceph osd crush create-or-move <osdname (id|osd.id)> <float[0.0-]>
	<args> [<args>...]

子命令 ``dump`` 用于转储 crush 图。

用法： ::

	ceph osd crush dump

子命令 ``get-tunable`` 用于获取 CRUSH 可调值 straw_calc_version 。

用法： ::

	ceph osd crush get-tunable straw_calc_version

子命令 ``link`` 用于把已存在条目 <name> 链接到 <args> 位置下。

用法： ::

	ceph osd crush link <name> <args> [<args>...]

子命令 ``move`` 可把已有条目 <name> 移动到 <args> 位置。

用法： ::

	ceph osd crush move <name> <args> [<args>...]

子命令 ``remove`` 把 crush 图中（任意位置，或 <ancestor> 之下的）的 <name> \
删掉。

用法： ::

	ceph osd crush remove <name> {<ancestor>}

子命令 ``rename-bucket`` 可把桶 <srcname> 重命名为 <dstname> 。

用法： ::

	ceph osd crush rename-bucket <srcname> <dstname>

子命令 ``reweight`` 把 crush 图中 <name> 的权重改为 <weight> 。

用法： ::

	ceph osd crush reweight <name> <float[0.0-]>

子命令 ``reweight-all`` 重新计算树的权重，以确保权重之和没算错。

用法： ::

	ceph osd crush reweight-all

子命令 ``reweight-subtree`` 用于把 CRUSH 图内 <name> 之下的所有叶子条目的\
权重改为 <weight> 。

用法： ::

	ceph osd crush reweight-subtree <name> <weight>

子命令 ``rm`` 把 crush 图中（任意位置，或 <ancestor> 之下的）的 <name> 删掉。

用法： ::

	ceph osd crush rm <name> {<ancestor>}

子命令 ``rule`` 用于创建 crush 规则，需额外加子命令。

子命令 ``create-erasure`` 可为纠删码存储池（用 <profile> 创建的））创建名为 \
<name> 的 crush 规则，默认为 default 。

用法： ::

	ceph osd crush rule create-erasure <name> {<profile>}

子命令 ``create-simple`` 创建从 <root> 开始、名为 <name> 的 crush 规则，副本\
将跨 <type> 类型进行散布，选择模式为 <firstn|indep> （默认 firstn ，indep 更\
适合纠删码存储池）。

用法： ::

	ceph osd crush rule create-simple <name> <root> <type> {firstn|indep}

子命令 ``dump`` 转储名为 <name> 的 crush 规则，默认全部转储。

用法： ::

	ceph osd crush rule dump {<name>}


子命令 ``ls`` 罗列 crush 规则。

用法： ::

	ceph osd crush rule ls

子命令 ``rm`` 删除 crush 规则 <name> 。

用法： ::

	ceph osd crush rule rm <name>

子命令 ``set`` 单独使用，把输入文件设置为 crush 图。

用法： ::

	ceph osd crush set

子命令 ``set`` 为 osdname 或 osd.id 更新 crush 图的位置和权重信\
息，把名为 <name> 的 OSD 权重设置为 <weight> 、位置设置为 <args> 。

用法： ::

	ceph osd crush set <osdname (id|osd.id)> <float[0.0-]> <args> [<args>...]

子命令 ``set-tunable`` 把可调值 <tunable> 设置为 <value> 。唯一\
能设置的可调值是 straw_calc_version 。

用法： ::

	ceph osd crush set-tunable straw_calc_version <value>

子命令 ``show-tunables`` 显示当前的 crush 可调值。

用法： ::

	ceph osd crush show-tunables

子命令 ``tree`` 用树状视图显示各 crush 桶、及各条目。

用法： ::

	ceph osd crush tree

子命令 ``tunables`` 设置 <profile> 中的 crush 可调值。

用法： ::

	ceph osd crush tunables legacy|argonaut|bobtail|firefly|hammer|optimal|default

子命令 ``unlink`` 从 crush 图中解链接出 <name> （任意位置，或 \
<ancestor> 之下的）。

用法： ::

	ceph osd crush unlink <name> {<ancestor>}

子命令 ``df`` 用于显示 OSD 利用率。

用法： ::

	ceph osd df {plain|tree}

子命令 ``deep-scrub`` 可启动指定 OSD 的深度洗刷。

用法： ::

	ceph osd deep-scrub <who>

子命令 ``down`` 把 osd(s) <id> [<id>...] 状态设置为 down 。

用法： ::

	ceph osd down <ids> [<ids>...]

子命令 ``dump`` 打印 OSD 图汇总。

用法： ::

	ceph osd dump {<int[0-]>}

子命令 ``erasure-code-profile`` 用于管理纠删码配置，需额外加子命令。

子命令 ``get`` 读取纠删码配置 <name> 。

用法： ::

	ceph osd erasure-code-profile get <name>

子命令 ``ls`` 罗列所有纠删码配置。

用法： ::

	ceph osd erasure-code-profile ls

子命令 ``rm`` 删除纠删码配置 <name> 。

用法： ::

	ceph osd erasure-code-profile rm <name>

子命令 ``set`` 用给定的参数 [<key[=value]> ...] 创建纠删码配置 \
<name> 。末尾加 --force 可覆盖已有配置（慎用）。

用法： ::

	ceph osd erasure-code-profile set <name> {<profile> [<profile>...]}

子命令 ``find`` 从 CRUSH 图中找到 osd <id> 并显示其位置。

用法： ::

	ceph osd find <int[0-]>

子命令 ``getcrushmap`` 获取 CRUSH 图。

用法： ::

	ceph osd getcrushmap {<int[0-]>}

子命令 ``getmap`` 获取 OSD 图。

用法： ::

	ceph osd getmap {<int[0-]>}

子命令 ``getmaxosd`` 显示最大 OSD 惟一标识符。

用法： ::

	ceph osd getmaxosd

子命令 ``in`` 把给出的 OSD <id> [<id>...] 标识为 in 状态。

用法： ::

	ceph osd in <ids> [<ids>...]

子命令 ``lost`` 把 OSD 标识为永久丢失。如果没有多个副本，此命令\
会导致数据丢失，慎用。

用法： ::

	ceph osd lost <int[0-]> {--yes-i-really-mean-it}

子命令 ``ls`` 显示所有 OSD 的惟一标识符。

用法： ::

	ceph osd ls {<int[0-]>}

子命令 ``lspools`` 罗列存储池。

用法： ::

	ceph osd lspools {<int>}

子命令 ``map`` 在 <pool> 存储池中找 <object> 对象所在的归置组号码。

用法： ::

	ceph osd map <poolname> <objectname>

子命令 ``metadata`` 为 osd <id> 取出元数据。

用法： ::

	ceph osd metadata {int[0-]} (default all)

子命令 ``out`` 把指定 OSD <id> [<id>...] 的状态设置为 out 。

用法： ::

	ceph osd out <ids> [<ids>...]

子命令 ``ok-to-stop`` 用于检查一些 OSD 是否能停止，而不会马上\
让其数据不可用。也就是说，尽管在降级模式（但还活跃着）下由于\
某些 PG 的失效导致数据冗余性降低了，所有数据仍然可读和可写。\
如果可以停止这些 OSD ，它就会返回一个成功代码；如果不行或者\
现在对于如何描述还没头绪，就返回一个错误代码和提示性消息。

用法： ::

  ceph osd ok-to-stop <id> [<ids>...]

子命令 ``pause`` 暂停 osd 。

用法： ::

	ceph osd pause

子命令 ``perf`` 打印 OSD 的性能统计摘要。

用法： ::

	ceph osd perf

子命令 ``pg-temp`` 设置 pg_temp 映射 pgid:[<id> [<id>...]] ，\
适用于开发者。

用法： ::

	ceph osd pg-temp <pgid> {<id> [<id>...]}

子命令 ``force-create-pg`` 可强行创建 pg <pgid> 。

用法： ::

	ceph osd force-create-pg <pgid>

子命令 ``pool`` 用于管理数据存储池，需额外加子命令。

子命令 ``create`` 创建存储池。

用法： ::

	ceph osd pool create <poolname> <int[0-]> {<int[0-]>} {replicated|erasure}
	{<erasure_code_profile>} {<rule>} {<int>}

子命令 ``delete`` 删除存储池。

用法： ::

	ceph osd pool delete <poolname> {<poolname>} {--yes-i-really-really-mean-it}

子命令 ``get`` 获取存储池参数 <var> 。

用法： ::

	ceph osd pool get <poolname> size|min_size|pg_num|pgp_num|crush_rule|write_fadvise_dontneed

以下命令只适用于分层存储池： ::

	ceph osd pool get <poolname> hit_set_type|hit_set_period|hit_set_count|hit_set_fpp|
	target_max_objects|target_max_bytes|cache_target_dirty_ratio|cache_target_dirty_high_ratio|
	cache_target_full_ratio|cache_min_flush_age|cache_min_evict_age|
	min_read_recency_for_promote|hit_set_grade_decay_rate|hit_set_search_last_n

以下命令只适用于纠删码存储池： ::

	ceph osd pool get <poolname> erasure_code_profile

子命令 ``get-quota`` 获取存储池的对象或字节数限额。

用法： ::

	ceph osd pool get-quota <poolname>

子命令 ``ls`` 用于罗列存储池。

用法： ::

	ceph osd pool ls {detail}

子命令 ``mksnap`` 拍下存储池 <pool> 的快照 <snap> 。

用法： ::

	ceph osd pool mksnap <poolname> <snap>

子命令 ``rename`` 把存储池 <srcpool> 重命名为 <destpool> 。

用法： ::

	ceph osd pool rename <poolname> <poolname>

子命令 ``rmsnap`` 删除存储池 <pool> 的快照 <snap> 。

用法： ::

	ceph osd pool rmsnap <poolname> <snap>

子命令 ``set`` 把存储池参数 <var> 的值设置为 <val> 。

用法： ::

	ceph osd pool set <poolname> size|min_size|pg_num|
	pgp_num|crush_rule|hashpspool|nodelete|nopgchange|nosizechange|
	hit_set_type|hit_set_period|hit_set_count|hit_set_fpp|debug_fake_ec_pool|
	target_max_bytes|target_max_objects|cache_target_dirty_ratio|
	cache_target_dirty_high_ratio|
	cache_target_full_ratio|cache_min_flush_age|cache_min_evict_age|
	min_read_recency_for_promote|write_fadvise_dontneed|hit_set_grade_decay_rate|
	hit_set_search_last_n
	<val> {--yes-i-really-mean-it}

子命令 ``set-quota`` 设置存储池的对象或字节数限额。

用法： ::

	ceph osd pool set-quota <poolname> max_objects|max_bytes <val>

子命令 ``stats`` 获取所有或指定存储池的统计信息。

用法： ::

	ceph osd pool stats {<name>}

子命令 ``application`` is used for adding an annotation to the given
pool. By default, the possible applications are object, block, and file
storage (corresponding app-names are "rgw", "rbd", and "cephfs"). However,
there might be other applications as well. Based on the application, there
may or may not be some processing conducted.

子命令 ``disable`` disables the given application on the given pool.

用法： ::

        ceph osd pool application disable <pool-name> <app> {--yes-i-really-mean-it}

子命令 ``enable`` adds an annotation to the given pool for the mentioned
application.

用法： ::

        ceph osd pool application enable <pool-name> <app> {--yes-i-really-mean-it}

子命令 ``get`` displays the value for the given key that is associated
with the given application of the given pool. Not passing the optional
arguments would display all key-value pairs for all applications for all
pools.

用法： ::

        ceph osd pool application get {<pool-name>} {<app>} {<key>}

子命令 ``rm`` removes the key-value pair for the given key in the given
application of the given pool.

用法： ::

        ceph osd pool application rm <pool-name> <app> <key>

子命令 ``set`` assosciates or updates, if it already exists, a key-value
pair with the given application for the given pool.

用法： ::

        ceph osd pool application set <pool-name> <app> <key> <value>

子命令 ``primary-affinity`` 设置主 OSD 亲和性，有效值范围 \
0.0 <= <weight> <= 1.0

用法： ::

	ceph osd primary-affinity <osdname (id|osd.id)> <float[0.0-1.0]>

子命令 ``primary-temp`` 设置 primary_temp 映射 pgid:<id>|-1 ，适用于开发者。

用法： ::

	ceph osd primary-temp <pgid> <id>

子命令 ``repair`` 让指定 OSD 开始修复。

用法： ::

	ceph osd repair <who>

子命令 ``reweight`` 把 OSD 权重改为 0.0 < <weight> < 1.0 之间的值。

用法： ::

	osd reweight <int[0-]> <float[0.0-1.0]>

子命令 ``reweight-by-pg`` 按归置组分布情况调整 OSD 的权重（考虑的过载百分\
比，默认 120 ）。

用法： ::

	ceph osd reweight-by-pg {<int[100-]>} {<poolname> [<poolname...]}
	{--no-increasing}

子命令 ``reweight-by-utilization`` 按利用率调整 OSD 的权重，还需考虑负载比\
率，默认 120 。

用法： ::

	ceph osd reweight-by-utilization {<int[100-]>}
	{--no-increasing}

子命令 ``rm`` 删除 OSD 运行图中的 OSD ，其编号为 <id> [<id>...] 。

用法： ::

	ceph osd rm <ids> [<ids>...]

子命令 ``destroy`` 把 OSD *id* 标记为 *destroyed （已销毁）*\
，并删除与之对应的的 cephx 密钥、以及 dm-crypt 配置、和守护\
进程私有的配置条目。

此命令不会从 crush 中删除这个 OSD ，也不会从 OSD 运行图中删除\
它；而是，一旦此命令正确无误地执行完，这个 OSD 的状态就是被标\
记为 *destroyed* 。

要把一个 OSD 标记为已销毁，它必须先被标记为
**lost （丢失）**\ 。

用法： ::

    ceph osd destroy <id> {--yes-i-really-mean-it}


子命令 ``purge`` 执行的是 ``osd destroy`` 、 ``osd rm`` 和
``osd crush remove`` 命令的合体。

用法： ::

    ceph osd purge <id> {--yes-i-really-mean-it}


子命令 ``safe-to-destroy`` 会检查在不降低整体数据冗余度或持久\
性的前提下，删除或销毁一个 OSD 是否安全。如果绝对安全，它会返\
回成功码；如果不是、或者现在还不能断定，它会返回错误码和提示\
消息。

用法： ::

  ceph osd safe-to-destroy <id> [<ids>...]


子命令 ``scrub`` 让指定 OSD 开始洗刷。

用法： ::

	ceph osd scrub <who>

子命令 ``set`` 设置关键字 <key> 。

用法： ::

	ceph osd set full|pause|noup|nodown|noout|noin|nobackfill|
	norebalance|norecover|noscrub|nodeep-scrub|notieragent

子命令 ``setcrushmap`` 把输入文件设置为 CRUSH 图。

用法： ::

	ceph osd setcrushmap

子命令 ``setmaxosd`` 设置最大 OSD 数值。

用法： ::

	ceph osd setmaxosd <int[0-]>


子命令 ``set-require-min-compat-client`` 强制集群向后兼容，\
使之与指定的客户端版本相兼容。用这个子命令无需做破坏当前配置\
的更改（如 crush 可调值、或使用新功能）。请注意，如果存在与\
指定版本 <version> 的功能不兼容的已连接守护进程或客户端，这\
个子命令会失败。要查看已连入集群的所有客户端的功能和版本，\
请看 `ceph features`_ 。

用法： ::

    ceph osd set-require-min-compat-client <version>


子命令 ``stat`` 打印 OSD 图摘要。

用法： ::

	ceph osd stat

子命令 ``tier`` 用于管理（存储池）分级，需额外加子命令。

子命令 ``add`` 把 <tierpool> （第二个）加到基础存储池 <pool>
（第一个）之前。

用法： ::

	ceph osd tier add <poolname> <poolname> {--force-nonempty}

子命令 ``add-cache`` 把尺寸为 <size> 的缓存存储池 <tierpool>
（第二个）加到现有存储池 <pool> （第一个）之前。

用法： ::

	ceph osd tier add-cache <poolname> <poolname> <int[0-]>

子命令 ``cache-mode`` 设置缓存存储池 <pool> 的缓存模式。

用法： ::

	ceph osd tier cache-mode <poolname> none|writeback|forward|readonly|
	readforward|readproxy

子命令 ``remove`` 删掉基础存储池 <pool> （第一个）的马甲存储池
<tierpool> （第二个）。

用法： ::

	ceph osd tier remove <poolname> <poolname>

子命令 ``remove-overlay`` 删除基础存储池 <pool> 的马甲存储池。

用法： ::

	ceph osd tier remove-overlay <poolname>

子命令 ``set-overlay`` 把 <overlaypool> 设置为基础存储池 <pool>
的马甲存储池。

用法： ::

	ceph osd tier set-overlay <poolname> <poolname>

子命令 ``tree`` 打印 OSD 树。

用法： ::

	ceph osd tree {<int[0-]>}

子命令 ``unpause`` 取消 osd 暂停。

用法： ::

	ceph osd unpause

子命令 ``unset`` 取消设置的关键字 <key> 。

用法： ::

	ceph osd unset full|pause|noup|nodown|noout|noin|nobackfill|
	norebalance|norecover|noscrub|nodeep-scrub|notieragent


pg
--

用于管理 OSD 内的归置组，需额外加子命令。

子命令 ``debug`` 可显示归置组的调试信息。

用法： ::

	ceph pg debug unfound_objects_exist|degraded_pgs_exist

子命令 ``deep-scrub`` 开始深度洗刷归置组 <pgid> 。

用法： ::

	ceph pg deep-scrub <pgid>

子命令 ``dump`` 可显示归置组图的人类可读版本（显示为纯文本时只有 'all' 合法）。

用法： ::

	ceph pg dump {all|summary|sum|delta|pools|osds|pgs|pgs_brief} [{all|summary|sum|delta|pools|osds|pgs|pgs_brief...]}

子命令 ``dump_json`` 只以 json 格式显示归置组图的人类可读版本。

用法： ::

	ceph pg dump_json {all|summary|sum|delta|pools|osds|pgs|pgs_brief} [{all|summary|sum|delta|pools|osds|pgs|pgs_brief...]}

子命令 ``dump_pools_json`` 只以 json 格式显示归置组存储池信息［译者：存疑］。

用法： ::

	ceph pg dump_pools_json

子命令 ``dump_stuck`` 显示卡顿归置组的信息。

用法： ::

	ceph pg dump_stuck {inactive|unclean|stale|undersized|degraded [inactive|unclean|stale|undersized|degraded...]}
	{<int>}

子命令 ``getmap`` 获取二进制归置组图，保存到 -o/stdout 。

用法： ::

	ceph pg getmap

子命令 ``ls`` 可根据指定存储池、 OSD 、状态罗列对应的归置组。

用法： ::

	ceph pg ls {<int>} {<pg-state> [<pg-state>...]}

子命令 ``ls-by-osd`` 用于罗列指定 OSD 上的归置组。

用法： ::

	ceph pg ls-by-osd <osdname (id|osd.id)> {<int>}
	{<pg-state> [<pg-state>...]}

子命令 ``ls-by-pool`` 用于罗列存储池 [poolname] 内的归置组。

用法： ::

	ceph pg ls-by-pool <poolstr> {<int>} {<pg-state> [<pg-state>...]}

子命令 ``ls-by-primary`` 可罗列主 OSD 为 [osd] 的归置组。

用法： ::

	ceph pg ls-by-primary <osdname (id|osd.id)> {<int>}
	{<pg-state> [<pg-state>...]}

子命令 ``map`` 显示归置组到 OSD 的映射关系。

用法： ::

	ceph pg map <pgid>

子命令 ``repair`` 开始修复归置组 <pgid> 。

用法： ::

	ceph pg repair <pgid>

子命令 ``scrub`` 开始洗刷归置组 <pgid> 。

用法： ::

	ceph pg scrub <pgid>

子命令 ``stat`` 显示归置组状态。

用法： ::

	ceph pg stat


quorum
------

让监视器加入或退出法定人数。

用法： ::

	ceph quorum enter|exit

注：此命令只对 ``ceph`` 命令能连接上的监视器管用。如果你想让某\
个具体的监视器加入或退出法定人数，可以用下面这个命令： ::

	ceph tell mon.<id> quorum enter|exit


quorum_status
-------------

报告监视器法定人数状态。

用法： ::

	ceph quorum_status


report
------

报告集群的全部状态，标签字符串可选。
Reports full status of cluster, optional title tag strings.

用法： ::

	ceph report {<tags> [<tags>...]}


scrub
-----

洗刷监视器的存储。

用法： ::

	ceph scrub


status
------

显示集群状态。

用法： ::

	ceph status


sync force
----------

强制监视器进行同步、并清除存储。

用法： ::

	ceph sync force {--yes-i-really-mean-it} {--i-know-what-i-am-doing}


tell
----

发命令给指定守护进程。

用法： ::

	ceph tell <name (type.id)> <args> [<args>...]


罗列所有可用的命令。

用法： ::

	ceph tell <name (type.id)> help


version
-------

显示监视器守护进程的版本。

用法： ::

	ceph version


选项
====

.. option:: -i infile

   指定一个输入文件，它将作为载荷与命令一起传递给监视器集群。仅用于某些特定\
   的监视器命令。

.. option:: -o outfile

   把响应中监视器集群返回的载荷写入 outfile 文件。只有某些特定的监视器命令\
   （如 psd getmap ）会返回载荷。

.. option:: -c ceph.conf, --conf=ceph.conf

   用 ceph.conf 配置文件而非默认的 ``/etc/ceph/ceph.conf`` 来确定启动时所用\
   的监视器地址。

.. option:: --id CLIENT_ID, --user CLIENT_ID

   用于认证的客户端 ID 。

.. option:: --name CLIENT_NAME, -n CLIENT_NAME

   用于认证的客户端名字。

.. option:: --cluster CLUSTER

   Ceph 集群名字。

.. option:: --admin-daemon ADMIN_SOCKET, daemon DAEMON_NAME

   提交管理套接字命令。

.. option:: --admin-socket ADMIN_SOCKET_NOPE

   你也许想要的是 --admin-daemon 。

.. option:: -s, --status

   显示集群状态。

.. option:: -w, --watch

   盯着集群的实时状态变更。

.. option:: --watch-debug

   盯着调试事件。

.. option:: --watch-info

   盯着一般信息事件。

.. option:: --watch-sec

   盯着安全事件。

.. option:: --watch-warn

   盯着告警事件。

.. option:: --watch-error

   盯着错误事件。

.. option:: --version, -v

   显示版本号。

.. option:: --verbose

   使输出更详细。

.. option:: --concise

   使输出简洁些。

.. option:: -f {json,json-pretty,xml,xml-pretty,plain}, --format

   输出格式。

.. option:: --connect-timeout CLUSTER_TIMEOUT

   设置连接集群的超时值。

.. option:: --no-increasing

   ``--no-increasing`` 默认是关闭的，所以 ``reweight-by-utilization``
   或 ``test-reweight-by-utilization`` 命令可以增加 osd 权重。\
   如果运行这些命令时加上这个选项，即使 osd 利用率偏低它也不会\
   增加 osd 权重。

.. option:: --block

   完成前一直阻塞（仅适用于 scrub 和 deep-scrub ）


使用范围
========

:program:`ceph` 是 Ceph 的一部分，这是个伸缩力强、开源、分布式\
的存储系统，更多信息参见 http://ceph.com/docs 。


参考
====

:doc:`ceph-mon <ceph-mon>`\(8),
:doc:`ceph-osd <ceph-osd>`\(8),
:doc:`ceph-mds <ceph-mds>`\(8)
