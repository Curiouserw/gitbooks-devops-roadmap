# Prerequisite

| Hostname | OS | Ceph版本 |
| :---: | :---: | :---: |
| allinone.curiouser.com | CentOS 7.4.1708 | 10.2.10（Jewel） |

1. 关闭防火墙和SeLinux
   ```bash
   systemctl disable firewalld ;
   systemctl stop firewalld ;
   sed -i "s/SELINUX=enforcing/SELINUX=disabled/" /etc/selinux/config ;
   setenforce 0 ;
   sestatus -v
   ```
2. SSH免密码登录打通
   ```bash
   ssh-keygen -t rsa -P "" -f ~/.ssh/id_rsa ;
   ssh-copy-id root@allinone.curiouser.com ;
   ssh allinone.curiouser.com
   ```
3. Hosts绑定IP地址域名解析
   ```bash
   echo "192.168.1.21 allinone.curiouser.com" >> /etc/hosts
   ```
4. 创建ceph用户并设置用户密码和为其添加root权限
   ```bash
   useradd ceph && echo ceph:ceph | chpasswd ;
   echo "ceph ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/ceph ;
   chmod 0440 /etc/sudoers.d/ceph
   ```
5. 配置Ceph和Epel的yum源仓库  
   ```bash 
   vim /etc/yum.repos.d/ceph.repo  
   [Ceph]  
   name=Ceph packages for $basearch  
   baseurl=http://download.ceph.com/rpm-jewel/el7/$basearch
   enabled=1  
   gpgcheck=1  
   type=rpm-md  
   gpgkey=https://download.ceph.com/keys/release.asc
   priority=1  
   [Ceph-noarch]  
   name=Ceph noarch packages  
   baseurl=http://download.ceph.com/rpm-jewel/el7/noarch
   enabled=1  
   gpgcheck=1  
   type=rpm-md  
   gpgkey=https://download.ceph.com/keys/release.asc  
   priority=1

   可以修改ceph源（外国的源总是timeout）   
   export CEPH_DEPLOY_REPO_URL=http://mirrors.163.com/ceph/rpm-jewel/el7 ;  
   export CEPH_DEPLOY_GPG_URL=http://mirrors.163.com/ceph/keys/release.asc
   ```

