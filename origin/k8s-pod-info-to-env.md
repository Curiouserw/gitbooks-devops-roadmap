# 一、将POD信息传递容器环境变量或文件

​	对于容器来说，有时候拥有自己的信息是很有用的，可避免与 Kubernetes 过度耦合。 Downward API 使得容器使用自己或者集群的信息，而不必通过 Kubernetes 客户端或 API 服务器来获得。

可以通过环境变量和 `downwardAPI` 卷提供给容器：

- 能通过`fieldRef`获得的：

  - `metadata.name` : Pod 名称

  - `metadata.namespace` : Pod 名字空间

  - `metadata.uid` : Pod 的 UID

  - `metadata.labels['<KEY>']` 

    Pod 标签 `<KEY>` 的值 (例如, `metadata.labels['mylabel']`）。以 `label-key="escaped-label-value"` 格式显示，每行显示一个标签

  - `metadata.annotations['<KEY>']` 

     Pod 的注解 `<KEY>` 的值（例如, `metadata.annotations['myannotation']`）。以 `annotation-key="escaped-annotation-value"` 格式显示，每行显示一个标签

- 能通过`resourceFieldRef`获得的：
  - `容器的 CPU 约束值`
  - `容器的 CPU 请求值`
  - `容器的内存约束值`
  - `容器的内存请求值`
  - `容器的巨页限制值`（前提是启用了 `DownwardAPIHugePages` [特性门控](https://kubernetes.io/zh/docs/reference/command-line-tools-reference/feature-gates/)）
  - `容器的巨页请求值`（前提是启用了 `DownwardAPIHugePages` [特性门控](https://kubernetes.io/zh/docs/reference/command-line-tools-reference/feature-gates/)）
  - `容器的临时存储约束值`
  - `容器的临时存储请求值`



有两种方式可以将 Pod 和 Container 字段呈现给运行中的容器：

## 1、传递给环境变量

```yaml
.......
spec:
  containers:
    - name: test-container
      ......
      env:
        - name: MY_NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        - name: MY_POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: MY_POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: MY_POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: MY_POD_SERVICE_ACCOUNT
          valueFrom:
            fieldRef:
              fieldPath: spec.serviceAccountName
        - name: MY_CPU_REQUEST
          valueFrom:
            resourceFieldRef:
              containerName: test-container
              resource: requests.cpu
        - name: MY_CPU_LIMIT
          valueFrom:
            resourceFieldRef:
              containerName: test-container
              resource: limits.cpu
        - name: MY_MEM_REQUEST
          valueFrom:
            resourceFieldRef:
              containerName: test-container
              resource: requests.memory
        - name: MY_MEM_LIMIT
          valueFrom:
            resourceFieldRef:
              containerName: test-container
              resource: limits.memory
```

`env` 字段是一个 [EnvVars](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.23/#envvar-v1-core). 对象的数组。 数组中第一个元素指定 `MY_NODE_NAME` 这个环境变量从 Pod 的 `spec.nodeName` 字段获取变量值。 

## 2、传递给文件进行挂载

```yaml
.....
spec:
  containers:
    - name: client-container
      image: k8s.gcr.io/busybox
      command: ["sh", "-c"]
      args:
      - while true; do
          if [[ -e /etc/podinfo/labels ]]; then
            echo -en '\n\n'; cat /etc/podinfo/labels; fi;
          if [[ -e /etc/podinfo/annotations ]]; then
            echo -en '\n\n'; cat /etc/podinfo/annotations; fi;
            
          if [[ -e /etc/podinfo/cpu_limit ]]; then
            echo -en '\n'; cat /etc/podinfo/cpu_limit; fi;
          if [[ -e /etc/podinfo/cpu_request ]]; then
            echo -en '\n'; cat /etc/podinfo/cpu_request; fi;
          if [[ -e /etc/podinfo/mem_limit ]]; then
            echo -en '\n'; cat /etc/podinfo/mem_limit; fi;
          if [[ -e /etc/podinfo/mem_request ]]; then
            echo -en '\n'; cat /etc/podinfo/mem_request; fi;
          sleep 5;
        done;
      volumeMounts:
        - name: podinfo
          mountPath: /etc/podinfo
  volumes:
    - name: podinfo
      downwardAPI:
        items:
          - path: "labels"
            fieldRef:
              fieldPath: metadata.labels
          - path: "annotations"
            fieldRef:
              fieldPath: metadata.annotations
          - path: "cpu_limit"
            resourceFieldRef:
              containerName: client-container
              resource: limits.cpu
              divisor: 1m
          - path: "cpu_request"
            resourceFieldRef:
              containerName: client-container
              resource: requests.cpu
              divisor: 1m
          - path: "mem_limit"
            resourceFieldRef:
              containerName: client-container
              resource: limits.memory
              divisor: 1Mi
          - path: "mem_request"
            resourceFieldRef:
              containerName: client-container
              resource: requests.memory
              divisor: 1Mi
```

**`如果容器以subPath卷挂载方式来使用 Downward API，则该容器无法收到更新事件`**



在输出中可以看到，`labels` 和 `annotations` 文件都在一个临时子目录中。 在这个例子，`..2982_06_02_21_47_53.299460680`。 在 `/etc/podinfo` 目录中，`..data` 是一个指向临时子目录 的符号链接。`/etc/podinfo` 目录中，`labels` 和 `annotations` 也是符号链接。

```
drwxr-xr-x  ... Feb 6 21:47 ..2982_06_02_21_47_53.299460680
lrwxrwxrwx  ... Feb 6 21:47 ..data -> ..2982_06_02_21_47_53.299460680
lrwxrwxrwx  ... Feb 6 21:47 annotations -> ..data/annotations
lrwxrwxrwx  ... Feb 6 21:47 labels -> ..data/labels

/etc/podinfo/..2982_06_02_21_47_53.299460680:
total 8
-rw-r--r--  ... Feb  6 21:47 annotations
-rw-r--r--  ... Feb  6 21:47 labels
```

用符号链接可实现元数据的动态原子性刷新；更新将写入一个新的临时目录， 然后通过使用[rename(2)](http://man7.org/linux/man-pages/man2/rename.2.html) 完成 `..data` 符号链接的原子性更新。



## 参考：

- https://kubernetes.io/zh/docs/tasks/inject-data-application/environment-variable-expose-pod-information/
- https://kubernetes.io/zh/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/#the-downward-api