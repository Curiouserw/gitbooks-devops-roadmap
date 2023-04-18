# Filebeat的多实例部署

# 一、上下文

一台服务器为了充分利用资源使用，通常安装了多个系统的组件。例如既是MySQL集群的节点，又是Redis集群的节点。如果该服务器之前已经部署了一个FIlebeat实例用来采集MySQL的慢查询日志，输出日志到指定的Logstash进行处理。而此时，有需求要收集该服务器上另外一个系统的组件日志数据。FIlebeat的配置中无法使用条件判断设置多个不同的输出目的地。此时可以直接部署多实例的filebeat。

# 二、部署配置

以采集API网关Kong的日志为例.

1. 创建新的filebeat配置文件`/etc/filebeat/filebeat-kong.yml`

2. 创建Filebeat系统服务启动配置文件 `/usr/lib/systemd/system/filebeat-kong.service`

   ```properties
   [Unit]
   Description=Filebeat sends kong log files to Logstash or directly to Elasticsearch.
   Documentation=https://www.elastic.co/products/beats/filebeat
   Wants=network-online.target
   After=network-online.target

   [Service]

   Environment="BEAT_LOG_OPTS=-e"
   # 指定Filebeat配置文件
   Environment="BEAT_CONFIG_OPTS=-c /etc/filebeat/filebeat-kong.yml" 
   # 不同实例filebeat的'path.data'一定要不一样
   Environment="BEAT_PATH_OPTS=-path.home /usr/share/filebeat -path.config /etc/filebeat -path.data /var/lib/filebeat-kong -path.logs /var/log/filebeat-kong"
   ExecStart=/usr/share/filebeat/bin/filebeat $BEAT_LOG_OPTS $BEAT_CONFIG_OPTS $BEAT_PATH_OPTS
   Restart=always

   [Install]
   WantedBy=multi-user.target
   ```

3. 启动filebeat服务

   ```bash
   sudo systemctl daemon-reload
   sudo systemctl start filebeat-kong.service
   sudo systemctl status filebeat-kong.service -l
   ```
