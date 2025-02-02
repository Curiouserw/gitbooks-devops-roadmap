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

获取硬件温度

```bash
# ARMv7 CPU
cat /proc/dmu/temperature

# ARMv8 CPU
cat /sys/class/thermal/thermal_zone0/temp | awk '{print $1 / 1000}'

# 2.4GHz
wl -i `nvram get wl0_ifname` phy_tempsense | awk '{print $1 / 2 + 20}'

# 5GHz
wl -i `nvram get wl1_ifname` phy_tempsense | awk '{print $1 / 2 + 20}'
```

https://www.snbforums.com/threads/how-to-read-the-temperature-without-merlin.80167/

https://www.snbforums.com/threads/script-to-monitor-temperatures-on-rt-ac68u.68775/

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

# 六 、KoolCenter

```bash
#!/bin/sh

# Check if the comm command is installed
if ! command -v comm &> /dev/null; then
  echo "The comm command is not installed.  Install Now"
  opkg install coreutils-comm
fi

# Get the current time
now=$(date +"%Y-%m-%d %H:%M:%S")

# Get the list of connected devices
devices=$(brctl showmacs br0 | awk '{print $2}')

# Check if the list of devices has changed
if [ ! -f /tmp/connected_devices ]; then
  # This is the first time the script is running, so save the list of devices to a file
  echo "$devices" > /tmp/connected_devices
else
  # Get the list of devices that were connected previously
  previous_devices=$(cat /tmp/connected_devices)

  # Compare the list of devices that are connected now to the list of devices that were connected previously
  new_devices=$(comm -23 <(echo "$devices") <(echo "$previous_devices"))

  # Send a notification to DingTalk for each new device
  for device in $new_devices; do
    curl -X POST https://oapi.dingtalk.com/robot/send \
      -H "Content-Type: application/json" \
      -d "{\"msgtype\": \"text\", \"text\": {\"content\": \"New device connected to bridge br0: $device ($now)\"}}"
  done

  # Update the list of connected devices
  echo "$devices" > /tmp/connected_devices
fi
```

# 七、开机自启与定时任务

## 1、开机自启脚本

https://github.com/RMerl/asuswrt-merlin.ng/wiki/User-scripts

- **`/jffs/scripts/services-start：`**系统服务启动后，间接通过`/koolshare/bin/ks-services-start.sh`调用`/koolshare/init.d/`路径下**V**开头的脚本
- **`/jffs/scripts/services-stop：`**重启时系统服务关闭后，间接通过`/koolshare/bin/ks-services-stop.sh`调用`/koolshare/init.d/`路径下**T**开头的脚本。(间接通过`/opt/etc/init.d/rc.unslung`调用`/opt/etc/init.d/路径下T开头的脚本`,将stop参数传递过去)
- **`/jffs/scripts/wan-start：`**WAN网卡启动之后，间接通过`/koolshare/bin/ks-wan-start.sh`调用`/koolshare/init.d/`路径下**S**开头的脚本，将start参数传递过去。但此时可能拨号仍未成功，网络会不太稳定
- **`/jffs/scripts/nat-start：`**：在 NAT 规则（即端口转发等）应用于 NAT 表后调用，间接通过`/koolshare/bin/ks-nat-start.sh`脚本调用`/koolshare/init.d/`路径下**N**开头的脚本，将start_nat参数传递过去。可在此处放置自己的 NAT 表自定义规则
- **`/jffs/scripts/post-mount：`**  在磁盘分区挂载后，间接通过`/koolshare/bin/ks-mount-start.sh`调用`/koolshare/init.d/`路径下**M**开头的脚本，将`start`和`$1`参数传递过去。`$1`通常为挂载点路径，例如`/tmp/mnt/sd`
- **`/jffs/scripts/unmount：`**在磁盘分区卸载后，间接通过`/koolshare/bin/ks-unmount.sh`调用`/koolshare/init.d/`路径下**U**开头的脚本，将`$1`参数传递过去。`$1`通常为挂载点路径，例如`/tmp/mnt/sd`

## 2、Cru定时任务

```bash
# 查看定时任务
cru l
# 添加定时任务
cru a 任务名 '0 22 * * 5 sh /tmp/mnt/sd/backup.sh'
```

# 八、应用

## 1、获取硬件温度输出Prometheus Metrics格式

```bash
#!/bin/bash
PROM_STAT_FILE=/tmp/muprome/temp.prom

# 获取 CPU 温度，设置异常处理
if CPUS_temp=$(cat /sys/class/thermal/thermal_zone0/temp 2>/dev/null); then
    CPUS_temp_celsius=$(echo "scale=1; $CPUS_temp / 1000" | bc)
else
    CPUS_temp_celsius=0
fi

# 获取 2.5GHz WiFi 温度，设置异常处理
if NET25_temp=$(wl -i eth6 phy_tempsense 2>/dev/null | awk '{print $1 / 2 + 20}'); then
    :
else
    NET25_temp=0
fi

# 获取 5GHz WiFi 温度，设置异常处理
if NET5S_temp=$(wl -i eth7 phy_tempsense 2>/dev/null | awk '{print $1 / 2 + 20}'); then
    :
else
    NET5S_temp=0
fi

{
  # 输出设备名称和温度
  echo "node_hwmon_chip_names{chip=\"CPU\", chip_name=\"CPU\"} 1"
  echo "node_hwmon_temp_celsius{chip=\"CPU\", chip_name=\"CPU\"} $CPUS_temp_celsius"

  echo "node_hwmon_chip_names{chip=\"2.5GHz WiFi\", chip_name=\"2.5GHz WiFi\"} 1"
  echo "node_hwmon_temp_celsius{chip=\"2.5GHz WiFi\", chip_name=\"2.5GHz WiFi\"} $NET25_temp"

  echo "node_hwmon_chip_names{chip=\"5GHz WiFi\", chip_name=\"5GHz WiFi\"} 1"
  echo "node_hwmon_temp_celsius{chip=\"5GHz WiFi\", chip_name=\"5GHz WiFi\"} $NET5S_temp"
} > "$PROM_STAT_FILE"
```

# 参考：

- https://github.com/decoderman/amtm/tree/master
