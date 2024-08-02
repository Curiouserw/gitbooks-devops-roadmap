# VMware vSphere虚拟化

# 一、ESXI简介

VMware vSphere是VMware的服务器虚拟化软件套件，vSphere 中的核心组件为 VMware ESXi（取代原ESX），ESXi是一款可以独立安装和运行在祼机上的系统，因此与他我们以往见过的VMwareWorkstation 软件不同的是它不再依存于宿主操作系统之上。它是裸机（bare-metal）虚拟化管理程序（hypervisor），用于创建和管理虚拟机（VM）。ESXi直接安装在服务器硬件上，不依赖底层操作系统，使其性能和效率更高。

官方文档：https://docs.vmware.com/cn/VMware-vSphere/index.html



| 端口                   | 协议    | 源                               | 对象              | 目的                                                         |
| ---------------------- | ------- | -------------------------------- | ----------------- | ------------------------------------------------------------ |
| 9                      | UDP     | vCenter Server                   | ESXi 主机         | wake on LAN 使用。                                           |
| 22                     | TCP     | SSH 客户端                       | ESXi 主机         | SSH 访问需要使用此端口                                       |
| 53                     | UDP     | ESXi 主机                        | DNS 服务器        | DNS 客户端                                                   |
| 68                     | UDP     | DHCP 服务器                      | ESXi 主机         | 适用于 IPv4 的 DHCP 客户端                                   |
| 80                     | TCP     | Web 浏览器                       | ESXi 主机         | “欢迎使用”页面，包含不同界面的下载链接                       |
| 161                    | UDP     | SNMP 服务器                      | ESXi 主机         | 允许主机连接到 SNMP 服务器                                   |
| 427                    | TCP/UDP | CIM 服务器                       | ESXi 主机         | CIM 客户端使用服务位置协议版本 2 (SLPv2) 查找 CIM 服务器     |
| 546                    | TCP/UDP | DHCP 服务器                      | ESXi 主机         | 适用于 IPv6 的 DHCP 客户端                                   |
| 547                    | TCP/UDP | ESXi 主机                        | DHCP 服务器       | 适用于 IPv6 的 DHCP 客户端                                   |
| 902                    | TCP/UDP | VMware vCenter Agent             | ESXi 主机         | vCenter Server Agent                                         |
| 2233                   | TCP     | ESXi 主机                        | vSAN 传输         | vSAN 可靠数据报传输。使用 TCP 并用于 vSAN 存储 IO。如果禁用，则 vSAN 无法工作。 |
| 3260                   | TCP     | ESXi 主机                        | 软件 iSCSI 客户端 | 支持软件 iSCSI                                               |
| 5671                   | TCP     | ESXi 主机                        | rabbitmqproxy     | 在 ESXi 主机上运行的代理，允许虚拟机内部运行的应用程序与 vCenter 网络域中运行的 AMQP 代理进行通信。虚拟机不必位于网络中，即无需网卡。代理将连接到 vCenter 网络域中的代理。因此，出站连接 IP 地址应至少包括当前正在使用的代理或未来的代理。如果客户要扩展，则可以添加代理。 |
| 5988/8889              | TCP     | CIM 服务器8889 OpenWSMAN守护进程 | ESXi 主机         | 5988 - 适用于 CIM（通用信息模型）的服务器<br>8889 - Web 服务管理（WS 管理是用于管理服务器、设备、应用程序和 Web 服务的 DMTF 开放式标准） |
| 5989                   | TCP     | CIM 安全服务器                   | ESXi 主机         | 适用于 CIM 的安全服务器                                      |
| 6999                   | UDP     | NSX 分布式逻辑路由器服务         | ESXi 主机         | NSX 虚拟分布式路由器服务 如果已安装 NSX VIB 且已创建 VDR 模块，则与此服务关联的防火墙端口将打开。如果没有 VDR 实例与主机关联，则该端口无需打开。 |
| 8000                   | TCP     | ESXi 主机                        | ESXi 主机         | vMotion - 通过 vMotion 迁移虚拟机时为必需项。ESXi 主机在端口 8000 上侦听远程 ESXi 主机中用于 vMotion 流量的 TCP 连接 |
| 8080                   | TCP     | vsanvp                           | ESXi 主机         | VSAN VASA 供应商提供程序。由 vCenter 中的存储管理服务 (SMS) 使用，以访问有关 Virtual SAN 存储配置文件、功能和合规性的信息。如果禁用，则 Virtual SAN 基于存储配置文件的管理 (SPBM) 无法工作。 |
| 8100<br/>8200<br/>8300 | TCP/UDP | Fault Tolerance                  | ESXi 主机         | 主机之间的流量，用于 vSphere Fault Tolerance (FT)。          |
| 8301<br/>8302          | UDP     | DVSSync                          | ESXi 主机         | DVSSync 端口可用于同步已启用 VMware FT 记录/重放的主机之间的分布式虚拟端口的状况。只有运行主虚拟机或备份虚拟机的主机才须打开这些端口。未使用 VMware FT 的主机无需打开这些端口。 |
| 12345<br/>23451        | UDP     | ESXi 主机                        | vSAN 群集服务     | 由 vSAN 使用的群集监控、成员资格和 Directory Service。       |
| 44046<br/>31031        | TCP     | ESXi 主机                        | HBR               | 用于 vSphere Replication 和 VMware Site Recovery Manager 的持续复制流量。 |
| 80,9000                | TCP     | ESXi 主机                        | vCenter Server    | vSphere Lifecycle Manager                                    |

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

