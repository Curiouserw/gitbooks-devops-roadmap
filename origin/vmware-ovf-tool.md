# OVF 管理工具VMWare OVF Tool

# 一、简介

VMWare OVF Tool是 一个可以在VMWare系列产品上导入导出虚拟机为OVF/OVA格式文件的命令行工具。

ESXI强大的客户端vSpere Client由于在MacOS上没有对应的版本。VMWare Fusion在MacOS上有客户端能对远程ESXI进行基本简单的管理操作！但是只能部署客户端本地的OVF/OVA文件，不能部署存储在远程Web服务器上的OVF模板文件到远程ESXI服务器上。在MacOS上，只能通过组合VMware Fusion+VMware OVF Tool+govc来实现Windows下vSphere Client的大部分功能。



**官方文档**：

1. https://www.vmware.com/support/developer/ovf/ovf410/ovftool-410_userguide.pdf

2. https://vmware.github.io/vic-product/assets/files/html/1.5/vic_vsphere_admin/deploy_vic_appliance_ovftool.html

# 二、安装配置

**下载地址**（需注册登录）：https://my.vmware.com/zh/group/vmware/details?downloadGroup=OVFTOOL430&productId=742 

MacOS dmg格式安装后的目录为：`/Applications/VMware OVF Tool`。命令没有系统环境变量中，通过设置软连接实现：`ln -s /Applications/VMware\ OVF\ Tool/ovftool /usr/local/bin`

## 配置文件

- 通用型的配置项可以写在配置文件中
- 注释行以`#`开头
- 配置项是ovftool的命令行参数，一行一个配置项
- 配置文件读取顺序为先全局配置文件`~/.ovftool`，后读取本地配置文件`.ovftool`

例如：`~/.ovftool` 

```bash
# 指定vSphere存储池位置
datastore=datastore1
# 指定磁盘类型
diskMode=thin
# 跳过vSphere连接的SSL认证
noSSLVerify
# 部署完成后开机
powerOn
# 接受所有用户的licenses
acceptAllEulas
# 指定日志输出级别
X:logLevel=error
# 指定日志输出文件
X:logFile=/tmp/ovftool.log
# 将日志输出控制台
X:logToConsole
# 跳过OVF原数据校验
skipManifestCheck
# 输出机器可读日志
machineOutput
# 在OVF中注入ENV配置
X:injectOvfEnv
# ESXI局域网适配器
network='VM Network=VM Network'
```

查看所有配置文件的配置项

```bash
ovftool --help config

# Contents of global configuration options (/Users/curiouser/.ovftool):
#  datastore=datastore1
#  diskMode=thin
#  noSSLVerify
#  powerOn
#  .....

# Currently no local configuration options in .ovftool
```

# 三、命令详解

## 命令格式

```bash
ovftool [参数项] 源目标 目的目标
```

### 源目标

源目标可以是以下资源

- OVF/OVA文件(a local file path, or an HTTP, HTTPS, or FTP URL)
- 本地虚拟机文件 (.vmx格式)
- vCenter, ESXi或VMware Server上的vAPP/locatoridentifying a virtual machine
- A vCloud Director locator identifying a virtual machine or a vApp in vCloud Director. 
- A local file path to a vApprun workspace entity.

### 目的目标

目的目标可以是以下资源

- A local file path for VMX, OVF, OVA, or vApprun workspace.
- A vSphere locator identifying a cluster, host, or a vSphere location.
- A vCloud Director locator identifying a virtual machine or a vApp in vCloud Director.

#### vSphere目的目标格式

```bash
vi://<username>:<password>@<host>:<port>/<search-term>
```

### 参数项

