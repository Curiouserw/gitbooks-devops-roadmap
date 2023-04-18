# kafka性能测试

# 一、简介

性能测试工具是 Kafka 本身提供的用于生产者性能测试` kafka-producer-perf-test.sh `和用于消费者性能测试的`kafka-consumer-perf-test.sh`

` kafka-producer-perf-test.sh `命令详解

```bash
usage: producer-performance \
          [-h] \
          --topic TOPIC \
          --num-records NUM-RECORDS \
          [--payload-delimiter PAYLOAD-DELIMITER] \
          --throughput THROUGHPUT \
          [--producer-props PROP-NAME=PROP-VALUE [PROP-NAME=PROP-VALUE ...]] \
          [--producer.config CONFIG-FILE]
          [--print-metrics] \
          [--transactional-id TRANSACTIONAL-ID] 
          [--transaction-duration-ms TRANSACTION-DURATION] 
          (--record-size RECORD-SIZE | --payload-file PAYLOAD-FILE)

This tool is used to verify the producer performance.

optional arguments:
  -h, --help                 show this help message and exit
  --topic TOPIC              produce messages to this topic
  --num-records NUM-RECORDS  number of messages to produce
  --payload-delimiter PAYLOAD-DELIMITER  provides delimiter to be used when --payload-file is provided. Defaults to new line. Note that this parameter will be ignored if --payload-file is not provided. (default: \n)
  --throughput THROUGHPUT    throttle maximum message throughput to *approximately* THROUGHPUT messages/sec. Set this to -1 to disable throttling.
  --producer-props PROP-NAME=PROP-VALUE [PROP-NAME=PROP-VALUE ...]
                         kafka producer related configuration properties like bootstrap.servers,client.id etc. These configs take precedence over those passed via --producer.config.
  --producer.config CONFIG-FILE    producer config properties file.
  --print-metrics        print out metrics at the end of the test. (default: false)
  --transactional-id TRANSACTIONAL-ID
                         The transactionalId to use if transaction-duration-ms is > 0. Useful when testing the performance of concurrent transactions. (default: performance-producer-default-transactional-id)
  --transaction-duration-ms TRANSACTION-DURATION   The max age of each transaction. The commitTransaction will be called after this time has elapsed. Transactions are only enabled if this value is positive. (default: 0) either --record-size or --payload-file must be specified but not both.
  --record-size RECORD-SIZE    message size in bytes. Note that you must provide exactly one of --record-size or --payload-file.
  --payload-file PAYLOAD-FILE  file to read the message payloads from. This works only for UTF-8 encoded text files. Payloads will be read from  this file and a payload will be randomly selected when sending messages. Note that you must provide exactly one                         of --record-size or --payload-file.
```

`kafka-consumer-perf-test.sh`命令详解

```bash
This tool helps in performance test for the full zookeeper consumer
Option                                   Description
------                                   -----------
--bootstrap-server <String: server to    REQUIRED unless --broker-list
  connect to>                              (deprecated) is specified. The server
                                           (s) to connect to.
--broker-list <String: broker-list>      DEPRECATED, use --bootstrap-server
                                           instead; ignored if --bootstrap-
                                           server is specified.  The broker
                                           list string in the form HOST1:PORT1,
                                           HOST2:PORT2.
--consumer.config <String: config file>  Consumer config properties file.
--date-format <String: date format>      The date format to use for formatting
                                           the time field. See java.text.
                                           SimpleDateFormat for options.
                                           (default: yyyy-MM-dd HH:mm:ss:SSS)
--fetch-size <Integer: size>             The amount of data to fetch in a
                                           single request. (default: 1048576)
--from-latest                            If the consumer does not already have
                                           an established offset to consume
                                           from, start with the latest message
                                           present in the log rather than the
                                           earliest message.
--group <String: gid>                    The group id to consume on. (default:
                                           perf-consumer-50295)
--help                                   Print usage information.
--hide-header                            If set, skips printing the header for
                                           the stats
--messages <Long: count>                 REQUIRED: The number of messages to
                                           send or consume
--num-fetch-threads <Integer: count>     Number of fetcher threads. (default: 1)
--print-metrics                          Print out the metrics.
--reporting-interval <Integer:           Interval in milliseconds at which to
  interval_ms>                             print progress info. (default: 5000)
--show-detailed-stats                    If set, stats are reported for each
                                           reporting interval as configured by
                                           reporting-interval
--socket-buffer-size <Integer: size>     The size of the tcp RECV size.
                                           (default: 2097152)
--threads <Integer: count>               Number of processing threads.
                                           (default: 10)
--timeout [Long: milliseconds]           The maximum allowed time in
                                           milliseconds between returned
                                           records. (default: 10000)
--topic <String: topic>                  REQUIRED: The topic to consume from.
--version                                Display Kafka version.
```



