

# Web IDE - VSCode - Code-Server

## 一、简介

文档：https://coder.com/docs/code-server/latest

Github：https://github.com/coder/code-server

Dockerhub：https://hub.docker.com/r/codercom/code-server

## 二、部署

```yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: developer-A
  namespace: ide
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 20Gi
  storageClassName: nfs-client
---
apiVersion: v1
kind: Service
metadata:
 name: developer-A-svc
 namespace: ide
spec:
 ports:
 - port: 80
   targetPort: 8080
 selector:
   code-server: developer-A
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: developer-A-vscode-server
  namespace: ide
spec:
  rules:
  - host: developer-A.curiouser.com
    http:
      paths:
      - backend:
          serviceName: developer-A-svc
          servicePort: 80
        path: /
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    code-server: developer-A
  name: developer-A
  namespace: ide
spec:
  selector:
    matchLabels:
      code-server: developer-A
  template:
    metadata:
      labels:
        code-server: developer-A
    spec:
      volumes:
        - name: user-workspace-data
          persistentVolumeClaim:
            claimName: developer-A
      hostname: developer-A
      containers:
      - image: hub.curiouser.com/vscode-server/backend:4.16.1-ubuntu
        imagePullPolicy: IfNotPresent
        name: vscode
        ports:
        - containerPort: 8080
        env:
        - name: HASHED_PASSWORD
          value: "4972cdcd065d9df443a8422c5a899d49be5b7b1e123ca9ff0663dbc8f461bf674"
        volumeMounts:
        - name: user-workspace-data
          mountPath: /home/coder
        resources:
          limits:
            cpu: "1500m"
            memory: 4000Mi
          requests:
            cpu: "100m"
            memory: 100Mi
        livenessProbe:
          failureThreshold: 3
          initialDelaySeconds: 10
          periodSeconds: 60
          successThreshold: 1
          tcpSocket:
            port: 8080
          timeoutSeconds: 2
        readinessProbe:
          failureThreshold: 3
          initialDelaySeconds: 10
          periodSeconds: 60
          successThreshold: 1
          tcpSocket:
            port: 8080
          timeoutSeconds: 2
```



## 三、定制镜像

- 固定开发环境
  - 统一运行时软件版本、系统版本、系统工具
  - 配置文件化、统一配置、后期自由扩展

> Dockerfile

```dockerfile
FROM codercom/code-server:4.16.1-ubuntu
ENV LANG=en_US.UTF-8 \
    LANGUAGE=en_US.UTF-8 \
    TZ=Asia/Shanghai

RUN sudo sed -i -e 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' -e 's/security.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list && \
    sudo apt update && \
    sudo apt install -y pip zsh vim jq telnet && \
    sudo apt-get clean && \
    sudo rm -rf /tmp/* /var/lib/apt/lists/* /var/tmp/*

ENTRYPOINT [ "/usr/bin/entrypoint.sh","--disable-telemetry","--disable-getting-started-override","--disable-file-downloads","--bind-addr","0.0.0.0:8080","."]
```

> Makefile

```makefile
IMAGE_BASE_PUSH = hub.curiouser.com/vscode-server
IMAGE_NAME = backend
IMAGE_VERSION = 4.16.1-ubuntu
all: build push
build:
		docker build --rm -f Dockerfile -t ${IMAGE_BASE_PUSH}/${IMAGE_NAME}:${IMAGE_VERSION} .
push:
		docker push ${IMAGE_BASE_PUSH}/${IMAGE_NAME}:${IMAGE_VERSION}
```

## 四、网络访问

### 1、VSCode内应用的访问

https://coder.com/docs/code-server/latest/guide#accessing-web-services