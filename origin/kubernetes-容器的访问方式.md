# Kubernetes容器的访问方式



# 一、简介

当在kubernetes中使用docker镜像启动成一个容器，形成一个POD时，kubernetes的CNI组件(例如Calico、OVS)会随机动态给分配一个IP地址。通过访问这个POD IP地址加应用服务监听暴露出来的端口即可访问POD中的服务。可是POD生命周期是动态化的，IP地址重启后会改变。为屏蔽POD IP地址的动态变化和对多POD实例的负载均衡，引入了Service这个资源对象。

# 二、Service

Service的类型大致可分成4种：

- **ClusterIP**： 默认方式。根据是否生成ClusterIP又可分为普通Service和Headless Service两类：
  - **普通Service**：通过为Kubernetes的Service分配一个集群内部可访问的固定虚拟IP（Cluster IP），实现集群内的访问。为最常见的方式。
  - **Headless Service**：该服务不会分配Cluster IP，也不通过kube-proxy做反向代理和负载均衡。而是通过DNS提供稳定的网络ID来访问，DNS会将headless service的后端直接解析为podIP列表。主要供StatefulSet使用。

- **NodePort**：通过将service的port映射到集群内每个节点的相同一个端口，实现通过nodeIP:nodePort从集群外访问服务。
- **LoadBalancer**：和nodePort类似，不过除了使用一个Cluster IP和nodePort之外，还会向所使用的公有云申请一个负载均衡器(负载均衡器后端映射到各节点的nodePort)，实现从集群外通过LB访问服务。
- **ExternalName**：是 Service 的特例。此模式主要面向运行在集群外部的服务，通过它可以将外部服务映射进k8s集群，且具备k8s内服务的一些特征（如具备namespace等属性），来为集群内部提供服务。此模式要求kube-dns的版本为1.7或以上。这种模式和前三种模式（除headless service）最大的不同是重定向依赖的是dns层次，而不是通过kube-proxy。



Service中主要涉及三种Port： * `port` 这里的port表示service暴露在clusterIP上的端口，clusterIP:Port 是提供给集群内部访问kubernetes服务的入口。

- targetPort
  containerPort，targetPort是pod上的端口，从port和nodePort上到来的数据最终经过kube-proxy流入到后端pod的targetPort上进入容器。

- nodePort
  nodeIP:nodePort 是提供给从集群外部访问kubernetes服务的入口。

总的来说，port和nodePort都是service的端口，前者暴露给从集群内访问服务，后者暴露给从集群外访问服务。从这两个端口到来的数据都需要经过反向代理kube-proxy流入后端具体pod的targetPort，从而进入到pod上的容器内。

1.3 IP
使用Service服务还会涉及到几种IP：

- ClusterIP
  Pod IP 地址是实际存在于某个网卡(可以是虚拟设备)上的，但clusterIP就不一样了，没有网络设备承载这个地址。它是一个虚拟地址，由kube-proxy使用iptables规则重新定向到其本地端口，再均衡到后端Pod。当kube-proxy发现一个新的service后，它会在本地节点打开一个任意端口，创建相应的iptables规则，重定向服务的clusterIP和port到这个新建的端口，开始接受到达这个服务的连接。

- Pod IP
  Pod的IP，每个Pod启动时，会自动创建一个镜像为gcr.io/google_containers/pause的容器，Pod内部其他容器的网络模式使用container模式，并指定为pause容器的ID，即：network_mode: "container:pause容器ID"，使得Pod内所有容器共享pause容器的网络，与外部的通信经由此容器代理，pause容器的IP也可以称为Pod IP。

- 节点IP
  Node-IP，service对象在Cluster IP range池中分配到的IP只能在内部访问，如果服务作为一个应用程序内部的层次，还是很合适的。如果这个service作为前端服务，准备为集群外的客户提供业务，我们就需要给这个服务提供公共IP了。指定service的spec.type=NodePort，这个类型的service，系统会给它在集群的各个代理节点上分配一个节点级别的端口，能访问到代理节点的客户端都能访问这个端口，从而访问到服务。

# 三、Ingress





# 参考

1. https://blog.csdn.net/liukuan73/article/details/82585732
2. 