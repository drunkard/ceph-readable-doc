=====================
 Jerasure 纠删码插件
=====================

*jerasure* 插件是最通用、最灵活的插件，也是 Ceph 纠删码存储池的默认插件。

*jerasure* 插件封装了 `Jerasure <https://bitbucket.org/jimplank/jerasure/>`_ \
库，要想更好地理解各参数，建议您仔细阅读 *jerasure* 文档。


创建 jerasure 配置
==================

要新建 *jerasure* 纠删码配置： ::

        ceph osd erasure-code-profile set {name} \
             plugin=jerasure \
             k={data-chunks} \
             m={coding-chunks} \
             technique={reed_sol_van|reed_sol_r6_op|cauchy_orig|cauchy_good|liberation|blaum_roth|liber8tion} \
             [ruleset-root={root}] \
             [ruleset-failure-domain={bucket-type}] \
             [directory={directory}] \
             [--force]

其中：


``k={data chunks}``

:描述: 各对象都被分割为\ **数据块**\ ，分别存储于不同 OSD 。
:类型: Integer
:是否必需: Yes.
:实例: 4


``m={coding-chunks}``

:描述: 计算各对象的\ **编码块**\ 、并存储于不同 OSD 。编码块的数量等同于在\
       不丢数据的前提下允许同时失效的 OSD 数量。

:类型: Integer
:是否必需: Yes.
:实例: 2


``technique={reed_sol_van|reed_sol_r6_op|cauchy_orig|cauchy_good|liberation|blaum_roth|liber8tion}``

:描述: *reed_sol_van* 技术更灵活：它足以设置 *k* 和 *m* 值。 *cauchy_good* 技\
       术更快，但你得谨慎地选择 *packetsize* 值。 *reed_sol_r6_op* 、 \
       *liberation* 、  *blaum_roth* 、 *liber8tion* 都是与 *RAID6* 等价的技\
       术，它们只能配置为 *m=2* 。

:类型: String
:是否必需: No.
:默认值: reed_sol_van


``packetsize={bytes}``

:描述: 以 *bytes* 大小的包为单位进行编码。确定合适的包尺寸很难， *jerasure* \
       文档对此有很详细的描述。

:类型: Integer
:是否必需: No.
:默认值: 2048


``ruleset-root={root}``

:描述: 规则集的第一步所用的 crush 桶的名字，例如 **step take default** 。
:类型: String
:是否必需: No.
:默认值: default


``ruleset-failure-domain={bucket-type}``

:描述: 确保两个数据块不会存储到同一故障域中的桶里。例如，假设故障域是 \
       **host** ，那么两个数据块就不会存储到同一主机。它被用于创建规则集阶\
       梯，如 **step chooseleaf host** 。

:类型: String
:是否必需: No.
:默认值: host


``directory={directory}``

:描述: 设置纠删码插件的路径，需是\ **目录**\ 。
:类型: String
:是否必需: No.
:默认值: /usr/lib/ceph/erasure-code


``--force``

:描述: 覆盖同名配置。
:类型: String
:是否必需: No.
