============
 归置组状态
============

检查集群状态时（如运行 ``ceph -s`` 或 ``ceph -w`` ）， Ceph 会\
报告归置组状态。一个归置组有一到多种状态，其最优状态为
``active+clean`` 。

*creating*
  Ceph 仍在创建归置组。

*activating*
  归置组互联好了，但还没活跃起来。

*active*
  Ceph 可处理到归置组的请求。

*clean*
  Ceph 把归置组内的对象复制了规定次数。

*down*
  包含必备数据的副本挂了，所以归置组离线。

*laggy*
  一个副本没有及时确认主 PG 签发的新租期， IO 临时停顿了。

*wait*
  负责此 PG 的 OSD 集合刚刚变更了、而且在前一个租期间隔过期前
  IO 临时停顿了，

*scrubbing*
  Ceph 正在检查归置组元数据，看是否有不一致的地方。

*deep*
  Ceph 正在检查比对归置组数据与存储的校验和。

*degraded*
  归置组内的对象还没复制到规定次数。

*inconsistent*
  Ceph 检测到了归置组内某一对象的一或多个副本间不一致（如各对\
  象大小不一、恢复后对象还没复制到副本那里、等等）。

*peering*
  归置组正在互联。

*repair*
  Ceph 正在检查归置组、并试图修复发现的不一致（如果可能的话）。

*recovering*
  Ceph 正在迁移/同步对象及其副本。

*forced_recovery*
  用户人为提高了那个 PG 的恢复优先级。

*recovery_wait*
  此归置组正排队等着启动恢复。

*recovery_toofull*
  一个恢复操作正在等待，因为目标 OSD 超过了它的占满率。

*recovery_unfound*
  由于有找不到的对象，恢复已停止。

*backfilling*
  Ceph 正在扫描并同步整个归置组的内容，而不是根据日志推算哪些\
  最新操作需要同步。 *Backfill* 是恢复的一种特殊情况。

*forced_backfill*
  用户人为提高了那个 PG 的回填优先级。

*backfill_wait*
  归置组正在排队，等候回填。

*backfill_toofull*
  一回填操作在等待，因为目标 OSD 使用率超过了占满率。

*backfill_unfound*
  由于有找不到的对象，回填已停止。

*incomplete*
  Ceph 探测到某一归置组丢失了写入信息，或者没有健康的副本。\
  如果你遇到了这个状态，试着启动一下可能包含所需信息的\
  失败 OSD 。如果是纠删码存储池，临时降低 min_size 也许能完成\
  恢复。

*stale*
  归置组处于一种未知状态——归置组运行图变更后就没再收到它的更新。

*remapped*
  归置组被临时映射到了另外一组 OSD ，它们不是 CRUSH 算法指定的。

*undersized*
  此归置组的副本数小于配置的存储池副本水平。

*peered*
  此归置组已互联，但是不能向客户端提供服务，因为其副本数没达到\
  本存储池的配置值（ min_size 参数）。在此状态下可以进行恢复，\
  所以此归置组最终能达到 min_size 。

*snaptrim*
  正在修剪快照。

*snaptrim_wait*
  在队列里等着修剪快照。

*snaptrim_error*
  修剪快照时遇挫，已停止。

*unknown*
  自从 mgr 启动以来， ceph-mgr 还没从 OSD 收到有关此 PG 状态的\
  任何信息。