# 二、生产者写入测试

向一个只有1个分区和1个副本的Topic主题`perf-producer-test` 中发送 100 万条消息，并且每条消息大小为 1024B 生产者，对应的 acks 参数为1

```bash
kafka-producer-perf-test.sh \
  --topic perf-producer-test \
  --num-records 1000000 \
  --record-size 1024 \
  --throughput -1 \
  --print-metrics \
  --producer-props bootstrap.servers=localhost:9092 acks=1
  
# --nurn-records：用来指定发送消息的总条数 
# --record-size：用来设置每条消息的字节数
# --throughput：用来进行限流控制，当设定的值小于0不限流，当设定的值大于0时，当发送的吞吐量大于该值时就会被阻塞一段时间。
# --print-metrics：测试完成后的Metrics信息
# --producer-props：参数用来指定生产者的配置，可同时指定多组配置，各组配置之间以空格分隔，与producer-props 参数对应的还有一个producer-config参数，它用来指定生产者的配置文件
```

测试结果

```bash
67164 records sent, 13430.1 records/sec (13.12 MB/sec), 1461.5 ms avg latency, 2014.0 ms max latency.
92910 records sent, 18430.9 records/sec (18.00 MB/sec), 1718.4 ms avg latency, 2038.0 ms max latency.
120060 records sent, 23741.3 records/sec (23.18 MB/sec), 1304.2 ms avg latency, 1688.0 ms max latency.
110055 records sent, 21853.7 records/sec (21.34 MB/sec), 1428.9 ms avg latency, 1660.0 ms max latency.
119745 records sent, 23949.0 records/sec (23.39 MB/sec), 1281.9 ms avg latency, 1692.0 ms max latency.
110910 records sent, 22182.0 records/sec (21.66 MB/sec), 1383.2 ms avg latency, 1516.0 ms max latency.
135255 records sent, 27051.0 records/sec (26.42 MB/sec), 1191.1 ms avg latency, 1580.0 ms max latency.
122490 records sent, 24498.0 records/sec (23.92 MB/sec), 1214.0 ms avg latency, 1527.0 ms max latency.

1000000 records sent, 22281.639929 records/sec (21.76 MB/sec), 1332.82 ms avg latency, 2038.00 ms max latency, 1357 ms 50th, 1822 ms 95th, 1955 ms 99th, 2010 ms 99.9th.

#  records sent：表示测试时发送的消息总数
#  records sec：表示以每秒发迭的消息数来统计吞吐量，括号中 MB/sec 表示以每秒发送的消息大小来统计吞吐量，
#  avg latency: 表示消息处理的平均耗时
#  max latency：表示消息处理的最大耗时 
#  50th,95th,99t,99.9th： 分别表示 50% 95 99% 99.9% 的消息处理耗时。

# 以下为测试完成后的Metrics信息
Metric Name                                                                                   Value
app-info:commit-id:{client-id=producer-1}                                                   : 66563e712b0b9f84
app-info:start-time-ms:{client-id=producer-1}                                               : 1608628521735
app-info:version:{client-id=producer-1}                                                     : 2.5.0
kafka-metrics-count:count:{client-id=producer-1}                                            : 102.000
producer-metrics:batch-size-avg:{client-id=producer-1}                                      : 15555.699
producer-metrics:batch-size-max:{client-id=producer-1}                                      : 15570.000
producer-metrics:batch-split-rate:{client-id=producer-1}                                    : 0.000
producer-metrics:batch-split-total:{client-id=producer-1}                                   : 0.000
producer-metrics:buffer-available-bytes:{client-id=producer-1}                              : 33554432.000
producer-metrics:buffer-exhausted-rate:{client-id=producer-1}                               : 0.000
producer-metrics:buffer-exhausted-total:{client-id=producer-1}                              : 0.000
producer-metrics:buffer-total-bytes:{client-id=producer-1}                                  : 33554432.000
producer-metrics:bufferpool-wait-ratio:{client-id=producer-1}                               : 0.861
producer-metrics:bufferpool-wait-time-total:{client-id=producer-1}                          : 37854962935.000
producer-metrics:compression-rate-avg:{client-id=producer-1}                                : 1.000
producer-metrics:connection-close-rate:{client-id=producer-1}                               : 0.000
producer-metrics:connection-close-total:{client-id=producer-1}                              : 0.000
producer-metrics:connection-count:{client-id=producer-1}                                    : 2.000
producer-metrics:connection-creation-rate:{client-id=producer-1}                            : 0.044
producer-metrics:connection-creation-total:{client-id=producer-1}                           : 2.000
producer-metrics:failed-authentication-rate:{client-id=producer-1}                          : 0.000
producer-metrics:failed-authentication-total:{client-id=producer-1}                         : 0.000
producer-metrics:failed-reauthentication-rate:{client-id=producer-1}                        : 0.000
producer-metrics:failed-reauthentication-total:{client-id=producer-1}                       : 0.000
producer-metrics:incoming-byte-rate:{client-id=producer-1}                                  : 114570.421
producer-metrics:incoming-byte-total:{client-id=producer-1}                                 : 5134588.000
producer-metrics:io-ratio:{client-id=producer-1}                                            : 0.224
producer-metrics:io-time-ns-avg:{client-id=producer-1}                                      : 70608.839
producer-metrics:io-wait-ratio:{client-id=producer-1}                                       : 0.561
producer-metrics:io-wait-time-ns-avg:{client-id=producer-1}                                 : 177047.069
producer-metrics:io-waittime-total:{client-id=producer-1}                                   : 25274885534.000
producer-metrics:iotime-total:{client-id=producer-1}                                        : 10079976609.000
producer-metrics:metadata-age:{client-id=producer-1}                                        : 44.621
producer-metrics:network-io-rate:{client-id=producer-1}                                     : 2975.167
producer-metrics:network-io-total:{client-id=producer-1}                                    : 133344.000
producer-metrics:outgoing-byte-rate:{client-id=producer-1}                                  : 23246321.000
producer-metrics:outgoing-byte-total:{client-id=producer-1}                                 : 1041667644.000
producer-metrics:produce-throttle-time-avg:{client-id=producer-1}                           : 0.000
producer-metrics:produce-throttle-time-max:{client-id=producer-1}                           : 0.000
producer-metrics:reauthentication-latency-avg:{client-id=producer-1}                        : NaN
producer-metrics:reauthentication-latency-max:{client-id=producer-1}                        : NaN
producer-metrics:record-error-rate:{client-id=producer-1}                                   : 0.000
producer-metrics:record-error-total:{client-id=producer-1}                                  : 0.000
producer-metrics:record-queue-time-avg:{client-id=producer-1}                               : 1329.508
producer-metrics:record-queue-time-max:{client-id=producer-1}                               : 2034.000
producer-metrics:record-retry-rate:{client-id=producer-1}                                   : 0.000
producer-metrics:record-retry-total:{client-id=producer-1}                                  : 0.000
producer-metrics:record-send-rate:{client-id=producer-1}                                    : 22429.068
producer-metrics:record-send-total:{client-id=producer-1}                                   : 1000000.000
producer-metrics:record-size-avg:{client-id=producer-1}                                     : 1110.000
producer-metrics:record-size-max:{client-id=producer-1}                                     : 1110.000
producer-metrics:records-per-request-avg:{client-id=producer-1}                             : 15.000
producer-metrics:request-latency-avg:{client-id=producer-1}                                 : 3.289
producer-metrics:request-latency-max:{client-id=producer-1}                                 : 305.000
producer-metrics:request-rate:{client-id=producer-1}                                        : 1487.617
producer-metrics:request-size-avg:{client-id=producer-1}                                    : 15623.765
producer-metrics:request-size-max:{client-id=producer-1}                                    : 15639.000
producer-metrics:request-total:{client-id=producer-1}                                       : 66672.000
producer-metrics:requests-in-flight:{client-id=producer-1}                                  : 0.000
producer-metrics:response-rate:{client-id=producer-1}                                       : 1487.650
producer-metrics:response-total:{client-id=producer-1}                                      : 66672.000
producer-metrics:select-rate:{client-id=producer-1}                                         : 3169.020
producer-metrics:select-total:{client-id=producer-1}                                        : 142758.000
producer-metrics:successful-authentication-no-reauth-total:{client-id=producer-1}           : 0.000
producer-metrics:successful-authentication-rate:{client-id=producer-1}                      : 0.000
producer-metrics:successful-authentication-total:{client-id=producer-1}                     : 0.000
producer-metrics:successful-reauthentication-rate:{client-id=producer-1}                    : 0.000
producer-metrics:successful-reauthentication-total:{client-id=producer-1}                   : 0.000
producer-metrics:waiting-threads:{client-id=producer-1}                                     : 0.000
producer-node-metrics:incoming-byte-rate:{client-id=producer-1, node-id=node--1}            : 12.562
producer-node-metrics:incoming-byte-rate:{client-id=producer-1, node-id=node-2}             : 115053.336
producer-node-metrics:incoming-byte-total:{client-id=producer-1, node-id=node--1}           : 563.000
producer-node-metrics:incoming-byte-total:{client-id=producer-1, node-id=node-2}            : 5134025.000
producer-node-metrics:outgoing-byte-rate:{client-id=producer-1, node-id=node--1}            : 2.254
producer-node-metrics:outgoing-byte-rate:{client-id=producer-1, node-id=node-2}             : 23345828.974
producer-node-metrics:outgoing-byte-total:{client-id=producer-1, node-id=node--1}           : 101.000
producer-node-metrics:outgoing-byte-total:{client-id=producer-1, node-id=node-2}            : 1041667543.000
producer-node-metrics:request-latency-avg:{client-id=producer-1, node-id=node--1}           : NaN
producer-node-metrics:request-latency-avg:{client-id=producer-1, node-id=node-2}            : 3.289
producer-node-metrics:request-latency-max:{client-id=producer-1, node-id=node--1}           : NaN
producer-node-metrics:request-latency-max:{client-id=producer-1, node-id=node-2}            : 305.000
producer-node-metrics:request-rate:{client-id=producer-1, node-id=node--1}                  : 0.045
producer-node-metrics:request-rate:{client-id=producer-1, node-id=node-2}                   : 1494.039
producer-node-metrics:request-size-avg:{client-id=producer-1, node-id=node--1}              : 50.500
producer-node-metrics:request-size-avg:{client-id=producer-1, node-id=node-2}               : 15624.232
producer-node-metrics:request-size-max:{client-id=producer-1, node-id=node--1}              : 51.000
producer-node-metrics:request-size-max:{client-id=producer-1, node-id=node-2}               : 15639.000
producer-node-metrics:request-total:{client-id=producer-1, node-id=node--1}                 : 2.000
producer-node-metrics:request-total:{client-id=producer-1, node-id=node-2}                  : 66670.000
producer-node-metrics:response-rate:{client-id=producer-1, node-id=node--1}                 : 0.045
producer-node-metrics:response-rate:{client-id=producer-1, node-id=node-2}                  : 1494.073
producer-node-metrics:response-total:{client-id=producer-1, node-id=node--1}                : 2.000
producer-node-metrics:response-total:{client-id=producer-1, node-id=node-2}                 : 66670.000
producer-topic-metrics:byte-rate:{client-id=producer-1, topic=perf-producer-test2}          : 23259932.490
producer-topic-metrics:byte-total:{client-id=producer-1, topic=perf-producer-test2}         : 1037067350.000
producer-topic-metrics:compression-rate:{client-id=producer-1, topic=perf-producer-test2}   : 1.000
producer-topic-metrics:record-error-rate:{client-id=producer-1, topic=perf-producer-test2}  : 0.000
producer-topic-metrics:record-error-total:{client-id=producer-1, topic=perf-producer-test2} : 0.000
producer-topic-metrics:record-retry-rate:{client-id=producer-1, topic=perf-producer-test2}  : 0.000
producer-topic-metrics:record-retry-total:{client-id=producer-1, topic=perf-producer-test2} : 0.000
producer-topic-metrics:record-send-rate:{client-id=producer-1, topic=perf-producer-test2}   : 22428.565
producer-topic-metrics:record-send-total:{client-id=producer-1, topic=perf-producer-test2}  : 1000000.000
```

