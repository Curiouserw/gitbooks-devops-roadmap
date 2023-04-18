# Pulsar命令行pulsar-admin

# 一、pulsar-admin

pulsar-admin工具，可以用来管理Pulsar的集群、Brokers、名称空间，租户等。

参考文档：https://pulsar.apache.org/docs/en/pulsar-admin/#list-failure-domains

## 1、命令格式

```bash
pulsar-admin 命令 子命令 参数
```
子命令

- `broker-stats`：收集Brokers的统计信息
- `brokers`：操作Brokers
- `clusters`：操作集群
- `functions`：操作Pulsar函数
- `functions-worker`：收集Pulsar函数Brokers的统计信息
- `namespaces`：操作管理命令空间
- `ns-isolation-policy`：操作管理命令空间的隔离策略
- `sources`：
- `sinks`
- `topics`：操作管理主题
- `tenants`：操作管理多租户
- `resource-quotas`：操作管理资源配额
- `schemas`：操作管理主题关联的模式



# 二、pulsar-admin常用操作

## 1、创建Topic

**创建一个没有分区的Topic**

```bash
pulsar-admin topics create persistent://tenant/namespace/topic
```

**创建一个有分区的Topic**

```bash
pulsar-admin topics create-partitioned-topic persistent://tenant/namespace/topic  --partitions 4
```

- 无论是有分区还是没有分区, 创建topic后,如果没有任何操作, 60s后pulsar会认为此topic是不活动的, 会自动进行删除, 以避免生成垃圾数据

## 2、列出Topic

```bash
pulsar-admin topics list tenant/namespace
```

## 3、删除Topic

**删除没有分区的Topic**

```bash
pulsar-admin topics delete persistent://tenant/namespace/topic
```

**删除有分区的Topic**

```bash
pulsar-admin topics delete-partitioned-topic persistent://tenant/namespace/topic
```

**注意：如果Topic仍有producer和subscription。Topics无法删除。需要先剔除producer和subscription。请参照13小节剔除subscription后再次尝试删除Topic。[相关文档](https://pulsar.apache.org/docs/admin-api-topics/#delete-topic)**

## 4、授权

```bash
pulsar-admin topics grant-permission --actions produce,consume --role application1 persistent://tenant/namespace/topic
```

## 5、获取权限

```bash
pulsar-admin topics grant-permission --actions produce,consume --role application1 persistent://tenant/namespace/topic
```

## 6、收回权限

```bash
pulsar-admin topics revoke-permission --role application persistent://tenant/namespace/topic

{
  "application1": [
    "consume",
    "produce"
  ]
}
```

## 7、获取租户列表

```bash
pulsar-admin tenants list
```

## 8、创建租户

```bash
pulsar-admin tenants create my-tenant
# 在创建租户时，可以使用-r或者–admin-roles标志分配管理角色。可以用逗号分隔的列表指定多个角色；
pulsar-admin tenants create my-tenant -r role1,role2,role3
```

## 9、在指定的租户下创建名称空间

```bash
pulsar-admin namespaces create tenant/namespace
```

## 10、获取所有的名称空间列表

```bash
pulsar-admin namespaces list tenant
```

## 11、删除名称空间

```bash
pulsar-admin namespaces delete tenant/namespace
```

## 12、获取名称空间的配置策略

```bash
pulsar-admin namespaces policies tenant/namespace
```

## 13、剔除消费者的订阅

```bash
pulsar-admin topics unsubscribe persistent://public/default/topic-partition-0 --subscription test --force

curl -XDELETE \
  http://brokers:8080/admin/v2/persistent/{tenant}/{namespace}/{topic}/subscription/{sub_name}?force=true
```

## 14、获取Topic状态

```bash
pulsar-admin topics stats persistent://public/default/topic-partition-0
```

具体返回值字段含义参考：https://pulsar.apache.org/docs/2.9.x/admin-api-topics#get-internal-stats

## 15、查找topic所在的broker信息

```
pulsar-admin persistent lookup persistent://public/default/topic-partition-0
```

## 16、查询topic的订阅信息

```bash
pulsar-admin persistent subscriptions persistent://public/default/topic-partition-0
```

## 17、查询最后一条消息的MessageID

```
pulsar-admin topics last-message-id persistent://public/default/topic-partition-0
```

## 18、跳过消费部分消息

```bash
pulsar-admin persistent skip --count 10 --subscription my-subscription persistent://public/default/topic-partition-0
```

## 19、跳过所有数据

```bash
pulsar-admin persistent skip-all --subscription my-subscription persistent://public/default/topic-partition-0
```

## 20、重置消费cursor到几分钟之前

```bash
pulsar-admin persistent reset-cursor --subscription my-subscription --time 10 persistent://public/default/topic-partition-0
```

## 21、设置消息保留策略

```bash
pulsar-admin namespaces set-retention persistent://public/default/topic-partition-0 \
  --size 10G \
  --time 3h
# size和time都为-1 ，则无限保留
# size和time都为0 ，则禁用保留策略
# size和time其中一个为0，则以另外一个为准作为保留策略

# 设置TTL，单位为秒
pulsar-admin namespaces set-message-ttl my-tenant/my-ns \
  --messageTTL 120 
```

# 三、pulsar-client常用操作

## 1、生产数据

如果topic不存在，pulsar会自动创建

```bash
pulsar-client produce persistent://public/default/topic --messages "Hello Pulsar I'm python client 1,Hello Pulsar I'm python client  2"

# -m, --messages ：以逗号分隔的消息字符串
# -n, --num-produce
```

## 2、消费数据

```bash
pulsar-client consume persistent://public/default/topic -s "first-subscription"

# -s, --subscription-name : 订阅者名称
# -t, --subscription-type : 订阅类型，Exclusive, Shared, Failover, Key_Shared
# -p, --subscription-position : 订阅位置。Latest, Earliest.
```
