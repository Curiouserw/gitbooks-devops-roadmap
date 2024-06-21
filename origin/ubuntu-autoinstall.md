# Ubuntu/Debian无人值守 Autoinstall

# 一、简介

**preseeds** 是基于 **debian-installer** （又名 **d-i** ）自动化安装程序的方法。

Ubuntu新 **autoinstall** 方式的自动安装主要在以下方面不同于 **preseeds** ：

- 格式完全不同（ **cloud-init config** 格式，通常是 **yaml** 格式，vs **debconf-set-selections** 格式）

- - 当一个问题的答案不存在于 **preseeds** 中时， **d-i** 停止执行并要求用户输入。 **autoinstalls** 自动安装不是这样的：默认情况下，如果有任何自动安装配置，安装程序会为任何未回答的问题采用默认设置（如没有默认设置则失败）。

    用户可以将配置中的特定部分指定为“交互式”，这意味着安装程序仍会停止并询问这些部分。

**Ubuntu 20.04 Server** 与 **23.04 Desktop** 之后的版本.    建议使用 Autoinstall 进行无人值守安装





# 二、Autoinstall配置文件生效的方式

- 通过 clound-int 提供 autoinstall 相关的配置（推荐）

  这种方式直接作用于安装程序，而无需修改安装介质。不过配置文件头得为：`#cloud-config`

  ```bash
  #cloud-config
  autoinstall:
      version: 1
      ...
  ```

- 在安装iso介质中提供

  > autoinstall.yaml

  ```yaml
  version: 1
  ....
  ```

  24.04 (Noble)之后可以使用 cloud-config 的格式

  ```yaml
  autoinstall:
      version: 1
      ....
  ```

  安装程序Subiquity默认搜索配置文件`autoinstall.yaml`的路径

  - 安装iso介质的根目录，或在
  - 安装系统的根文件系统上。
  - 如果放在其他路径下，也可以通过设置内核参数`subiquity.autoinstallpath=path/to/autoinstall.yaml`来指定。其中路径是相对于安装系统的根目录的。

  

配置文件执行的优先顺序

- 内核命令行参数 `subiquity.autoinstallpath`指定的配置文件
- 安装系统的根文件系统
- cloud-config
- 安装 ISO介质的根路径







# 三、内核参数

```bash
DEFAULT vesamenu.c32
#DEFAULT menu.c32
prompt 0
timeout 30
display boot.msg
menu background splash.jpg
menu title #### Curiouser PXE Boot Menus ####

label 'Ubuntu 20.04.6 LTS'
  menu label ^1> Install Ubuntu 20.04.6 LTS (Focal Fossa)
  menu default
  kernel Ubuntu/20.04.6/vmlinuz
  initrd Ubuntu/20.04.6/initrd
  # append 追加到内核的参数
  append debug root=/dev/ram0 ramdisk_size=2000000 ipv6.disable=1 fsck.mode=skip ip=dhcp url=http://192.168.1.1:8089/Ubuntu/20.04.6/ubuntu-20.04.6-live-server-amd64.iso autoinstall cloud-config-url=http://192.168.1.1:8089/Ubuntu/20.04.6/user-data
```

- autoinstall：默认安装程序中有个安全设置，防止带有 autoinstall.yaml 文件的USB驱动器擦除系统。参考：https://canonical-subiquity.readthedocs-hosted.com/en/latest/explanation/zero-touch-autoinstall.html。要绕过此提示，参数 autoinstall 必须在内核命令行设置。
- 

# 四、Autoinstall配置详解

## 1、文件内容格式

```bash
#cloud-config
autoinstall:
  version: 1
  配置项: ...
    参数: ...
  ......
```

## 2、#cloud-config：文件头部

头部标识文件类型为 `cloud-config`，这是云配置文件格式的标准。

```
#cloud-config
```

## 3、version

配置文件版本字段。目前，该值必须为 1。

```
version: 1
```

## 4、identity：身份设置 

用于设置系统的主机名、用户名和密码。密码应该使用加密形式。

