# 一、集群添加Node节点

Ansible脚本有新增节点的Playbook脚本，准备好新增节点的基础环境，在集群的ansible管理节点上执行该Playbook就行。

##　Context

| OKD版本 | OS版本 | Docker版本 | Ansible版本 |
| ------- | ------ | -------- | ------- |
| 3.11 | CentOS 7.5.1804 | 1.13.1 | 2.6.5 |



## 1. 新增节点Prerequisite

新增node节点IP地址及主机名：192.168.1.23 node6.okd.curiouser.com

* 开启seLinux
    ```bash
    sed -i "s/SELINUX=disabled/SELINUX=enforcing/" /etc/sysconfig/selinux && \
    setenforce 1
    ```
* 安装docker，jdk及基础软件
    ```bash
    yum install -y docker vim lrzsz wget unzip net-tools telnet bind-utils && \
    systemctl enable docker && \
    systemctl start docker && \
    systemctl status docker && \
    yum localinstall -y jdk-8u191-linux-x64.rpm && \
    docker info && \
    java -version
    ```
* 配置DNS，发现集群其他节点的IP地址与域名的映射关系.(注意DNSMasq服务端的iptables是否放行DNS的53 UDP端口)
  
  由于集群内有DNSMasq服务端，配置/etc/resolv.conf
  ```bash
  echo "nameserver 192.168.1.22" >> /etc/resolv.conf && \
  ping allinone311.okd.curiouser.com
  ```
  **Note**: 
  ```bash
  #DNSMasq服务端放行DNS的53 UDP端口
  iptables -I OS_FIREWALL_ALLOW -p udp -m udp --dport 53 -j ACCEPT && \
  iptables-save
  ```
* 配置Openshift的YUM源
    ```bash
    mkdir /etc/yum.repos.d/bak && \
    mv /etc/yum.repos.d/C* /etc/yum.repos.d/bak && \
    scp allinone311.okd.curiouser.com:/etc/yum.repos.d/all.repo /etc/yum.repos.d/ && \
    yum clean all && \
    yum makecache
    ```
   
 

## 2. ansible管理节点
  * 打通ansible管理节点到新增node节点的SSH免密通道
    ```bash
    ssh-copy-id -i root@node6.okd.curiouser.com && \
    ssh root@node1.okd.curiouser.com 
    ```
   * ansible管理节点的ansible主机清单文件inventory中添加新增节点相关信息
     ```bash
     [OSEv3:children]
     ...
     new_nodes
     
     [new_nodes]
     node1.okd.curiouser.com openshift_node_group_name="node-config-all-in-one"
     ```   
   * ansible管理节点执行新增节点的Ansible Playbook
     ```bash
     ansible-playbook /root/openshift-ansible/playbooks/openshift-node/scaleup.yml
     ```

## 注意1：
当执行脚本时tower主机会把它的dnsmasq配置/etc/dnsmasq.d/origin-upstream-dns.conf同步到新增节点/etc/dnsmasq.d/路径下。由于tower主机的/etc/dnsmasq.d/origin-upstream-dns.conf设置的上游DNS服务器为外网的。不希望新增节点的上游DNS服务器走外网，而是走tower主机，形成集群只有Tower主机一个节点的dns对外，其他主机作为Tower主机dns服务的客户端。所以当tower主机/etc/dnsmasq.d/origin-upstream-dns.conf同步到新增节点/etc/dnsmasq.d/路径下的时候，及时修改上游dns服务器为tower主机。然后重启dnsmasq。有两个明显的坑:

① 无法重启dnsmasq，报以下错误：
```bash
DBus error: Connection ":1.50" is not allowed to own the service "uk.org.thekelleys.dnsmasq" due to security policies in the configuration file
```
解决方案：重启dbus，再重启dnsmasq
```bash
systemctl restart dbus && \
systemctl restart dnsmasq
```
②tower主机的iptables服务开启，dns的53端口没有放开，导致新增节点的dns无法连接上游dns服务器（即Tower主机的dns服务）
解决方案：tower主机放行dns服务的UDP 53端口。（可在新增节点尝试nslookup解析域名试一下）
```bash
iptables -I OS_FIREWALL_ALLOW -p udp -m udp --dport 53 -j ACCEPT && \
iptables-save
```
## 注意2：
如果出现收集allinone节点facts超时的报错，出现一下错误提示

```bash
 The full traceback is:
Traceback (most recent call last):
  File "/tmp/ansible_d9POp0/ansible_modlib.zip/ansible/module_utils/basic.py", line 2853, in run_command
    cmd = subprocess.Popen(args, **kwargs)
  File "/usr/lib64/python2.7/subprocess.py", line 711, in __init__
    errread, errwrite)
  File "/usr/lib64/python2.7/subprocess.py", line 1308, in _execute_child
    data = _eintr_retry_call(os.read, errpipe_read, 1048576)
  File "/usr/lib64/python2.7/subprocess.py", line 478, in _eintr_retry_call
    return func(*args)
  File "/tmp/ansible_d9POp0/ansible_modlib.zip/ansible/module_utils/facts/timeout.py", line 37, in _handle_timeout
    raise TimeoutError(msg)
TimeoutError: Timer expired after 10 seconds
```
TimeoutError: Timer expired after 10 seconds
请在/etc/ansible/ansible.cfg 设置"gather_subset = !all"或者"gather_timeout=300"。原因可能是已经运行allinone节点上的facts（特别是docker images layer的挂载信息）过多，造成收集facts超时，默认收集facts超时时间是10。

相关连接：https://github.com/ansible/ansible/issues/43884



# 二、删除节点

1. 疏散要删除节点上的POD
    ```bash
    oc adm drain <node1> <node2> [--pod-selector=<pod_selector>] --force=true --grace-period=-1 --timeout=5s --delete-local-data=true
    ```
2. 删除Node
    ```bash
    oc delete node <node>
    ```