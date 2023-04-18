# TiDB TiFlash

# 一、简介

[TiFlash](https://github.com/pingcap/tiflash) 是 TiDB HTAP 形态的关键组件，它是 TiKV 的列存扩展，在提供了良好的隔离性的同时，也兼顾了强一致性。列存副本通过 Raft Learner 协议异步复制，但是在读取的时候通过 Raft 校对索引配合 MVCC 的方式获得 Snapshot Isolation 的一致性隔离级别。这个架构很好地解决了 HTAP 场景的隔离性以及列存同步的问题。

## TiFlash架构

<img src="../assets/tidb-storage-architecture.png" style="zoom:50%;" />

- TiFlash 提供列式存储，且拥有借助 ClickHouse 高效实现的协处理器层。除此以外，它与 TiKV 非常类似，依赖同样的 Multi-Raft 体系，以 Region 为单位进行数据复制和分散。

- TiFlash 以低消耗不阻塞 TiKV 写入的方式，实时复制 TiKV 集群中的数据，并同时提供与 TiKV 一样的一致性读取，且可以保证读取到最新的数据。TiFlash 中的 Region 副本与 TiKV 中完全对应，且会跟随 TiKV 中的 Leader 副本同时进行分裂与合并。

- TiFlash 可以兼容 TiDB 与 TiSpark，用户可以选择使用不同的计算引擎。

- TiFlash 推荐使用和 TiKV 不同的节点以做到 Workload 隔离，但在无业务隔离的前提下，也可以选择与 TiKV 同节点部署。

- TiFlash 暂时无法直接接受数据写入，任何数据必须先写入 TiKV 再同步到 TiFlash。TiFlash 以 learner 角色接入 TiDB 集群，TiFlash 支持表粒度的数据同步，部署后默认情况下不会同步任何数据，需要按照第三章手动完成指定表的数据同步。

- TiFlash 主要包含三个组件，除了主要的存储引擎组件，另外包含 tiflash proxy 和 pd buddy 组件，其中 tiflash proxy 主要用于处理 Multi-Raft 协议通信的相关工作，pd buddy 负责与 PD 协同工作，将 TiKV 数据按表同步到 TiFlash。

- 对于按表构建 TiFlash 副本的流程，TiDB 接收到相应的 DDL 命令后 pd buddy 组件会通过 TiDB 的 status 端口获取到需要同步的数据表信息，然后会将需要同步的数据信息发送到 PD，PD 根据该信息进行相关的数据调度。

## TiFlash特性

- **异步复制**

  TiFlash 中的副本以特殊角色 (Raft Learner) 进行异步的数据复制。这表示当 TiFlash 节点宕机或者网络高延迟等状况发生时，TiKV 的业务仍然能确保正常进行。

  这套复制机制也继承了 TiKV 体系的自动负载均衡和高可用：并不用依赖附加的复制管道，而是直接以多对多方式接收 TiKV 的数据传输；且只要 TiKV 中数据不丢失，就可以随时恢复 TiFlash 的副本。

- **一致性**

  TiFlash 提供与 TiKV 一样的快照隔离支持，且保证读取数据最新（确保之前写入的数据能被读取）。这个一致性是通过对数据进行复制进度校验做到的。

  每次收到读取请求，TiFlash 中的 Region 副本会向 Leader 副本发起进度校对（一个非常轻的 RPC 请求），只有当进度确保至少所包含读取请求时间戳所覆盖的数据之后才响应读取。

- **智能选择**

  TiDB 可以自动选择使用 TiFlash 列存或者 TiKV 行存，甚至在同一查询内混合使用提供最佳查询速度。这个选择机制与 TiDB 选取不同索引提供查询类似：根据统计信息判断读取代价并作出合理选择。

- **计算加速**

  TiFlash 对 TiDB 的计算加速分为两部分：列存本身的读取效率提升以及为 TiDB 分担计算。其中分担计算的原理和 TiKV 的协处理器一致：TiDB 会将可以由存储层分担的计算下推。能否下推取决于 TiFlash 是否可以支持相关下推。具体介绍请参阅[“TiFlash 支持的计算下推”](https://docs.pingcap.com/zh/tidb/stable/tiflash-supported-pushdown-calculations)一节。

# 二、部署

## 1. 添加TiFlash节点配置

编写 tiflash-scale-out.yaml 文件，添加该 TiFlash 节点信息（目前只支持 ip，不支持域名）：

```yaml
tiflash_servers:
  - host: 10.0.1.4
    config:
      logger.level: info
```

TiFlash节点部署配置参数

| 配置 | 含义 |
| --- | -- |
| host | 指定部署到哪台机器，字段值填 IP 地址，不可省略 |
| ssh_port | 指定连接目标机器进行操作的时候使用的 SSH 端口，若不指定，则使用 global 区块中的 ssh_port |
| tcp_port | TiFlash TCP 服务的端口，默认 9000 |
| http_port | TiFlash HTTP 服务的端口，默认 8123 |
| flash_service_port | TiFlash 提供服务的端口，TiDB 通过该端口从 TiFlash 读数据，默认 3930 |
| metrics_port | TiFlash 的状态端口，用于输出 metric 数据，默认 8234 |
| flash_proxy_port | 内置 TiKV 的端口，默认 20170 |
| flash_proxy_status_port | 内置 TiKV 的状态端口，默认为 20292 |
| deploy_dir | 指定部署目录，若不指定，或指定为相对目录，则按照 global 中配置的 deploy_dir 生成 |
| data_dir | 指定数据目录，若不指定，或指定为相对目录，则按照 global 中配置的 data_dir 生成，TiFlash 的数据目录支持多个，采用逗号分割 |
| log_dir | 指定日志目录，若不指定，或指定为相对目录，则按照 global 中配置的 log_dir 生成 |
| tmp_path: |TiFlash 临时文件的存放路径，默认使用 [path 或者 storage.latest.dir 的第一个目录] + "/tmp"|
| numa_node | 为该实例分配 NUMA 策略，如果指定了该参数，需要确保目标机装了 numactl，在指定该参数的情况下会通过 numactl 分配 cpubind 和 membind 策略。该字段参数为 string 类型，字段值填 NUMA 节点的 ID，例如 "0,1" |
| config | 该字段配置规则和 server_configs 里的 tiflash 配置规则相同，若配置了该字段，会将该字段内容和 server_configs 里的 tiflash 内容合并（若字段重叠，以本字段内容为准），然后生成配置文件并下发到 host 指定的机器 |
| learner_config | 每个 TiFlash 中内置了一个特殊的 TiKV 模块，该配置项用于配置这个特殊的 TiKV 模块，一般不建议修改这个配置项下的内容 |
| os | host 字段所指定的机器的操作系统，若不指定该字段，则默认为 global 中的 os |
| arch | host 字段所指定的机器的架构，若不指定该字段，则默认为 global 中的 arch |
| resource_control | 针对该服务的资源控制，如果配置了该字段，会将该字段和 global 中的 resource_control 内容合并（若字段重叠，以本字段内容为准），然后生成 systemd 配置文件并下发到 host 指定机器。resource_control 的配置规则同 global 中的 resource_control |

部署完成之后不能再修改的参数：`host、tcp_port、http_port、flash_service_port、flash_proxy_port、flash_proxy_status_port、metrics_port、deploy_dir、log_dir、tmp_path、arch、os`

## 2. 扩容TiFlash节点

```bash
tiup cluster scale-out <cluster-name> scale-out.yaml

# 需要当前执行命令的用户和新增的机器打通了互信，如果不满足已打通互信的条件，需要通过 `-p` 来输入新机器的密码，或通过 `-i` 指定私钥文件。
```

## 3. 查看集群TiFlash状态

```sh
tiup cluster display <cluster-name>
```

## 4. 缩容TiFlash节点

### ①调整数据表的副本数

在下线节点之前，确保 TiFlash 集群剩余节点数大于等于所有数据表的最大副本数，否则需要修改相关表的 TiFlash 副本数。

- 在 TiDB 客户端中针对所有副本数大于集群剩余 TiFlash 节点数的表执行：

```sql
alter table <库名>.<表名> set tiflash replica 0;
```

- 等待相关表的 TiFlash 副本被删除

### ②缩容TiFlash节点

```bash
tiup cluster scale-in <cluster-name> --node 10.0.1.4:9000
```

### ③手动缩容 TiFlash 节点

在特殊情况下（比如需要强制下线节点），或者 TiUP 操作失败的情况下，可以使用以下方法手动下线 TiFlash 节点。

- 使用 pd-ctl 的 store 命令在 PD 中查看该 TiFlash 节点对应的 store id。

  ```bash
  tiup ctl:<cluster-version> pd -u http://<pd_ip>:<pd_port> store
  # 如果集群中有多个 PD 实例，只需在以上命令中指定一个活跃 PD 实例的IP:端口即可。  
  ```

- 在 pd-ctl 中下线该 TiFlash 节点。

  ```bash
  tiup ctl:<cluster-version> pd -u http://<pd_ip>:<pd_port> store delete <store_id>
  # <store_id> 为上一步查到的该 TiFlash 节点对应的 store id。
  # 如果集群中有多个 PD 实例，只需在以上命令中指定一个活跃 PD 实例的IP:端口即可。
  ```

- 等待该 TiFlash 节点对应的 store 消失或者 state_name 变成 Tombstone 再关闭 TiFlash 进程。

- 手动删除 TiFlash 的数据文件，具体位置可查看在集群拓扑配置文件中 TiFlash 配置部分下的 data_dir 目录。

- 从 TiUP 拓扑信息中删除已经下线的 TiFlash 节点信息：

```bash
tiup cluster scale-in <cluster-name> --node <pd_ip>:<pd_port> --force
```

- 注意

  - 如果在集群中所有的 TiFlash 节点停止运行之前，没有取消所有同步到 TiFlash 的表，则需要手动在 PD 中清除同步规则，否则无法成功完成 TiFlash 节点的下线。

    - 查询当前 PD 实例中所有与 TiFlash 相关的数据同步规则。

      ```bash
      curl http://<pd_ip>:<pd_port>/pd/api/v1/config/rules/group/tiflash
      
      [
        {
          "group_id": "tiflash",
          "id": "table-45-r",
          "override": true,
          "start_key": "7480000000000000FF2D5F720000000000FA",
          "end_key": "7480000000000000FF2E00000000000000F8",
          "role": "learner",
          "count": 1,
          "label_constraints": [
            {
              "key": "engine",
              "op": "in",
              "values": [
                "tiflash"
              ]
            }
          ]
        }
      ]
      ```
    
    - 删除所有与 TiFlash 相关的数据同步规则。以 `id` 为 `table-45-r` 的规则为例
    
      ```bash
      curl -v -X DELETE http://<pd_ip>:<pd_port>/pd/api/v1/config/rule/tiflash/table-45-r
      ```

## 5. TiFlash 系统表

`information_schema.tiflash_replica` 系统表的列名及含义如下：

| 列名                | 含义                                                         |
| :------------------ | :----------------------------------------------------------- |
| **TABLE_SCHEMA**    | 数据库名                                                     |
| **TABLE_NAME**      | 表名                                                         |
| **TABLE_ID**        | 表 ID                                                        |
| **REPLICA_COUNT**   | TiFlash 副本数                                               |
| **LOCATION_LABELS** | 给 PD 的 hint，让 Region 的多个副本尽可能按照 LOCATION_LABELS 里的设置分散 |
| **AVAILABLE**       | 是否可用（0/1）                                              |
| **PROGRESS**        | 同步进度 [0.0~1.0]                                           |

## 5. TiFlash重要日志

| 日志信息                                                     | 日志含义                          |
| :----------------------------------------------------------- | :-------------------------------- |
| `[INFO] [<unknown>] ["KVStore: Start to persist [region 47, applied: term 6 index 10]"] [thread_id=23]` | 表示数据开始同步了                |
| `[DEBUG] [<unknown>] ["CoprocessorHandler: grpc::Status DB::CoprocessorHandler::execute(): Handling DAG request"] [thread_id=30]` | 表示开始处理一个 Coprocessor 请求 |
| `[DEBUG] [<unknown>] ["CoprocessorHandler: grpc::Status DB::CoprocessorHandler::execute(): Handle DAG request done"] [thread_id=30]` | 表示完成Coprocessor 请求的处理    |

你可以找到一个 Coprocessor 请求的开始或结束，然后通过日志前面打印的线程号找到该 Coprocessor 请求的其他相关日志。

- **注意：TiFlash默认配置的日志级别为Debug级别，会导致日志文件特别大，可修改配置设置级别为INFO。**

  ```yaml
  # 编辑集群组件配置
  tiup cluster edit-config <cluaster_name>
  
  ...
  tiflash_servers:
  - host: 10.0.1.4
    ...
    config:
      logger.level: info
  
  # 重启TiFlash
  tiup cluster reload <cluaster_name> -R tiflash -N tiflash主机:9000
  ```

## 6. TiFlash 配置参数

```bash
## TiFlash TCP/HTTP 等辅助服务的监听 host。建议配置成 0.0.0.0，即监听本机所有 IP 地址。
listen_host = "0.0.0.0"
## TiFlash TCP 服务的端口
tcp_port = 9000
## TiFlash HTTP 服务的端口
http_port = 8123
## 数据块元信息的内存 cache 大小限制，通常不需要修改
mark_cache_size = 5368709120
## 数据块 min-max 索引的内存 cache 大小限制，通常不需要修改
minmax_index_cache_size = 5368709120
## DeltaIndex 内存 cache 大小限制，默认为 0，代表没有限制
delta_index_cache_size = 0

## TiFlash 数据的存储路径。如果有多个目录，以英文逗号分隔。
## 从 v4.0.9 版本开始，不推荐使用 path 及 path_realtime_mode 参数。推荐使用 [storage] 下的配置项代替，这样在多盘部署的场景下能更好地利用节点性能。
## 从 v5.2.0 版本开始，如果要使用配置项 storage.io_rate_limit，需要同时将 TiFlash 的数据存储路径设置为 storage.main.dir。
## 当 [storage] 配置项存在的情况下，path 和 path_realtime_mode 两个配置会被忽略。
# path = "/tidb-data/tiflash-9000"
## 或
# path = "/ssd0/tidb-data/tiflash,/ssd1/tidb-data/tiflash,/ssd2/tidb-data/tiflash"
## 默认为 false。如果设为 true，且 path 配置了多个目录，表示在第一个目录存放最新数据，在其他目录存放较旧的数据。
# path_realtime_mode = false

## TiFlash 临时文件的存放路径。默认使用 [`path` 或者 `storage.latest.dir` 的第一个目录] + "/tmp"
# tmp_path = "/tidb-data/tiflash-9000/tmp"

## 存储路径相关配置，从 v4.0.9 开始生效
[storage]
    ## 该参数从 v5.2.0 开始废弃，请使用 `[storage.io_rate_limit]` 相关配置
    # bg_task_io_rate_limit = 0

    ## DTFile 储存文件格式
    ## * format_version = 1 老旧文件格式，已废弃
    ## * format_version = 2 v6.0.0 以前版本的默认文件格式
    ## * format_version = 3 v6.0.0 及以后版本的默认文件格式，具有更完善的检验功能
    # format_version = 3
    [storage.main]
    ## 用于存储主要的数据，该目录列表中的数据占总数据的 90% 以上。
    dir = [ "/tidb-data/tiflash-9000" ]
    ## 或
    # dir = [ "/ssd0/tidb-data/tiflash", "/ssd1/tidb-data/tiflash" ]

    ## storage.main.dir 存储目录列表中每个目录的最大可用容量。
    ## * 在未定义配置项，或者列表中全填 0 时，会使用目录所在的硬盘容量
    ## * 以 byte 为单位。目前不支持如 "10GB" 的设置
    ## * capacity 列表的长度应当与 dir 列表长度保持一致
    ## 例如：
    # capacity = [ 10737418240, 10737418240 ]
    [storage.latest]
    ## 用于存储最新的数据，大约占总数据量的 10% 以内，需要较高的 IOPS。
    ## 默认情况该项可留空。在未配置或者为空列表的情况下，会使用 storage.main.dir 的值。
    # dir = [ ]
    ## storage.latest.dir 存储目录列表中，每个目录的最大可用容量。
    # capacity = [ 10737418240, 10737418240 ]

    ## [storage.io_rate_limit] 相关配置从 v5.2.0 开始引入。
    [storage.io_rate_limit]
    ## 该配置项是 I/O 限流功能的开关，默认关闭。TiFlash 的 I/O 限流功能适用于磁盘带宽较小且磁盘带宽大小明确的云盘场景。
    ## I/O 限流功能限制下的读写流量总带宽，单位为 Byte，默认值为 0，即默认关闭 I/O 限流功能。
    # max_bytes_per_sec = 0
    ## max_read_bytes_per_sec 和 max_write_bytes_per_sec 的含义和 max_bytes_per_sec 类似，分别指 I/O 限流功能限制下的读流量总带宽和写流量总带宽。
    ## 分别用两个配置项控制读写带宽限制，适用于一些读写带宽限制分开计算的云盘，例如 GCP 上的 persistent disk。
    ## 当 max_bytes_per_sec 配置不为 0 时，优先使用 max_bytes_per_sec。
    # max_read_bytes_per_sec = 0
    # max_write_bytes_per_sec = 0

    ## 下面的参数用于控制不同 I/O 流量类型分配到的带宽权重，一般不需要调整。
    ## TiFlash 内部将 I/O 请求分成 4 种类型：前台写、后台写、前台读、后台读。
    ## I/O 限流初始化时，TiFlash 会根据下面的权重 (weight) 比例分配带宽。
    ## 以下默认配置表示每一种流量将获得 25 / (25 + 25 + 25 + 25) = 25% 的权重。
    ## 如果将 weight 配置为 0，则对应的 I/O 操作不会被限流。
    # foreground_write_weight = 25
    # background_write_weight = 25
    # foreground_read_weight = 25
    # background_read_weight = 25
    ## TiFlash 支持根据当前的 I/O 负载情况自动调整各种 I/O 类型的限流带宽，有可能会超过设置的权重。
    ## auto_tune_sec 表示自动调整的执行间隔，单位为秒。设为 0 表示关闭自动调整。
    # auto_tune_sec = 5
[flash]
    tidb_status_addr = tidb status 端口地址 # 多个地址以逗号分割
    service_addr =  TiFlash raft 服务 和 coprocessor 服务监听地址
# 多个 TiFlash 节点会选一个 master 来负责往 PD 增删 placement rule，通过 flash.flash_cluster 中的参数控制。
[flash.flash_cluster]
    refresh_interval = master 定时刷新有效期
    update_rule_interval = master 定时向 tidb 获取 TiFlash 副本状态并与 pd 交互
    master_ttl = master 选出后的有效期
    cluster_manager_path = pd buddy 所在目录的绝对路径
    log = pd buddy log 路径
[flash.proxy]
    addr = proxy 监听地址，不填则默认是 127.0.0.1:20170
    advertise-addr = 外部访问 addr 的地址，不填则默认是 "addr"
    data-dir = proxy 数据存储路径
    config = proxy 配置文件路径
    log-file = proxy log 路径
    log-level = proxy log 级别，默认是 "info"
    status-addr = 拉取 proxy metrics｜status 信息的监听地址，不填则默认是 127.0.0.1:20292
    advertise-status-addr = 外部访问 status-addr 的地址，不填则默认是 "status-addr"
[logger]
    ## log 级别（支持 trace、debug、information、warning、error），默认是 "debug"
    level = debug
    log = TiFlash log 路径
    errorlog = TiFlash error log 路径
    ## 单个日志文件的大小，默认是 "100M"
    size = "100M"
    ## 最多保留日志文件个数，默认是 10
    count = 10
[raft]
    pd_addr = pd 服务地址 # 多个地址以逗号隔开
[status]
    ## Prometheus 拉取 metrics 信息的端口，默认是 8234
    metrics_port = 8234
[profiles]
[profiles.default]
    ## 存储引擎的 segment 分裂是否使用逻辑分裂。使用逻辑分裂可以减小写放大，但是会造成一定程度的硬盘空间回收不及时。默认为 false。不建议修改默认选项。
    # dt_enable_logical_split = false
    
    ## 单次 coprocessor 查询过程中，对中间数据的内存限制，单位为 byte，默认为 0，表示不限制
    max_memory_usage = 0
    ## 所有查询过程中，对中间数据的内存限制，单位为 byte，默认为 0，表示不限制
    max_memory_usage_for_all_queries = 0
    ## 从 v5.0 引入，表示 TiFlash Coprocessor 最多同时执行的 cop 请求数量。如果请求数量超过了该配置指定的值，多出的请求会排队等待。如果设为 0 或不设置，则使用默认值，即物理核数的两倍。
    cop_pool_size = 0
    ## 从 v5.0 引入，表示 TiFlash Coprocessor 最多同时执行的 batch 请求数量。如果请求数量超过了该配置指定的值，多出的请求会排队等待。如果设为 0 或不设置，则使用默认值，即物理核数的两倍。
    batch_cop_pool_size = 0
    ## 从 v6.1 引入，指定 TiFlash 执行来自 TiDB 的 ALTER TABLE ... COMPACT 请求时，能同时并行处理的请求数量。
    ## 如果这个值没有设置或设为了 0，则会采用默认值（1）。
    manual_compact_pool_size = 1
    ## 从 v5.4.0 引入，表示是否启用弹性线程池，这项功能可以显著提高 TiFlash 在高并发场景的 CPU 利用率。默认为 true。
    # enable_elastic_threadpool = true
    # TiFlash 存储引擎的压缩算法，支持 LZ4、zstd 和 LZ4HC，大小写不敏感。默认使用 LZ4 算法。
    dt_compression_method = "LZ4"
    ## TiFlash 存储引擎的压缩级别，默认为 1。
    ## 如果 dt_compression_method 设置为 LZ4，推荐将该值设为 1；
    ## 如果 dt_compression_method 设置为 zstd ，推荐将该值设为 -1 或 1，设置为 -1 的压缩率更小，但是读性能会更好；
    ## 如果 dt_compression_method 设置为 LZ4HC，推荐将该值设为 9。
    dt_compression_level = 1
## 安全相关配置，从 v4.0.5 开始生效
[security]
    ## 从 v5.0 引入，控制是否开启日志脱敏
    ## 若开启该选项，日志中的用户数据会以 `?` 代替显示
    ## 注意，tiflash-learner 对应的安全配置选项为 `security.redact-info-log`，需要在tiflash-learner.toml中另外开启
    # redact_info_log = false

    ## 包含可信 SSL CA 列表的文件路径。如果你设置了该值，`cert_path` 和 `key_path` 中的路径也需要填写
    # ca_path = "/path/to/ca.pem"
    ## 包含 PEM 格式的 X509 certificate 文件路径
    # cert_path = "/path/to/tiflash-server.pem"
    ## 包含 PEM 格式的 X509 key 文件路径
    # key_path = "/path/to/tiflash-server-key.pem"
```

## 7. TiDB参数调优

- **对于 OLAP/TiFlash 专属的 TiDB 节点，建议调大读取并发数 [`tidb_distsql_scan_concurrency`](https://docs.pingcap.com/zh/tidb/stable/system-variables#tidb_distsql_scan_concurrency) 到 80**

  ```bash
  set @@tidb_distsql_scan_concurrency = 80;
  ```

- **开启 Super batch 功能**

  [`tidb_allow_batch_cop`](https://docs.pingcap.com/zh/tidb/stable/system-variables#tidb_allow_batch_cop-从-v40-版本开始引入) 变量用来设置从 TiFlash 读取时，是否把 Region 的请求进行合并。当查询中涉及的 Region 数量比较大，可以尝试设置该变量为 `1`（对带 `aggregation` 下推到 TiFlash Coprocessor 的请求生效），或设置该变量为 `2`（对全部下推到 TiFlash Coprocessor 请求生效）。

  ```bash
  set @@tidb_allow_batch_cop = 1;
  ```

- **尝试开启聚合推过 `Join` / `Union` 等 TiDB 算子的优化**

  [`tidb_opt_agg_push_down`](https://docs.pingcap.com/zh/tidb/stable/system-variables#tidb_opt_agg_push_down) 变量用来设置优化器是否执行聚合函数下推到 Join 之前的优化操作。当查询中聚合操作执行很慢时，可以尝试设置该变量为 1。

  ```bash
  set @@tidb_opt_agg_push_down = 1;
  ```

- **尝试开启 `Distinct` 推过 `Join` / `Union` 等 TiDB 算子的优化**

  [`tidb_opt_distinct_agg_push_down`](https://docs.pingcap.com/zh/tidb/stable/system-variables#tidb_opt_distinct_agg_push_down) 变量用来设置优化器是否执行带有 `Distinct` 的聚合函数（比如 `select count(distinct a) from t`）下推到 Coprocessor 的优化操作。当查询中带有 `Distinct` 的聚合操作执行很慢时，可以尝试设置该变量为 `1`。

  ```bash
  set @@tidb_opt_distinct_agg_push_down = 1;
  ```

# 三、构建 TiFlash 副本

TiFlash 部署完成后并不会自动同步数据，而需要手动指定需要同步的表。可通过 MySQL 客户端向 TiDB 发送 DDL 命令来为特定的表建立 TiFlash 副本。

## 1. 按表构建 TiFlash 副本

```sql
# 按表构建TiFlash副本SQL语法格式
ALTER TABLE 表名 SET TIFLASH REPLICA 副本个数;
# count 表示副本数，0 表示删除现有的TiFlash 副本

# 为表建立1个副本
ALTER TABLE `test`.`testtable` SET TIFLASH REPLICA 1;
# 删除副本
ALTER TABLE `test`.`testtable` SET TIFLASH REPLICA 0;
```

**注意事项：**

- 对于相同表的多次 DDL 命令，仅保证最后一次能生效。

- 假设有一张表 t 已经通过上述的 DDL 语句同步到 TiFlash，则通过以下语句创建的表也会自动同步到 TiFlash：

  ```sql
  CREATE TABLE table_name like t;
  ```

- 如果集群版本 < v4.0.6，若先对表创建 TiFlash 副本，再使用 TiDB Lightning 导入数据，会导致数据导入失败。需要在使用 TiDB Lightning 成功导入数据至表后，再对相应的表创建 TiFlash 副本。

- 如果集群版本以及 TiDB Lightning 版本均 >= v4.0.6，无论一个表是否已经创建 TiFlash 副本，你均可以使用 TiDB Lightning 导入数据至该表。但注意此情况会导致 TiDB Lightning 导入数据耗费的时间延长，具体取决于 TiDB Lightning 部署机器的网卡带宽、TiFlash 节点的 CPU 及磁盘负载、TiFlash 副本数等因素。

- 不推荐同步 1000 张以上的表，这会降低 PD 的调度性能。这个限制将在后续版本去除。

- v5.1 版本及后续版本将不再支持设置系统表的 replica。在集群升级前，需要清除相关系统表的 replica，否则升级到较高版本后将无法再修改系统表的 replica 设置。

## 2. 按库构建 TiFlash 副本

```sql
# 按库构建TiFlash副本SQL语法格式
ALTER DATABASE 库名 SET TIFLASH REPLICA 副本个数;
# count 表示副本数，0 表示删除现有的TiFlash 副本

# 为 `test` 库中的所有表建立1个TiFlash副本。
ALTER DATABASE `test` SET TIFLASH REPLICA 1;
# 删除为 `tpch50` 库建立的 TiFlash 副本
ALTER DATABASE `test` SET TIFLASH REPLICA 0;
```

**注意事项：**

- 该命令实际是为用户执行一系列 DDL 操作，对资源要求比较高。如果在执行过程中出现中断，已经执行成功的操作不会回退，未执行的操作不会继续执行。
- 从命令执行开始到该库中所有表都已**同步完成**之前，不建议执行和该库相关的 TiFlash 副本数量设置或其他 DDL 操作，否则最终状态可能非预期。非预期场景包括：
  - 先设置 TiFlash 副本数量为 2，在库中所有的表都同步完成前，再设置 TiFlash 副本数量为 1，不能保证最终所有表的 TiFlash 副本数量都为 1 或都为 2。
  - 在命令执行到结束期间，如果在该库下创建表，则**可能**会对这些新增表创建 TiFlash 副本。
  - 在命令执行到结束期间，如果为该库下的表添加索引，则该命令可能陷入等待，直到添加索引完成。
- 该命令会跳过系统表、视图、临时表以及包含了 TiFlash 不支持字符集的表。

## 3. 查看库表同步进度

### ①查看库表同步进度

```sql
# 查看库的同步进度
SELECT * FROM information_schema.tiflash_replica WHERE TABLE_SCHEMA = '<db_name>';

select table_schema,table_name,replica_count,progress from information_schema.tiflash_replica where progress !=1 ;

# 查看指定表的同步进度
SELECT * FROM information_schema.tiflash_replica WHERE TABLE_SCHEMA = '<db_name>' and TABLE_NAME = '<table_name>';
```

- 查询结果字段：

    - **AVAILABLE**

      表示该表的 TiFlash 副本是否可用。1 代表可用，0 代表不可用。副本状态为可用之后就不再改变，如果通过 DDL 命令修改副本数则会重新计算同步进度。

    - **PROGRESS** 

      示同步进度，在 0.0~1.0 之间，1 代表至少 1 个副本已经完成同步。

### ②查看未设置同步的表

```sql
SELECT TABLE_NAME FROM information_schema.tables where TABLE_SCHEMA = "<db_name>" and TABLE_NAME not in (SELECT TABLE_NAME FROM information_schema.tiflash_replica where TABLE_SCHEMA = "<db_name>");
```

## 4. 设置副本可用区

在配置副本时，如果为了考虑容灾，需要将 TiFlash 的不同数据副本分布到多个数据中心

### ①为TiFlash节点指定label

```yaml
tiflash_servers:
  - host: 172.16.5.81
    config:
      logger.level: "info"
    learner_config:
      server.labels:
        zone: "z1"
  - host: 172.16.5.82
    config:
      logger.level: "info"
    learner_config:
      server.labels:
        zone: "z1"
  - host: 172.16.5.85
    config:
      logger.level: "info"
    learner_config:
      server.labels:
        zone: "z2"
```

注：旧版本中的 `flash.proxy.labels` 配置无法处理可用区名字中的特殊字符，建议使用 `learner_config` 中的 `server.labels` 来进行配置。

### ②创建副本时指定label

```sql
ALTER TABLE 表名 SET TIFLASH REPLICA 副本个数 LOCATION LABELS 标签名;
# ALTER TABLE test SET TIFLASH REPLICA 2 LOCATION LABELS "zone";
```

### ③验证副本可用区调度状态

此时 PD 会根据设置的 label 进行调度，将表 `t` 的两个副本分别调度到两个可用区中。可以通过监控或 pd-ctl 来验证这一点：

```bash
$ pd -u<pd-host>:<pd-port> store
    ...
    "address": "172.16.5.82:23913",
    "labels": [
      { "key": "engine", "value": "tiflash"},
      { "key": "zone", "value": "z1" }
    ],
    "region_count": 4,
    ...
    "address": "172.16.5.81:23913",
    "labels": [
      { "key": "engine", "value": "tiflash"},
      { "key": "zone", "value": "z1" }
    ],
    "region_count": 5,
    ...
    "address": "172.16.5.85:23913",
    "labels": [
      { "key": "engine", "value": "tiflash"},
      { "key": "zone", "value": "z2" }
    ],
    "region_count": 9,
```

# 四、TiDB读取TiFlash

TiDB 提供三种读取 TiFlash 副本的方式。如果添加了 TiFlash 副本，而没有做任何 engine 的配置，则默认使用 CBO(Cost Based Optimization) 方式。

具体有没有选择 TiFlash 副本，可以通过 `explain analyze` 和`desc sql`语句查看

```sql
desc select count(*) from test.t;

+--------------------------+---------+--------------+---------------+--------------------------------+
| id                       | estRows | task         | access object | operator info                  |
+--------------------------+---------+--------------+---------------+--------------------------------+
| StreamAgg_9              | 1.00    | root         |               | funcs:count(1)->Column#4       |
| └─TableReader_17         | 1.00    | root         |               | data:TableFullScan_16          |
|   └─TableFullScan_16     | 1.00    | cop[tiflash] | table:t       | keep order:false, stats:pseudo |
+--------------------------+---------+--------------+---------------+--------------------------------+

explain analyze select count(*) from test.t;

+-----+---------+---------+------+-------------+--------------+-------------+--------+------+
| id  | estRows | actRows | task |access object|execution info|operator info| memory | disk |
+-----+---------+---------+------+-------------+--------------+-------------+--------+------+
| StreamAgg_9     |1.0|1|root|| time:83.8372ms, loops:2 | funcs:count(1)->Column#4   | 372 Bytes | N/A  |
| └─TableReader_17|1.0|1|root||time:83.76ms,loops:2,rpc num: 1,rpc time:83.57ms,proc keys:0|data:TableFullScan_16|152 Bytes|N/A |
|   └─TableFullScan_16|1.0|1|cop[tiflash]|table:t|time:43ms,loops:1|keep order:false,stats:pseudo|N/A|N/A|
+-----+---------+---------+------+-------------+--------------+-------------+--------+------+
```

- `cop[tiflash]` 表示该任务会发送至 TiFlash 进行处理。如果没有选择 TiFlash 副本，可尝试通过 `analyze table` 语句更新统计信息后，再查看 `explain analyze` 结果。

- 需要注意的是，如果表仅有单个 TiFlash 副本且相关节点无法服务，智能选择模式下的查询会不断重试，需要指定 Engine 或者手工 Hint 来读取 TiKV 副本。

## 1. 智能选择

对于创建了 TiFlash 副本的表，TiDB 优化器会自动根据代价估算选择是否使用 TiFlash 副本。

## 2. Engine 隔离

Engine 隔离是通过配置变量来指定所有的查询均使用指定 engine 的副本，可选 engine 为 `"tikv"`、`"tidb"` 和 `"tiflash"`（其中 "tidb" 表示 TiDB 内部的内存表区，主要用于存储一些 TiDB 系统表，用户不能主动使用），分别有 2 个配置级别：

- **TiDB 实例级别，即 INSTANCE 级别**

  在 TiDB 的配置文件添加如下配置项：

  ```bash
  [isolation-read]
  engines = ["tikv", "tidb", "tiflash"]
  ```

  **实例级别的默认配置为 `["tikv", "tidb", "tiflash"]`**

- **会话级别，即 SESSION 级别**

  会话级别的默认配置继承自 TiDB 实例级别的配置。最终的 engine 配置为会话级别配置，即会话级别配置会覆盖实例级别配置。比如实例级别配置了 "tikv"，而会话级别配置了 "tiflash"，则会读取 TiFlash 副本。当 engine 配置为 "tikv, tiflash"，即可以同时读取 TiKV 和 TiFlash 副本，优化器会自动选择。

  ```sql
  set @@session.tidb_isolation_read_engines = "逗号分隔的 engine list";
  # ....sql....
  
  或者
  set SESSION tidb_isolation_read_engines = "逗号分隔的engine";
  # ....sql....
  ```

**注意**

- 由于 TiDB Dashboard等组件需要读取一些存储于 TiDB 内存表区的系统表，因此建议实例级别 engine 配置中始终加入 "tidb" engine。
- 如果查询中的表没有对应engin的副本，比如配置了engine为 "tiflash"而该表没有 TiFlash 副本，则查询会报该表不存在该 engine 副本的错。

## 3. 手工Hint

手工 Hint 可以在满足 engine 隔离的前提下，强制 TiDB 对于某张或某几张表使用指定的副本，使用方法为：

```sql
select /*+ read_from_storage(tiflash[table_name]) */ ... from table_name;
```

如果在查询语句中对表设置了别名，在 Hint 语句中必须使用别名才能使 Hint 生效

```sql
select /*+ read_from_storage(tiflash[alias_a,alias_b]) */ ... from table_name_1 as alias_a, table_name_2 as alias_b where alias_a.column_1 = alias_b.column_2;
```

- 其中 `tiflash[]` 是提示优化器读取 TiFlash 副本，亦可以根据需要使用 `tikv[]` 来提示优化器读取 TiKV 副本。更多关于该 Hint 语句的语法可以参考 [READ_FROM_STORAGE](https://docs.pingcap.com/zh/tidb/stable/optimizer-hints#read_from_storagetiflasht1_name--tl_name--tikvt2_name--tl_name-)。

- 如果 Hint 指定的表在指定的引擎上不存在副本，则 Hint 会被忽略，并产生 warning。另外 Hint 必须在满足 engine 隔离的前提下才会生效，如果 Hint 中指定的引擎不在 engine 隔离列表中，Hint 同样会被忽略，并产生 warning。

**注意**

- MySQL 命令行客户端在 5.7.7 版本之前默认清除了 Optimizer Hints。如果需要在这些早期版本的客户端中使用 `Hint` 语法，需要在启动客户端时加上 `--comments` 选项，例如 `mysql -h 127.0.0.1 -P 4000 -uroot --comments`。

## 三种方式之间关系的总结

- Engine 隔离规定了总的可使用副本 engine 的范围
- 手工 Hint 可以在该范围内进一步实现语句级别及表级别的细粒度的 engine 指定，最终由 CBO 在指定的 engine 范围内根据代价估算最终选取某个 engine 上的副本。

## SQL注意事项

- TiDB 4.0.3 版本之前，在非只读 SQL 语句中（比如 `INSERT INTO ... SELECT`、`SELECT ... FOR UPDATE`、`UPDATE ...`、`DELETE ...`）读取 TiFlash，行为是未定义。

- TiDB 4.0.3 以及后续的版本，TiDB 内部会对非只读 SQL 语句忽略 TiFlash 副本以保证数据写入、更新、删除的正确性。对应的，如果使用了[智能选择](https://docs.pingcap.com/zh/tidb/stable/use-tidb-to-read-tiflash#智能选择)的方式，TiDB 会自动选择非 TiFlash 副本；如果使用了 [Engine 隔离](https://docs.pingcap.com/zh/tidb/stable/use-tidb-to-read-tiflash#engine-隔离)的方式指定仅读取 TiFlash 副本，则查询会报错；而如果使用了[手工 Hint](https://docs.pingcap.com/zh/tidb/stable/use-tidb-to-read-tiflash#手工-hint) 的方式，则 Hint 会被忽略。

# 五、TiFlash常见问题

## 1. TiFlash未能正常启动

该问题可能由多个因素构成，可以通过以下步骤依次排查：

1. 检查系统环境是否是 CentOS8。

   CentOS8 中缺少 `libnsl.so` 系统库，可以通过手动安装的方式解决：

   ```sh
   dnf install libnsl
   ```

2. 检查系统的 `ulimit` 参数设置。

   ```sh
   ulimit -n 1000000
   ```

3. 使用 PD Control 工具检查在该节点（相同 IP 和 Port）是否有之前未成功下线的 TiFlash 实例，并将它们强制下线。（下线步骤参考[手动缩容 TiFlash 节点](https://docs.pingcap.com/zh/tidb/stable/scale-tidb-using-tiup#方案二手动缩容-tiflash-节点)）

## 2. TiFlash副本始终处于不可用状态

该问题一般由于配置错误或者环境问题导致 TiFlash 处于异常状态，可以先通过以下步骤定位问题组件

- 使用 pd-ctl 检查 PD 的 [Placement Rules](https://docs.pingcap.com/zh/tidb/stable/configure-placement-rules) 功能是否开启

  ```bash
  echo 'config show replication' | /path/to/pd-ctl -u http://${pd-ip}:${pd-port}
  ```

- 如果返回 `true`，进入下一步。

- 如果返回 `false`，你需要先[开启 Placement Rules 特性](https://docs.pingcap.com/zh/tidb/stable/configure-placement-rules#开启-placement-rules-特性) 后再进入下一步。

- 通过 TiFlash-Summary 监控面板下的 UpTime 检查操作系统中 TiFlash 进程是否正常。

- 通过 pd-ctl 查看 TiFlash proxy 状态是否正常：

  ```bash
  echo "store" | /path/to/pd-ctl -u http://${pd-ip}:${pd-port}
  # store.labels 中含有 `{"key": "engine", "value": "tiflash"}` 信息的为 TiFlash proxy。
  ```

  查看 pd buddy 是否正常打印日志（日志路径的对应配置项 [flash.flash_cluster] log 设置的值，默认为 TiFlash 配置文件配置的 tmp 目录下）。

1. 检查配置的副本数是否小于等于集群 TiKV 节点数。若配置的副本数超过 TiKV 节点数，则 PD 不会向 TiFlash 同步数据；

   ```sh
   echo 'config placement-rules show' | /path/to/pd-ctl -u http://${pd-ip}:${pd-port}
   ```

   再确认 "default: count" 参数值。

   注意

   开启 Placement Rules 后，原先的 `max-replicas` 及 `location-labels` 配置项将不再生效。如果需要调整副本策略，应当使用 Placement Rules 相关接口。

2. 检查 TiFlash 节点对应 store 所在机器剩余的磁盘空间是否充足。默认情况下当磁盘剩余空间小于该 store 的 capacity 的 20%（通过 low-space-ratio 参数控制）时，PD 不会向 TiFlash 调度数据。

## 3. TiFlash查询时间不稳定

TiFlash 查询时间不稳定，同时错误日志中打印出大量的 Lock Exception

该问题是由于集群中存在大量写入，导致 TiFlash 查询时遇到锁并发生查询重试。

可以在 TiDB 中将查询时间戳设置为 1 秒前（例如：假设当前时间为 '2020-04-08 20:15:01'，可以在执行 query 前执行 `set @@tidb_snapshot='2020-04-08 20:15:00';`），来减小 TiFlash 查询碰到锁的可能性，从而减轻查询时间不稳定的程度。

## 4. 部分查询返回Region Unavailable错误

如果在 TiFlash 上的负载压力过大，会导致 TiFlash 数据同步落后，部分查询可能会返回 `Region Unavailable` 的错误。

在这种情况下，可以增加 TiFlash 节点分担负载压力。

## 5. 数据文件损坏

可依照如下步骤进行处理：

1. 参照[下线 TiFlash 节点](https://docs.pingcap.com/zh/tidb/stable/scale-tidb-using-tiup#方案二手动缩容-tiflash-节点)下线对应的 TiFlash 节点。
2. 清除该 TiFlash 节点的相关数据。
3. 重新在集群中部署 TiFlash 节点。

## 6. TiFlash 分析慢

如果语句中含有 MPP 模式不支持的算子或函数等，TiDB 不会选择 MPP 模式，可能导致分析慢。此时，可以执行 `EXPLAIN` 语句检查 SQL 中是否含有 TiFlash 不支持的函数或算子。

```sql
create table t(a datetime);
alter table t set tiflash replica 1;
insert into t values('2022-01-13');
set @@session.tidb_enforce_mpp=1;
explain select count(*) from t where subtime(a, '12:00:00') > '2022-01-01' group by a;
show warnings;
```

示例中，warning 消息显示，因为 TiDB 5.4 及更早的版本尚不支持 `subtime` 函数的下推，因此 TiDB 没有选择 MPP 模式。

```asciidoc
+---------+------+-----------------------------------------------------------------------------+
> | Level   | Code | Message                                                                     |
+---------+------+-----------------------------------------------------------------------------+
| Warning | 1105 | Scalar function 'subtime'(signature: SubDatetimeAndString, return type: datetime) is not supported to push down to tiflash now.       |
+---------+------+-----------------------------------------------------------------------------+
```

## 7. TiFlash 数据不同步

在部署完 TiFlash 节点且进行了数据的同步操作（ALTER 操作）之后，如果实际没有数据同步到 TiFlash 节点，可以通过以下步骤确认或解决问题：

1. 检查同步操作是否执行

   执行 `ALTER table <tbl_name> set tiflash replica <num>` 操作，查看是否有正常返回:

   - 如果有正常返回，进入下一步。
   - 如果无正常返回，请执行 `SELECT * FROM information_schema.tiflash_replica` 检查是否已经创建 TiFlash replica。如果没有，请重新执行 `ALTER table ${tbl_name} set tiflash replica ${num}`，查看是否有其他执行语句（如 `add index` ），或者检查 DDL 操作是否正常。

2. 检查 TiFlash Region 同步是否正常。

   查看 `progress` 是否有变化:

   - 如果有变化，说明 TiFlash 同步正常，进入下一步。
   - 如果没有变化，说明 TiFlash 同步异常，在 `tidb.log` 中，搜索 `Tiflash replica is not available` 相关日志。检查对应表的 `region have` 是否更新。如果无更新，请通过 `tiflash` 日志进一步排查。

3. 使用 pd-ctl 检查 PD 的 [Placement Rules](https://docs.pingcap.com/zh/tidb/stable/configure-placement-rules) 功能是否开启：

   ```sh
   echo 'config show replication' | /path/to/pd-ctl -u http://<pd-ip>:<pd-port>
   ```

   - 如果返回 `true`，进入下一步。
   - 如果返回 `false`，你需要先[开启 Placement Rules 特性](https://docs.pingcap.com/zh/tidb/stable/configure-placement-rules#开启-placement-rules-特性)，然后进入下一步。

4. 检查集群副本数 `max-replicas` 配置是否合理。

   - 如果 `max-replicas` 取值未超过 TiKV 节点数，进入下一步。
   - 如果 `max-replicas` 超过 TiKV 节点数，PD 不会向 TiFlash 同步数据。此时，请将 `max-replicas` 修改为小于等于 TiKV 节点数的整数。

> 注意： `max-replicas` 的默认值是 3。在生产环境中，TiKV 节点数一般大于该值；在测试环境中，可以修改为 1。
>
> ```bash
>    curl -X POST -d '{
>        "group_id": "pd",
>        "id": "default",
>        "start_key": "",
>        "end_key": "",
>        "role": "voter",
>        "count": 3,
>        "location_labels": [
>        "host"
>        ]
>    }' <http://172.16.x.xxx:2379/pd/api/v1/config/rule>
> ```

5. 检查 TiDB 是否为表创建 `placement-rule`。

   搜索 TiDB DDL Owner 的日志，检查 TiDB 是否通知 PD 添加 `placement-rule`。对于非分区表搜索 `ConfigureTiFlashPDForTable`；对于分区表，搜索 `ConfigureTiFlashPDForPartitions`。有关键字，进入下一步；没有关键字，收集相关组件的日志进行排查。

6. 检查 PD 是否为表设置 `placement-rule`

   可以通过 `curl http://<pd-ip>:<pd-port>/pd/api/v1/config/rules/group/tiflash` 查询比较当前 PD 上的所有 TiFlash Placement Rule。如果观察到有 id 为 `table-<table_id>-r` 的 Rule ，则表示 PD Rule 设置成功。

7. 检查 PD 是否正常发起调度

   查看 `pd.log` 日志是否出现 `table-<table_id>-r` 关键字，且之后是否出现 `add operator` 之类的调度行为。是，PD 调度正常；否，PD 调度异常。


## 8. TiFlash 数据同步卡住

如果 TiFlash 数据一开始可以正常同步，过一段时间后全部或者部分数据无法继续同步，你可以通过以下步骤确认或解决问题：

1. 检查磁盘空间。

   检查磁盘使用空间比例是否高于 `low-space-ratio` 的值（默认值 0.8，即当节点的空间占用比例超过 80% 时，为避免磁盘空间被耗尽，PD 会尽可能避免往该节点迁移数据）。

   - 如果磁盘使用率大于等于 `low-space-ratio`，说明磁盘空间不足。此时，请删除不必要的文件，如 `${data}/flash/` 目录下的 `space_placeholder_file` 文件（必要时可在删除文件后将 `reserve-space` 设置为 0MB）。
   - 如果磁盘使用率小于 `low-space-ratio` ，说明磁盘空间正常，进入下一步。

2. 检查是否有 `down peer` （`down peer` 没有清理干净可能会导致同步卡住）。

   - 执行 `pd-ctl region check-down-peer` 命令检查是否有 `down peer`。
   - 如果存在 `down peer`，执行 `pd-ctl operator add remove-peer <region-id> <tiflash-store-id>` 命令将其清除。

## 9. 数据同步慢

同步慢可能由多种原因引起，你可以按以下步骤进行排查。

1. 调整调度参数取值。

   - 调大 [`store limit`](https://docs.pingcap.com/zh/tidb/stable/configure-store-limit#使用方法)，加快同步速度。

2. 调整 TiFlash 侧负载。

   TiFlash 负载过大会引起同步慢，可通过 Grafana 中的 **TiFlash-Summary** 面板查看各个指标的负载情况：

   - `Applying snapshots Count`: `TiFlash-summary` > `raft` > `Applying snapshots Count`
   - `Snapshot Predecode Duration`: `TiFlash-summary` > `raft` > `Snapshot Predecode Duration`
   - `Snapshot Flush Duration`: `TiFlash-summary` > `raft` > `Snapshot Flush Duration`
   - `Write Stall Duration`: `TiFlash-summary` > `Storage Write Stall` > `Write Stall Duration`
   - `generate snapshot CPU`: `TiFlash-Proxy-Details` > `Thread CPU` > `Region task worker pre-handle/generate snapshot CPU`

   根据业务优先级，调整负载情况。

# 参考

- https://docs.pingcap.com/zh/tidb/stable/tiflash-overview
- https://docs.pingcap.com/zh/tidb/stable/tiup-cluster-topology-reference#tiflash_servers