```
identity:
  hostname: ubuntu-server
  password: "$6$rounds=4096$z...$..."
  username: ubuntu
```

密码可以使用以下方式生成

- 使用 `openssl passwd` 生成基于 SHA-512的密码

  ```bash
  $ openssl passwd -6 
  #Password: 
  Verifying - Password: 
  $6$rounds=5000$WiA0EmOC.kFC$MOd3X96skVeyDCJb1Hg... (省略)
  ```

- 使用whois包里的 mkpasswd命令生成


## 5、ssh：SSH设置

```
ssh:
  install-server: true
  allow-pw: true
```

`install-server` 指定是否安装 SSH 服务器，`allow-pw` 允许使用密码登录。

## 6、locale：区域设置

用于设置系统区域和语言

```
locale: en_US.UTF-8
```

## 6、keyboard：键盘布局 

用于设置键盘布局和变体

```yaml
keyboard:
  layout: us
  variant: ""
  toggle: null

# layout: string，默认 us
# variant: string，默认为空
# toggle: string或null，默认为null
```

## 7、storage：存储配置 

用于配置存储，支持`lvm`, `direct` and `zfs`。建议使用 LVM（逻辑卷管理）自动分区。

```

  storage:
    layout:
      name: lvm
      match:
        serial: CT*
  storage:
    layout:
      name: direct
      match:
        ssd: true
```

## 8、network：网络配置

用于配置网络，这里设置 `enp0s3` 接口使用 DHCP 自动获取 IP 地址。

```
network:
  network:
    version: 2
    ethernets:
      enp0s3:
        dhcp4: true
```

## 9、user-data：用户数据

用于设置用户数据，禁用或启用 root 用户，设置密码不过期

```
user-data:
  disable_root: false
  chpasswd:
    expire: false
```

## 10、early-commands

```
early-commands:
    - ping -c1 198.162.1.1
```

## 11、late-commands：后期命令 

用于配置后期的命令，安装完成后执行，这里记录安装成功的日志。

```yaml
late-commands:
  - echo "Installation complete" > /target/var/log/installer-success.log
```

## 12、apt：APT 配置

用于配置 APT 源

```
apt:
  primary:
    - arches: [default]
      uri: http://archive.ubuntu.com/ubuntu
```

## 13、error-commands

```yaml
error-commands:
  - tar c /var/log/installer | nc 192.168.0.1 1000
```



# 五、其他信息

## 1、安装过程的日志

- 自动化安装日志输出 tty1 窗口，安装过程可开起 tty2~tty6 窗口
- /var/log/curtin-install.log：下载安装日志

# 六、样例文件

## 1、Ubuntu 22.04.6

```yaml
#cloud-config
autoinstall:
  version: 1
  locale: en_US.UTF-8
  timezone: Asia/Shanghai
  keyboard:
    layout: us
  network:
    network:
      version: 2
      ethernets:
        ens32:
          dhcp4: true
  storage:
    layout:
      name: lvm
      reset-partition: true
  identity:
    realname: 'Ubuntu User'
    username: ubuntu
    password: '$6$ph/TO........密码............'
    hostname: ubuntu
  packages:
    - openssh-server
    - git
  apt:
    primary:
      - arches: [default]
        uri: https://mirrors.aliyun.com/ubuntu
  shutdown: reboot
```

# 参考

- https://ubuntu.com/server/docs/how-to-netboot-the-server-installer-on-amd64
- https://discourse.ubuntu.com/t/netbooting-the-live-server-installer/14510
- https://askubuntu.com/questions/1292032/how-can-current-ubuntu-versions-be-installed-via-pxe-network-boot-and-an-automat
- https://www.molnar-peter.hu/en/ubuntu-jammy-netinstall-pxe.html
- https://louwrentius.com/understanding-the-ubuntu-2004-lts-server-autoinstaller.html
- https://canonical-subiquity.readthedocs-hosted.com/en/latest/reference/autoinstall-reference.html#version
- https://hmli.ustc.edu.cn/doc/linux/ubuntu-autoinstall/