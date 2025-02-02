# 优化Shell脚本语法

# 一、`[[ ]]`进行条件判断

- `[[ ]]` 是 Bash 内置命令，性能比 `[]` 高

- `[[ ]]` 支持复杂条件和模式匹配，不需要像 `[]` 处理反斜杠转义
- `[[ ]]` 支持 `&&` 和 `||` 作为逻辑运算符，而  `[]` 需要使用 `-a` 和 `-o` 进行逻辑测试
- 在 `[[ ]]` 中，可以直接使用 `<` 和 `>` 进行字符串比较，而 `[` 中必须使用 `-lt` 和 `-gt`

# 二、`$(<file)` 读取小文件内容

```bash
content=$(< file.txt)
echo "$content"
```

# 三、while read逐行处理文件

```bash
while IFS= read -r line; do
  echo "$line"
done < file.txt
```

# 四、mapfile读取文件所有行到数组

```bash
mapfile -t lines < file.txt
for line in "${lines[@]}"; do
  echo "$line"
done
```

# 五、使用数组

```bash
items=("apple" "banana" "cherry")
for item in "${items[@]}"; do
    echo "$item"
done
```

# 六、`或||`实现异常处理

```bash
command1 || { echo "command1 failed"; exit 1; }
```

# 七、信号处理

## 1、语法

```bash
trap '命令' 信号

# 常见信号：
#   SIGINT (2)：通常是按 Ctrl+C 时触发的信号。
#   SIGTERM (15)：终止进程信号，常用于正常的进程终止。
#   EXIT：脚本退出时触发，不管是正常退出还是因错误中止。
#   SIGHUP (1)：终端挂起时触发，比如关闭终端窗口。
#   SIGKILL (9)：强制杀死进程信号，无法被捕获或忽略。
```

## 2、捕获多个信号

```bash
trap 'echo "Received SIGINT or SIGTERM. Exiting..."; exit' SIGINT SIGTERM

echo "Press Ctrl+C or send SIGTERM to terminate."
while true; do
  sleep 1
done
# 在这个例子中，SIGINT（通过 Ctrl+C 触发）和 SIGTERM 信号都会被捕获，脚本会执行 echo "Received SIGINT or SIGTERM" 然后退出。
```

## 3、忽略信号

```bash
trap '' SIGINT

echo "Try pressing Ctrl+C, but nothing will happen."
while true; do
  sleep 1
done

# 通过将 trap '' 与信号配合使用，可以忽略某个信号。例如，按 Ctrl+C 时（发送 SIGINT），脚本不会中断。
```

# 八、设置默认行为

## 1、`set -o noclobber`禁止覆盖文件

- **`noclobber`** 保护现有文件不被重定向覆盖，适合在不希望文件被意外修改时使用。
-  如果文件存在，会报错`bash: file.txt: cannot overwrite existing file`

- 可以用 `>|` 强制覆盖文件，即使 `noclobber` 启用。

## 2、`set -e`防止级联错误

使用 `set -e` 确保脚本在任何命令失败时立即退出，防止级联错误。

# 九、校验变量有效性

## 1、校验输入变量

```bash
if [[ -z "$1" ]]; then
    echo "Usage: $0 <argument>"
    exit 1
fi
```
