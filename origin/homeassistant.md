# HomeAssistant

# 一、简介

# 二、刻录安装

## 1、刻录

建议使用 树莓派官方刻录软件`Raspberry Pi Imager v2` 刻录在线或离线刻录 HA 到SSD 固态硬盘(推荐)中，不推荐 SD 卡，至于如何设置树莓派使用 SSD 硬盘作为启动盘，参考[树莓派设置USB SSD磁盘启动](raspberry-pi.md)

- `Raspberry Pi Imager v2` 刻录 OS 选项选择`Other specific-purpose OS` --> `Home automation` ---> `Home Assisstant` --> `Home Assisstant OS 16.3`
- 手动下载镜像文件后使用`Raspberry Pi Imager v2` 刻录 
  - [树莓派5镜像文件下载链接](https://github.com/home-assistant/operating-system/releases/download/16.3/haos_rpi5-64-16.3.img.xz)
  - [树莓派4镜像文件下载链接](https://github.com/home-assistant/operating-system/releases/download/16.3/haos_rpi4-64-16.3.img.xz)

## 2、加速配置准备阶段

由于 haoss需要从 ghcr.io 拉取 docker 镜像，网络不好会一直卡在配置中，一直卡..............

- 在 UI 界面日志中查找当前正在拉取的组件docker镜像版本（一般准备阶段会检测最新的版本镜像自动更新下载，主要还是`raspberrypi4-64-homeassistant` 这个组件的镜像比较大。也可以在https://github.com/orgs/home-assistant/packages 页面查看各个组件最新的镜像版本，带有 latest、stable 标签的是最新的版本号）

- 在可以跳过网络限制下载doceker 镜像的电脑上 `docker pull ghcr.io/home-assistant/raspberrypi4-64-homeassistant:2026.1.1后`打包镜像为 tar文件

  ```bash
  docker pull ghcr.io/home-assistant/raspberrypi4-64-homeassistant:2026.1.1
  docker save -o raspberrypi4-64-homeassistant-20260101.tar ghcr.io/home-assistant/raspberrypi4-64-homeassistant:2026.1.1
  ```

- 开启 nginx 静态代理这个tar 文件

- 在 HAOS 命令行界面登录 shell 环境 ，参照第三章第二节

- 使用 `curl` 命令下载到`/data`路径下，然后 `docker loda -i raspberrypi4-64-homeassistant-20260101.tar`

  ```bash
  curl -o /data/raspberrypi4-64-homeassistant-20260101.tar http://192.168.1.1:8080/raspberrypi4-64-homeassistant-20260101.tar
  docker load -i /data/raspberrypi4-64-homeassistant-20260101.tar
  ```

- 重启 HAOS

  ```bash
  ha > host reboot
  ```

- 将“`Advanced SSH & Web Terminal`”加载项的 docker 镜像也下载导入到 HAOS 中，后续在 UI 加载项界面安装加载项后，可以快速安装成功。在之后就可以直接远程ssh登录 HAOS了

  ```bash
  docker pull ghcr.io/hassio-addons/ssh/aarch64:22.0.3
  docker save -o ssh.tar ghcr.io/hassio-addons/ssh/aarch64:22.0.3
  
  
  curl -o /data/ssh.tar http://192.168.1.1:8080/ssh.tar
  docker load -i /data/ssh.tar
  ```

# 三、命令行配置

## 1、网络配置

```bash
# 设置 WIFI
ha > network update wlan0 --wifi-ssid "WIFI" --wifi-psk "WIFI密码" --wifi-mode infrastructure --wifi-auth wpa-psk --ipv4-method auto

ha > network info
```

## 2、shell环境

```bash
ha > login
#
```

## 3、其他系统信息

- observer端口：**4357**

- Web UI端口：**8123**

## 4、主机配置

```bash
# 重启
ha > host reboot

# 显示主机信息
ha > host info
```

# 四、系统配置

## 1、添加IP摄像头

设置 --> 设备与服务 --> 设备 --> 添加设备 --> 通用摄像头 

# 五、插件配置

## 1、SSH服务：Advanced SSH & Web Terminal

```yaml
ssh:
  username: root
  password: ""              # 置空则禁止 root 使用密码登录
  authorized_keys:
    - ssh-ed25519 AAAA*****HDvLH pc1
  sftp: true                  # 开启SFTP 必须设置username为 root
  compatibility_mode: false       # 不建议开启，开启是为了兼容老版本的客户端，开启后则不太安全
  allow_agent_forwarding: true
  allow_remote_port_forwarding: true
  allow_tcp_forwarding: true
zsh: true
share_sessions: true
packages: []
init_commands: []
```

## 2、MariaDB

设置密码及权限

```bash
databases:
  - homeassistant
logins:
  - password: 密码
    username: homeassistant
rights:
  - database: homeassistant
    username: homeassistant
    privileges:
      - ALTER

# 权限可设置ALTER、CREATE、CREATE ROUTINE、CREATE TEMPORARY TABLES、CREATE VIEW、DELETE、DELETE HISTORY、DROP、EVENT、GRANT OPTION、INDEX、INSERT、LOCK TABLES、SELECT、SHOW VIEW、TRIGGER、UPDATE

# 权限置空则表示授予全部权限ALL PRIVILEGES
```

# 六、第三方加载项库

- https://github.com/alexbelgium/hassio-addons

# 七、自定义加载项

## 1、目录结构

在 `config/addons` 下新建一个目录，例如：

```bash
├── README.md                           # 存储库的总体说明
└── lib                                 # 可供所有加载项共享的库或脚本
   └── ...
└── config                              # 配置文件示例或其他通用配置
   └── ...
└── snapshots                           # 载入点用于备份和恢复加载项数据
└── scripts                             # 辅助脚本，如编译或构建工具
└── Dockerfile                          # Docker 文件模板，为所有加载项提供基础设置
└──零散的目录（如「aircast」、「deconz」等） # 各个具体的加载项
   ├── Dockerfile                       # 每个加载项的 Docker 容器构建文件
   ├── configuration.yaml.example       # 示例配置文件
   ├── init.sh                          # 加载项启动时执行的脚本
   └── ...                              # 其他相关文件和资源
```

## 2、config.yaml

```yaml
name: "Hello World"
slug: "hello_world"
version: "1.0.0"
description: "Hello World Addon Example"
arch:
  - aarch64
  - armv7
  - armhf
init: false
```

## 3、Dockerfile

```bash
ARG BUILD_FROM
FROM $BUILD_FROM
COPY run.sh /
RUN chmod a+x /run.sh
CMD [ "/run.sh" ]
```

## 4、run.sh

```bash
#!/usr/bin/with-contenv bashio
echo "Hello World from custom addon!"
```

这只是最简单的示例，正常 Add-on 会根据功能添加更多内容。

## 5、安装自定义插件

1. 在 HA 的 **Add-on Store 页面**，点击右上角 **Refresh（刷新/检查更新）** → 这时候 HA 会扫描本地 `addons` 文件夹。
2. 你会看到一个新的 **Local Add-on（本地加载项）** 分类，里面有你的插件。
3. 点击它，然后选择 **Install（安装）** → **Start（启动）**。
4. 打开 **Logs（日志）** 查看输出内容确认运行。

## 6、定义配置与高级选项

Add-on 的配置文件 `config.yaml` 可以包含很多字段，例如：

| 字段      | 意义                                  |
| --------- | ------------------------------------- |
| `map`     | 挂载数据卷，如配置文件或 SSL 证书目录 |
| `ports`   | 暴露网络端口                          |
| `ingress` | 是否启用 HA 前端集成访问              |
| `options` | 插件可自定义的配置项                  |
| `schema`  | 插件配置表单规范                      |

## 7、小技巧与常见问题

🚀 **启用高级模式**
 如果你没看到某些插件（比如测试版的），可以去用户设置开启 **Advanced Mode（高级模式）** 让更多加载项可见。
