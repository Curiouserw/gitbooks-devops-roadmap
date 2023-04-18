# Linux zsh && oh-my-zsh

# 一、简介

Zsh 也许是目前最好用的 shell，是 bash 替代品中较为优秀的一个。

Zsh 官网：http://www.zsh.org/

Zsh具有以下主要优势：

- 完全兼容bash，之前bash下的使用习惯，shell脚本都可以完全兼容
- 更强大的tab补全
- 更智能的切换目录
- 命令选项、参数补齐
- 大小写字母自动更正
- 有着丰富多彩的主题
- 更强大的alias命令
- 智能命令错误纠正
- 集成各种类型的插件

[oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh) 是最为流行的 zsh 配置文件，提供了大量的主题和插件，极大的拓展了 zsh 的功能，推动了 zsh 的流行，有点类似于 rails 之于 ruby。

# 二、安装zsh

**`CentOS`**

```bash
yum install -y zsh
```

**`Ubuntu`**

```bash
apt-get install -y zsh
```

> 检查下系统的 shell：`$ cat /etc/shells`，你会发现多了一个：/bin/zsh

**`设置用户的默认shell`**

```bash
# 给root用户设置
chsh -s /bin/zsh root
# 给普通账户设置
chsh -s /bin/zsh 用户名
#　恢复bash
chsh -s /bin/bash [user]
```

# 三、安装oh-my-zsh

oh-my-zsh 帮我们整理了一些常用的 Zsh 扩展功能和主题，我们无需自己去捣搞 Zsh，直接用 oh-my-zsh 就足够了。

oh-my-zsh 官网：https://ohmyz.sh/

oh-my-zsh Github：https://github.com/robbyrussell/oh-my-zsh

**`curl`**

```bash
curl -Lo install.sh https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh

chmod +x install.sh
REPO="mirrors/oh-my-zsh" REMOTE="https://gitee.com/mirrors/oh-my-zsh.git" sh install.sh --skip-chsh

# 下载相关插件
git clone https://github.com/zsh-users/zsh-autosuggestions.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

# 四、oh-my-zsh配置

oh-my-zsh的配置文件路径为`~/.zshrc`

```properties
# If you come from bash you might have to change your $PATH.
# export PATH=$HOME/bin:/usr/local/bin:$PATH

# 指定oh-my-zsh的安装路径
export ZSH="/root/.oh-my-zsh"

# 设置主题。如果设置为"random", 每次启动oh-my-zsh会随机加载主题,可使用`echo $RANDOM_THEME`查看每次加载的主题
ZSH_THEME="alanpeabody"

# 设置随机的主题
# ZSH_THEME_RANDOM_CANDIDATES=( "robbyrussell" "agnoster" )

# 大小写是否敏感
# CASE_SENSITIVE="true"

# Uncomment the following line to use hyphen-insensitive completion.
# Case-sensitive completion must be off. _ and - will be interchangeable.
# HYPHEN_INSENSITIVE="true"

# 设置是否自动更新
# DISABLE_AUTO_UPDATE="true"

# 设置自动更新的天数。默认13天
# export UPDATE_ZSH_DAYS=13

# 设置是否开启`ls`进行颜色显示
# DISABLE_LS_COLORS="true"

# 设置是否显示终端标题
# DISABLE_AUTO_TITLE="true"

# 设置是否开启语法修正
# ENABLE_CORRECTION="true"

# Uncomment the following line to display red dots whilst waiting for completion.
# COMPLETION_WAITING_DOTS="true"

# Uncomment the following line if you want to disable marking untracked files
# under VCS as dirty. This makes repository status check for large repositories
# much, much faster.
# DISABLE_UNTRACKED_FILES_DIRTY="true"

# 历史输入命令的时间展示格式
# HIST_STAMPS="mm/dd/yyyy"

# 设置自定义配置文件的路径
# ZSH_CUSTOM=/path/to/new-custom-folder

# 设置存储历史命令的默认文件路径
# HISTFILE=~/.zsh_history

# 配置要加载的插件（配置的插件要能在 ~/.oh-my-zsh/plugins/* 下找到，自定义的插件目录为 ~/.oh-my-zsh/custom/plugins/ ）.注意：插件安装的越多，zsh的启动速度越慢，选择使用率最高的插件才是最好的选择
plugins=(
  git
  zsh-autosuggestions
  zsh-syntax-highlighting
  kubectl
  docker
  sudo
  extract
)
source /etc/profile
source $ZSH/oh-my-zsh.sh

# 用户配置

# 设置man文档的环境变量
# export MANPATH="/usr/local/man:$MANPATH"

# 设置语言环境变量
# export LANG=en_US.UTF-8

# 设置本地和远程sessions的首选编辑器
# if [[ -n $SSH_CONNECTION ]]; then
#   export EDITOR='vim'
# else
#   export EDITOR='mvim'
# fi

# 设置编译标志
# export ARCHFLAGS="-arch x86_64"

# 设置别名
alias ll="ls -alh"
alias k='kubectl'
. ~/.oh-my-zsh/custom/oc_zsh_completion
```

# 五、oh-my-zsh常用插件

- **git**：可以使用git缩写，默认自带

   `git add --all` => `gaa`

  查看所有缩写：`alias | grep git`

- **autojump**：快速跳转文件夹

- **last-working-dir**：可以记录上一次退出命令行时候的所在路径，并且在下一次启动命令行的时候自动恢复到上一次所在的路径。
  
- **wd**：快速地切换到常用的目录
  `wd add web`相当于给当前目录做了一个标识，标识名叫做 `web` ，我们下次如果再想进入这个目录，只需输入：`wd web`

- **catimg**：将图片的内容输出到命令行

`catimg demo.jpg`

- **zsh-syntax-highlighting**：命令高亮 正确路径自带下划线

  安装：`git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting`

- **zsh-autosuggestions**：自动补全可能的路径
  
  安装：`git clone git://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions`
  
- **sudo**：连按两次Esc添加或去掉sudo
  
- **extract**：功能强大的解压插件
  执行`x demo.tar.gz`
  
- **git-open**：在终端里打开当前项目的远程仓库地址

  安装：`git clone https://github.com/paulirish/git-open.git $ZSH_CUSTOM/plugins/git-open`
  
- **history**：内置，快速搜索history

# 六、oh-my-zsh常用主题

官方主题：https://github.com/robbyrussell/oh-my-zsh/wiki/Themes

# 七、升级

```bash
omz update
```

