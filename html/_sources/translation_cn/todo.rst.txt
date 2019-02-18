TODO for Chinese
================

Test ditaa 0.11.0
术语上方标注原文词语，用 HTML5 的 ruby + rt TAG 实现，但是rst现在还没有原生支持。
github 上 README 内的链接无效


TODO for raw doc (English version)
==================================

Cache Tier modes, do compare in table, include these column at least:
	cache-mode
	Description
	Usage scene
	Consistency

Internal mechanism of federated radosgw config, or the lowest(needed pools
at least) configuration for it.

Missed config option in doc, compares to list of 'ceph daemon osd.0 config show'

option warnings fix:

	/git/ceph/doc/man/8/ceph-deploy.rst:146: WARNING: option reference target not found: --mkfs
	/git/ceph/doc/man/8/ceph-disk.rst:84: WARNING: option reference target not found: --activate-key
	/git/ceph/doc/man/8/ceph-disk.rst:91: WARNING: option reference target not found: --mark-init
	/git/ceph/doc/man/8/ceph-disk.rst:104: WARNING: option reference target not found: --no-start-daemon
	/git/ceph/doc/man/8/ceph-disk.rst:107: WARNING: option reference target not found: --reactivate
	/git/ceph/doc/man/8/ceph-disk.rst:127: WARNING: option reference target not found: --activate-key
	/git/ceph/doc/man/8/ceph-disk.rst:127: WARNING: option reference target not found: --mark-init
	/git/ceph/doc/man/8/ceph-disk.rst:148: WARNING: option reference target not found: --activate-key
	/git/ceph/doc/man/8/ceph-disk.rst:148: WARNING: option reference target not found: --mark-init
	/git/ceph/doc/man/8/ceph-disk.rst:198: WARNING: option reference target not found: --reactivate
	/git/ceph/doc/man/8/ceph-disk.rst:212: WARNING: option reference target not found: --mark-out
	/git/ceph/doc/man/8/ceph-disk.rst:216: WARNING: option reference target not found: --deactivate-by-id
	/git/ceph/doc/man/8/ceph-disk.rst:235: WARNING: option reference target not found: --zap
	/git/ceph/doc/man/8/ceph-disk.rst:242: WARNING: option reference target not found: --destroy-by-id


Bad links to fix
----------------

Bad links to shoot down. They are grabbed in this command::

   # command ?

Below is bad links found by the command:

