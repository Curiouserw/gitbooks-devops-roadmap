# SSH私钥管理程序 SSH-Agent

# 一、ssh-agent能干什么？

1. **多SSH私钥管理**

   使用不同的密钥连接到不同的主机时，需要手动指定对应的密钥，ssh-agent可以帮助我们选择对应的密钥进行认证，不用手动指定密钥即可进行连接。

2. **避免重复输入加密私钥的密码**

   当私钥设置了密码，我们又需要频繁的使用私钥进行认证时，ssh-agent可以帮助我们免去重复的输入密码的操作

# 二、ssh-agent管理

## 1、安装

```bash
yum install -y openssh-clients
```

## 2、启动

`ssh-agent`启动有两种方式

- `ssh-agent #SHELL`

  在OS默认shell中再创建一个子shell，在子shell中运行ssh-agent进程，退出子shell自动结束代理。

- `eval '$(ssh-agent -s)'  `

  单独启动一个代理进程，退出当前shell时最好使用ssh-agent -k关闭对应代理

## 3、关闭

- 在当前bash中，使用`ssh-agent -k`命令可以关闭对应的ssh-agent进程
- 如果在退出了当前bash以后再使用'ssh-agent -k'命令，是无法关闭对应的ssh-agent进程的，此时使用kill命令
- 使用 ssh-agent $SHELL 命令启动的ssh-agent也可以使用'ssh-agent -k'命令关闭ssh代理。

## 4、锁定ssh-agent

```bash
# 加锁，需要输入加锁密码
ssh-add -x
# 解锁，需要输入加锁密码
ssh-add -X
```

# 三、私钥管理

## ①添加私钥到ssh-agent

```bash
ssh-add ~/.ssh/id_rsa
```

## ②删除ssh-agent管理的私钥

```bash
ssh-add -d ～/.ssh/id_rsa
# 删除ssh-agent中所有的私钥
ssh-add -D
```

## ③查看ssh-agent管理的私钥

```bash
ssh-add -l
```

## ④查看ssh-agent管理的私钥对应的公钥内容

```bash
ssh-add -L
```

