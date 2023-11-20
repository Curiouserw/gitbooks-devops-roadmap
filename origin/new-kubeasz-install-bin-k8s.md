# 集群节点规划

- **kubeasz**：3.6.2
- **k8s版本**：1.28.1
- **服务器OS**：Ubuntu 22.04.2
- **CRI**:  Containerd 1.6.23
- **CNI**：Cilium 1.13.6
- **kubeproxy模式**：IPVS
- **证书有效期**：100年
- **IngressContoller**：Traefik 2.10.4
- **CSI**：NFS Provisioner、LocalVolume Provisioner
- **开起功能**：
  - 审计日志

- **额外组件**：
  - **日志体系：EFK Stack**
    - Fluentd
    - ElasticSearch
    - Kibana

  - **监控体系：Prometheus Stack**
    - Metrics-server
    - Prometheus
    - Grafana
    - Alertmanager
    - Kube-state-metrics
    - Node-exporter


| 主机名 | 硬件配置  | IP地址          | 服务                                                         |
| ------ | --------- | --------------- | ------------------------------------------------------------ |
| Node1  | 8C16G100G | 192.168.150.206 | kubeasz Ansible<br>K8S Master<br>(etcd、apiserver、controllermanager、scheduler、kubelet、kueb-proxy、docker) |
| Node2  | 8C16G100G | 192.168.150.207 | K8S Master、K8S Worker(kubelet、kube-proxy、docker)、etcd    |
| Node3  | 8C16G100G | 192.168.150.208 | K8S Master、K8S Worker(kubelet、kube-proxy、docker)、etcd    |
| Node4  | 8C16G100G | 192.168.150.209 | K8S Worker(kubelet、kube-proxy、docker)                      |



# 第一步：初始化部署节点

```bash
sed -i '' -e '/192.168.150.206/d' -e '/192.168.150.207/d' -e '/192.168.150.208/d' -e '/192.168.150.209/d' ~/.ssh/known_hosts && \
local_ssh_pub=$(cat ~/.ssh/id_rsa.pub) && \
ssh -o "StrictHostKeyChecking no" root@192.168.150.206 \
  '
    echo "'"$local_ssh_pub"'" >> ~/.ssh/authorized_keys && 
    apt update && 
    apt install -y git nfs-kernel-server && 
    nk8s_node01_ip=$(ip addr show eth0 | grep -oP "inet \K[\d.]+") && 
    IFS='.' read -r ip_part1 ip_part2 ip_part3 ip_part4 <<< "$nk8s_node01_ip" && 
    nodename=nk8s-node01 && 
    ((ip_part4++)) &&
    nk8s_node02_ip="$ip_part1.$ip_part2.$ip_part3.$ip_part4" &&
    ((ip_part4++)) &&
    nk8s_node03_ip="$ip_part1.$ip_part2.$ip_part3.$ip_part4" &&
    ((ip_part4++)) &&
    nk8s_node04_ip="$ip_part1.$ip_part2.$ip_part3.$ip_part4" &&
    hostnamectl --static set-hostname $nodename && 
    ssh-keygen -t rsa -b 4096 -N "" -f ~/.ssh/id_rsa && 
    cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys && 
    sed -i "/$nk8s_node01_ip/d" /etc/hosts && 
    echo -e "\n$nk8s_node01_ip nk8s-node01\n$nk8s_node02_ip nk8s-node02\n$nk8s_node03_ip nk8s-node03\n$nk8s_node04_ip nk8s-node04\n" >> /etc/hosts && 
    release=3.6.2 && 
    wget -e use_proxy=yes -e https_proxy=http://home.curiouser.top:8001 https://github.com/easzlab/kubeasz/releases/download/$release/ezdown && 
    chmod +x ./ezdown && 
    mv ezdown /usr/local/bin && 
    ezdown -D && 
    ezdown -S && 
    ezdown -X cilium && 
    ezdown -X prometheus && 
    ezdown -X nfs-provisioner && 
    ezdown -X local-path-provisioner && 
    git config --global user.email "test@example.com" && 
    git config --global user.name "test" && 
    rm -rf /etc/kubeasz/.github && 
    sed -i "/clusters\//d" /etc/kubeasz/.gitignore && 
    cd /etc/kubeasz && 
    git init /etc/kubeasz && 
    git add /etc/kubeasz && 
    git commit -am "init commit" && 
    docker exec -i kubeasz ezctl new new-k8s && 
    cd /etc/kubeasz && 
    git add /etc/kubeasz && 
    git commit -am "add new-k8s cluster resource" && 
    echo  -e "alias dk='"'docker exec -it kubeasz'"'" >> ~/.bashrc && 
    systemctl enable nfs-server.service && \
    systemctl start nfs-server.service && \
    echo "/data/nfs/k8s-storage 192.168.150.0/24(rw,insecure,no_subtree_check,no_root_squash,sync)" >> /etc/exports && \
    mkdir -p /data/nfs/k8s-storage && \
    exportfs -a && \
    showmount -e $HOSTNAME
    reboot now
  '
```