6. (可选)手动安装下载ceph的rpm包（使用ceph-deploy install 安装ceph包网速太慢。）官方下载: [http://download.ceph.com/rpm-jewel/el7/x86_64/](http://download.ceph.com/rpm-jewel/el7/x86_64/) 。需要下载的包如下：  

   ```
ceph-10.2.10-0.el7.x86_64.rpm  
ceph-base-10.2.10-0.el7.x86_64.rpm  
ceph-common-10.2.10-0.el7.x86_64.rpm  
ceph-mds-10.2.10-0.el7.x86_64.rpm  
ceph-mon-10.2.10-0.el7.x86_64.rpm  
ceph-osd-10.2.10-0.el7.x86_64.rpm  
ceph-radosgw-10.2.10-0.el7.x86_64.rpm  
ceph-selinux-10.2.10-0.el7.x86_64.rpm  
rbd-mirror-10.2.10-0.el7.x86_64.rpm

yum localinstall -y ./*.rpm  
   ```

## 一、安装Ceph-Deploy

1. 安装Ceph-deploy

   ```bash
   yum install ceph-deploy -y
   ```

2. 安装ceph相关的软件

   ```bash
   ceph-deploy install $HOSTNAME
   ```

## 二、创建集群配置文件

1. 创建ceph-deploy的集群配置文件夹，路径并切换过去
   ```bash
   mkdir my-cluster ;cd my-cluster
   ```
   
2. 用 ceph-deploy 创建集群，用 new 命令、并指定主机作为初始监视器。  
   ```bash 
   ceph-deploy new $HOSTNAME

   # 该操作会在~/my-cluster下会生成三个文件

   -rw-rw-r-- 1 ceph ceph   251 Jan 12 16:34 ceph.conf  
   -rw-rw-r-- 1 ceph ceph 15886 Jan 12 16:30 ceph.log  
   -rw------- 1 ceph ceph    73 Jan 12 16:30 ceph.mon.keyring
   ```
   
   ceph.conf中默认的osd pool为3，对应了三个node节点。如果只有两个node节点，则需要修改ceph.conf中的默认值
   
   ```ini
   [global] 
   fsid = 25c13add-967e-4912-bb33-ebbc2cb9376d 
   mon_initial_members = allinone.curiouser.com 
   mon_host = 172.16.2.3 
   auth_cluster_required = cephx 
   auth_service_required = cephx 
   auth_client_required = cephx
   # =========新增部分============
   filestore_xattr_use_omap = true
   osd pool default size=1
   osd max object name len = 256
   osd max object namespace len = 256
   mon_pg_warn_max_per_osd = 500
   ```
   
   
## 三、创建Monitor

   ```bash
ceph-deploy mon create $HOSTNAME ;
ceph-deploy gatherkeys $HOSTNAME ;
ceph mds stat  #查看mds节点状态
   ```

## 四、创建OSD

**方式一**：

1. (可选)手动节点上挂载lvm存储到某个目录下，作为node节点上OSD的数据存储目录 
   ```bash 
   yum install -y lvm2 ;  
   disk=/dev/vdc ;  
   pvcreate ${disk} ;  
   vgcreate ${disk} ;  
   vgcreate -s 16m ceph-osd ${disk} ;  
   PE_Number=`vgdisplay ${disk}|grep "Free  PE"|awk '{print $5}'` ;  
   lvcreate -l  ${PE_Number} -n ceph-osd ceph-osd ;  
   mkfs.xfs /dev/ceph-osd/ceph-osd ;  
   mkdir -p /data/ceph/osd ;  
   chown -R ceph:ceph /data/ceph/osd ;  
   echo "/dev/ceph-osd/ceph-osd  /data/ceph/osd xfs defaults 0 0" >> /etc/fstab ;  
   mount -a ;  
   df -mh

   #LV的文件系统格式注意要xfs,CentOS推荐使用xfs的文件系统.如果是ext4，需要在/etc/ceph/ceph.conf 中添加参数用来限制文件名的长度
   osd max object name len = 256
   osd max object namespace len = 64
   # 之后重启osd服务
   systemctl restart  ceph-osd.target 
   ```

2. 准备并激活node节点上的OSD
   ```bash
   #准备Node节点上的OSD
   ceph-deploy osd prepare $HOSTNAME:/data/ceph/osd
   #激活Node节点上的OSD
   ceph-deploy osd activate $HOSTNAME:/data/ceph/osd
   #查看OSD状态
   ceph osd tree
   ```

**方式二**：(不是以目录为OSD数据存储设备，而是直接以硬盘。其实就是省去手动在硬盘上创建分区的操作)

   ```bash
#准备Node节点上的OSD
ceph-deploy osd prepare $HOSTNAME:/dev/vdc
#激活Node节点上的OSD
ceph-deploy osd activate $HOSTNAME:/dev/vdc1
#查看OSD状态
ceph osd tree
   ```

## 五、安装验证

   ```bash
#集群健康状态检查
$> ceph health
HEALTH_OK
$> ceph -s
$> systemctl is-enabled ceph-mds.target ceph-mon.target ceph-osd.target ceph-radosgw.target ceph.target
   ```

## 六、其他信息

1. Ceph相关的SystemD Units

   ```bash
   ceph-mds.target
   ceph-mon.target  
   ceph-osd.target
   ceph-radosgw.target
   ceph.target
   ```

2. 查看、 更新配置

   ```bash
   ceph --show-config
   ceph-deploy --overwrite-conf config push $HOSTNAME
   ```

3. 清除卸载Ceph

   ```bash
   ceph-deploy uninstall $HOSTNAME
   ceph-deploy purge $HOSTNAME
   rm -rf /var/lib/ceph/ /var/run/ceph
   rm -rf /data/ceph/osd/*
   ```

   

