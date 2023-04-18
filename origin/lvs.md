# LVS

# 一、简介

## 简介

## 工作原理

### **1、基于NAT的LVS模式负载均衡**

   NAT（Network Address Translation）即网络地址转换，原理是是通过修改数据报头，让内网IP地址可以访问外网，以及外部用用户可以访问位于内网的IP主机。LVS负载调度器可以使用两块网卡配置不同的IP地址，eth0设置为私有IP与内部网络通过交换设备相互连接，eth1设备为外网IP与外部网络联通。

​    第一步，用户访问到负载均衡的外网地址，该LVS外网IP称为Virtual IP Address（VIP），用户通过访问VIP，从而间接连接到后端的真实服务器（Real Server），并且这一切用户都是无感知的，用户以为自己访问的就是真实服务器，但用户不知道被访问的VIP仅仅是一个调度器，这个调度器后端还有服务器。

  第二步，用户将请求发送至eth1，此时LVS将根据预设的算法选择后端的一台真实服务器，将数据请求包转发给真实服务器，并且在转发之前LVS会修改数据包中的目标地址以及目标端口，目标地址与目标端口将被修改为选出的真实服务器IP地址以及相应的端口。

  第三步，真实的服务器将响应数据包返回给LVS调度器，调度器在得到响应的数据包后会将源地址和源端口修改为VIP及调度器相应的端口，修改完成后，由调度器将响应数据包发送回终端用户，另外，由于LVS调度器有一个连接Hash表，该表中会记录连接请求及转发信息，当同一个连接的下一个数据包发送给调度器时，从该Hash表中可以直接找到之前的连接记录，并根据记录信息选出相同的真实服务器及端口信息。

### **2、基于TUN的LVS负载均衡**

​    在LVS（NAT）模式的集群环境中，由于所有的数据请求及响应的数据包都需要经过LVS调度器转发，如果后端服务器的数量大于10台，则调度器就会成为整个集群环境的瓶颈。我们知道，数据请求包往往远小于响应数据包的大小。因为响应数据包中包含有客户需要的具体数据，所以LVS（TUN）的思路就是将请求与响应数据分离，让调度器仅处理数据请求，而让真实服务器响应数据包直接返回给客户端。其中，IP隧道（IP tunning）是一种数据包封装技术，它可以将原始数据包封装并添加新的包头（内容包括新的源地址及端口、目标地址及端口），从而实现将一个目标为调度器的VIP地址的数据包封装，通过隧道转发给后端的真实服务器（Real Server），通过将客户端发往调度器的原始数据包封装，并在其基础上添加新的数据包头（修改目标地址为调度器选择出来的真实服务器的IP地址及对应端口），LVS（TUN）模式要求真实服务器可以直接与外部网络连接，真实服务器在收到请求数据包后直接给客户端主机响应数据。

### **3、基于DR的LVS负载均衡**

在LVS（TUN）模式下，由于需要在LVS调度器与真实服务器之间创建隧道连接，这同样会增加服务器的负担。与LVS（TUN）类似，DR模式也叫直接路由模式，该模式中LVS依然仅承担数据的入站请求以及根据算法选出合理的真实服务器，最终由后端真实服务器负责将响应数据包发送返回给客户端。与隧道模式不同的是，直接路由模式（DR模式）要求调度器与后端服务器必须在同一个局域网内，VIP地址需要在调度器与后端所有的服务器间共享，因为最终的真实服务器给客户端回应数据包时需要设置源IP为VIP地址，目标IP为客户端IP，这样客户端访问的是调度器的VIP地址，回应的源地址也依然是该VIP地址（真实服务器上的VIP），客户端是感觉不到后端服务器存在的。由于多台计算机都设置了同样一个VIP地址，所以在直接路由模式中要求调度器的VIP地址是对外可见的，客户端需要将请求数据包发送到调度器主机，而所有的真实服务器的VIP地址必须配置在Non-ARP的网络设备上，也就是该网络设备并不会向外广播自己的MAC及对应的IP地址，真实服务器的VIP对外界是不可见的，但真实服务器却可以接受目标地址VIP的网络请求，并在回应数据包时将源地址设置为该VIP地址。调度器根据算法在选出真实服务器后，在不修改数据报文的情况下，将数据帧的MAC地址修改为选出的真实服务器的MAC地址，通过交换机将该数据帧发给真实服务器。整个过程中，真实服务器的VIP不需要对外界可见。

