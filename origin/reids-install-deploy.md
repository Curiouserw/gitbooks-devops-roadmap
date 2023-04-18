# Redis的安装部署及客户端

# 一、Redis单节点的安装部署

## 1、Docker

DockerHub镜像地址：https://hub.docker.com/_/redis/

```bash
docker run -d -p 6379:6379 --name redis redis:6.0.9-alpine
```

reids镜像相关信息：

- 环境变量配置：官方镜像不支持

- 持久化目录： /data

- 额外配置文件挂载：`-v /myredis/conf/redis.conf:/usr/local/etc/redis/redis.conf`

- 镜像定制：

  ```bash
  FROM redis:6.0.9-alpine
  COPY redis.conf /usr/local/etc/redis/redis.conf
  CMD [ "redis-server", "/usr/local/etc/redis/redis.conf" ]
  ```

## 2、Kubernetes

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

## 3、包管理器安装

### Brew

```bash
brew install redis
brew services start redis
```

### apt

```bash
apt install redis-server
```

### yum

```bash
yum install epel-release
yum install redis
```

## 4、源码安装

>  redis要求gcc版本高于5.3，CentOS7.4默认版本4.8.5，所以先升级gcc
>
>  redis要求tcl版本高于8.5，yum install -y tcl

```bash
yum -y install centos-release-scl
yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils
scl enable devtoolset-9 bash
echo "source /opt/rh/devtoolset-9/enable" >> /etc/profile
gcc -v
source /etc/profile 
```

```bash
version=6.0.9
wget https://download.redis.io/releases/redis-$version.tar.gz
tar xzf redis-$version.tar.gz
cd redis-$version
make
make install
# 二进制文件存放在src目录下,例如
source /etc/profile
nohup redis-server --protected-mode no >> /var/log/redis-server.log 2>&1 &
```

# 二、Redis配置

## 1、禁用重命名高危命令

```bash
[SECURITY]
rename-command FLUSHALL ""
rename-command FLUSHDB  ""
rename-command KEYS     "XXXXX"
rename-command CONFIG   "XXXXX"
# 重启Redis即可
```

# 三、Redis Cluster安装部署

# 四、Redis客户端

## 1、CLI

## 2、Application

### Another Redis Desktop Manager

Github：https://github.com/qishibo/AnotherRedisDesktopManager