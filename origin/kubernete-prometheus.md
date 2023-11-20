# Kubernetes下的监控体系

# 一、监控对象

| 监控对象        | 示例                       | 监控信息暴露方式                             |
| --------------- | -------------------------- | -------------------------------------------- |
| k8s主机         | k8s依赖的主机              | node exporter以daemonset形式每个节点部署一个 |
| K8s上的中间件   | redis、mysql、kafka、mongo | 每种服务自带对应的exporter                   |
| K8s上的系统服务 | traefik                    | traefik自带暴露metrics                       |
| k8s上的应用服务 | application                | 部署的应用开起的 Metrics                     |
|                 |                            |                                              |

```bash
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: application-php-fpm
  namespace: monitoring
spec:
  endpoints:
  - interval: 15s
    port: phpfpm-exporter
  namespaceSelector:
    matchNames:
    - test
    - stg
    - monitoring
  selector:
    matchLabels:
      prometheus-target: application-php-fpm
---
apiVersion: v1
kind: Service
metadata:
  labels:
    prometheus-target: application-php-fpm
  name: nginx
  namespace: stg
spec:
  ports:
  - name: nginx
    port: 80
    protocol: TCP
    targetPort: 80
  - name: phpfpm-exporter
    port: 9253
    protocol: TCP
    targetPort: 9253
  selector:
    app: saas-base-service
```
