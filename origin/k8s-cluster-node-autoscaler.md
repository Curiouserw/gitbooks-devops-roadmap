# k8s集群节点自动伸缩容

# 一、简介

自建k8s集群，使用阿里云的伸缩组服务进行集群节点的自动伸缩容。在使用kubernetes的AutoScaler服务监控k8s节点资源使用状态，调用阿里云的API触发集群ECS伸缩容

由于node弹性伸缩存在一定的时延，这个时延主要包含：采集时延(分钟级) + 判断时延(分钟级) + 伸缩时延(分钟级)结合业务，根据时间段，自动伸缩业务(CronHPA)来处理高峰数据，底层自动弹性伸缩kubernetes node增大容器资源池

https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler/cloudprovider/alicloud

# 二、准备节点的模板镜像

## 1、创建一个按量付费的ECS节点

- **OS：**Ubuntu 20.04.5
- **硬件：**8核16G
- **磁盘：**100G系统盘 200G数据盘

## 2、准备ECS节点的基础环境

①优化系统参数

```bash
# 一次性设置文件句柄量限制
sysctl -w fs.inotify.max_user_watches="81920"
# 永久性设置文件数量限制
echo "fs.inotify.max_user_watches = 81920" >>/etc/sysctl.conf
```

②安装基础软件

```bash
apt update && apt install -y lvm2 socat nfs-common docker.io
```

③创建基础文件路径、挂载NFS

```bash
mkdir -p /log/app/{test,stg}
# 挂载NFS
apt install -y nfs-common
echo "192.168.1.111:/k8s/ /mnt nfs vers=3,nolock,proto=tcp,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,_netdev,noresvport 0 0" >>/etc/fstab
# mount -a
```

④安装docker

```bash
 apt install -y docker.io socat
# 下载docker配置
wget http://nexus.curiouser.com:8081/repository/resources/k8s/daemon.json -P /etc/docker/
# 启动docker
systemctl daemon-reload
systemctl enable docker
systemctl restart docker
docker info
```

⑤下载解压sealos资源包

```bash
wget http://nexus.curiouser.com:8081/repository/resources/k8s/kube1.17.0.tar.gz -P /root
tar -zxvf /root/kube1.17.0.tar.gz -C /root/
# 执行检测脚本
cd /root/kube/shell/
sh init.sh
rm -rf /root/kube
```

⑥下载SSH密钥对

```bash
wget http://nexus.curiouser.com:8081/repository/resources/k8s/id_ed25519 -P /root/.ssh
chmod 600 /root/.ssh/id_ed25519
wget http://nexus.curiouser.com:8081/repository/resources/k8s/id_ed25519.pub -P /root/.ssh
chmod 600 /root/.ssh/id_ed25519.pub
cat /root/.ssh/id_ed25519.pub >>/root/.ssh/authorized_keys
```

⑦安装采集日志的组件

```bash
wget http://nexus.curiouser.com:8081/repository/resources/elk-install-packages/filebeat-7.5.1-amd64.deb -P /root
dpkg -i /root/filebeat-7.5.1-amd64.deb
mv /etc/filebeat/filebeat.yml /etc/filebeat/filebeat.yml.bak
wget http://nexus.curiouser.com:8081/repository/resources/k8s/filebeat.yml -P /etc/filebeat
rm -rf /root/filebeat-7.5.1-amd64.deb
systemctl enable filebeat
```

⑧清理

```bash
rm -rf /var/lib/apt/lists/*
```

## 3、创建ECS节点的镜像

# 三、阿里云的ESS

## 1、创建阿里云伸缩配置

## 2、创建阿里云伸缩组

- 获取伸缩组的ASG_ID

## 3、创建阿里云子账号授予

https://help.aliyun.com/document_detail/185763.html

- 创建子账号，获取子账号的`AccessKey ID`和 `AccessKey Secret`
- 授予子账号`AliyunESSFullAccess`的系统权限
- 创建以下自定义权限策略，并授予子账号

```json
{
  "Version": "1",
  "Statement": [
    {
      "Action": [
        "ess:Describe*",
        "ess:CreateScalingRule",
        "ess:RemoveInstances",
        "ess:ExecuteScalingRule",
        "ess:ModifyScalingRule",
        "ess:DetachInstances",
        "ecs:DescribeInstanceTypes"
      ],
      "Resource": [
        "*"
      ],
      "Effect": "Allow"
    }
  ]
}
```

# 参考

- https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler/cloudprovider/alicloud
- https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md
- https://www.jianshu.com/p/02304dd32056
- https://developer.aliyun.com/article/392170
- https://help.aliyun.com/document_detail/169655.html