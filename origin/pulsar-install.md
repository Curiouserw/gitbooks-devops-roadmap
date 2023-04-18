# Apache Pulsar的安装部署

# 一、简介

# 二、二进制安装

## 1、prerequisite

- 三台机器硬件：8核16G内存100G系统磁盘500G数据磁盘的CentOS 7（数据磁盘挂载到/data目录）
- 三台机器IP地址：192.168.1.121~123
- 三台都为Zookeeper集群、Apache Bookkeeper集群、Broker集群
- 三台机器安装Java JDK（过程省略）

## 2、下载

```bash
pulsar_version=2.7.1
curl -s -# https://archive.apache.org/dist/pulsar/pulsar-$pulsar_version/apache-pulsar-$pulsar_version-bin.tar.gz | tar zxvf - -C /opt
ln -s /opt/apache-pulsar-$pulsar_version /opt/pulsar
echo "export PULSAR_HOME=/opt/pulsar\nexport PATH=$PATH:$PULSAR_HOME/bin" >> /etc/profile
mkdir -p /data/pulsar/{data/bookkeeper/journal,logs} /data/zookeeper/{data,logs}
source /etc/profile
```

## 3、下载connectors

```bash
pulsar_version=2.7.1 && \
mkdir /opt/pulsar/connectors && \
nohup wget https://apachemirror.sg.wuchna.com/pulsar/pulsar-$pulsar_version/connectors/pulsar-io-kafka-$pulsar_version.nar -P /opt/pulsar/connectors >/dev/null &!
nohup wget https://ftp.kddi-research.jp/infosystems/apache/pulsar/pulsar-$pulsar_version/connectors/pulsar-io-redis-$pulsar_version.nar -P /opt/pulsar/connectors >/dev/null &!
nohup wget https://mirror-hk.koddos.net/apache/pulsar/pulsar-$pulsar_version/connectors/pulsar-io-netty-$pulsar_version.nar -P /opt/pulsar/connectors >/dev/null &!
nohup wget https://ftp.tsukuba.wide.ad.jp/software/apache/pulsar/pulsar-$pulsar_version/connectors/pulsar-io-jdbc-mariadb-$pulsar_version.nar -P /opt/pulsar/connectors >/dev/null &!
nohup wget https://ftp.wayne.edu/apache/pulsar/pulsar-$pulsar_version/connectors/pulsar-io-file-$pulsar_version.nar -P /opt/pulsar/connectors >/dev/null &!
nohup wget https://ftp.jaist.ac.jp/pub/apache/pulsar/pulsar-$pulsar_version/connectors/pulsar-io-elastic-search-$pulsar_version.nar -P /opt/pulsar/connectors >/dev/null &!
nohup wget https://ftp.jaist.ac.jp/pub/apache/pulsar/pulsar-$pulsar_version/connectors/pulsar-io-kafka-connect-adaptor-$pulsar_version.nar -P /opt/pulsar/connectors >/dev/null &!
nohup wget https://ftp.tsukuba.wide.ad.jp/software/apache/pulsar/pulsar-$pulsar_version/connectors/pulsar-io-canal-$pulsar_version.nar -P /opt/pulsar/connectors >/dev/null &!
nohup wget https://ftp.kddi-research.jp/infosystems/apache/pulsar/pulsar-$pulsar_version/connectors/pulsar-io-influxdb-$pulsar_version.nar -P /opt/pulsar/connectors >/dev/null &!
nohup wget https://apache.website-solution.net/pulsar/pulsar-$pulsar_version/connectors/pulsar-io-rabbitmq-$pulsar_version.nar -P /opt/pulsar/connectors >/dev/null &!
nohup wget https://ftp.yz.yamagata-u.ac.jp/pub/network/apache/pulsar/pulsar-$pulsar_version/connectors/pulsar-io-hdfs2-$pulsar_version.nar -P /opt/pulsar/connectors >/dev/null &!
nohup wget https://ftp.jaist.ac.jp/pub/apache/pulsar/pulsar-$pulsar_version/connectors/pulsar-io-hdfs3-$pulsar_version.nar -P /opt/pulsar/connectors >/dev/null &!
nohup wget https://ftp.jaist.ac.jp/pub/apache/pulsar/pulsar-$pulsar_version/connectors/pulsar-io-jdbc-postgres-$pulsar_version.nar -P /opt/pulsar/connectors >/dev/null &!
```

