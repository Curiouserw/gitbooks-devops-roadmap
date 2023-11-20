# 基于PXE的网络OS工具Netboot.xyz

# 一、简介

Github：https://github.com/netbootxyz/netboot.xyz

文档：https://netboot.xyz/docs/

# 二、部署

## 1、docker

## 2、ansible

## 3、手动

# 三、Asuswrt-Merlin

https://netboot.xyz/docs/kb/networking/asuswrt-merlin

`curl -o /jffs/tftproot/netboot.xyz.kpxe https://boot.netboot.xyz/ipxe/netboot.xyz.kpxe`

`curl -o /jffs/tftproot/netboot.xyz.efi https://boot.netboot.xyz/ipxe/netboot.xyz.efi`

`touch /jffs/configs/dnsmasq.conf.add`

```bash
enable-tftp
tftp-root=/jffs/tftproot
dhcp-match=set:bios,60,PXEClient:Arch:00000
dhcp-boot=tag:bios,netboot.xyz.kpxe,,192.168.1.1
dhcp-match=set:efi32,60,PXEClient:Arch:00002
dhcp-boot=tag:efi32,netboot.xyz.efi,,192.168.1.1
dhcp-match=set:efi32-1,60,PXEClient:Arch:00006
dhcp-boot=tag:efi32-1,netboot.xyz.efi,,192.168.1.1
dhcp-match=set:efi64,60,PXEClient:Arch:00007
dhcp-boot=tag:efi64,netboot.xyz.efi,,192.168.1.1
dhcp-match=set:efi64-1,60,PXEClient:Arch:00008
dhcp-boot=tag:efi64-1,netboot.xyz.efi,,192.168.1.1
dhcp-match=set:efi64-2,60,PXEClient:Arch:00009
dhcp-boot=tag:efi64-2,netboot.xyz.efi,,192.168.1.1
```

# 参考：

- http://www.360doc.com/content/19/0805/08/39899686_853175915.shtml