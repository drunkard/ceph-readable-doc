=====================
 Jerasure 纠删码插件
=====================
.. Jerasure erasure code plugin

*jerasure* 插件是个通用、灵活的插件。
然而， *jerasure* 库不再维护，且没有为当下的 CPU 指令集做相应更新，
因此无法提升编码和解码数据时的性能。

*jerasure* 插件封装了 `Jerasure
<https://github.com/ceph/jerasure>`_ 库，
要想更好地理解各参数，建议您阅读 ``jerasure`` 文档。
注意， ``jerasure.org`` 网站到 2023 年 5 月\
就和最初的项目没关联了，可能是冒牌的。


创建 jerasure 配置
==================
.. Create a jerasure profile

要新建 *jerasure* 纠删码配置：

.. prompt:: bash $

   ceph osd erasure-code-profile set {name} \
     plugin=jerasure \
     k={data-chunks} \
     m={coding-chunks} \
     technique={reed_sol_van|reed_sol_r6_op|cauchy_orig|cauchy_good|liberation|blaum_roth|liber8tion} \
     [crush-root={root}] \
     [crush-failure-domain={bucket-type}] \
     [crush-device-class={device-class}] \
     [directory={directory}] \
     [--force]


其中：

``k={data chunks}``

:描述: 各对象都被分割为\ **数据块**\ ，分别存储于不同 OSD 。
:类型: Integer
:是否必需: Yes.
:实例: 4


``m={coding-chunks}``

:描述: 计算各对象的\ **编码块**\ 、并存储于不同 OSD 。编码块的\
       数量等同于在不丢数据的前提下允许同时失效的 OSD 数量。

:类型: Integer
:是否必需: Yes.
:实例: 2


``technique={reed_sol_van|reed_sol_r6_op|cauchy_orig|cauchy_good|liberation|blaum_roth|liber8tion}``

:描述: *reed_sol_van* 技术更灵活：它足以设置 *k* 和 *m* 值。
       *cauchy_good* 技术更快，但你得谨慎地选择 *packetsize*
       值。 *reed_sol_r6_op* 、 *liberation* 、
       *blaum_roth* 、 *liber8tion* 都是与 *RAID6* 等价的技\
       术，它们只能配置为 *m=2* 。

       .. note:: 使用 ``blaum_roth`` 编码时，
          默认的 ``w=7`` 词大小是不理想的，
          因为当 ``w+1`` 是质数时，
          ``blaum_roth`` 的状态最佳。
          在使用 ``technique=blaum_roth`` 新建纠删码配置文件时，
          要把 ``w`` 设为比质数小 1 的整数（例如 ``6`` ）。
          参见 `Loic Dachary 向 ceph/ceph 提交的 f51d21b
          <https://github.com/ceph/ceph/commit/f51d21b53d26d4f27c950cb1ba3f989e713ab325>`_ ，
          以了解为何不能在源代码中轻易更改这一默认值，
          并参见 `Plank 和 Greenan 写的
          《Jerasure：一个为存储应用程序简化纠删编码的 C 库》
          第 29 页、项目符号第二条，
          <https://github.com/ceph/jerasure/blob/master/Manual.pdf>`_
          在使用 Blaum-Roth 编码时适用于 ``w`` 明确声明的限制。
          (有关使用 ``blaum_roth`` 编码时 ``w``
          正确值的信息由 Benjamin Mare 于
          2024 年 9 月提供给 Ceph 上游）。

:类型: String
:是否必需: No.
:默认值: reed_sol_van


``packetsize={bytes}``

:描述: 每次以 *bytes* 大小的包为单位进行编码。
       确定合适的包尺寸很难，
       *jerasure* 文档对此有很详细的描述。

:类型: Integer
:是否必需: No.
:默认值: 2048


``crush-root={root}``

:描述: CRUSH 规则的第一步所用的 crush 桶的名字，
       例如 **step take default** 。

:类型: String
:是否必需: No.
:默认值: default


``crush-failure-domain={bucket-type}``

:描述: 确保两个数据块不会存储到同一故障域中的桶里。
       例如，假设故障域是 **host** ，
       那么两个数据块就不会存储到同一主机。
       它被用于创建 CRUSH 规则 step ，
       如 **step chooseleaf host** 。

:类型: String
:是否必需: No.
:默认值: host


``crush-device-class={device-class}``

:描述: 使归置限于指定的设备类（比如 ``ssd`` 或 ``hdd`` ）
       之内，在 CRUSH 图内用的是 crush 设备类的名字。

:类型: String
:是否必需: No.


``directory={directory}``

:描述: 设置纠删码插件的路径，需是\ **目录**\ 。
:类型: String
:是否必需: No.
:默认值: /usr/lib/ceph/erasure-code


``--force``

:描述: 覆盖同名配置。
:类型: String
:是否必需: No.
