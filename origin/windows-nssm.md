# Windows下的进程守护工具NSSM

# 一、简介

**[NSSM](https://nssm.cc/)**(the Non-Sucking Service Manager)是Windows环境下一款**免安装**的服务管理软件，它可以将应用封装成服务，使之像windows服务可以设置自动启动等。并且可以监控程序运行状态，程序异常中断后自动启动，实现守护进程的功能。不仅支持图形界面操作，也完全支持命令行设置。

同类型的工具还有微软自己的srvany，不过nssm更加简单易用，并且功能强大。它的特点如下：

1. 支持普通exe程序（控制台程序或者带界面的Windows程序都可以）
2. 安装简单，修改方便
3. 可以重定向输出（并且支持Rotation）
4. 可以自动守护封装了的服务，程序挂掉了后可以自动重启
5. 可以自定义环境变量

这里面的每一个功能都非常实用，使用NSSM来封装服务可以大大简化我们的开发流程了。

- 开发的时候是一个普通程序，降低了开发难度，调试起来非常方便
- 安装简单，并且可以随时修改服务属性，更新也更加方便
- 可以利用控制台输出直接实现一个简单的日志系统
- 不用考虑再加一个服务实现服务守护功能



官方网站：https://nssm.cc/

下载地址：https://nssm.cc/download （下载解压到windows系统环境Path下，在CMD中可使用即可）

# 二、配置

使用文档：https://nssm.cc/usage

## 1、服务的配置

### ①安装服务的命令格式

​	`nssm install 服务名 参数项 [<args>...] `

### ②设置服务参数配置的命令格式

`nssm set <servicename> <parameter> [subparameter] value`

### ③重置服务参数配置的命令格式

`nssm reset <servicename> <parameter> [subparameter]`



# 三、服务的生命周期管理

## 1、注册一个服务

```bash
# 打开GUI界面配置一个服务
nssm install  

# 命令行配置一个服务
nssm install 服务名  "C:\Program Files\Java\jdk1.8.0_251\bin\java.exe"
```

## 2、配置一个服务

```bash
nssm set 服务名 Application “C:\Program Files\Java\jdk1.8.0_251\bin\java.exe”
nssm set 服务名 AppDirectory “C:\Application”
nssm set 服务名 AppParameters “-jar test.jar ”
nssm set 服务名 DisplayName "Test"
nssm set 服务名 Description "测试"
nssm set 服务名 Start SERVICE_AUTO_START
```

## 3、列出所有服务

```bash
nssm list
```

## 4、查看一个服务的配置

```bash
nssm get 
```

## 5、启动一个服务

```bash
nssm start 服务名
```

## 6、查看服务的状态

```bash
nssm status 服务名
```

## 7、停止一个服务

```bash
nssm stop 服务名
```

## 8、重启一个服务

```bash
nssm restart 服务名
```

## 9、删除一个服务

```bash
nssm remove 服务名
```

## 10、暂停/继续服务

```bash
nssm pause 服务名
nssm continue 服务名
```

## 11、手动轮转日志文件

```bash
nssm rotate 服务名
```


