# VSCode常用设置

## 1、集成Cygwin中的zsh

```json
{
	"terminal.integrated.profiles.windows": {
        "Cygwin-zsh": {
            "path": [
                "C:\\Softwares\\cygwin\\bin\\zsh.exe"
            ],
            "args": [],
            "icon": "terminal-bash"
        }
    },
    "terminal.integrated.defaultProfile.windows":"Cygwin-zsh",
}
```

## 2、集成终端下复制粘贴设置

```json
{
	"terminal.integrated.copyOnSelection": true,
    "terminal.integrated.rightClickBehavior": "paste",
}
```

## 3、`Remote SSH `插件指定`ssh config`路径

```json
{
	"remote.SSH.configFile": "C:\\Softwares\\cygwin\\home\\Maker\\.ssh\\config",
}
```

## 4、关闭发生遥测数据

```json
{
    "redhat.telemetry.enabled": false,
    "telemetry.telemetryLevel": "off",
}
```

## 5、控制搜索结果是否显示行号

```json
{
    "search.showLineNumbers": true,
}
```

## 6、文件自动保存

```json
{
    "files.autoSave": "afterDelay",
}
```

## 7、资源管理器文件夹排列间隔宽度

```json
{
    "workbench.tree.indent": 25,
}
```

