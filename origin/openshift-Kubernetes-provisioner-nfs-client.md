# 一、NFS Client Provisioner
https://github.com/kubernetes-incubator/external-storage/tree/master/nfs-client

https://www.kubernetes.org.cn/3894.html


Provisioner的定义原理: [openshift-Kubernetes的持久化存储](/origin/openshift-Kubernetes的持久化存储.md)


# 二、安装部署

## 1. 创建NFS服务端

```bash
yum install -y nfs-utils rpcbind && \
systemctl enable nfs && \
systemctl enable rpcbind && \
systemctl start nfs && \
systemctl start rpcbind && \
mkdir -p /data/nfs/appstorage-nfs-client-provisioner && \
echo "/data/nfs/appstorage-nfs-client-provisioner *(rw,no_root_squash,sync)" >> /etc/exports && \
exportfs -a && \
showmount -e $HOSTNAME
```

## 2. 创建RBAC

```yaml
kind: ServiceAccount
apiVersion: v1
metadata:
  name: nfs-client-provisioner
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-client-provisioner-runner
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
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    namespace: default
roleRef:
  kind: ClusterRole
  name: nfs-client-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
rules:
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    # replace with namespace where provisioner is deployed
    namespace: default
roleRef:
  kind: Role
  name: leader-locking-nfs-client-provisioner
  apiGroup: rbac.authorization.k8s.io
```

## 3. 修改Deployment并以此部署POD

先拉取镜像

```bash
docker pull quay-mirror.qiniu.com/external_storage/nfs-client-provisioner:latest && \
docker tag quay-mirror.qiniu.com/external_storage/nfs-client-provisioner:latest quay.io/external_storage/nfs-client-provisioner:latest && \
docker rmi quay-mirror.qiniu.com/external_storage/nfs-client-provisioner:latest
```

```yaml
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: nfs-client-provisioner
spec:
  replicas: 1
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          image: quay.io/external_storage/nfs-client-provisioner:latest
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: fuseim.pri/ifs
            - name: NFS_SERVER
              value: $HOSTNAME # NFS Server的地址
            - name: NFS_PATH
              value: /data/nfs/appstorage-nfs-client-provisioner # NFS Server要挂载的路径
      volumes:
        - name: nfs-client-root
          nfs:
            server: $HOSTNAME #指定NFS Server的地址
            path: /data/nfs/appstorage-nfs-client-provisioner #指定NFS Server要挂载的路径
```

# 三、使用

## 1. 创建StorageClass

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-client-storageclass
provisioner: fuseim.pri/ifs # or choose another name, must match deployment's env PROVISIONER_NAME'
parameters:
  archiveOnDelete: "false"  # When set to "false" your PVs will not be archived by the provisioner upon deletion of the PVC.

=======================================================补充内容=========================================================
#如果要将此storageclass设置为默认，在metadata里面添加以下注解。（这样创建PVC时就可以不用特意指定StorageClass）
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations: 
    "storageclass.kubernetes.io/is-default-class": "true"
  name: nfs-client-storageclass
provisioner: fuseim.pri/ifs # or choose another name, must match deployment's env PROVISIONER_NAME'
parameters:
  archiveOnDelete: "false"  # "false" 删除PVC时不会保留数据，"true"将保留PVC的数据，形成以"archived-"开头的文件夹
```

## 2. 创建PVC时使用

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: test-pvc
  #当默认storageclass就是nfs-client-storageclass，可不要该注解
  annotations:
    volume.beta.kubernetes.io/storage-class: "nfs-client-storageclass"
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Mi
```

# 四、测试

## 1. 创建一个PVC

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: test
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 100Mi
​
#========================================================================================================================
$ kubectl get pvc
NAME   STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS              AGE
test   Bound    pvc-e8a15786-5a09-11e9-ad53-000c296286d8   100Mi      RWX            nfs-client-storageclass   10m
$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM          STORAGECLASS              REASON   AGE
pvc-e8a15786-5a09-11e9-ad53-000c296286d8   100Mi      RWX            Delete           Bound    default/test   nfs-client-storageclass            10m
```

## 2. 创建一个POD使用PVC

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: counter
spec:
  containers:
  - name: count
    image: busybox
    args:
    - /bin/sh
    - -c
    - >
      i=0;
      while true;
      do
        echo "$i: $(date)" >> /var/log/1.log;
        echo "$(date) INFO $i" >> /var/log/2.log;
        i=$((i+1));
        sleep 1;
      done
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  volumes:
    - name: varlog
      persistentVolumeClaim:
        claimName: test
```

## 3. 查看NFS目录

```bash
/data/nfs/k8s-app-nfs-storage/
└── [drwxrwxrwx   32]  default-test-pvc-e8a15786-5a09-11e9-ad53-000c296286d8
    ├── [-rw-r--r--  947]  1.log
    └── [-rw-r--r-- 1.0K]  2.log
```