# TiCDC增量数据同步工具

# 一、简介

[TiCDC](https://github.com/pingcap/tiflow/tree/master/cdc) 是一款 TiDB 增量数据同步工具，通过拉取上游 TiKV 的数据变更日志，TiCDC 可以将数据解析为有序的行级变更数据输出到下游。

## TiCDC 架构

<img src="../assets/ticdc-architecture-1.jpg" style="zoom:50%;" />

- TiKV Server：代表 TiDB 集群中的 TiKV 节点，当数据发生改变时 TiKV 节点会主动将发生的数据改变以变更日志（KV change logs，简称 change logs）的方式发送给 TiCDC 节点。当然，当 TiCDC 节点发现收到的 change logs 并不是连续的，也会主动发起请求，获得需要的 change logs。
- TiCDC：代表运行了运行 TiCDC 进程的各个节点。每个节点都运行一个 TiCDC 进程，每个进程会从 TiKV 节点中拉取一个或者多个表中的数据改变，并通过 Sink 模块同步到下游系统。
- PD：代表 TiDB 集群中的调度模块，负责集群数据的事实调度，这个模块通常是由 3 个 PD 节点构成的，内部通过 etcd 集群来实现选举等高可用相关的能力。 TiCDC 集群使用了 PD 集群内置的 etcd 集群来保存自己的元数据信息，例如：节点的状态信息，changefeed 配置信息等。
- 目前 TiCDC 支持将数据同步到 TiDB，MySQL 数据库，以及 Kafka 等。

## 使用场景

- 提供多 TiDB 集群，跨区域数据高可用和容灾方案，保证在灾难发生时保证主备集群数据的最终一致性。
- 提供同步实时变更数据到异构系统的服务，为监控、缓存、全文索引、数据分析、异构数据库使用等场景提供数据源。

## 特性

- 提供 TiDB -> TiDB 之间数据容灾复制的能力，实现秒级别 RPO 和分钟级别 RTO
- 提供 TiDB 之间双向复制的能力，支持通过 TiCDC 构建多写多活的 TiDB 集群
- 提供 TiDB -> MySQL（或其他兼容 MySQL 协议的数据库）的低延迟的增量数据同步能力
- 提供 TiDB -> Kafka 增量数据同步能力，推荐的数据格式包含 [Canal-JSON](https://docs.pingcap.com/zh/tidb/v6.5/ticdc-canal-json)，[Avro](https://docs.pingcap.com/zh/tidb/v6.5/ticdc-avro-protocol) 等
- 提供表级别数据同步能力，支持同步过程中过滤数据库、表、DML、DDL 的能力
- 高可用架构，无单点故障；支持动态添加、删除 TiCDC 节点
- 支持通过 [Open API](https://docs.pingcap.com/zh/tidb/v6.5/ticdc-open-api) 进行集群管理，包括查询任务状态；动态修改任务配置；动态创建、删除任务等
- 从 v6.2 版本起，你可以通过配置 sink uri 参数 transaction-atomicity 来控制 TiCDC 是否拆分单表事务。拆分事务可以大幅降低 MySQL sink 同步大事务的延时和内存消耗。

## 数据同步顺序性

- TiCDC 对于所有的 DDL/DML 都能对外输出**至少一次**。
- TiCDC 在 TiKV/TiCDC 集群故障期间可能会重复发相同的 DDL/DML。对于重复的 DDL/DML：
  - MySQL sink 可以重复执行 DDL，对于在下游可重入的 DDL （譬如 truncate table）直接执行成功；对于在下游不可重入的 DDL（譬如 create table），执行失败，TiCDC 会忽略错误继续同步。
  - Kafka sink
    - Kafka sink 提供不同的数据分发策略，可以按照表、主键或 ts 等策略分发数据到不同 Kafka partition。 使用表、主键分发策略，可以保证某一行的更新数据被顺序的发送到相同 partition。
    - 对所有的分发策略，我们都会定期发送 Resolved TS 消息到所有的 topic/partition，表示早于该 Resolved TS 的消息都已经发送到 topic/partition，消费程序可以利用 Resolved TS 对多个 topic/partition 的消息进行排序。
    - Kafka sink 会发送重复的消息，但重复消息不会破坏 Resolved TS 的约束，比如在 changefeed 暂停重启后，可能会按顺序发送 msg1、msg2、msg3、msg2、msg3。你可以在 Kafka 消费端进行过滤。

## 数据同步一致性

- MySQL sink
  - TiCDC 开启 redo log 后保证数据复制的最终一致性
  - TiCDC **保证**单行的更新与上游更新顺序一致。
  - TiCDC **不保证**下游事务的执行顺序和上游完全一致。

## 暂不支持的场景

目前 TiCDC 暂不支持的场景如下：

- 暂不支持单独使用 RawKV 的 TiKV 集群。
- 暂不支持在 TiDB 中[创建 SEQUENCE 的 DDL 操作](https://docs.pingcap.com/zh/tidb/v6.5/sql-statement-create-sequence)和 [SEQUENCE 函数](https://docs.pingcap.com/zh/tidb/v6.5/sql-statement-create-sequence#sequence-函数)。在上游 TiDB 使用 SEQUENCE 时，TiCDC 将会忽略掉上游执行的 SEQUENCE DDL 操作/函数，但是使用 SEQUENCE 函数的 DML 操作可以正确地同步。

# 二、安装部署

## 1、使用 TiUP 部署包含 TiCDC 组件的全新 TiDB 集群

在使用 TiUP 部署全新 TiDB 集群时，支持同时部署 TiCDC 组件。你需要在 TiUP 启动 TiDB 集群时的配置文件中加入 TiCDC 相关的部分，以下是一个示例：

```sh
cdc_servers:
  - host: 10.0.1.20
    gc-ttl: 86400
    data_dir: "/cdc-data"
  - host: 10.0.1.21
    gc-ttl: 86400
    data_dir: "/cdc-data"
```

- `cdc_servers`详细配置参数

  `cdc_servers` 约定了将 TiCDC 服务部署到哪些机器上，同时可以指定每台机器上的服务配置，`cdc_servers` 是一个数组，每个数组元素包含以下字段：

  - `host`：指定部署到哪台机器，字段值填 IP 地址，不可省略
  - `ssh_port`：指定连接目标机器进行操作的时候使用的 SSH 端口，若不指定，则使用 `global` 区块中的 `ssh_port`
  - `port`：TiCDC 服务的监听端口，默认 8300
  - `deploy_dir`：指定部署目录，若不指定，或指定为相对目录，则按照 `global` 中配置的 `deploy_dir` 生成
  - `data_dir`：指定数据目录。若不指定该字段或指定为相对目录，数据目录则按照 `global` 中配置的 `data_dir` 生成。
  - `log_dir`：指定日志目录，若不指定，或指定为相对目录，则按照 `global` 中配置的 `log_dir` 生成
  - `gc-ttl`：TiCDC 在 PD 设置的服务级别 GC safepoint 的 TTL (Time To Live) 时长，单位为秒，默认值为 86400，即 24 小时
  - `tz`：TiCDC 服务使用的时区。TiCDC 在内部转换 timestamp 等时间数据类型和向下游同步数据时使用该时区，默认为进程运行本地时区。
  - `numa_node`：为该实例分配 NUMA 策略，如果指定了该参数，需要确保目标机装了 [numactl](https://linux.die.net/man/8/numactl)，在指定该参数的情况下会通过 [numactl](https://linux.die.net/man/8/numactl) 分配 cpubind 和 membind 策略。该字段参数为 string 类型，字段值填 NUMA 节点的 ID，例如 "0,1"
  - `config`：该字段配置规则和 `server_configs` 里的 `cdc` 内容合并（若字段重叠，以本字段内容为准），然后生成配置文件并下发到 `host` 指定的机器
  - `os`：`host` 字段所指定的机器的操作系统，若不指定该字段，则默认为 `global` 中的 `os`
  - `arch`：`host` 字段所指定的机器的架构，若不指定该字段，则默认为 `global` 中的 `arch`
  - `resource_control`：针对该服务的资源控制，如果配置了该字段，会将该字段和 `global` 中的 `resource_control` 内容合并（若字段重叠，以本字段内容为准），然后生成 systemd 配置文件并下发到 `host` 指定机器。`resource_control` 的配置规则同 `global` 中的 `resource_control`
  - `ticdc_cluster_id`：指定该服务器对应的 TiCDC 集群 ID。若不指定该字段，则自动加入默认 TiCDC 集群。该配置只在 v6.3.0 及以上 TiDB 版本中才生效。

## 2、使用 TiUP 在原有 TiDB 集群上新增或扩容 TiCDC 组件

创建名为 scale-out.yml 的配置文件，包含需要扩容的节点的配置信息

```yaml
cdc_servers:
  - host: 10.1.1.1
    gc-ttl: 86400
    data_dir: /tidb-data/cdc-8300
  - host: 10.1.1.2
    gc-ttl: 86400
    data_dir: /tidb-data/cdc-8300
  - host: 10.0.1.4:8300
    gc-ttl: 86400
    data_dir: /tidb-data/cdc-8300
```

> tiup cluster scale-out <cluster-name> scale-out.yml

## 3、使用 TiUP 在原有 TiDB 集群上移除或缩容 TiCDC 组件

```bash
tiup cluster scale-in <cluster-name> --node cdc_servers_ip:8300
```

## 4、使用 TiUP 变更 TiCDC 集群配置

> tiup cluster edit-config <cluster-name>

```bash
server_configs:
  tidb: {}
  tikv: {}
  pd: {}
  tiflash: {}
  tiflash-learner: {}
  pump: {}
  drainer: {}
  cdc:
    gc-ttl: 172800
```

> `tiup cluster reload -R cdc` 

## 5、使用 TiUP 终止和启动 TiCDC 节点

- 终止 TiCDC 节点：`tiup cluster stop -R cdc`
- 启动 TiCDC 节点：`tiup cluster start -R cdc`
- 重启 TiCDC 节点：`tiup cluster restart -R cdc`

## 6、使用 TiCDC 命令行工具来查看集群状态

```sh
cdc cli capture list
[
  {
    "id": "806e3a1b-0e31-477f-9dd6-f3f2c570abdd",
    "is-owner": true,
    "address": "127.0.0.1:8300",
    "cluster-id": "default"
  },
  {
    "id": "ea2a4203-56fe-43a6-b442-7b295f458ebc",
    "is-owner": false,
    "address": "127.0.0.1:8301",
    "cluster-id": "default"
  }
]
```

- `id`：表示服务进程的 ID。
- `is-owner`：表示该服务进程是否为 owner 节点。
- `address`：该服务进程对外提供接口的地址。
- `cluster-id`：该 TiCDC 的集群 ID，默认值为 `default`。

> 如果TiCDC 是用 TiUP 部署的，需要将以下命令中的 `cdc cli` 替换为 `tiup ctl:<cluster-version> cdc`，例如：`tiup ctl:v6.1.1 cdc capture list --pd=http://192.168.1.1:2379`。

# 三、任务操作

## 1、创建同步任务

```sh
cdc cli changefeed create \
	--server=http://10.0.10.25:8300 \
	--sink-uri="mysql://root:123456@127.0.0.1:3306/" \
	--changefeed-id="simple-replication-task"
```

## 2、查询同步任务列表

使用以下命令来查询同步任务列表：

```sh
cdc cli changefeed list --server=http://10.0.10.25:8300
[{
    "id": "simple-replication-task",
    "summary": {
      "state": "normal",
      "tso": 417886179132964865,
      "checkpoint": "2020-07-07 16:07:44.881",
      "error": null
    }
}]
```

- `checkpoint` 即为 TiCDC 已经将该时间点前的数据同步到了下游。
- `state` 为该同步任务的状态：
  - `normal`：正常同步
  - `stopped`：停止同步（手动暂停）
  - `error`：停止同步（出错）
  - `removed`：已删除任务（只在指定 `--all` 选项时才会显示该状态的任务。未指定时，可通过 `query` 查询该状态的任务）
  - `finished`：任务已经同步到指定 `target-ts`，处于已完成状态（只在指定 `--all` 选项时才会显示该状态的任务。未指定时，可通过 `query` 查询该状态的任务）。

## 3、查询特定同步任务

使用 `changefeed query` 命令可以查询特定同步任务（对应某个同步任务的信息和状态），指定 `--simple` 或 `-s` 参数会简化输出，提供最基本的同步状态和 checkpoint 信息。不指定该参数会输出详细的任务配置、同步状态和同步表信息。

```sh
cdc cli changefeed query -s --server=http://10.0.10.25:8300 --changefeed-id=simple-replication-task
{
 "state": "normal",
 "tso": 419035700154597378,
 "checkpoint": "2020-08-27 10:12:19.579",
 "error": null
}
```

简化输出各字段含义：

- `state` 代表当前 changefeed 的同步状态，各个状态必须和 `changefeed list` 中的状态相同。
- `tso` 代表当前 changefeed 中已经成功写入下游的最大事务 TSO。
- `checkpoint` 代表当前 changefeed 中已经成功写入下游的最大事务 TSO 对应的时间。
- `error` 记录当前 changefeed 是否有错误发生。

输出各字段含义：

- `info` 代表查询 changefeed 的同步配置。
- `status` 代表查询 changefeed 的同步状态信息。
  - `resolved-ts` 代表当前 changefeed 中已经成功从 TiKV 发送到 TiCDC 的最大事务 TS。
  - `checkpoint-ts` 代表当前 changefeed 中已经成功写入下游的最大事务 TS。
  - `admin-job-type`代表一个 changefeed 的状态：
    - `0`：状态正常。
    - `1`：任务暂停，停止任务后所有同步 `processor` 会结束退出，同步任务的配置和同步状态都会保留，可以从 `checkpoint-ts` 恢复任务。
    - `2`：任务恢复，同步任务从 `checkpoint-ts` 继续同步。
    - `3`：任务已删除，接口请求后会结束所有同步 `processor`，并清理同步任务配置信息。同步状态保留，只提供查询，没有其他实际功能。
- `task-status` 代表查询 changefeed 所分配的各个同步子任务的状态信息。

## 4、停止同步任务

```sh
cdc cli changefeed pause --server=http://10.0.10.25:8300 --changefeed-id simple-replication-task
```

- `--changefeed-id=uuid` 为需要操作的 `changefeed` ID。

## 5、恢复同步任务

使用以下命令恢复同步任务：

```sh
cdc cli changefeed resume --server=http://10.0.10.25:8300 --changefeed-id simple-replication-task
```

- `--changefeed-id=uuid` 为需要操作的 `changefeed` ID。
- `--overwrite-checkpoint-ts`：从 v6.2 开始支持指定 changefeed 恢复的起始 TSO。TiCDC 集群将从这个 TSO 开始拉取数据。该项支持 `now` 或一个具体的 TSO（如 434873584621453313），指定的 TSO 应在 (GC safe point, CurrentTSO] 范围内。如未指定该参数，默认从当前的 `checkpoint-ts` 同步数据。
- `--no-confirm`：恢复同步任务时无需用户确认相关信息。默认为 false。



注意

- 若 `--overwrite-checkpoint-ts` 指定的 TSO `t2` 大于 changefeed 的当前 checkpoint TSO `t1`（可通过 `cdc cli changefeed query` 命令获取），则会导致 `t1` 与 `t2` 之间的数据不会同步到下游，造成数据丢失。
- 若 `--overwrite-checkpoint-ts` 指定的 TSO `t2` 小于 changefeed 的当前 checkpoint TSO `t1`，则会导致 TiCDC 集群从一个旧的时间点 `t2` 重新拉取数据，可能会造成数据重复（例如 TiCDC 下游为 MQ sink）。

## 6、删除同步任务

使用以下命令删除同步任务：

```sh
cdc cli changefeed remove --server=http://10.0.10.25:8300 --changefeed-id simple-replication-task
```

- `--changefeed-id=uuid` 为需要操作的 `changefeed` ID。

## 7、更新同步任务配置

TiCDC 从 4.0.4 开始支持非动态修改同步任务配置，修改 changefeed 配置需要按照 `暂停任务 -> 修改配置 -> 恢复任务` 的流程。

```sh
cdc cli changefeed pause --pd=http://10.0.10.22:2379 --changefeed-id test-tidb-to-pulsar
cdc cli changefeed update --pd=http://10.0.10.22:2379 --changefeed-id test-tidb-to-pulsar --config ~/ticdc-changefeed.toml
cdc cli changefeed resume --pd=http://10.0.10.22:2379 --changefeed-id test-tidb-to-pulsar
```

当前支持修改的配置包括：

- changefeed 的 `sink-uri`
- changefeed 配置文件及文件内所有配置
- changefeed 是否使用文件排序和排序目录
- changefeed 的 `target-ts`

## 8、管理同步子任务处理单元 (`processor`)

- 查询 `processor` 列表：

  ```sh
  cdc cli processor list --server=http://10.0.10.25:8300
  ```

  ```sh
  [
          {
                  "id": "9f84ff74-abf9-407f-a6e2-56aa35b33888",
                  "capture-id": "b293999a-4168-4988-a4f4-35d9589b226b",
                  "changefeed-id": "simple-replication-task"
          }
  ]
  ```

- 查询特定 `processor`，对应于某个节点处理的同步子任务信息和状态：

  ```sh
  cdc cli processor query --server=http://10.0.10.25:8300 --changefeed-id=simple-replication-task --capture-id=b293999a-4168-4988-a4f4-35d9589b226b
  ```

  ```sh
  {
    "status": {
      "tables": {
        "56": {    # 56 表示同步表 id，对应 TiDB 中表的 tidb_table_id
          "start-ts": 417474117955485702
        }
      },
      "operation": null,
      "admin-job-type": 0
    },
    "position": {
      "checkpoint-ts": 417474143881789441,
      "resolved-ts": 417474143881789441,
      "count": 0
    }
  }
  ```

以上命令中：

- `status.tables` 中每一个作为 key 的数字代表同步表的 id，对应 TiDB 中表的 tidb_table_id。
- `resolved-ts` 代表当前 Processor 中已经排序数据的最大 TSO。
- `checkpoint-ts` 代表当前 Processor 已经成功写入下游的事务的最大 TSO。

## 9、其他

### 9.1 输出行变更的历史值 *从 v4.0.5 版本开始引入*

默认配置下，同步任务输出的 TiCDC Open Protocol 行变更数据只包含变更后的值，不包含变更前行的值，因此该输出数据不满足 TiCDC Open Protocol 的消费端使用行变更历史值的需求。

从 v4.0.5 开始，TiCDC 支持输出行变更数据的历史值。若要开启该特性，需要在 changefeed 的配置文件的根级别指定以下配置：

```toml
enable-old-value = true
```

从 v5.0 开始默认启用该特性，开启该特性后 TiCDC Open Protocol 的输出格式参考 [TiCDC 开放数据协议 - Row Changed Event](https://docs.pingcap.com/zh/tidb/v6.5/ticdc-open-protocol#row-changed-event)。

### 9.2 同步启用了 TiDB 新的 Collation 框架的表

从 v4.0.15、v5.0.4、v5.1.1 和 v5.2.0 开始，TiCDC 支持同步启用了 TiDB [新的 Collation 框架](https://docs.pingcap.com/zh/tidb/v6.5/character-set-and-collation#新框架下的排序规则支持)的表。

### 9.3 同步没有有效索引的表

从 v4.0.8 开始，TiCDC 支持通过修改任务配置来同步没有有效索引的表。若要开启该特性，需要在 `changefeed` 配置文件的根级别进行如下指定：

```toml
enable-old-value = true
force-replicate = true
```

警告

对于没有有效索引的表，`INSERT` 和 `REPLACE` 等操作不具备可重入性，因此会有数据冗余的风险。TiCDC 在同步过程中只保证数据至少分发一次，因此开启该特性同步没有有效索引的表，一定会导致数据冗余出现。如果不能接受数据冗余，建议增加有效索引，譬如增加具有 `AUTO RANDOM` 属性的主键列。

### 9.4 Unified Sorter 功能

注意

从 v6.0.0 开始，TiCDC 内部默认使用 DB Sorter 引擎来对数据进行排序，不再使用 Unified Sorter。建议用户不再主动配置 Sorter 项。

Unified Sorter 是 TiCDC 中的排序引擎功能，用于缓解以下场景造成的内存溢出问题：

- 如果 TiCDC 数据订阅任务的暂停中断时间长，其间积累了大量的增量更新数据需要同步。
- 从较早的时间点启动数据订阅任务，业务写入量大，积累了大量的更新数据需要同步。

对 v4.0.13 版本之后的 `cdc cli` 创建的 changefeed，默认开启 Unified Sorter。对 v4.0.13 版本前已经存在的 changefeed，则使用之前的配置。

要确定一个 changefeed 上是否开启了 Unified Sorter 功能，可执行以下示例命令查看（假设 PD 实例的 IP 地址为 `http://10.0.10.25:2379`）：

```sh
cdc cli --server="http://10.0.10.25:8300" changefeed query --changefeed-id=simple-replication-task | grep 'sort-engine'
```

以上命令的返回结果中，如果 `sort-engine` 的值为 "unified"，则说明 Unified Sorter 已在该 changefeed 上开启。

## 10、任务注意

- 如果服务器使用机械硬盘或其他有延迟或吞吐有瓶颈的存储设备，Unified Sorter 性能会受到较大影响。
- Unified Sorter 默认使用 `data_dir` 储存临时文件。建议保证硬盘的空闲容量大于等于 500 GiB。对于生产环境，建议保证每个节点上的磁盘可用空间大于（业务允许的最大）`checkpoint-ts` 延迟 * 业务高峰上游写入流量。此外，如果在 `changefeed` 创建后预期需要同步大量历史数据，请确保每个节点的空闲容量大于等于要追赶的同步数据。

# 四、同步实例

Changefeed 是 TiCDC 中的单个同步任务。Changefeed 将一个 TiDB 集群中数张表的变更数据输出到一个指定的下游。TiCDC 集群可以运行和管理多个 Changefeed。

## 1、复制增量数据到 MySQL 兼容数据库

```sh
cdc cli changefeed create \
    --server=http://10.0.10.25:8300 \
    --sink-uri="mysql://root:123456@127.0.0.1:3306/" \
    --changefeed-id="simple-replication-task"
Create changefeed successfully!
ID: simple-replication-task
Info: {"sink-uri":"mysql://root:123456@127.0.0.1:3306/","opts":{},"create-time":"2020-03-12T22:04:08.103600025+08:00","start-ts":415241823337054209,"target-ts":0,"admin-job-type":0,"sort-engine":"unified","sort-dir":".","config":{"case-sensitive":true,"filter":{"rules":["*.*"],"ignore-txn-start-ts":null,"ddl-allow-list":null},"mounter":{"worker-num":16},"sink":{"dispatchers":null},"scheduler":{"type":"table-number","polling-time":-1}},"state":"normal","history":null,"error":null}
```

- `--server`：TiCDC 集群中任意一个 TiCDC 服务器的地址。
- `--changefeed-id`：同步任务的 ID，格式需要符合正则表达式 `^[a-zA-Z0-9]+(\-[a-zA-Z0-9]+)*$`。如果不指定该 ID，TiCDC 会自动生成一个 UUID（version 4 格式）作为 ID。
- `--sink-uri`：同步任务下游的地址，详见 [Sink URI 配置 `mysql`/`tidb`](https://docs.pingcap.com/zh/tidb/v6.5/ticdc-sink-to-mysql#sink-uri-配置-mysqltidb)。
- `--start-ts`：指定 changefeed 的开始 TSO。TiCDC 集群将从这个 TSO 开始拉取数据。默认为当前时间。
- `--target-ts`：指定 changefeed 的目标 TSO。TiCDC 集群拉取数据直到这个 TSO 停止。默认为空，即 TiCDC 不会自动停止。
- `--config`：指定 changefeed 配置文件，详见：[TiCDC Changefeed 配置参数](https://docs.pingcap.com/zh/tidb/v6.5/ticdc-changefeed-config)。

Sink URI 用于指定 TiCDC 目标系统的连接信息，遵循以下格式：

```inform7
[scheme]://[userinfo@][host]:[port][/path]?[query_parameters]
```

- 注意`/path` 不适用于 MySQL sink。

一个通用的配置样例如下所示：

```sh
--sink-uri="mysql://root:123456@127.0.0.1:3306"
```

URI 中可配置的参数如下：

| 参数                    | 描述                                                         |
| :---------------------- | :----------------------------------------------------------- |
| `root`                  | 下游数据库的用户名。                                         |
| `123456`                | 下游数据库密码。（可采用 Base64 进行编码）                   |
| `127.0.0.1`             | 下游数据库的 IP。                                            |
| `3306`                  | 下游数据的连接端口。                                         |
| `worker-count`          | 向下游执行 SQL 的并发度（可选，默认值为 `16`）。             |
| `max-txn-row`           | 向下游执行 SQL 的 batch 大小（可选，默认值为 `256`）。       |
| `ssl-ca`                | 连接下游 MySQL 实例所需的 CA 证书文件路径（可选）。          |
| `ssl-cert`              | 连接下游 MySQL 实例所需的证书文件路径（可选）。              |
| `ssl-key`               | 连接下游 MySQL 实例所需的证书密钥文件路径（可选）。          |
| `time-zone`             | 连接下游 MySQL 实例时使用的时区名称，从 v4.0.8 开始生效。（可选。如果不指定该参数，使用 TiCDC 服务进程的时区；如果指定该参数但使用空值，则表示连接 MySQL 时不指定时区，使用下游默认时区）。 |
| `transaction-atomicity` | 指定事务的原子性级别（可选，默认值为 `none`）。当该值为 `table` 时 TiCDC 保证单表事务的原子性，当该值为 `none` 时 TiCDC 会拆分单表事务。 |

若需要对 Sink URI 中的数据库密码使用 Base64 进行编码，可以参考如下命令：

```sh
echo -n '123456' | base64   # 假设待编码的密码为 123456
```

编码后的密码如下：

```sh
MTIzNDU2
```

- 注意当 Sink URI 中包含特殊字符时，如 `! * ' ( ) ; : @ & = + $ , / ? % # [ ]`，需要对 URI 特殊字符进行转义处理。你可以使用 [URI Encoder](https://meyerweb.com/eric/tools/dencoder/) 工具对 URI 进行转义。

## 2、复制增量数据到 Kafka

## 3、复制增量数据到 Pulsar

```bash
cdc cli changefeed create \
    --pd=http://192.168.1.9:2379 \
    --sink-uri="pulsar://192.168.1.9:6050/testticdc?protocol=canal-json&compressionType=LZ4" \
    --changefeed-id="tidb-cdc-pulsar" \
    --config /home/tidb/ticdc-changefeed.toml
    
```


# 五、监控



# 六、故障处理

## 1、如何处理 TiCDC 同步任务的中断？

目前已知可能发生的同步中断包括以下场景：

- 下游持续异常，TiCDC 多次重试后仍然失败。
  - 该场景下 TiCDC 会保存任务信息，由于 TiCDC 已经在 PD 中设置的 service GC safepoint，在 `gc-ttl` 的有效期内，同步任务 checkpoint 之后的数据不会被 TiKV GC 清理掉。
  - 处理方法：在下游恢复正常后，通过 HTTP 接口恢复同步任务。
- 因下游存在不兼容的 SQL 语句，导致同步不能继续。
  - 该场景下 TiCDC 会保存任务信息，由于 TiCDC 已经在 PD 中设置的 service GC safepoint，在 `gc-ttl` 的有效期内，同步任务 checkpoint 之后的数据不会被 TiKV GC 清理掉。
  - 处理方法：
    1. 先通过 `cdc cli changefeed query` 查询同步任务状态信息，记录 `checkpoint-ts` 值。
    2. 使用新的任务配置文件，增加`ignore-txn-start-ts` 参数跳过指定 `start-ts` 对应的事务。
    3. 通过 HTTP API 停止旧的同步任务，使用 `cdc cli changefeed create`，指定新的任务配置文件，指定 `start-ts` 为刚才记录的 `checkpoint-ts`，启动新的同步任务恢复同步。
- 在 v4.0.13 及之前的版本中 TiCDC 同步分区表存在问题，导致同步停止。
  - 该场景下 TiCDC 会保存任务信息，由于 TiCDC 已经在 PD 中设置的 service GC safepoint，在 `gc-ttl` 的有效期内，同步任务 checkpoint 之后的数据不会被 TiKV GC 清理掉。
  - 处理方法：
    1. 通过 `cdc cli changefeed pause -c <changefeed-id>` 暂停同步。
    2. 等待约一分钟后，通过 `cdc cli changefeed resume -c <changefeed-id>` 恢复同步。

## 2、TiCDC 同步时，在下游执行 DDL 语句失败会有什么表现，如何恢复？

如果某条 DDL 语句执行失败，同步任务 (changefeed) 会自动停止，checkpoint-ts 断点时间戳为该条出错 DDL 语句的结束时间戳 (finish-ts) 减去一。如果希望让 TiCDC 在下游重试执行这条 DDL 语句，可以使用 `cdc cli changefeed resume` 恢复同步任务。例如：

```sh
cdc cli changefeed resume -c test-cf --server=http://127.0.0.1:8300
```

如果希望跳过这条出错的 DDL 语句，可以将 changefeed 的 start-ts 设为报错时的 checkpoint-ts 加上一，然后通过 `cdc cli changefeed create` 新建同步任务。假设报错时的 checkpoint-ts 为 `415241823337054209`，可以进行如下操作来跳过该 DDL 语句：

```sh
cdc cli changefeed remove --server=http://127.0.0.1:8300 --changefeed-id simple-replication-task
cdc cli changefeed create --server=http://127.0.0.1:8300 --sink-uri="mysql://root:123456@127.0.0.1:3306/" --changefeed-id="simple-replication-task" --sort-engine="unified" --start-ts 415241823337054210
```

## 3、当 changefeed 的下游为类 MySQL 数据库时，TiCDC 执行了一个耗时较长的 DDL 语句，阻塞了所有其他 changefeed，应该怎样处理？

1. 首先暂停执行耗时较长的 DDL 的 changefeed。此时可以观察到，这个 changefeed 暂停后，其他的 changefeed 不再阻塞了。
2. 在 TiCDC log 中搜寻 `apply job` 字段，确认耗时较长的 DDL 的 `start-ts`。
3. 手动在下游执行该 DDL 语句，执行完毕后进行下面的操作。
4. 修改 changefeed 配置，将上述 `start-ts` 添加到 `ignore-txn-start-ts` 配置项中。
5. 恢复被暂停的 changefeed。

## 4、如何处理 TiCDC 创建同步任务或同步到 MySQL 时遇到 `Error 1298: Unknown or incorrect time zone: 'UTC'` 错误？

这是因为下游 MySQL 没有加载时区，可以通过 [mysql_tzinfo_to_sql](https://dev.mysql.com/doc/refman/8.0/en/mysql-tzinfo-to-sql.html) 命令加载时区，加载后就可以正常创建任务或同步任务。

```sh
mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -u root mysql -p
```

显示类似于下面的输出则表示导入已经成功：

```sh
Enter password:
Warning: Unable to load '/usr/share/zoneinfo/iso3166.tab' as time zone. Skipping it.
Warning: Unable to load '/usr/share/zoneinfo/leap-seconds.list' as time zone. Skipping it.
Warning: Unable to load '/usr/share/zoneinfo/zone.tab' as time zone. Skipping it.
Warning: Unable to load '/usr/share/zoneinfo/zone1970.tab' as time zone. Skipping it.
```

如果下游是特殊的 MySQL 环境（某种公有云 RDS 或某些 MySQL 衍生版本等），使用上述方式导入时区失败，就需要通过 sink-uri 中的 `time-zone` 参数指定下游的 MySQL 时区。可以首先在 MySQL 中查询其使用的时区：

```sh
show variables like '%time_zone%';
+------------------+--------+
| Variable_name    | Value  |
+------------------+--------+
| system_time_zone | CST    |
| time_zone        | SYSTEM |
+------------------+--------+
```

然后在创建同步任务和创建 TiCDC 服务时使用该时区：

```sh
cdc cli changefeed create --sink-uri="mysql://root@127.0.0.1:3306/?time-zone=CST" --server=http://127.0.0.1:8300
```

注意

CST 可能是以下四个不同时区的缩写：

- 美国中部时间：Central Standard Time (USA) UT-6:00
- 澳大利亚中部时间：Central Standard Time (Australia) UT+9:30
- 中国标准时间：China Standard Time UT+8:00
- 古巴标准时间：Cuba Standard Time UT-4:00

在中国，CST 通常表示中国标准时间，使用时请注意甄别。

# 参考：

- https://docs.pingcap.com/zh/tidb/v6.1/manage-ticdc#%E8%87%AA%E5%AE%9A%E4%B9%89-kafka-sink-%E7%9A%84-topic-%E5%92%8C-partition-%E7%9A%84%E5%88%86%E5%8F%91%E8%A7%84%E5%88%99