# Sealos部署Kubernetes

# 一、简介

Github：https://github.com/fanux/sealos

官网：https://www.sealyun.com/

# 二、安装部署

## 1、部署前准备

| 主机名  | IP地址      |
| :------ | :---------- |
| master0 | 192.168.1.2 |
| master1 | 192.168.1.3 |
| master2 | 192.168.1.4 |
| node0   | 192.168.1.5 |

- 必须同步所有服务器时间
- 所有服务器主机名不能重复
- 操作系统支持：centos7.6以上、ubuntu16.04以上，推荐：centos7.7
- 内核推荐4.14以上
- **kubernetes .0版本不建议上生产环境!!!**
- 如果使用阿里云/华为云主机部署。 默认的pod网段会和阿里云的dns网段冲突， 建议自定义修改pod网段, 在init的时候指定`--podcidr` 来修改

## 2、部署

### ①下载安装

下载并安装二进制工具sealos和离线安装资源包

```sh
wget -c https://sealyun.oss-cn-beijing.aliyuncs.com/latest/sealos && \
chmod +x sealos && mv sealos /usr/bin 

wget -c https://sealyun.oss-cn-beijing.aliyuncs.com/2fb10b1396f8c6674355fcc14a8cda7c-v1.20.0/kube1.21.0.tar.gz    
```

### ②安装

```bash
sealos init --passwd '123456' \
    --master 192.168.1.2-192.168.1.4 \ 
    --node 192.168.1.5 \
    --pkg-url /root/kube1.21.0.tar.gz \
    --version v1.20.0
```

> 参数含义

| 参数名  | 含义                                                         | 示例                    |
| :------ | :----------------------------------------------------------- | :---------------------- |
| passwd  | 服务器密码                                                   | 123456                  |
| master  | k8s master节点IP地址                                         | 192.168.1.2-192.168.1.4 |
| node    | k8s node节点IP地址                                           | 192.168.1.3             |
| pkg-url | 离线资源包地址，支持下载到本地，或者一个远程地址             | /root/kube1.20.0.tar.gz |
| version | [资源包](https://www.sealyun.com/goodsDetail?type=cloud_kernel&name=kubernetes)对应的版本 | v1.20.0                 |
| apiserver  | apiserver的虚拟IP地址与域名映射。配置在/etc/hosts中 | 默认值`"apiserver.cluster.local"` |
| cert-sans | kubernetes apiServerCertSANs ex. 47.0.0.22 sealyun.com | |
| interface  |  |默认值为`"eth.*|en.*|em.*"`|
| ipip | 是否开起ipip模式 | 默认值为true |
| kubeadm-config  | kubeadm-config.yaml模板文件地址 | |
| lvscare-image  | lvscare镜像名 | 默认值为`"fanux/lvscare"` |
| lvscare-tag  | lvscare镜像标签 | 默认值为`“latest”` |
| mtu  | ipip模式的mtu值 | 默认值为`"1440"` |
| network  | 指定安装安装的CNI组件 | 默认值为`"calico"` |
| pk  | SSH私钥文件路径 | 默认值为`"/root/.ssh/id_rsa"` |
| pk-passwd | SSH私钥密码 |  |
| podcidr  | POD的IP地址网络段 | 默认值为`"100.64.0.0/10"` |
| repo  | 指定安装时镜像拉取的镜像注册仓库地址 | 默认值为`"k8s.gcr.io"` |
| svccidr  | k8s service的IP地址网络段 | 默认值为`"10.96.0.0/12"` |
| user  | ssh用户名 | 默认值为`"root"` |
| vip  | 虚拟IP地址 | 默认值为`"10.103.97.2"` |
| vlog | 设置kubeadm的日志级别 | |
| without-cni | 如果设置为true，则不安装CNI组件 | |
| config | 集群配置文件路径（全局配置参数） | 默认为`$HOME/.sealos/config.yaml` |
| info | 指定安装时的日志输出级别 | true为Info级别，false为Debug级别 |
|  |  |  |

## 3、验证操作

```bash
mv ~/kube/bin/kubectl /usr/local/bin/
kubectl completion bash > ~/.kube/completion.bash.inc
kubectl get pod --all-namespaces
kubectl get nodes 

apt install -y ipvsadm
ipvsadm -Ln
```

## 4、其他管理操作

### ①增加master节点

```shell
sealos join --master 192.168.1.6 --master 192.168.1.7
sealos join --master 192.168.1.6-192.168.1.9  # 或者多个连续IP
```

### ②增加node节点

```shell
sealos join --node 192.168.1.6 --node 192.168.1.7
sealos join --node 192.168.1.6-192.168.1.9  # 或者多个连续IP
```

### ③删除指定master节点

```shell
sealos clean --master 192.168.1.6 --master 192.168.1.7
sealos clean --master 192.168.1.6-192.168.1.9  # 或者多个连续IP
```

### ④删除指定node节点

```shell
sealos clean --node 192.168.1.6 --node 192.168.1.7
sealos clean --node 192.168.1.6-192.168.1.9  # 或者多个连续IP
```

### ⑤清理集群

```shell
sealos clean --all -f
```

### ⑥备份集群

```shell
sealos etcd save
```



