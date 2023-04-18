# 一、方案

使用官方压测工具pulsar-perf ，利用多线程模拟生产者和消费者在并发情况下发送或消费数据的情形，以测试pulsar的读写性能

详细文档参考：https://pulsar.apache.org/docs/en/performance-pulsar-perf/

命令参数：https://pulsar.apache.org/docs/en/reference-cli-tools/#pulsar-perf

# 二、测试指标

- 作为生产者并发写入不同类型Topic的TPS
- 作为消费者并发读取消息的QPS

# 三、测试环境

基础

- 三台8核，16G内存，100G SSD系统盘(4800IOPS)、1T 高效数据盘(5000 IOPS)的阿里云ECS服务器
- 操作系统为Ubuntu 20.04，均安装JDK 11.0.10
- 数据盘以LVM挂载/data路径下
- IP地址：192.168.170.121~123

Pulsar

- Pulsar版本：2.7.1
- 使用内置Zookeeper
- Apache Zookeeper、Apache Bookkeeper 、Pulsar Broker均以集群形式分布在三台集群上。
- Apache Zookeeper、Apache Bookkeeper 、Pulsar Broker的JVM -Xms -Xmx均设置为8Gb

测试命令

- 在192.168.170.121上执行测试命令



# 四、测试命令及结果

## 1、并发写入持久化Topic

```
pulsar-perf produce \
  --batch-max-messages 10000 \              # 设置批量发送批次的最大消息数
  --num-producers 5 \                                   # 设置每个主题的生产者数量
  --num-test-threads 5 \                            # 设置进程个数
  --rate 5000 \                                             # 设置发送消息的速率                     
  --size 4096 \                                             # 设置发送单个消息的大小，单位bytes字节
  persistent://public/default/perf-test11
  
 其他默认参数
   单个pulsar broker建立的最大TCP连接数：100
   发送的消息数量：                                         0（一直发,不限制.不停不中断）
   预热时间：                                                    1s
   主题个数：                                                    1个
   批量发送数据的窗口时间:                             1ms
   单次批量发送数据大小:                              4194304 bytes
```

```
Throughput produced:   4073.5  msg/s ---    127.3 Mbit/s --- failure      0.0 msg/s --- Latency: mean: 472.911 ms - med: 103.862 - 95pct: 1210.079 - 99pct: 1223.351 - 99.9pct: 1236.391 - 99.99pct: 1244.527 - Max: 1249.999
Throughput produced:   5000.5  msg/s ---    156.3 Mbit/s --- failure      0.0 msg/s --- Latency: mean: 227.782 ms - med:   7.242 - 95pct: 1557.415 - 99pct: 1883.111 - 99.9pct: 1968.999 - 99.99pct: 1977.207 - Max: 1978.335
Throughput produced:   5000.7  msg/s ---    156.3 Mbit/s --- failure      0.0 msg/s --- Latency: mean: 363.717 ms - med:   8.834 - 95pct: 1605.079 - 99pct: 1796.095 - 99.9pct: 1856.335 - 99.99pct: 1864.623 - Max: 1865.927
Throughput produced:   4296.8  msg/s ---    134.3 Mbit/s --- failure      0.0 msg/s --- Latency: mean: 151.693 ms - med:   7.453 - 95pct: 1121.647 - 99pct: 1417.671 - 99.9pct: 1486.287 - 99.99pct: 1492.615 - Max: 1493.135
Throughput produced:   5704.3  msg/s ---    178.3 Mbit/s --- failure      0.0 msg/s --- Latency: mean: 249.943 ms - med:   8.440 - 95pct: 1692.911 - 99pct: 1980.311 - 99.9pct: 2052.695 - 99.99pct: 2068.639 - Max: 2073.943
```

