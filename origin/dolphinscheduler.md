# DolphinScheduler

# 一、简介

文档：https://dolphinscheduler.apache.org/en-us/docs/3.1.8

Github：https://github.com/apache/dolphinscheduler

## 依赖

- postgresql/mysql
- zookeeper

# 二、helm部署至k8s

## 1、安装Postgresql

```bash
helm upgrade --install postgresql-for-dolphinscheduler \
  --namespace tools \
  --version 13.1.5 \
  bitnami/postgresql \
  --set global.postgresql.auth.postgresPassword=***postgres用户的密码*** \
  --set global.postgresql.auth.username=***另外创建的用户名***  \
  --set global.postgresql.auth.password=***另外创建的用户密码*** \
  --set global.postgresql.auth.database=***另外创建的Database*** 
```

## 2、下载charts

```bash
dolphinscheduler_version=3.1.8 && \
curl -s https://dlcdn.apache.org/dolphinscheduler/$dolphinscheduler_version/apache-dolphinscheduler-$dolphinscheduler_version-src.tar.gz | tar -zx -C . apache-dolphinscheduler-$dolphinscheduler_version-src/deploy/kubernetes/dolphinscheduler && \
cd apache-dolphinscheduler-$dolphinscheduler_version-src/deploy/kubernetes/dolphinscheduler
helm repo add bitnami https://charts.bitnami.com/bitnami
helm dependency update .
```

## 3、推送所需镜像到镜像仓库

```bash
for i in {master,tools,alert-server};do
	docker pull dolphinscheduler.docker.scarf.sh/apache/dolphinscheduler-$i:3.1.8 && \
	docker tag dolphinscheduler.docker.scarf.sh/apache/dolphinscheduler-$i:3.1.8 hub.test.com/dolphinscheduler/dolphinscheduler-$i:3.1.8  && \
	docker push hub.test.com/dolphinscheduler/dolphinscheduler-$i:3.1.8
done
```

## 4、定制镜像

**定制镜像是为了支持**

- **Python 3 、Pip3**
  - **需要在dolphinscheduler-worker服务容器中安装 Python 3**
- **在数据源集成中支持连接MySQL、Oracle**
  - **需要在dolphinscheduler-worker, dolphinscheduler-api服务容器中的/opt/dolphinscheduler/libs/路径下放置对应的驱动 jar**

### ①定制dolphinscheduler-api镜像

> Dockerfile

```bash
FROM dolphinscheduler.docker.scarf.sh/apache/dolphinscheduler-api:3.1.8

RUN wget -q https://repo1.maven.org/maven2/mysql/mysql-connector-java/8.0.16/mysql-connector-java-8.0.16.jar -O /opt/dolphinscheduler/libs/mysql-connector-java-8.0.16.jar && \
  sed -i -e 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' -e 's/security.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list && \
    apt-get update && \
    rm -rf /var/lib/apt/lists/*
```

> docker build --rm -f Dockerfile -t hub.test.com/dolphinscheduler/dolphinscheduler-api:3.1.8 .
> docker push hub.test.com/dolphinscheduler/dolphinscheduler-api:3.1.8

### ②定制dolphinscheduler-worker镜像

>  Dockerfile

```bash
FROM dolphinscheduler.docker.scarf.sh/apache/dolphinscheduler-worker:3.1.8

RUN wget -q https://repo1.maven.org/maven2/mysql/mysql-connector-java/8.0.16/mysql-connector-java-8.0.16.jar -O /opt/dolphinscheduler/libs/mysql-connector-java-8.0.16.jar && \
  sed -i -e 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' -e 's/security.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends python3 python3-pip && \
    pip3 install --no-cache-dir -i https://mirrors.aliyun.com/pypi/simple/ pandas PyMySQL SQLAlchemy xlwt xlsxwriter pypinyin eventlet && \
    rm -rf /var/lib/apt/lists/*
```

> docker build --rm -f Dockerfile -t hub.test.com/dolphinscheduler/dolphinscheduler-worker:3.1.8 .
> docker push hub.test.com/dolphinscheduler/dolphinscheduler-worker:3.1.8

## 5、部署

```bash
helm upgrade --install dolphinscheduler . -n tools \
  --set image.registry="harbor.test.com/dolphinscheduler" \
  --set image.pullSecret=pull-harbor \
  --set postgresql.enabled=false \
  --set common.sharedStoragePersistence.enabled=true \
  --set common.sharedStoragePersistence.storageClassName=local-nfs-client \
  --set common.fsFileResourcePersistence.enabled=true \
  --set common.fsFileResourcePersistence.storageClassName=local-nfs-client \
  --set common.configmap.FS_DEFAULT_FS="file:///" \
  --set externalDatabase.host=postgresql-for-dolphinscheduler.tools.svc \
  --set externalDatabase.username=***pg的dolphinscheduler使用的用户名*** \
  --set externalDatabase.password=***pg的dolphinscheduler使用的用户密码*** \
  --set master.persistentVolumeClaim.enabled=true \
  --set master.persistentVolumeClaim.storageClassName=local-nfs-client \
  --set worker.persistentVolumeClaim.enabled=true \
  --set worker.persistentVolumeClaim.dataPersistentVolume.enabled=true \
  --set worker.persistentVolumeClaim.dataPersistentVolume.storageClassName=local-nfs-client \
  --set worker.persistentVolumeClaim.logsPersistentVolume.enabled=true \
  --set worker.persistentVolumeClaim.logsPersistentVolume.storageClassName=local-nfs-client \
  --set api.persistentVolumeClaim.enabled=true \
  --set api.persistentVolumeClaim.storageClassName=local-nfs-client \
  --set ingress.enabled=true \
  --set ingress.host="dolphinscheduler.test.com"
```

为了解决部署后无法创建租户的问题，修改部署后的 Configmap: dolphinscheduler-configs。然后重启 dolphinscheduler-master、dolphinscheduler-api、dolphinscheduler-worker

```bash
data:
   common_properties: |-
     ...
     resource.hdfs.fs.defaultFS=file:///
     ...
```

问题具体原因参考：

- https://dolphinscheduler.apache.org/en-us/docs/3.1.8/guide/installation/kubernetes#:~:text=Support%20Matrix-,FAQ,-Appendix%2DConfiguration
- https://blog.csdn.net/orchidofocean/article/details/132272168

# 三、配置 

## 1、配置K8S集群

### ①创建 K8S系统账号

权限

- 不能创建 NS
- 可以创建 job、pod

```bash

kubectl create role dolphinscheduler-excutor --verb=create,get,list,watch ---resource-name=jobs --resource=batch

kubectl create rolebinding dolphinscheduler-excutor-binding --clusterrole=dolphinscheduler-excutor --user=dolphinscheduler-excutor --namespace=dolphinscheduler-workspace

```



```bash
cat <<EOF | kubectl create -f -
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: dolphinscheduler-workspace
  name: dolphinscheduler-excutor-role
rules:
- apiGroups: ["batch"]
  resources: ["jobs"]
  verbs: ["create", "get", "list"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: dolphinscheduler-excutor-rolebinding
  namespace: dolphinscheduler-workspace
subjects:
- kind: User
  name: dolphinscheduler-excutor
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: dolphinscheduler-excutor-role
  apiGroup: rbac.authorization.k8s.io
EOF
```

- serviceaccount/dolphinscheduler-excutor
- clusterrole.rbac.authorization.k8s.io/dolphinscheduler-excutor-role
- rolebinding.rbac.authorization.k8s.io/dolphinscheduler-excutor
- clusterrolebinding.rbac.authorization.k8s.io/dolphinscheduler-excutor





