# Aria2

# 一、简介

aria2 是一个轻量级的多协议和多源命令行下载实用程序。它支持 HTTP/HTTPS、FTP、SFTP、BitTorrent 和 Metalink。aria2 可以通过内置的 JSON-RPC 和 XML-RPC 接口进行操作。

GitHub：https://github.com/aria2/aria2

官方网站：https://aria2.github.io/

官方文档：https://aria2.github.io/manual/en/html/index.html

## BT 服务器

## DHT 网络节点数据

由于 tracker 对 BT 下载起到客户端协调和调控的重要作用，所以一旦被封锁会严重影响 BT 下载。早年中国大陆对 tracker 的封锁曾一度导致 BT 下载销声匿迹，这也促使了 DHT 网络的诞生。

DHT 网络由无数节点组成，只要是开启 DHT 功能的 BT 客户端都是一个节点，所以你也可以是其中的一份子。当接触到一个节点，通过这个节点又能接触到更多的节点，接触的节点越多，你获取资源的能力就越强，下载的速度间接也就会有提升。即使在完全不连上 Tracker 服务器的情况下，也可以很好的下载。

此外磁力链接是完全依赖于 DHT 网络的，所以如果你没办法成功连接到 DHT 节点，那么是完全无法进行下载的。所以这也是为什么有些小伙伴使用种子能下载，而磁力链接完全无法下载的原因之一。

# 二、安装配置

## 1、安装

```bash
apt-get install aria2
yum -y install aria2
opkg install aria2
```

## 2、配置样例详解