# 第二步：设置其他节点基础设置

```bash
local_ssh_pub=$(cat ~/.ssh/id_rsa.pub)
nk8s_node01_ssh_pub=$(ssh root@192.168.150.206 'cat .ssh/id_rsa.pub')
k8s_nodes_ips=("192.168.150.207" "192.168.150.208" "192.168.150.209")
for ((i=1; i<=${#k8s_nodes_ips[@]}; i++)); do
 ssh -o "StrictHostKeyChecking no" root@${k8s_nodes_ips[$i]} \
 '
   echo "'"$local_ssh_pub"'" >> ~/.ssh/authorized_keys &&
   echo "'"$nk8s_node01_ssh_pub"'" >> ~/.ssh/authorized_keys &&
   apt update && 
   apt install -y git && 
   nodename="nk8s-node'"$(printf "%02d" $((i+1)))"'" && 
   sed -i "/'"${k8s_nodes_ips[$i]}"'/d" /etc/hosts && 
   hostnamectl --static set-hostname $nodename && 
   reboot now
 '
done

ansible --user root -i "192.168.150.206,192.168.150.207,192.168.150.208,192.168.150.209" all -m shell -a "hostname"
```

# 第三步：修改kubeasz部署配置文件

```bash
scp -r root@192.168.150.206:/etc/kubeasz/clusters/new-k8s ~/Desktop
```

> ~/Desktop/new-k8s/config.yml

```yaml

```

> ~/Desktop/new-k8s/hosts

```ini

```

修改后同步到 Tool节点

```bash
rsync --exclude='.DS_Store' -r ~/Desktop/new-k8s/ root@192.168.150.206:/etc/kubeasz/clusters/new-k8s/
```

# 第四步：修改集群部署配置

- 修改集群 SSL/TLS证书中组织的名字
- 新增随机生成 Grafana admin用户的密码，修改部署时安装图标插件，新增ServiceMonitor的标签
- 新增部署 Prometheus时挂载持久化PVC
- 修改 NFS Provisioner部署时的PVC 回收策略
- 开起 k8s 审计日志功能

