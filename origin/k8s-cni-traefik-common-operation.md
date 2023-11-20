# Traefik Ingress Controller

# 一、简介

Traefik文档：https://doc.traefik.io/traefik/

Traefik-Helm-Chart Github：https://github.com/traefik/traefik-helm-chart

Traefik 创建路由规则有多种方式：

- 原生 `Ingress`
- 使用 CRD `IngressRoute`
- 使用 `GatewayAPI` 

相较于原生 Ingress 写法，ingressRoute 是 2.1 以后新增功能，简单来说，他们都支持路径 (path) 路由和域名 (host) HTTP 路由，以及 HTTPS 配置，区别在于 IngressRoute 需要定义 CRD 扩展，但是它支持了 TCP、UDP 路由以及中间件等新特性，强烈推荐使用 ingressRoute

# 二、部署安装

开起功能

- 设置 HTTP 服务端口为 80、HTTPS 服务端口 443、TCP 服务端口为 799
- 以 HostPort 方式暴露相关服务端口
- 开起Prometheus Metrics监控端点
- 持久化记录响应状态码为400~599的请求日志
- 创建Dashboard的 IngressRoute

## 1、Helm 部署

```bash
helm upgrade --install --atomic traefik-ingress-controller traefik/traefik \
        --version 24.0.0 \
        --namespace kube-system \
        --set log.general.level=INFO \
        --set securityContext.capabilities.add={NET_BIND_SERVICE} \
        --set securityContext.runAsGroup=0 \
        --set securityContext.runAsUser=0 \
        --set securityContext.runAsNonRoot=false \
        --set podSecurityContext.runAsNonRoot=false \
        --set ports.traefik.hostPort=9000 \
        --set ports.tcp.port=799 \
        --set ports.tcp.hostPort=799 \
        --set ports.web.port=80 \
        --set ports.web.hostPort=80 \
        --set ports.websecure.port=443 \
        --set ports.websecure.hostPort=443 \
        --set ports.metrics.port=9110 \
        --set ports.metrics.exposedPort=9110 \
        --set deployment.replicas=2 \
        --set persistence.enabled=true \
        --set persistence.size=20G \
        --set metrics.prometheus.serviceMonitor.enable=true \
        --set metrics.prometheus.serviceMonitor.additionalLabels.release=prometheus \
        --set persistence.storageClass=local-nfs-storage \
        --set globalArguments="{"--entryPoints.tcp.address=:799/tcp","--entrypoints.websecure.address=:443","--entrypoints.web.address=:80","--entryPoints.metrics.address=:9110","--api.disabledashboardad=false","--global.sendanonymoususage=false","--global.checknewversion=false","--accesslog=true","--accesslog.fields.names.accesslog","--accesslog.fields.headers.defaultmode=keep","--accesslog.filepath=/data/400-599-reponse-json.log","--accesslog.format=json","--accesslog.filters.statuscodes=400-599"}" \
        --set service.type=ClusterIP \
        --set ingressRoute.dashboard.dashboard="Host(\`traefik-dashboard.test.com\`) && PathPrefix(\`/dashboard\`) || PathPrefix(\`/api\`)" \
        --set ingressRoute.dashboard.entryPoints[0]=web \
        --set hostNetwork=true
```

通过域名访问 Traefik Dashboard：http://traefik-dashboard.test.com/dashboard/

## 2、验证测试

### ①创建测试 POD

```bash
kubectl -n kube-public run nginx2 --image=nginx --port=80 --labels="app=nginx2" && \
kubectl -n kube-public expose pod nginx2 --port=80 --target-port=80 --protocol=TCP && \
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx2
  namespace: kube-public
spec:
  rules:
  - host: "nginx2.test.com"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx2
            port:
              number: 80
EOF
```

### ②访问测试

通过命令行访问：

```bash
ingress_controller_node=`kubectl -n kube-system get pod -l app.kubernetes.io/name=traefik -ojson | jq -r '.items[0] | .status.hostIP'` && \
curl -s --noproxy "traefik-dashboard.test.com" --resolve traefik-dashboard.test.com:80:$ingress_controller_node  http://traefik-dashboard.test.com/dashboard/ && \
curl -s --noproxy "nginx.test.com" --resolve nginx.test.com:80:$ingress_controller_node  http://nginx.test.com && \
curl -s -k --noproxy "nginx.test.com" --resolve nginx.test.com:443:$ingress_controller_node  https://nginx.test.com
```

# 三、路由TCP服务

