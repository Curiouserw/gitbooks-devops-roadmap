# 一、Openshift容器网络简介

Openshift容器网络默认是基于Open vSwitch（OVS）实现的。

Openshift提供两种网络方案：
* ovs-subnet(子网模式)：为集群节点上的容器提供一个扁平化的二层虚拟网路，所有在这个二层网路中容器可直接通信。
* ovs-multitenet(多租户模式)：基于项目的网络隔离，即不同项目间的容器之间不能直接通信。启动多租户网络隔离后，每个项目创建后都会被分配一个虚拟网络ID（Virtual Network ID ,VNID）.OVS网桥会为该项目的所有数据流量标记上VNID，在默认情况下，只有数据包上的VNID与目标容器所在项目的VNID匹配上后，数据包才允许被转发到目标容器中。当有些项目的容器应用是通过公共服务的，后期可通过配置将多个项目见的网络连通，或者将项目设置为全局可访问。

# 二、启动多租户网络

需要将集群中所有的master节点配置文件`/etc/origin/master/master-config.yaml`和node节点配置文件`/etc/origin/node/node-config.yaml`中的`networkPluginName`的属性值从`redhat/openshift-ovs-subnet`修改为`redhat/openshift-ovs-multitenant`，然后重启Openshift集群Master节点的`origin-master-controllers.service`服务和Node节点的`origin-node.service服`务

# 三、测试，查看网络隔离

1. 在一个项目中的一个pod的终端中`ping/telnet/curl/nslook`另一个项目中的pod的ip地址或者对应svc的FQDN`（<service>.<pod_namespace>.svc.cluster.local）`
   
2. 查看namespace的Netid是否一致
    ```bash
    $ oc get netnamespaces
    NAME                                NETID      EGRESS IPS
    default                             0          []
    kube-public                         5899696    []
    kube-service-catalog                0          []
    demo                                13843039   []
    dubbo                               11344186   []
    jenkins                             13843039   []
    ```
    当NETID相同时，表示这个两个project的网络是相通的
    
    当NETID为0时，表示这个Project的网络全局可访问

# 四、连通隔离的网络

    # project 1,2,3中所有的pod，service可以通过容器IP相互访问（通过service的FQDN不能相互访问）
    oc adm pod-network join-projects --to=<project1> <project2> <project3>
    
    #将某个project中所有的pod和service设置为全局可访问
    oc adm pod-network make-projects-global <project2> <project3>

# 参考链接

https://docs.okd.io/3.11/admin_guide/managing_networking.html
