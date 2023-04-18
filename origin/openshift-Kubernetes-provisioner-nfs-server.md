# 一、NFS Server Provisioner

Github项目地址：https://github.com/kubernetes-incubator/external-storage/tree/v5.2.0/nfs

Provisioner的定义原理：[openshift-Kubernetes的持久化存储](/origin/openshift-Kubernetes的持久化存储.md)

NFS Provisioner的部署文档：https://github.com/kubernetes-incubator/external-storage/blob/master/nfs/docs/deployment.md

NFS Provisioner的使用文档：https://github.com/kubernetes-incubator/external-storage/blob/master/nfs/docs/usage.md

# 二、在Kubernetes上部署

## 1、（可选）预拉取镜像

```bash
docker pull quay-mirror.qiniu.com/kubernetes_incubator/nfs-provisioner:latest && \
docker tag quay-mirror.qiniu.com/kubernetes_incubator/nfs-provisioner:latest quay.io/kubernetes_incubator/nfs-provisioner:latest && \
docker rmi quay-mirror.qiniu.com/kubernetes_incubator/nfs-provisioner:latest
```

## 2、创建PodSecurityPolicy

```yaml
apiVersion: extensions/v1beta1
kind: PodSecurityPolicy
metadata:
  name: nfs-provisioner
spec:
  fsGroup:
    rule: RunAsAny
  allowedCapabilities:
  - DAC_READ_SEARCH
  - SYS_RESOURCE
  runAsUser:
    rule: RunAsAny
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  volumes:
  - configMap
  - downwardAPI
  - emptyDir
  - persistentVolumeClaim
  - secret
  - hostPath
```

## 3、创建RBAC

```yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-provisioner-runner
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
    resources: ["services", "endpoints"]
    verbs: ["get"]
  - apiGroups: ["extensions"]
    resources: ["podsecuritypolicies"]
    resourceNames: ["nfs-provisioner"]
    verbs: ["use"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-provisioner
     # replace with namespace where provisioner is deployed
    namespace: default
roleRef:
  kind: ClusterRole
  name: nfs-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-provisioner
rules:
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-provisioner
    # replace with namespace where provisioner is deployed
    namespace: default
roleRef:
  kind: Role
  name: leader-locking-nfs-provisioner
  apiGroup: rbac.authorization.k8s.io
```

## 4、使用deployment创建POD（推荐）

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nfs-provisioner
---
kind: Service
apiVersion: v1
metadata:
  name: nfs-provisioner
  labels:
    app: nfs-provisioner
spec:
  ports:
    - name: nfs
      port: 2049
    - name: mountd
      port: 20048
    - name: rpcbind
      port: 111
    - name: rpcbind-udp
      port: 111
      protocol: UDP
  selector:
    app: nfs-provisioner
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nfs-provisioner
spec:
  selector:
    matchLabels:
      app: nfs-provisioner
  replicas: 1
  strategy:
    type: Recreate 
  template:
    metadata:
      labels:
        app: nfs-provisioner
    spec:
      serviceAccount: nfs-provisioner
      containers:
        - name: nfs-provisioner
          image: quay.io/kubernetes_incubator/nfs-provisioner:latest
          ports:
            - name: nfs
              containerPort: 2049
            - name: mountd
              containerPort: 20048
            - name: rpcbind
              containerPort: 111
            - name: rpcbind-udp
              containerPort: 111
              protocol: UDP
          securityContext:
            capabilities:
              add:
                - DAC_READ_SEARCH
                - SYS_RESOURCE
          args:
           # 定义提供者的名称，存储类通过此名称指定提供者
            - "-provisioner=example.com/nfs"
          env:
            - name: POD_IP
              valueFrom:
                fieldRef:
                  fieldPath: status.podIP
            - name: SERVICE_NAME
              value: nfs-provisioner
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          imagePullPolicy: "IfNotPresent"
          volumeMounts:
            - name: export-volume
              mountPath: /export
      volumes:
        - name: export-volume
          hostPath:
            path: /srv