```yaml
## '#'开头为注释内容
# 是否模拟运行。如果设置为"是", aria2 将仅检查远程文件是否存在而不会下载文件内容. 此选项仅对 HTTP/FTP 下载生效. 如果设置为 true, BT 下载将会直接取消.
dry-run= false

## ============================================== 下载相关设置 ================================================ ##
# 文件的保存路径(可使用绝对路径或相对路径), 默认: 当前启动位置
dir=/data/download/aria2
# 日志文件路径
log=/data/aria2/aria2.log
# 记录日志级别
log-level=debug
# 最小文件分片大小, 添加时可指定, 取值范围1M -1024M, 默认:20M
# 假定size=10M, 文件为20MiB 则使用两个来源下载; 文件为15MiB 则使用一个来源下载
min-split-size=10M
# 同一服务器连接数, 添加时可指定, 默认:1
max-connection-per-server=5
# 启用磁盘缓存, 0为禁用缓存, 需1.16以上版本, 默认:16M
disk-cache=0
# 单个任务最大线程数, 添加时可指定, 默认:5
split=5
# 文件预分配方式, 能有效降低磁盘碎片, 默认:prealloc，预分配所需时间: none < falloc ? trunc < prealloc
# falloc和trunc则需要文件系统和内核支持、 NTFS建议使用falloc, EXT3/4建议trunc, MAC 下需要注释此项
file-allocation=none
# 最大同时下载任务数, 运行时可修改, 默认:5
max-concurrent-downloads=3
# 单个任务下载速度限制, 默认:0
max-download-limit=0
# 整体下载速度限制, 运行时可修改, 默认:0
max-overall-download-limit=0
# 整体上传速度限制, 运行时可修改, 默认:0
max-overall-upload-limit=0
lowest-speed-limit=0
retry-wait=10
max-tries=0
# 是否启用 MMap。内存中存放映射文件. 当文件空间没有预先分配至, 此选项无效. 参见 --file-allocation.
enable-mmap=false
# 单个任务上传速度限制, 默认:0
max-upload-limit=0
# 禁用IPv6, 默认:false
disable-ipv6=true
# 连接超时时间、设置建立 HTTP/FTP/代理服务器 连接的超时时间(秒). 当连接建立后, 此选项不再生效, 请使用 --timeout 选项.
connect-timeout=300

## ============================================== 下载执行额外命令 ============================================== ##
# 下载停止后执行的命令，从 正在下载 到 删除、错误、完成 时触发。暂停被标记为未开始下载，故与此项无关。
# on-download-stop=/root/.aria2/delete.sh

# 下载完成后执行的命令，此项未定义则执行 下载停止后执行的命令 (on-download-stop)
on-download-complete=/data/aria2/Scripts/download-complete.sh

# 下载错误后执行的命令，此项未定义则执行 下载停止后执行的命令 (on-download-stop)
# on-download-error=

# 下载暂停后执行的命令
# on-download-pause=

# 下载开始后执行的命令
on-download-start=/data/aria2/Scripts/start-download.sh

# BT下载完成后执行的命令
on-bt-download-complete=/data/aria2/Scripts/download-complete.sh
## ============================================== RPC相关设置 ================================================ ##
# 启用RPC, 默认:false
enable-rpc=true
# 事件轮询方式, 取值:[epoll, kqueue, port, poll, select], 不同系统默认值不同
event-poll=select
# RPC监听端口, 端口被占用时可以修改, 默认:6800
rpc-listen-port=6800
# 允许RPC所有来源, 默认:false
rpc-allow-origin-all=true
# 允许非外部访问, 默认:false
rpc-listen-all=true
# 设置的RPC授权令牌, v1.18.4新增功能, 取代 --rpc-user 和 --rpc-passwd 选项
rpc-secret=123456
## ============================================== 进度会话保存相关 ================================================ ##
# 断点续传
continue=true
# 定时保存会话, 0为退出时才保存, 需1.16.1以上版本, 默认:0
save-session-interval=60
# 在Aria2退出时保存`错误/未完成`的下载任务到会话文件
save-session=/data/aria2/aria2.session
# 从会话文件中读取下载任务
input-file=/data/aria2/aria2.session
# 强制保存会话, 即使任务已经完成, 默认:false
# 较新的版本开启后会在任务完成后依然保留.aria2文件
force-save=true
## ============================================== BT/PT下载相关 ============================================== ##
# 打开DHT功能, PT需要禁用, 默认:true
enable-dht=true
# DHT网络监听端口, 默认:6881-6999
dht-listen-port=10513
# BT监听端口, 当端口被屏蔽时使用, 默认:6881-6889
listen-port=6881-6889,10523
# 当下载的是一个种子(以.torrent结尾)时, 自动开始BT任务, 默认:true
follow-torrent=true
# 种子交换, PT需要禁用, 默认:true
enable-peer-exchange=true
# 本地节点查找, PT需要禁用, 默认:false
bt-enable-lpd=false
# BT校验相关, 默认:true
bt-hash-check-seed=true
# 单个种子最大连接数，0表示不限制，默认:55
bt-max-peers=55
# 最小分享率。当种子的分享率达到这个数时, 自动停止做种, 0为一直做种, 默认:1.0
seed-ratio=1.0

# 客户端伪装, PT需要
peer-id-prefix=-UT2210-
user-agent=uTorrent/2210(25130)
referer=*

bt-require-crypto=true
# 保存磁力链接元数据为种子文件(.torrent文件), 默认:false
bt-save-metadata=false
# 继续之前的BT任务时, 无需再次校验, 默认:false
bt-seed-unverified=false
dht-file-path=/data/aria2/dht.dat

ca-certificate=/etc/ssl/certs/ca-certificates.crt
## BT 服务器地址 ##
bt-tracker=BT 服务器
```

网友整理的Tracker 列表：https://github.com/XIU2/TrackersListCollection/blob/master/README-ZH.md

## 3、启动管理

```bash
# 后台启动
aria2c --conf-path /data/aria2/aria2.conf -D ; ps -e |grep aria2c

# 查看aria2c是否存在
ps -ef |grep aria2c

# 查看aria2c的 RPC 端口是否正常
netstat -lanp |grep 6800

# 杀掉进程
ps -ef |grep aria2c |grep -v grep |awk '{print $2}' | xargs kill -9 ;ps -ef |grep aria2c 
```

## 4、iptables放行DHT端口

```bash
aria2_dht_listen_port=`cat /data/aria2/aria2.conf | grep -w "dht-listen-port" |awk -F "=" '{print $2}'`
aria2_bt_listen_port=`cat /data/aria2/aria2.conf | grep -Eo '^listen-port=.*' | awk -F"=" '{print $2}' | sed 's/-/:/g'`

# 打开 DHT 监听端口
iptables -I INPUT -p udp --dport $aria2_dht_listen_port -j ACCEPT -m comment --comment "aria2_dht_udp_port"
iptables -I INPUT -p tcp --dport $aria2_dht_listen_port -j ACCEPT -m comment --comment "aria2_dht_tcp_port"
# 打开 BT 监听端口
iptables -I INPUT -p tcp -m multiport --dports $aria2_bt_listen_port -j ACCEPT -m comment --comment "aria2_bt_port"

# 查看开放的端口
iptables -L INPUT 

# 关闭打开的端口
iptables -D INPUT -p udp --dport $aria2_dht_listen_port -j ACCEPT -m comment --comment "aria2_dht_udp_port"
iptables -D INPUT -p tcp --dport $aria2_dht_listen_port -j ACCEPT -m comment --comment "aria2_dht_tcp_port"
iptables -D INPUT -p tcp -m multiport --dports $aria2_bt_listen_port -j ACCEPT -m comment --comment "aria2_bt_port"
```

