# **Summary**

* [简介](README.md)

## Part Ⅰ :  容器云Openshift
* 安装
    * [Allinone](origin/openshift-allinone安装.md)
    * [OCP 4.3 集群](origin/openshift-ocp43-install.md)
* 基础知识
* 集群管理
    * [数据持久化](origin/openshift-Kubernetes的持久化存储.md)
        * [NFS Client provisioner](origin/openshift-Kubernetes-provisioner-nfs-client.md)
        * [NFS Server Provisioner](origin/openshift-Kubernetes-provisioner-nfs-server.md)
        * [Glusterfs Provisioner](origin/openshift-Kubernetes-provisioner-glusterfs.md)
        * [Ceph FileSystem Provisioner](origin/openshift-Kubernetes-provisioner-cephfs.md)
        * [Ceph RBD Provisioner](origin/openshift-Kubernetes-provisioner-cephrbd.md)
    * 管理
        * 资源对象管理
            * [常见资源对象操作](origin/openshift-资源对象常见操作.md)
            * [将Secret和ConfigMap以文件的形式挂载到容器](origin/openshift-将Secret和ConfigMap以文件的形式挂载到容器.md)
        * 集群管理
            * [节点管理](origin/openshift-集群节点管理.md)
            * [节点状态监控](origin/openshift-使用Cockpit监控集群节点的系统状态.md)
            * [集群组件TLS证书管理](origin/openshift-集群组件TLS证书管理.md)
            * [定制WebConsole界面](origin/openshift-WebConsole定制化.md)
            * [集群管理遇到的问题](origin/openshift-no-IP-addresses-available-in-range-set解决方案.md)
            * [用户认证](origin/openshift-openshift的用户认证.md)
            * [用户权限管理实例](origin/openshift-openshift用户权限管理实例.md)
    * 网络
        * [openshift开启router的haproxy-statisc](origin/openshift-开启router的haproxy-statisc.md)
        * [openshift的多租户网络](origin/openshift-多租户网络.md)
    * 安全审计
        * [Kubernetes的审计日志功能](origin/openshift-kubernetes的审计日志功能.md)
* 工具应用部署
    * [Elasticsearch容器化部署](origin/openshift-elasticsearch容器化部署.md)
    * [Kibana容器化部署](origin/openshift-Kibana容器化部署.md)

## Part Ⅱ：容器云Kubernetes
* 基础
  * [Kubernetes的集群角色及插件](origin/kubernetes-集群角色及插件.md)
  * [将POD信息传递容器环境变量或文件](origin/k8s-pod-info-to-env.md)
  * [无状态应用deployment部署文件](origin/k8s-deployment.md)
* 原理
  * [kubernetes容器的访问方式](origin/kubernetes-容器的访问方式.md)
  * [kubernetes的容器网络CNI](origin/kubernetes的容器网络.md)
  * [kube-proxy的实现方式之iptables与ipvs模式](origin/kubernetes-kube-proxy-iptables-ipvs.md)
* 系统应用
  * 网络CNI
    * Traefik
      * [部署及操作](origin/k8s-cni-traefik-common-operation.md)
      * [todo-多版本Traefix替换更新方案]()
* 安装
  * [Kubeadm安装单机版Kubernetes](origin/kubernetes-使用Kubeadm安装单机版Kubernetes.md)
  * [Kubeasz二进制安装Kubernetes集群](origin/kubernetes-Kubeasz二进制安装集群.md)
  * [新版Kubeasz二进制安装Kubernetes集群](origin/new-kubeasz-install-bin-k8s.md)
  * [Sealos安装](origin/k8s-install-sealos.md)