![2021-04-20-15-19-25.png](https://cdn.nlark.com/yuque/0/2021/png/782346/1618905766081-1b827d20-a70a-418a-910c-ef6ae0ead326.png?x-oss-process=image%2Fresize%2Cw_1500)



```
pulsar-perf produce \
  --batch-max-messages 10000 \              # 设置批量发送批次的最大消息数
  --num-producers 5 \                                   # 设置每个主题的生产者数量
  --num-test-threads 5 \                            # 设置进程个数
  --rate 8000 \                                             # 设置发送消息的速率                     
  --size 4096 \                                             # 设置发送单个消息的大小，单位bytes字节
  persistent://public/default/perf-test
  
 其他默认参数
   单个pulsar broker建立的最大TCP连接数：100
   发送的消息数量：                                         0（一直发,不限制.不停不中断）
   预热时间：                                                    1s
   主题个数：                                                    1个
   批量发送数据的窗口时间:                             1ms
   单次批量发送数据大小:                              4194304 bytes
```

```
Throughput produced:   6552.2  msg/s ---    204.8 Mbit/s --- failure      0.0 msg/s --- Latency: mean:   8.609 ms - med:   6.987 - 95pct:  16.778 - 99pct:  37.272 - 99.9pct:  68.692 - 99.99pct:  72.774 - Max:  73.817
Throughput produced:   8772.0  msg/s ---    274.1 Mbit/s --- failure      0.0 msg/s --- Latency: mean: 596.719 ms - med: 273.773 - 95pct: 1966.231 - 99pct: 2160.895 - 99.9pct: 2225.311 - 99.99pct: 2235.519 - Max: 2238.799
Throughput produced:   7993.3  msg/s ---    249.8 Mbit/s --- failure      0.0 msg/s --- Latency: mean: 537.369 ms - med: 168.972 - 95pct: 1815.615 - 99pct: 2078.911 - 99.9pct: 2164.095 - 99.99pct: 2177.231 - Max: 2179.247
Throughput produced:   6447.4  msg/s ---    201.5 Mbit/s --- failure      0.0 msg/s --- Latency: mean: 600.288 ms - med: 336.561 - 95pct: 1775.967 - 99pct: 1957.775 - 99.9pct: 2022.439 - 99.99pct: 2033.391 - Max: 2034.239
Throughput produced:   9575.8  msg/s ---    299.2 Mbit/s --- failure      0.0 msg/s --- Latency: mean: 728.759 ms - med: 482.111 - 95pct: 2022.175 - 99pct: 2234.175 - 99.9pct: 2332.463 - 99.99pct: 2341.391 - Max: 2345.439
Throughput produced:   6827.5  msg/s ---    213.4 Mbit/s --- failure      0.0 msg/s --- Latency: mean: 756.798 ms - med: 463.187 - 95pct: 2216.655 - 99pct: 2511.359 - 99.9pct: 2572.815 - 99.99pct: 2586.511 - Max: 2590.655
```

![2021-04-20-15-31-37.png](https://cdn.nlark.com/yuque/0/2021/png/782346/1618905809375-02fb332c-acba-41bb-a586-9ab20bf9693c.png?x-oss-process=image%2Fresize%2Cw_1500)

## 2、并发写入非持久化Topic

```
pulsar-perf produce \
  --batch-max-messages 10000 \              # 设置批量发送批次的最大消息数
  --num-producers 5 \                                   # 设置每个主题的生产者数量
  --num-test-threads 5 \                            # 设置进程个数
  --rate 5000 \                                             # 设置发送消息的速率                     
  --size 4096 \                                             # 设置发送单个消息的大小，单位bytes字节
  non-persistent://public/default/perf-test6
  
 其他默认参数
   单个pulsar broker建立的最大TCP连接数：100
   发送的消息数量：                                         0（一直发,不限制.不停不中断）
   预热时间：                                                    1s
   主题个数：                                                    1个
   批量发送数据的窗口时间:                             1ms
   单次批量发送数据大小:                              4194304 bytes
```

```
Throughput produced:   4562.4  msg/s ---    142.6 Mbit/s --- failure      0.0 msg/s --- Latency: mean:   0.718 ms - med:   0.718 - 95pct:   1.162 - 99pct:   1.228 - 99.9pct:   2.022 - 99.99pct:   8.470 - Max:  11.320
Throughput produced:   4999.8  msg/s ---    156.2 Mbit/s --- failure      0.0 msg/s --- Latency: mean:   0.709 ms - med:   0.702 - 95pct:   1.149 - 99pct:   1.198 - 99.9pct:   3.810 - 99.99pct:  12.648 - Max:  23.229
Throughput produced:   5000.2  msg/s ---    156.3 Mbit/s --- failure      0.0 msg/s --- Latency: mean:   0.701 ms - med:   0.701 - 95pct:   1.145 - 99pct:   1.193 - 99.9pct:   1.802 - 99.99pct:   7.321 - Max:  15.233
Throughput produced:   4999.9  msg/s ---    156.2 Mbit/s --- failure      0.0 msg/s --- Latency: mean:   0.706 ms - med:   0.705 - 95pct:   1.151 - 99pct:   1.203 - 99.9pct:   2.960 - 99.99pct:   5.138 - Max:   5.723
Throughput produced:   5000.1  msg/s ---    156.3 Mbit/s --- failure      0.0 msg/s --- Latency: mean:   0.703 ms - med:   0.701 - 95pct:   1.148 - 99pct:   1.194 - 99.9pct:   3.385 - 99.99pct:   5.997 - Max:   6.320
Throughput produced:   4999.9  msg/s ---    156.2 Mbit/s --- failure      0.0 msg/s --- Latency: mean:   0.697 ms - med:   0.697 - 95pct:   1.147 - 99pct:   1.194 - 99.9pct:   1.298 - 99.99pct:   2.889 - Max:   4.888
```

![2021-04-20-15-28-14.png](https://cdn.nlark.com/yuque/0/2021/png/782346/1618905783857-2f0e68be-7c2e-47fe-a9c4-920f59f5132c.png?x-oss-process=image%2Fresize%2Cw_1500)



```
pulsar-perf produce \
  --batch-max-messages 10000 \              # 设置批量发送批次的最大消息数
  --num-producers 5 \                                   # 设置每个主题的生产者数量
  --num-test-threads 5 \                            # 设置进程个数
  --rate 8000 \                                             # 设置发送消息的速率                     
  --size 4096 \                                             # 设置发送单个消息的大小，单位bytes字节
  non-persistent://public/default/perf-test8
  
 其他默认参数
   单个pulsar broker建立的最大TCP连接数：100
   发送的消息数量：                                         0（一直发,不限制.不停不中断）
   预热时间：                                                    1s
   主题个数：                                                    1个
   批量发送数据的窗口时间:                             1ms
   单次批量发送数据大小:                              4194304 bytes
```



```
Throughput produced:   7329.1  msg/s ---    229.0 Mbit/s --- failure      0.0 msg/s --- Latency: mean:   0.710 ms - med:   0.703 - 95pct:   1.159 - 99pct:   1.230 - 99.9pct:   3.044 - 99.99pct:  10.701 - Max:  13.688
Throughput produced:   8000.3  msg/s ---    250.0 Mbit/s --- failure      0.0 msg/s --- Latency: mean:   0.699 ms - med:   0.694 - 95pct:   1.149 - 99pct:   1.199 - 99.9pct:   2.149 - 99.99pct:   7.919 - Max:  11.846
Throughput produced:   8000.8  msg/s ---    250.0 Mbit/s --- failure      0.0 msg/s --- Latency: mean:   0.697 ms - med:   0.693 - 95pct:   1.146 - 99pct:   1.198 - 99.9pct:   2.326 - 99.99pct:   5.694 - Max:   6.710
Throughput produced:   8000.9  msg/s ---    250.0 Mbit/s --- failure      0.0 msg/s --- Latency: mean:   0.741 ms - med:   0.700 - 95pct:   1.156 - 99pct:   1.228 - 99.9pct:  12.492 - 99.99pct:  22.429 - Max:  33.309
Throughput produced:   8000.7  msg/s ---    250.0 Mbit/s --- failure      0.0 msg/s --- Latency: mean:   0.696 ms - med:   0.693 - 95pct:   1.146 - 99pct:   1.196 - 99.9pct:   1.295 - 99.99pct:   5.035 - Max:   5.478
Throughput produced:   8000.7  msg/s ---    250.0 Mbit/s --- failure      0.0 msg/s --- Latency: mean:   0.696 ms - med:   0.693 - 95pct:   1.146 - 99pct:   1.195 - 99.9pct:   1.279 - 99.99pct:   5.308 - Max:   8.603
```

![2021-04-20-15-36-32.png](https://cdn.nlark.com/yuque/0/2021/png/782346/1618905822182-4835dddf-500d-4e83-9b8f-f4c77b45328f.png?x-oss-process=image%2Fresize%2Cw_1500)