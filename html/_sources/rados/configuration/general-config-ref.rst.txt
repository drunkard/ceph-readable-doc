==============
 常规配置参考
==============

.. confval:: admin_socket
   :default: /var/run/ceph/$cluster-$name.asok
.. confval:: pid_file
.. confval:: chdir
.. confval:: fatal_signal_handlers
.. describe:: max_open_files

   如果设置了， :term:`Ceph 存储集群`\ 启动时
   会设置操作系统级的最大打开 FD 数量，
   即文件描述符最大允许值，
   大小合适的值可防止 Ceph 守护进程们耗尽文件描述符。

   :类型: 64-bit Integer
   :是否必需: No
   :默认值: ``0``