之后将整个`/opt/apache-pulsar-2.7.1`目录拷贝到另外台主机上，设置一下软连，配置一下系统变量

```bash
for i in {2..3};do
	scp -r /opt/apache-pulsar-2.7.1 root@192.168.1.12$i:/opt/ ;
	ssh root@192.168.1.12$i -c 'ln -s /opt/apache-pulsar-2.7.1 /opt/pulsar && '
done
```

## 4、部署Zookeeper集群

```bash
sed -i \
  -e 's/dataDir=data\/zookeeper/dataDir=\/data\/zookeeper\/data/g' \
  -e '$a server.1=192.168.1.121:2888:3888\nserver.2=192.168.1.122:2888:3888\nserver.3=192.168.1.123:2888:3888\n' \
  /opt/pulsar/conf/zookeeper.conf && \
echo 1 > /data/zookeeper/data/myid && \
PULSAR_EXTRA_OPTS="-Dstats_server_port=8001" pulsar-daemon start zookeeper && \
jps -l  && \
netstat -lanp|grep 2181
```

上述命令三台机器要执行

## 5、初始化pulsar集群元数据到Zookeeper中

下述命令只用执行一遍即可

```bash
pulsar initialize-cluster-metadata \
  --cluster pulsar-cluster-prod \
  --zookeeper 192.168.1.121:2181 \
  --configuration-store 192.168.1.121:2181 \
  --web-service-url http://192.168.1.121:8080,192.168.1.122:8080,192.168.1.123:8080 \
  --broker-service-url pulsar://192.168.1.121:6650,192.168.1.122:6650,192.168.1.123:6650
```

## 6、部署Apache Bookkeeper

```bash
sed -i \
 -e 's/journalDirectory=data\/bookkeeper\/journal/journalDirectory=\/data\/pulsar\/data\/bookkeeper\/journal/g' \
 -e 's/ledgerDirectories=data\/bookkeeper\/ledgers/ledgerDirectories=\/data\/pulsar\/data\/bookkeeper\/ledgers/g' \
 -e 's/zkServers=localhost:2181/zkServers=192.168.1.121:2181,192.168.1.122:2181,192.168.1.123:2181/g' \
 /opt/pulsar/conf/bookkeeper.conf && \
pulsar-daemon start bookie && \
jps -l
```

## 7、部署Broker

```bash
sed -i \
  -e 's/zookeeperServers=/zookeeperServers=192.168.1.121:2181,192.168.1.122:2181,192.168.1.123:2181/g' \
  -e 's/configurationStoreServers=/configurationStoreServers=192.168.1.121:2181,192.168.1.122:2181,192.168.1.123:2181/g' \
 -e 's/clusterName=/clusterName=pulsar-cluster-prod/g' \
 /opt/pulsar/conf/broker.conf && \
pulsar-daemon start broker && \
jps -l 
```

## 8、测试

### ①配置客户端配置

```bash
sed -i \
  -e 's/webServiceUrl=http:\/\/localhost:8080\//webServiceUrl=http:\/\/192.168.1.121:8080,192.168.1.122:8080,192.168.1.123:8080\//g' \
  -e 's/brokerServiceUrl=pulsar:\/\/localhost:6650\//brokerServiceUrl=pulsar:\/\/192.168.1.121:6650,192.168.1.122:6650,192.168.1.123:6650\//g' \
  /opt/pulsar/conf/client.conf
```

### ②配置命名空间的权限

```bash
pulsar-admin namespaces set-persistence -a 3 -e 3 -w 3 -r 3 public/default
pulsar-admin namespaces get-persistence public/default
```

