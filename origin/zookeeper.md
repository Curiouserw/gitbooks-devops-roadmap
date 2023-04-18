# Zookeeper常用操作

# 一、Zookeeper日志

## 1、日志简介

zookeeper服务器会产生三类日志：

- **事务日志**

  > 事务日志指zookeeper在正常运行过程中，针对所有的更新操作，在返回客户端“更新成功”的响应前，zookeeper会保证已经将本次更新操作的事务日志已经写到磁盘上，只有这样，整个更新操作才会生效

  > 事务日志文件的命名规则为log.\**，文件大小为64MB，**表示写入该日志的第一个事务的ID，十六进制表示

  > 事务日志文件为二进制格式，只能通过zookeeper自带的jar包读取。
  >
  > 将libs中的slf4j-api-1.6.1.jar文件和zookeeper根目录下的zookeeper-3.4.9.jar文件复制到临时文件夹tmplibs中，然后执行以下命令
  >
  > ```bash
  > java -classpath .:slf4j-api-1.6.1.jar:zookeeper-3.4.6.jar org.apache.zookeeper.server.LogFormatter /data/zookeeper/data/version-2/log.684e26700 |more
  > ```

- **快照日志**

  > zookeeper的数据在内存中是以树形结构进行存储的，而快照就是每隔一段时间就会把整个DataTree的数据序列化后存储在磁盘中，这就是快照日志文件。

  > 快照文件的命名规则为snapshot.\**，其中**表示zookeeper触发快照的那个瞬间，提交的最后一个事务的ID

  > 事务日志文件为二进制格式，只能通过zookeeper自带的jar包读取。
  >
  > 将libs中的slf4j-api-1.6.1.jar文件和zookeeper根目录下的zookeeper-3.4.9.jar文件复制到临时文件夹tmplibs中，然后执行以下命令
  >
  > ```bash
  > java -classpath .:slf4j-api-1.6.1.jar:zookeeper-3.4.6.jar org.apache.zookeeper.server.SnapshotFormatter /data/zookeeper/data/version-2/snapshot.684e26700 |more
  > ```

- **log4j日志**

当zookeeper集群进行频繁的数据读写操作时，会产生大量的事务日志信息，将两类日志分开存储会提高系统性能，建议将事务日志（dataLogDir）与快照日志（dataLog）单独配置（在没有dataLogDir配置项的时候，zookeeper默认将事务日志文件和快照日志文件都存储在dataDir对应的目录下）



## 2、清理方式

### ①自动清理

在zookeeper 3.4.0以后，zookeeper提供了自动清理snapshot和事务日志功能，通过配置zoo.cfg下的autopurge.snapRetainCount和autopurge.purgeInterval这两个参数实现日志文件的定时清理。

- autopurge.snapRetainCount这个参数指定了需要保留的文件数目，默认保留3个；
- autopurge.purgeInterval这个参数指定了清理频率，单位是小时，需要填写一个1或者更大的数据，默认0表示不开启自动清理功能。

### ②手动清理

```bash
/opt/zookeeper/bin/zkCleanup.sh 数据目录 -n 保留日志个数
```

### ③自定义清理脚本

```bash
#!/bin/bash
#snapshot file dir
dataDir=/data/zookeeper/data/version-2
#tran log dir
dataLogDir=/data/zookeeper/data/version-2
logDir=/data/zookeeper/logs
#Leave 60 files
count=60
count=$[$count+1]
ls -t $dataLogDir/log.* | tail -n +$count | xargs rm -f
ls -t $dataDir/snapshot.* | tail -n +$count | xargs rm -f
ls -t $logDir/zookeeper.log.* | tail -n +$count | xargs rm -f
```



**参考**：

1. https://www.kancloud.cn/ningjing_home/ceph/710759
2. https://ningyu1.github.io/site/post/89-zookeeper-cleanlog/