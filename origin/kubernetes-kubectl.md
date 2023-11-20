# Kubectl

# 一、下载kubectl

kubectl github下载地址：https://github.com/kubernetes/kubectl/releases

# 二、创建配置文件夹

**Linux**

  ```bash
  mkdir ~/.kube
  ```

**Windows CMD**

  ```bash
mkdir %USERPROFILE%\.kube
# %USERPROFILE%  当前用户目录  
  ```

# 三、创建编辑kubectl配置文件

```yaml
apiVersion: v1
# 集群信息
clusters:
- cluster:
    certificate-authority-data: **CA证书***
    server: https://开发k8s环境APIServer的IP地址:6443
  name: k8s-dev
- cluster:
    certificate-authority-data: **CA证书***
    server: https://测试k8s环境APIServer的IP地址:8443
  name: k8s-test
- cluster:
    certificate-authority-data: **CA证书***
    server: https://UAT k8s环境APIServer的IP地址:8443
  name: k8s-uat
- cluster:
    certificate-authority-data: **CA证书***
    server: https://生产k8s环境APIServer的IP地址:8443
  name: k8s-pro
# 集群上下文环境
contexts:
- context:
    cluster: k8s-dev
    user: k8s-dev-admin
  name: k8s-dev
- context:
    cluster: k8s-test
    user: k8s-test-admin
  name: k8s-test
- context:
    cluster: k8s-uat
    user: k8s-uat-admin
  name: k8s-uat
- context:
    cluster: k8s-pro
    user: k8s-pro-readonly
  name: k8s-pro
# 当前使用的上下文环境  
current-context: k8s-dev
kind: Config
preferences: {}
#集群用户信息及证书信息
users:
- name: k8s-dev
  user:
    client-certificate-data: **用户证书**
    client-key-data： **用户私钥**
- name: k8s-test
  user:
    client-certificate-data: **用户证书**
    client-key-data： **用户私钥**
- name: k8s-uat
  user:
    client-certificate-data: **用户证书**
    client-key-data： **用户私钥**
- name: k8s-pro
  user:
    client-certificate-data: **用户证书**
    client-key-data： **用户私钥**

```

# 四、切换Kubernetes集群上下文

```bash
#切换至开发k8s环境上下文
kubectl config use-context k8s-dev

#切换至开发k8s环境上下文
kubectl config use-context k8s-test

#切换至开发k8s环境上下文
kubectl config use-context k8s-uat

#切换至开发k8s环境上下文
kubectl config use-context k8s-pro
```

# 五、kubectl命令快捷操作

## 1、设置别名快速使用kubectl命令

**Windows**

```bash
doskey  k=kubectl $*
# $*表示这个命令还可能有其他参数
```

**Linux**

```bash
alias k='kubectl'
```

## 2、设置别名快速切换集群上下文

**Windows**

```bash
doskey k2d=kubectl config use-context k8s-dev
doskey k2t=kubectl config use-context k8s-test
doskey k2u=kubectl config use-context k8s-uat
doskey k2p=kubectl config use-context k8s-pro
```

**Linux**

```bash
alias k2d='kubectl config use-context k8s-dev'
alias k2t='kubectl config use-context k8s-test'
alias k2u='kubectl config use-context k8s-uat'
alias k2p='kubectl config use-context k8s-pro'
```

## 3、设置别名永久生效

**Windows**
   - ①创建bat脚本cmdalias.cmd
       ```bash
       @doskey k=kubectl $*
       @doskey k2d=kubectl config use-context k8s-dev
       @doskey k2t=kubectl config use-context k8s-test
       @doskey k2u=kubectl config use-context k8s-uat
       @doskey k2p=kubectl config use-context k8s-pro
       # @表示执行这条命令时不显示这条命令本身
       ```

   - ②修改注册表
     - `方式1`：手动在注册HKEY_CURRENT_USER\Software\Microsoft\Command Processor下添加一项AutoRun，把值设为bat脚本的路径
     - `方式2`：创建编写一个注册表修改文件，名为：add-regkey.reg，双击行这个文件,导入注册表添加的值

            Windows Registry Editor Version 5.00
            [HKEY_CURRENT_USER\Software\Microsoft\Command Processor]
            "AutoRun"="%USERPROFILE%\\.kube\\cmdalias.cmd"  


**Linux**


    echo "alias k='kubectl'" >> /etc/profile && \
    echo "alias k2d='kubectl config use-context k8s-dev'" >> /etc/profile && \
    echo "alias k2t='kubectl config use-context k8s-test'" >> /etc/profile && \
    echo "alias k2u='kubectl config use-context k8s-uat'" >> /etc/profile && \
    echo "alias k2p='kubectl config use-context k8s-pro'" >> /etc/profile && \
    source /etc/profile


# 六、Kubectl Config命令详解

   ```bash
  1. If the --kubeconfig flag is set, then only that file is loaded.  The flag may only be set once and no merging takes
place.
  2. If $KUBECONFIG environment variable is set, then it is used a list of paths (normal path delimitting rules for your
system).  These paths are merged.  When a value is modified, it is modified in the file that defines the stanza.  When a
value is created, it is created in the first file that exists.  If no files in the chain exist, then it creates the last
file in the list.
  3. Otherwise, ${HOME}/.kube/config is used and no merging takes place.

  Usage:
      kubectl config SUBCOMMAND [options]

  Available Commands:
      current-context Displays the current-context
      delete-cluster  Delete the specified cluster from the kubeconfig
      delete-context  Delete the specified context from the kubeconfig
      get-clusters    Display clusters defined in the kubeconfig
      get-contexts    Describe one or many contexts
      rename-context  Renames a context from the kubeconfig file.
      set             Sets an individual value in a kubeconfig file
      set-cluster     Sets a cluster entry in kubeconfig
      set-context     Sets a context entry in kubeconfig
      set-credentials Sets a user entry in kubeconfig
      unset           Unsets an individual value in a kubeconfig file
      use-context     Sets the current-context in a kubeconfig file
      view            Display merged kubeconfig settings or a specified kubeconfig file
   ```

