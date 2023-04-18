# Sublime Text使用总结

# 一、简介

Sublime Text是一款具有代码高亮、语法提示、自动完成且反应快速的编辑器软件

Sublime Text具有漂亮的用户界面和强大的功能，例如代码缩略图，[Python](https://baike.baidu.com/item/Python)的插件，代码段等。还可自定义键绑定，菜单和工具栏。Sublime Text 的主要功能包括：拼写检查，书签，完整的 Python API ， Goto 功能，即时项目切换，多选择，多窗口等等。Sublime Text 是一个跨平台的编辑器，同时支持[Windows](https://baike.baidu.com/item/Windows)、[Linux](https://baike.baidu.com/item/Linux)、[Mac OS X](https://baike.baidu.com/item/Mac OS X)等操作系统。

# 二、安装

Sublime Text官网：http://www.sublimetext.com/3

# 三、插件管理器Package Control

Package Control：https://packagecontrol.io/installation

- 使用`Ctrl +` `打开Sublime Text控制台
- 将下面的代码粘贴到控制台里

**Sublime Text 3**

```python
import urllib.request,os,hashlib; h = '6f4c264a24d933ce70df5dedcf1dcaee' + 'ebe013ee18cced0ef93d5f746d80ef60'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
```

**Sublime Text 2**

```python
import urllib2,os,hashlib; h = '6f4c264a24d933ce70df5dedcf1dcaee' + 'ebe013ee18cced0ef93d5f746d80ef60'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); os.makedirs( ipp ) if not os.path.exists(ipp) else None; urllib2.install_opener( urllib2.build_opener( urllib2.ProxyHandler()) ); by = urllib2.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); open( os.path.join( ipp, pf), 'wb' ).write(by) if dh == h else None; print('Error validating download (got %s instead of %s), please try manual install' % (dh, h) if dh != h else 'Please restart Sublime Text to finish installation')
```

**给Package Controller设置代理**

1. Sublime Text > `Preferences > Package Settings > Package Control > Settings - User` 

2. 编辑 `Package Control.sublime-settings`，添加两行:

   ```bash
   "http_proxy": "http://代理IP地址:3128",
   "https_proxy": "http://代理IP地址:3128",
   ```

**解决无法安装插件问题**

使用第三方Channel，见[附件](../assets/SublimeTest-channel_v3.json)

Preference-->Package Settings-->Package Control-->Settings User

```json
{
	"bootstrapped": true,
	"channels":
	[
		"D:/Sublime Text 3/data/channel_v3.json"
	]
}
```

# 四、常用快捷键

| 快捷键         | 功能                                                         |
| -------------- | ------------------------------------------------------------ |
| **Ctrl+H**         | 查找替换                                                     |
| **Ctrl+F**         | 查找内容                                                     |
| **Ctrl+Shift+F** | 在文件夹内查找内容，可进行替换                               |
| **Ctrl+L**     | 选择一行                                                     |
| **Ctrl+Shift+D** | 复制当前行到下行                                             |
| **Ctrl+K+U**   | 大写光标所在词                                               |
| **Ctrl+K+L**   | 小写光标所在词                                               |
| **Ctrl+Shift+D** | 复制光标所在整行到下一行                                     |
| **Ctrl+Shift+L** | 鼠标选中多行（按下快捷键），即可同时编辑这些行               |
| **Ctrl+Shift+↑/↓** | 可以移动此行代码，与上/下行互换                              |
| **Ctrl+Shift+←/→** | 向右/向右单位性地选中文本                                    |
| **Alt+Shift+1** | 窗口分屏，恢复默认1屏（非小键盘的数字）                      |
| **Alt+Shift+2** | 左右分屏-2列                                                 |
| **Alt+Shift+3** | 左右分屏-3列                                                 |
| **Alt+Shift+4** | 左右分屏-3列                                                 |
| **Alt+Shift+5** | 等分4屏                                                      |
| **Alt+Shift+8** | 垂直分屏-2屏                                                 |
| **Alt+Shift+9** | 垂直分屏-3屏                                                 |
| **Ctrl+K+B**   | 开启/关闭侧边栏                                              |
| **Ctrl+/**     | 注释单行                                                     |
| **Ctrl+Shift+/** | 注释多行                                                     |
| **Ctrl+K+K**   | 从光标处开始删除代码至行尾                                   |
| **Ctrl+Shift+K** | 删除整行                                                     |
| **Tab**        | 向右缩进                                                     |
| **Shift+Tab**  | 向左缩进                                                     |
| **Ctrl+J**     | 合并选中的多行代码为一行                                     |
| **Shift+↑/↓/←/→** | 向上/下/左/右选中文本                                        |
| **Ctrl+K+0**   | 展开所有折叠代码                                             |
| **Ctrl+M**     | 光标移动至括号内结束或开始的位置                             |
| **Ctrl+Shift+M** | 选择括号内的内容                                             |
| **Ctrl+D**     | 选中光标所占的文本，继续操作则会选中下一个相同的文本         |
| **Alt+F3**     | 选中文本按下快捷键，即可一次性选择全部的相同文本进行同时编辑 |
| **Ctrl+G**     | 跳转到第几行                                                 |
| **Ctrl+Shift+W** | 关闭所有打开文件                                             |
| **Ctrl+Shift+V** | 粘贴并格式化                                                 |
| **Ctrl+X**         | 删除当前行                                                   |
| **Ctrl+Z**     | 撤销                                                         |
| **Ctrl+Y**     | 恢复撤销                                                     |
| **Ctrl+U**     | 软撤销                                                       |
| **Ctrl+T**     | 左右字母互换                                                 |
| **Ctrl+Tab**   | 按文件浏览过的顺序，切换当前窗口的标签页                     |
| **Ctrl+PageDown** | 向左切换当前窗口的标签页                                     |
| **Ctrl+PageUp** | 向右切换当前窗口的标签页                                     |
| **Ctrl+W**     | 关闭当前打开文件                                             |
| **Ctrl+Shift+W** | 关闭所有打开文件                                             |
| **Ctrl+Shift+P** | 打开命令面板                                                 |
| **Ctrl+：**    | 打开搜索框，自动带#，输入关键字，查找文件中的变量名、属性名等。 |
| **Ctrl+R**     | 打开搜索框，自动带@，输入关键字，查找文件中的函数名          |
| **Ctrl+P**     | 打开搜索框。<br/>1、输入当前项目中的文件名，快速搜索文件<br/>2、@和关键字，查找文件中函数名<br/>3、：和数字，跳转到文件中该行代码<br/>4、#和关键字，查找变量名 |



# 五、常用插件

| 插件名               | 功能                                                         | 描述                                                         |
| -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **DeleteBlankLines**     | 去除文本中的空白行                                           | Windows:<br/>    Ctrl+Alt+Backspace --> Delete Blank Lines<br/>    Ctrl+Alt+Shift+Backspace --> Delete Surplus Blank Lines<br/>Linux:<br/>    Ctrl+Alt+Backspace --> Delete Blank Lines<br/>    Ctrl+Alt+Shift+Backspace --> Delete Surplus Blank Lines |
| **ChineseLocalizations** | 汉化Sublime Text                                             | 请使用主菜单的 帮助/Language 子菜单来切换语言。 目前支持 简体中文 繁体中文 日本語。 要换回英语不需要卸载本插件，请直接从菜单切换英文。 |
| **HTML-CSS-JS Prettify** | HTML/CSS/JS代码格式化                                        | 需要安装NodeJS并设置node执行文件的路径<br>右键-->HTML-CSS-JS Prettify-->Set ‘node’ Path |
| **GBK Encoding Support** | 支持gbk编码                                                  |                                                              |
| **Alignment**        | 代码格式的自动对齐                                           | 默认快捷键Ctrl+Alt+A                                         |
| **Clipboard History** | 粘贴板历史记录，方便使用复制/剪切的内容                      | Ctrl+alt+v：显示历史记录<br/>Ctrl+alt+d：清空历史记录<br/>Ctrl+shift+v：粘贴上一条记录（最旧）<br/>Ctrl+shift+alt+v：粘贴下一条记录（最新） |
| **ConvertToUTF8**    | 编辑并保存目前编码不被 Sublime Text 支持的文件               |                                                              |
| **IMESupport**       | 支持中文输入法跟随光标                                       |                                                              |
| **AutoFileName**     | 自动完成文件名的输入，如图片选取                             |                                                              |
| **Trailing spaces**  | 检测并一键去除代码中多余的空格                               | 一键删除多余空格：CTRL+SHITF+T（需配置），更多配置请点击标题。快捷键配置：在Preferences / Key Bindings – User加上{ "keys": ["ctrl+shift+t"], "command": "delete_trailing_spaces" } |
| **FileDiffs**        | 比较当前文件与选中的代码、剪切板中代码、另一文件、未保存文件之间的差别。可配置为显示差别在外部比较工具，精确到行。 | 右键标签页，出现FileDiffs Menu或者Diff with Tab…选择对应文件比较即可 |
| **DocBlockr**        | 生成优美注释                                                 | 标准的注释，包括函数名、参数、返回值等，并以多行显示，手动写比较麻烦。输入/*、/**然后回车，还有很多用法，请参照https://sublime.wbond.net/packages/DocBlockr |
| **SideBarEnhancements** | 增强型侧边栏                                                 |                                                              |
| **Terminal**         | 接使用终端打开你的项目文件夹，并支持使用快捷键。             | 默认调用系统自带的`PowerShell` <br/>ctrl+shift+t 打开文件所在文件夹，<br/>ctrl+shift+alt+t 打开文件所在项目的根目录文件夹 |
| **SFTP**             | 快速编辑远程服务器上的文件                                   |                                                              |
