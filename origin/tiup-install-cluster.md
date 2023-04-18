

# 使用TIUP部署TiDB 4.0集群

# 一、简介

[TiUP](https://github.com/pingcap/tiup) 是 TiDB 4.0 版本引入的集群运维工具，[TiUP cluster](https://github.com/pingcap/tiup/tree/master/components/cluster) 是 TiUP 提供的使用 Golang 编写的集群管理组件，通过 TiUP cluster 组件就可以进行日常的运维工作，包括部署、启动、关闭、销毁、弹性扩缩容、升级 TiDB 集群；管理 TiDB 集群参数。

目前 TiUP 可以支持部署 TiDB、TiFlash、TiDB Binlog、TiCDC，以及监控系统。

## Linux 操作系统版本要求

| Linux 操作系统平台       | 版本         |
| ------------------------ | ------------ |
| Red Hat Enterprise Linux | 7.3 及以上   |
| CentOS                   | 7.3 及以上   |
| Oracle Enterprise Linux  | 7.3 及以上   |
| Ubuntu LTS               | 16.04 及以上 |

## TiDB各组件端口

| 组件              | 默认端口 | 说明                                               |
| ----------------- | -------- | -------------------------------------------------- |
| TiDB              | 4000     | 应用及 DBA 工具访问通信端口                        |
| TiDB              | 10080    | TiDB 状态信息上报通信端口                          |
| TiKV              | 20160    | TiKV 通信端口                                      |
| TiKV              | 20180    | TiKV 状态信息上报通信端口                          |
| PD                | 2379     | 提供 TiDB 和 PD 通信端口                           |
| PD                | 2380     | PD 集群节点间通信端口                              |
| TiFlash           | 9000     | TiFlash TCP 服务端口                               |
| TiFlash           | 8123     | TiFlash HTTP 服务端口                              |
| TiFlash           | 3930     | TiFlash RAFT 服务和 Coprocessor 服务端口           |
| TiFlash           | 20170    | TiFlash Proxy 服务端口                             |
| TiFlash           | 20292    | Prometheus 拉取 TiFlash Proxy metrics 端口         |
| TiFlash           | 8234     | Prometheus 拉取 TiFlash metrics 端口               |
| Pump              | 8250     | Pump 通信端口                                      |
| Drainer           | 8249     | Drainer 通信端口                                   |
| CDC               | 8300     | CDC 通信接口                                       |
| Prometheus        | 9090     | Prometheus 服务通信端口                            |
| Node_exporter     | 9100     | TiDB 集群每个节点的系统信息上报通信端口            |
| Blackbox_exporter | 9115     | Blackbox_exporter 通信端口，用于 TiDB 集群端口监控 |
| Grafana           | 3000     | Web 监控服务对外服务和客户端(浏览器)访问端口       |
| Alertmanager      | 9093     | 告警 web 服务端口                                  |
| Alertmanager      | 9094     | 告警通信端口                                       |

# 二、集群节点准备

## 0、集群组件规划

| 集群角色/集群节点 | tools.tidb4.curiouser.com | node1.tidb4.curiouser.com | node2.tidb4.curiouser.com | node3.tidb4.curiouser.com |
| ----------------- | :-----------------------: | :-----------------------: | :-----------------------: | :-----------------------: |
| TiUP              |             ✓             |                           |                           |                           |
| TiDP              |                           |             ✓             |             ✓             |             ✓             |
| PD Server         |                           |             ✓             |             ✓             |             ✓             |
| TiKV              |                           |             ✓             |             ✓             |             ✓             |
| TiFlash           |             ✓             |                           |                           |                           |
| CDC               |             ✓             |                           |                           |                           |
| pump              |             ✓             |                           |                           |                           |
| grafana           |             ✓             |                           |                           |                           |
| prometheus        |             ✓             |                           |                           |                           |
| alertmanager      |             ✓             |                           |                           |                           |
| Zookeeper         |             ✓             |                           |                           |                           |
| Kafka             |             ✓             |                           |                           |                           |

## 1、节点准备

### ①所有节点创建tidb用户并加入sudoer

```bash
useradd -m tidb
echo "tidb ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
passwd tidb
```

### ②所有节点系统优化

```bash
# 关闭系统 swap
echo "vm.swappiness = 0">> /etc/sysctl.conf
swapoff -a && swapon -a

# 关闭透明大页
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/defrag
sysctl -p

# 配置网络参数
echo "fs.file-max = 1000000">> /etc/sysctl.conf
echo "net.core.somaxconn = 32768">> /etc/sysctl.conf
echo "net.ipv4.tcp_tw_recycle = 0">> /etc/sysctl.conf
echo "net.ipv4.tcp_syncookies = 0">> /etc/sysctl.conf
echo "vm.overcommit_memory = 1">> /etc/sysctl.conf
sysctl -p

# 配置tidb用户的打开文件句柄限制
cat << EOF >>/etc/security/limits.conf
tidb           soft    nofile          1000000
tidb           hard    nofile          1000000
tidb           soft    stack          32768
tidb           hard    stack          32768
EOF
```

### ③打通tools节点的root用户到所有节点tidb用户的免秘钥登录

```bash
ssh-keygen -t rsa -P "" -f ~/.ssh/id_rsa
ssh-copy-id -i tidb@tools.tidb4.curiouser.com
ssh-copy-id -i tidb@node1.tidb4.curiouser.com
ssh-copy-id -i tidb@node2.tidb4.curiouser.com
ssh-copy-id -i tidb@node3.tidb4.curiouser.com
```

### ④所有节点安装`numactl`

在生产环境中，因为硬件机器配置往往高于需求，为了更合理规划资源，会考虑单机多实例部署 TiDB 或者 TiKV。NUMA 绑核工具的使用，主要为了防止 CPU 资源的争抢，引发性能衰退。NUMA 绑核是用来隔离 CPU 资源的一种方法，适合高配置物理机环境部署多实例使用。

```bash
yum install -y numactl
```

### ⑤所有节点路径挂载参数

`mount point / does not have 'nodelalloc' option set和 mount point / does not have 'noatime' option set`

`/etc/fstab`

```bash
UUID=59d9ca7b-4f39-4c0c-9334-c561234576b5 /  ext4    defaults,nodelalloc,noatime        1 1
```

# 三、集群安装

以下所有命令都在tools节点，tidb用户，bash环境下执行（zsh环境不支持tiup的某些脚本）

## 1、安装tiup命令

```shell
curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
source .bash_profile
tiup cluster
tiup --binary cluster

# 添加tiup命令自动补全功能
tiup completion zsh > "${fpath[1]}/_tiup"
```

## 2、下载离线安装包

```bash
tidb_version=5.1.0
wget https://download.pingcap.org/tidb-community-server-v$tidb_version-linux-amd64.tar.gz
wget https://download.pingcap.org/tidb-community-server-v6.1.1-linux-amd64.tar.gz
tiup mirror clone /opt/tidb-community-server-v$tidb_version-linux-amd64.tar.gz ${tidb_version} --os=linux --arch=amd64
```

## 3、查看目前tiup命令支持安装的tidb版本

```bash
tiup list tidb
```

## 4、创建集群组件的拓扑配置文件

```yaml
# # Global variables are applied to all deployments and used as the default value of
# # the deployments if a specific deployment value is missing.
global:
  user: "tidb"
  ssh_port: 22
  arch: "amd64"

server_configs:
  tidb:
    oom-action: "cancel"
    mem-quota-query: 25769803776
    log.query-log-max-len: 4096
    log.file.log-rotate: true
    log.file.max-size: 300
    log.file.max-days: 7
    log.file.max-backups: 14
    log.slow-threshold: 300
    binlog.enable: true
    binlog.ignore-error: true
  tikv:
    global.log-rotation-timespan: "168h"
    readpool.unified.max-thread-count: 12
    readpool.storage.use-unified-pool: false
    readpool.coprocessor.use-unified-pool: true
    coprocessor.split-region-on-table: false
    storage.block-cache.capacity: "16GB"
    raftstore.capacity: "10GB"
  pd:
    log.file.max-size: 300
    log.file.max-days: 28
    log.file.max-backups: 14
    log.file.log-rotate: true
    replication.location-labels: ["host"]
    schedule.leader-schedule-limit: 4
    schedule.region-schedule-limit: 2048
    schedule.replica-schedule-limit: 64
  tiflash:
    logger.level: "info"
    # Maximum memory usage for processing a single query. Zero means unlimited.
    profiles.default.max_memory_usage: 10000000000
    # Maximum memory usage for processing all concurrently running queries on the server. Zero means unlimited.
    profiles.default.max_memory_usage_for_all_queries: 0

pd_servers:
  - host: 192.168.1.71
    ssh_port: 22
    name: pd-1
    client_port: 2379
    peer_port: 2380
    deploy_dir: /data/tidb/pd-2379
    data_dir: /data/tidb/pd-2379/data
    log_dir: /data/tidb/pd-2379/logs
  - host: 192.168.1.72
    ssh_port: 22
    name: pd-2
    client_port: 2379
    peer_port: 2380
    deploy_dir: /data/tidb/pd-2379
    data_dir: /data/tidb/pd-2379/data
    log_dir: /data/tidb/pd-2379/logs
  - host: 192.168.1.73
    ssh_port: 22
    name: pd-3
    client_port: 2379
    peer_port: 2380
    deploy_dir: /data/tidb/pd-2379
    data_dir: /data/tidb/pd-2379/data
    log_dir: /data/tidb/pd-2379/logs

tidb_servers:
  - host: 192.168.1.71
    port: 4000
    status_port: 10080
    deploy_dir: "/data/tidb/tidb-4000"
    log_dir: "/data/tidb/tidb-4000/logs"
    numa_node: "0"
  - host: 192.168.1.72
    port: 4000
    status_port: 10080
    deploy_dir: "/data/tidb/tidb-4000"
    log_dir: "/data/tidb/tidb-4000/logs"
    numa_node: "0"
  - host: 192.168.1.73
    port: 4000
    status_port: 10080
    deploy_dir: "/data/tidb/tidb-4000"
    log_dir: "/data/tidb/tidb-4000/logs"
    numa_node: "0"

tikv_servers:
  - host: 192.168.1.71
    port: 20160
    status_port: 20180
    deploy_dir: "/data/tidb/tikv-20160"
    data_dir: "/data/tidb/tikv-20160/data"
    log_dir: "/data/tidb/tikv-20160/logs"
    numa_node: "0"
    config:
      server.labels: { host: "tikv1" }
  - host: 192.168.1.72
    port: 20160
    status_port: 20180
    deploy_dir: "/data/tidb/tikv-20160"
    data_dir: "/data/tidb/tikv-20160/data"
    log_dir: "/data/tidb/tikv-20160/logs"
    numa_node: "0"
    config:
      server.labels: { host: "tikv2" }
  - host: 192.168.1.73
    port: 20160
    status_port: 20180
    deploy_dir: "/data/tidb/tikv-20160"
    data_dir: "/data/tidb/tikv-20160/data"
    log_dir: "/data/tidb/tikv-20160/logs"
    numa_node: "0"
    config:
      server.labels: { host: "tikv3" }
tiflash_servers:
  - host: 192.168.1.70
    data_dir: "/data/tidb/tiflash-9000/data"
    deploy_dir: "/data/tidb/tiflash-9000"
    log_dir: "/data/tidb/tiflash-9000/logs"
    ssh_port: 22
    tcp_port: 9000
    http_port: 8123
    flash_service_port: 3930
    flash_proxy_port: 20170
    flash_proxy_status_port: 20292
    metrics_port: 8234
    numa_node: "0"
    # The following configs are used to overwrite the `server_configs.tiflash` values.
    config:
      logger.level: "info"
    learner_config:
      log-level: "info"
pump_servers:
  - host: 192.168.1.70
    ssh_port: 22
    port: 8250
    deploy_dir: "/data/tidb/pump-8249"
    data_dir: "/data/tidb/pump-8249/data"
    log_dir: "/data/tidb/pump-8249/logs"
    numa_node: "0"
    # The following configs are used to overwrite the `server_configs.drainer` values.
    config:
      gc: 7
tispark_masters:
  - host: 192.168.1.70
    ssh_port: 22
    port: 7077
    web_port: 8080
    deploy_dir: "/data/tidb/tispark-master-7077"
    java_home: "/opt/jdk"
    spark_config:
      spark.driver.memory: "2g"
      spark.eventLog.enabled: "True"
      spark.tispark.grpc.framesize: 268435456
      spark.tispark.grpc.timeout_in_sec: 100
      spark.tispark.meta.reload_period_in_sec: 60
      spark.tispark.request.command.priority: "Low"
      spark.tispark.table.scan_concurrency: 256
    spark_env:
      SPARK_EXECUTOR_CORES: 5
      SPARK_EXECUTOR_MEMORY: "10g"
      SPARK_WORKER_CORES: 5
      SPARK_WORKER_MEMORY: "10g"

# NOTE: multiple worker nodes on the same host is not supported by Spark
tispark_workers:
  - host: 192.168.1.70
    ssh_port: 22
    port: 7078
    web_port: 8081
    deploy_dir: "/data/tidb/tispark-worker-7078"
    java_home: "/opt/jdk"

cdc_servers:
  - host: 192.168.1.70
    ssh_port: 22
    port: 8300
    deploy_dir: "/data/tidb/cdc-8300"
    log_dir: "/data/tidb/cdc-8300/logs"

monitored:
  node_exporter_port: 9100
  blackbox_exporter_port: 9115
  deploy_dir: "/data/tidb/monitored-9100"
  data_dir: "/data/tidb/monitored-9100/data"
  log_dir: "/data/tidb/monitored-9100/logs"

monitoring_servers:
  - host: 192.168.1.70
    ssh_port: 22
    port: 9090
    deploy_dir: "/data/tidb/prometheus-9090"
    data_dir: "/data/tidb/prometheus-9090/data"
    log_dir: "/data/tidb/prometheus-9090/logs"

grafana_servers:
  - host: 192.168.1.70
    ssh_port: 22
    port: 3000
    deploy_dir: "/data/tidb/grafana-3000"

# alertmanager_servers:
#   - host: 192.168.1.70
#     ssh_port: 22
#     web_port: 9093
#     cluster_port: 9094
#     deploy_dir: "/data/tidb/alertmanager-9093"
#     data_dir: "/data/tidb/alertmanager-9093/data"
#     log_dir: "/data/tidb/alertmanager-9093/logs"
```

## 5、检查集群拓扑文件及机器系统配置

```bash
tiup cluster check <集群拓扑文件路径 | 集群名> [参数]

# 若集群尚未部署，需要传递将用于部署集群的topology.yml文件，tiup-cluster 会根据该文件的内容连接到对应机器去检查。若集群已经部署，则可以使用集群的名字 `<cluster-name>` 作为检查对象。若传递的是集群名字，则需要配合 `--cluster` 选项使用。

--apply BOOLEAN  
#				尝试自动修复失败的检查项。该选项默认关闭，默认值为 `false`。在命令中添加该选项，并传入 `true` 值或不传值，均可开启此功能目前仅会尝试修复以下项目：
#              - SELinux
#              - 防火墙
#              - irqbalance
#              - 内核参数
#              - 系统 Limits
#              - THP（透明大页）
--cluster  BOOLEAN 
#				支持对未部署的集群进行检查，也支持对已部署的集群进行检查。若选择的格式为 `tiup cluster check <cluster-name>` 则必须加上该选项：`tiup cluster check <cluster-name> --cluster`。该选项默认关闭，默认值为 `false`。在命令中添加该选项，并传入 `true` 值或不传值，均可开启此功能。

-N, --node STRINGS 
# 			指定要检查的节点。该选项的值为以逗号分割的节点 ID 列表，节点ID为tiup-component-cluster-display 命令返回的集群状态表格的第一列。如果不指定该选项，默认检查所有节点，即 `[]`。注意：若同时指定了 `-R, --role`，那么将检查它们的交集中的服务。
-R, --role STRINGS
# 			指定要检查的角色。该选项的值为以逗号分割的节点角色列表，角色为tiup-component-cluster-display命令返回的集群状态表格的第二列。如果不指定该选项，默认检查所有角色。注意：若同时指定了 `-N, --node`，那么将检查它们的交集中的服务。
--enable-cpu BOOLEAN
# 			默认情况下 tiup-cluster 不检查 CPU 核心数，该选项用于启用 CPU 核心数检查。 该选项默认关闭，默认值为 `false`。在命令中添加该选项，并传入 `true` 值或不传值，均可开启此功能。
--enable-disk BOOLEAN
# 			默认情况下 tiup-cluster 不进行 fio 磁盘性能测试，该选项用于启用 fio 磁盘性能测试。该选项默认关闭，默认值为 `false`。在命令中添加该选项，并传入 `true` 值或不传值，均可开启此功能。
--enable-mem
#				默认情况下 tiup-cluster 不检查内存大小，该选项用于启用内存大小检查。
-u, --user string 
#				指定连接目标机器的用户名，该用户在目标机器上需要有免密 sudo root 的权限。默认为当前执行命令的用户。注意：仅当--cluster选项为 false 时该选项有效，否则该值固定为部署集群时拓扑文件中指定的用户名。
-i, --identity_file  string，
#				指定连接目标机器的密钥文件。默认 ~/.ssh/id_rsa。注意：仅当 `--cluster` 选项为 false 时该选项有效，否则该值固定为 `${TIUP_HOME}/storage/cluster/clusters/<cluster-name>/ssh/id_rsa`
-p, --password BOOLEAN
#				在连接目标机器时使用密码登录，该选项默认关闭，默认值为 `false`。在命令中添加该选项，并传入 `true` 值或不传值，均可开启此功能，：
#	 对于指定了 `--cluster` 的集群，密码为部署集群时拓扑文件中指定的用户的密码
#	对于未指定 `--cluster` 的集群，密码为 `-u/--user` 参数指定的用户的密码
```

该命令同时也会检测集群的机器硬件和软件环境是否满足正常运行条件。

| 检查项                 | 检查信息                                                     |
| ---------------------- | ------------------------------------------------------------ |
| **操作系统版本**       | 检查部署机操作系统发行版和版本：目前仅支持部署在 CentOS 7 的操作系统上，之后随兼容性改进可能支持更多系统版本。 |
| **CPU EPOLLEXCLUSIVE** | 检查部署机 CPU 是否支持 EPOLLEXCLUSIVE。                     |
| **numactl**            | 检查部署机是否安装 numactl，若用户配置绑核，则必须安装 numactl。 |
| **系统时间**           | 检查部署机系统时间是否同步：将部署机系统时间与中控机对比，偏差超出某一阈值（500ms）后报错。 |
| **系统时区**           | 检查部署机系统时区是否同步：将部署机系统的时区配置进行对比，如果时区不一致则报错。 |
| **时间同步服务**       | 检查部署机是否配置了时间同步服务：即 ntpd 是否在运行         |
| **Swap 分区**          | 检查部署机是否启用 Swap 分区：建议禁用 Swap 分区             |
| **THP（透明大页）**    | 检查部署机是否启用透明大页：建议禁用透明大页。               |
| **SELinux**            | 检查 SELinux 是否启用：建议用户禁用 SELinux。                |
| **防火墙**             | 检查 FirewallD 服务是否启用：建议用户禁用 FirewallD 或为 TiDB 集群各服务添加允许规则。 |
| **irqbalance**         | 检查 irqbalance 服务是否启用：建议用户启用 irqbalance 服务。 |
| **磁盘挂载参数**       | 检查 ext4 分区的挂载参数：确保挂载参数包含 nodelalloc,noatime 选项。 |
| **端口占用**           | 检查部署机上是否已有进程占用了端口：检查拓扑中定义的端口（包括自动补全的默认端口）在部署机上是否已被占用。 |
| **CPU 核心数**         | 检查部署机 CPU 信息：建议生产集群 CPU 逻辑核心数 >= 16       |
| **内存大小**           | 检查部署机的内存大小：建议生产集群总内存容量 >= 32Gb。       |
| **fio 磁盘性能测试**   | 使用 fio 测试 data_dir 所在磁盘的性能，包括三个测试项目：  <br/>fio_randread_write_latency <br/>fio_randread_write <br/>fio_randread<br/>默认不进行 fio 磁盘性能测试，需要通过选项 `--enable-disk` 启用。 |
| **内核参数**           | 检查各项内核参数的值： <br>     net.ipv4.tcp_tw_recycle: 0 <br/>     net.ipv4.tcp_syncookies: 0 <br/>     net.core.somaxconn: 32768 <br/>     vm.swappiness: 0 <br/>     vm.overcommit_memory: 0 或 1 <br/>     fs.file-max: 1000000 |
| **系统限制**           | 检查 /etc/security/limits.conf 中各项 limit 值：<br/>deploy-user    soft   nofile    1000000<br/>deploy-user    hard   nofile    1000000<br/>deploy-user    soft   stack     10240<br/>其中 deploy-user为部署、运行 TiDB 集群的用户，最后一列的数值为要求达到的最小值。 |

**检查输出含有以下信息：**

- **Node**：目标节点
- **Check**：检查项
- **Result**：检查结果（Pass | Warn | Fail）
- **Message**：结果描述

## 6、分发组件文件到各节点

```bash
tiup cluster deploy 集群名字 tidb版本 ./集群拓扑配置.yaml --user tidb -y
```

## 7、启动当前集群各组件

```bash
tiup cluster start 集群名字
tiup cluster start 集群名字 -R 组件1,组件2
```

启动集群操作会按 `PD -> TiKV -> Pump -> TiDB -> TiFlash -> Drainer` 的顺序启动整个 TiDB 集群所有组件（同时也会启动监控组件）

## 8、验证

①查看当前集群的状态

```bash
tiup cluster display 集群名字
```

②查看哪个 PD 节点提供了 TiDB Dashboard 服务

```bash
tiup cluster display 集群名字 --dashboard
```

## 9、其他信息

### ①Grafana默认用户名密码

`admin/admin`

### ②TiDB Dashboard默认用户名密码

`root 密码为空`

### ③组件拓扑配置文件样例

- 全配置参数模版：https://github.com/pingcap/tiup/blob/master/examples/topology.example.yaml
- TiDB：https://github.com/pingcap/tidb/blob/master/config/config.toml.example
- PD：https://github.com/tikv/pd/blob/v4.0.0-rc/conf/config.toml
- TiKV：https://github.com/tikv/tikv/blob/v4.0.0-rc/etc/config-template.toml

### ④tiup启动集群操作顺序

按 `PD -> TiKV -> Pump -> TiDB -> TiFlash -> Drainer -> TiCDC -> Prometheus -> Grafana -> Alertmanager` 的顺序启动整个 TiDB 集群所有组件

# 四、集群其他操作

## 1、修改集群组件配置参数

集群运行过程中，如果需要调整某个组件的参数，可以使用 `edit-config` 命令来编辑参数。具体的操作步骤如下：

1. 以编辑模式打开该集群的配置文件：

   ```bash
   tiup cluster edit-config ${cluster-name}
   ```

2. 设置参数：

   首先确定配置的生效范围，有以下两种生效范围：

   - 如果配置的生效范围为该组件全局，则配置到 `server_configs`。例如：

     ```yaml
     server_configs:
       tidb:
         log.slow-threshold: 300
     ```

   - 如果配置的生效范围为某个节点，则配置到具体节点的 `config` 中。例如：

     ```yaml
     tidb_servers:
     - host: 10.0.1.11
         port: 4000
         config:
             log.slow-threshold: 300
     ```

   参数的格式参考 [TiUP 配置参数模版](https://github.com/pingcap/tiup/blob/master/examples/topology.example.yaml)。**配置项层次结构使用 `.` 表示**。

3. 执行 `reload` 命令滚动分发配置、重启相应组件：

   ```bash
   tiup cluster reload ${cluster-name} [-N <nodes>] [-R <roles>]
   ```

**示例**：如果要调整 tidb-server 中事务大小限制参数 `txn-total-size-limit` 为 `1G`，该参数位于 [performance](https://github.com/pingcap/tidb/blob/v4.0.0-rc/config/config.toml.example) 模块下，调整后的配置如下：

```yaml
server_configs:
  tidb:
    performance.txn-total-size-limit: 1073741824
```

然后执行 `tiup cluster reload ${cluster-name} -R tidb` 命令滚动重启。

## 2、集群组件节点重启

`tiup cluster restart` 用于重启指定集群的所有或部分服务。

```bash
tiup cluster restart <cluster-name> [参数]

参数
-N, --node（strings，默认为 []，表示所有节点）。该选项的值为以逗号分割的节点 ID 列表，节点 ID 为集群状态表格的第一列。
-R, --role（strings，默认为 []，表示所有角色）。该选项的值为以逗号分割的节点角色列表，角色为集群状态表格的第二列。

tiup restart test-tidb -R  tidb -N 127.0.0.1:4000
```

## 3、重命名集群

部署并启动集群后，可以通过 `tiup cluster rename` 命令来对集群重命名：

```bash
tiup cluster rename ${cluster-name} ${new-name}
```

> **注意：**
>
> - 重命名集群会重启监控（Prometheus 和 Grafana）。
> - 重命名集群之后 Grafana 可能会残留一些旧集群名的面板，需要手动删除这些面板。

## 4、关闭集群

关闭集群操作会按 Drainer -> TiFlash -> TiDB -> Pump -> TiKV -> PD 的顺序关闭整个 TiDB 集群所有组件（同时也会关闭监控组件）：

```bash
tiup cluster stop ${cluster-name}
```

和 `start` 命令类似，`stop` 命令也支持通过 `-R` 和 `-N` 参数来只停止部分组件。

例如，下列命令只停止 TiDB 组件：

```bash
tiup cluster stop ${cluster-name} -R tidb
```

下列命令只停止 `1.2.3.4` 和 `1.2.3.5` 这两台机器上的 TiDB 组件：

```bash
tiup cluster stop ${cluster-name} -R tidb -N 1.2.3.4:4000,1.2.3.5:4000
```

## 5、清除集群数据

此操作会关闭所有服务，并清空其数据目录或/和日志目录，并且无法恢复，需要**谨慎操作**。

清空集群所有服务的数据，但保留日志：

```bash
tiup cluster clean ${cluster-name} --data
```

清空集群所有服务的日志，但保留数据：

```bash
tiup cluster clean ${cluster-name} --log
```

清空集群所有服务的数据和日志：

```bash
tiup cluster clean ${cluster-name} --all 
```

清空 Prometheus 以外的所有服务的日志和数据：

```bash
tiup cluster clean ${cluster-name} --all --ignore-role prometheus
```

清空节点 `192.168.1.70:9000` 以外的所有服务的日志和数据：

```bash
tiup cluster clean ${cluster-name} --all --ignore-node 192.168.1.70:9000
```

清空部署在 `172.16.13.12` 以外的所有服务的日志和数据：

```bash
tiup cluster clean ${cluster-name} --all --ignore-node 192.168.1.70
```

## 6、删除集群

```bash
tiup cluster destroy 集群名字
```

## 7、集群组件升级

在官方组件提供了新版之后，你可以使用 `tiup update` 命令来升级组件。除了以下几个参数，该命令的用法基本和 `tiup install` 相同：

- `--all`：升级所有组件
- `--nightly`：升级至 nightly 版本
- `--self`：升级 TiUP 自己至最新版本
- `--force`：强制升级至最新版本

示例一：升级所有组件至最新版本

```shell
tiup update --all
```

示例二：升级所有组件至 nightly 版本

```shell
tiup update --all --nightly
```

示例三：升级 TiUP 至最新版本

```shell
tiup update --self
```

## 8、集群组件扩容

编辑组件节点扩容节点信息的文件 `tidb-servers-scale-out-new-node.yaml`

TIDB配置文件参考

```ini
tidb_servers:
  - host: 10.0.1.5
    ssh_port: 22
    port: 4000
    status_port: 10080
    deploy_dir: /data/deploy/install/deploy/tidb-4000
    log_dir: /data/deploy/install/log/tidb-4000
```

TiKV 配置文件参考：

```ini
tikv_servers:
  - host: 10.0.1.5
    ssh_port: 22
    port: 20160
    status_port: 20180
    deploy_dir: /data/deploy/install/deploy/tikv-20160
    data_dir: /data/deploy/install/data/tikv-20160
    log_dir: /data/deploy/install/log/tikv-20160
```

PD 配置文件参考：

```ini
pd_servers:
  - host: 10.0.1.5
    ssh_port: 22
    name: pd-1
    client_port: 2379
    peer_port: 2380
    deploy_dir: /data/deploy/install/deploy/pd-2379
    data_dir: /data/deploy/install/data/pd-2379
    log_dir: /data/deploy/install/log/pd-2379
```

可以使用 `tiup cluster edit-config <cluster-name>` 查看当前集群的配置信息，因为其中的 `global` 和 `server_configs` 参数配置默认会被 `scale-out.yaml` 继承，因此也会在 `scale-out.yaml` 中生效。

> 此处假设当前执行命令的用户和新增的机器打通了互信，如果不满足已打通互信的条件，需要通过 `-p` 来输入新机器的密码，或通过 `-i` 指定私钥文件。

执行扩容命令

```shell
tiup cluster scale-out 集群名 tidb-servers-scale-out-new-node.yaml
```

预期输出 Scaled cluster `<cluster-name>` out successfully 信息，表示扩容操作成功。

## 9、集群组件缩容

```bash
tiup cluster scale-in  -N $[节点IP]:[组件端口] ${cluster-name}
```

## 10、集群组件实例清理

你可以使用 `tiup clean` 命令来清理组件实例，并删除工作目录。如果在清理之前实例还在运行，会先 kill 相关进程。该命令用法如下：

```bash
tiup clean [tag] [flags]
```

支持以下参数：

- `--all`：清除所有的实例信息

其中 tag 表示要清理的实例 tag，如果使用了 `--all` 则不传递 tag。

示例一：清理 tag 名称为 `experiment` 的组件实例

```shell
tiup clean experiment
```

示例二：清理所有组件实例

```shell
tiup clean --all
```

## 10、集群组件卸载

TiUP 安装的组件会占用本地磁盘空间，如果不想保留过多老版本的组件，可以先查看当前安装了哪些版本的组件，然后再卸载某个组件。

你可以使用 `tiup uninstall` 命令来卸载某个组件的所有版本或者特定版本，也支持卸载所有组件。该命令用法如下：

```bash
tiup uninstall [component][:version] [flags]
```

支持的参数：

- `--all`：卸载所有的组件或版本
- `--self`：卸载 TiUP 自身

component 为要卸载的组件名称，version 为要卸载的版本，这两个都可以省略，省略任何一个都需要加上 `--all` 参数：

- 若省略版本，加 `--all` 表示卸载该组件所有版本
- 若版本和组件都省略，则加 `--all` 表示卸载所有组件及其所有版本

示例一：卸载 v3.0.8 版本的 TiDB

```shell
tiup uninstall tidb:v3.0.8
```

示例二：卸载所有版本的 TiKV

```shell
tiup uninstall tikv --all
```

示例三：卸载所有已经安装的组件

```shell
tiup uninstall --all
```

## 11、集群节点命令操作

通过 `tiup cluster deploy` 完成部署操作才可以通过 `exec` 命令来进行集群级别管理工作，例如

```bash
tiup cluster exec 集群名字 --sudo --command "yum install -y numactl" 
# 或者
tiup cluster exec 集群名字 --sudo --command "yum install -y numactl" -R PD
# 或者
tiup cluster exec 集群名字 --sudo --command "yum install -y numactl" -N node1
```

# 五、tiup命令

## 1、查看集群列表

```bash
tiup cluster list
```

# 六、搭建本地开发集群

以MacOS为例

## 1、下载安装

```bash
curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh

# TiUP 安装完成后会提示 Shell profile 文件的绝对路径。在执行以下 source 命令前，需要将 ${your_shell_profile} 修改为 Shell profile 文件的实际位置。
source ${your_shell_profile}
```

## 2、启动

```bash
# 执行 tiup playground 命令会运行最新版本的 TiDB 集群，其中 TiDB、TiKV、PD 和 TiFlash 实例各 1 个.
# 也可以指定 TiDB 版本以及各组件实例个数，命令类似于：tiup playground v6.1.0 --db 2 --pd 3 --kv 3
# --without-monitor : 设置不使用 Prometheus 和 Grafana 的监控功能。若不添加此参数，则默认开启监控功能。
tiup playground 
tiup playground v6.1.1 --db 3 --pd 3 --kv 3
```

## 3、持久化启动

在组件启动之前，TiUP 会先为它创建一个目录，然后将组件放到该目录中运行。组件会将所有数据生成在该目录中，目录的名字就是该组件运行时指定的 tag 名称。如果不指定 tag，则会随机生成一个 tag 名称，并且在实例终止时 自动删除 工作目录。如果希望分配固定目录，可以用 -T/--tag 来指定目录名字，这样多次执行使用同样的 tag 就能读写到同一批文件。

```bash
tiup playground v6.1.1 --tag=my-tidb-611
# 数据存储在路径'~/.tiup/data/标签名' 目录下
```

## 4、访问

```bash
# 查看已启动集群的信息
tiup playground display
tiup status 

# 通过 http://127.0.0.1:9090 访问 TiDB 的 Prometheus 管理界面。
# 通过 http://127.0.0.1:2379/dashboard 访问 TiDB Dashboard 页面，默认用户名为 root，密码为空。
# 通过 http://127.0.0.1:3000 访问 TiDB 的 Grafana 界面，默认用户名和密码都为 admin
```

参考：https://docs.pingcap.com/zh/tidb/stable/quick-start-with-tidb#%E9%83%A8%E7%BD%B2%E6%9C%AC%E5%9C%B0%E6%B5%8B%E8%AF%95%E9%9B%86%E7%BE%A4

# 七、问题总结

## 1、tiup命令升级

如果使用`tiup update --self`命令无法自动升级tiup命令本身，可以先备份`~/.tiup`目录后直接重装tiup命令，重装后检查新装命令是否能正常获取TIDB集群信息

```bash
cp -r ~/.tiup ~/.tiup-bak
curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
tiup cluster display 集群名称
```