## IPVS调度算法

针对不同的网络服务需求和服务器配置，IPVS调度器实现了如下八种负载调度算法:

- 轮叫(Round Robin)
- 加权轮叫(Weighted Round Robin)
- 最少链接(Least Connections)
- 加权最少链接(Weighted Least Connections)
- 基于局部性的最少链接(Locality-Based Least Connections)
- 带复制的基于局部性最少链接(Locality-Based Least Connections with Replication)
- 目标地址散列(Destination Hashing)
- 源地址散列(Source Hashing)

# 二、

# 三、操作

## 1、安装

```bash
yum install ipvsadm
```

## 2、前端服务管理

```bash
# 添加前端服务
ipvsadm -A -t 192.16.1.16:80 -s rr

# 修改前端服务的调度算法
ipvsadm -E -t 192.16.1.16:80 -s wlc

# 删除所有前端服务
ipvsadm -C
```

## 3、后端服务器管理

```bash
# 添加后端服务器
ipvsadm -a -t 192.16.1.16:80 -r 192.16.1.11 -g -w 1
ipvsadm -a -t 192.16.1.16:80 -r 192.16.1.15 -g -w 1

# 删除集群服务中的一个真实服务器
ipvsadm -d -t 192.16.1.16:80 -r 192.16.0.11

# 修改集群服务中的一个真实服务器的权重值
ipvsadm -e -t 192.16.1.16:80 -r 192.16.0.11:80 -m -w 2
```

## 4、查看

```bash
ipvsadm -Lcn

# 显示速率信息
ipvsadm -L --rate
# 对虚拟服务器和真是服务器排序输出
ipvsadm -L --sort
# 显示统计信息
ipvsadm -L --stats
# 显示tcp tcpfin udp 的timeout值
ipvsadm -L --timeout
```

## 5、规则管理

```bash
# 保存规则到文件
ipvsadm -Sn > /etc/sysconfig/ipvsadm
# 加载规则文件
ipvsadm -R < /etc/sysconfig/ipvsadm
```

## 6、SystemD服务管理规则

> /usr/lib/systemd/system/ipvsadm.service

```ini
[Unit]
Description=Initialise the Linux Virtual Server
After=syslog.target network.target
 
[Service]
Type=oneshot
ExecStart=/bin/bash -c "exec /sbin/ipvsadm-restore < /etc/sysconfig/ipvsadm"   #重启服务器后，默认从/etc/sysconfig/ipvsadm该文件中读取规则，将ipvsadm服务设置开机自启动后开机将自动加载规则
ExecStop=/bin/bash -c "exec /sbin/ipvsadm-save -n > /etc/sysconfig/ipvsadm"    #停止ipvsadm服务时(执行systemctl stop ipvsadm时)，自动将规则写入/etc/sysconfig/ipvsadm中
ExecStop=/sbin/ipvsadm -C
RemainAfterExit=yes
[Install]
WantedBy=multi-user.target.
```

`systemctl daemon-reload && systemctl start ipvsadm` 

## 7、ipvs管理

```bash
# 清空转发请求计数器
ipvsadm -Z
```

# 参考

- https://blog.csdn.net/weixin_34406061/article/details/92974711
- https://blog.csdn.net/yangyijun1990/article/details/109145347
- https://blog.csdn.net/weixin_49172531/article/details/116144786
- https://cloud.tencent.com/developer/article/2054205
- https://juejin.cn/post/7169552758167568392
- https://blog.csdn.net/lupengfei1009/article/details/86514445
- https://blog.csdn.net/weixin_34406061/article/details/92974711