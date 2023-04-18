# K8S安全之认证与授权策略机制

# 一、简介

官方文档：https://kubernetes.io/docs/reference/access-authn-authz/rbac/

还是得先从kubernetes集群角色说起

- **ETCD**：存储所有k8s资源状态数据
- **API Server**：对外暴露操作ETCD等REST API接口



Kubernetes的API Server有众多的资源**REST API**接口，同时还有众多依赖API Server进行操作的集群组件，例如Controller Manager等。API Server为了保护API请求的合法性。API Serve内部需要先验证请求的权限。需要验证

目前Kubernetes支持的授权策略有：

- **ABAC**：基于属性的访问控制
- **RBAC**：基于角色的访问控制
- **Webhook**：
- **Node**
- **AlwaysDeny**：拒绝所有的请求，一般用于测试。
- **AlwaysAllow**：表示允许所有的请求，不进行认证授权。（默认配置）

从1.6版本起，Kubernetes 默认启用RBAC访问控制策略。从1.8开始，RBAC已成为稳定的功能。API Server启用RABC需要设置启动参数`–-authorization-mode=RBAC`，。







# 二、RBAC API 资源对象

Kubernetes的RBAC认证授权策略使用`rbac.authorization.k8s.io  `  API组



## Role

## ClusterRole

## RoleBinding

## ClusterRoleBind



# 三、应用

## 1、限制客户端用户只能访问或操作指定命名空间的特定资源

| 场景用户 | 用户角色  | 限制Namespace | 限制资源对象   | 语义化的限制动作                               |
| -------- | --------- | ------------- | -------------- | ---------------------------------------------- |
| 开发者   | developer | test          | pod、configmap | 只能查看pod/configmap<br>能登录到pod中进行操作 |

### ①创建RBAC相关的资源声明文件

- ServiceAccount
- ClusterRole
- RoleBinding
- ClusterRoleBinding

```yaml
#=========================ServiceAccount======================
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: developer
  namespace: default
# 或者使用命令：kubectl -n default create sa developer
#=========================ClusterRole======================
--- 
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: developer
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - pods/attach
  - pods/exec
  - pods/log
  - pods/status
  - configmaps
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - services
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - pods/exec
  verbs:
  - create
# Portforward
- apiGroups:
  - ""
  resources:
  - pods
  - pods/portforward
  verbs:
  - get
  - list
  - create

#=========================RoleBinding======================

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer
  namespace: test
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: developer
subjects:
- kind: ServiceAccount
  name: developer
  namespace: default
# 或者使用命令：kubectl create rolebinding developer --clusterrole=developer --serviceaccount=default:developer --namespace=test
#=========================ClusterRoleBinding======================
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: developer
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: developer
subjects:
- kind: ServiceAccount
  name: developer
  namespace: default
```

### ②获取serviceaccount的secret

```bash
k -n default get secrets 
k -n default get secrets developer-token-** -oyaml
```

获得如下secrets的详细内容

```yaml
apiVersion: v1
type: kubernetes.io/service-account-token
kind: Secret
metadata:
  namespace: default
data:
  ca.crt: ******12345678********	# API Server服务端的CA证书
  namespace: ZGVmYXVsdA==					# 该字符串为“default”base64转码后的值
  token: *******ABCDEF********* 	# 该token是经过base64处理的，需要进行解码处理
```

base64解码secret中的Token

```bash
echo "*******ABCDEF*********" | base64 -d
```

### ③组装kubeconfig

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: ******12345678********
    server: https://master.k8s117.curiouser.com:8443
  name: k8s117
contexts:
- context:
    cluster: k8s117
    namespace: test
    user: k8s117-developer
  name: k8s117-developer
current-context: k8s117-developer
kind: Config
preferences: {}
users:
- name: k8s117-developer
  user:
    token: "*******ABCDEF*********base64解码后的值"
```

### ④测试

- 看是否能获取所有的命名空间（不能）
- 看是否能查看test命名空间下的所有POD和ConfigMap（能）
- 看是否能登录到test命名空间下的POD（能）
- 看是否能使用kube-proxy端口转发test命名空间下POD的端口（能）