# 七、Kubectl常用操作

## 1、节点维护

```bash
# 1. 设置节点不可被调度
kubectl cordon 节点名
# 2. 驱逐节点上的POD
kubectl drain 节点名 --delete-local-data --ignore-daemonsets --force
  # --delete-local-data  即使pod使用了emptyDir也删除
	# --ignore-daemonsets  忽略deamonset控制器的pod，如果不忽略，deamonset控制器控制的pod被删除后可能马上又在此节点上启动起来,会成为死循环；
	# --force  不加force参数只会删除该NODE上由ReplicationController, ReplicaSet, DaemonSet,StatefulSet or Job创建的Pod，加了后还会删除'裸奔的pod'(没有绑定到任何replication controller)

# 3. 下掉节点
kubectl delete node stg-k8s-node1

# 4. 修复操作

# 5. 设置节点可被调度
kubectl uncordon 节点名
```

## 2、Deployment历史版本管理

```bash
# 查看版本历史版本
kubectl rollout history deployment test
# 查看指定历史版本的详细信息
kubectl rollout history deployment test --revision=1

# 回滚到上一个版本
kubectl rollout undo deployment/test
# 回滚到指定版本
kubectl rollout undo deployment/test --to-revision=2

# 查看Deployment的发布状态
kubectl rollout status deploy/test
```

## 3、直接修改资源对象参数

```bash
kubectl set image deployment/test nginx=nginx:1.15
# 直接更新所有容器镜像
kubectl set image daemonset abc *=nginx:1.9.1
```

## 4、运行测试容器

```bash
# 创建POD
kubectl run nginx --image=nginx --port=80 --labels="app=nginx"
kubectl expose pod nginx --name=nginx --port=80 --target-port=80 --protocol=TCP

# 创建Deployment
kubectl create deployment nginx --image=nginx --port=80 --replicas=1
kubectl expose deployment nginx --port=80 --target-port=80
```

## 5、等待验证k8s资源pod部署状态

```bash
kubectl apply -f k8s-job-application.yaml
kubectl -n $NAMESPACE wait --for=condition=complete --timeout=50s job/$CI_PROJECT_NAME || exit_code=$? | \
if [ $exit_code -ne 0 ];then
      ROLLBACK_ID=$(kubectl -n $NAMESPACE rollout undo deployment/$CI_PROJECT_NAME -ojson | jq -r '.status.observedGeneration') ;
      curl -s https://oapi.dingtalk.com/robot/send?access_token="$PIPELINE_DINGDING_ROBOT_TOKEN" -H 'Content-Type: application/json' -d '{"msgtype": "markdown","markdown": {"title": "Gitlab流水线部署失败","text": "['$CI_PROJECT_NAME-cronjob']('$CI_PROJECT_URL'/-/tree/'$CI_BUILD_REF_NAME')的'$APPENV'环境第['$CI_PIPELINE_ID']('$CI_PIPELINE_URL')号流水线'$CI_JOB_STAGE'阶段失败，已回滚至最近一个稳定版本'$ROLLBACK_ID'，请检查相关错误！"},"at": {"isAtAll": true}}' > /dev/null;
      kubectl -n $NAMESPACE logs -l job=$CI_PROJECT_NAME ;
      kubectl -n $NAMESPACE delete job $CI_PROJECT_NAME ;
  exit 1;
fi
```

## 6、kubectl apply从标准输入中读取资源文件

```yaml
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx
  namespace: kube-public
spec:
  rules:
  - host: "nginx.test.com"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx
            port:
              name: web
EOF
```

## 7、kubectl patch更新k8s资源

Kubectl patch 支持以下 3 种 patch 类型：

- **strategic patch(默认)**：根据不同字段 patchStrategy 决定具体的合并 patch 策略。 Strategic merge patch 并非通用的 RFC 标准，而是 Kubernetes 特有的一种更新 Kubernetes 资源对象的方式。与 JSON merge patch 和 JSON patch 相比，strategic merge patch 更为强大。

  例如：给deployment添加configmap类型的 Volume 声明与挂载

  ```bash
  kubectl patch deployment nginx \
    -p '{"spec":{"template":{"spec":{"volumes":[{"name":"nginx-stubstatus-config","configMap":{"name":"nginx-stubstatus-config"}}],"containers":[{"name":"nginx","volumeMounts":[{"name":"nginx-stubstatus-config","mountPath":"/etc/nginx/conf.d"}]}]}}}}'
  ```

- **json merge patch**：遵循 JSON Merge Patch, RFC 7386[1] 规范，根据 patch 中提供的期望更改的字段及其对应的值，更新到目标中。
- **json patch**：遵循 JSON Patch, RFC 6902[2] 规范，通过明确的指令表示具体的操作。
  
  例如：给Service暴露的端口添加名字
  
   ```bash
   kubectl patch service nginx-exporter -n kube-public \
     --type='json' \
     -p='[{"op": "add", "path": "/spec/ports/0/name", "value": "application-metrics-exporter"}]'
   ```

# 参考连接

1. https://blog.csdn.net/u013360850/article/details/83315188
2. https://www.awaimai.com/2445.html
2. https://stackoverflow.com/questions/44686568/tell-when-job-is-complete
