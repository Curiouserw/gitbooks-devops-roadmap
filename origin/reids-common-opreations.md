# Redis常用操作

# 一、全局命令

## 0、配置操作

### ①添加设置

```bash
config set parameter value
```

### ②获取设置

```bash
config get parameter 
```

### ③重置INFO命令统计的一些计算器

```bash
config resetstat
# 被重置的数据如下:
#   Keyspace hits
#   Keyspace misses
#   Number of commands processed
#   Number of connections received
#   Number of expired keys
```

## 1、DB库操作

### ①切换库

```bash
select db
# 16个DB库，0-15.默认0库
```

## 2、客户端操作

### ①查询客户端连接信息

```bash
client list
```

### ②剔除杀掉指定客户端

```bash
client kill ip:port
```

### ③返回当前连接名

```bash
client getname
# 返回当前连接由CLIENT SETNAME设置的名字。如果没有用CLIENT SETNAME设置名字，将返回一个空的回复。
```

### ④返回当前连接的ID

```bash
client id
# 每个ID符合如下约束：
#  1.永不重复。当调用命令CLIENT ID返回相同的值时，调用者可以确认原连接未被断开，只是被重用 ，因此仍可以认为是同一连接
#  2.ID值单调递增。若某一连接的ID值比其他连接的ID值大，可以确认该连接是较新创建的
```

## 3、全局数据操作

### ①清除数据

```bash
# 清除DB中的所有数据
flushdb 
# 清除所有库中的数据
flushall
```

## 4、监控统计操作

### ①实时显示接受到的命令

```bash
monitor
```

### ②Redis服务器与统计信息

```bash
info [section]

# section:
#   1.server: Redis服务器的一般信息
#   2.clients: 客户端的连接部分
#   3.memory: 内存消耗相关信息
#   4.persistence: RDB和AOF相关信息
#   5.stats: 一般统计
#   6.replication: 主/从复制信息
#   7.cpu: 统计CPU的消耗
#   8.commandstats: Redis命令统计
#   9.cluster: Redis集群信息
#   10.keyspace: 数据库的相关统计
# all: 返回所有信息
# default: 值返回默认设置的信息
# 如果没有使用任何参数时，默认为default。
```

#### info memory指标

| 指标                          | 含义                                                         |
| :---------------------------- | :----------------------------------------------------------- |
| **used_memory**               | 由 Redis 分配器分配的内存总量，包含了redis进程内部的开销和数据占用的内存，以字节（byte）为单位，即当前redis使用内存大小。 |
| **used_memory_human**         | 已更直观的单位展示分配的内存总量。                           |
| **used_memory_rss**           | 向操作系统申请的内存大小，与 top 、 ps等命令的输出一致，即redis使用的物理内存大小。 |
| **used_memory_rss_human**     | 已更直观的单位展示向操作系统申请的内存大小。                 |
| **used_memory_peak**          | redis的内存消耗峰值(以字节为单位)，即历史使用记录中redis使用内存峰值。 |
| **used_memory_peak_human**    | 以更直观的格式返回redis的内存消耗峰值                        |
| **used_memory_peak_perc**     | 使用内存达到峰值内存的百分比，used_memory/ used_memory_peak) *100%，即当前redis使用内存/历史使用记录中redis使用内存峰值*100% |
| **used_memory_overhead**      | Redis为了维护数据集的内部机制所需的内存开销，包括所有客户端输出缓冲区、查询缓冲区、AOF重写缓冲区和主从复制的backlog。 |
| **used_memory_startup**       | Redis服务器启动时消耗的内存                                  |
| **used_memory_dataset**       | 数据实际占用的内存大小，即used_memory-used_memory_overhead   |
| **used_memory_dataset_perc**  | 数据占用的内存大小的百分比，100%*(used_memory_dataset/(used_memory-used_memory_startup)) |
| **total_system_memory**       | 整个系统内存                                                 |
| **total_system_memory_human** | 以更直观的格式显示整个系统内存                               |
| **used_memory_lua**           | Lua脚本存储占用的内存                                        |
| **used_memory_lua_human**     | 以更直观的格式显示Lua脚本存储占用的内存                      |
| **maxmemory**                 | Redis实例的最大内存配置                                      |
| **maxmemory_human**           | 以更直观的格式显示Redis实例的最大内存配置                    |
| **maxmemory_policy**          | 当达到maxmemory时的淘汰策略                                  |
| **mem_fragmentation_ratio**   | 碎片率，used_memory_rss/ used_memory。ratio指数>1表明有内存碎片，越大表明越多，<1表明正在使用虚拟内存，虚拟内存其实就是硬盘，性能比内存低得多，这是应该增强机器的内存以提高性能。一般来说，mem_fragmentation_ratio的数值在1 ~ 1.5之间是比较健康的。详解 |
| **mem_allocator**             | 内存分配器                                                   |
| **active_defrag_running**     | 表示没有活动的defrag任务正在运行，1表示有活动的defrag任务正在运行（defrag:表示内存碎片整理）详解 |
| **lazyfree_pending_objects**  | 0表示不存在延迟释放的挂起对象                                |

