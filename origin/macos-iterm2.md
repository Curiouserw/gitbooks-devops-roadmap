

# Iterm2快捷调优

## 1、单词跳跃

`Profiles` -> `Default` -> `Keys` -> `Key Mapings`

Option + 右方向键 (单词向右跳跃移动）：Action => `（Send Keystrokes）Send Escape Sequence`   Esc+ => `f`

Option + 左方向键 (单词向右跳跃移动）：Action => `（Send Keystrokes）Send Escape Sequence`   Esc+ => `b`

Option + Delete键（单词跳跃删除）： Action => `（Send Keystrokes）Send Hex Codes: 0x17 或者 0x1b 0x08 `   

Command + Delete键（整行删除）： Action => `（Send Keystrokes）Send Hex Codes: 0x15`

<img src="../assets/iterm2-word-move-delete-1.png" style="zoom:80%;" />

## 2、右键粘贴

`Preferences` -> `Pointer` -> `Bindings`

<img src="../assets/iterm2-paste-from-clip-by-right-button.png" style="zoom: 80%;" />

## 3、显示系统监控状态

`Profiles` -> `Default` -> `Session` -> `Status bar enabled`

<img src="../assets/iterm2-status-bar-1.png" style="zoom:80%;" />

时间组件参数配置

<img src="../assets/iterm2-status-bar-2.png" style="zoom:67%;" />

## 4、导入配色方案

配色主题网站：https://iterm2colorschemes.com/

个人的配色方案：[Curiouser.itermcolors](../assets/Curiouser.itermcolors)

![](../assets/iterm2-terminal-effect-1.png)

