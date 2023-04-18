# 一、简介

TiDB Ansible是 PingCAP 基于 Ansible playbook 功能编写的集群部署工具

TIDB Ansible Github：https://github.com/pingcap/tidb-ansible

官方文档：https://pingcap.com/docs-cn/stable/how-to/deploy/orchestrated/ansible/

# 二、TiDB Ansible使用指南

## 1、版本

目前 tidb-ansible 主要分支有：master、release-2.1、release-2.0、release-1.1、release-1.0；在比较老的版本中 tidb-ansible 是不区分 tag 的，应一些用户的需求，我们从 v2.1.1 和 v.2.0.10 版本开始对 tidb-ansible 打 Tag，不同 Tag 的 tidb-ansible 来操作对应版本的 TiDB 集群。

- master 分支用来部署 latest 版本（最新版本，目前就是指 3.0.0-BETA）集群，latest 版本由于还没有 GA，所以不区分 Tag。
- release-2.1 分支用来部署 2.1.x 版本的集群，其中包含 v2.1.1 及之后版本的 Tag。不同 tag 之间会有一些 pr 的修改，可能包含参数等变更，所以建议使用相匹配的版本来操作集群，比如：
  - 使用 v2.1.5 Tag 的 tidb-ansible 来部署管理 2.1.5 版本的集群
  - 从 2.1.3 版本升级到 2.1.5 版本，也需要使用 v2.1.5 Tag 的 tidb-ansible 来操作
- release-2.0 分支和 2.1 分支类似。
- release-1.1 和 release-1.0 分支属于老版本（很少用户使用这个版本），没有 Tag。

## 2、目录结构

tidb-ansible 实际上是 playbook 和文件（binary、配置文件等）的集合。每个 playbook 都包含了一系列的操作指令，当执行 playbook 时，就会按照定义好的指令对不同节点做分发 binary、生成配置文件、启停服务等操作。这里需要注意，yml 文件对格式要求很严格，同一级别的配置一定要缩进对齐。

- **ini 文件**

  主要是 hosts.ini 和 inventory.ini，ini 文件以 [xxx]（组名称） 分组，在 playbook 中可以定义对哪个组做哪些操作。ansible-playbook 默认使用当前目录中的 inventory.ini 文件，在 tidb-ansible/ansible.cfg 中设置。

  - hosts.ini 用来初始化机器，添加用户、设置互信、同步时间等。

  - inventory.ini 用来部署集群，包含集群的拓扑和一些常用变量。如果有自定义变量或者单机多个实例，ip 或者别名要区分开，一定不要相同，不然变量之间会互相覆盖。如下截图包含两个示例，每个示例中的变量都会互相覆盖，最终只会有一个生效。

- **conf 目录**

  集群各组件的默认初始化配置文件

- **scripts 目录**

  主要存放 Grafana Dashboard 相关的文件、环境校验脚本、日常使用常用脚本。其中 .json 文件就是我们平常监控上看到的 dashboard 的内容，也可以在 grafana 上手动 import 导入。

- **group_vars 目录**

  包含了不同组的变量和全局变量。需要注意的是，group_vars 中的全局/组变量优先级比 inventory.ini 中的全局/组变量优先级高，但是比 inventory.ini 中 host 后面设置的变量优先级低。

- **roles 目录**

  roles 里面的内容比较多，核心功能都在这个目录中，tidb-ansible 目录下的 playbook 执行时也是在调用 roles 中对应的角色。下面以 roles/pd 为例具体说明：

  - defaults 是必须存在的目录，存储一些默认变量信息，这个变量的优先级最低
  - files 目录存放由 copy 或 script 等模块调用的文件
  - meta 目录定义当前角色的特殊设定及其依赖关系，至少应该包含一个名为main.yml的文件，其它的文件需要在此文件中通过include 进行包含
  - tasks 是存放 playbook 的目录，其中 main.yml 是主入口文件，在 main.yml 中导入其他 yml 文件
  - templates 目录存放模板文件，template 模块会将模板文件中的变量替换为实际值，然后覆盖到客户机指定路径上
  - vars 目录存放角色用到的变量，这里我们用来存放默认配置文件信息