# 四、其他

# 五、SOAP API 

## 1、获取Session、Cookie

```bash
#!/bin/bash
esxi_host="https://192.168.1.1/sdk"
username="root"
userpassword="*******"

getsession_req_xml="<Envelope xmlns=\"http://schemas.xmlsoap.org/soap/envelope/\" xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"><Header><operationID>esxui-bc44</operationID></Header><Body><Login xmlns=\"urn:vim25\"><_this type=\"SessionManager\">ha-sessionmgr</_this><userName>$username</userName><password>$userpassword</password><locale>zh-CN</locale></Login></Body></Envelope>"

response=$(curl -skil -D - -X POST "$esxi_host" -H 'Content-Type: text/xml' --data-raw "$getsession_req_xml")

# 提取 Set-Cookie 头并保存到变量中
cookie=$(echo "$response" | grep -i 'Set-Cookie' | sed 's/Set-Cookie: \(.*\);.*/\1/' | head -n 1 | awk -F";" '{print $1}')
# 提取响应体中的 key 并保存到变量中
key=$(echo "$response" | grep "LoginResponse" | sed -n 's/.*<key>\(.*\)<\/key>.*/\1/p')
```

## 2、ESXI关机

```bash
shutdown_req_xml="<Envelope xmlns=\"http://schemas.xmlsoap.org/soap/envelope/\" xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"><Header><operationID>esxui-a514</operationID></Header><Body><ShutdownHost_Task xmlns=\"urn:vim25\"><_this type=\"HostSystem\">ha-host</_this><force>true</force></ShutdownHost_Task></Body></Envelope>"

curl -sk -X POST "$esxi_host" \
--header "vmware_soap_session: $key" \
--header "Cookie: $cookie " \
--header 'Content-Type: text/xml' \
--data-raw "$shutdown_req_xml"
```

- 从vSphere 7.x开始不支持 root用户修改系统文件
  - 参考：https://knowledge.broadcom.com/external/article/344767/the-root-account-can-no-longer-change-pe.html

- 要安装自定义 VIB，必须将 ESXi 主机的接受程度改为 CommunitySupported。
  - https://knowledge.broadcom.com/external/article/317482/creating-custom-firewall-rules-in-vmware.html
  - https://elatov.github.io/2018/03/creating-a-vib-to-modify-a-firewall-rule-on-an-esxi-host/
  - https://docs.vmware.com/cn/VMware-vSphere/7.0/com.vmware.vsphere.security.doc/GUID-8912DD42-C6EA-4299-9B10-5F3AEA52C605.html
  - https://docs.vmware.com/cn/VMware-vSphere/7.0/com.vmware.vsphere.security.doc/GUID-171B99EA-15B3-4CC5-8B9A-577D8336FAA0.html





- https://blogs.vmware.com/vsphere/files/2012/09/vibauthor-how-to-v0.1.docx
- https://www.altaro.com/vmware/how-to-create-persistent-firewall-rules-on-esxi/
- https://williamlam.com/2023/07/creating-a-custom-vib-for-esxi-8-x.html
- https://www.veeam.com/kb2291
- https://elatov.github.io/2018/03/creating-a-vib-to-modify-a-firewall-rule-on-an-esxi-host/
- https://blog.csdn.net/aplsc/article/details/109284462
- https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.security.doc/GUID-A45872F4-C47A-4B16-8221-7BE2F363AC38.html
- https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.security.doc/GUID-7A8BEFC8-BF86-49B5-AE2D-E400AAD81BA3.html
- https://docs.vmware.com/cn/VMware-vSphere/7.0/com.vmware.vsphere.security.doc/GUID-A45872F4-C47A-4B16-8221-7BE2F363AC38.html
