必备知识
========
.. Essentials (tl;dr)

本章包含必要信息，每个 Ceph 开发者都应该知道。


项目领导
--------
.. Leads

Ceph 项目是由 Sage Weil 领导的。另外，各主要项目组件有自己的领\
导，下面的表格罗列了所有领导、以及他们在 `GitHub`_ 上的昵称。

.. _github: https://github.com/

========= ================ =============
Scope     Lead             GitHub nick
========= ================ =============
Ceph      Sage Weil        liewegas
RADOS     Neha Ojha        neha-ojha
RGW       Yehuda Sadeh     yehudasa
RGW       Matt Benjamin    mattbenjamin
RBD       Jason Dillaman   dillaman
CephFS    Patrick Donnelly batrick
Dashboard Lenz Grimmer     LenzGr
MON       Joao Luis        jecluis
Build/Ops Ken Dreyer       ktdreyer
Docs      Zac Dover        zdover23
========= ================ =============

上述表格里的 Ceph 专有缩写在 :doc:`/architecture` 里面有解释。


历史
----
.. History

请翻阅 `Wikipedia 的 History 这章`_\ 。

.. _`Wikipedia 的 History 这章`: https://en.wikipedia.org/wiki/Ceph_%28software%29#History


软件许可
--------
.. Licensing

Ceph 是自由软件。

除非另有声明， Ceph 的源代码会以 LGPL2.1 或 LGPL3.0 许可证\
发布。其完整内容在源码树顶极目录下的 `COPYING`_ 文件内。

.. _`COPYING`:
   https://github.com/ceph/ceph/blob/master/COPYING


源代码仓库
----------
.. Source code repositories

The source code of Ceph lives on `GitHub`_ in a number of repositories below
the `Ceph "organization"`_.

.. _`Ceph "organization"`: https://github.com/ceph

To make a meaningful contribution to the project as a developer, a working
knowledge of git_ is essential.

.. _git: https://git-scm.com/doc

Although the `Ceph "organization"`_ includes several software repositories,
this document covers only one: https://github.com/ceph/ceph.


Redmine 问题跟踪器
------------------
.. Redmine issue tracker

Although `GitHub`_ is used for code, Ceph-related issues (Bugs, Features,
Backports, Documentation, etc.) are tracked at http://tracker.ceph.com,
which is powered by `Redmine`_.

.. _Redmine: http://www.redmine.org

The tracker has a Ceph project with a number of subprojects loosely
corresponding to the various architectural components (see
:doc:`/architecture`).

Mere `registration`_ in the tracker automatically grants permissions
sufficient to open new issues and comment on existing ones.

.. _registration: http://tracker.ceph.com/account/register

要报告软件缺陷或者提议新功能，请\ `跳转到 Ceph 项目`_\ 并点击 \
`New issue`_ 。

.. _`跳转到 Ceph 项目`: http://tracker.ceph.com/projects/ceph
.. _`New issue`: http://tracker.ceph.com/projects/ceph/issues/new


.. _mailing-list:

邮件列表
--------

Ceph Development Mailing List
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
The ``dev@ceph.io`` list is for discussion about the development of Ceph,
its interoperability with other technology, and the operations of the
project itself.

The email discussion list for Ceph development is open to all. Subscribe by
sending a message to ``dev-request@ceph.io`` with the following line in the
body of the message::

    subscribe ceph-devel


Ceph Client Patch Review Mailing List
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
The ``ceph-devel@vger.kernel.org`` list is for discussion and patch review
for the Linux kernel Ceph client component. Note that this list used to
be an all-encompassing list for developers. When searching the archives, 
remember that this list contains the generic devel-ceph archives before mid-2018.

Subscribe to the list covering the Linux kernel Ceph client component by sending
a message to ``majordomo@vger.kernel.org`` with the following line in the body
of the message::

    subscribe ceph-devel


Other Ceph Mailing Lists
~~~~~~~~~~~~~~~~~~~~~~~~

还有\ `其他与 Ceph 相关的邮件列表`_\ 。

.. _`其他与 Ceph 相关的邮件列表`: https://ceph.com/irc/


.. _irc:

IRC
---

In addition to mailing lists, the Ceph community also communicates in real
time using `Internet Relay Chat`_.

.. _`Internet Relay Chat`: http://www.irchelp.org/

The Ceph community gathers in the #ceph channel of the Open and Free Technology
Community (OFTC) IRC network.

Created in 1988, Internet Relay Chat (IRC) is a relay-based, real-time chat
protocol. It is mainly designed for group (many-to-many) communication in
discussion forums called channels, but also allows one-to-one communication via
private message. On IRC you can talk to many other members using Ceph, on
topics ranging from idle chit-chat to support questions. Though a channel might
have many people in it at any one time, they might not always be at their
keyboard; so if no-one responds, just wait around and someone will hopefully
answer soon enough.

