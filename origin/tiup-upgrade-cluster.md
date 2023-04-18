# TiUP升级TIDB集群

# 一、简介

文档：https://docs.pingcap.com/zh/tidb/stable/upgrade-tidb-using-tiup

## 升级兼容性说明

- TiDB 目前暂不支持版本降级或升级后回退。
- 使用 TiDB Ansible 管理的 4.0 版本集群，需要先按照 [4.0 版本文档的说明](https://docs.pingcap.com/zh/tidb/v4.0/upgrade-tidb-using-tiup)将集群导入到 TiUP (`tiup cluster`) 管理后，再按本文档说明升级到 6.1.0 版本及后续修订版本。
- 若要将 3.0 之前的版本升级至 6.1.0 版本：
  1. 首先[通过 TiDB Ansible 升级到 3.0 版本](https://docs.pingcap.com/zh/tidb/v3.0/upgrade-tidb-using-ansible)。
  2. 然后按照 [4.0 版本文档的说明](https://docs.pingcap.com/zh/tidb/v4.0/upgrade-tidb-using-tiup)，使用 TiUP (`tiup cluster`) 将 TiDB Ansible 配置导入。
  3. 将集群升级至 4.0 版本。
  4. 按本文档说明将集群升级到 6.1.0 版本。
- 支持 TiDB Binlog，TiCDC，TiFlash 等组件版本的升级。
- 具体不同版本的兼容性说明，请查看各个版本的 [Release Note](https://docs.pingcap.com/zh/tidb/stable/release-notes)。请根据各个版本的 Release Note 的兼容性更改调整集群的配置。
- 升级 v5.3 之前版本的集群到 v5.3 及后续版本时，默认部署的 Prometheus 会从 v2.8.1 升级到 v2.27.1，v2.27.1 提供更多的功能并解决了安全风险。Prometheus v2.27.1 相对于 v2.8.1 存在 Alert 时间格式变化，详情见 [Prometheus commit](https://github.com/prometheus/prometheus/commit/7646cbca328278585be15fa615e22f2a50b47d06)。

## 适用升级路径

- 从 TiDB 4.0 版本升级至 TiDB 6.1 及后续修订版本
- 从 TiDB 5.0-5.4 版本升级至 TiDB 6.1 及后续修订版本
- 从 TiDB 6.0 版本升级至 TiDB 6.1 及后续修订版本

## 升级方式

- **不停机升级**

  默认的升级 方式，即升级过程中集群仍然可以对外提供服务。升级时会对各节点逐个迁移 leader 后再升级和重启，因此对于大规模集群需要较长时间才能完成整个升级操作。

  - 滚动升级会逐个升级所有的组件。升级 TiKV 期间，会逐个将 TiKV 上的所有 leader 切走再停止该 TiKV 实例。默认超时时间为 5 分钟（300 秒），超时后会直接停止该实例。
  - 使用 `--force` 参数可以在不驱逐 leader 的前提下快速升级集群至新版本，但是该方式会忽略所有升级中的错误，在升级失败后得不到有效提示，请谨慎使用。
  - 如果希望保持性能稳定，则需要保证 TiKV 上的所有 leader 驱逐完成后再停止该 TiKV 实例，可以指定 `--transfer-timeout` 为一个更大的值，如 `--transfer-timeout 3600`，单位为秒。
  - 若想将 TiFlash 从 5.3 之前的版本升级到 5.3 及之后的版本，必须进行 TiFlash 的停机升级。参考如下步骤，可以在确保其他组件正常运行的情况下升级 TiFlash：
    1. 关闭 TiFlash 实例：`tiup cluster stop <cluster-name> -R tiflash`
    2. 使用 `--offline` 参数在不重启（只更新文件）的情况下升级集群：`tiup cluster upgrade <cluster-name> <version> --offline`
    3. reload 整个集群：`tiup cluster reload <cluster-name>`。此时，TiFlash 也会正常启动，无需额外操作。

- **停机升级**

  如果业务有维护窗口可供数据库停机维护，则可以使用停机升级的方式快速进行升级操作。

# 二、升级前准备

以TiDB 5.1.0 升级至 6.1.1为例

- tiup版本：1.52

## 1、备份数据和TiUP

- 备份TiDB数据

  - 使用BR工具对TIDB数据物理文件进行备份，详情参考文档：[TiDB-BR分布式冷备份与恢复](tidb-br-backup-restore.md)
  - 使用dumpling工具对TiDB数据进行逻辑备份，详情参考：[TiDB-Dumpling：从TiDB/MySQL导出数据](tidb-dumpling-export.md)

- 备份TiUP配置

  ```bash
  cp -r .tiup .tiup5-bak20221029
  ```

## 2、升级 TiUP 和 TiUP Cluster

- 升级 TiUP 和TiUP Cluster 版本（建议 `tiup` 和`tiup cluster` 版本不低于 `1.10.0`）

  ```bash
  # 设置TiUP的软件镜像源
  tiup mirror show
  tiup mirror set https://tiup-mirrors.pingcap.com
  tiup mirror show
  
  # 查看TiUP和TiUP Cluster版本
  tiup --version
  tiup cluster --version
  
  # 升级 TiUP 和 TiUP Cluster
  tiup update --all
  
  # 查看TiUP和TiUP Cluster版本
  tiup --version
  tiup cluster --version
  # 查看TiUP和TiUP Cluster支持升级组件的版本
  tiup cluster list
  tiup list
  tiup list tidb
  ```

# 三、升级步骤

## 1、编辑 TiUP Cluster 拓扑配置文件

```sh
tiup cluster edit-config <cluster-name>
```

**注意：**

- 升级到 6.1.0 版本前，请确认已在 4.0 修改的参数在 6.1.0 版本中是兼容的，可参考 [TiKV 配置文件描述](https://docs.pingcap.com/zh/tidb/stable/tikv-configuration-file)。

- 以下 TiKV 参数在 TiDB v5.0 已废弃。如果在原集群配置过以下参数，需要通过 `edit-config` 编辑模式删除这些参数：

  - pessimistic-txn.enabled

  - server.request-batch-enable-cross-command

  - server.request-batch-wait-duration

## 2、检查当前集群的健康状况

在升级前对集群当前的 region 健康状态进行检查

```sh
tiup cluster check <cluster-name> --cluster
```

执行结束后，最后会输出 region status 检查结果。

- 如果结果为 "All regions are healthy"，则说明当前集群中所有 region 均为健康状态，可以继续执行升级；
- 如果结果为 "`Regions are not fully healthy: m miss-peer, n pending-peer" 并提示 "Please fix unhealthy regions before other operations`."，则说明当前集群中有 region 处在异常状态，应先排除相应异常状态，并再次检查结果为 "All regions are healthy" 后再继续升级。

## 3、不停机升级 TiDB 集群

```sh
tiup cluster upgrade <cluster-name> <version>

# tiup cluster upgrade test-tidb v6.1.1
```

## 4、升级后验证

- **集群验证**

  ```bash
  tiup cluster display <cluster-name>
  
  # Cluster type:       tidb
  # Cluster name:       <cluster-name>
  # Cluster version:    v6.1.1
  ```

- **Dashboard验证**

  ```bash
  tiup cluster display <cluster-name> --dashboard
  ```

- **客户端登录操作**

  - Navicat 进行连接，执行数据操作

- **数据一致性验证**

  - 验证重要表的条数是否一致

- **dm同步任务**

  - 查看用于同步数据到TiDB的DM任务状态
  - 验证DM数据同步效果

- **监控告警**

  - 查看Grafana监控是否正常

- **业务验证**

  - 检查使用TiDB的应用是否有重连机制
  - 验证业务受到影响的应用在升级完成后是否正常

# 三、升级报错处理

## 1、升级时报错中断，处理完报错后，如何继续升级

重新执行 `tiup cluster upgrade` 命令进行升级，升级操作会重启之前已经升级完成的节点。如果不希望重启已经升级过的节点，可以使用 `replay` 子命令来重试操作，具体方法如下：

1. 使用 `tiup cluster audit` 命令查看操作记录：

   ```sh
   tiup cluster audit
   ```

   在其中找到失败的升级操作记录，并记下该操作记录的 ID，下一步中将使用 `<audit-id>` 表示操作记录 ID 的值。

2. 使用 `tiup cluster replay <audit-id>` 命令重试对应操作：

   ```sh
   tiup cluster replay <audit-id>
   ```

## 2、 升级过程中 evict leader 等待时间过长，如何跳过该步骤快速升级

可以指定 `--force`，升级时会跳过 `PD transfer leader` 和 `TiKV evict leader` 过程，直接重启并升级版本，对线上运行的集群性能影响较大。命令如下：

```sh
tiup cluster upgrade <cluster-name> <version> --force
```

## 3、升级完成后更新 pd-ctl 等周边工具版本

可通过 TiUP 安装对应版本的 `ctl` 组件来更新相关工具版本：

```bash
tiup install ctl:v6.1.0
```

## 4、Dashboard显示集群中未启动必要组件NgMonitoring

NgMonitoring 是 TiDB v5.4.0 及以上集群中内置的高级监控组件，用于支撑 TiDB Dashboard 的 持续性能分析 和 Top SQL 等功能。使用较新版本 TiUP 部署或升级集群时，NgMonitoring 会自动部署；使用 TiDB Operator 部署集群时，需要依据启用持续性能分析手动部署 NgMonitoring。

①检查 TiUP Cluster 版本，NgMonitoring 组件需要较高版本的部署工具支持（TiUP v1.9.0 及以上）：

```sh
tiup cluster --version

# 如果 TiUP 版本低于 v1.9.0，升级 TiUP 和 TiUP Cluster 版本至最新。
tiup update --all
```

②在中控机上，通过 TiUP 添加 ng_port 配置项，然后重启 Prometheus 节点。

- 以编辑模式打开集群的配置文件：

```sh
tiup cluster edit-config ${cluster-name}
```

- 在 `monitoring_servers` 下面增加 `ng_port:12020` 参数：

```bash
monitoring_servers:
- host: 172.16.6.6
  ng_port: 12020
```

- 重启 Prometheus 节点：

```sh
tiup cluster reload ${cluster-name} --role prometheus -N 192.168.1.7:9000
```
