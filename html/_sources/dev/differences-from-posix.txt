=====================
 与 POSIX 标准的差异
=====================


Ceph 确实由于各种原因在某些地方偏离了严谨的 POSIX 语义：

- Sparse files propagate incorrectly to tools like df. They will only
  use up the required space, but in df will increase the "used" space
  by the full file size. We do this because actually keeping track of
  the space a large, sparse file uses is very expensive.
- In shared simultaneous writer situations, a write that crosses
  object boundaries is not necessarily atomic. This means that you
  could have writer A write "aa|aa" and writer B write "bb|bb"
  simultaneously (where | is the object boundary), and end up with
  "aa|bb" rather than the proper "aa|aa" or "bb|bb".
