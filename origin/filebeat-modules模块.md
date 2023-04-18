# Filebeat的Modules模块

# 一、简介

Filebeat采集日志文件除了可以自定义配置Input采集器、Processor处理器、输出目的地等，还提供大量的模板配置Modules来快速地配置采集通用格式的日志文件。例如Nginx标准格式的日志文件。

Filebeat的Module简化了`常见日志格式`的收集、解析和可视化。

一个典型的Module(例如，对于Nginx日志的Module)由一个或多个**Fileset**组成(对于Nginx，则是`access`和`error`日志文件)。Fileset包含以下内容:

- **Filebeat输入配置**：其中包含查找日志文件的默认路径，而这些默认路径取决于操作系统。Filebeat配置还负责在需要时将多行事件拼接在一起。
- **Elasticsearch Ingest节点Pipeline定义**：用于解析日志行。
- **字段定义**：用于为每个字段配置Elasticsearch的正确类型。它们还包含每个字段的简短描述。
- **Kibana表盘样本**：可以用来可视化日志文件。

## Filebeat支持的Modules

Filebeat模块需要Elasticsearch 5.2或更高版本。

| 类型                                                         |
| ------------------------------------------------------------ |
| [*Modules overview*](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-modules-overview.html) |
| [*Apache module*](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-apache.html) |
| [*Auditd module*](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-auditd.html) |
| [*AWS module*](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-aws.html) |
| [*CEF module*](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-cef.html) |
| [*Cisco module*](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-cisco.html) |
| [*Coredns Module*](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-coredns.html) |
| [*Elasticsearch module*](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-elasticsearch.html) |
| [*Envoyproxy Module*](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-envoyproxy.html) |
| [*Google Cloud module*](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-googlecloud.html) |
| [*haproxy module*](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-haproxy.html) |
| [*IBM MQ module*](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-ibmmq.html) |
| [*Icinga module*](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-icinga.html) |
| [*IIS module*](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-iis.html) |
| [*Iptables module*](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-iptables.html) |
| [*Kafka module*](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-kafka.html) |
| [*Kibana module*](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-kibana.html) |
| [*Logstash module*](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-logstash.html) |
| [*MongoDB module*](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-mongodb.html) |
| [*MSSQL module*](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-mssql.html) |
| [*MySQL module*](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-mysql.html) |
| [*nats module*](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-nats.html) |
| [*NetFlow module*](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-netflow.html) |
| [*Nginx module*](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-nginx.html) |
| [*Osquery module*](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-osquery.html) |
| [*Palo Alto Networks module*](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-panw.html) |
| [*PostgreSQL module*](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-postgresql.html) |
| [*RabbitMQ module*](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-rabbitmq.html) |
| [*Redis module*](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-redis.html) |
| [*Santa module*](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-santa.html) |
| [*Suricata module*](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-suricata.html) |
| [*System module*](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-system.html) |
| [*Traefik module*](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-traefik.html) |
| [*Zeek (Bro) Module*](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-module-zeek.html) |

# 二、Module配置

## 1. 加载Modules

在`/etc/filebeat/filebeat.yml`中配置加载Modules

```yaml
filebeat.config.modules:
  path: ${path.config}/modules.d/*.yml
  reload.enabled: true
  reload.period: 10s
```

指定特殊的filebeat全局配置文件来配置加载Modules

```bash
filebeat -c /etc/filebeat/filebeat-kong.yaml modules enable nginx
```

## 2. 查看所有Module

```bash
filebeat modules list
```

## 3. 启用Module

Filebeat的Modules配置文件通常在`/etc/filebeat/modules.d`下

Filebeat提供了几种启用模块的不同方法：

- 命令行启用module

    ```bash
    filebeat modules enable module名
    ```

- 在配置文件`/etc/filebeat/filebeat.yml`中启用Modules

  ```yaml
  filebeat.modules:
  - module: nginx
  - module: mysql
  - module: system
  ```

- 在运行时使用Modules

  ```bash
  filebeat --modules nginx,mysql,system
  ```

## 4. 配置Module变量参数

Filebeat的Modules配置文件通常在`/etc/filebeat/modules.d`下。当module不启用时，自带的Module配置文件是以`.disabled`后缀的。启用后，会自动去掉`.disabled`后缀，此时可以修改module的默认变量参数

- **在运行时配置Module变量参数**

  ```bash
  filebeat -e --modules 模块名 -M "nginx.access.var.paths=[/var/log/nginx/access.log*]"
  ```
  
- **在Modules的配置文件中配置变量参数**

  以Nginx模块的配置文件为例`/etc/filebeat/modules.d/nginx.yml`

  ```yaml
  - module: nginx
    # 设置Nginx访问日志fileset
    access:
      input:
    	  close_eof: true
      enabled: true
      # 日志文件路径。如果为空，默认根据操作系统版本自行选择日志文件路径.
      var.paths: [ "/var/log/nginx/access.log", "/var/log/nginx/admin_access.log" ]
    # 设置Nginx错误日志fileset
    error:
      enabled: true
      # 日志文件路径。如果为空，默认根据操作系统版本自行选择日志文件路径.
      var.paths: ["/var/log/nginx/error.log"]
  ```

**注意**：例如想要给Nginx模块的fileset添加一个参数`close_eof: true`，可使用以下参数

- 配置文件

  ```yaml
  - module: nginx
    access:
      input:
        close_eof: true
  ```

- 命令行

  ```bash
  filebeat -e --modules nginx -M "*.*.input.close_eof=true" 
  # 或者
  filebeat -e --modules nginx -M "nginx.*.input.close_eof=true"
  # 或者
  filebeat -e --modules nginx -M "nginx.access.input.close_eof=true"
  ```
