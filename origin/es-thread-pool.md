# ElasticSearch的进程池

# 一、简介

每个Elasticsearch节点内部都维护着多个线程池，每一类型的操作都被分配于不同的线程池



# 二、线程池类型

- **fixed**

有着固定个数的线程池，个数由size参数指定。允许你指定一个队列（大小使用queue_size属性指定，默认是-1，即无限制）用来保存请求，直到有一个空闲的线程来执行请求。如果Elasticsearch无法把请求放到队列中（队列满了），该请求将被拒绝。

```bash
threadpool.write.size: 8
threadpool.write.queue_size: 100
```

- **scaling**

动态个数的线程池，个数与工作负载成比例，在core参数与max参数之间浮动，同样keep_alive参数指定了闲置线程被回收的时间。

```bash
threadpool.warmer.core: 1
threadpool.warmer.max: 8
threadpool.warmer.keep_alive: 2m
```

- **fixed_auto_queue_size**

>  此功能是试验性的，在将来的版本中可能会完全更改或删除。 Elastic会尽力解决所有问题，但是实验性功能不受官方GA功能的支持SLA约束。不推荐使用[7.7.0，不推荐使用实验性fixed_auto_queue_size线程池类型，该类型将在8.0中删除。

固定个数但大小浮动的线程池，个数由size参数指定，大小在`min_queue_size`与`max_queue_size`之间浮动

```bash
thread_pool.search.queue_size: 500
#queue_size允许控制没有线程执行它们的挂起请求队列的初始大小。
thread_pool.search.size: 200
#size参数控制线程数，默认为核心数乘以5。
thread_pool.search.min_queue_size:10
#min_queue_size设置控制queue_size可以调整到的最小量。
thread_pool.search.max_queue_size: 1000
#max_queue_size设置控制queue_size可以调整到的最大量。
thread_pool.search.auto_queue_frame_size: 2000
#auto_queue_frame_size设置控制在调整队列之前进行测量的操作数。它应该足够大，以便单个操作不会过度偏向计算。
thread_pool.search.target_response_time: 6s
#target_response_time是时间值设置，指示线程池队列中任务的目标平均响应时间。如果任务通常超过此时间，则将调低线程池队列以拒绝任务。
```

# 三、核心线程池

| 线程                    | 类型                  | 作用                                                       | 默认配置                                                     |
| ----------------------- | --------------------- | ---------------------------------------------------------- | ------------------------------------------------------------ |
| **generic**             | scaling               | 用于一些通用操作，如node discovery                         |                                                              |
| **search**              | fixed_auto_queue_size | 索引的`count/search/sugges`操作                            | size = `int((可用cpu核心数*3)/2)+ 1`<br/>queue = 1000        |
| **search_throttled**    | fixed_auto_queue_size | `search_throttled`类型索引的`count/search/suggest/get`操作 | size =1<br/>queue = 100                                      |
| **write**               | fixed                 | 对单个文档的`index/delete/update`操作和`bulk`批量插入操作  | size = 可用cpu核心数(最多多1个)<br>queue = 1000              |
| **get**                 | fixed                 | `get`操作                                                  | size = 可用cpu核心数<br/>queue = 1000                        |
| **analyze**             | fixed                 | `analyze`操作                                              | size = 1<br/>queue = 16                                      |
| **snapshot**            | scaling               | 对索引的`snapshot/restore`操作                             | keep-alive of `5m` and a max of `min(5, (可用cpu核心数)/2)`. |
| **system_write**        | fixed                 | 系统索引的写操作                                           | maximum size of `min(5, (可用cpu核心数/ 2)`                  |
| **system_read**         | fixed                 | 系统索引的读操作                                           | maximum size of `min(5, (可用cpu核心数/ 2)`                  |
| **refresh**             | scaling               | `refresh`操作                                              | keep-alive of `5m` and a max of `min(10, (可用cpu核心数)/2)`. |
| **flush**               | scaling               | `flush`，`synced flush`，`translog fsync`等操作            | keep-alive of `5m` and a max of `min(5, (可用cpu核心数)/2)`. |
| **force_merge**         | fixed                 | `force merge`操作                                          | size = 1 ，队列没有大小限制                                  |
| **fetch_shard_store**   | scaling               | 监控分片的存储                                             | keep-alive of `5m` and 最大大小为(2*可用cpu核心数)           |
| **fetch_shard_started** | scaling               | 监控分片的状态                                             | keep-alive of `5m` and 最大大小为(2*可用cpu核心数)           |
| **listener**            | scaling               | 设置为true时，主要用于为Java客户端执行动作                 | max of `min(10, (可用cpu核心数) / 2)`                        |

从ElasticSearch5.0 开始，无法通过api更改线程池的配置，需要更改`elasticsearch.yml`并重启才能生效配置



# 四、线程信息

## 1、查询节点上的热点线程

### API接口

```bash
GET /_nodes/hot_threads

# 查询指定节点上的热点进程
GET /_nodes/节点ID/hot_threads
```

### 接口参数

- **ignore_idle_threads**（可选，布尔值，默认为true）

  如果为true，则会过滤掉已知的空闲线程（例如，在套接字选择中等待，或从空队列中获取任务）

- **interval**：（可选，时间单位，默认为500ms）

  执行热点线程的采样间隔

- **snapshots：**（可选，整数，默认为10）

  它是要获取的堆栈跟踪（在特定时间点嵌套的方法调用序列）数量

- **threads**（可选，整数，默认为3。也就是返回TOP 3 热点线程。）

  查看由type参数确定的信息，ElasticSearch将采用指定数量的最“热门”线程。

- **master_timeout：**（可选，时间单位，默认为30s）

  指定等待连接到主节点的时间段。如果在超时到期之前未收到任何响应，则请求将失败并返回错误。

- **timeout：**（可选，时间单位，默认为30s）

  指定等待响应的时间段。如果在超时到期之前未收到任何响应，则请求将失败并返回错误。

- **type：**（可选，字符串，默认cpu）

  要采样的类型。可用的选项是：block线程阻塞状态的时间；cpu线程占据CPU时间；wait 线程等待状态的时间。

### 返回信息样本

```bash
::: {log-node1}{w_AbxWEDSqa11saW1WXjEQ}{M2p9u125R_2wiVznDV2-ug}{192.168.1.6}{192.168.1.6:9300}{dilm}{ml.machine_memory=16656637952, rack=r1, xpack.installed=true, ml.max_open_jobs=20}
   Hot threads at 2020-12-15T09:48:50.840Z, interval=500ms, busiestThreads=3, ignoreIdleThreads=true:
   
    2.9% (14.3ms out of 500ms) cpu usage by thread 'elasticsearch[log-node1][refresh][T#1]'
     8/10 snapshots sharing following 2 elements
       java.base@13.0.1/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:628)
       java.base@13.0.1/java.lang.Thread.run(Thread.java:830)
   
    2.4% (12.1ms out of 500ms) cpu usage by thread 'elasticsearch[log-node1][refresh][T#2]'
     8/10 snapshots sharing following 2 elements
       java.base@13.0.1/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:628)
       java.base@13.0.1/java.lang.Thread.run(Thread.java:830)

::: {log-node2}{fllcX9yABXqLYabqgUi7Bw}{YJmD534TlykN_xDx11duA}{192.168.1.7}{192.168.1.7:9300}{dilm}{ml.machine_memory=33566380032, rack=r1, ml.max_open_jobs=20, xpack.installed=true}
   Hot threads at 2020-12-15T09:48:50.840Z, interval=500ms, busiestThreads=3, ignoreIdleThreads=true:
   
    3.4% (17.1ms out of 500ms) cpu usage by thread 'elasticsearch[log-node2][refresh][T#3]'
     8/10 snapshots sharing following 2 elements
       java.base@13.0.1/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:628)
       java.base@13.0.1/java.lang.Thread.run(Thread.java:830)
```

## 2、查询节点上的线程详细信息

```bash
GET /_nodes/thread_pool
```

```bash
{
  "_nodes" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  },
  "cluster_name" : "log",
  "nodes" : {
    "fllcX9yf455LYabqgUi7Bw" : {
      "name" : "log-node2",
      "transport_address" : "192.168.1.7:9300",
      "host" : "192.168.1.7",
      "ip" : "192.168.1.7",
      "version" : "7.5.1",
      "build_flavor" : "default",
      "build_type" : "rpm",
      "build_hash" : "3ae9ac9a93c9512551cf95d88e1e18d96",
      "roles" : [
        "ingest",
        "master",
        "data",
        "ml"
      ],
      "attributes" : {
        "ml.machine_memory" : "33566380032",
        "rack" : "r1",
        "ml.max_open_jobs" : "20",
        "xpack.installed" : "true"
      },
      "thread_pool" : {
        "watcher" : {
          "type" : "fixed",
          "size" : 40,
          "queue_size" : 1000
        },
        "force_merge" : {
          "type" : "fixed",
          "size" : 1,
          "queue_size" : -1
        },
				# .....省略.....
      }
    }
  }
}
```



# 参考

1. https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-threadpool.html#fixed-auto-queue-size