





```json
{
  "routing": {...},
  "inbounds": [
    {
      ...
    },
    {
      "port": 12345, //开放的端口号
      "protocol": "dokodemo-door",
      "settings": {
        "network": "tcp,udp",
        "followRedirect": true // 这里要为 true 才能接受来自 iptables 的流量
      },
      "sniffing": {
        "enabled": true,
        "destOverride": ["http", "tls"]
      }
    }
  ],
  "outbounds": [
    {
      ...
      "streamSettings": {
        ...
        "sockopt": {
          "mark": 255  //这里是 SO_MARK，用于 iptables 识别，每个 outbound 都要配置；255可以改成其他数值，但要与下面的 iptables 规则对应；如果有多个 outbound，最好将所有 outbound 的 SO_MARK 都设置成一样的数值
        }
      }
    }
    ...
  ]
}
```

设定 TCP 透明代理的 iptables 规则，命令如下(#代表注释)：



```bash
 # 新建一个名为 V2RAY 的链
iptables -t nat -N V2RAY
# 直连 192.168.0.0/16 
iptables -t nat -A V2RAY -d 192.168.0.0/16 -j RETURN 
# 直连 SO_MARK 为 0xff 的流量(0xff 是 16 进制数，数值上等同与上面配置的 255)，此规则目的是避免代理本机(网关)流量出现回环问题
iptables -t nat -A V2RAY -p tcp -j RETURN -m mark --mark 0xff 
# 其余流量转发到 12345 端口（即 V2Ray）
iptables -t nat -A V2RAY -p tcp -j REDIRECT --to-ports 12345 
# 对局域网其他设备进行透明代理
iptables -t nat -A PREROUTING -p tcp -j V2RAY 
# 对本机进行透明代理
iptables -t nat -A OUTPUT -p tcp -j V2RAY 



# 查看链表规则
iptables -L V2RAY -t nat
```

然后设定 UDP 流量透明代理的 iptables 规则，命令如下

```bash
ip rule add fwmark 1 table 100
ip route add local 0.0.0.0/0 dev lo table 100
iptables -t mangle -N V2RAY_MASK
iptables -t mangle -A V2RAY_MASK -d 192.168.0.0/16 -j RETURN
iptables -t mangle -A V2RAY_MASK -p udp -j TPROXY --on-port 12345 --tproxy-mark 1
iptables -t mangle -A PREROUTING -p udp -j V2RAY_MASK

```





- https://blog.indigo.codes/2021/11/20/iptables-tproxy-and-home/
- https://guide.v2fly.org/app/transparent_proxy.html#%E4%BC%98%E7%82%B9
- https://guide.v2fly.org/app/tproxy.html#%E8%AE%BE%E7%BD%AE%E7%BD%91%E5%85%B3