# 三、消费者性能测试

消费主题`perf-producer-test`中的 100 万条消息

```bash
kafka-consumer-perf-test.sh \
  --topic perf-producer-test \
  --messages 1000000 \
  --print-metrics \
  --broker-list localhost:9092
```

测试结果

```bash
| start.time | end.time | data.consumed.in.MB | MB.sec | data.consumed.in.nMsg | nMsg.sec | rebalance.time.ms | fetch.time.ms | fetch.MB.sec | fetch.nMsg.sec
==================================================================================================
| 2020-12-22 09:28:31:384 | 2020-12-22 09:28:47:769 | 976.5625 | 59.6010 |  1000000 | 61031.4312 | 1608629311854 |  -1608629295469 | -0.0000 | -0.0006

# start.time: 起始运行时间 
# end.time: 结束运行时
# data.consumed.in.MB: 消息总量（ 单位为 MB ）
# MB.sec: 按字节大小计算的消费吞吐量（ MB单位为 MB ）、
# data.consumed.in.nMsg :消费的消息总数
# nMsg.sec: 按消息个数计算的吞吐量
# rebalance.time.ms：再平衡的时间,单位为ms 
# fetch.time.ms：拉取消息的持续时间，单位为ms 
# fetch.MB.sec: 每秒拉取消息的字节大小，单位MB 
# fetch.nMsg.sec: 每秒拉取消息的个数 ,其中 fetch.time. s= end.time - start.time - rebalance.time.ms a


Metric Name                                                                                  Value
consumer-coordinator-metrics:assigned-partitions:{client-id=consumer-perf-consumer-89119-1} : 0.000
consumer-coordinator-metrics:commit-latency-avg:{client-id=consumer-perf-consumer-89119-1} : 11.750
consumer-coordinator-metrics:commit-latency-max:{client-id=consumer-perf-consumer-89119-1} : 30.000
consumer-coordinator-metrics:commit-rate:{client-id=consumer-perf-consumer-89119-1} : 0.097
consumer-coordinator-metrics:commit-total:{client-id=consumer-perf-consumer-89119-1} : 4.000
consumer-coordinator-metrics:failed-rebalance-rate-per-hour:{client-id=consumer-perf-consumer-89119-1} : 77.983
consumer-coordinator-metrics:failed-rebalance-total:{client-id=consumer-perf-consumer-89119-1} : 1.000
consumer-coordinator-metrics:heartbeat-rate:{client-id=consumer-perf-consumer-89119-1} : 0.116
consumer-coordinator-metrics:heartbeat-response-time-max:{client-id=consumer-perf-consumer-89119-1} : 9.000
consumer-coordinator-metrics:heartbeat-total:{client-id=consumer-perf-consumer-89119-1} : 5.000
consumer-coordinator-metrics:join-rate:{client-id=consumer-perf-consumer-89119-1} : 0.022
consumer-coordinator-metrics:join-time-avg:{client-id=consumer-perf-consumer-89119-1} : 32.000
consumer-coordinator-metrics:join-time-max:{client-id=consumer-perf-consumer-89119-1} : 32.000
consumer-coordinator-metrics:join-total:{client-id=consumer-perf-consumer-89119-1} : 1.000
consumer-coordinator-metrics:last-heartbeat-seconds-ago:{client-id=consumer-perf-consumer-89119-1} : 1.000
consumer-coordinator-metrics:last-rebalance-seconds-ago:{client-id=consumer-perf-consumer-89119-1} : 16.000
consumer-coordinator-metrics:partition-assigned-latency-avg:{client-id=consumer-perf-consumer-89119-1} : 0.000
consumer-coordinator-metrics:partition-assigned-latency-max:{client-id=consumer-perf-consumer-89119-1} : 0.000
consumer-coordinator-metrics:partition-lost-latency-avg:{client-id=consumer-perf-consumer-89119-1} : NaN
consumer-coordinator-metrics:partition-lost-latency-max:{client-id=consumer-perf-consumer-89119-1} : NaN
consumer-coordinator-metrics:partition-revoked-latency-avg:{client-id=consumer-perf-consumer-89119-1} : 0.000
consumer-coordinator-metrics:partition-revoked-latency-max:{client-id=consumer-perf-consumer-89119-1} : 0.000
consumer-coordinator-metrics:rebalance-latency-avg:{client-id=consumer-perf-consumer-89119-1} : 101.000
consumer-coordinator-metrics:rebalance-latency-max:{client-id=consumer-perf-consumer-89119-1} : 101.000
consumer-coordinator-metrics:rebalance-latency-total:{client-id=consumer-perf-consumer-89119-1} : 101.000
consumer-coordinator-metrics:rebalance-rate-per-hour:{client-id=consumer-perf-consumer-89119-1} : 78.086
consumer-coordinator-metrics:rebalance-total:{client-id=consumer-perf-consumer-89119-1} : 1.000
consumer-coordinator-metrics:sync-rate:{client-id=consumer-perf-consumer-89119-1} : 0.022
consumer-coordinator-metrics:sync-time-avg:{client-id=consumer-perf-consumer-89119-1} : 24.000
consumer-coordinator-metrics:sync-time-max:{client-id=consumer-perf-consumer-89119-1} : 24.000
consumer-coordinator-metrics:sync-total:{client-id=consumer-perf-consumer-89119-1} : 1.000
consumer-fetch-manager-metrics:bytes-consumed-rate:{client-id=consumer-perf-consumer-89119-1, topic=perf-producer-test2} : 22454094.6
consumer-fetch-manager-metrics:bytes-consumed-rate:{client-id=consumer-perf-consumer-89119-1} : 22453606.095
consumer-fetch-manager-metrics:bytes-consumed-total:{client-id=consumer-perf-consumer-89119-1, topic=perf-producer-test2}：100602.000
consumer-fetch-manager-metrics:bytes-consumed-total:{client-id=consumer-perf-consumer-89119-1} : 1033000602.000
consumer-fetch-manager-metrics:fetch-latency-avg:{client-id=consumer-perf-consumer-89119-1} : 13.353
consumer-fetch-manager-metrics:fetch-latency-max:{client-id=consumer-perf-consumer-89119-1} : 288.000
consumer-fetch-manager-metrics:fetch-rate:{client-id=consumer-perf-consumer-89119-1} : 21.633
consumer-fetch-manager-metrics:fetch-size-avg:{client-id=consumer-perf-consumer-89119-1, topic=perf-producer-test2} : 1037149.199
consumer-fetch-manager-metrics:fetch-size-avg:{client-id=consumer-perf-consumer-89119-1} : 1037149.199
consumer-fetch-manager-metrics:fetch-size-max:{client-id=consumer-perf-consumer-89119-1, topic=perf-producer-test2} : 1038179.000
consumer-fetch-manager-metrics:fetch-size-max:{client-id=consumer-perf-consumer-89119-1} : 1038179.000
consumer-fetch-manager-metrics:fetch-throttle-time-avg:{client-id=consumer-perf-consumer-89119-1} : 0.000
consumer-fetch-manager-metrics:fetch-throttle-time-max:{client-id=consumer-perf-consumer-89119-1} : 0.000
consumer-fetch-manager-metrics:fetch-total:{client-id=consumer-perf-consumer-89119-1} : 996.000
```

