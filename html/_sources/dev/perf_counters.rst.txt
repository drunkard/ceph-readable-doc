.. Perf counters

============
 性能计数器
============

The perf counters provide generic internal infrastructure for gauges and counters.  The counted values can be both integer and float.  There is also an "average" type (normally float) that combines a sum and num counter which can be divided to provide an average.

The intention is that this data will be collected and aggregated by a tool like ``collectd`` or ``statsd`` and fed into a tool like ``graphite`` for graphing and analysis.  Also, note the :doc:`../mgr/prometheus`.


.. Access

如何访问
--------
性能计数器可以通过管理套接字访问，例如： ::

   ceph daemon osd.0 perf schema
   ceph daemon osd.0 perf dump


.. Collections

数据集
------
数值被分组并命名为不同的集，通常表示一个子系统或者一个\
子系统例程。例如，内部的 ``throttle`` 机制会报告它如何节流，\
其各例程的命名类似如下： ::


    throttle-msgr_dispatch_throttler-hbserver
    throttle-msgr_dispatch_throttler-client
    throttle-filestore_bytes
    ...


Schema
------

The ``perf schema`` command dumps a json description of which values are available, and what their type is.  Each named value as a ``type`` bitfield, with the following bits defined.

+------+-------------------------------------+
| bit  | meaning                             |
+======+=====================================+
| 1    | floating point value                |
+------+-------------------------------------+
| 2    | unsigned 64-bit integer value       |
+------+-------------------------------------+
| 4    | average (sum + count pair), where   |
+------+-------------------------------------+
| 8    | counter (vs gauge)                  |
+------+-------------------------------------+

Every value will have either bit 1 or 2 set to indicate the type
(float or integer).

If bit 8 is set (counter), the value is monotonically increasing and
the reader may want to subtract off the previously read value to get
the delta during the previous interval.

If bit 4 is set (average), there will be two values to read, a sum and
a count.  If it is a counter, the average for the previous interval
would be sum delta (since the previous read) divided by the count
delta.  Alternatively, dividing the values outright would provide the
lifetime average value.  Normally these are used to measure latencies
(number of requests and a sum of request latencies), and the average
for the previous interval is what is interesting.

Instead of interpreting the bit fields, the ``metric type`` has a
value of either ``guage`` or ``counter``, and the ``value type``
property will be one of ``real``, ``integer``, ``real-integer-pair``

Here is an example of the schema output::

  {
    "throttle-bluestore_throttle_bytes": {
        "val": {
            "type": 2,
            "metric_type": "gauge",
            "value_type": "integer",
            "description": "Currently available throttle",
            "nick": ""
        },
        "max": {
            "type": 2,
            "metric_type": "gauge",
            "value_type": "integer",
            "description": "Max value for throttle",
            "nick": ""
        },
        "get_started": {
            "type": 10,
            "metric_type": "counter",
            "value_type": "integer",
            "description": "Number of get calls, increased before wait",
            "nick": ""
        },
        "get": {
            "type": 10,
            "metric_type": "counter",
            "value_type": "integer",
            "description": "Gets",
            "nick": ""
        },
        "get_sum": {
            "type": 10,
            "metric_type": "counter",
            "value_type": "integer",
            "description": "Got data",
            "nick": ""
        },
        "get_or_fail_fail": {
            "type": 10,
            "metric_type": "counter",
            "value_type": "integer",
            "description": "Get blocked during get_or_fail",
            "nick": ""
        },
        "get_or_fail_success": {
            "type": 10,
            "metric_type": "counter",
            "value_type": "integer",
            "description": "Successful get during get_or_fail",
            "nick": ""
        },
        "take": {
            "type": 10,
            "metric_type": "counter",
            "value_type": "integer",
            "description": "Takes",
            "nick": ""
        },
        "take_sum": {
            "type": 10,
            "metric_type": "counter",
            "value_type": "integer",
            "description": "Taken data",
            "nick": ""
        },
        "put": {
            "type": 10,
            "metric_type": "counter",
            "value_type": "integer",
            "description": "Puts",
            "nick": ""
        },
        "put_sum": {
            "type": 10,
            "metric_type": "counter",
            "value_type": "integer",
            "description": "Put data",
            "nick": ""
        },
        "wait": {
            "type": 5,
            "metric_type": "gauge",
            "value_type": "real-integer-pair",
            "description": "Waiting latency",
            "nick": ""
        }
  }


Dump
----

The actual dump is similar to the schema, except that average values are grouped.  For example::

 {
   "throttle-msgr_dispatch_throttler-hbserver" : {
      "get_or_fail_fail" : 0,
      "get_sum" : 0,
      "max" : 104857600,
      "put" : 0,
      "val" : 0,
      "take" : 0,
      "get_or_fail_success" : 0,
      "wait" : {
         "avgcount" : 0,
         "sum" : 0
      },
      "get" : 0,
      "take_sum" : 0,
      "put_sum" : 0
   },
   "throttle-msgr_dispatch_throttler-client" : {
      "get_or_fail_fail" : 0,
      "get_sum" : 82760,
      "max" : 104857600,
      "put" : 2637,
      "val" : 0,
      "take" : 0,
      "get_or_fail_success" : 0,
      "wait" : {
         "avgcount" : 0,
         "sum" : 0
      },
      "get" : 2637,
      "take_sum" : 0,
      "put_sum" : 82760
   }
 }

