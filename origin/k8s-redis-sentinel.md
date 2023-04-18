# k8s上redis主从哨兵模式的问题

# 一、上下文

主从哨兵模式

- master 节点扩容
- slave 节点扩容

# 二、问题

- k8s集群外客户端无法连接至redis 集群节点
- master pod故障重建后IP地址更换，无法加入原先的集群

# 三、解决方案

## 1、固定Redis PODIP地址
