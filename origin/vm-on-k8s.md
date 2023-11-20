# KubeVirt：VM on K8S

# 一：简介

Github：https://github.com/kubevirt/kubevirt

文档：https://kubevirt.io/user-guide/architecture/

原理：使用 k8s的调度、网络、存储等功能在 k8s node节点运行、管理 KVM 虚拟机

暂不调研。需要 k8s 的节点开起虚拟化功能。目前测试 K8s 集群服务器在云服务商上。

# 二、安装

## 1、部署operator

```bash
export VERSION=$(curl -s https://api.github.com/repos/kubevirt/kubevirt/releases | grep tag_name | grep -v -- '-rc' | sort -r | head -1 | awk -F': ' '{print $2}' | sed 's/,//' | xargs)
echo $VERSION
kubectl create -f "https://github.com/kubevirt/kubevirt/releases/download/$VERSION/kubevirt-operator.yaml"
```

- **namespace/kubevirt**
- **customresourcedefinition.apiextensions.k8s.io/kubevirts.kubevirt.io**
- **priorityclass.scheduling.k8s.io/kubevirt-cluster-critical**
- **clusterrole.rbac.authorization.k8s.io/kubevirt.io:operator**
- **serviceaccount/kubevirt-operator**
- **role.rbac.authorization.k8s.io/kubevirt-operator**
- **rolebinding.rbac.authorization.k8s.io/kubevirt-operator-rolebinding**
- **clusterrole.rbac.authorization.k8s.io/kubevirt-operator**
- **clusterrolebinding.rbac.authorization.k8s.io/kubevirt-operator**
- **deployment.apps/virt-operator**

## 2、部署 CRD

```bash
kubectl create -f "https://github.com/kubevirt/kubevirt/releases/download/$VERSION/kubevirt-cr.yaml"
```

- 7 pods
- 3 services
- 1 daemonset
- 3 deployment apps
- 3 replica sets



## 3、安装 virtctl

```bash
VERSION=$(kubectl get kubevirt.kubevirt.io/kubevirt -n kubevirt -o=jsonpath="{.status.observedKubeVirtVersion}")
ARCH=$(uname -s | tr A-Z a-z)-$(uname -m | sed 's/x86_64/amd64/') || windows-amd64.exe
echo ${ARCH}
curl -L -o virtctl "https://github.com/kubevirt/kubevirt/releases/download/$VERSION/virtctl-$VERSION-$ARCH"
chmod +x virtctl
mv virtctl /usr/local/bin
```