* 集群管理
  * [kubernetes集群性能监控](origin/prometheus-Kubernetes或Openshift的Prometheus监控体系.md)
  * [kubernetes集群组件](origin/kubernetes-集群组件.md)
  * [kubectl](origin/kubernetes-kubectl.md)
  * [Network Policy容器流量管理](origin/kubernetes-NetworkPolicy.md)
  * [k8s集群的压力测试](origin/k8s-pressure-test.md)
  * [TODO：k8s集群节点自动伸缩容](origin/k8s-cluster-node-autoscaler.md)
  * [HPA：POD自动伸缩容](origin/k8s-pod-autoscaler.md)
* [用户认证ServiceAccount与授权策略RBAC](origin/kubernetes-serviceaccount-rbac.md)
* [K8S应用管理工具Helm](origin/kubernetes-helm.md)
  * [helm charts编写规则](origin/Kubernetes-helm-charts编写规则.md)
* 问题
  * [Service与SpringBoot应用启动参数冲突的问题排查及解决方案](origin/Service与SpringBoot应用启动参数冲突的问题排查及解决方案.md)
  * [K8S上redis主从哨兵模式问题的解决方案](origin/k8s-redis-sentinel.md)
* [辅助工具](origin/kubernetes-tools.md)
* [Doing：K8S 多集群管理与网络互联](origin/k8s-multi-clusters.md)
* [VM On K8S](origin/vm-on-k8s.md)

## Part Ⅲ：持续集成与持续部署

* [CICD优化总结](origin/cicd-summary.md)
* Jenkins
    * [Jenkins API](origin/jenkins-api.md)
    * 管理
        * [Jenkins共享库Shared Libraries](origin/jenkins-SharedLibraries.md)
        * [Jenkins 命令行接口](origin/jenkins-cli-command.md)
    * Pipeline
        * [声明式Declarative语法](origin/jenkins-声明式Declarative-pipeline语法.md)
    * 插件
        * [Kubernetes Plugin](origin/Jenkins-在Kubernetes上使用Kubernetes插件动态创建Slave节点.md)
        * [Pipeline Utility Steps](origin/jenkins-pipeline-utility-steps.md)
        * [Nexus Platform Plugin](origin/jenkins-Nexus-Platform的使用.md)
        * [Mail Plugin](origin/jenkins-配置SMTP邮箱服务.md)
        * [Mail Extension](origin/jenkins-Mailer邮箱功能扩展插件Email-Extension.md)
        * [Gitlab](origin/jenkins-gitlab插件的使用.md)
        * [Generic Webhook Trigger](origin/jenkins-generic-webhook-trigger插件.md)
* Gitlab
    * [部署与配置](origin/gitlab-install.md)
    * 管理
        * [配置SMTP邮件服务](origin/gitlab-配置SMTP邮件服务.md)
        * [代码仓库的备份与恢复](origin/gitlab-backuprestore.md)
        * [Gitlab版本升级](origin/gitlab-upgrade.md)
        * [Gitlab的制品仓库](origin/gitlab-package-registry.md)
        * [Gitlab的Webhook](origin/gitlab-配置代码仓库事件触发器Webhook.md)
        * [Gitlab的服务端git hook](origin/gitlab-server-hook.md)
    * Gitlab CI/CD
        * [Gitlab Runner](origin/gitlab-runner.md)
        * [Gitlab Pipeline](origin/gitlab-pipeline.md)
        * [流水线：MR触发代码扫描+MR审批+合并触发CICD](origin/gitlab-cicd-mr-scan-mr-approval.md)
        * [解耦代码仓库路径下的容器化和k8s部署资源等非业务代码文件](origin/gitlab-kickout-apache-wget.md)
    * [Gitlab Restful API](origin/gitlab-api.md)
* Drone
    * [部署与配置](origin/drone-install-basics.md)