- **download** 目录

  `local_prepare.yml` playbook下载的集群组件二进制压缩安装包

- **resource** 目录

  - 

## 3、常用 playbook 说明

- **deploy.yml** 用来部署集群。执行 deploy 操作会自动将配置文件、binary 等分发到对应的节点上；如果是已经存在的集群，执行时会对比配置文件、binary 等信息，如果有变更就会覆盖原来的文件并且将原来的文件备份到 backup（默认） 目录。
- **start.yml** 用来启动集群。注意：这个操作只是 start 集群，不会对配置等信息做任何更改
- **stop.yml** 用来停止集群。与 start 一样，不会对配置等做任何修改。
- **rolling_update.yml** 用来逐个更新集群内的节点。此操作会按 PD、TiKV、TiDB 的顺序以 1 并发对集群内的节点逐个做 stop → deploy → start 操作，其中对 PD 和 TiKV 操作时会先迁移掉 leader，将对集群的影响降到最低。一般用来对集群做配置更新或者升级。
- **rolling_update_monitor.yml** 用来逐个更新监控组件，与 rolling_update.yml 功能一样，面向的组件有区别。
- **unsafe_cleanup.yml** 用来清掉整个集群。这个操作会先将整个集群停掉服务，然后删除掉相关的目录，操作不可逆，需要谨慎。
- **unsafe_cleanup_data.yml** 用来清理掉 PD、TiKV、TiDB 的数据。执行时会先将对应服务停止，然后再清理数据，操作不可逆，需要谨慎。这个操作不涉及监控。

# 三、Prerequisite

## 1、服务器准备

### Linux 操作系统版本

| Linux 操作系统平台       | 版本         |
| :----------------------- | :----------- |
| Red Hat Enterprise Linux | 7.3 及以上   |
| CentOS（推荐）           | 7.3 及以上   |
| Oracle Enterprise Linux  | 7.3 及以上   |
| Ubuntu LTS               | 16.04 及以上 |

### 服务器配置推荐

#### 开发测试环境

| **组件** | **CPU** | **内存** | **本地存储** | **网络** | **实例数量(最低要求)** |
| :------- | :------ | :------- | :----------- | :------- | :--------------------- |
| TiDB     | 8核+    | 16 GB+   | 无特殊要求   | 千兆网卡 | 1（可与 PD 同机器）    |
| PD       | 4核+    | 8 GB+    | SAS, 200 GB+ | 千兆网卡 | 1（可与 TiDB 同机器）  |
| TiKV     | 8核+    | 32 GB+   | SSD, 200 GB+ | 千兆网卡 | 3                      |

