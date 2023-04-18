# Git Submodule子模块

# 一、简介

经常碰到这种情况：当你在一个Git 项目上工作时，你需要在其中使用另外一个Git 项目。也许它是一个第三方开发的Git 库或者是你独立开发和并在多个父项目中使用的。这个情况下一个常见的问题产生了：你想将两个项目单独处理但是又需要在其中一个中使用另外一个。

在Git 中你可以用子模块`submodule`来管理这些项目，`submodule`允许你将一个Git 仓库当作另外一个Git 仓库的子目录。这允许你克隆另外一个仓库到你的项目中并且保持你的提交相对独立。

# 二、git submodule命令详解

## Git submodule命令

> git submodule 指令

## 指令

### add 

> add [-b <branch>] [-f|--force] [--name <name>] [--reference <repository>] [--depth <depth>] [--] <repository> [<path>]



### status

> status [--cached] [--recursive] [--] [<path>...]

### init

> init [--] [<path>...]

### deinit 

>deinit [-f|--force] (--all|[--] <path>...)

### update

> update [--init] [--remote] [-N|--no-fetch] [--[no-]recommend-shallow] [-f|--force] [--checkout|--rebase|--merge] [--reference <repository>] [--depth <depth>] [--recursive]
>        [--jobs <n>] [--] [<path>...]

### summary

> summary [--cached|--files] [(-n|--summary-limit) <n>] [commit] [--] [<path>...]

### foreach

> foreach [--recursive] <command>

### sync

> sync [--recursive] [--] [<path>...]

### absorbgitdirs

> 

# 三、操作

## 添加子模块

```bash
git submodule add ssh://git@gitlab.apps.okd311.curiouser.com:30022/Demo/git-submodule-public.git public
```

## 查看子模块

```bash
git submodule
 1b76c7ccb8e9a8460690433ffe5e14c3e9219890 public (heads/master)
```

## 初始化子模块

```bash

```

## 更新子模块

```bash
git submodule update
```

## 更新子模块

```bash
 git submodule update --remote
```



# 参考

1. https://www.cnblogs.com/Again/articles/6686105.html