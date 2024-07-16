# CoreOS无人值守自动化配置Ignition详解

# 一、简介

Ignition 是一个用于配置 Fedora CoreOS 和其他容器操作系统的配置系统。它在系统首次引导时运行，并对系统进行初始化配置。Ignition 通过读取配置文件来创建文件系统、设置网络、配置用户和密码、下载文件等操作。
由于 CoreOS 设计用于无状态运行，root 文件系统通常是只读的，运行时配置和数据存储在临时文件系统或持久存储中。

官方文档：https://coreos.github.io/ignition/

# 二、PXE配置

```bash
LABEL 'CoreOS 40.20240616.3.0 x86_64'
    menu label ^1> Install CoreOS 40.20240616.3.0 x86_64
    kernel CoreOS/40.20240616.3.0/fedora-coreos-40.20240616.3.0-live-kernel-x86_64
    initrd CoreOS/40.20240616.3.0/fedora-coreos-40.20240616.3.0-live-initramfs.x86_64.img,CoreOS/40.20240616.3.0/fedora-coreos-40.20240616.3.0-live-rootfs.x86_64.img
    APPEND coreos.inst.install_dev=/dev/sda coreos.inst.ignition_url=http://192.168.1.1:8089/CoreOS/40.20240616.3.0/config.ign ignition.firstboot ignition.platform.id=vmware
```

- **kernel文件：**kernel` 文件是操作系统的核心部分，负责管理系统资源（如 CPU、内存、I/O 设备）并提供底层服务（如进程管理、内存管理、文件系统、网络）。在系统启动时，BIOS 或 UEFI 固件会将控制权交给引导加载程序（如 GRUB），引导加载程序再将内核加载到内存中并启动内核。

- **initramfs文件：**临时根文件系统，包含用于启动系统的必要文件和驱动程序。它通常包含一个初始的 `init` 脚本，用于进行早期系统初始化任务，如挂载实际的根文件系统。内核启动后，会将 `initramfs` 文件解压缩到内存中，并运行其中的 `init` 脚本。`init` 脚本负责加载所需的驱动程序、初始化设备和挂载实际的根文件系统。

- **rootfs文件：**（root filesystem）文件是实际的根文件系统，包含操作系统的所有必要文件、库和应用程序。它是系统在启动完成后运行的主要文件系统。

  在 `initramfs` 完成其初始化任务后，控制权将移交给 `rootfs` 中的初始化系统（如 `systemd` 或 `init`）。系统将切换到实际的根文件系统，并继续启动和配置操作系统，直到进入用户空间。

# 三、Ignition参数

Ignition 配置文件是 JSON 格式的，包含以下部分：

- `ignition`：包含 Ignition 版本信息

- `storage`：用于定义文件系统、分区、文件、目录和链接
- `systemd`：用于定义和管理 systemd 单元文件
- `networkd`：用于配置网络
- `passwd`：用于配置用户和组

参数类似于 KickStart，具体参考：https://coreos.github.io/ignition/configuration-v3_4/

# 四、butane转换校验工具

ignition配置文件为 json 格式的。跟 kickstart 的 yaml格式的不太一样。但是可以使用工具`butane`将yaml 格式的转换为json 格式的。在转换过程中还可以校验语法。

下载地址：https://github.com/coreos/butane/releases

转换命令：`butane --pretty --strict example.yaml > config.ign`

# 五、示例

## 1、示例配置功能

- 设置市区
- 设置代理
- 设置键盘布局
- 设置网络系统配置
- 设置主机名
- 安装 Docker
- 创建用户core
- 关闭selinux
- 设置 SSHD
- 安装后通知

```yaml
variant: fcos
version: 1.5.0
storage:
  links:
    - path: /etc/localtime
      overwrite: true
      target: /usr/share/zoneinfo/Asia/Shanghai
  files:
    - path: /etc/notify.sh
      mode: 760
      contents:
        inline: |
          $!/bin/bash
          bstarttime=$(date +'%Y年%m月%d日%H:%M:%S') mip=$(ip a |grep ens -A 10 |grep -w inet | awk '{print $2}' | awk -F'/' '{print $1}') cpus=$(lscpu | grep "^CPU(s):" |awk '{print $2}') mem_total=$(free -h | grep "^Mem:" | awk '{print $2}') disk_name=$(lsblk -d -o NAME,SIZE | grep sd |awk '{print $1}') disk_total=$(lsblk -d -o NAME,SIZE | grep sd |awk '{print $2}') 
          payload='{"msgtype": "markdown","markdown": {"title": "Comprehens", "text": "### CoreOS虚拟机操作系统已安装完成:\n#### **安装时间**: '$bstarttime'\n####   **主机IP**: '$mip'\n#### **硬件信息**: '$cpus核' '$mem_total内存'  **系统盘**'$disk_name' '$disk_total'\n#### **发行版本**: '`cat /etc/redhat-release`'\n#### **内核版本**: '$(uname -r)'\n####   **主机名**: '$HOSTNAME'"}}'
          curl -sSk -XPOST -H "Content-Type: application/json" -d "$payload" https://oapi.dingtalk.com/robot/send?access_token=...
          curl --resolve bark.test.com:443:192.168.1.1 "https://bark.test.com/....../Rocky已于${bstarttime}安装完成,IP地址${mip}"
    - path: /etc/example-proxy.env
      mode: 0644
      contents:
        inline: |
          https_proxy="http://192.168.1.1:11100"
          all_proxy="http://192.168.1.1:11100"
          http_proxy="http://192.168.1.1:11100"
          HTTP_PROXY="http://192.168.1.1:11100"
          HTTPS_PROXY="http://192.168.1.1:11100"
          no_proxy="192.168.1.0/24,127.0.0.1,0.0.0.0,localhost"
    - path: /etc/ssh/sshd_config.d/20-enable-passwords.conf
      mode: 0644
      contents:
        inline: |
          PasswordAuthentication yes
    - path: /etc/vconsole.conf
      mode: 0644
      contents:
        inline: KEYMAP=us
    - path: /etc/sysctl.d/90-k8s.conf
      contents:
        inline: |
          fs.file-max=6815744
          net.core.rmem_max=4194304
          net.core.wmem_max=1048576
          net.ipv4.ip_forward=1
          net.ipv4.ip_local_reserved_ports=30000-32767
          kernel.pid_max=4194304
          net.ipv4.conf.all.arp_announce=2
          net.ipv4.neigh.default.gc_thresh1=8192
          net.ipv4.neigh.default.gc_thresh2=32768
          net.ipv4.neigh.default.gc_thresh3=65536
          net.ipv6.neigh.default.gc_thresh1=8192
          net.ipv6.neigh.default.gc_thresh2=32768
          net.ipv6.neigh.default.gc_thresh3=65536
          vm.max_map_count=262144
          fs.inotify.max_user_watches=65536
          fs.inotify.max_user_instances=8192
          net.ipv4.tcp_syncookies=1
          kernel.panic=10
          kernel.panic_on_oops=1
          vm.overcommit_memory=1
          net.netfilter.nf_conntrack_max=1000000
          net.nf_conntrack_max=1000000
    - path: /etc/hostname
      mode: 0644
      contents:
        inline: coreos
    - path: /etc/selinux/config 
      mode: 0644
      overwrite: true
      contents:
        inline: |
          # disabled selinux
          SELINUX=disabled
          SELINUXTYPE=targeted
