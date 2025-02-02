# AppleScript

# 一、简介

## 1. **变量**

AppleScript 中的变量定义直接使用 `set` 关键字。

```
set myVar to "Hello, World!"
set number to 10
```

- `to` 是赋值关键字。
- AppleScript 支持基本数据类型，如字符串、数字、列表、记录等。

## 2. **数据类型**

- **字符串**：使用双引号

  ```bash
  set greeting to "Hello, AppleScript!"
  ```

- **数字**：支持整数和浮点数

  ```bash
  set number to 42
  set pi to 3.1415
  ```

- **布尔值**：`true` 或 `false`

  ```bash
  set isRunning to true
  ```

- **列表**：类似于数组，用花括号 `{}` 定义。

  ```bash
  set colors to {"red", "green", "blue"}
  ```

- **记录**：类似于字典，用花括号包含键值对定义。

  ```bash
  set person to {name:"Alice", age:30}
  ```

## 3. **注释**

使用 `--` 添加单行注释，`(* ... *)` 添加多行注释。

```bash
-- 这是一个单行注释
(* 这是
   多行注释 *)
```

## 4. **条件语句**

使用 `if...then...else` 结构。

```bash
set age to 18
if age ≥ 18 then
    display dialog "Adult"
else
    display dialog "Minor"
end if
```

## 5. repeat循环

### ①无限循环

```bash
repeat
    -- 无限循环
end repeat
```

### ②固定次数循环

```bash
repeat 5 times
    display dialog "Hello!"
end repeat
```

### ③遍历列表

```bash
set colors to {"red", "green", "blue"}
repeat with color in colors
    display dialog color
end repeat
```

### ④条件循环

```bash
set count to 1
repeat until count > 5
    display dialog count
    set count to count + 1
end repeat
```

## 6. **处理字符串**

AppleScript 提供一些简单的字符串操作。

```bash
set text1 to "Hello"
set text2 to "World"
set greeting to text1 & " " & text2 -- 连接字符串，结果为 "Hello World"
```

## 7. **函数和处理程序**

AppleScript 中的函数称为“处理程序”（handlers），用 `on` 定义。

```bash
on greet(name)
    display dialog "Hello, " & name
end greet

greet("Alice")  -- 调用处理程序
```

## 8. **调用系统命令**

`do shell script` 用于执行 shell 命令。

```bash
set result to do shell script "date"
display dialog result
```

## 9. **应用控制**

`tell` 语句用于控制应用程序，可以打开、操作窗口等。

```bash
tell application "Finder"
    activate  -- 激活 Finder 应用
    set myFolder to folder "Documents" of home
    open myFolder
end tell
```

## 10. 错误处理

可以使用 try...on error 结构来捕获错误。

```bash
try
   set result to 10 / 0  -- 这将引发错误
on error errMsg
   display dialog "Error: " & errMsg
end try
```

# 二、示例

## 1、**显示对话框**

```bash
display dialog "Hello, World!" buttons {"OK", "Cancel"} default button "OK"
```

## 2、获取用户输入

```bash
set userName to text returned of (display dialog "Enter your name:" default answer "")
display dialog "Hello, " & userName
```

## 3、控制 Safari 打开 URL

```bash
tell application "Safari"
    open location "https://www.apple.com"
    activate
end tell
```

## 4、检查文件是否存在

```bash
tell application "Finder"
    if exists file "Macintosh HD:Users:yourusername:Documents:example.txt" then
        display dialog "File exists."
    else
        display dialog "File does not exist."
    end if
end tell
```

## 5、获取剪贴板内容

```bash
set clipboardContent to the clipboard as text
display dialog clipboardContent
```

## 6、批量重命名文件

```bash
tell application "Finder"
    set myFolder to folder "Documents:MyFiles" of home
    set fileList to every file of myFolder
    repeat with aFile in fileList
        set name of aFile to "Renamed " & (get name of aFile)
    end repeat
end tell
```

## 7、弹窗选取多个图片

```bash
tell application "Finder"
    -- 弹出 Finder 文件选择窗口以选择第一张图片
    set firstImage to choose file of type {"public.image"} with prompt "请选择第一张图片" without multiple selections allowed
    set firstImagePath to POSIX path of firstImage

    -- 提示用户继续选择第二张图片
    display dialog "已选取第一张图片。请继续选择第二张图片。" buttons {"OK"} default button "OK"

    -- 再次弹出 Finder 文件选择窗口以选择第二张图片
    set secondImage to choose file of type {"public.image"} with prompt "请选择第二张图片" without multiple selections allowed
    set secondImagePath to POSIX path of secondImage

    -- 返回选中的两个文件的路径
    return {firstImagePath, secondImagePath}
end tell
```

# 二、osascript

`osascript`是macOS上的一个命令行工具，用于执行AppleScript脚本或者JavaScript脚本。它的名称来源于”Open Scripting Architecture”（OSA）