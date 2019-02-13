===============
 jerasure 插件
===============


简介
----

jerasure 插件可接受的参数有：

::
 
	ceph osd erasure-code-profile set myprofile \
		directory=<dir>         \ # plugin directory absolute path
		plugin=jerasure         \ # plugin name (only jerasure)
		k=<k>                   \ # data chunks (default 2)
		m=<m>                   \ # coding chunks (default 2)
		technique=<technique>   \ # coding technique

可用的编码技术有： *reed_sol_van* 、 *reed_sol_r6_op* 、 *cauchy_orig* 、 \
*cauchy_good* 、 *liberation* 、 *blaum_roth* 和 *liber8tion* 。

此插件的源码位于 *src/erasure-code/jerasure* 目录下。它只是这里代码的封装： \
`https://github.com/ceph/jerasure <https://github.com/ceph/jerasure>`_
和 `https://github.com/ceph/gf-complete <https://github.com/ceph/gf-complete>`_ ，\
它们通过 *.gitmodules* 配置被嵌进了最新稳定版。该配置里的软件库是上游软件库\
（ `http://jerasure.org/jerasure/jerasure \
<http://jerasure.org/jerasure/jerasure>`_ 和 \
`http://jerasure.org/jerasure/gf-complete \
<http://jerasure.org/jerasure/gf-complete>`_ ）的副本。二者的差异，如果有的\
话，应该只是拉取请求的不同。
