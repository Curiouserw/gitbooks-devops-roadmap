相关链接
官方文档： https://github.com/kubernetes-incubator/external-storage/tree/master/ceph/cephfs
Provisioner的定义原理：Kubernetes的存储--> StorageClass provisioner
姊妹篇：

# Preflight

## 1. Openshift创建cephfs命名空间

```bash
oc new-project cephfs --display-name="Ceph FileSystem Provisioner"
```

## 2. 拉取镜像

```bash
docker pull quay-mirror.qiniu.com/external_storage/cephfs-provisioner:latest && \
docker tag quay-mirror.qiniu.com/external_storage/cephfs-provisioner:latest quay.io/external_storage/cephfs-provisioner:latest && \
docker rmi quay-mirror.qiniu.com/external_storage/cephfs-provisioner:latest
```

# 一、安装部署

## 1. 获取Ceph Filesystem Client.admin用户的密钥环

```yaml
ceph auth get client.admin
# [client.admin]
#        key = AQCinINcLykNLhAA7Xr6o+Q2jYeyc5j58JeQeQ==
#        caps mds = "allow *"
#        caps mon = "allow *"
#        caps osd = "allow *"
```

## 2. 创建Secrets

```bash
oc create secret generic cephfs-secret-admin --from-literal=key='AQCinINcLykNLhAA7Xr6o+Q2jYeyc5j58JeQeQ==' --namespace=cephfs
```

## 3. 创建RBAC

```yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: cephfs-provisioner
  namespace: cephfs
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: cephfs-provisioner
  namespace: cephfs
rules:
  - apiGroups: [""]
    resources: ["secrets"]
    verbs: ["create", "get", "delete"]
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: cephfs-provisioner
  namespace: cephfs
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: cephfs-provisioner
subjects:
- kind: ServiceAccount
  name: cephfs-provisioner
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: cephfs-provisioner
  namespace: cephfs
rules:
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["create", "update", "patch"]
  - apiGroups: [""]
    resources: ["services"]
    resourceNames: ["kube-dns","coredns"]
    verbs: ["list", "get"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: cephfs-provisioner
subjects:
  - kind: ServiceAccount
    name: cephfs-provisioner
    namespace: cephfs
roleRef:
  kind: ClusterRole
  name: cephfs-provisioner
  apiGroup: rbac.authorization.k8s.io
```

## 4. 使用Deployment创建Ceph-FileSystem-provisioner的POD

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: cephfs-provisioner
  namespace: cephfs
spec:
  replicas: 1
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: cephfs-provisioner
    spec:
      containers:
      - name: cephfs-provisioner
        image: "quay.io/external_storage/cephfs-provisioner:latest"
        env:
        - name: PROVISIONER_NAME
          value: ceph.com/cephfs
        - name: PROVISIONER_SECRET_NAMESPACE
          value: cephfs
        command:
        - "/usr/local/bin/cephfs-provisioner"
        args:
        - "-id=cephfs-provisioner-1"
      serviceAccount: cephfs-provisioner
```

# 二、使用

## 1. 创建StorageClass

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: cephfs
provisioner: ceph.com/cephfs
parameters:
    monitors: allinone.okd311.curiouser.com:6789
    adminId: admin
    adminSecretName: cephfs-secret-admin
    adminSecretNamespace: "cephfs"
    claimRoot: /pvc-volumes
```

## 2、创建PVC时使用

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: cephfs-test
spec:
  storageClassName: cephfs
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```