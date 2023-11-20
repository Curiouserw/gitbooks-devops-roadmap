# 部署ELK

# 一、Docker 部署

## 1、Elasticsearch

**镜像信息**

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

**Docker Compose**

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

**Ansible二进制脚本部署GitHub地址**：https://github.com/elastic/ansible-elasticsearch

## 2、Kibana

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

## 3、Logstash

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

# 二、Helm 部署

## 1、部署单个组件

```bash
helm repo add elastic https://helm.elastic.co && \
helm repo update && \

helm upgrade --install \
  --version 7.17.3 \
  --namespace logging \
  elasticsearch-logging elastic/elasticsearch \
  --set cluster-name='elasticsearch-logging' \
  --set tests.enabled=false \
  --set replicas=1 \
  --set minimumMasterNodes=1 
```

## 2、安装CRD部署各组件

官方文档：https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-overview.html

ECK(Elastic Clound Kubernetes)支持的组件版本:

- Kubernetes 1.24-1.28
- OpenShift 4.9-4.13
- Google Kubernetes Engine (GKE), Azure Kubernetes Service (AKS), and Amazon Elastic Kubernetes Service (EKS)
- Helm: 3.2.0+
- Elasticsearch, Kibana, APM Server: 6.8+, 7.1+, 8+
- Enterprise Search: 7.7+, 8+
- Beats: 7.0+, 8+
- Elastic Agent: 7.10+ (standalone), 7.14+ (Fleet), 8+
- Elastic Maps Server: 7.11+, 8+
- Logstash: 8.7+

### ①安装ECS Operator

```bash
helm repo add elastic https://helm.elastic.co 
helm repo update 

helm upgrade --install --atomic \
  elastic-operator elastic/eck-operator \
  -n kube-system \
  --set managedNamespaces='{logging}' \
  --set telemetry.disabled=true 
```

**安装的 CRD：**

- agents.agent.k8s.elastic.co

- apmservers.apm.k8s.elastic.co

- beats.beat.k8s.elastic.co

- elasticmapsservers.maps.k8s.elastic.co

- elasticsearchautoscalers.autoscaling.k8s.elastic.co

- elasticsearches.elasticsearch.k8s.elastic.co

- enterprisesearches.enterprisesearch.k8s.elastic.co

- kibanas.kibana.k8s.elastic.co

- logstashes.logstash.k8s.elastic.co

- stackconfigpolicies.stackconfigpolicy.k8s.elastic.co

**创建的k8s资源**

- rbac资源
  - serviceaccount：elastic-operator
  - secret：elastic-webhook-server-cert"
  - clusterrole：elastic-operator、elastic-operator-view、elastic-operator-edit
  - clusterrolebinding：elastic-operator
- configmap：elastic-operator
- sts：elastic-operator
- service ：elastic-webhook-server
- admissionregistration：elastic-webhook.k8s.elastic.co

### ②使用CRD部署各组件

```bash
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Namespace
metadata:
  name: logging
---
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: logging
  namespace: logging
spec:
  version: 7.17.3
  nodeSets:
  - name: es
    count: 1
    config:
      xpack.security.enabled: true
      thread_pool.snapshot.max: 8
      path.repo: ["/elasticsearch-snapshots-repo"]
      node.store.allow_mmap: false
    volumeClaimTemplates:
      - metadata:
          name: elasticsearch-snapshots-repo
        spec:
          accessModes:
          - ReadWriteOnce
          resources:
            requests:
              storage: 100Gi
          storageClassName: local-nfs-storage
      - metadata:
          name: elastic-internal-elasticsearch-plugins-local
        spec:
          accessModes:
          - ReadWriteOnce
          resources:
            requests:
              storage: 10Gi
          storageClassName: local-nfs-storage
      - metadata:
          name: elasticsearch-logs
        spec:
          accessModes:
          - ReadWriteOnce
          resources:
            requests:
              storage: 20Gi
          storageClassName: local-nfs-storage
      - metadata:
          name: elasticsearch-data
        spec:
          accessModes:
          - ReadWriteOnce
          resources:
            requests:
              storage: 50Gi
          storageClassName: local-nfs-storage    
    podTemplate:
      spec:
        volumes:
          - name: elasticsearch-snapshots-repo
            persistentVolumeClaim:
              claimName: elasticsearch-snapshots-repo         
        initContainers:
        - name: install-plugins
          command:
          - sh
          - -c
          - |
            bin/elasticsearch-plugin install --batch analysis-icu analysis-smartcn
        containers:
        - name: elasticsearch
          volumeMounts:
          - name: elasticsearch-snapshots-repo
            mountPath: /elasticsearch-snapshots-repo
          env:
          - name: ES_JAVA_OPTS
            value: -Xms2g -Xmx2g
          resources:
            requests:
              memory: 1Gi
              cpu: 1
            limits:
              memory: 3Gi
              cpu: 2
---
apiVersion: kibana.k8s.elastic.co/v1
kind: Kibana
metadata:
  name: logging
  namespace: logging
spec:
  version: 7.17.3
  count: 1
  config:
    i18n.locale: "zh-CN"
  elasticsearchRef:
    name: logging
  http:
    tls:
      selfSignedCertificate:
        disabled: true
  podTemplate:
      spec:
        containers:
        - name: kibana
          env:
            - name: NODE_OPTIONS
              value: "--max-old-space-size=2048"
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: logging-kibana
  namespace: logging
spec:
  rules:
  - host: "kibana.test.com"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: logging-kb-http
            port:
              number: 5601
EOF
```

### ③验证

- 查看es检查状态

  ```bash
  kubectl exec -it logging-es-es-0 -c elasticsearch -- /bin/bash -c 'BASIC_AUTH_PSW=`cat $PROBE_PASSWORD_PATH`
  curl -XGET  -s -k -u "$PROBE_USERNAME:$BASIC_AUTH_PSW" "https://127.0.0.1:9200/"'
  ```

- 访问kibana：http://kibana.test.com

  获取kibana的elastic账号密码：

  ```bash
  es_instance=`kubectl get elasticsearch |grep -v NAME |awk '{print $1}'`
  kubectl get secrets ${es_instance}-es-elastic-user
  ```

  使用 curl 查看 Kibana 登录页服务状态

  ```bash
  ingress_controller_node=`kubectl -n kube-system get pod -l app.kubernetes.io/name=traefik -ojson | jq -r '.items[0] | .status.hostIP'`
  curl -s --noproxy "kibana.test.com" --resolve kibana.test.com:80:$ingress_controller_node http://kibana.test.com/login
  ```
