# 一、Kubernetes持久化存储简介

通常情况下，我们可以认为容器或者Pod的生命周期时短暂的，当容器被销毁时，容器内部的数据也同时被清除。

对于容器，数据持久化存储的重要性不言而喻。Docker有存储卷的概念，用来将磁盘上的或另一个容器中的目录挂载到容器的某一个路径下。即使容器挂掉了，挂载Volume中的数据依旧存在。然而没有对其生命周期进行管理。而Kubernetes提供了多种不同类型资源的Volume存储卷，供POD挂载到容器的不同路径下,常见的有：
- ***emptyDir***：pod被调度到某个宿主机上的时候才创建，而同一个pod内的容器都能读写EmptyDir中的同一个文件。删除容器并不会对它造成影响，只有删除整个Pod时，它才会被删除，它的生命周期与所挂载的Pod一致
- ***hostPath***：将宿主机的文件系统的文件或目录挂接到Pod中
- ***secret***：将Kubernetes中secret对象资源挂载到POD中
- ***configMap***：将Kubernetes中config对象资源挂载到POD中
- ***persistentVolumeClaim***：将PersistentVolume挂接到Pod中作为存储卷。使用此类型的存储卷，用户不需要关注存储卷的详细信息。
- ***nfs***
- ***glusterfs***
- ***cephfs***
- ***vspherevolume***
- ***iscsi***
- ....


对于以上大部分的volume类型，对使用用户是极其不友好的。理解他们体系中的概念配置是一件复杂的事情，有时我们其实并不关心他们的各种存储实现，只希望能够简单安全可靠地存储数据。所以K8S对存储的供应和使用做了抽象，以API形式提供给管理员和用户使用。因此引入了两个新的API资源：Persistent Volume（持久卷PV）和Persistent Volume Claim（持久卷申请PVC）。

PVC负责定义使用多大的存储空间，什么样的读写方式等常见要求即可，而PV负责抽象各种存储系统的技术细节（例如存储系统IP地址端口，客户端证书密钥等），满足PVC的存储需求，继而作为Kubernetes集群的存储对象资源。

---

```yaml
apiVersion: "v1"
kind: "PersistentVolumeClaim"
metadata:
  name: "ceph-pvc-test"
  namespace: "default"
spec:
  accessModes:
    - "ReadWriteMany"
  resources:
    requests:
      storage: "2Gi"
  volumeName: "pv-nfs-test"  # 指定PV
```

## PersistentVolumesClaim的属性

  - ***Access Modes***
      * ReadWriteOnce —— 该volume只能被单个节点以读写的方式映射
      * ReadOnlyMany  —— 该volume可以被多个节点以只读方式映射
      * ReadWriteMany —— 该volume只能被多个节点以读写的方式映射
  - ***Volume Modes***：在Kubernetes 1.9之前，所有卷插件都在pv上创建了一个文件系统。现在，可以将volumeMode的值设置为block以使用原始块设备，或者将filesystem设置为使用文件系统。如果省略该值，则默认为filesystem。
  - ***Resources***：指定使用多大的存储空间
  - ***Selector***：PVC可以指定标签选择器进行更深度的过滤PV，只有匹配了选择器标签的PV才能绑定给PVC。选择器包含两个字段：
      - matchLabels（匹配标签） - PV必须有一个包含该值得标签
      - matchExpressions（匹配表达式） - 一个请求列表，包含指定的键、值的列表、关联键和值的操作符。合法的操作符包含In，NotIn，Exists，和DoesNotExist。
  　　所有来自matchLabels和matchExpressions的请求，都是逻辑与关系的，它们必须全部满足才能匹配上。
  - ***Class***

---

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: ceph-pv-test     
spec:
  capacity:
    storage: 2Gi    
  accessModes:
    - ReadWriteOnce 
  rbd:              
    monitors:       
      - 192.168.122.133:6789
    pool: rbd
    image: ceph-image
    user: admin
    secretRef:
      name: ceph-secret 
    fsType: ext4        
    readOnly: false
  persistentVolumeReclaimPolicy: Retain
  claimRef:
    name: "pvc-test"
    namespace: "default"
