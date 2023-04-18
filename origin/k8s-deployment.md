# 无状态应用K8s部署文件Deployment

# 一、简介



## 二、说明

## 1、添加注解以标注发布变更历史

Deployment或者statefulset添加`kubernetes.io/change-cause`的注解用以标注发布变更历史。可以使用镜像版本号作为说明，也可以时间戳等。

```yaml
...
metadata:
  annotations:
    kubernetes.io/change-cause: $IMAGE_NAME
...
```

```bash
$ kubectl rollout history deployment test-nginx
REVISION  CHANGE-CAUSE
1        harbor.curiouser.com/test-nginx/stg:20210908-a3das215
2        harbor.curiouser.com/test-nginx/stg:20210908-8020cdfh
```

## 2、command字段和args字段

| 描述           | Docker字段名称 | Kubernetes字段名称 |
| :------------- | :------------- | :----------------- |
| 容器执行的命令 | Entrypoint     | command            |
| 传给命令的参数 | Cmd            | args               |

如果要覆盖Docker容器默认的 `Entrypoint` 与 `Cmd`，需要遵循如下规则：

- 如果在 Pod 配置中没有设置 `command` 或者 `args`，那么将使用 Docker 镜像自带的命令及其参数。
- 如果在 Pod 配置中只设置了 `command` 但是没有设置 `args`，那么容器启动时只会执行该命令，Docker 镜像中自带的命令及其参数会被忽略。
- 如果在 Pod 配置中只设置了 `args`，那么 Docker 镜像中自带的命令会使用该新参数作为其执行时的参数。
- 如果在 Pod 配置中同时设置了 `command` 与 `args`，那么 Docker 镜像中自带的命令及其参数会被忽略。容器启动时只会执行配置中设置的命令，并使用配置中设置的参数作为命令的参数。

```yaml
spec:
  containers:
  - name: command-demo-container
    image: debian
    command: ["/bin/sh"]
    args: ["-c", "可以前台运行的进程命令"]
```

- 环境变量需要加上括号，类似于 `"$(VAR)"`。这是在 `command` 或 `args` 字段使用变量的格式要求。