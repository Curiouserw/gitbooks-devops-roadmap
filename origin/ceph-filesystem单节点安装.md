# Context

| Hostname | OS | Ceph版本 |
| :--- | :--- | :--- |
| allinone.curiouser.com | CentOS 7.4.1708 | 10.2.10（Jewel） |

一个cephfs至少要求两个librados存储池，一个为data，一个为metadata。当配置这两个存储池时，注意：

* 为metadata pool设置较高级别的副本级别，因为metadata的损坏可能导致整个文件系统不用

* 建议metadata pool使用低延时存储，比如SSD，因为metadata会直接影响客户端的响应速度

# Preflight

1. 一个 clean+active 的cluster（[Ceph RBD单节点安装](/infrastructure/install/ceph-rbddan-jie-dian-an-zhuang.md "Ceph单节点安装")）

```bash
cluster fb506b4e-43b8-4634-acb9-ea3ee5a97b91
     health HEALTH_OK
     monmap e1: 1 mons at {allinone=192.168.1.96:6789/0}
            election epoch 29, quorum 0 allinone
      fsmap e4: 1/1/1 up {0=allinone.okd311.curiouser.com=up:active}
     osdmap e113: 1 osds: 1 up, 1 in
            flags sortbitwise,require_jewel_osds
      pgmap v61453: 192 pgs, 3 pools, 2639 MB data, 985 objects
            2730 MB used, 94500 MB / 97231 MB avail
                 192 active+clean
```

# 一、操作

1. 部署元数据服务器MDS
   ```bash
   ceph-deploy mds create $HOSTNAME
   ```
2. 创建cephfs需要的两个存储池：一个pool用来存储数据，一个pool用来存储元数据
   ```bash
   ceph osd pool create cephfs_data 64
   ceph osd pool create cephfs_metadata 64
   ```
3. 创建CephFS

   ```bash
   ceph fs new cephfs cephfs_metadata cephfs_data
   ceph fs ls
   ```

# 二、验证

```bash
$ ceph mds stat
e4: 1/1/1 up {0=allinone.okd311.curiouser.com=up:active}
```

# 三、客户端挂载

**Kernel方式**

```bash
#加载rbd内核模块
modprobe rbd
lsmod | grep rbd

# 获取client.admin用户的秘钥
ceph auth get client.admin
# [client.admin]
#        key = AQCinINcLykNLhAA7Xr6o+Q2jYeyc5j58JeQeQ==
#        caps mds = "allow *"
#        caps mon = "allow *"
#        caps osd = "allow *"


mkdir  /mnt/mycephfs
mount -t ceph allinone.okd311.curiouser.com:/ /mnt/mycephfs -o name=admin,secret=AQCinINcLykNLhAA7Xr6o+Q2jYeyc5j58JeQeQ==
```

**FUSE方式**

```bash
yum -y install ceph-fuse
ceph-fuse -k /etc/ceph/ceph.client.admin.keyring -m 192.168.197.154:6789 ~/mycephfs/
```