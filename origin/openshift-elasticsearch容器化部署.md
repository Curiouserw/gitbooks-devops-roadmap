# 一、拉取镜像

```shell
docker pull docker.io/elasticsearch/elasticsearch:6.6.1
#或者
docker pull docker.elastic.co/elasticsearch/elasticsearch:6.6.1
```

# 二、Docker部署

#### 修改系统

```shell
echo "vm.max_map_count=262144" >> /etc/sysctl.conf
sysctl -w vm.max_map_count=262144
```

#### Docker单节点部署

```shell
docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.io/elasticsearch/elasticsearch:6.6.1
```

#### Docker compose集群部署

```yaml
version: '2.2'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.6.1
    container_name: elasticsearch
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata1:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - esnet
  elasticsearch2:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.6.1
    container_name: elasticsearch2
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - "discovery.zen.ping.unicast.hosts=elasticsearch"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata2:/usr/share/elasticsearch/data
    networks:
      - esnet

volumes:
  esdata1:
    driver: local
  esdata2:
    driver: local
networks:
  esnet:
```

# 三、OKD上部署

## DeploymentConfig

```yaml
apiVersion: apps.openshift.io/v1
kind: DeploymentConfig
metadata:
  labels:
    app: elasticsearch
  name: elasticsearch
spec:
  replicas: 1
  selector:
    app: elasticsearch
    deploymentconfig: elasticsearch
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: elasticsearch
        deploymentconfig: elasticsearch
    spec:
      containers:
        - env:
            - name: discovery.type
              value: single-node
            - name: cluster.name
              value: curiouser
            - name: bootstrap.memory_lock
              value: 'true'
            - name: path.repo
              value: /usr/share/elasticsearch/snapshots-repository
            - name: TZ
              value: Asia/Shanghai
            - name: ES_JAVA_OPTS
              value: '-Xms1g -Xmx2g'
            - name: xpack.monitoring.collection.enabled
              value: 'true'
            - name: xpack.security.enabled
              value: 'true'
            - name: ELASTIC_USERNAME
              value: "elastic"
            - name: "ELASTIC_PASSWORD"
              value: "elastic"  
          image: 'docker.elastic.co/elasticsearch/elasticsearch:7.1.1'
          imagePullPolicy: IfNotPresent
          livenessProbe:
            failureThreshold: 3
            initialDelaySeconds: 90
            periodSeconds: 10
            successThreshold: 1
            tcpSocket:
              port: 9200
            timeoutSeconds: 1
          name: elasticsearch
          ports:
            - containerPort: 9200
              protocol: TCP
            - containerPort: 9300
              protocol: TCP
          readinessProbe:
            failureThreshold: 3
            initialDelaySeconds: 80
            periodSeconds: 10
            successThreshold: 1
            tcpSocket:
              port: 9200
            timeoutSeconds: 1
          resources:
            limits:
              cpu: '2'
              memory: 3Gi
            requests:
              cpu: '1'
              memory: 2Gi
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          volumeMounts:
            - mountPath: /usr/share/elasticsearch/data
              name: elasticsearch-data
            - mountPath: /usr/share/elasticsearch/snapshots-repository
              name: elasticsearch-snapshots-repository
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      volumes:
        - name: elasticsearch-data
          persistentVolumeClaim:
            claimName: elasticsearch-data
        - name: elasticsearch-snapshots-repository
          persistentVolumeClaim:
            claimName: elasticsearch-snapshots-repository
  test: false
  triggers:
    - type: ConfigChange
```

## SVC

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: elasticsearch
  name: elasticsearch
spec:
  ports:
  - name: 9200-tcp
    port: 9200
    protocol: TCP
    targetPort: 9200
  - name: 9300-tcp
    port: 9300
    protocol: TCP
    targetPort: 9300
  selector:
    deploymentconfig: elasticsearch
  sessionAffinity: None
  type: ClusterIP
```

## 数据目录PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  annotations:
    volume.beta.kubernetes.io/storage-class: nfs-client-storageclass
  name: elasticsearch-data
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
```

