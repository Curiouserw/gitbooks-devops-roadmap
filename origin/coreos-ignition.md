# CoreOS无人值守自动化配置Ignition详解

# 一、简介





官方文档：https://coreos.github.io/ignition/



# 二、PXE配置

```bash
LABEL 'CoreOS 40.20240616.3.0 x86_64'
    menu label ^5> MemTest: test memory and cpu
    kernel CoreOS/40.20240616.3.0/fedora-coreos-40.20240616.3.0-live-kernel-x86_64
    initrd CoreOS/40.20240616.3.0/fedora-coreos-40.20240616.3.0-live-initramfs.x86_64.img,CoreOS/40.20240616.3.0/fedora-coreos-40.20240616.3.0-live-rootfs.x86_64.img
    APPEND coreos.inst.install_dev=/dev/sda coreos.inst.ignition_url=http://192.168.1.1:8089/CoreOS/40.20240616.3.0/config.ign
```



# 三、Ignition参数



# 四、butane转换校验工具

ignition配置文件为 json 格式的。跟 kickstart 的 yaml格式的不太一样。但是可以使用工具`butane`将yaml 格式的转换为json 格式的。在转换过程中还可以校验语法。

下载地址：https://github.com/coreos/butane/releases

转换命令：`butane --pretty --strict example.yaml > config.ign`

# 五、示例

```yaml
variant: fcos
version: 1.5.0
storage:
  links:
    - path: /etc/localtime
      overwrite: true
      target: /usr/share/zoneinfo/Asia/Shanghai
  files:
    - path: /etc/example-proxy.env
      mode: 0644
      contents:
        inline: |
          https_proxy="http://192.168.1.1:0110"
          all_proxy="http://192.168.1.1:0110"
          http_proxy="http://192.168.1.1:0110"
          HTTP_PROXY="http://192.168.1.1:0110"
          HTTPS_PROXY="http://192.168.1.1:0110"
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
        Before=zincati.service
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
passwd:
  users:
    - name: core
      groups: [wheel, sudo]
      password_hash: $1$6/87......d/
      ssh_authorized_keys:
        - ssh-rsa AAA......
```









# 参考

- https://docs.fedoraproject.org/zh_Hans/fedora-coreos/bare-metal/#_installing_from_the_network
- https://docs.fedoraproject.org/zh_Hans/fedora-coreos/producing-ign/#_getting_butane