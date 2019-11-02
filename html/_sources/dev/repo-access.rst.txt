Notes on Ceph repositories
==========================

.. Special branches

特殊分支
--------

* ``master``: current tip (integration branch)
* 与 :ref:`ceph-releases` 中罗列的版本相对应的发布分支（比如
  ``luminous`` ）。


.. Rules

规则
----

源码库全都在 github 上。

* Any branch pushed to ceph-ci.git will kick off builds that will
  generate packages and repositories on shaman.ceph.com. Try
  not to generate unnecessary load.  For private, unreviewed work,
  only push to branches named ``wip-*``.  This avoids colliding with
  any special branches.

* 特殊分支不应该合并未经重审的东西；

* Preferred means of review is via github pull requests to capture any
  review discussion.

* For multi-patch series, the pull request can be merged via github,
  and a Reviewed-by: ... line added to the merge commit.

* For single- (or few-) patch merges, it is preferable to add the
  Reviewed-by: directly to the commit so that it is also visible when
  the patch is cherry-picked for backports.

* All backports should use ``git cherry-pick -x`` to capture which
  commit they are cherry-picking from.
