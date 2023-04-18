# Nexus Pypi仓库的使用

# 一、简介

Nexus针对Python制品仓库的管理与使用

官方文档：https://help.sonatype.com/repomanager3/formats/pypi-repositories



**Group类型仓库**

- pypi-public

  ![](../assets/nexus-pypi-1.png)

**Proxy类型仓库**

- pypi-aliyun：http://mirrors.aliyun.com/pypi/

  ![](../assets/nexus-pypi-2.png)

- pupi-python：https://pypi.python.org/

  ![](../assets/nexus-pypi-3.png)

**Hosted类型仓库**

- pypi-hosted

  ![](../assets/nexus-pypi-4.png)



# 二、客户端使用

## 1、pip3配置仓库源

### ①全局配置

编辑 `~/.pip/pip.conf` 

```ini
[global]

# 用户名密码已配置在URL中。如果密码中包含特殊字符，使用特殊字符对应ASCII码的16进制进行代替。例如：密码中包含的“#”的ASCII为35,对应的16进制表示%23。密码中包含的“/”的ASCII为47,对应的16进制表示%2F。

index-url = http://pypi:******@nexus-ip:8081/repository/pypi-public/simple
trusted-host =  192.168.150.88
```

### ②临时配置

```bash
pip3 install flask \
-i http://pypi:******@nexus-ip:8081/repository/pypi-public/simple \
--trusted-host 192.168.150.88
```



### 注意

- 使用`pip3 config list` 可查看pip配置
- 官方已不再推荐支持 `easy_install` ，详见一下链接

- - https://setuptools.readthedocs.io/en/latest/easy_install.html
  - https://packaging.python.org/discussions/pip-vs-easy-install/

## 2、使用twine上传python制品到私有仓库

### ①安装twine

```bash
pip3 install twine \
-i http://pypi:******@nexus-ip:8081/repository/pypi-public/simple \
--trusted-host 192.168.150.88
```

### ②编写setup.py

```python
import setuptools
import os
import requests

# 将README.md中的描述文字作为制品的详细描述
with open("README.md", "r") as fh:
    long_description = fh.read()

# 将requirements.txt中依赖模块的版本信息作为制品的依赖描述
if os.path.exists("requirements.txt"):
    install_requires = io.open("requirements.txt").read().split("\n")
else:
    install_requires = []

setuptools.setup(
    # 项目命名
    name="demotest",
    # 版本
    version="0.0.1",
    # 作者
    author="curiouser",
    # 作者邮箱
    author_email="*******@163.com",
    # 项目制品包的简要描述
    description="test",
    # 项目制品包的详细描述  
    long_description=long_description,
    # 制品包详细描述的格式
    long_description_content_type="text/markdown",
    # 项目代码仓库地址
    url="https://github.com/test",
    
    packages=setuptools.find_packages(),
    
    classifiers=[
        "Programming Language :: Python :: 3",
        "License :: OSI Approved :: MIT License",
        "Operating System :: OS Independent",
    ],
    install_requires = install_requires,
    # 是否打包文件夹内的所有数据
    include_package_data=true,
    package_data = {
            # If any package contains *.txt or *.rst files, include them:
            'chinesename': ['source/*.txt', "source/*.json"],
    },
    # 如果需要支持脚本方法运行，可以配置入口点
    entry_points={
       'console_scripts': [
           'chinesename = chinesename.run:main'
       ]
    }
)
```

编写README.md

setuptools官方文档：https://packaging.python.org/guides/distributing-packages-using-setuptools/

setup.py推荐规则：https://github.com/pypa/sampleproject/blob/master/setup.py

### ③打包项目

```bash
python3 setup.py sdist bdist_wheel

# 打完包，会在当前目录下的dist目录下产生源文件tar.gz，分发文件.whl两个项目包
```

### ④上传项目包到hosted仓库

```bash
# 检测包
twine check dist/*

# 上传包
twine upload dist/* \
--repository-url http://nexus-ip:8081/repository/pypi-hosted/
```

### ⑤验证

![](../assets/nexus-pypi-5.png)

### ⑥下载使用

```bash
sudo pip3 install demotest==0.0.1
# 或者
sudo pip3 install demotest==0.0.1 \
-i http://pypi:******@nexus-ip:8081/repository/pypi-public/simple \
--trusted-host 192.168.150.88
```

![](../assets/nexus-pypi-6.png)