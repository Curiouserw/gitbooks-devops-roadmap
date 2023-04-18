# Kubernetes下的监控体系

# 一、监控对象



| 监控对象        | 示例                       | 监控信息暴露方式                             |
| --------------- | -------------------------- | -------------------------------------------- |
| k8s主机         | k8s依赖的主机              | node exporter以daemonset形式每个节点部署一个 |
| K8s上的中间件   | redis、mysql、kafka、mongo | 每种服务自带对应的exporter                   |
| K8s上的系统服务 | traefik                    | traefik自带暴露metrics                       |