architecture.rst:91: [broken] http://ceph.com/papers/weil-crush-sc06.pdf: HTTP Error 404: Not Found
cephfs/mantle.rst:40: [broken] https://sourceforge.net/projects/mdtest/: [Errno 104] Connection reset by peer
dev/cpu-profiler.rst:5: [broken] http://oprofile.sourceforge.net/about/: [Errno 104] Connection reset by peer
dev/development-workflow.rst:126: [broken] http://ceph.com/resources/mailing-list-irc/: HTTP Error 404: Not Found
dev/development-workflow.rst:57: [broken] http://wiki.ceph.com/Planning/CDS/: HTTP Error 404: Not Found
dev/documenting.rst:62: [broken] http://ditaa.sourceforge.net/: [Errno 104] Connection reset by peer
dev/documenting.rst:78: [broken] http://ditaa.sourceforge.net/: [Errno 104] Connection reset by peer
dev/documenting.rst:27: [broken] https://raw.github.com/ceph/ceph/master/doc/api/librados.rst: HTTP Error 404: Not Found
dev/index.rst:135: [broken] https://ceph.com/resources/mailing-list-irc/: HTTP Error 404: Not Found
dev/index.rst:23: [broken] https://ceph.com/resources/publications/: <urlopen error ('_ssl.c:645: The handshake operation timed out',)>
dev/index.rst:429: [broken] https://github.com/mygithubaccount/ceph: HTTP Error 404: Not Found
dev/index.rst:147: [broken] https://ceph.com/resources/mailing-list-irc/: HTTP Error 404: Not Found
dev/index.rst:449: [broken] https://github.com/mygithubaccount/ceph.git: HTTP Error 404: Not Found
dev/index.rst:721: [broken] http://ceph.com/resources/mailing-list-irc/: HTTP Error 404: Not Found
dev/index.rst:805: [broken] http://docs.ceph.com/teuthology/docs/LAB_SETUP.html: <urlopen error [Errno 101] Network is unreachable>
dev/index.rst:1109: [broken] https://github.com/ceph/ceph/tree/master/qa/suites/rados/thrash/fs: HTTP Error 404: Not Found
dev/index.rst:1174: [broken] https://docs.docker.com/linux/: HTTP Error 403: Forbidden
dev/osd_internals/erasure_coding/developer_notes.rst:190: [broken] https://github.com/ceph/ceph/blob/v0.78/src/test/osd/ErasureCodePluginExample.cc: HTTP Error 404: Not Found
dev/placement-group.rst:46: [broken] http://ceph.com/wiki/Changing_the_number_of_PGs: HTTP Error 404: Not Found
dev/rbd-layering.rst:87: [broken] http://marc.info/?l=ceph-devel&m=129867273303846: <urlopen error [Errno -3] Temporary failure in name resolution>
install/get-packages.rst:141: [broken] https://download.ceph.com/packages/google-perftools/debian: HTTP Error 404: Not Found
install/get-packages.rst:147: [broken] https://download.ceph.com/debian-testing/dists: ('The read operation timed out',)
install/mirrors.rst:16: [broken] http://au.ceph.com/: <urlopen error [Errno 101] Network is unreachable>
install/upgrading-ceph.rst:449: [broken] http://download.ceph.com/rpm: timed out
install/upgrading-ceph.rst:187: [broken] http://ceph.com/dev-notes/cephs-new-monitor-changes: timed out
rados/operations/crush-map.rst:11: [broken] http://ceph.com/papers/weil-crush-sc06.pdf: HTTP Error 404: Not Found
rados/operations/crush-map.rst:374: [broken] http://ceph.com/papers/weil-crush-sc06.pdf: HTTP Error 404: Not Found
rados/operations/crush-map.rst:443: [broken] http://ceph.com/papers/weil-crush-sc06.pdf: HTTP Error 404: Not Found
rados/operations/erasure-code-jerasure.rst:7: [broken] https://bitbucket.org/jimplank/jerasure/: HTTP Error 404: Not Found
rados/operations/operating.rst:111: [broken] http://manpages.ubuntu.com/manpages/raring/en/man8/initctl.8.html: HTTP Error 404: Not Found
rados/troubleshooting/cpu-profiling.rst:5: [broken] http://oprofile.sourceforge.net/about/: [Errno 104] Connection reset by peer
rados/troubleshooting/memory-profiling.rst:60: [broken] http://google-perftools.googlecode.com/svn/trunk/doc/heapprofile.html: <urlopen error [Errno 101] Network is unreachable>
radosgw/keystone.rst:25: [broken] http://docs.openstack.org/developer/keystone/configuringservices.html: HTTP Error 404: Not Found
radosgw/s3/ruby.rst:196: [broken] http://amazon.rubyforge.org/: <urlopen error timed out>
radosgw/s3/ruby.rst:216: [broken] http://amazon.rubyforge.org/doc/: <urlopen error timed out>
rbd/libvirt.rst:165: [broken] http://www.libvirt.org/virshcmdref.html: timed out
rbd/rbd-cloudstack.rst:84: [broken] http://cloudstack.apache.org/docs/en-US/Apache_CloudStack/4.2.0/html/Admin_Guide/primary-storage-add.html: HTTP Error 404: Not Found
rbd/rbd-cloudstack.rst:102: [broken] http://cloudstack.apache.org/docs/en-US/Apache_CloudStack/4.2.0/html/Admin_Guide/compute-disk-service-offerings.html: HTTP Error 404: Not Found
rbd/rbd-openstack.rst:68: [broken] http://docs.ceph.com/docs/dumpling/rbd/rbd-openstack: <urlopen error [Errno 101] Network is unreachable>
release-notes.rst:152: [broken] https://github.com/ceph/ceph/pull/13617: ('The read operation timed out',)
release-notes.rst:1222: [broken] http://github.com/ceph/ceph/pull/12817: ('The read operation timed out',)
release-notes.rst:3053: [broken] https://: <urlopen error no host given>
release-notes.rst:3065: [broken] http://github.com/ceph/ceph/pull/9585: ('The read operation timed out',)
release-notes.rst:4860: [broken] http://github.com/ceph/ceph/pull/7075: ('The read operation timed out',)
release-notes.rst:5133: [broken] http://github.com/ceph/ceph/pull/6440: ('The read operation timed out',)
release-notes.rst:5201: [broken] http://github.com/ceph/ceph/pull/6898: ('The read operation timed out',)
release-notes.rst:6612: [broken] http://github.com/ceph/ceph/pull/6898: ('The read operation timed out',)
release-notes.rst:7040: [broken] http://github.com/ceph/ceph/pull/7075: ('The read operation timed out',)
release-notes.rst:7072: [broken] http://github.com/ceph/ceph/pull/6440: ('The read operation timed out',)
start/documenting-ceph.rst:291: [broken] http://ditaa.sourceforge.net/: [Errno 104] Connection reset by peer
start/documenting-ceph.rst:291: [broken] http://ditaa.sourceforge.net/: [Errno 104] Connection reset by peer
start/documenting-ceph.rst:291: [broken] http://ditaa.sourceforge.net/: [Errno 104] Connection reset by peer
start/documenting-ceph.rst:291: [broken] http://ditaa.sourceforge.net/: [Errno 104] Connection reset by peer
start/documenting-ceph.rst:291: [broken] http://ditaa.sourceforge.net/: [Errno 104] Connection reset by peer
start/documenting-ceph.rst:433: [broken] http://docutils.sourceforge.net/docs/user/rst/quickstart.html: [Errno 104] Connection reset by peer
start/documenting-ceph.rst:436: [broken] http://docutils.sourceforge.net/docs/user/rst/quickstart.html: [Errno 104] Connection reset by peer
start/documenting-ceph.rst:493: [broken] http://docutils.sourceforge.net/docs/ref/rst/directives.html: [Errno 104] Connection reset by peer
start/get-involved.rst:14: [broken] http://ceph.com/community/planet-ceph/: HTTP Error 404: Not Found
start/get-involved.rst:28: [broken] http://news.gmane.org/gmane.comp.file-systems.ceph.user: HTTP Error 404: Not Found
start/get-involved.rst:32: [broken] http://news.gmane.org/gmane.comp.file-systems.ceph.devel: HTTP Error 404: Not Found
start/get-involved.rst:17: [broken] https://wiki.ceph.com/: <urlopen error timed out>
start/quick-start-preflight.rst:62: [broken] https://fedoraproject.org/wiki/EPEL: <urlopen error ('_ssl.c:645: The handshake operation timed out',)>