- 验证测试环境中的 TiDB 和 PD 可以部署在同一台服务器上。
- 如进行性能相关的测试，避免采用低性能存储和网络硬件配置，防止对测试结果的正确性产生干扰。
- TiKV 的 SSD 盘推荐使用 NVME 接口以保证读写更快。
- 如果仅验证功能，建议使用 [Docker Compose 部署方案](https://pingcap.com/docs-cn/stable/how-to/get-started/deploy-tidb-from-docker-compose)单机进行测试。
- TiDB 对于磁盘的使用以存放日志为主，因此在测试环境中对于磁盘类型和容量并无特殊要求。

#### 生产环境

| **组件** | **CPU** | **内存** | **硬盘类型** | **网络**            | **实例数量(最低要求)** |
| :------- | :------ | :------- | :----------- | :------------------ | :--------------------- |
| TiDB     | 16核+   | 32 GB+   | SAS          | 万兆网卡（2块最佳） | 2                      |
| PD       | 4核+    | 8 GB+    | SSD          | 万兆网卡（2块最佳） | 3                      |
| TiKV     | 16核+   | 32 GB+   | SSD          | 万兆网卡（2块最佳） | 3                      |
| 监控     | 8核+    | 16 GB+   | SAS          | 千兆网卡            | 1                      |

- 生产环境中的 TiDB 和 PD 可以部署和运行在同服务器上，如对性能和可靠性有更高的要求，应尽可能分开部署。
- 生产环境强烈推荐使用更高的配置。
- TiKV 硬盘大小配置建议 PCI-E SSD 不超过 2 TB，普通 SSD 不超过 1.5 TB。

### 实践服务器配置

| IP          | 硬件配置                                     | TiDB角色                                                     |
| ----------- | -------------------------------------------- | ------------------------------------------------------------ |
| 192.168.6.1 | 16C64G100G 1TB数据盘LVM2挂载到路径 Centos7.3 | TiDB1 PD1 TiKV1                                              |
| 192.168.6.2 | 16C64G100G 1TB数据盘LVM2挂载到路径 Centos7.3 | TiDB2 PD2 TiKV2                                              |
| 192.168.6.3 | 16C64G100G 1TB数据盘LVM2挂载到路径 Centos7.3 | TiDB3 PD3 TiKV3                                              |
| 192.168.6.4 | 16C64G100G 1TB数据盘LVM2挂载到路径 Centos7.3 | Zookeeper、Kafka、Pump Server、Drainer、Importer、Lightning、kafka Exporter、Spark Master、 |
| 192.168.6.5 | 16C64G100G 1TB数据盘LVM2挂载到路径 Centos7.3 | Ansible、Prometheus、Grafana、 Pushgateway、Alertmanager、Spark Slave |



## 2、所有部署主机添加数据盘

所有部署主机的100G数据磁盘为`/dev/vdb`

## 3、Ansible节点准备

⓪下载TIDB Ansile

```bash
tag=v3.0.12 && \
git clone -b $tag https://github.com/pingcap/tidb-ansible.git
```

①安装 Ansible 及其依赖

目前，TiDB release-2.0、release-2.1、release-3.0 及最新开发版本兼容 Ansible 2.4 ~ 2.7.11 (2.4 ≤ Ansible ≤ 2.7.11)。Ansible 及相关依赖的版本信息记录在 `tidb-ansible/requirements.txt` 文件中。

```bash
cd /home/tidb/tidb-ansible && \
sudo pip install -r ./requirements.txt && \
ansible --version

# ansible 2.7.11
#  config file = /home/tidb/tidb-ansible/ansible.cfg
#  configured module search path = [u'/home/tidb/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
#  ansible python module location = /usr/lib/python2.7/site-packages/ansible
#  executable location = /bin/ansible
#  python version = 2.7.5 (default, Nov  6 2016, 00:28:07) [GCC 4.8.5 20150623 (Red Hat 4.8.5-11)]
```

Ansible默认配置文件在`/home/tidb/tidb-ansible/ansible.cfg`

## 4、Ansible节点与部署目标机器的通信准备

⓪配置Ansible节点与部署目标机器的SSH 互信及 sudo 规则

在部署目标机器上创建 `tidb` 用户，并配置 sudo 规则，配置中控机与部署目标机器之间的 SSH 互信。

`vi /home/tidb/tidb-ansible/host.ini`

```ini
[servers]
192.168.6.1
192.168.6.2
192.168.6.3
192.168.6.4
192.168.6.5
[all:vars]
username = tidb
ntp_server = pool.ntp.org
```

```bash
ansible-playbook -i /home/tidb/tidb-ansible/hosts.ini create_users.yml -u root -k
```

## 5、在部署目标机器上安装 NTP 服务

```bash
ansible-playbook -i /home/tidb/tidb-ansible/hosts.ini deploy_ntp.yml -u tidb -b
```

## 6、格式化部署目标机器的数据盘以LVM2方式挂载到/data/tidb

```bash
cd /home/tidb/tidb-ansible/ && \
ansible -i hosts.ini all -m yum -a 'name=lvm2 state=present' && \
ansible -i hosts.ini all -m shell -a 'parted -s -a optimal /dev/vdb mklabel gpt -- mkpart primary ext4 1 -1 && disk=/dev/vdb1 && pvcreate ${disk} && vgcreate ${disk} && vgcreate -s 16m tidb ${disk} && PE_Number=`vgdisplay ${disk}|grep "Free  PE"|awk '{print $5}'` && lvcreate -l ${PE_Number} -n tidb tidb && mkfs.ext4 /dev/tidb/tidb && mkdir -p /data/tidb && chown -R tidb:tidb /data/tidb && echo "/dev/tidb/tidb /data/tidb ext4 defaults 0 0" >> /etc/fstab && mount -a && df -m'
```

# 四、安装

## 1、准备TiDB Ansible的部署主机清单`inventory.ini`

```bash
## TiDB Cluster Part
[tidb_servers]
192.168.6.1
192.168.6.2
192.168.6.3
[tikv_servers]
node1.pro.tidb ansible_host=192.168.6.1 deploy_dir=/data/tidb tikv_port=20171 labels="host=node1"
node2.pro.tidb ansible_host=192.168.6.2 deploy_dir=/data/tidb tikv_port=20171 labels="host=node2"
node3.pro.tidb ansible_host=192.168.6.3 deploy_dir=/data/tidb tikv_port=20171 labels="host=node3"

[pd_servers]
192.168.6.1
192.168.6.2
192.168.6.3

[spark_master]
192.168.6.4
[spark_slaves]
192.168.6.5
[lightning_server]
192.168.6.4
[importer_server]
192.168.6.4

## Monitoring Part: prometheus and pushgateway servers
[monitoring_servers]
192.168.6.5
[grafana_servers]
192.168.6.5
# node_exporter and blackbox_exporter servers
[monitored_servers]
192.168.6.1
192.168.6.2
192.168.6.3
192.168.6.4
192.168.6.5
[alertmanager_servers]
192.168.6.7
[kafka_exporter_servers]
192.168.6.4
## Binlog Part
[pump_servers]
192.168.6.4
[drainer_servers]
192.168.6.4
## Group variables
[pd_servers:vars]
location_labels = ["host"]

## Global variables
[all:vars]
deploy_dir = /data/tidb

## Connection ssh via normal user
ansible_user = tidb
cluster_name = Curiouser-TiDB
tidb_version = v3.0.12
# process supervision, [systemd, supervise]
process_supervision = systemd
timezone = Asia/Shanghai
enable_firewalld = False
# check NTP service
enable_ntpd = True
set_hostname = False
## binlog trigger
enable_binlog = True
# kafka cluster address for monitoring, example:
kafka_addrs = "192.168.6.4:9092"
# zookeeper address of kafka cluster for monitoring, example:
zookeeper_addrs = "192.168.6.4:2181"
# enable TLS authentication in the TiDB cluster
enable_tls = False
# KV mode
deploy_without_tidb = False
# wait for region replication complete before start tidb-server.
wait_replication = True
# Optional: Set if you already have a alertmanager server.
# Format: alertmanager_host:alertmanager_port
alertmanager_target = ""
grafana_admin_user = "admin"
grafana_admin_password = "admin"
### Collect diagnosis
collect_log_recent_hours = 2
enable_bandwidth_limit = True
# default: 10Mb/s, unit: Kbit/s
collect_bandwidth_limit = 10000
```

## 2、修改各组件默认配置

⓪修改TiKV默认配置文件`/home/tidb/tidb-ansible/conf/tikv.yml `

①修改PD默认配置文件`/home/tidb/tidb-ansible/conf/pd.yml`

②修改TiDB默认配置文件`/home/tidb/tidb-ansible/conf/tidb.yml`

③修改Spark默认配置文件`/home/tidb/tidb-ansible/conf/spark-defaults.yml`和`/home/tidb/tidb-ansible/conf/spark-env.yml`

④修改pump默认配置文件`/home/tidb/tidb-ansible/conf/pump.yml`

⑤修改 lightning默认配置文件`/home/tidb/tidb-ansible/conf/tidb-lightning.yml`

⑥修改 importer默认配置文件`/home/tidb/tidb-ansible/conf/tidb-importer.yml`

⑦修改 drainer默认配置文件`/home/tidb/tidb-ansible/conf/drainer.toml`

⑧修改组件二进制包下载默认配置文件`/home/tidb/tidb-ansible/conf/binary_packages.yml`

和`/home/tidb/tidb-ansible/conf/common_packages.yml`

## 3、部署准备

⓪测试部署目标服务器的SSH通信

```bash
ansible -i inventory.ini all -m shell -a 'whoami'
```

如果所有 server 均返回 `tidb`，表示 SSH 互信配置成功

```bash
ansible -i inventory.ini all -m shell -a 'whoami' -b
```

如果所有 server 均返回 `root`，表示 `tidb` 用户 sudo 免密码配置成功。

①下载组件二进制包

```bash
ansible-playbook local_prepare.yml
```

②开启TiKV数据加密存储功能，创建加密token文件

TiKV 只接受 hex 格式的 token 文件，文件的长度必须是 2^n，并且小于等于 1024。

目前 TiKV 数据加密存储存在以下限制：

- 对之前没有开启加密存储的集群，不支持开启该功能。
- 已经开启加密功能的集群，不允许关闭加密存储功能。
- 同一集群内部，不允许部分 TiKV 实例开启该功能，部分 TiKV 实例不开启该功能。对于加密存储功能，所有 TiKV 实例要么都开启该功能，要么都不开启该功能。这是由于 TiKV 实例之间会有数据迁移，如果开启了加密存储功能，迁移过程中数据也是加密的。

```bash
/home/tidb/tidb-ansible/resources/bin/tikv-ctl random-hex --len 256 > /home/tidb/tidb-ansible/cipher-file-256
```

③在TiKV默认配置文件`/home/tidb/tidb-ansible/conf/tikv.yml `中添加配置

```ini
[security]
# Cipher file 的存储路径
cipher-file = "/data/tidb/conf/cipher-file-256"
```

## 4、初始化部署目标主机系统环境，修改内核参数

```bash
ansible-playbook bootstrap.yml
```

## 5、分发组件安装资源到部署目标主机

```bash
ansible-playbook deploy.yml
```

分发TiKV 数据加密存储token文件到TiKV目标主机上

```bash
ansible -i inventory.ini tikv_servers -m copy -a 'src=/home/tidb/tidb-ansible/cipher-file-256 dest=/data/tidb/conf/cipher-file-256 owner=tidb group=tidb'
```

## 6、启动 TiDB 集群

```bash
ansible-playbook start.yml
```

## 7、验证

- 使用 MySQL 客户端连接 TiDB 集群。TiDB 服务的默认端口为 `4000`

  ```bash
  mysql -u root -h 192.168.6.1 -P 4000
  ```

- 浏览器访问Grafana：http://192.168.6.5:3000 （默认帐号与密码：`admin/admin`）



# 五、集群操作

## 1、ansible控制集群

### ⓪启动集群

此操作会按顺序启动整个 TiDB 集群所有组件（包括 PD、TiDB、TiKV 等组件和监控组件）。

```bash
ansible-playbook start.yml
```

### ①关闭集群

此操作会按顺序关闭整个 TiDB 集群所有组件（包括 PD、TiDB、TiKV 等组件和监控组件）。

```bash
ansible-playbook stop.yml
```

### ②清除集群数据

此操作会关闭 TiDB、Pump、TiKV、PD 服务，并清空 Pump、TiKV、PD 数据目录。

```bash
ansible-playbook unsafe_cleanup_data.yml
```

### ③销毁集群

此操作会关闭集群，并清空部署目录，若部署目录为挂载点，会报错，可忽略。

```bash
ansible-playbook unsafe_cleanup.yml
```

### ④更新组件配置并滚动重启

```bash
ansible-playbook rolling_update.yml --tags=组件标签
```

配置并重启 Prometheus 服务

```bash
ansible-playbook rolling_update_monitor.yml --tags=prometheus
```

### ⑤只操作组件某节点上的其中一个服务



```bash
ansible-playbook 操作playbook --tags=组件 -l 组件节点的ip或hostname或inventry中的别名

# 例如停掉192.168.6.3节点上的所有服务
ansible-playbook stop.yml -l 192.168.6.3

# 关闭您想要下线的 DM-worker 实例
ansible-playbook stop.yml --tags=dm-worker -l dm_worker3
```



# 六、压力测试

详见：https://pingcap.com/docs-cn/stable/benchmark/how-to-run-sysbench/

# 七、扩容缩容

## 1、扩容组件节点

### 扩容TiKV节点

**注意**：

- 如果 `inventory.ini` 中为节点配置了别名，如 `node101 ansible_host=192.168.6.101`，执行 ansible-playbook 时 -l 请指定别名，以下步骤类似。例如：`ansible-playbook bootstrap.yml -l node101,node102`

⓪编辑 `inventory.ini` 文件和 `hosts.ini` 文件，添加节点信息

`hosts.ini`

```ini
[servers]
...省略...
192.168.6.6

[all:vars]
...省略...
```

`inventory.ini`

```ini
...省略...
[tikv_servers]
node4.pro.tidb ansible_host=192.168.6.6 deploy_dir=/data/tidb tikv_port=20171 labels="host=node4"
...省略...
```

①配置SSH 互信及 sudo 规则、安装 NTP 服务、挂载数据盘、初始化新增节点，分发安装资源

```bash
ansible-playbook -i hosts.ini create_users.yml -l 192.168.6.6 -u root -k

ansible-playbook -i hosts.ini deploy_ntp.yml -u tidb -b

cd /home/tidb/tidb-ansible/ && \
ansible -i hosts.ini 192.168.6.6 -m yum -a 'name=lvm2 state=present' && \
ansible -i hosts.ini 192.168.6.6 -m shell -a 'parted -s -a optimal /dev/vdb mklabel gpt -- mkpart primary ext4 1 -1 && disk=/dev/vdb1 && pvcreate ${disk} && vgcreate ${disk} && vgcreate -s 16m tidb ${disk} && PE_Number=`vgdisplay ${disk}|grep "Free  PE"|awk '{print $5}'` && lvcreate -l ${PE_Number} -n tidb tidb && mkfs.ext4 /dev/tidb/tidb && mkdir -p /data/tidb && chown -R tidb:tidb /data/tidb && echo "/dev/tidb/tidb /data/tidb ext4 defaults 0 0" >> /etc/fstab && mount -a && df -m'

ansible-playbook bootstrap.yml -l node4.pro.tidb

ansible-playbook deploy.yml -l node4.pro.tidb

ansible -i hosts.ini 192.168.6.6 -m copy -a 'src=/home/tidb/tidb-ansible/cipher-file-256 dest=/home/tidb/conf/cipher-file-256 owner=tidb group=tidb'
```

②启动新节点服务

```bash
ansible-playbook start.yml -l node4.pro.tidb
```

③更新 Prometheus 配置并重启

```bash
ansible-playbook rolling_update_monitor.yml --tags=prometheus
```



### 扩容PD节点

⓪编辑 `inventory.ini` 文件和 `hosts.ini` 文件，添加节点信息

`hosts.ini`

```ini
[servers]
...省略...
192.168.6.7

[all:vars]
...省略...
```

`inventory.ini`

```ini
...省略...
[pd_servers]
192.168.6.7
...省略...
```

①配置SSH 互信及 sudo 规则、安装 NTP 服务、挂载数据盘、初始化新增节点，分发安装资源

```bash
ansible-playbook -i hosts.ini create_users.yml -l 192.168.6.7 -u root -k

ansible-playbook -i hosts.ini deploy_ntp.yml -u tidb -b

cd /home/tidb/tidb-ansible/ && \
ansible -i hosts.ini 192.168.6.7 -m yum -a 'name=lvm2 state=present' && \
ansible -i hosts.ini 192.168.6.7 -m shell -a 'parted -s -a optimal /dev/vdb mklabel gpt -- mkpart primary ext4 1 -1 && disk=/dev/vdb1 && pvcreate ${disk} && vgcreate ${disk} && vgcreate -s 16m tidb ${disk} && PE_Number=`vgdisplay ${disk}|grep "Free  PE"|awk '{print $5}'` && lvcreate -l ${PE_Number} -n tidb tidb && mkfs.ext4 /dev/tidb/tidb && mkdir -p /data/tidb && chown -R tidb:tidb /data/tidb && echo "/dev/tidb/tidb /data/tidb ext4 defaults 0 0" >> /etc/fstab && mount -a && df -m'

ansible-playbook bootstrap.yml -l 192.168.6.7

ansible-playbook deploy.yml -l 192.168.6.7
```

②登录新增的 PD 节点，编辑启动脚本：`/data/tidb/scripts/run_pd.sh`

- 移除 `--initial-cluster="xxxx" \` 配置，注意这里不能在行开头加注释符 #。

- 添加 `--join="http://192.168.6.1:2379" \`，IP 地址 （192.168.6.7） 可以是集群内现有 PD IP 地址中的任意一个。

③在新增 PD 节点中启动 PD 服务

```bash
/data/tidb/scripts/start_pd.sh
```

③使用 `pd-ctl` 检查新节点是否添加成功：

```bash
/home/tidb/tidb-ansible/resources/bin/pd-ctl -u "http://192.168.6.1:2379" -d member
```

④启动监控服务

```bash
ansible-playbook start.yml -l 192.168.6.7
```

⑤更新集群的配置：

```bash
ansible-playbook deploy.yml
```

⑥重启 Prometheus，新增扩容的 PD 节点的监控：

```bash
ansible-playbook rolling_update_monitor.yml --tags=prometheus
```

## 2、缩容组件节点

⓪停止节点上所有的服务：

注意：如果该服务器上还有其他服务（例如 `TiDB`），则还需要使用 `--tags组件标签 ` 来指定服务（例如 `-t tidb`）。

```bash
ansible-playbook stop.yml -l 192.168.6.9
```

①编辑 `inventory.ini` 文件，移除节点信息

②更新 Prometheus 配置并重启

```bash
ansible-playbook rolling_update_monitor.yml --tags=prometheus
```

### 缩容 TiKV 节点

⓪使用 `pd-ctl` 从集群中移除节点：

- 查看 TiKV node节点的 store id：

  ```bash
  /home/tidb/tidb-ansible/resources/bin/pd-ctl -u "http://192.168.6.1:2379" -d store
  ```

- 从集群中移除TiKV node节点，假如 store id 为 10：

  ```bash
  /home/tidb/tidb-ansible/resources/bin/pd-ctl -u "http://192.168.6.1:2379" -d store delete 10
  ```

- 使用 Grafana 或者 `pd-ctl` 检查节点是否下线成功（下线需要一定时间，下线节点的状态变为 Tombstone 就说明下线成功了）：

  ```bash
  /home/tidb/tidb-ansible/resources/bin/pd-ctl -u "http://192.168.6.1:2379" -d store 10
  ```

①下线成功后，停止 TiKV node节点上的服务：

```bash
ansible-playbook stop.yml --tags=tikv -l 192.168.6.9
```

编辑 `inventory.ini` 文件，移除节点信息

②更新 Prometheus 配置并重启：

```bash
ansible-playbook stop.yml --tags=prometheus
ansible-playbook start.yml --tags=prometheus
```

### 缩容 PD 节点

TiKV 中的 PD Client 会缓存 PD 节点列表，但目前不会定期自动更新，只有在 PD leader 发生切换或 TiKV 重启加载最新配置后才会更新；为避免 TiKV 缓存的 PD 节点列表过旧的风险，在扩缩容 PD 完成后，PD 集群中至少要包含两个扩缩容操作前就已经存在的 PD 节点成员，如果不满足该条件需要手动执行 PD transfer leader 操作，更新 TiKV 中的 PD 缓存列表。

⓪使用 `pd-ctl` 从集群中移除节点：

- 查看 node2 节点的 name：

```bash
/home/tidb/tidb-ansible/resources/bin/pd-ctl -u "http://192.168.6.1:2379" -d member
```

- 从集群中移除 node2，假如 name 为 pd2：

```bash
/home/tidb/tidb-ansible/resources/bin/pd-ctl -u "http://192.168.6.1:2379" -d member delete name pd2
```

- 使用 `pd-ctl` 检查节点是否下线成功（PD 下线会很快，结果中没有 node2 节点信息即为下线成功）：

```bash
/home/tidb/tidb-ansible/resources/bin/pd-ctl -u "http://192.168.6.1:2379" -d member
```

①下线成功后，停止 node2 上的服务：

```bash
ansible-playbook stop.yml --tags=pd -l 192.168.6.9
```

②编辑 `inventory.ini` 文件，移除节点信息

③更新集群的配置：

```bash
ansible-playbook deploy.yml
```

④重启 Prometheus，移除缩容的 PD 节点的监控：

```bash
ansible-playbook stop.yml --tags=prometheus
ansible-playbook start.yml --tags=prometheus
```