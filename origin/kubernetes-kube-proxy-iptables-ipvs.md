# Kube-proxy的iptables与ipvs实现方式

# 一、什么是IPVS？

IPVS (IP Virtual Server，IP虚拟服务器)是基于Netfilter的、作为linux内核的一部分实现传输层负载均衡的技术，通常称为第4层LAN交换。

IPVS集成在LVS(Linux Virtual Server)中，它在主机中运行，并在真实服务器集群前充当负载均衡器。IPVS可以将对TCP/UDP服务的请求转发给后端的真实服务器，并使真实服务器的服务在单个IP地址上显示为虚拟服务。因此IPVS天然支持Kubernetes Service。

# 二、为什么选择IPVS？

随着kubernetes使用量的增长，其资源的可扩展性变得越来越重要。特别是对于使用kubernetes运行大型工作负载的开发人员或者公司来说，service的可扩展性至关重要。

kube-proxy是为service构建路由规则的模块，之前依赖iptables来实现主要service类型的支持，比如(ClusterIP和NodePort)。但是iptables很难支持上万级的service，因为iptables纯粹是为防火墙而设计的，并且底层数据结构是内核规则的列表。

kubernetes早在1.6版本就已经有能力支持5000多节点，这样基于iptables的kube-proxy就成为集群扩容到5000节点的瓶颈。举例来说，如果在一个5000节点的集群，我们创建2000个service，并且每个service有10个pod，那么我们就会在每个节点上有至少20000条iptables规则，这会导致内核非常繁忙。

ipvs (IP Virtual Server) 实现了传输层负载均衡，也就是我们常说的4层LAN交换，作为Linux 内核的一部分。ipvs运行在主机上，在真实服务器集群前充当负载均衡器。ipvs可以将基于TCP和UDP的服务请求转发到真实服务器上，并使真实服务器的服务在单个IP 地址上显示为虚拟服务。

基于IPVS的集群内负载均衡就可以完美的解决这个问题。IPVS是专门为负载均衡设计的，并且底层使用哈希表这种非常高效的数据结构，几乎可以允许无限扩容。

# 三、IPVS与IPTABLES的区别

IPVS模式在Kubernetes v1.8中引入，并在v1.9中进入了beta。 1.11中实现了GA(General Availability)。IPTABLES模式在v1.1中添加，并成为自v1.2以来的默认操作模式。 IPVS和IPTABLES都基于netfilter。 IPVS模式和IPTABLES模式之间的差异如下：

- IPVS为大型集群提供了更好的可扩展性和性能。
- IPVS支持比iptables更复杂的负载平衡算法（最小负载，最少连接，位置，加权等）。
- IPVS支持服务器健康检查和连接重试等。

# 四、IPVS要求

1. k8s版本 >= v1.11
2. 使用ipvs需要安装相应的工具来处理”yum install ipset ipvsadm -y“
3. 确保 ipvs已经加载内核模块， ip_vs、ip_vs_rr、ip_vs_wrr、ip_vs_sh、nf_conntrack_ipv4(如果这些内核模块不加载，当kube-proxy启动后，会退回到iptables模式)

# 五、IPVS原理分析



# 参考

1. https://blog.csdn.net/fanren224/article/details/86548398
2. https://www.jianshu.com/p/89f126b241db
3. https://segmentfault.com/a/1190000016333317