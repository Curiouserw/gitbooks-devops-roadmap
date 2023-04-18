# Dockerfile优化

# 一、指令格式化

## LABEL

```bash
LABEL vendor=ACME\ Incorporated \
      com.example.is-beta= \
      com.example.is-production="" \
      com.example.version="0.0.1-beta" \
      com.example.release-date="2015-02-12"
```

## ENV

Dockerfile中ENV指令像RUN指令一样，每一个都会创建一个临时层。

```bash
ENV JAVA_HOME=/opt/jdk1.8.0_241 \
    CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib 
ENV PATH=$PATH:$JAVA_HOME/bin
```

## RUN

```bash
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        apt-transport-https \
        ca-certificates && \
    rm -rf /var/lib/apt/lists/* 
```

# 二、指令优化

## 1、减少RUN指令，合并命令

```bash
RUN useradd -s /sbin/nologin -m -u 1001 curiouser && \
    mkdir -p /home/curiouser/{data,logs} && \
    rm -rf /etc/yum.repos.d/C* && \
    yum install -q -y git && \
    yum clean all && \
    curl -s http://192.168.1.7/repository/tools/jdk-8u241-linux-x64.tar.gz | tar -xC /opt/
```

## 2、使用COPY来代替ADD

对于使用ADD指令下载远程服务器上的tar包并解压，建议使用以下方式代替

```bash
RUN curl -s http://192.168.1.7/repository/tools/jdk-8u241-linux-x64.tar.gz | tar -xC /opt/
```

# 三、最小化基础镜像，减小镜像体积

## 1、尽量使用Alpine作为基础镜像

- Alpine镜像大小最多才几MB。
- 使用APK命令装最小化需求的软件包

```bash
FROM alpine:3.11.5
RUN sed -i "s/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g" /etc/apk/repositories \
    && apk add --no-cache git
```

# 四、尽量不使用root用户

在做基础运行时镜像时，创建运行时普通用户和用户组，并做工作区与权限限制，启动服务时尽量使用普通用户。

## gosu

```bash
FROM alpine:3.11.5
RUN sed -i "s/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g" /etc/apk/repositories \
    && apk add --no-cache gosu

```

## 参考：

https://blog.csdn.net/boling_cavalry/article/details/93380447

# 五、使用进程管理工具来处理进程信号

为防止容器中的进程变成僵尸进程，

## dumb-init

Github地址：https://github.com/Yelp/dumb-init

```bash
FROM alpine:3.11.5
RUN sed -i "s/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g" /etc/apk/repositories \
    && apk add --no-cache dumb-init

# Runs "/usr/bin/dumb-init -- /my/script --with --args"
ENTRYPOINT ["dumb-init", "--"]

# or if you use --rewrite or other cli flags
# ENTRYPOINT ["dumb-init", "--rewrite", "2:3", "--"]

CMD ["/my/script", "--with", "--args"]
```

## 参考：

https://www.infoq.cn/article/2016/01/dumb-init-Docker

https://www.cnblogs.com/sunsky303/p/11046681.html

# 六、移除所有缓存等不必要信息

- 删除解压后的源压缩包（参考第二章第二节）
- 清理包管理器下载安装软件时的缓存
  - 使用Alipine镜像中APK命令安装包时记得加上`--no-cache`
  - 使用Ubuntu镜像中的APT命令安装软件后记得` rm -rf /var/lib/apt/lists/*`

# 七、使用合理的ENTRYPOINT脚本

示例：

```bash
#!/bin/bash
set -e

if [ "$1" = 'postgres' ]; then
    chown -R postgres "$PGDATA"

    if [ -z "$(ls -A "$PGDATA")" ]; then
        gosu postgres initdb
    fi

    exec gosu postgres "$@"
fi

exec "$@"
```

# 八、其他建议

## 1、设置时区

```
FROM alpine:3.11.5
ENV TZ=Asia/Shanghai
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories \
    && apk add --no-cache tzdata \
    && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo "Asia/Shanghai" > /etc/timezone
```

## 2、设置系统语言

```
FROM alpine:3.11.5
ENV LANG=en_US.UTF-8 \
    LANGUAGE=en_US.UTF-8
    
RUN apk --no-cache add ca-certificates \ 
    && wget -q -O /etc/apk/keys/sgerrand.rsa.pub https://alpine-pkgs.sgerrand.com/sgerrand.rsa.pub \ 
    && wget -q https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.29-r0/glibc-2.29-r0.apk \ 
    && wget -q https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.29-r0/glibc-bin-2.29-r0.apk \
    && wget -q https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.29-r0/glibc-i18n-2.29-r0.apk \
    && apk add glibc-2.29-r0.apk glibc-bin-2.29-r0.apk glibc-i18n-2.29-r0.apk \
    && rm -rf /usr/lib/jvm glibc-2.29-r0.apk glibc-bin-2.29-r0.apk  glibc-i18n-2.29-r0.apk \
    && /usr/glibc-compat/bin/localedef --force --inputfile POSIX --charmap UTF-8 "$LANG" || true \
    && echo "export LANG=$LANG" > /etc/profile.d/locale.sh \
    && apk del glibc-i18n
```

## 3、使用Label标注作者、软件版本等元信息

```bash
FROM alpine:3.11.5
LABEL Author=Curiouser \
      Mail=****@163.com \
      PHP=7.3 \
      Tools=“git、vim、curl” \
      Update="添加用户组"
```

## 4、指定工作区

```
WORKDIR /var/wwww
```

## 5、RUN指令显示优化

```bash
RUN set -eux ; \
    ls -al
```

# 九、镜像构建

## 1、命名

原则是见名知意。可使用三段式

>  镜像仓库地址/类型库/镜像名:版本号

- registry/runtime/Java:8.1.2
- registry/runtime/php-fpm-nginx:7.3-1.14
- registry/cicd/kubctl-helm:1.17-3.0
- registry/cicd/git-compose-docker:v1
- registry/applications/demo:git_commit_id

## 2、使用Makefile

```bash
IMAGE_BASE = registry/runtime
IMAGE_NAME = php-fpm
IMAGE_VERSION = 7.3
all: build push
build:
        docker build --rm -f Dockerfile -t ${IMAGE_BASE}/${IMAGE_NAME}:${IMAGE_VERSION} .
push:
        docker push ${IMAGE_BASE}/${IMAGE_NAME}:${IMAGE_VERSION}
# 构建并推送
make 
# 仅构建
make build 
# 仅推送
make push
```

# 参考

1. https://docs.docker.com/develop/develop-images/dockerfile_best-practices/
2. https://www.artindustrial-it.com/2017/09/20/10-best-practices-for-creating-good-docker-images/
3. https://gist.github.com/StevenACoffman/41fee08e8782b411a4a26b9700ad7af5
4. https://snyk.io/blog/10-docker-image-security-best-practices/

