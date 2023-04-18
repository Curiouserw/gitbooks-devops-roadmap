# 一、Master节点上的组件

- **kube-apiserver**：对外暴露了Kubernetes API。它是的 Kubernetes 前端控制层。它被设计为水平扩展，即通过部署更多实例来缩放。
- **kube-controller-manager**：运行控制器，它们是处理集群中常规任务的后台线程。逻辑上，每个控制器是一个单独的进程，但为了降低复杂性，它们都被编译成独立的可执行文件，并在单个进程中运行。这些控制器包括:
    - 节点控制器: 当节点移除时，负责注意和响应。
    - 副本控制器: 负责维护系统中每个副本控制器对象正确数量的 Pod。
    - 端点控制器: 填充端点(Endpoints) 对象(即连接 Services & Pods)。
    - 服务帐户和令牌控制器: 为新的命名空间创建默认帐户和 API 访问令牌
- **kube-scheduler**：监视没有分配节点的新创建的 Pod，选择一个节点供他们运行。
- **etcd**：用于 Kubernetes 的后端存储。存储所有集群数据。
- **cloud-controller-manager**：用于与底层云提供商交互的控制器。云控制器管理器可执行组件是 Kubernetes v1.6 版本中引入的 Alpha 功能。仅运行云提供商特定的控制器循环。您必须在 - kube-controller-manager 中禁用这些控制器循环，您可以通过在启动 kube-controller-manager 时将 --cloud-provider 标志设置为external来禁用控制器循环。允许云供应商代码和 Kubernetes 核心彼此独立发展，在以前的版本中，Kubernetes 核心代码依赖于云提供商特定的功能代码。在未来的版本中，云供应商的特定代码应由云供应商自己维护，并与运行 Kubernetes 的云控制器管理器相关联。以下控制器具有云提供商依赖关系:
    - 节点控制器: 用于检查云提供商以确定节点是否在云中停止响应后被删除
    - 路由控制器: 用于在底层云基础架构中设置路由
    - 服务控制器: 用于创建，更新和删除云提供商负载平衡器
    - 数据卷控制器: 用于创建，附加和装载卷，并与云提供商进行交互以协调卷

# 二、Node节点上的组件

- **kubelet**：是主要的节点代理,它监测已分配给其节点的 Pod(通过 apiserver 或通过本地配置文件)，提供如下功能:
    - 挂载 Pod 所需要的数据卷(Volume)。
    - 下载 Pod 的 secrets。
    - 通过 Docker 运行(或通过 rkt)运行 Pod 的容器。
    - 周期性的对容器生命周期进行探测。
    - 如果需要，通过创建镜像Pod（Mirror Pod） 将 Pod 的状态报告回系统的其余部分。
    - 将节点的状态报告回系统的其余部分。
- **kube-proxy**：通过维护主机上的网络规则并执行连接转发，实现了Kubernetes服务抽象
- **Container Runtime**：运行容器的底层平台。Kubernetes支持的容器平台：`Docker`、`containerd`、`cri-o`、`rktlet`

# 三、插件

## 网络插件

- **[ACI](https://www.github.com/noironetworks/aci-containers)**: provides integrated container networking and network security with Cisco ACI.
- **[Calico](https://docs.projectcalico.org/latest/getting-started/kubernetes/)** is a secure L3 networking and network policy provider.
- **[Canal](https://github.com/tigera/canal/tree/master/k8s-install)** unites Flannel and Calico, providing networking and network policy.
- **[Cilium](https://github.com/cilium/cilium)** is a L3 network and network policy plugin that can enforce HTTP/API/L7 - policies transparently. Both routing and overlay/encapsulation mode are - supported.
- **[CNI-Genie](https://github.com/Huawei-PaaS/CNI-Genie)** enables Kubernetes to seamlessly connect to a choice of CNI plugins,such as Calico, Canal, Flannel, Romana, or Weave.
- **[Contiv](http://contiv.github.io/)** provides configurable networking (native L3 using BGP, overlay using - vxlan, classic L2, and Cisco-SDN/ACI) for various use cases and a rich policy - framework. Contiv project is fully open sourced. The installer provides both - kubeadm and non-kubeadm based installation options.
- **[Contrail](http://www.juniper.net/us/en/products-services/sdn/contrail/contrail-networking/)**, based on Tungsten Fabric, is a open source, multi-cloud network virtualization and policy management platform. Contrail and Tungsten Fabric are integrated with orchestration systems such as Kubernetes, OpenShift, OpenStack and Mesos, and provide isolation modes for virtual machines, containers/pods and bare metal workloads.
- **[Flannel](https://github.com/coreos/flannel/blob/master/Documentation/kubernetes.md)** is an overlay network provider that can be used with Kubernetes.
- **[Knitter](https://github.com/ZTE/Knitter/)** is a network solution supporting multiple networking in Kubernetes.
- **[Multus](https://github.com/Intel-Corp/multus-cni)** is a Multi plugin for multiple network support in Kubernetes to support - all CNI plugins (e.g. Calico, Cilium, Contiv, Flannel), in addition to SRIOV, - DPDK, OVS-DPDK and VPP based workloads in Kubernetes.
- **[NSX-T](https://docs.vmware.com/en/VMware-NSX-T/2.0/nsxt_20_ncp_kubernetes.pdf)** Container Plug-in (NCP) provides integration between VMware NSX-T and - container orchestrators such as Kubernetes, as well as integration between - NSX-T and container-based CaaS/PaaS platforms such as Pivotal Container Service - (PKS) and OpenShift.
- **[Nuage](https://github.com/nuagenetworks/nuage-kubernetes/blob/v5.1.1-1/docs/kubernetes-1-installation.rst)** is an SDN platform that provides policy-based networking between - Kubernetes Pods and non-Kubernetes environments with visibility and security - monitoring.
- **[Romana](http://romana.io/)** is a Layer 3 networking solution for pod networks that also supports the - NetworkPolicy API. Kubeadm add-on installation details available here.
- **[Weave Net](https://www.weave.works/docs/net/latest/kube-addon/)** provides networking and network policy, will carry on working on both - sides of a network partition, and does not require an external database

## 服务发现插件

- **[CoreDNS](https://coredns.io/)** is a flexible, extensible DNS server which can be installed as the in-cluster DNS for pods

## 可视化及控制插件

- **[Dashboard](https://github.com/kubernetes/dashboard#kubernetes-dashboard)** is a dashboard web interface for Kubernetes.
- **[Weave Scope](https://www.weave.works/documentation/scope-latest-installing/#k8s)** is a tool for graphically visualizing your containers, pods, services etc. Use it in conjunction with a Weave Cloud account or host the UI yourself

# 参考链接

https://kubernetes.io/docs/concepts/cluster-administration/addons/





