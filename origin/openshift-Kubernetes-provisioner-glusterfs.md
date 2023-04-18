# 一、OKD集群中添加容器化的GlusteFS

## Prerequisite

- OKD集群（3.11）至少有三个节点
- OKD官方操作指南：https://docs.okd.io/3.11/install_config/persistent_storage/persistent_storage_glusterfs.html#install-config-persistent-storage-persistent-storage-glusterfs
- GlusterFS官方操作指南：https://docs.gluster.org/en/latest/Administrator%20Guide/overview/
- heketi-cli官方操作指南：https://github.com/heketi/heketi

配置ansible主机清单/etc/ansible/hosts

```yaml
[OSEv3:children]
...
glusterfs
​
[OSEv3:vars]
...
openshift_storage_glusterfs_namespace=app-storage
openshift_storage_glusterfs_storageclass=true
openshift_storage_glusterfs_storageclass_default=false
openshift_storage_glusterfs_block_deploy=true
openshift_storage_glusterfs_block_host_vol_size=100
openshift_storage_glusterfs_block_storageclass=true
openshift_storage_glusterfs_block_storageclass_default=false
​
[glusterfs]
allinone311.okd.curiouser.com glusterfs_devices='[ "/dev/vdf" ]'
node1.okd.curiouser.com glusterfs_devices='[ "/dev/vdd" ]'
node2.okd.curiouser.com glusterfs_devices='[ "/dev/vdd" ]'
#至少是三个节点
```

glusterfs节点上安装软件

```bash
yum install glusterfs-fuse && \
yum update glusterfs-fuse
```

配置glusterfs节点上的Selinux

```bash
setsebool -P virt_sandbox_use_fusefs on && \
setsebool -P virt_use_fusefs on
```

执行openshift ansible playbook

```bash
ansible-playbook /root/openshift-ansible/playbooks/openshift-glusterfs/config.yml
```

# 二、向OKD集群中添加集群外的GlusteFS

配置ansible主机清单/etc/ansible/hosts 

```yaml
[OSEv3:children]
...
glusterfs
​
[OSEv3:vars]
...
openshift_storage_glusterfs_namespace=app-storage
openshift_storage_glusterfs_storageclass=true
openshift_storage_glusterfs_storageclass_default=false
openshift_storage_glusterfs_block_deploy=true
openshift_storage_glusterfs_block_host_vol_size=100
openshift_storage_glusterfs_block_storageclass=true
openshift_storage_glusterfs_block_storageclass_default=false
openshift_storage_glusterfs_is_native=false
openshift_storage_glusterfs_heketi_is_native=true
openshift_storage_glusterfs_heketi_executor=ssh
openshift_storage_glusterfs_heketi_ssh_port=22
openshift_storage_glusterfs_heketi_ssh_user=root
openshift_storage_glusterfs_heketi_ssh_sudo=false
openshift_storage_glusterfs_heketi_ssh_keyfile="/root/.ssh/id_rsa"
​
[glusterfs]
gluster1.example.com glusterfs_ip=192.168.10.11 glusterfs_devices='[ "/dev/xvdc", "/dev/xvdd" ]'
gluster2.example.com glusterfs_ip=192.168.10.12 glusterfs_devices='[ "/dev/xvdc", "/dev/xvdd" ]'
gluster3.example.com glusterfs_ip=192.168.10.13 glusterfs_devices='[ "/dev/xvdc", "/dev/xvdd" ]'
```

执行openshift ansible playbook

```bash
ansible-playbook /root/openshift-ansible/playbooks/openshift-glusterfs/config.yml
```

# 三、卸载

```bash
ansible-playbook -e "openshift_storage_glusterfs_wipe=true" /root/openshift-ansible/playbooks/openshift-glusterfs/uninstall.yml 
```

# 四、OKD中通过storage动态使用glusterfs作为PVC的后端存储

## 1. 创建storage class

创建storage class(ansible playbook执行过程中会自动创建storageclass)

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: glusterfs-storage
provisioner: kubernetes.io/glusterfs
parameters:
  resturl: 'http://heketi-storage.app-storage.svc:8080'
  restuser: admin
  secretName: heketi-storage-admin-secret
  secretNamespace: app-storage
reclaimPolicy: Delete
volumeBindingMode: Immediate
```

如果使用的集群外的Glusterfs集群，需要手动创建storage class。

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: glusterfs-storage
provisioner: kubernetes.io/glusterfs
parameters:
  resturl: "http://10.42.0.0:8080"
  restauthenabled: "false"
```

## 2. 创建PVC时使用Glusterfs的storage class

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
 name: gluster1
spec:
 accessModes:
  - ReadWriteMany
 resources:
   requests:
        storage: 30Gi
 storageClassName: glusterfs-storage
```


# 五、主机上mount挂载使用容器化的GlusterFS 

挂载命令格式：

```bash
mount -t glusterfs GlusterFS容器化pod所在的节点IP地址:/volume_name /mnt/glusterfs
```

示例：

```bash
$ mount -t glusterfs 172.16.1.4:/vol_fe0de9d2f43731d1af7a5dc296041d83 /mnt/glusterfs && \
  df -mh 
172.16.1.4:/vol_fe0de9d2f43731d1af7a5dc296041d83            10G  136M  9.9G   2% /mnt/glusterfs
```