```


## PersistentVolumes的属性


   * ***Capacity***：指定存储容量大小
   * ***Volume Mode***：在Kubernetes 1.9之前，所有卷插件都在pv上创建了一个文件系统。现在，可以将volumeMode的值设置为block以使用原始块设备，或者将filesystem设置为使用文件系统。如果省略该值，则默认为filesystem。
   * ***Class***: 一个PV可以有一种class，通过设置storageClassName属性来选择指定的StorageClass。有指定class的PV只能绑定给请求该class的PVC。没有设置storageClassName属性的PV只能绑定给未请求class的PVC(过去，使用volume.beta.kubernetes.io/storage-class注解，而不是storageClassName属性。该注解现在依然可以工作，但在Kubernetes的未来版本中已经被完全弃用了)
   * ***Reclaim Policy***
   * ~~***Mount Options***~~
   * ***Node Affinity***
   * ***Access Modes***
       * ReadWriteOnce —— 该volume只能被单个节点以读写的方式映射
       * ReadOnlyMany  —— 该volume可以被多个节点以只读方式映射
       * ReadWriteMany —— 该volume只能被多个节点以读写的方式映射

---

## PersistentVolumes的周期状态

   * ***Available***: 空闲的，未绑定给PVC
   * ***Bound***: 绑定上了某个PVC
   * ***Released***: PVC已经删除了，但是PV还没有被回收
   * ***Failed***: PV在自动回收中失败了

## PV支持的存储系统:

  * ***GCEPersistentDisk***
  * ***AWSElasticBlockStore***
  * ***AzureFile***
  * ***AzureDisk***
  * ***CSI***
  * ***FC*** (Fibre Channel)
  * ***Flexvolume***
  * ***Flocker***
  * ***NFS***
  * ***iSCSI***
  * ***RBD*** (Ceph Block Device)
  * ***CephFS***
  * ***Cinder*** (OpenStack block storage)
  * ***Glusterfs***
  * ***VsphereVolume***
  * ***Quobyte Volumes***
  * ***HostPath*** (Single node testing only – local storage is not supported in any way and WILL NOT WORK in a multi-node cluster)
  * ***Portworx Volumes***
  * ***ScaleIO Volumes***
  * ***StorageOS***



## PV和PVC 之间的关联遵循如下的生命周期：
  1. ***Provisioning-供应***:  PV的创建阶段，有以下两种创建方式
       - `静态手工`：集群管理员通过`手工`的方式创建pv
       - `动态自动`：通过PersistentVolume Controller动态调度，Kubernetes将能够按照用户的需要，根据PVC的资源请求，寻找StorageClasse定义的符合要求的底层存储`自动`创建其需要的存储卷。
  2. ***Binding-绑定***: PV分配绑定到PVC
  3. ***Using-使用***： POD挂载使用PVC类型的Volume
  4. ***Reclaiming-回收***：PV释放后的回收利用策略
        - `Retain保留`: 保留现场，人工回收
        - `Delete删除`: 自动删除，动态删除后端存储。需要IaaS层的支持，目前只有Ceph RBD和OpenStack Cinder支持
        - `Recycle复用`：通过rm -rf删除卷上的所有数据。目前只有NFS和HostPath支持（逐渐在抛弃该方式，建议使用）



# 二、使用StorageClass提供动态存储供应

通常情况下，Kubernetes集群管理员需要手工创建所需的PV存储资源。从Kubernetes 1.2以后可以使用Storageclass实现`动态自动`地根据用户需求创建某种存储系统类型的PV。同时，可以定义多个 StorageClass ，给集群提供不同存储系统类型的PV资源。


## 1. 定义创建StorageClass

每一个存储类都必须包含以下参数
- `provisioner`: 决定由哪个Provisioner来创建PV
- `parameters`: Provisioner需要的参数,可选项：Delete(Default),Retain
- `reclaimPolicy`: PV的回收策略

可选参数：
- `Mount Options`
- `Volume Binding Mode`
- `Allowed Topologies`

Note: StorageClass一旦被创建，将不能被更新

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ceph-rbd
# 指定Provisioner
provisioner: kubernetes.io/rbd
parameters:
  monitors: 10.20.30.40:6789
  adminId: kube
  adminSecretName: ceph-secret
  adminSecretNamespace: kube-system
  pool: kube
  userId: kube
  userSecretName: ceph-secret-user
  userSecretNamespace: default
  fsType: ext4
  imageFormat: "2"
  imageFeatures: "layering"    
```

