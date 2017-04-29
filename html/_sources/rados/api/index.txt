===================
 Ceph 存储集群 API
===================

:term:`Ceph 存储集群`\ 提供了消息传递层协议，用于客户端与 :term:`Ceph 监视器`\ 和 \
:term:`OSD` 交互， ``librados`` 以库形式为 :term:`Ceph 客户端`\ 提供了这个功能。\
所有 Ceph 客户端可以用 ``librados`` 或 ``librados`` 里封装的相同功能和对象存储交\
互，例如 ``librbd`` 和 ``libcephfs`` 就利用了此功能。你可以用 ``librados`` 直接\
和 Ceph 交互（如和 Ceph 兼容的应用程序、你自己的 Ceph 接口、等等）。


.. toctree::
   :maxdepth: 2 

   librados 简介 <librados-intro>
   librados (C) <librados>
   librados (C++) <libradospp>
   librados (Python) <python>
