# 搭建Allinone全组件Openshift 3.11

## 一、Overviews

#### **Prerequisite**

1. IP地址：192.168.1.86
2. CentOS：7.5.1804
3. 硬盘划分
   1. 系统盘60G / 
   2. 数据盘100G /var/lib/docker ; 100G /data/nfs
4. 开启Selinux
  ```bash
   sed -i "s/SELINUX=disabled/SELINUX=enforcing/" /etc/selinux/config && \
   setenforce 0
   ```

#### **Context**

1. Docker：版本 1.13，Overlay2(执行Ansible准备脚本时会进行安装)
2. Openshift：版本 3.11
3. Kubernetes：版本 v1.11.0

## 二、使用Ansible安装部署

1. **设置主机名并在本地Host文件中添加IP地址域名映射关系**
  ```bash
  ipaddr=$(ip addr | awk '/^[0-9]+: / {}; /inet.*global/ {print gensub(/(.*)\/(.*)/, "\\1", "g", $2)}'| sed -n '1p') && \
  echo $ipaddr $HOSTNAME >> /etc/hosts
  ```
2. **配置中科大Openshift的YUM 源**
    ```bash
      mkdir /etc/yum.repos.d/bak && \
      mv /etc/yum.repos.d/C* /etc/yum.repos.d/bak && \
      bash -c ' cat > /etc/yum.repos.d/all.repo << EOF
    [base]
    name=CentOS-\$releasever - Base
    baseurl=http://mirrors.ustc.edu.cn/centos/\$releasever/os/\$basearch/
    gpgcheck=0
    [updates]
    name=CentOS-\$releasever - Updates
    baseurl=http://mirrors.ustc.edu.cn/centos/\$releasever/updates/\$basearch/
    gpgcheck=0
    [extras]
    name=CentOS-\$releasever - Extras
    baseurl=http://mirrors.ustc.edu.cn/centos/\$releasever/extras/\$basearch/
    gpgcheck=0
    [openshift]
    name=Openshift
    baseurl=http://mirrors.ustc.edu.cn/centos/\$releasever/paas/\$basearch/openshift-origin311/
    gpgcheck=0
    [epel]
    name=Centos EPEL
    baseurl=http://mirrors.ustc.edu.cn/epel/7/x86_64/
    gpgcheck=0
    EOF' && \
      yum clean all && \
      yum makecache
    ```
3. **安装基础软件**
   ```bash
   yum install -y git vim net-tools lrzsz unzip bind-utils yum-utils bridge-utils python-passlib wget java-1.8.0-openjdk-headless httpd-tools lvm2
   ```
4. **安装Ansible 2.6.5**
    ```bash
    yum install -y https://releases.ansible.com/ansible/rpm/release/epel-7-x86_64/ansible-2.6.5-1.el7.ans.noarch.rpm
    ```
5. **获取openshift ansible部署脚本代码，禁用ansible脚本中的指定repo**
    ```bash
    git clone https://github.com/openshift/openshift-ansible.git -b release-3.11 && \
    sed -i 's/enabled=1/enabled=0/g' /root/openshift-ansible/roles/openshift_repos/templates/CentOS-OpenShift-Origin.repo.j2 && \
    sed -i 's/enabled=1/enabled=0/g' /root/openshift-ansible/roles/openshift_repos/templates/CentOS-OpenShift-Origin311.repo.j2
    ```
6. **(可选)将附件中定制化的OKD登陆页面文件放置/etc/origin/master/custom路径下**（[自定义的登陆首页](../assets/openshift-custom-login.html "自定义的登陆首页")）
   ```bash
   # 路径需要新建
   mkdir -p /etc/origin/master/custom
   ```