* [Nexus](origin/nexus-简介.md)
    * 配置
        * [使用OrientDB Console在DB层面修改配置](origin/nexus-使用OrientDB Console在DB层面修改配置.md)
        * [设置SMTP邮件服务](origin/nexus-设置SMTP邮件服务.md)
    * 权限
    * 仓库管理
        * [Maven](origin/nexus-maven仓库的配置与使用.md)
        * [NPM](origin/nexus-npm仓库的配置与使用.md)
        * [YUM](origin/nexus-yum仓库的配置与使用.md)
        * [Composer](origin/nexus-composer.md)
        * [Pypi](origin/nexus-pypi.md)
        * [Helm](origin/nexus-helm.md)
    * [数据备份恢复](origin/nexus-数据的备份恢复.md)
    * [API](/origin/nexus-api.md)
    * [Jenkins相关插件](origin/nexus-使用jenkins插件上传CI流程制品到Nexus仓库.md)
* [SonarQube静态代码扫描分析](origin/SonarQube静态代码扫描分析简介.md)
    * [SonarScanner-将扫描结果以comment的形式回写到gitlab](origin/sonarscanner-将扫描结果以comment的形式回写到gitlab.md)
    * [Sonarqube使用Gitlab登录](origin/sonarqube-gitlab-auth.md)
    * [IDE本地插件扫描检查](origin/sonarqube-ide-scanner.md)
* LDAP
    * [OpenLDAP安装部署-查询-管理](origin/openldap-basic.md)
    * 管理客户端
        * [go-ldap-admin](origin/go-ldap-admin.md)
    * 第三方系统集成
        * [Jenkins](origin/ldap-Jenkins对接LDAP.md)
        * [SonarQube](origin/ldap-SonarQube对接LDAP.md)
        * [Gitlab](origin/ldap-Gitlab对接LDAP.md)
        * [Nexus](origin/ldap-Nexus对接LDAP.md)
        * [Grafana](origin/ldap-Grafana对接LDAP.md)
        * [Jira](origin/ldap-jira接LDAP.md)
* Apollo
    * [Apollo简介部署与配置](origin/apollo-deplopy-setup.md)
    * [应用接入Apollo](origin/apollo-application.md)
* 项目管理工具
    * Jira
        * [Jira的部署](origin/jira-部署.md)
    * Redmine
        * [redmine的备份与恢复](origin/redmine-backuprestore.md)
* [Harbor](origin/harbor.md)
* [Vault](origin/vault.md)
* Alfred
    * [Alfred Workflow](origin/alfred.md)
* [Web IDE: VSCode](origin/vscode-on-k8s.md)
* [DolphinScheduler](origin/dolphinscheduler.md)