### ③查询键值在内存中占用的字节数

```bash
memory usage key
# 返回的结果是key的值以及为管理该key分配的内存总字节数。对于嵌套数据类型，可以使用选项SAMPLES，其中COUNT表示抽样的元素个数，默认值为5。当需要抽样所有元素时，使用SAMPLES 0

# 如上，实际数据为空，但是存储时仍然耗费了一些内存，这些内存用于Redis 服务器维护内部数据结构。随着key和value的增大，内存使用量和key 大小基本成 线性关系。
```

### ④获取最后一次同步磁盘的时间

```bash
lastsave
# 执行成功时返回UNIX时间戳
```

## 5、慢日志操作

Redis慢查询日志是一个记录超过指定执行时间的查询的系统。 这里的执行时间不包括IO操作，比如与客户端通信，发送回复等等，而只是实际执行命令所需的时间（这是唯一在命令执行过程中线程被阻塞且不能同时处理其他请求的阶段）。

慢查询日志在内存中堆积，因此不会写入一个包含慢速命令执行信息的文件。 这使得慢查询日志非常快，你可以开启所有命令的日志记录（设置_slowlog-log-slower-than_参数值为零），但性能较低。

### ①获取慢日志

```bash
slowlog get [条数]
```

每一个条目由四个字段组成：

- 每个慢查询条目的唯一的递增标识符。
- 处理记录命令的unix时间戳。
- 命令执行所需的总时间，以微秒为单位。
- 组成该命令的参数的数组。

条目的唯一ID可以用于避免慢查询条目被多次处理（例如，你也许有一个脚本使用每个新的慢查询日志条目给你发送报警邮件）。

条目ID在Redis服务器运行期间绝不会被重置，仅在Redis服务重启才重置它。

### ②设置慢查询日志

```bash
config set _slowlog-log-slower-than_ milliseconds
# Redis命令的执行时间超过多少微秒将会被记录。 请注意，
#   1.负数将会关闭慢查询日志
#   2.0将强制记录每一个命令
config set _slowlog-max-len_ 慢日志条目最大条数
# 设置慢查询日志的长度。 最小值是0。 当一个新命令被记录，且慢查询日志已经达到其最大长度时，将从记录命令的队列中移除删除最旧的命令以腾出空间。
```

### ③获取慢查询日志的最新条目ID

```bash
slowlog len
```

### ④重置慢查询日志

```bash
slowlog reset
# 删除后，信息将永远丢失
```

# 二、Key操作

## 查找key

查找所有符合给定模式pattern（正则表达式）的 key 

```bash
key 
# 支持的正则表达模式：

# h?llo 匹配 hello, hallo 和 hxllo
# h*llo 匹配 hllo 和 heeeello
# h[ae]llo 匹配 hello 和 hallo, 但是不匹配 hillo
# h[^e]llo 匹配 hallo, hbllo, … 但是不匹配 hello
# h[a-b]llo 匹配 hallo 和 hbllo
```

## 查询key的值

```bash
get key
# 不存在则返回nil
```

## 批量获取key的值

```bash
mget key [key .....]
# mget 1 2 3
```

## 查询当前DB的key总数

```bash
dbsize
```

## 检查key是否存在

```bash
exist key
# 存在返回1，不存在返回0
```

## 查询key的过期时间

```bash
ttl key # 查询键在多少秒后过期 （>0 剩余过期时间；-1 没有设置过期时间；-2 键不存在）
pttl key # 查询键在多少毫秒后过期
```

## 查询key的数据类型

```bash
type key （如果键不存在，则返回none）
```

## 删除Key

```bash
del key
```

## 统计Key

```bash
scan cursor match 正则表达式
```

# 三、String类型键的操作命令

## 1、DDL

### ①设置键值

```bash
set key value [ex] [px] [nx|xx]
# ex 秒级过期时间
# px 毫秒级过期时间
```

### ②设置键值并返回原值

```bash
getset key value
```

### ③批量设置键值

```bash
mset key value [key value]
# mset 1 a 2 b 3 c
```

### ④设置键的过期时间