## 5、验证测试

- 命令行下载文件

- 测试 RPC

  > curl http://192.168.1.1:6800/jsonrpc?jsoncallback=1

# 三、Web 管理页面

## 1、AriaNG

Github：https://github.com/mayswind/AriaNg

> git clone -b 1.3.6 https://github.com/mayswind/AriaNg.git  /data/aria2

```bash
server {
  listen 8089;
  error_log  /data/nginx/logs/nginx-ariang-error.log;
  access_log /data/nginx/logs/nginx-ariang-access.log  main;
  server_name localhost ;
  location / {
    root  /data/aria2/AriaNg;
    index index.html index.htm;
    autoindex on;
  }
}
```

# 四、RPC 接口

## 1、列出所有的 RPC 接口

```bash
curl -s http://127.0.0.1:16811/jsonrpc \
  -H "Content-Type: application/json"  \
  -d '{"jsonrcp":"2.0","id":"1","method":"system.listMethods","params":["token:ZTA1ODA1$aZWZjYTd"]}' \
  | jq -r '.'
  
  
{
  "id": "1",
  "jsonrpc": "2.0",
  "result": [
    "aria2.addUri",
    "aria2.addTorrent",
    "aria2.getPeers",
    "aria2.addMetalink",
    "aria2.remove",
    "aria2.pause",
    "aria2.forcePause",
    "aria2.pauseAll",
    "aria2.forcePauseAll",
    "aria2.unpause",
    "aria2.unpauseAll",
    "aria2.forceRemove",
    "aria2.changePosition",
    "aria2.tellStatus",
    "aria2.getUris",
    "aria2.getFiles",
    "aria2.getServers",
    "aria2.tellActive",
    "aria2.tellWaiting",
    "aria2.tellStopped",
    "aria2.getOption",
    "aria2.changeUri",
    "aria2.changeOption",
    "aria2.getGlobalOption",
    "aria2.changeGlobalOption",
    "aria2.purgeDownloadResult",
    "aria2.removeDownloadResult",
    "aria2.getVersion",
    "aria2.getSessionInfo",
    "aria2.shutdown",
    "aria2.forceShutdown",
    "aria2.getGlobalStat",
    "aria2.saveSession",
    "system.multicall",
    "system.listMethods",
    "system.listNotifications"
  ]
}
```

## 2、列出所有的通知接口

```bash
curl -s http://127.0.0.1:16811/jsonrpc \
  -H "Content-Type: application/json"  \
  -d '{"jsonrcp":"2.0","id":"1","method":"system.listNotifications","params":["token:****************"]}' \
  | jq -r '.'
  
{
  "id": "1",
  "jsonrpc": "2.0",
  "result": [
    "aria2.onDownloadStart",
    "aria2.onDownloadPause",
    "aria2.onDownloadStop",
    "aria2.onDownloadComplete",
    "aria2.onDownloadError",
    "aria2.onBtDownloadComplete"
  ]
}
```

## 获取下载信息

```bash
curl -s http://127.0.0.1:16811/jsonrpc \
  -H "Content-Type: application/json"  \
  -d '{"jsonrcp":"2.0","id":"1","method":"aria2.tellStatus","params":["token:********","19d68********49f64f"]}' \
  | jq -r '.'
```

# 参考

- http://ivo-wang.github.io/2019/04/18/%E5%85%B3%E4%BA%8Earia2%E6%9C%80%E5%AE%8C%E6%95%B4%E7%9A%84%E4%B8%80%E7%AF%87/
- https://www.jianshu.com/p/6adf79d29add
- https://www.senra.me/aria2-conf-file-parameters-translation-and-explanation/
- https://github.com/ngosang/trackerslist
- https://github.com/XIU2/TrackersListCollection
- https://p3terx.com/archives/solved-aria2-cant-download-magnetic-link-bt-seed-and-slow-speed.html
- # https://p3terx.github.io/aria2.conf/aria2.conf