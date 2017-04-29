================
 归置组术语解释
================

当你执行诸如 ``ceph -w`` 、 ``ceph osd dump`` 、及其他和归置组相关的命令时， \
Ceph 会返回下列术语及其值：

*Peering*
   （建立互联）
   是一种过程，它使得存储着同一归置组的所有 OSD 对归置组内的所有对象及其元数据统一\
   意见。需要注意的是，达成一致不意味着它们都有最新内容。

*Acting Set*
   （在任集合）
   一列有序 OSD ，它们为某一特定归置组（或其中一些元版本）负责。

*Up Set*
   The ordered list of OSDs responsible for a particular placement
   group for a particular epoch according to CRUSH. Normally this
   is the same as the *Acting Set*, except when the *Acting Set* has 
   been explicitly overridden via ``pg_temp`` in the OSD Map.

   ［译者：此处原文没看懂……瞎译如下］
   （当选集合）
   一列有序 OSD ，它们依据 CRUSH 算法为某一归置组的特定元版本负责。它通常和 \
   *Acting Set* 相同，除非 *Acting Set* 被 OSD 运行图里的 ``pg_temp`` 显式地覆盖\
   掉了。

*Current Interval* 或 *Past Interval*
   某一特定归置组所在 *Acting Set* 和 *Up Set* 未更改时的一系列 OSD 运行图元版本。

*Primary*
   （主 OSD ）
   *Acting Set* 的成员（按惯例第一个），它负责协调互联，并且是归置组内惟一接受客户\
   端初始写入的 OSD 。

*Replica*
   （副本 OSD ）
   一归置组的 *Acting Set* 内不是主 OSD 的其它 OSD ，它们被同等对待、并由主 OSD \
   *激活*\ 。

*Stray*
   （彷徨 OSD ）
   某一 OSD ，它不再是当前 *Acting Set* 的成员，但还没被告知它可以删除那个归置组副\
   本。

*Recovery*
   （恢复）
   确保 *Acting Set* 内、一归置组中的所有对象的副本都存在于所有 OSD 上。一旦\ \
   *互联*\ 完成，\ *主 OSD* 就以接受写操作，且\ *恢复*\ 进程可在后台进行。

*PG Info* 
   Basic metadata about the placement group's creation epoch, the version
   for the most recent write to the placement group, *last epoch started*, 
   *last epoch clean*, and the beginning of the *current interval*.  Any
   inter-OSD communication about placement groups includes the *PG Info*, 
   such that any OSD that knows a placement group exists (or once existed) 
   also has a lower bound on *last epoch clean* or *last epoch started*.

   （归置组信息）
   ［译者：此处原文没看透……瞎译如下］
   基本元数据，关于归置组创建元版本、向归置组的最新写版本、最近的开始元版本（ \
   *last epoch started* ）、最近的干净元版本（ *last epoch clean* ）、和当前间隔\
   （ *current interval* ）的起点。 OSD 间关于归置组的任何通讯都包含 PG Info ，这\
   样任何知道一归置组存在（或曾经存在）的 OSD 也必定有 *last epoch clean* 或 \
   *last epoch started* 的下限。

*PG Log*
   （归置组日志）
   一归置组内对象的一系列最近更新。注意，这些日志在 *Acting Set* 内的所有 OSD 确认\
   更新到某点后可以删除。

*Missing Set*
   （缺失集合）
   各 OSD 都记录更新日志，而且如果它们包含对象内容的更新，会把那个对象加入一个待更\
   新列表，这个列表叫做那个 ``<OSD,PG>`` 的 *Missing Set* 。

*Authoritative History*
   （权威历史）
   一个完整、完全有序的操作集合，如果再次执行，可把一 OSD 上的归置组副本还原到最新。

*Epoch*
   （元版本）
   单递增 OSD 运行图版本号。

*Last Epoch Start*
   （最新起始元版本）
   一最新元版本，在这点上，一归置组所对应 *Acting Set* 内的所有节点都对\ \
   *权威历史*\ 达成了一致、并且\ *互联*\ 被认为成功了。

*up_thru*
   （领导拍板）
   *主 OSD* 要想成功完成\ *互联*\ ，它必须通过当前 OSD 图\ *元版本*\ 通知一个监视\
   器，让此监视器在 osd 图中设置其 *up_thru* 。这会使\ *互联*\ 进程忽略之前的 \
   *Acting Set* ，因为它经历特定顺序的失败后一直不能\ *互联*\ ，比如像下面的第二周期：

   - *acting set* = [A,B]
   - *acting set* = [A]
   - *acting set* = [] 之后很短时间（例如同时失败、但探测是交叉的）
   - *acting set* = [B] （ B 重启了、但 A 没有）

*Last Epoch Clean*
   （最新干净元版本）
   最近的 *Epoch* ，这时某一特定归置组所在 *Acting Set* 内的所有节点都全部更新了\
   （包括归置组日志和对象内容）。在这点上，\ *恢复*\ 被认为已完成。
