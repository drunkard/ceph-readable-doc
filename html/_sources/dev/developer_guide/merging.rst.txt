.. _merging:

合并提交：范围和节奏
====================

提交会根据 Ceph 发布周期内的特定条件并入各个分支，本章整理出了这些条件。

开发版（即 x.0.z ）
-------------------

What ?
^^^^^^

* features
* bug fixes

Where ?
^^^^^^^

Features are merged to the master branch. Bug fixes should be merged
to the corresponding named branch (e.g. "jewel" for 10.0.z, "kraken"
for 11.0.z, etc.). However, this is not mandatory - bug fixes can be
merged to the master branch as well, since the master branch is
periodically merged to the named branch during the development
releases phase. In either case, if the bugfix is important it can also
be flagged for backport to one or more previous stable releases.

When ?
^^^^^^

After the stable release candidates of the previous release enters
phase 2 (see below).  For example: the "jewel" named branch was
created when the infernalis release candidates entered phase 2. From
this point on, master was no longer associated with infernalis. As
soon as the named branch of the next stable release is created, master
starts getting periodically merged into it.

分支合并
^^^^^^^^
.. Branch merges

* 稳定版的分支会周期性地并入 master ；
* master 分支会周期性地并入稳定版；
* 每次开发版 x.0.z 发布后， master 会立即并入稳定版分支；

稳定版候选（即 x.1.z ）阶段一
-----------------------------
.. Stable release candidates (i.e. x.1.z) phase 1

What ?
^^^^^^

* bug fixes only

Where ?
^^^^^^^

The branch of the stable release (e.g. "jewel" for 10.0.z, "kraken"
for 11.0.z, etc.) or master.  Bug fixes should be merged to the named
branch corresponding to the stable release candidate (e.g. "jewel" for
10.1.z) or to master. During this phase, all commits to master will be
merged to the named branch, and vice versa. In other words, it makes
no difference whether a commit is merged to the named branch or to
master - it will make it into the next release candidate either way.

When ?
^^^^^^

After the first stable release candidate is published, i.e. after the
x.1.0 tag is set in the release branch.

Branch merges
^^^^^^^^^^^^^

* 稳定版的分支会周期性地并入 master ；
* master 分支会周期性地并入稳定版；
* 每次候选版 x.1.z 发布后， master 会立即并入稳定版分支；

稳定版候选（即 x.1.z ）阶段二
-----------------------------
.. Stable release candidates (i.e. x.1.z) phase 2

What ?
^^^^^^
* bug fixes only

Where ?
^^^^^^^
The branch of the stable release (e.g. "jewel" for 10.0.z, "kraken"
for 11.0.z, etc.). During this phase, all commits to the named branch
will be merged into master. Cherry-picking to the named branch during
release candidate phase 2 is done manually since the official
backporting process only begins when the release is pronounced
"stable".

When ?
^^^^^^

After Sage Weil decides it is time for phase 2 to happen.

分支合并
^^^^^^^^
.. Branch merges

* The branch of the stable release is merged periodically into master.

稳定版（即 x.2.z ）
-------------------
.. Stable releases (i.e. x.2.z)

What ?
^^^^^^

* bug fixes
* features are sometime accepted
* commits should be cherry-picked from master when possible

* commits that are not cherry-picked from master must be about a bug unique to
  the stable release
* see also `the backport HOWTO`_

.. _`the backport HOWTO`:
  http://tracker.ceph.com/projects/ceph-releases/wiki/HOWTO#HOWTO

Where ?
^^^^^^^

The branch of the stable release (hammer for 0.94.x, infernalis for 9.2.x,
etc.)

When ?
^^^^^^

After the stable release is published, i.e. after the "vx.2.0" tag is set in
the release branch.

分支合并
^^^^^^^^
.. Branch merges

不会再并入。

