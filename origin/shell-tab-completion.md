# Shell命令自动补全

# 一、自动补全的类型

- **命令补全：**输入命令的前几个字母后按 `Tab`，自动补全可执行程序名。

- **路径补全：**输入部分路径后按 `Tab`，自动补全文件或目录

- **参数/选项补全：**部分命令支持补全选项或参数（需安装补全脚本）

- **变量补全：**全环境变量名

# 二、不同Shell的自动补全

- **Bash**

  - 基础补全：默认支持命令和路径补全

  - 增强补全：需安装 `bash-completion` 包，支持复杂命令（如 `git`、`docker`）

    ```bash
    sudo apt install bash-completion
    ```

- **Zsh**

  - 功能更强大，支持上下文感知补全（如根据历史命令推断参数）

  - 推荐使用框架（如 **Oh My Zsh**）扩展功能。
    **配置**：在 `~/.zshrc` 中启用插件：`plugins=(git docker npm)`

- **Fish**
  - 开箱即用，无需配置即可智能补全，支持实时提示

# 三、Bash的complete

- **关联补全规则**：为命令定义参数补全逻辑（如选项、文件路径、固定列表等）。
- **动态补全**：通过回调函数动态生成候选补全列表。
- **覆盖默认规则**：修改或扩展系统或第三方命令的默认补全行为。

## 1、基本语法

```bash
complete [选项] [命令名]
```

常用选项：

| 选项               | 作用                               |
| :----------------- | :--------------------------------- |
| `-W "词1 词2 ..."` | 指定静态候选词列表（空格分隔）     |
| `-F 函数名`        | 指定动态生成候选列表的 Bash 函数   |
| `-C 命令`          | 调用外部命令生成候选列表           |
| `-d`               | 仅补全目录（类似 `cd` 的默认行为） |
| `-f`               | 仅补全文件名                       |
| `-A alias`         | 补全别名                           |
| `-A export`        | 补全环境变量名                     |
| `-r`               | 移除已注册的补全规则               |

------

## 2、静态补全（固定列表）

为命令 `mycmd` 定义固定候选词（`start`、`stop`、`restart`）

```bash
complete -W "start stop restart" mycmd
```

- **效果**：输入 `mycmd [Tab]` 时，自动补全这三个词。

## 3、动态补全（自定义函数）

为命令 `git` 动态补全分支名（示例简化逻辑）

```bash
_git_branch_complete() {
    local branches=$(git branch --list | cut -c 3-)
    COMPREPLY=($(compgen -W "$branches" -- "${COMP_WORDS[COMP_CWORD]}"))
}
complete -F _git_branch_complete git
```

- **关键变量**：
  - `COMPREPLY`：存储补全候选的数组（必须赋值）。
  - `COMP_WORDS`：当前命令行所有单词的数组。
  - `COMP_CWORD`：当前光标所在单词的索引。
- **工具函数**：`compgen -W` 用于过滤候选词。

## 4、补全文件类型

为命令 `convert` 仅补全 `.png` 和 `.jpg` 文件：

```bash
complete -f -X '!*.@(png|jpg)' convert
```

- `-X`：排除模式（`!` 表示取反）。
- `@(png|jpg)`：Bash 扩展模式（需启用 `extglob`）。

## 5、多级补全

根据前一个参数决定后续补全内容（例如 `docker` 的子命令）：

```bash
_docker_complete() {
    local prev=${COMP_WORDS[COMP_CWORD-1]}
    case $prev in
        run) COMPREPLY=($(compgen -W "--name -it -v" -- "$cur")) ;;
        exec) COMPREPLY=($(docker ps --format "{{.Names}}")) ;;
        *) COMPREPLY=($(compgen -W "run exec ps stop" -- "$cur")) ;;
    esac
}
complete -F _docker_complete docker
```

## 6、补全环境变量

为 `echo` 补全环境变量名（需先定义变量）：

```bash
complete -A export echo
```