7. **配置Ansible部署Openshift的主机清单/etc/ansible/hosts**

  ```
  [OSEv3:children]
  masters
  nodes
  etcd
  nfs
  [OSEv3:vars]
  openshift_ip=192.168.1.86
  openshift_public_ip=192.168.1.86
  ansible_default_ipv4.address=192.168.1.86
  ansible_ssh_user=root
  openshift_deployment_type=origin
  deployment_type=origin
  openshift_release=3.11
  openshift_image_tag=v3.11.0
  ansible_ssh_pass=**Root用户SSH密码**
  ######################### Components Cert and CA Expire Days #################
  openshift_hosted_registry_cert_expire_days=36500
  openshift_ca_cert_expire_days=36500
  openshift_node_cert_expire_days=36500
  openshift_master_cert_expire_days=36500
  etcd_ca_default_days=36500

  ####################### Multitenant Network #######################
  os_sdn_network_plugin_name=redhat/openshift-ovs-multitenant

  ####################### OKD #######################
  openshift_clock_enabled=true
  openshift_enable_unsupported_configurations=True
  openshift_node_groups=[{'name': 'allinone', 'labels': ['node-role.kubernetes.io/master=true', 'node-role.kubernetes.io/infra=true', 'node-role.kubernetes.io/compute=true']}]
  openshift_disable_check=memory_availability,disk_availability,package_availability,package_update,docker_image_availability,docker_storage_driver,docker_storage

  ####################### OKD master config #######################

  openshift_master_api_port=8443
  openshift_master_cluster_public_hostname=allinone.okd311.curiouser.com
  openshift_master_cluster_hostname=allinone.okd311.curiouser.com
  openshift_master_default_subdomain=apps.okd311.curiouser.com
  openshift_master_identity_providers=[{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider'}]
  openshift_master_htpasswd_users={'admin':'$apr1$eG8zNL.C$fvACBzDJ7.N7KdJORT12E0'}
  openshift_master_oauth_template=custom/login.html
  openshift_master_session_name=ssn
  openshift_master_session_max_seconds=3600

  ####################### Docker #######################

  container_runtime_docker_storage_setup_device=/dev/sdb
  container_runtime_docker_storage_type=overlay2

  openshift_examples_modify_imagestreams=true
  openshift_docker_options="--selinux-enabled -l warn --ipv6=false --insecure-registry=0.0.0.0/0 --log-opt max-size=10M --log-opt max-file=3 --registry-mirror=https://zlsoueh7.mirror.aliyuncs.com"

  ####################### Web Console #######################

  openshift_web_console_extension_script_urls=["https://xhua-static.sh1a.qingstor.com/allinone/allinone-webconsole.js"]
  openshift_web_console_extension_stylesheet_urls=["https://hermes-uat.curiouser.com/curiouser/M00/00/3A/rBACF1vz8NyALOS3AAApT8C9PDY549.css"]

  ####################### Registry #######################

  openshift_hosted_registry_storage_kind=nfs
  openshift_hosted_registry_storage_access_modes=['ReadWriteMany']
  openshift_hosted_registry_storage_nfs_directory=/data/nfs
  openshift_hosted_registry_storage_nfs_options='*(rw,root_squash,sync,no_wdelay)'
  openshift_hosted_registry_storage_volume_name=registry
  openshift_hosted_registry_storage_volume_size=10Gi

  ####################### metrics #######################

  openshift_metrics_install_metrics=true
  openshift_metrics_image_version=v3.11.0
  openshift_metrics_storage_kind=nfs
  openshift_metrics_storage_access_modes=['ReadWriteOnce']
  openshift_metrics_storage_nfs_directory=/data/nfs
  openshift_metrics_storage_nfs_options='*(rw,root_squash,sync,no_wdelay)'
  openshift_metrics_storage_volume_name=metrics
  openshift_metrics_storage_volume_size=10Gi

  ####################### logging #######################

  openshift_logging_install_logging=true
  openshift_logging_image_version=v3.11.0
  openshift_logging_es_ops_nodeselector={"node-role.kubernetes.io/infra":"true"}
  openshift_logging_es_nodeselector={"node-role.kubernetes.io/infra":"true"}
  openshift_logging_elasticsearch_pvc_size=5Gi
  openshift_logging_storage_kind=nfs
  openshift_logging_storage_access_modes=['ReadWriteOnce']
  openshift_logging_storage_nfs_directory=/data/nfs
  openshift_logging_storage_nfs_options='*(rw,root_squash,sync,no_wdelay)'
  openshift_logging_storage_volume_name=logging
  openshift_logging_storage_volume_size=10Gi

  ################### Prometheus Cluster Monitoring ###################

  openshift_cluster_monitoring_operator_install=true
  openshift_cluster_monitoring_operator_prometheus_storage_enabled=true
  openshift_cluster_monitoring_operator_alertmanager_storage_enabled=true
  openshift_cluster_monitoring_operator_prometheus_storage_capacity=50Gi
  openshift_cluster_monitoring_operator_alertmanager_storage_capacity=5Gi

  ##################### Disable Components #############

  openshift_enable_service_catalog=false
  ansible_service_broker_install=false

  [masters]
  allinone.okd311.curiouser.com
  [etcd]
  allinone.okd311.curiouser.com
  [nfs]
  allinone.okd311.curiouser.com
  [nodes]
  allinone.okd311.curiouser.com openshift_node_group_name='allinone'
  ```
9. **创建NFS挂载目录**

  ```
  pvcreate /dev/sdc && \
  vgcreate -s 4m data /dev/sdc && \
  lvcreate --size 45G -n nfs data && \
  mkfs.xfs /dev/data/nfs && \
  echo "/dev/data/nfs /data/nfs xfs defaults 0 0" >> /etc/fstab && \
  mkdir /data/nfs -p && \
  mount -a && \
  df -mh
  ```
