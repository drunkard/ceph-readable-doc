===========
 Journaler
===========

``journaler allow split entries``

:描述: 允许一条目跨越条带边界。
:类型: Boolean
:是否必需: No
:默认值: ``true``


``journaler write head interval``

:描述: 多频繁地更新日志头部对象？
:类型: Integer
:是否必需: No
:默认值: ``15``


``journaler prefetch periods``

:描述: 重放日志时预读多少条带。
:类型: Integer
:是否必需: No
:默认值: ``10``


``journal prezero periods``

:描述: 把写位置前的多少条带清零。
:类型: Integer
:是否必需: No
:默认值: ``10``


``journaler batch interval``

:描述: 人为导致的最大额外延时。
:类型: Double
:是否必需: No
:默认值: ``.001``


``journaler batch max``

:描述: 我们会延迟回写的最大字节数。
:类型: 64-bit Unsigned Integer
:是否必需: No
:默认值: ``0``