## 7、使用外部命令生成补全

通过 `ls` 命令补全文件名

```bash
complete -C 'ls -1' myls
```

------

## 8、补全脚本的存放位置

Bash 自动加载以下目录中的补全脚本：

```bash
/etc/bash_completion.d/      # 系统级补全（需 root 权限）
~/.bash_completion           # 用户级自定义补全
```

- **推荐工具**：安装 `bash-completion` 包以增强系统命令的补全支持：

  ```bash
  # Debian/Ubuntu
  sudo apt install bash-completion
  
  # RedHat/CentOS
  sudo yum install bash-completion
  ```

## 9、调试与重载

1. **手动重载补全脚本**

   ```bash
   source ~/.bash_completion  # 重新加载用户补全脚本
   ```

2. **查看已注册的补全规则**

   ```bash
   complete -p                 # 列出所有补全规则
   complete -p docker          # 查看特定命令的补全规则
   ```

3. **删除补全规则**

   ```bash
   complete -r mycmd           # 移除对 mycmd 的补全
   ```

# 四、Zsh的compdef

`compdef` 是 Zsh 中用于定义**自定义补全规则**的核心命令。它允许用户为特定命令或函数指定补全行为（例如参数、选项、文件类型等），是 Zsh 强大自动补全功能的关键工具之一。

## 1、基本语法

```bash
compdef [补全函数名] [命令名]
```

- **补全函数名**：负责生成补全候选列表的 Zsh 函数。
- **命令名**：需要添加补全的目标命令。

------

## 2、为自定义命令添加补全

假设有一个命令 `mycmd`，希望输入 `mycmd [TAB]` 时补全选项 `start`、`stop`、`restart`：

```bash
# 定义补全函数
_mycmd() {
    local -a subcmds
    subcmds=('start:Start service' 'stop:Stop service' 'restart:Restart service')
    _describe 'command' subcmds
}

# 将补全函数关联到命令
compdef _mycmd mycmd
```

- `_describe` 是 Zsh 内置的补全工具函数，用于生成带描述的候选列表。

## 3、补全文件类型

为命令 `convert` 补全指定扩展名的文件（如 `.png` 和 `.jpg`）：

```bash
_convert() {
    _arguments '1: :->input' '2: :->output'
    case $state in
        input) _files -g "*.png *.jpg" ;;
        output) _files ;;
    esac
}
compdef _convert convert
```

- `_arguments` 定义命令参数的位置和逻辑。
- `_files` 用于补全文件路径，`-g` 指定通配符过滤。

------

## 4、补全函数的编写

补全函数通常以 `_命令名` 命名（如 `_docker`），并遵循 Zsh 补全系统的规范。常用工具函数：

| 函数/命令    | 作用                             |
| :----------- | :------------------------------- |
| `_arguments` | 定义命令参数和选项的补全规则     |
| `_values`    | 补全键值对（如 `--format=json`） |
| `_files`     | 补全文件路径                     |
| `_describe`  | 补全带描述的候选列表             |
| `_message`   | 显示自定义提示信息               |

## 5、补全脚本的存放位置

Zsh 会自动加载以下目录中的补全脚本：


- `~/.zsh/completions/`：用户自定义补全
- `/usr/local/share/zsh/site-functions/`：系统级补全（如 Homebrew 安装的补全）
- `$fpath`：Zsh 的函数路径（通过 `echo $fpath` 查看）

将补全脚本放在上述目录后，重启 `Zsh` 或运行 `compinit` 即可生效。

## 6、调试与重载

- **手动重载补全**

  ```bash
  unfunction _mycmd && autoload -U _mycmd  # 重新加载补全函数
  compdef _mycmd mycmd                     # 重新关联命令
  ```

- **调试补全函数**

  ```bash
  zstyle ':completion:*' verbose yes      # 显示详细补全信息
  tracecomp mycmd                         # 跟踪补全过程
  ```