```bash
expire key senconds/milliseconds # 设置key在second秒/milliseconds毫秒后过期
expire key timestamp/milliseconds-timestamp # 设置key在秒级时间戳/毫秒时间戳戳后过期 
```

### ⑤重命名键

```bash
rename key newkey
```

### ⑥追加内容到String类型键的末尾

```bash
append key value
```

## 2、DQL

### ①查询string类型键的长度

```bash
strlen key
```

### ②查询String类型键指定长度的值

```bash
getrange key start end
# start和end指从0开始的开始与结束偏移量
```

# 四、Hash操作命令

## 1、DDL

### ①创建hash字段

```bash
hset key field value
```

### ②批量创建hash字段

```bash
hmset key field value [field value ...]
```

### ③删除hash中一个或多个field

```bash
hdel key field [key field ....]
```

## 2、DQL

### ①获取字段的值

```bash
hset get key field
```

### ②批量获取字段的值

```bash
hmget key field [field ...]
```

### ③获取所有字段的数量

```bash
hlen key
```

### ④获取所有的字段

```bash
hkeys key
```

### ⑤获取指定字段的长度

```bash
hstrlen key field
```

### ⑥获取所有的字段值

```bash
hvals key
```

### ⑦判断是否存在field

```bash
hexists key field
```

### ⑧读取所有的field与值

```bash
hgetall key
```

# 五、List操作命令

## 1、DDL

### ①将一个或多个的值插入列表的头部

```bash
lpush key value1 [value2 ...]
# 如果 key 不存在，那么在进行 push 操作前会创建一个空列表。
# 元素是从最左端的到最右端的、一个接一个被插入到 list 的头部。
# redis版本>= 2.4才可以接受多个 value 参数。< 2.4 的 Redis 只能每条命令 push 一个值。
# 返回值是在 push 操作后的 list 长度。

lpushx key value1 [value2 ...]
# 只有当 key 已经存在并且存着一个 list 的时候，在这个 key 下面的 list 的头部插入 value。
```

### ② 将一个或多个的值插入列表的尾部

```bash
rpush key value1 [value2 ...]
# 如果 key 不存在，那么会创建一个空的列表然后再进行 push 操作
# 元素是从左到右一个接一个从列表尾部插入
# 返回值是在 push 操作后的列表长度。

rpushx key value1 [value2 ...]
# 只有当 key 已经存在并且存着一个 list 的时候，在这个 key 下面的 list 的尾部部插入 value。
```

### ③插入已有列表某个元素前面

```bash
linsert key before value new_value
```

### ④弹出列表第一个元素并返回元素的值

```bash
lpop key
# 返回第一个元素的值，或者当 key 不存在时返回 nil。
```

### ⑤弹出列表最后一个元素并返回元素的值

```bash
rpop key
# 当 key 不存在的时候返回 nil。
```

### ⑥弹出列表中的最后一个元素，并将其追加到另外一个列表的头部

```bash
rpoplpush list1 list2
# 如果 list2 不存在，那么会返回 nil 值，并且不会执行任何操作。 如果 list1 和 list2 是同样的，那么这个操作等同于移除列表最后一个元素并且把该元素放在列表头部， 所以这个命令也可以当作是一个旋转列表的命令
```

### ⑦从列表中删除指定个数的重复元素

```bash
lrem key count value
# 从存于 key 的列表里移除前 count 次出现的值为 value 的元素。 这个 count 参数通过下面几种方式影响这个操作：
#   count > 0: 从头往尾移除值为 value 的元素。
#   count < 0: 从尾往头移除值为 value 的元素。
#   count = 0: 移除所有值为 value 的元素。
```

### ⑧设置指定索引位置元素的值

```bash
 lset key index value
# 当index超出范围时会返回一个error。
```

## 2、DQL

### ①根据索引获取一个元素

```bash
lindex key 0
# 0 是表示第一个元素， 1 表示第二个元素，并以此类推。 负数索引用于指定从列表尾部开始索引的元素。在这种方法下，-1 表示最后一个元素，-2 表示倒数第二个元素，并以此往前推。
```

### ②查询指定范围内的元素

```bash
lrange key start end
#  start 和 end 偏移量都是基于0的下标
# 当下标超过list范围的时候不会产生error。 如果start比list的尾部下标大的时候，会返回一个空列表。 如果stop比list的实际尾部大的时候，Redis会当它是最后一个元素的下标。
```

### ③获取队列的长度

```bash
llen key
# 如果 key 不存在，那么就被看作是空list，并且返回长度为 0。 当存储在 key 里的值不是一个list的话，会返回error。
```