**Kubernetes支持的Provisioner**

Provisioner | 是否内置插件 | 配置例子
------- | ------- | -------
AWSElasticBlockStore | ✓ | [AWS](https://kubernetes.io/docs/concepts/storage/storage-classes/#aws)
AzureFile | ✓ | [Azure File](https://kubernetes.io/docs/concepts/storage/storage-classes/#azure-file)
AzureDisk | ✓ | [Azure Disk](https://kubernetes.io/docs/concepts/storage/storage-classes/#azure-disk)
CephFS | – | –
Cinder | ✓ | [OpenStack Cinder](https://kubernetes.io/docs/concepts/storage/storage-classes/#openstack-cinder)
FC | – | –
FlexVolume | – | –
Flocker | ✓ | –
GCEPersistentDisk | ✓ | [GCE](https://kubernetes.io/docs/concepts/storage/storage-classes/#gce)
Glusterfs | ✓ | [Glusterfs](https://kubernetes.io/docs/concepts/storage/storage-classes/#glusterfs)
iSCSI | – | –
PhotonPersistentDisk | ✓ | –
Quobyte | ✓ | [Quobyte](https://kubernetes.io/docs/concepts/storage/storage-classes/#quobyte)
NFS | – | –
RBD | ✓ | [Ceph RBD](https://kubernetes.io/docs/concepts/storage/storage-classes/#ceph-rbd)
VsphereVolume | ✓ | [vSphere](https://kubernetes.io/docs/concepts/storage/storage-classes/#vsphere)
PortworxVolume | ✓ | [Portworx Volume](https://kubernetes.io/docs/concepts/storage/storage-classes/#portworx-volume)
ScaleIO | ✓ | [ScaleIO](https://kubernetes.io/docs/concepts/storage/storage-classes/#scaleio)
StorageOS | ✓ | [StorageOS](https://kubernetes.io/docs/concepts/storage/storage-classes/#storageos)
Local | – | [Local](https://kubernetes.io/docs/concepts/storage/storage-classes/#local)

1. StorageClas可以支持第三方的Provisioner，只要该插件符合Kubernetes的规范
2. 内置的Provisioner名称带有“kubernetes.io”前缀
3. Github仓库：[https://github.com/kubernetes-incubator/external-storage](https://github.com/kubernetes-incubator/external-storage) 有官方支持的第三方Provisioner

## 2. 指定StorageClass动态创建PV

在Kubernetes v1.6之前的版本，通过volume.beta.kubernetes.io/storage-class注释类请求动态供应存储；
在Kubernetes v1.6版本之后，用户应该使用PersistentVolumeClaim对象的storageClassName参数来请求动态存储。


```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim1
spec:
  accessModes:
    - ReadWriteOnce
# 指定所使用的存储类，此存储类将会自动创建符合要求的PV
 storageClassName: ceph-rbd
 resources:
    requests:
      storage: 30Gi
```

## 3. 指定默认的StorageClass

创建StorageClass时可添加添加storageclass.kubernetes.io/is-default-class注解来指定为默认的存储类。

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations: 
    "storageclass.kubernetes.io/is-default-class": "true" # 将此storageclass设置为默认
  name: nfs-client-storageclass
provisioner: fuseim.pri/ifs
parameters:
  archiveOnDelete: "true"
```

* 一个集群中，最多只能有一个默认的存储类
* 如果没有默认的存储类，在PersistentVolumeClaim中也没有显示指定storageClassName，将无法创建PersistentVolume。

# 参考链接
1. https://kubernetes.io/docs/concepts/storage/volumes/
2. https://kubernetes.io/docs/concepts/storage/storage-classes/
3. https://www.kubernetes.org.cn/4078.html
