# Elasticsearch集群升级

# 一、简介

对于升级重启情况，在执行升级操作时，集群可能：

- 滚动重启（节点依次重启，期间服务可以正常访问，但性能可能受到部分影响，建议在集群负载不高时进行）
- 全量重启（所有节点完全关闭后重启，期间服务不可访问，需谨慎选择）

## 版本升级路径

| Upgrade from                   | **7.17.5 的推荐升级路径**                                    |
| ------------------------------ | ------------------------------------------------------------ |
| 以前的 7.17 版本（例如7.17.0） | [滚动升级](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/rolling-upgrades.html) 至7.17.5 |
| 7.0–7.16                       | [滚动升级](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/rolling-upgrades.html) 至7.17.5 |
| 6.8                            | [滚动升级](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/rolling-upgrades.html) 至7.17.5 |
| 6.0–6.7                        | 先[滚动升级](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/rolling-upgrades.html) 至6.8<br/>再[滚动升级](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/rolling-upgrades.html) 至7.17.5 |
| 5.6                            | 先[滚动升级](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/rolling-upgrades.html) 至6.8<br/>再[滚动升级](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/rolling-upgrades.html) 至7.17.5 |
| 5.0–5.5                        | 先[滚动升级](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/rolling-upgrades.html) 至5.6<br>再[滚动升级](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/rolling-upgrades.html) 至6.86<br/>再[滚动升级](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/rolling-upgrades.html) 至7.17.5 |

## 支持滚动升级的版本路径

- 同一主要版本的次要版本之间
- 从 5.6 到 6.8
- 从 6.8 到 7.17.5
- 从 7.17.0 到 7.17.5 的任何版本
- 从 6.7 或更早版本直接升级到 7.17.5 需要 [完全重启集群](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/restart-upgrade.html)。
- **不**支持从 6.8 到 7.0 的升级路径（全集群重启和滚动升级）。

# 二、升级步骤

## 0、升级检查

- 检查新版本使用的Java版本，如果Java版本不兼容，在第四步之后升级Java

## 1、**禁用分片分配**

```bash
PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.enable": "primaries"
  }
}
```

## 2、**(可选的)停止不必要的索引并执行同步刷新**

```bash
POST _flush/synced
```

## 3、(可选的)**暂时停止与活动机器学习作业和数据馈送相关的任务**

```bash
POST _flush/synced
```

## 4、**关闭单个节点**

```bash
systemctl stop elasticsearch.service
```

## 5、**升级关闭的节点**

### 二进制包

```bash
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```

```bash
[elasticsearch]
name=Elasticsearch repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=0
autorefresh=1
type=rpm-md
```

```bash
yum install --enablerepo=elasticsearch elasticsearch
```

## 6、**升级插件**

```bash
/usr/share/elasticsearch/bin/elasticsearch-plugin list
/usr/share/elasticsearch/bin/elasticsearch-plugin remove analysis-smartcn repository-s3
/usr/share/elasticsearch/bin/elasticsearch-plugin install analysis-smartcn repository-s3
```

## 7、**启动升级的节点**

```bash
systemctl start elasticsearch
```

- 在安装过程中，旧的配置文件不会被覆盖，新的配置文件会以rpmnew结尾
- 启动过程中，可能旧的配置文件的值被废弃了或无效的，需要根据具体报错及时修改

## 8、等待节点恢复

- 没有未分配的分片
- 节点状态由`Yellow`转为`Green`

```bash
# 查看集群健康状态 
GET /_cluster/health
GET _cat/health?v=true

# 查看集群恢复状态 
GET _cat/recovery
```

## 9、**重复4~8步骤升级余下节点**

## 10、**重新启用分片分配**

```bash
PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.enable": null
  }
}
```

## 11、升级Kibana

`/etc/yum.repos.d/kibana.repo`  

```ini
[kibana]
name=Kibana repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=0
autorefresh=1
type=rpm-md
```

```bash
yum install --enablerepo=kibana kibana
```



# 参考

- https://www.elastic.co/guide/en/elasticsearch/reference/7.17/setup-upgrade.html

- https://www.elastic.co/guide/en/elasticsearch/reference/7.17/rolling-upgrades.html