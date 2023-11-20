# Asuswrt-Merlin

# 一、简介

官网：https://www.asuswrt-merlin.net/

文档：https://github.com/RMerl/asuswrt-merlin.ng/wiki



```bash
cru a test '*/1 * * * * date "+%Y-%m-%d %H:%M:%S" >> /tmp/testaaa1'
```



```bash
service restart_dnsmasq
```



# 二、NVRAM

```bash
nvram get odmpid

nvram get extendno

nvram get custom_clientlist 

nvram savefile 
# 所有的 ENV 会保存在/data/nvramdefault.txt
```

# 三、ATM

https://github.com/RMerl/asuswrt-merlin.ng/wiki/AMTM



# 四、Entware



# 五、OpenVPN

/usr/sbin/openvpn --version

>OpenVPN 2.6.5 arm-buildroot-linux-gnueabi [SSL (OpenSSL)] [LZO] [LZ4] [EPOLL] [MH/PKTINFO] [AEAD]
>library versions: OpenSSL 1.1.1u  30 May 2023, LZO 2.10

```
/tmp/etc/openvpn/server1
├── ca.crt
├── ca.key
├── ccd
├── client.ovpn
├── config.ovpn
├── dh.pem
├── fw_nat.sh
├── fw.sh
├── ovpn-down -> /sbin/rc
├── ovpn-up -> /sbin/rc
├── server.crt
├── server.key
├── status
└── vpn-watchdog1.sh
```

参考：

- 
- https://github.com/decoderman/amtm/tree/master