### ③创建消费者消费消息

```bash
pulsar-client consume persistent://public/default/test -n 100 -s "consumer-test" -t "Exclusive"
```

### ④创建生产者产生消息

```bash
pulsar-client produce persistent://public/default/test -n 1 -m "Hello Pulsar"
```

# 三、Docker(standalone)

```bash
version: "3.4"
services:
  standalone:
    container_name: standalone-pulsar
    hostname: localhost
    image: streamnative/sn-pulsar:2.10.1.6
    volumes:
      - /data/pulsar/data:/pulsar/data
    command: >
      bash -c "bin/apply-config-from-env.py conf/standalone.conf &&
      exec bin/pulsar standalone -nss -nfw" # disable stream storage and functions worker
    environment:
      allowAutoTopicCreationType: partitioned
      brokerDeleteInactiveTopicsEnabled: "false"
      PULSAR_PREFIX_messagingProtocols: kafka
      PULSAR_PREFIX_kafkaListeners: PLAINTEXT://0.0.0.0:9092
      PULSAR_PREFIX_kafkaAdvertisedListeners: PLAINTEXT://0.0.0.0:9092
      PULSAR_PREFIX_brokerEntryMetadataInterceptors: org.apache.pulsar.common.intercept.AppendIndexMetadataInterceptor
    ports:
      - 6650:6650
      - 8080:8080
      - 9092:9092

```

# 四、Kubernetes

```bash
helm repo add pulsar https://pulsar.apache.org/charts
helm update
helm search repo pulsar -l
latest_version=$(helm search repo pulsar -l | grep -v "CHART VERSION" | awk '{print $3}' | sort -n | tail -1)
helm show values pulsar/pulsar > pulsar-$latest_version-values.yaml
helm upgrade --install pulsar -n pulsar -f pulsar-$latest_version-values.yaml pulsar/pulsar
```

# 五、Pulsar Manager

Github：https://github.com/apache/pulsar-manager#access-pulsar-manager

以二进制方式安装为例，docker或k8s相关安装配置的参考：https://github.com/apache/pulsar-manager#access-pulsar-manager 和 https://github.com/apache/pulsar-manager/blob/master/src/README.md 

## 1、下载安装

```bash
pulsar_manager_version=0.2.0
curl -s -#  https://dist.apache.org/repos/dist/release/pulsar/pulsar-manager/pulsar-manager-0.2.0/apache-pulsar-manager-$pulsar_manager_version-bin.tar.gz | tar zxvf - -C /tmp && \
tar -xvf /tmp/pulsar-manager/pulsar-manager.tar -C /opt && \
cp -r /tmp/pulsar-manager/dist /opt/pulsar-manager/ui && \
rm -rf /tmp/pulsar-manager
```

## 2、编辑配置文件

只修改`/opt/pulsar-manager/application.properties`的以下配置项，其他不用

```bash
# 开启Swagger
swagger.enabled=true
# 设置默认集群
default.environment.name=pulsar-cluster-prod
default.environment.service_url=http://127.0.0.1:8080
```

## 3、启动

```bash
nohup /opt/pulsar-manager/bin/pulsar-manager >/dev/null 2>&1 &
```

## 4、设置用户名密码

```bash
CSRF_TOKEN=$(curl http://localhost:7750/pulsar-manager/csrf-token) && echo $CSRF_TOKEN
curl \
   -H 'X-XSRF-TOKEN: $CSRF_TOKEN' \
   -H 'Cookie: XSRF-TOKEN=$CSRF_TOKEN;' \
   -H "Content-Type: application/json" \
   -X PUT http://localhost:7750/pulsar-manager/users/superuser \
   -d '{"name": "用户名", "password": "密码", "description": "Administrator", "email": "邮箱地址"}'
```

## 5、访问

[http://pulsar-manager服务器地址:7750/ui/index.html]()

[http://pulsar-manager服务器地址:7750/swagger-ui.html]()