```bash
--acceptAllEulas            : Accept all end-user licenses agreements without being prompted.
--allowAllExtraConfig       : Whether we allow all the ExtraConfig options. These options are a security risk as they control low-level and potential unsafe options on the VM.
--allowExtraConfig          : Whether we allow ExtraConfig options. These options are a security risk as they control low-level and potential unsafe options on the VM.
--annotation                : Add annotation to vi, vmx, vapprun, vCloud, OVF, and OVA source locators
--authdPortSource           : Use this to override default vmware authd port (902) when using a host as source.
--authdPortTarget           : Use this to override default vmware authd port (902) when using a host as target.
--chunkSize                 : Specifies the chunk size to use for files in a generated OVF package. The default is not to chunk. The chunk size without unit is assumed to be in megabytes. Accepted units
                              are b, kb, mb, gb; e.g., 2gb or 100kb.
--compress                  : Compress the disks in an OVF package. Value must be between 1 and 9. 1 is the fastest, but gives the worst compression, whereas 9 is the slowest, but gives the best
                              compression.
--computerName              : Sets the computer name in the guest for a VM using the syntax --computerName:<VM ID>=<value>. Only applies to vCloud targets version 5.5 or newer.
--coresPerSocket            : Specifies the distribution of the total number of CPUs over a number of virtual sockets using the syntax --coresPerSocket:<VM ID>=<value>. Only applies to vCloud targets version 5.5 or newer.
-ds/--datastore             : Target datastore name for a VI locator.
--decodeBase64              : Decode option values with Base64.
--defaultStorageProfile     : The storage profile for all VMs in the OVF package. The value should be an SPBM profile ID. Only applies to VI targets version 5.5 or newer.
--defaultStorageRawProfile  : The storage profile for all VMs in the OVF package. The value should be raw SPBM profile. The value will overwrite that in --defaultStorageProfile. 
                             Only applies to VI targets version 5.5 or newer.
--deploymentOption          : Selects what deployment option to use (if the source OVF package supports multiple options.)
--disableVerification       : Skip validation of signature and certificate.
-dm/--diskMode              : Select target disk format. Supported formats are: monolithicSparse, monolithicFlat, twoGbMaxExtentSparse, twoGbMaxExtentFlat, seSparse (VI target), 
                            : eagerZeroedThick (VI target), thin (VI target), thick (VI target), sparse, and flat
--diskSize                  : Sets the size of a VM disk in megabytes using the syntax --diskSize:<VM ID>,<disk instance ID>=<value>. Only applies to vCloud targets version 5.5 or newer.
--eula                      : EULA to be inserted in the first virtual system or virtual system collection in the OVF. If the EULA is in a file, use the option --eula@=filename instead.
--exportDeviceSubtypes      : Enables export of resource subtype for CD/Floppy/Parallel/Serial devices. This can limit portability as not all device backings are supported on all hypervisors.
                            : The default is false.
--exportFlags               : Specifies one or more export flags to control what gets exported. The supported values for VI sources are mac, uuid, and extraconfig. 
                            : Supported value for vCloud sources are preserveIdentity. One or more options can be provided, separated by commas.
--extraConfig               : Sets an ExtraConfig element for all VirtualHardwareSections. The syntax is --extraConfig:<key>=<value>. Applies to vi, vmx, vapprun, vCloud, ovf, and ova source
                              locators.
--fencedMode                : If a parent network exists on the vCloud target, this property specifies the connectivity to the parent. Possible values are bridged, isolated, and natRouted.
-h /--help                  : Prints this message.
--hideEula                  : In OVF probe mode, hides the EULA.
--ipAllocationPolicy        : IP allocation policy for a deployed OVF package.Supported values are: dhcpPolicy, transientPolicy, fixedPolicy, fixedAllocatedPolicy.
--ipProtocol                : Select what IP protocol to use (IPv4, IPv6).
--lax                       : Relax OVF specification conformance and virtual hardware compliance checks. Use only if you know what you are doing.
--locale                    : Selects locale for target.
--machineOutput             : Output OVF Tool messages in a machine friendly manner.
--makeDeltaDisks            : Build delta disk hierarchy from the given source locator.
--maxVirtualHardwareVersion : The maximal virtual hardware version to generate.
--memorySize                : Sets the memory size in megabytes of a VM using the syntax --memorySize:<VM ID>=<value>. Only applies to vCloud targets version 5.5 or newer.
-n /--name                  : Specifies target name (defaults to source name).
--net                       : Set a network assignment in the deployed OVF package. A network assignment is set using the syntax --net:<OVF name>=<target name>. If the target is vCloud 5.5 or newer,
                            : a fence mode can also be specified using the syntax --net:<OVF name>=<target name>,<fence mode>. Possible fence mode values are: bridged, isolated, and natRouted.
-nw/--network               : Target network for a VI deployment.
--nic                       : Specifies NIC configuration in a VM using the syntax --nic:<VM ID>,<index>=<OVF net name>,<isPrimary>,<ipAddressingMode>,<ipAddress>. 
                            : Possible values for ipAddressingMode are: DHCP, POOL, MANUAL, and NONE. ipAddress is optional and should only be used when ipAddressingMode is set to MANUAL. Only applies to vCloud targets version 5.5 or newer.
--noDisks                   : Disable disk conversion.
--noImageFiles              : Do not include image files in destination.
--noSSLVerify               : Skip SSL verification for VI connections.
--numberOfCpus              : Sets the number of CPUs for a VM using the syntax --numberOfCpus:<VM ID>=<value>. Only applies to vCloud targets version 5.5 or newer.
-o /--overwrite             : Force overwrites of existing files.
--powerOffSource            : Ensures a VM/vApp is powered off before importing from a VI source.
--powerOffTarget            : Ensures a VM/vApp is powered off before overwriting a VI target.
--powerOn                   : Powers on a VM/vApp deployed on a VI target.
--privateKey                : Sign OVF package with the given private key (.pem file). The file must contain a private key and a certificate.
--privateKeyPassword        : Password for the private key. Should be used in conjunction with privateKey if the private key requires password authentication.
                            : If required and not specified, the tool will prompt for the password.
--prop                      : Set a property in the deployed OVF package. A property is set using the syntax --prop:<key>=<value>.
--proxy                     : Proxy used for HTTP[S] access.
--proxyNTLMAuth             : Enable NTLM authentication for proxy.
-q /--quiet                 : No output to screen except errors.
--schemaValidate            : Validate OVF descriptor against OVF schema.
--shaAlgorithm              : Select SHA digest algorithm when creating OVF package. Supported values are SHA1, SHA256 and SHA512. Default value is SHA256.
--skipManifestCheck         : Skip validation of OVF package manifest.
--skipManifestGeneration    : Skip generation of OVF package manifest.
--sourcePEM                 : File path to PEM formatted file used to verify VI connections.
--sourceSSLThumbprint       : SSL fingerprint of SOURCE. OVF Tool verifies the SSL fingerprint it gets from SOURCE if the value is set.
-st/--sourceType            : Explicitly express that source is OVF, OVA, VMX, VI, vCloud, ISO, FLP, vApprun
--sslCipherList             : Use this to override default OpenSSL ciphers suite.
--sslVersion                : Use this to set preferred TLS/SSL version for HTTPS connections. The valid values are as following:
                                TLSv1_0: Set preferred TLS/SSL version to TLSv1.0.
                                TLSv1_1: Set preferred TLS/SSL version to TLSv1.1.
                                TLSv1_2: Set preferred TLS/SSL version to TLSv1.2.
--storageProfile            : Sets the storage profile for a VM using the syntax --storageProfile:<VM ID>=<value>.Only applies to vCloud targets version 5.5 or newer.
--targetPEM                 : File path to PEM formatted file used to verify VI connections.
--targetSSLThumbprint       : SSL fingerprint of TARGET. OVF Tool verifies the SSL fingerprint it gets from TARGET if the value is set.
-tt/--targetType            : Explicitly express that target is OVF, OVA, VMX, VI, vCloud, ISO, FLP, vApprun
--vCloudTemplate            : Create only a vApp template. Default value is false
--vService                  : Set a vService assignment in the deployed OVF package. A vService assignment is set using the syntax --vService:<dependencyId>=<providerId>.
--verifyOnly                : Do not upload the source but only verify it against the target host. Applies to VI 4 targets only.
-v /--version               : Prints the version of this tool.
--viCpuResource             : Specify the CPU resource settings for VI-locator targets. The syntax is --viCpuResource=<shares>:<reservation>:<limit>.
--viMemoryResource          : Specify the CPU resource settings for VI-locator targets. The syntax is --viMemoryResource=<shares>:<reservation>:<limit>.
-vf/--vmFolder              : Target VM folder in VI inventory (relative to datacenter).
```

