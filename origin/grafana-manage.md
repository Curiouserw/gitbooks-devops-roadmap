# Grafana管理

# 一、简介

## 内置可配数据源

- [Alertmanager](https://grafana.com/docs/grafana/latest/datasources/alertmanager/)

- [AWS CloudWatch](https://grafana.com/docs/grafana/latest/datasources/aws-cloudwatch/)

- [Azure Monitor](https://grafana.com/docs/grafana/latest/datasources/azure-monitor/)

- [Elasticsearch](https://grafana.com/docs/grafana/latest/datasources/elasticsearch/)

- [Google Cloud Monitoring](https://grafana.com/docs/grafana/latest/datasources/google-cloud-monitoring/)

- [Graphite](https://grafana.com/docs/grafana/latest/datasources/graphite/)

- [InfluxDB](https://grafana.com/docs/grafana/latest/datasources/influxdb/)

- [Jaeger](https://grafana.com/docs/grafana/latest/datasources/jaeger/)

- [Loki](https://grafana.com/docs/grafana/latest/datasources/loki/)

- [Microsoft SQL Server (MSSQL)](https://grafana.com/docs/grafana/latest/datasources/mssql/)

- [MySQL](https://grafana.com/docs/grafana/latest/datasources/mysql/)

- [OpenTSDB](https://grafana.com/docs/grafana/latest/datasources/opentsdb/)

- [PostgreSQL](https://grafana.com/docs/grafana/latest/datasources/postgres/)

- [Prometheus](https://grafana.com/docs/grafana/latest/datasources/prometheus/)

- [Pyroscope](https://grafana.com/docs/grafana/latest/datasources/pyroscope/)

- [Tempo](https://grafana.com/docs/grafana/latest/datasources/tempo/)

- [Testdata](https://grafana.com/docs/grafana/latest/datasources/testdata/)

- [Zipkin](https://grafana.com/docs/grafana/latest/datasources/zipkin/)

  





# 二、告警处理



# 三、provisioning集成默认配置

Granfa集成配置可在 grafana配置文件 grafana.ini中设置其路径`;provisioning = conf/provisioning` 

## 1、集成默认数据源

例如集成默认 Loki 数据源，创建文件`loki.yaml`

```yaml
apiVersion: 1
datasources:
  - name: Loki
    type: loki
    access: proxy
    editable: true
    url: http://localhost:3100
    jsonData:
      tlsSkipVerify: true
      httpHeaderName1: "X-Scope-OrgID"
    secureJsonData:
      httpHeaderValue1: "fake"
```

## 2、集成仪表面板配置

## 3、集成插件

参考

- https://grafana.com/docs/grafana/latest/administration/provisioning
- https://grafana.com/docs/grafana/latest/setup-grafana/configure-grafana/#provisioning