```

## 5、（可选）使用StatefulSet创建POD

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nfs-provisioner
---
kind: Service
apiVersion: v1
metadata:
  name: nfs-provisioner
  labels:
    app: nfs-provisioner
spec:
  ports:
    - name: nfs
      port: 2049
    - name: mountd
      port: 20048
    - name: rpcbind
      port: 111
    - name: rpcbind-udp
      port: 111
      protocol: UDP
  selector:
    app: nfs-provisioner
---
kind: StatefulSet
apiVersion: apps/v1
metadata:
  name: nfs-provisioner
spec:
  selector:
    matchLabels:
      app: nfs-provisioner
  serviceName: "nfs-provisioner"
  replicas: 1
  template:
    metadata:
      labels:
        app: nfs-provisioner
    spec:
      serviceAccount: nfs-provisioner
      terminationGracePeriodSeconds: 10
      containers:
        - name: nfs-provisioner
          image: quay.io/kubernetes_incubator/nfs-provisioner:latest
          ports:
            - name: nfs
              containerPort: 2049
            - name: mountd
              containerPort: 20048
            - name: rpcbind
              containerPort: 111
            - name: rpcbind-udp
              containerPort: 111
              protocol: UDP
          securityContext:
            capabilities:
              add:
                - DAC_READ_SEARCH
                - SYS_RESOURCE
          args:
            - "-provisioner=example.com/nfs"
          env:
            - name: POD_IP
              valueFrom:
                fieldRef:
                  fieldPath: status.podIP
            - name: SERVICE_NAME
              value: nfs-provisioner
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          imagePullPolicy: "IfNotPresent"
          volumeMounts:
            - name: export-volume
              mountPath: /export
      volumes:
        - name: export-volume
          hostPath:
            path: /srv
```


# 三、使用
创建StorageClass

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: example-nfs
provisioner: example.com/nfs
parameters:
  mountOptions: "vers=4.1"
​
=======================================================补充内容=========================================================
​
#其他参数：
gid: # "none" or a supplemental group like "1001". NFS shares will be created with permissions such that pods running with the supplemental group can read & write to the share, but non-root pods without the supplemental group cannot. Pods running as root can read & write to shares regardless of the setting here, unless the rootSquash parameter is set true. If set to "none", anybody root or non-root can write to the share. Default (if omitted) "none".
rootSquash: # "true" or "false". Whether to squash root users by adding the NFS Ganesha root_id_squash or kernel root_squash option to each export. Default "false".
mountOptions: # a comma separated list of mount options for every PV of this class to be mounted with. The list is inserted directly into every PV's mount options annotation/field without any validation. Default blank "".  
​
#如果要将此storageclass设置为默认，在metadata里面添加以下注解。（这样创建PVC时就可以不用特意指定StorageClass）
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  #注解
  annotations: 
    "storageclass.kubernetes.io/is-default-class": "true"
  name: example-nfs
provisioner: example.com/nfs
parameters:
  mountOptions: "vers=4.1"
```

创建PVC时指定storageclass

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: nfs
  annotations:
    volume.beta.kubernetes.io/storage-class: "example-nfs"
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Mi
```      

# 四、测试

创建一个PVC

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: test
  annotations:
    volume.beta.kubernetes.io/storage-class: "example-nfs"
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 100Mi
​
#========================================================================================================================
$ kubectl get pvc
NAME  STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
test  Bound     pvc-e00e9603-56a5-11e9-95dd-080027c8ba17   100Mi      RWX            example-nfs    5m3s
$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM         STORAGECLASS   REASON   AGE
pvc-e00e9603-56a5-11e9-95dd-080027c8ba17   100Mi      RWX            Delete           Bound    default/test   example-nfs             5m9s

```

创建一个POD使用PVC

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

查看/srv目录

```bash
/srv
├── [-rw-r--r-- 4.4K]  ganesha.log
├── [-rw-------   36]  nfs-provisioner.identity
├── [drwxrwsrwx   32]  pvc-e00e9603-56a5-11e9-95dd-080027c8ba17
│   ├── [-rw-r--r-- 5.1K]  1.log
│   └── [-rw-r--r-- 5.7K]  2.log
└── [-rw------- 1.1K]  vfs.conf
注意：删除掉PVC，PV也会自动删除，底层的NFS目录也会跟着删除
```