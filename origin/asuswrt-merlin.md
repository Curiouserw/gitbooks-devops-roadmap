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

# 参考：

- https://github.com/decoderman/amtm/tree/master
