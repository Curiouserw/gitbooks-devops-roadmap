# Wazuh 主动防御

# 一、简介

# 二、默认自带的响应脚本

## 1、Unix-like

- Linux/Unix： `/var/ossec/active-response/bin`
- macOS ： `/Library/Ossec/active-response/bin` directory.

| 脚本名                                                       | 功能描述                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [disable-account](https://github.com/wazuh/wazuh/blob/v4.8.2/src/active-response/disable-account.c) | 通过设置 passwd-l 禁用帐户                                   |
| [firewall-drop](https://github.com/wazuh/wazuh/blob/v4.8.2/src/active-response/firewalls/default-firewall-drop.c) | 将 IP 添加到 iptables 拒绝列表                               |
| [firewalld-drop](https://github.com/wazuh/wazuh/blob/v4.8.2/src/active-response/firewalld-drop.c) | 将 IP 添加到 firewalld drop列表（需安装firewalld）           |
| [host-deny](https://github.com/wazuh/wazuh/blob/v4.8.2/src/active-response/host-deny.c) | 将 IP 添加到 /etc/hosts.deny 文件                            |
| [ip-customblock](https://github.com/wazuh/wazuh/blob/v4.8.2/src/active-response/ip-customblock.c) | 自定义 OSSEC 块，可轻松修改以实现自定义响应                  |
| [ipfw](https://github.com/wazuh/wazuh/blob/v4.8.2/src/active-response/firewalls/ipfw.c) | ipfw 创建的防火墙drop响应脚本（需安装IPFW）                  |
| [npf](https://github.com/wazuh/wazuh/blob/v4.8.2/src/active-response/firewalls/npf.c) | npf 创建的防火墙drop响应脚本（需安装NPF）                    |
| [wazuh-slack](https://github.com/wazuh/wazuh/blob/v4.8.2/src/active-response/wazuh-slack.c) | Posts notifications on Slack. Requires a slack hook URL passed as an `extra_args`. |
| [pf](https://github.com/wazuh/wazuh/blob/v4.8.2/src/active-response/firewalls/pf.c) | pf 创建的防火墙drop响应脚本（需安装PF）                      |
| [restart.sh](https://github.com/wazuh/wazuh/blob/master/src/active-response/restart.sh) | ossec.conf 更改后自动重启 Wazuh agent 或 manager.            |
| [restart-wazuh](https://github.com/wazuh/wazuh/blob/v4.8.2/src/active-response/restart-wazuh.c) | ossec.conf 更改后自动重启 Wazuh agent 或 manager.            |
| [route-null](https://github.com/wazuh/wazuh/blob/v4.8.2/src/active-response/route-null.c) | 将 IP 地址添加到空路由                                       |
| [kaspersky](https://github.com/wazuh/wazuh/blob/master/src/active-response/kaspersky.c) | 与卡巴斯基端点安全集成。这使用 Kaspersky Endpoint Security for Linux CLI 根据触发器执行相关命令。 |

## 2、Windows

 `C:\Program Files (x86)\ossec-agent\active-response\bin`

| 脚本名                                                       | 功能描述                                         |
| ------------------------------------------------------------ | ------------------------------------------------ |
| [netsh.exe](https://github.com/wazuh/wazuh/blob/v4.8.2/src/active-response/netsh.c) | 使用 `netsh`禁止 IP                              |
| [restart-wazuh.exe](https://github.com/wazuh/wazuh/blob/v4.8.2/src/active-response/restart-wazuh.c) | ossec.conf 更改后自动重启 Wazuh agent 或 manager |
| [route-null.exe](https://github.com/wazuh/wazuh/blob/v4.8.2/src/active-response/route-null.c) | 将 IP 地址添加到空路由                           |

# 三、自定义响应脚本

