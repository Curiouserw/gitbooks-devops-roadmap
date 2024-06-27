# vSphere

# 一、ESXI简介

VMware vSphere是VMware的服务器虚拟化软件套件，vSphere 中的核心组件为 VMware ESXi（取代原ESX），ESXi是一款可以独立安装和运行在祼机上的系统，因此与他我们以往见过的VMwareWorkstation 软件不同的是它不再依存于宿主操作系统之上。它是裸机（bare-metal）虚拟化管理程序（hypervisor），用于创建和管理虚拟机（VM）。ESXi直接安装在服务器硬件上，不依赖底层操作系统，使其性能和效率更高。

# 二、硬件兼容性查询

对于老硬件、不知名硬件的主机。在安装 ESXI 前，可以在以下网址查询硬件是否支持，能支持到什么版本。

查询条件可根据硬件设备类型，硬件厂商等

## 1、硬件兼容性

- **CPU兼容性：**https://www.vmware.com/resources/compatibility/search.php?deviceCategory=cpu
- **IO设备**：https://www.vmware.com/resources/compatibility/search.php?deviceCategory=io
- **GPU直通**：https://www.vmware.com/resources/compatibility/search.php?deviceCategory=vmdirect
- ......

## 2、已知不兼容硬件

- **主板自带的螃蟹卡网卡**
- **老nvme版本协议的 SSD硬盘**
  - 老协议NVME驱动在`6.7 U2`被砍掉了，7.0以上版本砍掉了大量老协议规范及驱动
  - ESXI支持的NVME协议最低是1.3
  - 参考：
    - https://williamlam.com/2019/05/quick-tip-crucial-nvme-ssd-not-recognized-by-esxi-6-7.html
    - https://www.pcjsb.com/archives/esxi6.7-7.0he-8.0wu-fa-shi-bie-lao-xie-yi-nvmegu-tai-ying-pan-de-jie-jue-fang-fa

# 三、ESXI安装

## 1、刻录ESXI ISO文件到U盘

## 2、开起主板的虚拟化配置

## 3、从U盘启动

## 4、进入安装程序
