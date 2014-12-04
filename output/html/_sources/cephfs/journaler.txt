===========
 Journaler
===========

``journaler allow split entries``

:Description: 允许一条目跨越条带边界。
:Type: Boolean
:Required: No
:Default: ``true``


``journaler write head interval``

:Description: 多频繁地更新日志头部对象？
:Type: Integer
:Required: No
:Default: ``15``


``journaler prefetch periods``

:Description: 重放日志时预读多少条带。
:Type: Integer
:Required: No
:Default: ``10``


``journal prezero periods``

:Description: 把写位置前的多少条带清零。
:Type: Integer
:Required: No
:Default: ``10``

``journaler batch interval``

:Description: 人为导致的最大额外延时。
:Type: Double
:Required: No
:Default: ``.001``


``journaler batch max``

:Description: 我们会延迟回写的最大字节数。
:Type: 64-bit Unsigned Integer 
:Required: No
:Default: ``0``