## Part Ⅳ：日志/监控/告警
* Logging
    * [日志系统数据在个组件中的流转格式](origin/logging-日志系统数据在个组件中的流转格式.md)
    * [ELK系列安装部署](origin/elk-install.md)
    * Kafka/Zookeeper
        * [原理](origin/kafka-origin.md)
        * [基础知识](origin/logging-kafka基础知识.md)
        * [kafka常用操作](origin/logging-kafka常用操作.md)
        * [Zookeeper常用操作](origin/zookeeper.md)
        * [kafka连接调试脚本](origin/kafka-client-connect-procedure-tools.md)
        * [Zookeeper和kafka的WebUI工具](origin/zk-kafka-ui.md)
        * [命令行Kafka工具Kaf](origin/kafka-shell-kaf.md)
    * Filebeat
        * [filebeat简介安装配置](origin/filebeat-简介安装配置.md)
        * [filebeat常用配置以及问题总结](origin/filebeat-summary.md)
        * [filebeat多实例安装部署](origin/filebeat-多实例部署.md)
        * [Filebeat Modules](origin/filebeat-modules模块.md)
            * [Nginx Module](origin/filbeat-nginx-module.md)
    * Logstash
        * [Logastash简介安装配置Pipeline](origin/logstash-简介安装配置Pipeline.md)
        * **Logastash Pipeline示例**
          * [Logastash采集MySQL慢查询日志到Elasticsearch](origin/logstash-采集MySQL慢查询日志到Elasticsearch.md)
        * Filter
          * [grok插件](origin/logstash-filter-grok.md)
        * [Logastash常用filter实现的功能](origin/logstash-filter-summary.md)
    * Elasticsearch
        * [基础知识](origin/elasticsearch-基础知识.md)
            * API Endpoints
                * [_cat](origin/elasticsearch--_cat-API.md)
                * [index](origin/elasticsearch-index-api.md)
                * [bulk](origin/elasticsearch-bulk-api.md)
            * [Ingest节点的Pipeline](origin/elasticsearch-ingest节点.md)
            * [数据的路由分配](origin/elasticsearch-数据的分配路由.md)
            * [es进程池](origin/es-thread-pool.md)
            * [elasticsearch性能测试](origin/elasticsearch-benchmarks.md)
            * [es template模板](origin/es-template.md)
            * [插件：中文分词器ik](origin/es-plugin-chinese-ik.md)
        * 管理
            * [Xpack](origin/elasticsearch-7.1的xpack权限控制.md)
            * [Snapshots](origin/elasticSearch-索引的快照备份与恢复.md)
            * [插件管理](origin/elasticsearch-插件管理.md)
            * [ES与Kibana的TLS/SSL/Xpack权限控制](origin/elasticsearch-ssl-tls.md)
            * [索引快照清理策略](origin/elasticsearch-index-clean-snapshots.md)
            * [官方示例数据集](origin/elasticsearch-sample-data.md)
            * [集群升级](origin/elasticsearch-cluster-upgrade.md)
        * 性能
            * [优化](origin/elasticsearch-optimizing.md)
        * [问题总结](origin/elasticsearch-问题总结.md)
* Metrics
    * [Kubernetes的监控体系](origin/kubernete-prometheus.md)
    * Prometheus
        * [Prometheus基础概念及PromSQL](origin/prometheus-basic.md)
        * [kube-prometheus: Prometheus Operator安装部署](origin/kube-prometheus.md)
        * [二进制Docker部署Prometheus生态](origin/prometheus-binary-docker-deploy.md)
        * [Ceph Exporter](origin/prometheus-Ceph-Exporter对接Prometheus以监控ceph集群.md)
        * [Blackbox Exporter](origin/prometheus-blackbox-exporter.md)
        * [收集Nginx内置Metrics](origin/prometheus-nginx-exporter.md)
    * Grafana
        * [Grafana的备份恢复](origin/grafana-backup-restore.md)
        * [Grafana管理](origin/grafana-manage.md)
* Tracing
    * [Jaeger](origin/jaeger.md)
    * Matomo
* [Sentry日志聚合告警平台](origin/sentry.md)
    * [Logstash与Sentry对接](origin/sentry-logstash对接Sentry.md)

## Part V：基础
* Docker
    * 基础知识
      * [Docker原理](origin/docker原理.md)
      * [容器中的进程管理](origin/docker-process-manager.md)
      * [容器中的用户管理](origin/docker-user-process-manage.md)
      * [Docker常见操作](origin/docker-summary.md)
    * Dockerfile
      * [Dockerfile中CMD与ENTRYPOINT命令的区别](origin/docker-Dockerfile中CMD与ENTRYPOINT命令的区别.md)
      * [使用Makefile操作Dockerfile.md](origin/docker-使用Makefile操作Dockerfile.md)
      * [多阶段构建](origin/docker-multi-stage-build.md)
      * [Alpine镜像](origin/docker-alpine.md)
      * [语法扫描工具Hadolint](origin/dockerfile-hadolint.md)
      * [Dockerfile优化](origin/dockerfile-optimization.md)
* Shell脚本
    * [Shell脚本](origin/shell-scripts-summary.md)
    * [常用bash脚本功能](origin/bash-scirpts.md)  
    * [Shell脚本优化](origin/shell-script-perf.md)
