MDS 内部数据结构
================

*CInode*
  CInode 包含某一文件的元数据，每个文件都有一个 CInode 。它\
  存储着类似谁拥有这个文件、此文件有多大这样的信息。

*CDentry*
  CDentry 用于把索引节点和文件（或目录）名关联到一起。一个
  CDentry 最多可链接到一个 CInode （也可以不链接任何 CInode
  ），一个 CInode 可被多个 CDentry 链接。

*CDir*
  CDir 仅存在于目录索引节点下，它用于在目录下链接 CDentry 。\
  目录被分片时，一个 CInode 可以有多个 CDir 。

这些数据结构可以这样链接： ::

  CInode
  CDir
   |   \
   |      \
   |         \
  CDentry   CDentry
  CInode    CInode
  CDir      CDir
   |         |  \
   |         |     \
   |         |        \
  CDentry   CDentry  CDentry
  CInode    CInode   CInode

完成此文档时， CInode 的大小约为 1400 字节， CDentry 约为
400 字节， CDir 约为 700 字节。这些数据结构非常大，如果你想\
加入新字段，要多加小心。

*OpenFileTable*
  打开文件表用于跟踪打开的文件、及其父目录。（由于此表的存在）
  MDS 恢复时可以轻松获取已打开文件的路径，节省了大量载入\
  已打开文件的索引节点的时间。表里的每条都对应一个索引节点，它\
  记录的是索引节点的链接信息（父索引节点和 dentry 名字）。 MDS
  通过递归地查询父级索引节点的链接关系就能描绘出此索引节点的\
  路径。打开文件表存储在 RADOS 对象的 omap 里，表的条目与 omap
  内的键值对相对应。