```bash
random_grafana_admin_passw=`specialchars='!@#$%^&*()_+' && \
astring=$(echo $(date +%s)$RANDOM | md5sum | base64 | tr -dc A-Za-z0-9 | head -c 15 ) && \
randomchar=${specialchars:$RANDOM % ${#specialchars}:1} && \
randompos=$(( $RANDOM % ( ${#astring} + 1 ) ))  && \
echo ${astring:0:$randompos}${randomchar}${astring:$randompos}` && \
ssh root@192.168.150.206 ' \
  for i in `grep -rl "XS" /etc/kubeasz/roles/deploy/templates`;do sed -i "s/XS/ZZMED/g" $i ;done && \
  sed -i -e "s#adminPassword: Admin1234\!#adminPassword: '$random_grafana_admin_passw'#g" -e "/skipTlsVerify:\ true/a \ \ plugins:\n\ \ \ \ -\ grafana-piechart-panel\n\ \ \ \ -\ grafana-clock-panel\n\ \ persistence:\n\ \ \ \ enabled: true\n\ \ serviceMonitor:\n\ \ \ \ labels:\n\ \ \ \ \ \ release: prometheus" -e "s/storageSpec: {}/storageSpec:\n      volumeClaimTemplate:\n        spec:\n          storageClassName: \"local-nfs-storage\"\n          accessModes: [\"ReadWriteOnce\"]\n          resources:\n            requests:\n              storage: 50Gi\n        selector:\n          matchLabels:\n            app: prometheus/g" /etc/kubeasz/roles/cluster-addon/templates/prometheus/values.yaml.j2 && \
  sed -i "s/archiveOnDelete: \"false\"/archiveOnDelete: \"true\"/g" /etc/kubeasz/roles/cluster-addon/templates/nfs-provisioner/nfs-provisioner.yaml.j2 && \
  sed -i -e "s/--v=2/--v=2 \\\\\n  --audit-log-path=\/var\/log\/audit\/audit.log \\\\\n  --audit-policy-file=\/etc\/kubernetes\/audit-policy.yaml \\\\\n  --audit-log-maxage=5 \\\\\n  --audit-log-compress \\\\\n  --audit-log-maxbackup=5 \\\\\n  --audit-log-maxsize=200/g" /etc/kubeasz/roles/kube-master/templates/kube-apiserver.service.j2 && \
  sed -i "/\/etc\/kubernetes\/{{ item }}/ {N;N;N; s/\(.*\)/\1\n  - audit-policy.yaml/}" /etc/kubeasz/roles/kube-master/tasks/main.yml  && \
  cat > /etc/kubeasz/clusters/new-k8s/audit-policy.yaml << EOF
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
  - level: None
    users: ["system:kube-proxy"]
    verbs: ["watch"]
    resources:
      - group: ""
        resources: ["endpoints", "services", "services/status"]
  - level: None
    users: ["system:unsecured"]
    namespaces: ["kube-system"]
    verbs: ["get"]
    resources:
      - group: ""
        resources: ["configmaps"]
  - level: None
    users: ["kubelet"]
    verbs: ["get"]
    resources:
      - group: ""
        resources: ["nodes", "nodes/status"]
  - level: None
    userGroups: ["system:nodes"]
    verbs: ["get"]
    resources:
      - group: ""
        resources: ["nodes", "nodes/status"]
  - level: None
    users:
      - system:kube-controller-manager
      - system:cloud-controller-manager
      - system:kube-scheduler
      - system:serviceaccount:kube-system:endpoint-controller
    verbs: ["get", "update"]
    namespaces: ["kube-system"]
    resources:
      - group: ""
        resources: ["endpoints"]
  - level: None
    users: ["system:apiserver"]
    verbs: ["get"]
    resources:
      - group: ""
        resources: ["namespaces", "namespaces/status", "namespaces/finalize"]
  - level: None
    users: ["cluster-autoscaler"]
    verbs: ["get", "update"]
    namespaces: ["kube-system"]
    resources:
      - group: ""
        resources: ["configmaps", "endpoints"]
  - level: None
    users:
      - system:kube-controller-manager
      - system:cloud-controller-manager
    verbs: ["get", "list"]
    resources:
      - group: "metrics.k8s.io"
  - level: None
    nonResourceURLs:
      - /healthz*
      - /version
      - /swagger*
  - level: None
    resources:
      - group: ""
        resources: ["events"]
  - level: Request
    users: ["kubelet", "system:node-problem-detector", "system:serviceaccount:kube-system:node-problem-detector"]
    verbs: ["update","patch"]
    resources:
      - group: ""
        resources: ["nodes/status", "pods/status"]
    omitStages:
      - "RequestReceived"
  - level: Request
    userGroups: ["system:nodes"]
    verbs: ["update","patch"]
    resources:
      - group: ""
        resources: ["nodes/status", "pods/status"]
    omitStages:
      - "RequestReceived"
  - level: Request
    users: ["system:serviceaccount:kube-system:namespace-controller"]
    verbs: ["deletecollection"]
    omitStages:
      - "RequestReceived"
  - level: Metadata
    resources:
      - group: ""
        resources: ["secrets", "configmaps", "serviceaccounts/token"]
      - group: authentication.k8s.io
        resources: ["tokenreviews"]
    omitStages:
      - "RequestReceived"
  - level: Request
    verbs: ["get", "list", "watch"]
    omitStages:
      - "RequestReceived"
    resources:
      - group: ""
      - group: "admissionregistration.k8s.io"
      - group: "apiextensions.k8s.io"
      - group: "apiregistration.k8s.io"
      - group: "apps"
      - group: "authentication.k8s.io"
      - group: "authorization.k8s.io"
      - group: "autoscaling"
      - group: "batch"
      - group: "certificates.k8s.io"
      - group: "extensions"
      - group: "metrics.k8s.io"
      - group: "networking.k8s.io"
      - group: "node.k8s.io"
      - group: "policy"
      - group: "rbac.authorization.k8s.io"
      - group: "scheduling.k8s.io"
      - group: "storage.k8s.io"
  - level: RequestResponse
    omitStages:
      - "RequestReceived"
    resources:
      - group: ""
      - group: "admissionregistration.k8s.io"
      - group: "apiextensions.k8s.io"
      - group: "apiregistration.k8s.io"
      - group: "apps"
      - group: "authentication.k8s.io"
      - group: "authorization.k8s.io"
      - group: "autoscaling"
      - group: "batch"
      - group: "certificates.k8s.io"
      - group: "extensions"
      - group: "metrics.k8s.io"
      - group: "networking.k8s.io"
      - group: "node.k8s.io"
      - group: "policy"
      - group: "rbac.authorization.k8s.io"
      - group: "scheduling.k8s.io"
      - group: "storage.k8s.io"
  - level: Metadata
    omitStages:
      - "RequestReceived"
EOF
'
```

# 第五步：开始部署 K8s 集群

```bash
ssh root@192.168.150.206 'docker exec -i kubeasz ezctl setup new-k8s all'
```

# 第六步：验证k8s

```bash
kubectl config delete-cluster zzmed-new-k8s-cluster ; \
kubectl config delete-user zzmed-new-k8s-cluster-admin ; \
kubectl config delete-context zzmed-new-k8s-cluster ; \
kubectl config set-cluster zzmed-new-k8s-cluster --server=https://192.168.150.206:6443 && \
kubectl config set-credentials zzmed-new-k8s-cluster-admin && \
ssh root@192.168.150.206 'cat .kube/config' | yq -r '.clusters.[0].cluster.certificate-authority-data ' | xargs kubectl config set clusters.zzmed-new-k8s-cluster.certificate-authority-data && \
ssh root@192.168.150.206 'cat .kube/config' | yq -r '.users.[0].user.client-certificate-data' | xargs kubectl config set users.zzmed-new-k8s-cluster-admin.client-certificate-data && \
ssh root@192.168.150.206 'cat .kube/config' | yq -r '.users.[0].user.client-key-data' | xargs kubectl config set users.zzmed-new-k8s-cluster-admin.client-key-data && \
kubectl config set-context zzmed-new-k8s-cluster --cluster=zzmed-new-k8s-cluster --user=zzmed-new-k8s-cluster-admin --namespace=kube-system && \
kubectl config use-context zzmed-new-k8s-cluster && \
kubectl -n kube-public run nginx --image=nginx --port=80 --labels="app=nginx" && \
kubectl -n kube-public expose pod nginx --name=nginx --port=80 --target-port=80 --protocol=TCP && \
kubectl get nodes && \
kubectl get ns && \
kubectl get pod -A && \
kubectl -n kube-public get pod -l app=nginx -o wide && \
kubectl -n kube-public get svc -l app=nginx 
```

Prometheus：http://192.168.150.206:30901

Alertmanager：http://192.168.150.206:30902/

Grafana：http://192.168.150.206:30903

# (可选)第七步：其他操作

```bash
ssh root@192.168.150.206 \
  '
    apt update && 
    apt install -y zsh && 
    chsh -s /bin/zsh root && 
    curl -x http://home.curiouser.top:8001 -sLo install.sh https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh &&
    chmod +x install.sh && 
    REPO="mirrors/oh-my-zsh" REMOTE="https://gitee.com/mirrors/oh-my-zsh.git" sh install.sh --skip-chsh && 
    git config --global https.proxy http://home.curiouser.top:8001 && 
    git config --global https.proxy https://home.curiouser.top:8001 && 
    git clone https://github.com/zsh-users/zsh-autosuggestions.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions && 
    git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting && 
    sed -i -e "/ZSH_THEME/d" -e "10 a ZSH_THEME=\"ys\"" -e "s/plugins=(git)/plugins=(git kubectl docker sudo history extract zsh-autosuggestions zsh-syntax-highlighting )/g" ~/.zshrc && 
    echo -e "alias dk='"'docker exec -it kubeasz'"'\nexport PATH=/opt/kube/bin:/opt/kube/bin/containerd-bin/:\$PATH\nsource <(crictl completion zsh)\nsource <(kubectl completion zsh)" >> ~/.zshrc
  '
```

# 第八步：安装配置其他系统

## 1、安装 Traefik

参照：[Traefik的部署及操作](k8s-cni-traefik-common-operation.md)

## 2、配置应用暴露metrics Prometheus Service Monitor

```bash
# 创建 Nginx 配置文件 configmap
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-stubstatus-config
  namespace: kube-public
data:
  nginx_stub_status.conf: |-
    server {
      listen       80;
      server_name  localhost;
      location / {
          root   /usr/share/nginx/html;
          index  index.html index.htm;
      }
    }
    server {
      listen 8080;
      location /nginx_status {
        stub_status on;
      }
    }
EOF

# 创建 Nginx 测试部署资源
kubectl -n kube-public create deployment nginx --image=nginx --port=80 && \
# 创建 Nginx 的 SVC。暴露 pod 的 80、8080 端口
kubectl -n kube-public expose deployment nginx --name=nginx --port=80 --target-port=80 --protocol=TCP && \
kubectl -n kube-public expose deployment nginx --name=nginx-stubstatus --port=8080 --target-port=8080 --protocol=TCP && \
# 修改 Nginx 部署资源挂载配置文件
kubectl -n kube-public patch deployment nginx -p \
'{"spec":{"template":{"spec":{"volumes":[{"name":"nginx-stubstatus-config","configMap":{"name":"nginx-stubstatus-config"}}],"containers":[{"name":"nginx","ports":[{"containerPort": 8080,"protocol":"TCP","name":"metrics"},{"containerPort": 80,"protocol":"TCP","name":"app"}],"volumeMounts":[{"name":"nginx-stubstatus-config","mountPath":"/etc/nginx/conf.d"}]}]}}}}' && \
# 创建 Nginx exporter部署资源
kubectl -n kube-public create deployment nginx-exporter \
  --image=nginx/nginx-prometheus-exporter:0.10.0 \
  --port=9113 \
  -- nginx-prometheus-exporter "-nginx.scrape-uri=http://nginx-stubstatus.kube-public.svc:8080/nginx_status" && \
# 创建 Nginx exporter的SVC。暴露pod 9113端口
kubectl -n kube-public expose deployment nginx-exporter --port=9113 --target-port=9113 --protocol=TCP  --labels=prometheus-target=app-metrics && \
# 修改 Nginx exporter SVC的标签
kubectl -n kube-public patch service nginx-exporter -n kube-public --type='json' -p='[{"op": "add", "path": "/spec/ports/0/name", "value": "app-metrics"}]' && \

cat <<EOF | kubectl apply -f -
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: application-metrics
  namespace: monitor
  labels:
    release: prometheus
spec:
  endpoints:
  - interval: 15s
    scheme: http
    port: app-metrics
  namespaceSelector:
    matchNames:
    - kube-public
  selector:
    matchLabels:
      prometheus-target: app-metrics
EOF
```

## 3、安装 EFK

参照：[ELK系列安装部署 第二章节](elk-install.md)

### 3.3 使用Fluent Operator部署 Fluentd

文档：https://docs.fluentbit.io/manual/

Helm Charts：https://github.com/fluent/helm-charts

#### ①使用Helm安装Fluent Operator

fluent-operator

```bash
helm repo add fluent https://fluent.github.io/helm-charts && \
helm search repo fluent

helm upgrade --install --atomic \
  -n kube-system \
  fluent-operator fluent/fluent-operator \
  --set containerRuntime=containerd \
  --set fluentbit.enable=false \
  --set fluentbit.crdsEnable=false \
  --set fluentbit.output.es.enable=true \
  --set fluentbit.output.es.host="logging-es-http.logging.svc"
```

fluent-bit

```bash
helm upgrade --install  \
  -n logging \
  fluent-bit fluent/fluent-bit \
  --set testFramework.enabled=false
```



```bash
kubectl delete cm fluent-bit && \

cat <<EOF | kubectl create -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluent-bit
  namespace: logging
data:
  custom_parsers.conf: |
    [PARSER]
        Name docker_no_time
        Format json
        Time_Keep Off
        Time_Key time
        Time_Format %Y-%m-%dT%H:%M:%S.%L
  fluent-bit.conf: |
    [SERVICE]
        Daemon                 Off
        Flush                  1
        Log_Level              info
        Parsers_File           /fluent-bit/etc/parsers.conf
        Parsers_File           /fluent-bit/etc/conf/custom_parsers.conf
        HTTP_Server            On
        HTTP_Listen            0.0.0.0
        HTTP_Port              2020
        Health_Check           On

    [INPUT]
        Name                   tail
        Path                   /var/log/containers/*.log
        multiline.parser       docker, cri
        Tag                    kube_containers.*
        Mem_Buf_Limit          5MB
        Skip_Long_Lines        On
        
    [INPUT]
        Name                   tail
        Tag                    kube_audit.*
        Path                   /var/log/audit/audit.log
        Parser                 json
        DB                     /var/log/flb_kube_audit.db
        Refresh_Interval       10
        
    [INPUT]
        Name                   systemd
        Tag                    kube_node_sysd.*
        Systemd_Filter         _SYSTEMD_UNIT=kubelet.service
        Read_From_Tail         On

    [FILTER]
        Name                   kubernetes
        Match                  kube_containers.*
        Merge_Log              On
        Keep_Log               Off
        K8S-Logging.Parser     On
        K8S-Logging.Exclude    On

    [OUTPUT]
        Name                   es
        Match                  kube_audit.*
        Host                   logging-es-http.logging.svc
        Port                   9200
        compress               gzip
        HTTP_User              elastic
        HTTP_Passwd            ky673H5s3Q3JMSm90N5qAWP0
        tls                    On
        tls.verify             Off
        Logstash_Format        On
        Logstash_Prefix        kube_audit
        Retry_Limit            False

    [OUTPUT]
        Name                   es
        Match                  kube_containers.*
        Host                   logging-es-http.logging.svc
        Port                   9200
        compress               gzip
        HTTP_User              elastic
        HTTP_Passwd            ky673H5s3Q3JMSm90N5qAWP0
        tls                    On
        tls.verify             Off
        Logstash_Format        On
        Logstash_Prefix        kube_containers
        Retry_Limit            False

    [OUTPUT]
        Name                   es
        Match                  kube_node_sysd.*
        Include_Tag_Key        true
        Host                   logging-es-http.logging.svc
        Port                   9200
        compress               gzip
        HTTP_User              elastic
        HTTP_Passwd            ky673H5s3Q3JMSm90N5qAWP0
        tls                    On
        tls.verify             Off
        Logstash_Format        On
        Logstash_Prefix        kube_node_systemd
        Retry_Limit            False
EOF
```

```bash
export POD_NAME=$(kubectl get pods --namespace logging -l "app.kubernetes.io/name=fluent-bit,app.kubernetes.io/instance=fluent-bit" -o jsonpath="{.items[0].metadata.name}")
kubectl --namespace logging port-forward $POD_NAME 2020:2020
curl http://127.0.0.1:2020
```



fluent

```bash
helm upgrade --install  \
  -n logging \
  fluentd fluent/fluentd \
```

```bash
sed -i -e 's/deb.debian.org/mirrors.ustc.edu.cn/g' -e 's/security.debian.org/mirrors.ustc.edu.cn/g' /etc/apt/sources.list && apt update && apt install -y curl
curl -XGET  -s -k -u "elastic:ky673H5s3Q3JMSm90N5qAWP0" "https://logging-es-http.logging.svc:9200/"
```



#### ②安装 Fluentd采集 k8s 容器日志与审计日志

fluentd输出到elasticsearch文档：

- https://github.com/uken/fluent-plugin-elasticsearch#usage
- https://docs.fluentd.org/output/elasticsearch

```bash
```



# 参考

- https://github.com/prometheus-community/helm-charts/tree/main/charts
- https://github.com/prometheus-community/helm-charts/blob/main/charts/prometheus/values.yaml#L870