* [osascript脚本](origin/apple-osascript.md)
* Maven
    * Maven POM项目对象模型
    * [Mave Settings文件详解](origin/maven-Settings配置文件详解.md)
    * [Maven 生命周期阶段](origin/maven-生命周期阶段.md)
    * Maven 多模块构建
    * [Maven常见操作](origin/maven-operations.md)
* Git
    * [git原理](origin/git-原理.md)
    * [git常用操作](origin/git-常用操作.md)
    * [git .gitignore文件](origin/git-gitignore文件.md)
    * [git merge与git rebase](origin/origin-git-merge-git-rebase.md)
    * [git submodule](origin/git-submodule.md)
    * [git workflow工作流](origin/git-workflows.md)
    * [git hook钩子](origin/git-hooks.md)
    * [git提交规范](origin/git-standard-commit-message.md)
* [正则表达式](origin/regular-expression详解.md)
* SSL/TLS
  - [SSL/TLS基础概念](origin/ssl-tls.md)
    - [证书生成工具](origin/ssl-tls-tools.md)
* Ceph
    * 安装
        * [Ceph RBD单节点安装](origin/ceph-rbd单节点安装.md)
        * [Ceph FileSystem单节点安装](origin/ceph-filesystem单节点安装.md)
    * 基础知识
* 性能压力测试
    * [测试概念指标](origin/test-concept-indicator.md)
    * 工具
        * [wrk](origin/wrk.md)
        * [sysbench](origin/benchmark-tools-sysbench.md)
    * 接口测试
        * [Jmeter](origin/jmeter.md)
        * [Apache Benchmark](origin/apache-benchmark.md)
    * 中间件性能测试
        * MySQL
            * [MySQL性能测试之sysbench](origin/sysbench-mysql.md)
        * Redis
            * [Redis性能测试之redis-benchmark](origin/redis-benchmark.md)
        * Kafka
            * [Kafka性能测试](origin/kafka-perf.md)
* PXE+Kickstart/AutoInstall
    * [PXE网络无人值守自动安装OS](origin/pxe-kickstart无人值守部署OS.md)
    * [PXE引导配置文件详解](origin/pxe-引导配置文件参数详解.md)
    * 无人值守自动化配置安装
      * [Rocky/CentOS/Readhat: Kickstart详解](origin/pxe-kickstart文件参数详解.md)
      * [Ubuntu/Debian: AutoInstall详解](origin/ubuntu-autoinstall.md)
      * [Fedora CoreOS: Ignition详解](origin/coreos-ignition.md)
* [netboot.xyz](origin/netboot.xyz.md)
* Tool

    * [Sublime Text 3](origin/tool-SublimeText.md)
    * [VSCode](origin/vscode.md)
* Windows
    * [CMD发送SMTP邮件](origin/windows-cmd发送SMTP邮件.md)
    * [Windows小技巧](origin/windows-小技巧.md)
    * [Windows Server管理](origin/windows-server.md)
    * [Windows进程守护工具NSSM](origin/windows-nssm.md)
    * [Windows 无人值守部署服务](origin/windows-deployment-service-aik.md)
    * [主机的网络唤醒WOL服务](origin/windows-wakeuponlan.md)
    * [Exe4J+Inno setup: 将Java代码打包成EXE可安装执行软件](origin/windows-java-exe-install.md)
    * [PowerShell](origin/windows-powershell.md)
* MacOS
    * [MacOS小技巧](origin/macos-tips.md)
