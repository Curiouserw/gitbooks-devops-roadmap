# Python 环境的搭建

# 一、安装

## 1、源码编译安装

### CentOS

```bash
version=3.7.7
yum install -y sqlite-devel readline-devel tk-devel
wget https://www.python.org/ftp/python/$version/Python-$version.tgz
tar -xzf Python-$version.tgz
cd Python-$version
./configure --enable-optimizations
make
make install
```

### Ubuntu

```bash
version=3.7.7
apt update
apt install -y build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev \
			libsqlite3-dev libreadline-dev libffi-dev wget libbz2-dev
wget https://www.python.org/ftp/python/$version/Python-$version.tgz
tar -xzf Python-$version.tgz
cd Python-$version
./configure --enable-optimizations
make
make install
```

## 2、包管理器安装

### APT(Ubuntu/Debian)

```bash
apt install software-properties-common
add-apt-repository ppa:deadsnakes/ppa
apt-get update
apt-get install python3.6
```

# 二、Python的包管理器pip

## 1、 安装

### YUM

```
yum install -y epel-release ;\
yum install python-pip
```

### APT

```
apt-get install python3-pip
```

## 2、升级

```bash
pip3 install -U pip
# 或者
pthon3 -m pip install -U pip
```

## 3、安装依赖

```bash
pip3 install django
# 或者
python3.6 -m pip install django
```

## 4、固定安装的依赖

```bash
pip3 freeze > requirements.txt
```

# 三、Python虚拟隔离环境Virtualenv

如果我们要同时开发多个应用程序，那这些应用程序都会共用一个Python，就是安装在系统的Python 3。如果应用A需要jinja 2.7，而应用B需要jinja 2.6怎么办？这种情况下，每个应用可能需要各自拥有一套“独立”的Python运行环境。virtualenv就是用来为一个应用创建一套“隔离”的Python运行环境。

## 1、安装

```bash
pip3 install virtualenv
```

## 2、配置使用

```bash
virtualenv --python=python3.6 .
# 上述命令会在当前路径创建lib,include,bin目录

# --no-site-packages，参数设置已经安装到系统Python环境中的所有第三方包都不会复制过来，可以得到一个不带任何第三方包的“干净”的Python运行环境。
```

## 3、激活当前virtualenv

```bash
source ./bin/activate  
# 注意终端发生了变化
```

## 4、安装依赖

```bash
(venv) pip install -y requirement.txt  
```

## 5、关闭virtualenv

```ruby
(venv)deactivate
```

## 6、打包当前虚拟环境

```bash
(venv) -relocatable ./
```

## 7、固定当前虚拟环境中安装的依赖

```bash
(venv) pip freeze > requirements.txt
```

