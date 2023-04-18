# Openshift容器日志系统EFK的管理

Openshift 3.11如果是使用官方ansible playbook安装的话，可配置安装EFK来收集存储分析Openshift上所有容器的日志。
Fluentd以Daemonset的形式部署在所有Node节点上，监控采集docker容器目录/var/docker/container（每个容器的日志默认都会以 json-file 的格式存储于 /var/lib/docker/containers/<容器id>/<容器id>-json.log 下）.
将采集的日志数据存放在elasticsearch中。并配置了一个Cronjob定时清理elasticsearch中指定过期的Index。

1. 查看所有的Index
    
    ```bash
    oc exec $es-pod-name -- curl -s --cert /etc/elasticsearch/secret/admin-cert --key /etc/elasticsearch/secret/admin-key --cacert /etc/elasticsearch/secret/admin-ca https://localhost:9200/_cat/indices?v
    ```

2. 清理所有3月份的ES索引

    ```bash
    oc exec $es-pod-name -- curl -s --cert /etc/elasticsearch/secret/admin-cert --key /etc/elasticsearch/secret/admin-key --cacert /etc/elasticsearch/secret/admin-ca -XDELETE https://localhost:9200/*.2018.03.*
    ```

3. 通过环境变量设置Cronjob定时清理超过三天的Index，只保存三天的Index
    
    ```bash
    oc edit cj logging-curator -n openshift-logging 
    ```

    相关环境变量:
    - K8S_HOST_URL: https://kubernetes.default.svc.cluster.local
    - ES_HOST: logging-es
    - ES_PORT: "9200"
    - ES_CLIENT_CERT: /etc/curator/keys/cert
    - ES_CLIENT_KEY: /etc/curator/keys/key
    - ES_CA: /etc/curator/keys/ca
    - CURATOR_DEFAULT_DAYS: "3"
    - CURATOR_SCRIPT_LOG_LEVEL: INFO
    - CURATOR_LOG_LEVEL: ERROR
    - CURATOR_TIMEOUT: "300"

# 参考连接
1. https://docs.openshift.com/container-platform/3.11/install_config/aggregate_logging.html#aggregate-logging-understanding-the-deployment
2. https://hub.docker.com/r/openshift/origin-logging-curator5
3. https://github.com/openshift/origin-aggregated-logging
