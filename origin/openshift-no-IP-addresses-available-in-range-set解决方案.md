# 问题一

**描述**

正在运行的Openshift Allinone 3.11集群创建POD突然报出

        network: failed to allocate for range 0: no IP addresses available in range set

**现象**

整个Allinone集群所有的POD不超多100个，但是/var/lib/cni/networks/openshift-sdn/的IP地址文件却有252个。造成当前节点的容器网络无法再为POD分配IP地址

**解决方案**

1. 将对应节点标记为不可调用

        oc adm manage-node node1.test.openshift.com --schedulable=false

2. 驱散对应节点上的POD

        oc adm drain node1.test.openshift.com --ignore-daemonsets

3. 停止docker和origin-node服务

        systemctl stop docker origin-node.service

4. 删除/var/lib/cni/networks/openshift-sdn/路径下所有文件

        rm -rf /var/lib/cni/networks/openshift-sdn/*

5. 重启docker和origin-node服务

        systemctl start docker origin-node.service

6. 将节点标记为可调度

        oc adm manage-node node1.test.openshift.com --schedulable=true

### 相关链接
https://access.redhat.com/solutions/3328541
https://github.com/debianmaster/openshift-examples/issues/59
https://github.com/cloudnativelabs/kube-router/issues/383
https://github.com/jsenon/api-cni-cleanup/blob/master/k8s/deployment.yml#L42