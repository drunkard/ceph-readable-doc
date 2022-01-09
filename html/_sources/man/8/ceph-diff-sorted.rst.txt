:orphan:

==============================================
 ceph-diff-sorted -- 逐行比较两个排序过的文件
==============================================

.. program:: ceph-diff-sorted

提纲
====

| **ceph-diff-sorted** *file1* *file2*


描述
====

:program:`ceph-diff-sorted` 是个简单的 *diff* 工具，
对已经按字典排序过的两个文件的比较做过针对性的优化。

与 POSIX 系统里的标准 `diff` 工具相比，它的输出简化了。
尖括号（ < 和 > ）用于展示在一个文件却不在另一个文件里的行。
这个输出与 `patch` 工具不兼容。

这个工具的出现是为了比较巨型文件（例如包含数十亿行的），
而标准的 `diff` 工具不能高效地处理。已知行是排序过的，
此工具才能在内存开销最低的情况下高效地处理。

每个文件都需要先完成按字典排序。大多数 POSIX 系统\
都用 *LANG* 环境变量来确定 `sort` 工具的排序方式。
要按字典顺序排序，我们需要这样：

        $ LANG=C sort some-file.txt >some-file-sorted.txt


实例
====

比较两个文件::

        $ ceph-diff-sorted fileA.txt fileB.txt

退出状态
========
.. Exit Status

完成时，退出码会设置成下列之一：

0
  文件相同
1
  文件不同
2
  用法问题（例如命令行参数个数不对）
3
  打开输入文件遇到问题
4
  文件内容有问题（例如没排序过、或遇到空行）


使用范围
========

**ceph-diff-sorted** 是 Ceph 的一部分，这是个伸缩力强、开源、
分布式的存储系统，更多信息参见 https://docs.ceph.com 。


参考
====

:doc:`rgw-orphan-list <rgw-orphan-list>`\(8)