Registration
~~~~~~~~~~~~

If you intend to use the IRC service on a continued basis, you are advised to
register an account. Registering gives you a unique IRC identity and allows you
to access channels where unregistered users have been locked out for technical
reasons.


Channels
~~~~~~~~

To connect to the OFTC IRC network, download an IRC client and configure it to
connect to ``irc.oftc.net``. Then join one or more of the channels. Discussions
inside #ceph are logged and archives are available online.

Here are the real-time discussion channels for the Ceph community:

  -  #ceph
  -  #ceph-devel
  -  #cephfs
  -  #ceph-dashboard
  -  #ceph-orchestrators
  -  #sepia


.. _submitting-patches:

补丁的提交
----------

The canonical instructions for submitting patches are contained in the
file `CONTRIBUTING.rst`_ in the top-level directory of the source-code
tree. There may be some overlap between this guide and that file.

.. _`CONTRIBUTING.rst`:
  https://github.com/ceph/ceph/blob/master/CONTRIBUTING.rst

All newcomers are encouraged to read that file carefully.


从源码构建
----------
.. Building from source

请参考 :doc:`/install/build-ceph` 。


用 ccache 加速本地构建
----------------------
.. Using ccache to speed up local builds

`ccache`_ can make the process of rebuilding the ceph source tree faster. 

Before you use `ccache`_ to speed up your rebuilds of the ceph source tree,
make sure that your source tree is clean and will produce no build failures.
When you have a clean source tree, you can confidently use `ccache`_, secure in
the knowledge that you're not using a dirty tree.

Old build artifacts can cause build failures. You might introduce these
artifacts unknowingly when switching from one branch to another. If you see
build errors when you attempt a local build, follow the procedure below to
clean your source tree.

Cleaning the Source Tree
~~~~~~~~~~~~~~~~~~~~~~~~

.. prompt:: bash $

  ninja clean
  
.. note:: The following commands will remove everything in the source tree 
          that isn't tracked by git. Make sure to back up your log files 
          and configuration options before running these commands.

.. prompt:: bash $

   git clean -fdx; git submodule foreach git clean -fdx

Building Ceph with ccache
~~~~~~~~~~~~~~~~~~~~~~~~~

``ccache`` is available as a package in most distros. To build ceph with
ccache, run the following command.

.. prompt:: bash $

  cmake -DWITH_CCACHE=ON ..

Using ccache to Speed Up Build Times
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``ccache`` can be used for speeding up all builds of the system. For more
details, refer to the `run modes`_ section of the ccache manual. The default
settings of ``ccache`` can be displayed with the ``ccache -s`` command.

.. note:: We recommend overriding the ``max_size``. The default is 10G.
          Use a larger value, like 25G. Refer to the `configuration`_ section
          of the ccache manual for more information.

To further increase the cache hit rate and reduce compile times in a
development environment, set the version information and build timestamps to
fixed values. This makes it unnecessary to rebuild the binaries that contain
this information.

This can be achieved by adding the following settings to the ``ccache``
configuration file ``ccache.conf``::

  sloppiness = time_macros
  run_second_cpp = true

Now, set the environment variable ``SOURCE_DATE_EPOCH`` to a fixed value (a
UNIX timestamp) and set ``ENABLE_GIT_VERSION`` to ``OFF`` when running
``cmake``:

.. prompt:: bash $

  export SOURCE_DATE_EPOCH=946684800
  cmake -DWITH_CCACHE=ON -DENABLE_GIT_VERSION=OFF ..

.. note:: Binaries produced with these build options are not suitable for
  production or debugging purposes, as they do not contain the correct build
  time and git version information.

.. _`ccache`: https://ccache.samba.org/
.. _`run modes`: https://ccache.samba.org/manual.html#_run_modes
.. _`configuration`: https://ccache.samba.org/manual.html#_configuration


开发模式的集群
--------------
.. Development-mode cluster

参考 :doc:`/dev/quick_guide` 。


Kubernetes/Rook 开发集群
------------------------
.. Kubernetes/Rook development cluster

参考 :ref:`kubernetes-dev`


.. _backporting:

补丁移植（ Backporting ）
-------------------------

All bugfixes should be merged to the ``master`` branch before being
backported. To flag a bugfix for backporting, make sure it has a
`tracker issue`_ associated with it and set the ``Backport`` field to a
comma-separated list of previous releases (e.g. "hammer,jewel") that you think
need the backport.
The rest (including the actual backporting) will be taken care of by the
`Stable Releases and Backports`_ team.

.. _`tracker issue`: http://tracker.ceph.com/
.. _`Stable Releases and Backports`: http://tracker.ceph.com/projects/ceph-releases/wiki

Guidance for use of cluster log
-------------------------------

If your patches emit messages to the Ceph cluster log, please consult
this: :doc:`/dev/logging`.