## snapshot repository存储目录PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  annotations:
    volume.beta.kubernetes.io/storage-class: nfs-client-storageclass
  name: elasticsearch-snapshots-repository
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
```

# 四. Kubernetes部署

## Deployment

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  labels:
    elastic-app: elasticsearch
    role: master
  name: elasticsearch-master
  namespace: elk
spec:
  replicas: 1
  revisionHistoryLimit: 10
  strategy:
    type: Recreate
  selector:
    matchLabels:
      elastic-app: elasticsearch
      role: master
  template:
    metadata:
      labels:
        elastic-app: elasticsearch
        role: master
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: kubernetes.io/hostname
                    operator: In
                    values:
                      - node1.k8s.curiouser.com
      initContainers:
        - name: init-scheduler
          image: busybox:latest
          imagePullPolicy: IfNotPresent
          command: ['sh', '-c', 'chmod -R 777 /usr/share/elasticsearch/data /usr/share/elasticsearch/snapshots-repository && chown -R 1000.0 /usr/share/elasticsearch/data /usr/share/elasticsearch/snapshots-repository']
          volumeMounts:
            - name: elasticsearch-data
              mountPath: /usr/share/elasticsearch/data
            - name: elasticsearch-snapshots-repository
              mountPath: /usr/share/elasticsearch/snapshots-repository
      containers:
        - name: elasticsearch-master-data
          image: docker.elastic.co/elasticsearch/elasticsearch:7.2.0
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 9200
              protocol: TCP
            - containerPort: 9300
              protocol: TCP
          env:
            - name: "cluster.name"
              value: "Curiouser"
            - name: "bootstrap.memory_lock"
              value: "false"
            - name: discovery.type
              value: single-node
            - name: "node.master"
              value: "true"
            - name: "node.data"
              value: "true"
            - name: "node.ingest"
              value: "false"
            - name: xpack.monitoring.collection.enabled
              value: "true"
            - name: "xpack.monitoring.elasticsearch.collection.enabled"
              value: "true"
            - name: "xpack.security.enabled"
              value: "true"
            - name: "path.repo"
              value: "/usr/share/elasticsearch/snapshots-repository"
            - name: "ES_JAVA_OPTS"
              value: "-Xms2048m -Xmx2048m"
            - name: TZ
              value: Asia/Shanghai
            - name: "xpack.monitoring.exporters.my_local.type"
              value: "local"
            - name: "xpack.monitoring.exporters.my_local.use_ingest"
              value: "false"
          resources:
            requests:
              memory: "2Gi"
              cpu: "2"
            limits:
              memory: "4096Mi"
              cpu: "3"
          readinessProbe:
            failureThreshold: 1
            initialDelaySeconds: 60
            periodSeconds: 60
            successThreshold: 1
            tcpSocket:
              port: 9200
            timeoutSeconds: 1
          livenessProbe:
            failureThreshold: 1
            initialDelaySeconds: 60
            periodSeconds: 60
            successThreshold: 1
            tcpSocket:
              port: 9200
            timeoutSeconds: 1
          volumeMounts:
            - name: elasticsearch-data
              mountPath: "/usr/share/elasticsearch/data"
            - name: elasticsearch-snapshots-repository
              mountPath: "/usr/share/elasticsearch/snapshots-repository"
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      securityContext: {}
      schedulerName: default-scheduler
      volumes:
        - name: elasticsearch-data
          persistentVolumeClaim:
            claimName: elasticsearch-data
        - name: elasticsearch-snapshots-repository
          persistentVolumeClaim:
            claimName: elasticsearch-snapshots-repository
```

## PersistentVolumeClaim

```yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  annotations:
    volume.beta.kubernetes.io/storage-class: cephfs
  labels:
    app: elasticsearch
  name: elasticsearch-data
  namespace: elk
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 20Gi
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  annotations:
    volume.beta.kubernetes.io/storage-class: cephfs
  labels:
    app: elasticsearch
  name: elasticsearch-snapshots-repository
  namespace: elk
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 30Gi
```

## Service

```yaml
kind: Service
apiVersion: v1
metadata:
  labels:
    elastic-app: elasticsearch-service
  name: elasticsearch
  namespace: elk
spec:
  ports:
    - port: 9200
      targetPort: 9200
      protocol: TCP
  selector:
    elastic-app: elasticsearch
  type: ClusterIP
```