* Linux
    * [Linux小技巧](origin/linux-小技巧.md)
    * [比较两个文件的不同](origin/linux-diff.md)
    * [NFS与FTP](origin/linux-baseserver.md)
    * [后台启动进程](origin/linux-daemon.md)
    * [文本处理](origin/linux-文本处理.md)
    * [SSH私钥代理ssh-agent](origin/ssh-agent.md)
    * [htpasswd](origin/linux-htpasswd.md)
    * [YAML文本处理工具shyaml](origin/linux-shyaml.md)
    * [JSON文本处理工具jq](origin/linux-jq.md)
    * [yaml文本处理工具yq](origin/yaml-yq.md)
    * [XML文本处理工具xmllint](origin/linux-xmllint.md)
    * [Curl命令详解](origin/linux-curl.md)
    * [rsync命令详解](origin/linux-rsync.md)
    * [LVM原理及使用](origin/linux-lvm.md)
    * [Linux交换分区](origin/linux-交换分区.md)
    * [硬盘工具命令：dd](origin/linux-硬盘读写性能测试.md)
    * [Vim小技巧](origin/vim-小技巧.md)
    * [Yum-RPM包管理](origin/linux-yum.md)
    * [ZSH](origin/linux-zsh.md)
    * [Systemd-进程管理](origin/linux-进程管理工具SystemD.md)
    * [僵尸进程与孤儿进程](origin/linux-zombie-orphaned-process.md)
    * [proc文件详解](origin/linux-proc.md)
    * [7Zip解压缩](origin/7zip.md)

- Linux排错优化
  - 硬件
    - [磁盘I/O监控](origin/linux-io-monitor.md)
  - 系统
    - [系统进程：top](origin/linux-top.md)
    - [ip/ifconfig](origin/linux-ip-ifconfig.md)
    - [ps](origin/linux-ps.md)
  - 网络
    - [iperf](origin/iperf.md)
    - [tcpdump网络抓包](origin/network-tcpdump.md)
    - [nc](origin/nc.md)
    - [mtr](origin/network-mtr.md)
    - [hping](origin/linux-hping.md)
    - [traceroute](origin/network-traceroute.md)
    - [ipvsadm](origin/ipvsadm.md)
- Iptables
  - [iptables详解](origin/linux-iptables.md)
* [MySQL](origin/mysql-basic.md)
    * [常见操作及SQL](origin/mysql-common-operations.md)
    * [用户权限管理](origin/mysql-user-privileges.md)
    * [MySQL SQL Mode : ONLY_FULL_GROUP_BY](origin/mysql-mode-only-full-groupby.md)
    * [Binglog](origin/mysql-binlog.md)
    * 备份与恢复
      * [MySQL的数据备份与恢复](origin/mysql-backup-restore.md)
      * [MysqlDump常用操作](origin/mysql-dumper.md)
      * [Xtrabackup备份](origin/xtrabackup-backup.md)
    * 基础概念
      * [存储过程语法](origin/mysql-procedure-grammar.md)
      * [常用存储过程](origin/mysql-procedure.md)
      * [事物隔离级别](origin/mysql-transaction.md)
      * [临时表](origin/mysql-temporary.md)
    * [MySQL常见问题解决](origin/mysql-problem-solution.md)
* Redis
    * [基础概念](origin/redis-basic.md)
    * [常用操作](origin/reids-common-opreations.md)
    * [安装部署](origin/reids-install-deploy.md)
    * [redis的持久化策略](origin/redis-persistence.md)
    * [数据迁移备份恢复](origin/redis-backup-restore.md)
* 负载均衡与代理
    * [Keepalived](origin/keepalived.md)
    * [LVS](origin/lvs.md)
* [代理服务器](origin/正反向代理服务的区别.md)
    * [透明代理的应用](origin/transparent_proxy.md)
    * [正向代理](origin/常见正向代理服务软件之间的区别.md)
        * Squid
            * [简介安装日志](origin/squid-简介安装.md)
            * [ACL访问权限](origin/squid-acl访问权限控制.md)
* Tengine
    * [Tengine基础](origin/tengine-basic.md)
* Nginx
    * [Nginx安装配置](origin/nginx-install-setup.md)
    * [Nginx常用配置及功能](origin/nginx-config.md)
    * [Nginx日志写入kafka](origin/nginx-log-kafka.md)
    * [Nginx的WAF：Modsecurity](origin/nginx-docker.md)