- **TLS Extensions:**TLS 扩展于 2003 年以一个独立的规范（[RFC 3546](https://tools.ietf.org/html/rfc3546)）被提出，经过不断的发展：[RFC 4366](https://tools.ietf.org/html/rfc4366)、[RFC 6066](https://tools.ietf.org/html/rfc6066) 等，先后被加入到 TLS1.1、TLS1.2、TLS1.3 中。它能让 Client 和 Server 在不更新 TLS 的基础上，获得新的功能。 在 ClientHello 中声明多个自己可以支持的 Extensions，Server 收到 ClientHello 以后，依次解析 Extensions，有些如果需要立即回应，就在 ServerHello 中作出回应，有些不需要回应，或者 Server 不支持的 Extensions 就不用响应，忽略不处理。ClientHello 中，Extension 字段位于 Compression Methods 字段之后

- **SNI(Server Name Indication)：**在 Nginx 中可以通过指定不同的 `server_name` 来配置多个站点。HTTP/1.1 协议请求头中的 `Host` 字段可以标识出当前请求属于哪个站点。但是在 TLS 协议中，没有提供一种机制来告诉 Server 它正在建立连接的 Server 的名称，那么对于在同一个地址，并且还使用不同证书的情况下，Server 怎么知道该发送哪个证书？为了解决这个问题，SNI 应运而生。最初是 2003 年标准化的，在 [RFC 6066](https://tools.ietf.org/html/rfc6066#page5) 中有更新。它允许 Server 在同一个网络地址上托管多个启用了 TLS 的服务，要求 Client 在初始 TLS 握手期间指定要连接到哪个服务。
- **Traefik的TCP路由与SNI**：从 Traefik 2.0 开始支持 TCP 路由，也支持在相同的 `entryPoints`（traefik 中的入口端口） 中定义不同的 TCP 路由，但是我们都知道，TCP 是传输层协议，没有任何 SNI 类的机制来保证同一地址入口可以处理不同的服务。Traefik 支持通过 SNI 在每台主机上进行路由，因为这是通过 TCP 进行路由的惟一标准方法，但是 TCP 本身没有 SNI，因此必须使用 TLS。

```bash
kubectl -n kube-public run redis --image=redis:alpine --port=6379 --labels="app=redis" && \
kubectl -n kube-public expose pod redis --name=redis --port=6379 --target-port=6379 --protocol=TCP && \
cat <<'EOF' | kubectl apply -f -
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRouteTCP
metadata:
  name: redis
  namespace: kube-public
spec:
  entryPoints:
    # 自建的 Entrypoint
    - tcp
  routes:
  # 由于TCP路由配置需要 SNI，而SNI又是依赖TLS。如果没有证书的话，可以使用通配符*(适配ip)进行配置
  - match: HostSNI(`*`)
    services:
    - name: redis
      port: 6379
EOF

ingress_controller_node=`kubectl -n kube-system get pod -l app.kubernetes.io/name=traefik -ojson | jq -r '.items[0] | .status.hostIP'` && \
redis-cli -h $ingress_controller_node -p 799 'info'
```

使用同一个 Entrypoint通过 SNI 路由不同的 TCP 服务暂调试不成功

```bash
kubectl -n kube-public run mysql --image=mysql --port=3306 --env="MYSQL_ROOT_PASSWORD=test123" --labels="app=mysql" && \
kubectl -n kube-public expose pod mysql --name=mysql --port=3306 --target-port=3306 --protocol=TCP && \

kubectl create secret tls redis-tls --cert=redis-tls.crt --key=redis-tls.key
openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout mysql-tls.key -out mysql-tls.crt -subj "/CN=mysql.test.com"
kubectl create secret tls mysql-tls --cert=mysql-tls.crt --key=mysql-tls.key
openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout redis-tls.key -out redis-tls.crt -subj "/CN=redis.test.com"

####
openssl genpkey -algorithm RSA -out mysql-tls.key
openssl req -new -subj "/C=CN/ST=Shanghai/L=Shanghai/O=Curiouser/OU=devops/CN=mysql.test.com" -key mysql-tls.key -out mysql-tls.csr
openssl x509 -req -days 3650 -in mysql-tls.csr -signkey mysql-tls.key -out mysql-tls.crt
kubectl create secret tls mysql-tls --cert=mysql-tls.crt --key=mysql-tls.key
openssl genpkey -algorithm RSA -out redis-tls.key
openssl req -new -subj "/C=CN/ST=Shanghai/L=Shanghai/O=Curiouser/OU=devops/CN=redis.test.com" -key redis-tls.key -out redis-tls.csr
openssl x509 -req -days 3650 -in redis-tls.csr -signkey redis-tls.key -out redis-tls.crt
kubectl create secret tls redis-tls --cert=redis-tls.crt --key=redis-tls.key
####

cat <<'EOF' | kubectl apply -f -
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRouteTCP
metadata:
  name: redis
  namespace: kube-public
spec:
  entryPoints:
    - tcp
  routes:
  - match: HostSNI(`redis.test.com`)
    services:
    - name: redis
      port: 6379
  tls:
    secretName: redis-tls
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRouteTCP
metadata:
  name: mysql
  namespace: kube-public
spec:
  entryPoints:
    - tcp
  routes:
  - match: HostSNI(`mysql.test.com`)
    services:
    - name: mysql
      port: 3306
  tls:
    secretName: mysql-tls
EOF
```

# 四、插件使用

## 1、sablier

```bash
cat <<'EOF' | kubectl apply -f -
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
    name: sablier
    namespace: kube-public
spec:
    plugin:
        sablier:
            dynamic:
                displayName: My Title
                refreshFrequency: 5s
                showDetails: "true"
                theme: hacker-terminal
            group: default
            names: whoami,nginx
            sablierUrl: http://nginx.test.com
            sessionDuration: 1m
EOF
```

# 五、常用操作

## 1、设置简单认证

### ①使用htpasswd生成密码文件

```bash
htpasswd -bc basic-auth-secret username password
```

### ②创建包含用户名密码的secret

```bash
kubectl create secret generic test-basic-auth --from-file=basic-auth-secret --namespace=test
```

### ③ingress配置中添加注解

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: traefik
    ingress.kubernetes.io/auth-type: basic
    ingress.kubernetes.io/auth-secret: test-basic-auth
  name: test
  namespace: test
.....省略.......
```

# 参考：

1. https://doc.traefik.io/traefik/v1.7/configuration/backends/kubernetes/