systemd:
  units:
    - name: rpm-ostree-install-docker-ce.service
      enabled: true
      contents: |
        [Unit]
        Description=Install Docker CE
        Wants=network-online.target
        After=network-online.target
        ConditionPathExists=!/var/lib/%N.stamp
        [Service]
        Type=oneshot
        RemainAfterExit=yes
        EnvironmentFile=/etc/example-proxy.env
        ExecStart=/usr/bin/curl --output-dir "/etc/yum.repos.d" --remote-name https://download.docker.com/linux/fedora/docker-ce.repo
        ExecStart=/usr/bin/rpm-ostree override remove moby-engine containerd runc --install docker-ce
        ExecStart=/usr/bin/touch /var/lib/%N.stamp
        ExecStart=/usr/bin/systemctl --no-block reboot
        [Install]
        WantedBy=multi-user.target
    - name: deploy-post-notification.service
      enabled: true
      contents: |
        [Unit]
        Description=Notify After Deploy 
        Wants=network-online.target
        After=network-online.target
        [Service]
        Type=oneshot
        RemainAfterExit=yes
        ExecStart=/usr/bin/bash /etc/notify.sh
        [Install]
        WantedBy=multi-user.target
    - name: clean-deploy.service
      enabled: true
      contents: |
        [Unit]
        Description=Notify After Deploy 
        Wants=network-online.target
        After=deploy-post-notification.service
        ConditionPathExists=/etc/notify.sh
        [Service]
        Type=oneshot
        RemainAfterExit=yes
        ExecStart=/usr/bin/systemctl disable deploy-post-notification.service ; /usr/bin/rm -f /etc/notify.sh /etc/systemd/system/deploy-post-notification.service
        [Install]
        WantedBy=multi-user.target