* [GitBook](origin/gitbook-简介安装配置.md)
* [Telegram机器人](origin/telegram-Bot机器.md)
* OpenVPN

    * [OpenVPN Server](origin/openvpn-server.md)
* iDRAC
    * [iDRAC](origin/idrac.md)
* [vSphere ESXI](origin/vsphere-esxi.md)
    * [ESXI 管理常用命令](origin/vsphere-esxi-commands.md)
    * [VMware Tools](origin/vmware-tools.md)
    * [vCenter](origin/vSphere-vCenter.md)
    * [ESXI使用Synology的ISCSI存储](origin/esxi-synology-iscsi.md)
    * OVF模板
      * [OVF模板详解](origin/vsphere-ovf.md)
      * [VMWare OVF Tools](origin/vmware-ovf-tool.md)
    * SDK/CLI
      * [Go语言CLI: govc](origin/vsphere-govc.md)
* [钉钉机器人](origin/dingding-customrobot.md)
* [经典面试题](origin/经典面试题.md)
* [Aliyun CLI](origin/aliyun-cli.md)
* [音、视频处理工具: ffmpeg](origin/audio-video-fffmpeg.md)
* [图片处理工具：Imagemagick](origin/images-tool-imagemagick.md)
* [PDF处理工具：Ghostscript](origin/ghostscript.md)
* [Nvidia](origin/nvidia.md)
* [Virtualbox 虚拟机管理](origin/virtualbox-manager.md)
* [阿里云产品使用总结](origin/cloud-product-summary.md)
* [RustDesk：可自建远程控制软件](origin/rustdesk.md)
* [Poste：自建邮件服务器](origin/poste-mailserver.md)
* [使用 Jlink构建最小化依赖的 JRE 环境](origin/jlink-jre.md)
* [Aria2](origin/aria2.md)
* [零散知识汇总](origin/others.md)
* [BarkServer通知](origin/notify-bard.md)
* [开源画图平台:Excalidraw](origin/excalidraw.md)


## Part Ⅵ：数据存储处理

- Cloudera
  - [安装部署](origin/cloudera-install.md)
  - [集群管理](origin/cloudera-cluster-manage.md)
  - [性能及高可用测试](origin/cloudera-performance-ha-test.md)
  - 服务操作
    - [HDFS](origin/cloudera-hdfs.md)
- TiDB
  - 安装部署
    - [Ansible二进制部署管理](origin/tidb-ansible.md)
    - [使用TIUP部署TiDB集群](origin/tiup-install-cluster.md)
    - [TiUP升级集群](origin/tiup-upgrade-cluster.md)
  - ELT工具
    - [DM(Data Migration)数据增量全量同步至TiDB](origin/tidb-dm.md)
    - [Dumpling从TiDB/MySQL导出数据](origin/tidb-dumpling-export.md)
    - [Lightning：导入数据到TIDB](origin/tidb-lighting-import.md)
    - [BR：分布式冷备份与恢复](origin/tidb-br-backup-restore.md)
    - [Sync-diff-inspector：数据对比](origin/tidb-sync-diff-inspector.md)
    - [统计邮件通知慢SQL](origin/tidb-slow-sql-statistics.md)
    - [TiCDC: 增量数据同步](origin/tidb-cdc.md)
  - [大数据量的迁移方案对比](origin/bigdata-migrate-operation.md)
  - [TiDB管理](origin/tidb-management.md)
  - [TiDB性能调优](origin/tidb-performance.md)
  - [TiDB TiFlash](origin/tidb-tiflash.md)
  - [TiDB资源限制隔离](origin/tidb-resource-limit.md)
- 开源对象存储
  - [Minio](origin/minio.md)
  - [JuiceFS](origin/juicefs.md)
  - [SeaweedFS](origin/seaweedfs.md)
