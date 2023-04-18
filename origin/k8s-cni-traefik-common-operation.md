# Traefik常用操作

# 一、设置简单认证

### 1、使用htpasswd工具快速生成用户名密码文件

```bash
htpasswd -bc basic-auth-secret username password
```

### 2、创建包含用户名密码的secret

```bash
kubectl create secret generic test-basic-auth --from-file=basic-auth-secret --namespace=test
```

### 3、ingress配置中添加注解

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

### 参考：

1. https://doc.traefik.io/traefik/v1.7/configuration/backends/kubernetes/

