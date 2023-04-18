# vSphere go命令行管理工具govc

# 一、简介

VMware vSphere APIs (ESXi and/or vCenter)的go语言客户端

- [govc](https://github.com/vmware/govmomi/blob/master/govc) - vSphere CLI
- [vcsim](https://github.com/vmware/govmomi/blob/master/vcsim) - vSphere API mock framework
- [toolbox](https://github.com/vmware/govmomi/blob/master/toolbox) - VM guest tools framework

支持的ESXi / vCenter版本：

-  ESXi / vCenter 6.0, 6.5 , 6.7 (5.5和5.1版本功能部分支持, 但官方不再支持)

**GitHub地址**：https://github.com/vmware/govmomi

**govc下载地址**：https://github.com/vmware/govmomi/releases

**govc使用手册**：https://github.com/vmware/govmomi/blob/master/govc/USAGE.md

# 二、安装配置

## 1、安装

在[govc下载地址](https://github.com/vmware/govmomi/releases)下载对应平台的二进制包

```bash
curl -L $URL_TO_BINARY | gunzip > /usr/local/bin/govc
chmod +x /usr/local/bin/govc
```

MacOS对应`govc_darwin_amd64.gz`

## 2、配置

govc是通过设置环境变量进行配置的。

- `GOVC_URL`：ESXi或vCenter实例的地址

  默认协议`https` ，URL路径为 `/sdk` 。可在URL中设置用户名密码，例如： `https://user:pass@host/sdk`.

  如果用户名密码中包含特殊字符( `\`, `#` , `:`)，可以在 `GOVC_USERNAME` ， `GOVC_PASSWORD` 单独设置用户名密码。

- `GOVC_USERNAME`：用户名

- `GOVC_PASSWORD`：密码

- `GOVC_TLS_CA_CERTS`：指定CA证书

  ```bash
  $ export GOVC_TLS_CA_CERTS=~/.govc_ca.crt
  # 多证书设置
  $ export GOVC_TLS_CA_CERTS=~/ca-certificates/bar.crt:~/ca-certificates/foo.crt
  ```

- `GOVC_TLS_KNOWN_HOSTS`：指定验证证书的指纹

  ```bash
  $ export GOVC_TLS_KNOWN_HOSTS=~/.govc_known_hosts
  $ govc about.cert -u host -k -thumbprint | tee -a $GOVC_TLS_KNOWN_HOSTS
  $ govc about -u user:pass@host
  ```

- `GOVC_TLS_HANDSHAKE_TIMEOUT`: TLS握手的超时时间

- `GOVC_INSECURE`：关闭证书验证

  ```bash
  export GOVC_INSECURE=1
  ```

- `GOVC_DATACENTER`：

- `GOVC_DATASTORE`：

- `GOVC_NETWORK`：

- `GOVC_RESOURCE_POOL`：

- `GOVC_HOST`：

- `GOVC_GUEST_LOGIN`：

- `GOVC_VIM_NAMESPACE`：

- `GOVC_VIM_VERSION`：

以上变量可在`~/.zshrc或/etc/profile或~/.bashrc`中设置，同时可使用`govc env`查看设置。

# 三、VM的创建

## 命令详情

```bash
Usage: govc vm.create [OPTIONS] NAME

Create VM.

For a list of possible '-g' IDs, see:
http://pubs.vmware.com/vsphere-6-5/topic/com.vmware.wssdk.apiref.doc/vim.vm.GuestOsDescriptor.GuestOsIdentifier.html

Examples:
  govc vm.create -on=false vm-name
  govc vm.create -cluster cluster1 vm-name # use compute cluster placement
  govc vm.create -datastore-cluster dscluster vm-name # use datastore cluster placement
  govc vm.create -m 2048 -c 2 -g freebsd64Guest -net.adapter vmxnet3 -disk.controller pvscsi vm-name

Options:
  -annotation=           VM description
  -c=1                   Number of CPUs
  -cluster=              Use cluster for VM placement via DRS
  -datastore-cluster=    Datastore cluster [GOVC_DATASTORE_CLUSTER]
  -disk=                 Disk path (to use existing) OR size (to create new, e.g. 20GB)
  -disk-datastore=       Datastore for disk file
  -disk.controller=scsi  Disk controller type
  -ds=                   Datastore [GOVC_DATASTORE]
  -firmware=bios         Firmware type [bios|efi]
  -folder=               Inventory folder [GOVC_FOLDER]
  -force=false           Create VM if vmx already exists
  -g=otherGuest          Guest OS ID
  -host=                 Host system [GOVC_HOST]
  -iso=                  ISO path
  -iso-datastore=        Datastore for ISO file
  -link=true             Link specified disk
  -m=1024                Size in MB of memory
  -net=                  Network [GOVC_NETWORK]
  -net.adapter=e1000     Network adapter type
  -net.address=          Network hardware address
  -on=true               Power on VM
  -pool=                 Resource pool [GOVC_RESOURCE_POOL]
  -version=              ESXi hardware version [5.0|5.5|6.0|6.5|6.7]
```

## 示例

```bash
govc vm.create -m 2048 -c 2 -disk=30G -host.ip=192.168.1.8 test1
```

# 四、VM的管理

具体命令详解可查看文档：https://github.com/vmware/govmomi/blob/master/govc/USAGE.md

## 1、查询操作

### 查看所有VM

```bash
govc find . -type m
```

### 查看所有开机的VM

```bash
govc find . -type m -runtime.powerState poweredOn
```

## 2、VM电源的开启与关闭

```bash
vmname=test
# 开启VM电源
govc vm.power -on -M $vmname
# 关闭VM电源
govc vm.power -off -M $vmname
```

## 3、VM的销毁

使用VMWare OVF Tool部署OVF/OVA模板到远程ESXI，详见[OVF 管理工具VMWare OVF Tool](vmware-ovf-tool.md)

### 关闭VM电源，并删除VM

```bash
govc vm.destroy vm_name
```

## 4、在VM中进行的操作

### Prerequisite

- VM安装VMware-Tools工具后进行重启
  - 可直接使用包管理工具安装,例如:`yum install -y open-vm-tools`
  - 手动省略，太麻烦

- 设置要访问VM的名字及登录用户密码

  ```bash
  GOVC_GUEST_LOGIN="root:******"
  # 如果密码中包含特殊字符“!”，使用"\"进行转义."@"不需要转义
  vmname="test"
  ```

### 命令

```bash
govc guest.* -vm $vmname 
```

- **guest.chmod**：修改VM中文件的权限

- **guest.chown**：设置VM中文件的所有者

- **guest.df**：显示VM中文件的使用情况

  ```bash
  govc guest.df -vm $vmname
  ```

- **guest.download**：拷贝VM中的文件到本地

- **guest.getenv**：查看VM中的环境变量

- **guest.kill**：杀掉VM中的进程

- **guest.ls**：查看VM中的文件系统

  ```bash
  # 例如查看指定VM中“/root”下的文件夹
  govc guest.ls -vm $vmname /root
  ```

- **guest.mkdir**：在VM中创建文件夹

  ```bash
  govc guest.mkdir -vm $vmname /root/test
  ```

- **guest.mktemp**：在VM中创建临时文件或文件夹

- **guest.mv**：在VM中移动文件

- **guest.ps**：查看VM中的进程

- **guest.rm**：删除VM中的文件

- **guest.rmdir**：删除VM中的文件夹

- **guest.run**：在VM中运行命令，并显示输出结果

  ```bash
  Usage: govc guest.run [OPTIONS] PATH [ARG]...
  
  Examples:
    govc guest.run -vm $vmname ifconfig
    govc guest.run -vm $vmname ifconfig eth0
    cal | govc guest.run -vm $vmname -d - cat
    govc guest.run -vm $vmname -d "hello $USER" cat
    govc guest.run -vm $vmname curl -s :invalid: || echo $? # exit code 6
    govc guest.run -vm $vmname -e FOO=bar -e BIZ=baz -C /tmp env
  
  Options:
    -C=                    The absolute path of the working directory for the program to start
    -d=                    Input data string. A value of '-' reads from OS stdin
    -e=[]                  Set environment variables
    -i=false               Interactive session
    -l=:                   Guest VM credentials [GOVC_GUEST_LOGIN]
    -vm=                   Virtual machine [GOVC_VM]
  
  govc guest.run -l $GOVC_GUEST_LOGIN -vm $vmname sh -c /root/beforeShutDown.sh
  ```

- **guest.start**：在VM中启动程序，并显示输出结果

  ```bash
  Usage: govc guest.start [OPTIONS] PATH [ARG]...
  
  Examples:
    govc guest.start -vm $vmname /bin/mount /dev/hdb1 /data
    pid=$(govc guest.start -vm $vmname /bin/long-running-thing)
    govc guest.ps -vm $vmname -p $pid -X
  
  Options:
    -C=                    The absolute path of the working directory for the program to start
    -e=[]                  Set environment variable (key=val)
    -i=false               Interactive session
    -l=:                   Guest VM credentials [GOVC_GUEST_LOGIN]
    -vm=                   Virtual machine [GOVC_VM]
  ```

- **guest.touch**：在VM中创建文件

- **guest.upload**：上传本地文件到VM中

  ```bash
  govc guest.upload -vm $vmname ./**.tar.gz /root/***.tar.gz
  ```

## 5、VM的磁盘管理

### ①创建新磁盘挂载到虚拟机中

```bash
Usage: govc vm.disk.create [OPTIONS]

Create disk and attach to VM.

Examples:
  govc vm.disk.create -vm $name -name $name/disk1 -size 10G
  govc vm.disk.create -vm $name -name $name/disk2 -size 10G -eager -thick -sharing sharingMultiWriter

Options:
  -controller=           Disk controller
  -ds=                   Datastore [GOVC_DATASTORE]
  -eager=false           Eagerly scrub new disk
  -mode=persistent       Disk mode (persistent|nonpersistent|undoable|independent_persistent|independent_nonpersistent|append)
  -name=                 Name for new disk
  -sharing=              Sharing (sharingNone|sharingMultiWriter)
  -size=10.0GB           Size of new disk
  -thick=false           Thick provision new disk
  -vm=                   Virtual machine [GOVC_VM]
```

### ②挂载已创建的VMDK磁盘到虚拟机中

```bash
Usage: govc vm.disk.attach [OPTIONS]

Attach existing disk to VM.

Examples:
  govc vm.disk.attach -vm $name -disk $name/disk1.vmdk
  govc vm.disk.attach -vm $name -disk $name/shared.vmdk -link=false -sharing sharingMultiWriter
  govc device.remove -vm $name -keep disk-* # detach disk(s)

Options:
  -controller=           Disk controller
  -disk=                 Disk path name
  -ds=                   Datastore [GOVC_DATASTORE]
  -link=true             Link specified disk
  -mode=                 Disk mode override (persistent|nonpersistent|undoable|independent_persistent|independent_nonpersistent|append)
  -persist=true          Persist attached disk
  -sharing=              Sharing (sharingNone|sharingMultiWriter)
  -vm=                   Virtual machine [GOVC_VM]
```

### ③更新VM磁盘配置

可用于扩容磁盘大小

```bash
Usage: govc vm.disk.change [OPTIONS]

Change some properties of a VM's DISK

In particular, you can change the DISK mode, and the size (as long as it is bigger)

Examples:
  govc vm.disk.change -vm VM -disk.key 2001 -size 10G
  govc vm.disk.change -vm VM -disk.label "BDD disk" -size 10G
  govc vm.disk.change -vm VM -disk.name "hard-1000-0" -size 12G
  govc vm.disk.change -vm VM -disk.filePath "[DS] VM/VM-1.vmdk" -mode nonpersistent

Options:
  -disk.filePath=        Disk file name
  -disk.key=0            Disk unique key
  -disk.label=           Disk label
  -disk.name=            Disk name
  -mode=                 Disk mode (persistent|nonpersistent|undoable|independent_persistent|independent_nonpersistent|append)
  -sharing=              Sharing (sharingNone|sharingMultiWriter)
  -size=0B               New disk size
  -vm=                   Virtual machine [GOVC_VM]
```

## 6、查看VM信息

```bash
Usage: govc vm.info [OPTIONS] VM...

Display info for VM.

The '-r' flag displays additional info for CPU, memory and storage usage,
along with the VM's Datastores, Networks and PortGroups.

Examples:
  govc vm.info $vm
  govc vm.info -r $vm | grep Network:
  govc vm.info -json $vm
  govc find . -type m -runtime.powerState poweredOn | xargs govc vm.info

Options:
  -e=false               Show ExtraConfig
  -g=true                Show general summary
  -r=false               Show resource summary
  -t=false               Show ToolsConfigInfo
  -waitip=false          Wait for VM to acquire IP address
```

## 7、VM的快照

### ①创建VM快照

```bash
Usage: govc snapshot.remove [OPTIONS] NAME

Remove snapshot of VM with given NAME.

NAME can be the snapshot name, tree path, moid or '*' to remove all snapshots.

Examples:
  govc snapshot.remove -vm my-vm happy-vm-state

Options:
  -c=true                Consolidate disks
  -r=false               Remove snapshot children
  -vm=                   Virtual machine [GOVC_VM]
```

### ②快照恢复

```bash
Usage: govc snapshot.revert [OPTIONS] [NAME]

Revert to snapshot of VM with given NAME.

If NAME is not provided, revert to the current snapshot.
Otherwise, NAME can be the snapshot name, tree path or moid.

Examples:
  govc snapshot.revert -vm my-vm happy-vm-state

Options:
  -s=false               Suppress power on
  -vm=                   Virtual machine [GOVC_VM]
```

### ③查看快照

```bash
Usage: govc snapshot.tree [OPTIONS]

List VM snapshots in a tree-like format.

The command will exit 0 with no output if VM does not have any snapshots.

Examples:
  govc snapshot.tree -vm my-vm
  govc snapshot.tree -vm my-vm -D -i -d

Options:
  -C=false               Print the current snapshot name only
  -D=false               Print the snapshot creation date
  -c=true                Print the current snapshot
  -d=false               Print the snapshot description
  -f=false               Print the full path prefix for snapshot
  -i=false               Print the snapshot id
  -vm=                   Virtual machine [GOVC_VM]
```

# 五、连接VM Console

## 1、Prerequisite

- 安装VMWare Remote Console

  下载地址：https://my.vmware.com/web/vmware/details?downloadGroup=VMRC1101&productId=742

## 2、命令详解

```bash
govc vm.console --help
Usage: govc vm.console [OPTIONS] VM

Generate console URL or screen capture for VM.

One of VMRC, VMware Player, VMware Fusion or VMware Workstation must be installed to
open VMRC console URLs.

Options:
  -capture=                      Capture console screen shot to file
  -cert=                         Certificate [GOVC_CERTIFICATE]
  -dc=ha-datacenter              Datacenter [GOVC_DATACENTER]
  -debug=false                   Store debug logs [GOVC_DEBUG]
  -dump=false                    Enable Go output
  -h5=false                      Generate HTML5 UI console link
  -json=false                    Enable JSON output
  -k=true                        Skip verification of server certificate [GOVC_INSECURE]
  -key=                          Private key [GOVC_PRIVATE_KEY]
  -persist-session=true          Persist session to disk [GOVC_PERSIST_SESSION]
  -tls-ca-certs=                 TLS CA certificates file [GOVC_TLS_CA_CERTS]
  -tls-known-hosts=              TLS known hosts file [GOVC_TLS_KNOWN_HOSTS]
  -u=https://@192.168.1.8/sdk  ESX or vCenter URL [GOVC_URL]
  -vim-namespace=vim25           Vim namespace [GOVC_VIM_NAMESPACE]
  -vim-version=6.7               Vim version [GOVC_VIM_VERSION]
  -vm=                           Virtual machine [GOVC_VM]
  -vm.dns=                       Find VM by FQDN
  -vm.ip=                        Find VM by IP address
  -vm.ipath=                     Find VM by inventory path
  -vm.path=                      Find VM by path to .vmx file
  -vm.uuid=                      Find VM by UUID
```



```bash
# MacOSX VMR
open $(govc vm.console my-vm)

# MacOSX H5
open $(govc vm.console -h5 my-vm) 

# Linux VMRC
xdg-open $(govc vm.console my-vm)  

# Linux H5
xdg-open $(govc vm.console -h5 my-vm)      
```



```bash
govc vm.console my-vm
govc vm.console -capture screen.png my-vm  # screen capture
govc vm.console -capture - my-vm | display # screen capture to stdout

```

# 六、获取VM信息

## 1、获取VM的IP地址

```bash
Usage: govc vm.ip [OPTIONS] VM...

List IPs for VM.

By default the vm.ip command depends on vmware-tools to report the 'guest.ipAddress' field and will
wait until it has done so.  This value can also be obtained using:

  govc vm.info -json $vm | jq -r .VirtualMachines[].Guest.IpAddress

When given the '-a' flag, only IP addresses for which there is a corresponding virtual nic are listed.
If there are multiple nics, the listed addresses will be comma delimited.  The '-a' flag depends on
vmware-tools to report the 'guest.net' field and will wait until it has done so for all nics.
Note that this list includes IPv6 addresses if any, use '-v4' to filter them out.  IP addresses reported
by tools for which there is no virtual nic are not included, for example that of the 'docker0' interface.

These values can also be obtained using:

  govc vm.info -json $vm | jq -r .VirtualMachines[].Guest.Net[].IpConfig.IpAddress[].IpAddress

When given the '-n' flag, filters '-a' behavior to the nic specified by MAC address or device name.

The 'esxcli' flag does not require vmware-tools to be installed, but does require the ESX host to
have the /Net/GuestIPHack setting enabled.

The 'wait' flag default to 1hr (original default was infinite).  If a VM does not obtain an IP within
the wait time, the command will still exit with status 0.

Examples:
  govc vm.ip $vm
  govc vm.ip -wait 5m $vm
  govc vm.ip -a -v4 $vm
  govc vm.ip -n 00:0c:29:57:7b:c3 $vm
  govc vm.ip -n ethernet-0 $vm
  govc host.esxcli system settings advanced set -o /Net/GuestIPHack -i 1
  govc vm.ip -esxcli $vm

Options:
  -a=false               Wait for an IP address on all NICs
  -esxcli=false          Use esxcli instead of guest tools
  -n=                    Wait for IP address on NIC, specified by device name or MAC
  -v4=false              Only report IPv4 addresses
  -wait=1h0m0s           Wait time for the VM obtain an IP address
```

# 七、导出VM为OVF模板

## 1、命令

```bash
Usage: govc export.ovf [OPTIONS] DIR

Export V

Options:
  -f=false               Overwrite existing
  -i=false               Include image files (*.{iso,img})
  -prefix=true           Prepend target name to image filenames if missing
  -name=                 Specifies target name (defaults to source name)
  -sha=0                 Generate manifest using SHA 1, 256, 512 or 0 to skip
  -vm=                   Virtual machine [GOVC_VM]
```

## 2、示例

```bash
govc export.ovf -vm vmname .
```

# 八、ESXI主机的存储磁盘管理

## 1、查看ESXI主机存储

```bash
Usage: govc datastore.info [OPTIONS] [PATH]...

Display info for Datastores.

Examples:
  govc datastore.info
  govc datastore.info vsanDatastore
  # info on Datastores shared between cluster hosts:
  govc object.collect -s -d " " /dc1/host/k8s-cluster host | xargs govc datastore.info -H
  # info on Datastores shared between VM hosts:
  govc ls /dc1/vm/*k8s* | xargs -n1 -I% govc object.collect -s % summary.runtime.host | xargs govc datastore.info -H

Options:
  -H=false           Display info for Datastores shared between hosts
  
Name:        datastore1
  Path:      /ha-datacenter/datastore/datastore1
  Type:      VMFS
  URL:       /vmfs/volumes/5ad72ff6-920c8d20-3ee7-848f69e3e398
  Capacity:  829.0 GB
  Free:      503.9 GB
```

## 2、查看已有的数据存储

```bash
Usage: govc datastore.ls [OPTIONS] [FILE]...

Options:
  -R=false               List subdirectories recursively
  -a=false               Do not ignore entries starting with .
  -ds=                   Datastore [GOVC_DATASTORE]
  -l=false               Long listing format
  -p=false               Append / indicator to directories
```

## 3、创建VMDK磁盘

```bash
Usage: govc datastore.disk.create [OPTIONS] VMDK

Create VMDK on DS.

Examples:
  govc datastore.mkdir disks
  govc datastore.disk.create -size 24G disks/disk1.vmdk
  govc datastore.disk.create disks/parent.vmdk disk/child.vmdk

Options:
  -a=lsiLogic            Disk adapter
  -d=thin                Disk format
  -ds=                   Datastore [GOVC_DATASTORE]
  -f=false               Force
  -size=10.0GB           Size of new disk
  -uuid=                 Disk UUID
```

## 4、查看VMDK磁盘

```bash
Usage: govc datastore.disk.info [OPTIONS] VMDK

Query VMDK info on DS.

Examples:
  govc datastore.disk.info disks/disk1.vmdk

Options:
  -c=false               Chain format
  -d=false               Include datastore in output
  -ds=                   Datastore [GOVC_DATASTORE]
  -p=true                Include parents
  -uuid=false            Include disk UUID

Name:      disks/disk1.vmdk
  Type:    thin
  Parent:
```

## 5、下载VMDK磁盘到本地

```bash
Usage: govc datastore.download [OPTIONS] SOURCE DEST

Copy SOURCE from DS to DEST on the local system.

If DEST name is "-", source is written to stdout.

Examples:
  govc datastore.download vm-name/vmware.log ./local.log
  govc datastore.download vm-name/vmware.log - | grep -i error

Options:
  -ds=                   Datastore [GOVC_DATASTORE]
  -host=                 Host system [GOVC_HOST]
```

# 九、ESXI主机管理

## 1、查看ESXI主机信息

```bash
Usage: govc host.info [OPTIONS]

Options:
  -host=                 Host system [GOVC_HOST]
  
# 示例
govc host.info -host.ip=192.168.1.8

Name:              localhost.localdomain
  Path:            /ha-datacenter/host/localhost./localhost.localdomain
  Manufacturer:    Dell Inc.
  Logical CPUs:    24 CPUs @ 3058MHz
  Processor type:  Intel(R) Xeon(R) CPU           X5675  @ 3.07GHz
  CPU usage:       104 MHz (0.1%)
  Memory:          98291MB
  Memory usage:    4583 MB (4.7%)
  Boot time:       2020-04-01 01:21:16.499148 +0000 UTC
  State:           connected
```

## 2、查看ESXI主机日志文件

```bash
Usage: govc logs.ls [OPTIONS]

List diagnostic log keys.

Examples:
  govc logs.ls
  govc logs.ls -host host-a

Options:
  -host=                 Host system [GOVC_HOST]
```

```bash
hostd     /var/log/hostd.log
vmkernel  /var/log/vmkernel.log
vpxa      /var/log/vpxa.log
```

## 3、实时查看ESXI主机日志

```bash
Usage: govc logs [OPTIONS]

View VPX and ESX logs.

The '-log' option defaults to "hostd" when connected directly to a host or
when connected to VirtualCenter and a '-host' option is given.  Otherwise,
the '-log' option defaults to "vpxd:vpxd.log".  The '-host' option is ignored
when connected directly to a host.  See 'govc logs.ls' for other '-log' options.

Examples:
  govc logs -n 1000 -f
  govc logs -host esx1
  govc logs -host esx1 -log vmkernel

Options:
  -f=false               Follow log file changes
  -host=                 Host system [GOVC_HOST]
  -log=                  Log file key
  -n=25                  Output the last N log lines
```

## 4、下载ESXI日志

```bash
Usage: govc logs.download [OPTIONS] [PATH]...

Generate diagnostic bundles.

A diagnostic bundle includes log files and other configuration information.

Use PATH to include a specific set of hosts to include.

Examples:
  govc logs.download
  govc logs.download host-a host-b

Options:
  -default=false         Specifies if the bundle should include the default server
```

## 5、查看EXSI的资产

```bash
Usage: govc ls [OPTIONS] [PATH]...

List inventory items.

Examples:
  govc ls -l '*'
  govc ls -t ClusterComputeResource host
  govc ls -t Datastore host/ClusterA/* | grep -v local | xargs -n1 basename | sort | uniq

Options:
  -L=false               Follow managed object references
  -i=false               Print the managed object reference
  -l=false               Long listing format
  -t=                    Object type
```

```
/ha-datacenter/vm/OCP4.3-Tools (VirtualMachine)
/ha-datacenter/vm/test1 (VirtualMachine)
/ha-datacenter/vm/node1.cloudera.curiouser.com (VirtualMachine)
/ha-datacenter/vm/node2.cloudera.curiouser.com (VirtualMachine)
/ha-datacenter/vm/node3.cloudera.curiouser.com (VirtualMachine)
/ha-datacenter/vm/OKD3.11-Allinone (VirtualMachine)
/ha-datacenter/vm/Vsphere vCenter 6.0 (VirtualMachine)
/ha-datacenter/vm/PXE Kickstart (VirtualMachine)
/ha-datacenter/vm/Allinone K8S114 (VirtualMachine)
/ha-datacenter/network/VM Network (Network)
/ha-datacenter/host/localhost. (ComputeResource)
/ha-datacenter/datastore/datastore1 (Datastore)
```

## 6、ESXI主机电源管理

关机

```bash
Usage: govc host.shutdown [OPTIONS] HOST...

Shutdown HOST.

Options:
  -f=false               Force shutdown when host is not in maintenance mode
  -host=                 Host system [GOVC_HOST]
  -r=false               Reboot host
  
示例：
govc host.shutdown -f -host.ip ESXI_IP
```

## 7、维护状态的管理

### ①进入维护状态

```bash
Usage: govc host.maintenance.enter [OPTIONS] HOST...

Put HOST in maintenance mode.

While this task is running and when the host is in maintenance mode,
no VMs can be powered on and no provisioning operations can be performed on the host.

Options:
  -evacuate=false        Evacuate powered off VMs
  -host=                 Host system [GOVC_HOST]
  -timeout=0             Timeout
  
# 示例：
govc host.maintenance.enter -host.ip=192.168.1.8
```

### ②退出维护状态

```bash
Usage: govc host.maintenance.exit [OPTIONS] HOST...

Take HOST out of maintenance mode.

This blocks if any concurrent running maintenance-only host configurations operations are being performed.
For example, if VMFS volumes are being upgraded.

The 'timeout' flag is the number of seconds to wait for the exit maintenance mode to succeed.
If the timeout is less than or equal to zero, there is no timeout.

Options:
  -host=                 Host system [GOVC_HOST]
  -timeout=0             Timeout

# 示例：
govc host.maintenance.exit -host.ip=192.168.1.8
```

## 8、查看ESXI主机开启的服务

```bash
Usage: govc host.service.ls [OPTIONS]

List HOST services.

Options:
  -host=                 Host system [GOVC_HOST]

# 示例
govc host.service.ls -host.ip 192.168.1.8

Key             Policy  Status   Label
DCUI            on      Running  Direct Console UI
TSM             on      Running  ESXi Shell
TSM-SSH         on      Running  SSH
lbtd            on      Running  Load-Based Teaming Daemon
lwsmd           off     Stopped  Active Directory Service
ntpd            on      Stopped  NTP Daemon
pcscd           off     Stopped  PC/SC Smart Card Daemon
sfcbd-watchdog  on      Stopped  CIM Server
snmpd           on      Stopped  SNMP Server
vmsyslogd       on      Running  Syslog Server
vprobed         off     Stopped  VProbe Daemon
vpxa            on      Running  VMware vCenter Agent
xorg            on      Stopped  X.Org Server
```

## 9、ESXI主机上服务的管理

```bash
Usage: govc host.service [OPTIONS] ACTION ID

Apply host service ACTION to service ID.

Where ACTION is one of: start, stop, restart, status, enable, disable

Examples:
  govc host.service enable TSM-SSH
  govc host.service start TSM-SSH

Options:
  -host=                 Host system [GOVC_HOST]
 
# 示例
govc host.service -host.ip 192.168.1.8 status TSM
```

## 10、ESXI主机硬盘信息

```bash
Usage: govc host.storage.info [OPTIONS]

Show HOST storage system information.

Examples:
  govc find / -type h | xargs -n1 govc host.storage.info -unclaimed -host

Options:
  -host=                 Host system [GOVC_HOST]
  -refresh=false         Refresh the storage system provider
  -rescan=false          Rescan all host bus adapters
  -rescan-vmfs=false     Rescan for new VMFSs
  -t=lun                 Type (hba,lun)
  -unclaimed=false       Only show disks that can be used as new VMFS datastores
  
# 示例：
govc host.storage.info -host.ip 192.168.1.8

Name                                     Type       Capacity  Model
/vmfs/devices/cdrom/mpx.vmhba1:C0:T0:L0  cdrom      -         DVD-ROM DS-8D3SH
/vmfs/devices/disks/naa.6d4ae520b6ed2    disk       666.6GB   PERC 6/i         (local)
/vmfs/devices/genscsi/t10.DP_000         enclosure  -         BACKPLANE
```

## 11、ESXI主机用户会话的管理

### ①查看会话

```bash
Usage: govc session.ls [OPTIONS]

List active sessions.

Examples:
  govc session.ls
  govc session.ls -json | jq -r .CurrentSession.Key

# 示例
Key                                   Name     Time              Idle    Host         Agent
520ea520  vpxuser  2020-04-01 01:22  1s      127.0.0.1    VMware-client/5.1.0
525b5d4f  root     2020-04-01 12:18  26m14s  192.168.1.7  VMware-client/6.5.0
52a68e2e  dcui     2020-04-01 01:25  4m43s   127.0.0.1    VMware-client/5.1.0
52b127da  root     2020-04-01 01:22  3m42s   127.0.0.1
52c429cc  root     2020-04-01 12:22  21m44s  192.168.1.7  VMware-client/6.5.0
52d9125d  root     2020-04-01 10:42    .     192.168.1.7  govc/0.22.1
```

### ②删除会话

```bash
Usage: govc session.rm [OPTIONS] KEY...

Remove active sessions.

Examples:
  govc session.ls | grep root
  govc session.rm 5279e245-e6f1-4533-4455-eb94353b213a
```

### ③退出当前会话

```bash
Usage: govc session.logout [OPTIONS]

Logout the current session.

The session.logout command can be used to end the current persisted session.
The session.rm command can be used to remove sessions other than the current session.

Examples:
  govc session.logout
```

## 12、ESXI主机网络的管理

### ①查看虚拟网卡信息

```bash
Usage: govc host.vnic.info [OPTIONS]

Options:
  -host=                 Host system [GOVC_HOST]

# 示例：
govc host.vnic.info -host.ip 192.168.1.8

Device:            vmk0
Network label:     Management Network
Switch:            vSwitch0
IP address:        192.168.1.8
TCP/IP stack:      defaultTcpipStack
Enabled services:  management
```

### ②查看虚拟交换机信息

```bash
Usage: govc host.vswitch.info [OPTIONS]

Options:
  -host=                 Host system [GOVC_HOST]
  
# 示例：
govc host.vswitch.info -host.ip=192.168.1.8

Name:                    vSwitch0
Portgroup:               VM Network, Management Network
Pnic:                    vmnic0
MTU:                     1500
Ports:                   4352
Ports Available:         4344
Allow promiscuous mode:  No
Allow forged transmits:  Yes
Allow MAC changes:       Yes
```

### ③创建虚拟交换机

```bash
Usage: govc host.vswitch.add [OPTIONS] NAME

Options:
  -host=                 Host system [GOVC_HOST]
  -mtu=0                 MTU
  -nic=                  Bridge nic device
  -ports=128             Number of ports
# 示例  
govc host.vswitch.add -host.ip 192.168.1.8 test

Name:                    test
Portgroup:
Pnic:
MTU:                     1500
Ports:                   4352
Ports Available:         4343
Allow promiscuous mode:  No
Allow forged transmits:  Yes
Allow MAC changes:       Yes
```

### ④删除虚拟交换机

```bash
Usage: govc host.vswitch.remove [OPTIONS] NAME

Options:
  -host=                 Host system [GOVC_HOST]
```

## 13、ESXI用户角色的管理

### ①查看所有用户角色

```bash
Usage: govc role.ls [OPTIONS] [NAME]

List authorization roles.

If NAME is provided, list privileges for the role.

Examples:
  govc role.ls
  govc role.ls Admin

Options:
  -i=false               Use moref instead of inventory path


NoAccess   Used for restricting granted access
Anonymous  Not logged-in user (cannot be granted)
View       Visibility access (cannot be granted)
ReadOnly   See details of objects, but not make changes
Admin      Full access rights
```

### ②创建用户角色

```bash
Usage: govc role.create [OPTIONS] NAME [PRIVILEGE]...

Create authorization role.

Optionally populate the role with the given PRIVILEGE(s).

Examples:
  govc role.create MyRole
  govc role.create NoDC $(govc role.ls Admin | grep -v Datacenter.)

Options:
  -i=false               Use moref instead of inventory path
```

### ③删除用户角色

```bash
Usage: govc role.remove [OPTIONS] NAME

Remove authorization role.

Examples:
  govc role.remove MyRole
  govc role.remove MyRole -force

Options:
  -force=false           Force removal if role is in use
  -i=false               Use moref instead of inventory path
```

### ④更新用户角色

```bash
Usage: govc role.update [OPTIONS] NAME [PRIVILEGE]...

Update authorization role.

Set, Add or Remove role PRIVILEGE(s).

Examples:
  govc role.update MyRole $(govc role.ls Admin | grep VirtualMachine.)
  govc role.update -r MyRole $(govc role.ls Admin | grep VirtualMachine.GuestOperations.)
  govc role.update -a MyRole $(govc role.ls Admin | grep Datastore.)
  govc role.update -name RockNRole MyRole

Options:
  -a=false               Add given PRIVILEGE(s)
  -i=false               Use moref instead of inventory path
  -name=                 Change role name
  -r=false               Remove given PRIVILEGE(s)
```

### ⑤查看用户角色的授予信息

```bash
Usage: govc role.usage [OPTIONS] NAME...

List usage for role NAME.

Examples:
  govc role.usage
  govc role.usage Admin

Options:
  -i=false               Use moref instead of inventory path
```

## 14、Datastore操作

### ①查看远程ESXI的Datastore详细信息

```bash
Usage: govc datastore.info [OPTIONS] [PATH]...

Examples:
  govc datastore.info
  govc datastore.info vsanDatastore
  # info on Datastores shared between cluster hosts:
  govc object.collect -s -d " " /dc1/host/k8s-cluster host | xargs govc datastore.info -H
  # info on Datastores shared between VM hosts:
  govc ls /dc1/vm/*k8s* | xargs -n1 -I% govc object.collect -s % summary.runtime.host | xargs govc datastore.info -H

Options:
  -H=false               Display info for Datastores shared between hosts
# 示例： govc datastore.info
Name:        datastore1
  Path:      /ha-datacenter/datastore/datastore1
  Type:      VMFS
  URL:       /vmfs/volumes/5ad72ff6-920c8d20-3ee7-848f69e3e398
  Capacity:  829.0 GB
  Free:      506.1 GB
```

### ②查看远程ESXI的Datastore目录

```bash
Usage: govc datastore.ls [OPTIONS] [FILE]...

Options:
  -R=false               List subdirectories recursively
  -a=false               Do not ignore entries starting with .
  -ds=                   Datastore [GOVC_DATASTORE]
  -l=false               Long listing format
  -p=false               Append / indicator to directories
```

### ③上传本地文件到远程ESXI主机Datastore

```bash
Usage: govc datastore.upload [OPTIONS] SOURCE DEST

If SOURCE name is "-", read source from stdin.

Examples:
  govc datastore.upload -ds datastore1 ./config.iso vm-name/config.iso
  genisoimage ... | govc datastore.upload -ds datastore1 - vm-name/config.iso

Options:
  -ds=                   Datastore [GOVC_DATASTORE]
```

### ④下载远程ESXI主机Datastore中的文件到本地

```bash
Usage: govc datastore.download [OPTIONS] SOURCE DEST

Copy SOURCE from DS to DEST on the local system.

If DEST name is "-", source is written to stdout.

Examples:
  govc datastore.download vm-name/vmware.log ./local.log
  govc datastore.download vm-name/vmware.log - | grep -i error

Options:
  -ds=                   Datastore [GOVC_DATASTORE]
  -host=                 Host system [GOVC_HOST]
```

### ⑤删除远程ESXI主机Datastore中的文件

```bash
Usage: govc datastore.rm [OPTIONS] FILE

Remove FILE from DATASTORE.

Examples:
  govc datastore.rm vm/vmware.log
  govc datastore.rm vm
  govc datastore.rm -f images/base.vmdk

Options:
  -ds=                   Datastore [GOVC_DATASTORE]
  -f=false               Force; ignore nonexistent files and arguments
  -namespace=false       Path is uuid of namespace on vsan datastore
  -t=true                Use file type to choose disk or file manager
```

### ⑥移动远程ESXI主机Datastore中的文件

```bash
Usage: govc datastore.mv [OPTIONS] SRC DST

Move SRC to DST on DATASTORE.

Examples:
  govc datastore.mv foo/foo.vmx foo/foo.vmx.old
  govc datastore.mv -f my.vmx foo/foo.vmx

Options:
  -dc-target=            Datacenter destination (defaults to -dc)
  -ds=                   Datastore [GOVC_DATASTORE]
  -ds-target=            Datastore destination (defaults to -ds)
  -f=false               If true, overwrite any identically named file at the destination
  -t=true                Use file type to choose disk or file manager
```

### ⑦在远程ESXI主机Datastore创建文件路径

```bash
Usage: govc datastore.mkdir [OPTIONS] DIRECTORY

Options:
  -ds=                   Datastore [GOVC_DATASTORE]
  -namespace=false       Return uuid of namespace created on vsan datastore
  -p=false               Create intermediate directories as needed
```

# 十、Windows虚拟机的创建

①创建空的虚拟机

```bash
gnvm -c 8 -m 20480 -disk=100G  -host.ip=192.168.1.103 -on=false -g windows9_64Guest -disk.controller=lsilogic-sas windows 
```

注意：

- 磁盘控制器要选择：`lsilogic-sas`，不然在安装过程显示找不到系统磁盘
- SCSI 磁盘控制器类型：`lsilogic|buslogic|pvscsi|lsilogic-sas`
- 虚拟机OS类型要加上。类型匹配标识见：https://wiki.abiquo.com/display/doc/Guest+operating+system+definition+for+VMware+vSphere+5+and+6

②添加空的CDROM光驱设备(VM需在关机状态)

```bash
govc device.cdrom.add -vm windows
```

③将OS镜像ISO文件加到CDROM设备中

```bash
govc device.cdrom.insert -vm windows -device cdrom-3000 Images/Windows\ 10\ Desktop/Windows10_All_x86_64.iso
```

④设置BIOS启动顺序

```bash
govc device.boot -vm windows -delay 100 -order cdrom,ethernet,disk
```

⑤VM开机

```bash
govc vm.power -on -M windows
```

开机以后就会显示从CD光驱的ISO进行引导启动啦

⑥登录控制台

```bash
open $(govc vm.console windows)
```

⑦(可选)快捷命令

```bash
gnvm -c 8 -m 20480 -disk=100G  -host.ip=192.168.1.103 -on=false -g windows9_64Guest -disk.controller=lsilogic-sas windows \
&& govc device.cdrom.add -vm windows \
&& govc device.cdrom.insert -vm windows -device cdrom-3000 Images/Windows\ 10\ Desktop/Windows10_All_x86_64.iso \
&& govc device.boot -vm windows -delay 100 -order cdrom,ethernet,disk \
&& govc vm.power -on -M windows \
&& gco windows
```









# 附录: 常用命令别名

```bash
# 查看所有VM
alias gsp='govc find . -type m'

# 查看所有开机的VM
alias gspo='govc find . -type m -runtime.powerState poweredOn'

# 关闭VM电源
alias gvpof='govc vm.power -off -M $@'

# 开启VM电源
alias gvpo='govc vm.power -on -M $@'

# 获取VM的IP地址
alias ggip='govc vm.ip $@'

# 获取VM的详细信息
alias ggvi=‘govc vm.info $@’

# 销毁VM
alias gdv='govc vm.destroy $@'

# 断开ESXI电源
alias gpof—esxi='govc host.shutdown -f -host.ip 192.168.1.8'

# 进入维护状态
alias gim='govc host.maintenance.enter -host.ip=192.168.1.8'

# 退出维护状态
alias gom='govc host.maintenance.exit -host.ip=192.168.1.8'

# 创建VM
alias gnvm='govc vm.create -host.ip=192.168.1.8 $@'

# 连接VM Console
alias gco='{ IFS= read -r vm && open $(g7b6cc489180160eeb4a7a39ebb25d524e04c9f5e7b6cc489180160eeb4a7a39ebb25d524e04c9f5eovc vm.console $vm); } <<<'
```