- Apache Pulsar
  - [基础概念](origin/pulsar-basic.md)
  - [安装部署](origin/pulsar-install.md)
  - [Pulsar的CLI命令](origin/pulsar-cli.md)
  - [Pulsar性能测试](origin/pulsar-perf-test.md)
  - [Pulsar的Kafka协议适配器KoP](origin/pulsar-kafka-kop.md)
  - [Pulsar的WebSocket API](origin/pulsar-websocket-api.md)
  - [Python Pulsar Client ](origin/pulsar-python-client.md)
- Pentaho Data Integration(kettle)
  - [安装部署及基础概念](origin/kettle-basic.md)
- Canal: MySQL Binlog同步工具
  - [简介安装部署](origin/canal-install-basic.md)
- SQLite
  - [SQLCipher](origin/sqlite-cipher.md)
- [NAS数据迁移](origin/nfs-migrate.md)


## Part Ⅶ：Code

- Python学习笔记
  - [环境搭建：安装配置](origin/python-basic.md)
  - [JupyterHub/RStuido](origin/python-workerbench-jupyterhub.md)
  - [Python总结](origin/python-summary.md)
  - [Flask](origin/flask.md)

- Golang学习笔记
  - [基础语法](origin/golang-basic.md)
  - [go的并发](origin/golang-concurrent-programming.md)
  - [Web框架Gin的使用总结](origin/go-gin.md)
  - [net/http的使用总结](origin/go-net-http-summary.md)
  - [常见工具包的使用总结](origin/go-utils.md)
  - [Golang加解密](origin/golang-encryption.md)
  - [embed: 嵌入静态资源文件](origin/golang-embed.md)
  - [X509/PKI证书](origin/golang-x509-certificate.md)
- [HTML/CSS学习笔记](origin/html-css-learn.md)

- JavaScript学习笔记
  - [SSE（Server-Sent Event）](origin/sse.md)
  - [JS/JQuery/Vue3/TS基础知识学习](origin/js-jquery-vue-learn.md)
  - [Vue3基础知识学习](origin/vue3.md)
  - [Vite打包工具](origin/vue-vite.md)
  - [JavaScript常用工具函数](origin/js-kits.md)
  - [前端代码工具集](origin/web-devtools.md)
  - [Element-Plus与Echarts](origin/elementplus-echarts.md)

## Part VIII：Hackintosh

- [OpenCore](origin/hcakintosh-opencore.md)

## Part X：安全

- 服务器安全
  - [Linux安全](origin/linux-security.md)
- 数据安全
  - [MySQL数据安全](origin/mysql-data-security.md)
- CIS互联网安全基准
  - [CIS互联网安全基准-概览](origin/cis-benchmark.md)
  - [CIS的Kubernetes安全基准](origin/cis-k8s-benchmark.md)
- EDR-SIEM安全系统Wazuh
  - [Wazuh基础](origin/wazuh-basic.md)
  - [Wazuh探测、告警、响应](origin/wazuh-monitor-alert.md)
  - [Wazuh主动防御](origin/wazuh-threat-active-defense.md)
  - [Wazuh恶意软件检测：集成Yara](origin/wazuh-malware-detect.md)
- 攻防
  - [攻防基础](origin/att-ck.md)
- [开源Waf：Nginx-Modesecurity](origin/nginx-modsecurity.md)

## Part IX: 硬件

- [PC硬件规格标准](origin/hardwares.md)
- Synology
  - [Synology管理](origin/synology-management.md)
  - [vSphere备份管理Synology ABB](origin/synology-abb-vsphere.md)
  - [群晖Synology的iSCSI](origin/iSCSI-简介配置使用.md)
- [华硕梅林Asuswrt-Merlin](origin/asuswrt-merlin.md)
- [Raspberry Pi树莓派](origin/raspberry-pi.md)
- [Arduino](origin/arduino.md)
  - [Arduino Nano](origin/arduino-nano.md)
  - [Arduino IDE及编程语法](origin/arduino-ide.md)
- [摄像头：视频-音频-云台](origin/camera.md)
- [LED](origin/led.md)