passwd:
  users:
    - name: core
      groups: [root]
      password_hash: $1$6.......
      ssh_authorized_keys:
        - ssh-rsa AAA.....
```

## 2、转换JSON格式后的配置

> butane --pretty --strict focs-example.yml  > config.ign

```json
{
  "ignition": {
    "version": "3.4.0"
  },
  "passwd": {
    "users": [
      {
        "groups": [
          "root"
        ],
        "name": "core",
        "passwordHash": "$1$6......",
        "sshAuthorizedKeys": [
          "ssh-rsa AAAA...."
        ]
      }
    ]
  },
  "storage": {
    "files": [
      {
        "path": "/etc/notify.sh",
        "contents": {
          "compression": "gzip",
          "source": "data:;base64,H4sIAA.....具体配置的Base64编码......"
        },
        "mode": 760
      },
      {
        "path": "/etc/example-proxy.env",
        "contents": {
          "compression": "gzip",
          "source": "data:;base64,H4sIA.....具体配置的Base64编码......"
        },
        "mode": 420
      },
      {
        "path": "/etc/ssh/sshd_config.d/20-enable-passwords.conf",
        "contents": {
          "compression": "",
          "source": "data:,PasswordAuthentication%20yes%0A"
        },
        "mode": 420
      },
      {
        "path": "/etc/vconsole.conf",
        "contents": {
          "compression": "",
          "source": "data:,KEYMAP%3Dus"
        },
        "mode": 420
      },
      {
        "path": "/etc/sysctl.d/90-k8s.conf",
        "contents": {
          "compression": "gzip",
          "source": "data:;base64,H4sI.....具体配置的Base64编码......"
        }
      },
      {
        "path": "/etc/hostname",
        "contents": {
          "compression": "",
          "source": "data:,coreos"
        },
        "mode": 420
      },
      {
        "overwrite": true,
        "path": "/etc/selinux/config",
        "contents": {
          "compression": "",
          "source": "data:,%23%20disabled%20selinux%0ASELINUX%3Ddisabled%0ASELINUXTYPE%3Dtargeted%0A"
        },
        "mode": 420
      }
    ],
    "links": [
      {
        "overwrite": true,
        "path": "/etc/localtime",
        "target": "/usr/share/zoneinfo/Asia/Shanghai"
      }
    ]
  },
  "systemd": {
    "units": [
      {
        "contents": "[Unit]\nDescription=Install Docker CE\nWants=network-online.target\nAfter=network-online.target\nConditionPathExists=!/var/lib/%N.stamp\n[Service]\nType=oneshot\nRemainAfterExit=yes\nEnvironmentFile=/etc/example-proxy.env\nExecStart=/usr/bin/curl --output-dir \"/etc/yum.repos.d\" --remote-name https://download.docker.com/linux/fedora/docker-ce.repo\nExecStart=/usr/bin/rpm-ostree override remove moby-engine containerd runc --install docker-ce\nExecStart=/usr/bin/touch /var/lib/%N.stamp\nExecStart=/usr/bin/systemctl --no-block reboot\n[Install]\nWantedBy=multi-user.target\n",
        "enabled": false,
        "name": "rpm-ostree-install-docker-ce.service"
      },
      {
        "contents": "[Unit]\nDescription=Notify After Deploy \nWants=network-online.target\nAfter=network-online.target\n[Service]\nType=oneshot\nRemainAfterExit=yes\nExecStart=/usr/bin/bash /etc/notify.sh\n[Install]\nWantedBy=multi-user.target\n",
        "enabled": true,
        "name": "deploy-post-notification.service"
      },
      {
        "contents": "[Unit]\nDescription=Notify After Deploy \nWants=network-online.target\nAfter=deploy-post-notification.service\nConditionPathExists=\n[Service]\nType=oneshot\nRemainAfterExit=yes\nExecStart=/usr/bin/systemctl disable deploy-post-notification.service ; /usr/bin/rm -f /etc/notify.sh /etc/systemd/system/deploy-post-notification.service\n[Install]\nWantedBy=multi-user.target\n",
        "enabled": true,
        "name": "clean-deploy.service"
      }
    ]
  }
}

```

# 参考

- https://docs.fedoraproject.org/zh_Hans/fedora-coreos/bare-metal/#_installing_from_the_network
- https://docs.fedoraproject.org/zh_Hans/fedora-coreos/producing-ign/#_getting_butane