## 帮助命令格式

```bash
ovftool --help 文档主题

# 文档主题
 locators    : For detailed source and destination locator syntax
 examples    : For examples of use
 config      : For syntax of configuration files
 debug       : For debug purpose
 integration : For a list of options primarily used when ovftool is exec'ed
               from another tool or shellscript.
```

## 快速操作命令别名

```bash
echo "nvm_2C4G80G() { ovftool --name=\"\$@\" http://192.168.1.7:32770/repository/tools/ovf/empty/2C4G80G.ovf vi://root:****@192.168.1.103 ;}" >> ~/.zshrc

echo "nvm_4C8G100G() { ovftool --name=\"\$@\" http://192.168.1.7:32770/repository/tools/ovf/empty/4C8G100G.ovf vi://root:****@192.168.1.103 ;}" >> ~/.zshrc

source ~/.zshrc
```

# 四、操作实例

## 1、将存储在Nexus RAW仓库中的OVF模板部署遇到远程ESXI中

```bash
ovftool \
--name="test1" \
--X:injectOvfEnv \
--X:logFile=./ovftool.log \
--X:enableHiddenProperties \
--allowExtraConfig \
--machineOutput \
--X:waitForIp \
http://192.168.1.7:8080/repository/tools/ovf/empty/2C4G80G.ovf \
vi://root:'*******'@192.168.1.103

# Nexus中的OVF模板文件标注VMDK位置需要修改为HTTP协议类型的URL地址
```

## 2、导出远程ESXI中虚拟机的OVF模板到本地

```bash
ovftool vi://root:'***'@192.168.1.103/empty-6C20G100G ./empty-6C20G100G.ovf
```

## 3、导入本地OVF模板到远程ESXI中

```bash
ovftool -ds=hdd -dm=thin -n=VM名字 本地OVF模板文件路径 vi://root@192.168.1.103
# -dm=thin是精简置备的意思，-n是新主机的名字
```

# 五、问题

1、导入VM OVF模板时报`A general system error occurred: Fault cause: vim.fault.FileNotFound`

**原因：**

​	`OVF模板文件中配置的有CDROM设备，而CDROM有使用ISO文件的挂载，而本地可能没有对应的ISO文件`

**解决方案：**

​	`编辑VM的OVF文件，搜索iso关键字，找到对应的设备定义，注释或删除掉`

**参考：**https://communities.vmware.com/t5/Open-Virtualization-Format-Tool/ovftool-fails-with-Error-vim-fault-FileNotFound/m-p/2649133





# 参考

1. http://www.mamicode.com/info-detail-1301117.html
2. https://vmware.github.io/vic-product/assets/files/html/1.5/vic_vsphere_admin/deploy_vic_appliance_ovftool.html
3. https://www.virtuallyghetto.com/2014/07/quick-tip-handy-ovftool-4-0-advanced-options.html