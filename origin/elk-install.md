# 部署ELK

# 一、Elasticsearch

## Docker

镜像信息

- Docker Hub：https://hub.docker.com/_/elasticsearch
- 官方文档：https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html
- 数据目录：/usr/share/elasticsearch/data

```bash
docker run -d \
  --name elasticsearch \
  -p 9200:9200 \
  -e TZ=Asia/Shanghai \
  -e "cluster.name=docker-desktop" \
  -e "bootstrap.memory_lock=true" \
  -e "discovery.type=single-node" \
  -e ES_JAVA_OPTS="-Xms2g -Xmx2g" \
  -e "xpack.monitoring.collection.enabled=true" \
  -e "xpack.security.authc.api_key.enabled=true" \
  -e "xpack.security.enabled=true" \
  -e ELASTIC_PASSWORD=Curiouser \
  elasticsearch:7.10.1
```

## Docker Compose

```bash
version: '2.2'
services:
  es01:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.5.2
    container_name: es01
    environment:
      - node.name=es01
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es02,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data01:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - elastic
  es02:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.5.2
    container_name: es02
    environment:
      - node.name=es02
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data02:/usr/share/elasticsearch/data
    networks:
      - elastic
  es03:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.5.2
    container_name: es03
    environment:
      - node.name=es03
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es02
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data03:/usr/share/elasticsearch/data
    networks:
      - elastic

volumes:
  data01:
    driver: local
  data02:
    driver: local
  data03:
    driver: local

networks:
  elastic:
    driver: bridge
```



## Ansible二进制

Ansible脚本GitHub地址：https://github.com/elastic/ansible-elasticsearch





# 二、Kibana

## Docker

```bash
docker run -d \
  --name kibana \
  --link elasticsearch:elasticsearch \
  -p 5601:5601 \
  -e TZ=Asia/Shanghai \
  -e ELASTICSEARCH_USERNAME=elastic \
  -e ELASTICSEARCH_PASSWORD=Curiouser \
  -e I18N_LOCALE=zh-CN \
  -e XPACK_SECURITY_ENABLED=TRUE \
  -e XPACK_ENCRYPTEDSAVEDOBJECTS_ENCRYPTIONKEY=ZjdlNDE1ZjJiM2M4ZGI0MjdkZDRlYzQ0 \
  -e XPACK_SECURITY_ENABLED=true \
  -e XPACK_SECURITY_AUTHC_API_KEY_ENABLED=true \
  kibana:7.10.1
```



# 三、Logstash

## Docker

```bash
docker run -d \
  --name logstash \
  --link elasticsearch:elasticsearch \
  -p 9600:9600 \
  -p 5044:5044 \
  -e TZ=Asia/Shanghai \
  -e XPACK_MONITORING_ELASTICSEARCH_USERNAME=elastic \
  -e XPACK_MONITORING_ELASTICSEARCH_PASSWORD=Curiouser \
  -e MONITORING_ENABLED=true \
  -v ~/logstash.conf:/usr/share/logstash/pipeline/logstash.conf \
  logstash:7.5.1
```



# 四、其他

## 验证

```bash
curl -u elastic:Curiouser http://127.0.0.1:9200/_cat/nodes?v
```

