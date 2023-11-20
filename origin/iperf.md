# 网络工具 iperf

# 一、简介

# 二、安装

```bash
apt install -y iperf iperf3
yum install -y iperf iperf3
```

# 三、命令详解

# 四、实际应用

## 1、网络测速

在要测试机上启动一个 iperf3 服务器(192.168.1.7)，等待来自客户端的连接。

> iperf3 -s -p 111

在另一台计算机上作为客户端，运行以下命令，客户端将开始向服务器发送数据，同时测量网络的带宽和其他性能指标。

> iperf3 -c <服务器IP> -p 111
> 
> Server listening on TCP port 111
> TCP window size: 85.3 KByte (default)
> ------------------------------------------------------------
> [  4] local 192.168.1.7 port 111 connected with 192.168.1.35 port 54716 (peer 2.1.5)
> [ ID] Interval       Transfer     Bandwidth
> [  4]  0.0-10.0 sec  1.10 GBytes   941 Mbits/sec

知识回顾之二进制：

- `1 Bytes = 8 bits`
- `1 KB(Kilobyte) = 1024 Bytes`
- `1 MB(Megabyte) = 1024 *1024 Bytes`

