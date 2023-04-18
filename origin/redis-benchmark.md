# Redis性能测试工具：redis-benchmark

# 一、简介

基本命令语法如下：

```
redis-benchmark [option] [option value]
```

可选参数如下所示：

| 选项      | 描述                                       | 默认值    |
| :-------- | :----------------------------------------- | :-------- |
| **-h**    | 指定服务器主机名                           | 127.0.0.1 |
| **-p**    | 指定服务器端口                             | 6379      |
| **-s**    | 指定服务器 socket                          |           |
| **-c**    | 指定并发连接数                             | 50        |
| **-n**    | 指定请求数                                 | 10000     |
| **-d**    | 以字节的形式指定 SET/GET 值的数据大小      | 2         |
| **-k**    | 1=keep alive 0=reconnect                   | 1         |
| **-r**    | SET/GET/INCR 使用随机 key, SADD 使用随机值 |           |
| **-P**    | 通过管道传输请求                           | 1         |
| **-q**    | 强制退出 redis。仅显示 query/sec 值        |           |
| **--csv** | 以 CSV 格式输出                            |           |
| **-l**    | 生成循环，永久执行测试                     |           |
| **-t**    | 仅运行以逗号分隔的测试命令列表。           |           |
| **-I**    | Idle 模式。仅打开 N 个 idle 连接并等待。   |           |

# 二、示例

## 1、测试所有命令

```bash
redis-benchmark -n 100000 -q


PING_INLINE: 106044.54 requests per second
PING_BULK: 104058.27 requests per second
SET: 106382.98 requests per second
GET: 106496.27 requests per second
INCR: 109289.62 requests per second
LPUSH: 100806.45 requests per second
RPUSH: 105042.02 requests per second
LPOP: 104602.52 requests per second
RPOP: 88809.95 requests per second
SADD: 103842.16 requests per second
HSET: 94250.71 requests per second
SPOP: 94517.96 requests per second
LPUSH (needed to benchmark LRANGE): 93283.58 requests per second
LRANGE_100 (first 100 elements): 26795.28 requests per second
LRANGE_300 (first 300 elements): 11272.69 requests per second
LRANGE_500 (first 450 elements): 7966.22 requests per second
LRANGE_600 (first 600 elements): 6308.35 requests per second
MSET (10 keys): 77101.00 requests per second
```



## 2、测试设置 10 万随机 key 连续 SET 100 万次

```bash
redis-benchmark -t set -r 100000 -n 1000000
====== SET ======
  1000000 requests completed in 10.21 seconds
  50 parallel clients													# 模拟 50 个客户端
  3 bytes payload
  keep alive: 1

99.38% <= 1 milliseconds
99.99% <= 2 milliseconds
99.99% <= 3 milliseconds
100.00% <= 5 milliseconds
100.00% <= 6 milliseconds
100.00% <= 6 milliseconds
97971.98 requests per second

```

## 3、使用 pipelining进行测试

Redis是一种基于客户端-服务端模型以及请求/响应协议的TCP服务。

这意味着通常情况下一个请求会遵循以下步骤：

- 客户端向服务端发送一个查询请求，并监听Socket返回，通常是以阻塞模式，等待服务端响应。
- 服务端处理命令，并将结果返回给客户端。

默认情况下，每个客户端都是在一个请求完成之后才发送下一个请求 服务器几乎是按顺序读取每个客户端的命令。Redis 支持 [pipelining](http://www.redis.cn/topics/pipelining)，一次请求/响应服务器能实现处理新的请求即使旧的请求还未被响应。这样就可以将*多个命令*发送到服务器，而不用等待回复，最后在一个步骤中读取该答复。使得可以一次性执行多条命令成为可能。 Redis pipelining 可以提高服务器的 TPS。 

```shell
# 设置16 条命令使用 pipelining
redis-benchmark -r 1000000 -n 2000000 -t get,set,lpush,lpop -P 16 -q

SET: 472255.00 requests per second
GET: 559753.69 requests per second
LPUSH: 569638.31 requests per second
LPOP: 571428.56 requests per second

# 不使用pipelining
redis-benchmark -r 1000000 -n 2000000 -t get,set,lpush,lpop -q

SET: 86422.95 requests per second
GET: 88409.52 requests per second
LPUSH: 87680.84 requests per second
LPOP: 80994.61 requests per second
```

## 4、使用Lua脚本测试

```bash
redis-benchmark -n 100000 -q script load "redis.call('set','foo','bar')"

script load redis.call('set','foo','bar'): 105042.02 requests per second
```



# 三、影响Redis 性能的因素

## 1、网络带宽和延迟

网络带宽和延迟通常是最大短板。建议在基准测试之前使用 ping 来检查服务端到客户端的延迟。根据带宽，可以计算出最大吞吐量。 比如将 4 KB 的字符串塞入 Redis，吞吐量是 100000 q/s，那么实际需要 3.2 Gbits/s 的带宽，所以需要 10 GBits/s 网络连接， 1 Gbits/s 是不够的。 在很多线上服务中，Redis 吞吐会先被网络带宽限制住，而不是 CPU。 为了达到高吞吐量突破 TCP/IP 限制，最后采用 10 Gbits/s 的网卡， 或者多个 1 Gbits/s 网卡。

## 2、CPU

由于Redis是单线程模型，Redis 更喜欢大缓存快速 CPU， 而不是多核。

## 3、连接方式

如果服务器和客户端都运行在同一个机器上面，那么 TCP/IP loopback 和 unix domain sockets 都可以使用。对 Linux 来说，使用 unix socket 可以比 TCP/IP loopback 快 50%。 默认 redis-benchmark 是使用 TCP/IP loopback。 当大量使用 pipelining 时候，unix domain sockets 的优势就不那么明显了。

## 4、部署环境

Redis 在 VM 上会变慢。虚拟化对普通操作会有额外的消耗，Redis 对系统调用和网络终端不会有太多的 overhead。建议把 Redis 运行在物理机器上， 特别是当你很在意延迟时候。在最先进的虚拟化设备（VMWare）上面，redis-benchmark 的测试结果比物理机器上慢了一倍，很多 CPU 时间被消费在系统调用和中断上面。

