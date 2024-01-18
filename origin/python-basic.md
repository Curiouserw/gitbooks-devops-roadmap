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
python3 -m pip install -U pip
python3 -m pip install --upgrade pip
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

# 四、AnaConda、MiniConda

## 1、设置 conda 源

`~/.condarc`

```bash
# 配置了 Conda 包搜索的顺序。Conda 将按照列表中的顺序搜索包，直到找到第一个匹配的包为止
channels:
  - defaults

show_channel_urls: true											# 是否在安装过程中显示从哪个频道下载的包
auto_update_conda: false										# 配置了是否在每次使用 Conda 命令时自动更新 Conda。
create_default_packages:										# 配置了创建新环境时默认安装的包
  - numpy
  - pandas
anaconda_anon_usage: false									# 配置是否允许匿名使用数据的收集
channel_priority: strict								# 配置了Conda是否优先使用指定的频道."strict"(严格)表示只使用指定频道,"flexible"(灵活)表示可以使用其他频道
channel_alias:															# 设置频道别名，以便更方便地引用频道
  my_alias: https://my_channel_url

default_channels:														# 设置了 Conda 搜索包时的默认频道列表
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
custom_channels:														# 设置自定义频道
  conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  msys2: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  bioconda: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  menpo: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  pytorch: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  pytorch-lts: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  simpleitk: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  deepmodeling: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/
```

`conda clean -i` 清除索引缓存

参考：https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/

## 2、包管理

```bash
# 列出当前激活环境的所有包
conda list
# 列出一个非激活环境的所有包
conda list -n base
# 为指定环境安装某个包
conda install -n base package_name
# 从指定 channel 为指定环境安装某个包
conda install -n base -c defaults package_name
```

## 3、环境管理

```bash
# 列出当前所有环境
conda info --envs
conda env list

# 创建包含某些包的环境
conda create --name your_env_name numpy scipy
# 创建指定Python版本下包含某些包的环境
conda create --name your_env_name python=3.8 numpy scipy

# 激活某个环境
activate your_env_name
# 关闭某个环境
deactivate your_env_name
# 克隆某个环境
conda create --name new_env_name --clone old_env_name
# 删除某个环境
conda remove --name your_env_name --all
# 分享环境
conda env export > share_env.yml
# 创建该环境
conda env create -f share_env.yml
```

## 4、配置操作

```bash
# 清除一下缓存
conda clean -i
# 清除所有缓存
conda clean --all
# 查看全部配置信息
conda config --show
# 查看源的配置信息
conda config --show-sources
# 查看源的详细信息
conda info

# 升级 conda
conda update -n base conda
```
