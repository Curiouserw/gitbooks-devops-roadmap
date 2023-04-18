# keepalived

# 一、简介

虚拟路由冗余协议 (Virtual Router Redundancy Protocol，简称VRRP



# 二、安装配置

## 安装

### 包管理器安装

```bash
yum/apt install -y keepalived

# 配置文件：/etc/keepalived/keepalived.conf
# 程序文件：/usr/sbin/keepalived
# Unit File：keepalived.service
# Unit File的环境配置文件：/etc/sysconfig/keepalived
```

### 源码安装

```bash
version=2.1.5
curl -x https://www.keepalived.org/software/keepalived-$version.tar.gz | tar -xC /opt
cd /opt/keepalived-$version
./configure
make
make install
keepalived --version
```

## 配置

`/etc/keepalived/keepalived.conf`

```bash
global_defs {
   
   router_id Curiouser   				# 声明虚拟路标识符，一般会写当前主机名称
   vrrp_skip_check_adv_addr  		# 所有报文都检查比较消耗性能，此配置为如果收到的报文和上一个报文是同一个路由器发出的则跳过检查报文中的源地址。
   vrrp_iptables #禁用防火墙策略,keepalived默认启动时会自动生成iptables策略，因此我们启用此项就可以不生成iptables的策略。
   
   #严格遵守VRRP协议,即不允许以下三种状况:1.没有VIP地址;2.单播邻居;3.在VRRP版本2中有IPv6地址
   
   #vrrp_strict #由于我下面配置基于单播地址发送消息,因此我这里手动禁用了严格模式，直接注释即可。
   vrrp_garp_interval 0 				# ARP报文发送延迟时间,0表示不延迟。
   vrrp_gna_interval 0 					# 消息发送延迟,0表示不延迟。
}

vrrp_script check_nginx {
    script "/etc/keepalived/keepalived-untils.sh check_ngix"
    interval  10
    weight -20
}

#使用vrrp_instance指令定义一个VIP实例名称,这里定义VIP实例的名称为"VIP1",生产环境建议该名称和业务相结合。
vrrp_instance VIP1 {
    state BACKUP  							# 指定当前实例默认角色,当前节点在此虚拟路由器上的初始状态，状态为MASTER或者BACKUP
    
    #定义工作模式为非抢占模式(即当master重启,VIP会飘移到其它节点,重启完成后并不会将vip抢过来),需要两个keepalived的state的值均为"BACKUP",让它们根据个节点的优先级选择对应的master nopreempt 

    interface eth1 							# 指定通过哪个本地网卡发送vrrp广播
    virtual_router_id 27 				# 定义当前虚拟路由器惟一标识,该id的范围是0-255,注意，用一组keepalived的id编号必须要一致
    priority 150 								# 当前物理节点在此虚拟路由器中的优先级；范围1-254
    advert_int 2 								# 定义vrrp广播的时间间隔，默认1s
    unicast_src_ip 192.168.1.11 # 指定单播地址的源地址,需要禁用严格模式"vrrp_strict"
    
    
    unicast_peer {
        172.30.1.103						# 指定单播地址的对端地址
    }  
  
    authentication {
        auth_type PASS 					# 定义认证机制,密码仅支持8位
        auth_pass 12345678
    }
    
    #指定虚拟IP地址,可以指定多个。
    virtual_ipaddress {
        192.168.1.11 dev eth1 label eth1:0
        192.168.1.12 dev eth1 label eth1:1
    }
    
    track_script {
        check_nginx
    }
    
    #定义通知脚本,当前节点成为主节点时触发的脚本
    notify_master "/etc/keepalived/notify.sh master"
    #定义通知脚本,当前节点转为备节点时触发的脚本
    notify_backup "/etc/keepalived/notify.sh backup"
    #定义通知脚本,当前节点转为“失败”状态时触发的脚本
    notify_fault "/etc/keepalived/notify.sh fault"
}
```







脚本

`keepalived-untils.sh`

```bash
#!/bin/sh

case $1 in 
  check_ngix )
  	nginxpid=$(ps -C nginx --no-header|wc -l)
    #1.判断Nginx是否存活,如果不存活则尝试启动Nginx
    if [ $nginxpid -eq 0 ];then
        systemctl start nginx
        sleep 3
        #2.等待3秒后再次获取一次Nginx状态
        nginxpid=$(ps -C nginx --no-header|wc -l) 
        #3.再次进行判断, 如Nginx还不存活则停止Keepalived,让地址进行漂移,并退出脚本  
        if [ $nginxpid -eq 0 ];then
            systemctl stop keepalived
       fi
    fi
  ;;
	notify_master )
	
	;;
	notify_slave )
	
	;;
	
	notify_fault )
	
	;;
	
	* )
	;;
esac
```

