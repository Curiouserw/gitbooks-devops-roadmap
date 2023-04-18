# 一、Cockpit简介
Cockpit 是一个自由开源的服务器管理软件，它使得我们可以通过它好看的 web 前端界面轻松地管理我们的 GNU/Linux 服务器。Cockpit 使得 linux 系统管理员、系统维护员和开发者能轻松地管理他们的服务器并执行一些简单的任务，例如管理存储、检测日志、启动或停止服务以及一些其它任务。它的报告界面添加了一些很好的功能使得可以轻松地在终端和 web 界面之间切换。另外，它不仅使得管理一台服务器变得简单，更重要的是只需要一个单击就可以在一个地方同时管理多个通过网络连接的服务器。它非常轻量级，web 界面也非常简单易用。在这篇博文中，我们会学习如何安装 Cockpit 并用它管理我们的运行着 Fedora、CentOS、Arch Linux 以及 RHEL 发行版操作系统的服务器。下面是 Cockpit 在我们的 GNU/Linux 服务器中一些非常棒的功能：
* 它包含 systemd 服务管理器。
* 有一个用于故障排除和日志分析的 Journal 日志查看器。
* 包括 LVM 在内的存储配置比以前任何时候都要简单。
* 用 Cockpit 可以进行基本的网络配置。
* 可以轻松地添加和删除用户以及管理多台服务器。

# 二、Cockpit安装

1. 所有集群节点安装cockpit并启动服务

        yum install -y cockpit cockpit-docker cockpit-kubernetes ;\
        systemctl enable cockpit ;\
        systemctl start cockpit ;\
        netstat -lanp |grep 9090

1. iptables放行端口

        vi /etc/sysconfig/iptables 
        #-A INPUT -p tcp -m state --state NEW -m tcp --dport 9090 -j ACCEPT
        systemctl restart iptables 