11. **（可选）预先拉取安装过程中可能使用的镜像**
  ```
  docker pull docker.io/openshift/origin-node:v3.11.0 && \
  docker pull docker.io/openshift/origin-control-plane:v3.11.0 && \
  docker pull docker.io/openshift/origin-haproxy-router:v3.11.0 && \
  docker pull docker.io/openshift/origin-deployer:v3.11.0 && \
  docker pull docker.io/openshift/origin-pod:v3.11.0 && \
  docker pull docker.io/openshift/origin-docker-registry:v3.11.0 && \
  docker pull docker.io/openshift/origin-console:v3.11.0 && \
  docker pull docker.io/openshift/origin-service-catalog:v3.11.0 && \
  docker pull docker.io/openshift/origin-web-console:v3.11.0 && \
  docker pull docker.io/cockpit/kubernetes:latest && \
  docker pull docker.io/openshift/oauth-proxy:v1.1.0 && \
  docker pull docker.io/openshift/origin-docker-builder:v3.11.0 && \
  docker pull docker.io/openshift/prometheus-alertmanager:v0.15.2 && \
  docker pull docker.io/openshift/prometheus-node-exporter:v0.16.0 && \
  docker pull docker.io/openshift/prometheus:v2.3.2 && \
  docker pull docker.io/grafana/grafana:5.2.1 && \
  docker pull quay-mirror.qiniu.com/coreos/kube-rbac-proxy:v0.3.1 && \
  docker tag quay-mirror.qiniu.com/coreos/kube-rbac-proxy:v0.3.1 quay.io/coreos/kube-rbac-proxy:v0.3.1 && \
  docker rmi quay-mirror.qiniu.com/coreos/kube-rbac-proxy:v0.3.1 && \
  docker pull quay-mirror.qiniu.com/coreos/etcd:v3.2.22 && \
  docker tag quay-mirror.qiniu.com/coreos/etcd:v3.2.22 quay.io/coreos/etcd:v3.2.22 && \
  docker rmi quay-mirror.qiniu.com/coreos/etcd:v3.2.22 && \
  docker pull quay-mirror.qiniu.com/coreos/kube-state-metrics:v1.3.1 && \
  docker tag quay-mirror.qiniu.com/coreos/kube-state-metrics:v1.3.1 quay.io/coreos/kube-state-metrics:v1.3.1 && \
  docker rmi quay-mirror.qiniu.com/coreos/kube-state-metrics:v1.3.1 && \
  docker pull quay-mirror.qiniu.com/coreos/configmap-reload:v0.0.1 && \
  docker tag quay-mirror.qiniu.com/coreos/configmap-reload:v0.0.1 quay.io/coreos/configmap-reload:v0.0.1 && \
  docker rmi quay-mirror.qiniu.com/coreos/configmap-reload:v0.0.1 && \
  docker pull quay-mirror.qiniu.com/coreos/cluster-monitoring-operator:v0.1.1 && \
  docker tag quay-mirror.qiniu.com/coreos/cluster-monitoring-operator:v0.1.1 quay.io/coreos/cluster-monitoring-operator:v0.1.1 && \
  docker rmi quay-mirror.qiniu.com/coreos/cluster-monitoring-operator:v0.1.1 && \
  docker pull quay-mirror.qiniu.com/coreos/prometheus-config-reloader:v0.23.2 && \
  docker tag quay-mirror.qiniu.com/coreos/prometheus-config-reloader:v0.23.2 quay.io/coreos/prometheus-config-reloader:v0.23.2 && \
  docker rmi quay-mirror.qiniu.com/coreos/prometheus-config-reloader:v0.23.2 && \
  docker pull quay-mirror.qiniu.com/coreos/prometheus-operator:v0.23.2  && \
  docker tag quay-mirror.qiniu.com/coreos/prometheus-operator:v0.23.2 quay.io/coreos/prometheus-operator:v0.23.2 && \
  docker rmi quay-mirror.qiniu.com/coreos/prometheus-operator:v0.23.2
  ```
12. **执行OKD Ansible Playbook**  
    先执行安装检查的Playbook

    ```bash
    ansible-playbook /root/openshift-ansible/playbooks/prerequisites.yml
    ```

    再执行安装Playbook

    ```
    ansible-playbook /root/openshift-ansible/playbooks/deploy_cluster.yml
    ```

13. **授予admin用户以管理员权限**

  ```
  oc adm policy add-cluster-role-to-user cluster-admin admin
  ```

## 三、配置Openshift的后端存储

* 使用Ceph RBD作为后端存储
  1. 搭建单节点的Ceph，详见（[Ceph RBD单节点安装](/infrastructure/install/ceph-rbddan-jie-dian-an-zhuang.md "Ceph单节点安装")）
  2. 创建Storageclass
* 使用NFS作为后端存储：详见

### 参考文章链接

当主机有多网卡时指定组件监听的网卡IP地址：[https://github.com/ViaQ/Main/blob/master/README-install.md](https://github.com/ViaQ/Main/blob/master/README-install.md)

