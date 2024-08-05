# PowerShell

# 一、简介

PowerShell [是基于.NET](http://xn--6kq92uzzl.net/) Framework，[因此它支持.NET](http://xn--1bsu1ioyh0lc72l.net/) Framework 中的所有对象类型和方法。这使得它可以访问广泛的系统资源和服务，包括注册表、文件系统、进程和网络。

PowerShell 还支持管道和别名，这使得它非常适合于自动化任务。管道允许你将一个命令的输出作为另一个命令的输入，而别名允许你使用简短的名称来引用命令。

# 二、自带函数

- **Get-ChildItem (dir)：** 获取指定目录下的文件和文件夹。
- **Set-Location (cd)：** 更改当前目录。
- **New-Item (mkdir)：** 创建新文件夹。
- **Remove-Item (rmdir)：** 删除文件或文件夹。
- **Copy-Item (copy)：** 复制文件或文件夹。
- **Move-Item (move)：** 移动文件或文件夹。
- **Rename-Item (ren)：** 重命名文件或文件夹。
- **Get-Process：** 获取正在运行的进程。
- **Start-Process：** 启动新进程。
- **Stop-Process：** 停止正在运行的进程。
- **Get-Service：** 获取已安装的服务。
- **Start-Service：** 启动服务。
- **Stop-Service：** 停止服务。
- **Get-EventLog：** 获取事件日志。
- **New-EventLog：** 创建新事件日志。
- **Clear-EventLog：** 清除事件日志。
- **Get-NetworkAdapter：** 获取网络适配器。
- **Enable-NetworkAdapter：** 启用网络适配器。
- **Disable-NetworkAdapter：** 禁用网络适配器。
- **Get-NetIPConfiguration：** 获取网络适配器的 IP 配置。
- **Set-NetIPConfiguration：** 设置网络适配器的 IP 配置。
- **Get-Content：** 获取文件的内容。
- **Set-Content：** 设置文件的内容。
- **Add-Content：** 向文件末尾追加内容。
- **Remove-Content：** 清空文件的内容。
- **Compare-Object：** 比较两个对象之间的差异。
- **Select-Object：** 从对象中选择特定的属性。
- **Where-Object：** 根据条件过滤对象。
- **Sort-Object：** 对对象进行排序。
- **Group-Object：** 将对象分组。
- **ForEach-Object：** 对每个对象执行指定的命令。
- **New-Object：** 创建新对象。
- **Invoke-Command：** 在远程计算机上执行命令。
- **Get-Date：** 获取当前日期和时间。
- **Set-Date：** 设置当前日期和时间。
- **Get-Random：** 获取随机数。
- **ConvertTo-Html：** 将对象转换为 HTML 格式。
- **ConvertFrom-Html：** 将 HTML 格式转换为对象。
- **ConvertTo-Json：** 将对象转换为 JSON 格式。
- **ConvertFrom-Json：** 将 JSON 格式转换为对象。

# 三、常用脚本或命令

## 1. 获取系统信息脚本
```bash
Get-ComputerInfo | Format-List
```
## 2. 获取正在运行的进程脚本
```bash
Get-Process | Format-Table -Property Name, Id, ProcessName, MainWindowTitle
```
## 3. 停止正在运行的服务脚本
```bash
Stop-Service -Name "ServiceName"
```
## 4. 创建新文件夹脚本
```bash
New-Item -Path "C:\NewFolder" -ItemType Directory
```
## 5. 复制文件脚本
```bash
Copy-Item -Path "C:\SourceFile.txt" -Destination "C:\DestinationFolder\DestinationFile.txt"
```
## 6. 重命名文件脚本
```bash
Rename-Item -Path "C:\OldFileName.txt" -NewName "C:\NewFileName.txt"
```
## 7. 删除文件脚本
```bash
Remove-Item -Path "C:\FileToDelete.txt"
```
## 8. 创建新注册表项脚本
```bash
New-Item -Path "HKLM:\Software\NewKey" -RegistryHive LocalMachine
```
## 9. 设置注册表值脚本
```bash
Set-ItemProperty -Path "HKLM:\Software\NewKey" -Name "ValueName" -Value "ValueData"
```
## 10. 获取注册表值脚本
```bash
Get-ItemProperty -Path "HKLM:\Software\NewKey" -Name "ValueName"
```
## 11. 删除注册表项脚本
```bash
Remove-Item -Path "HKLM:\Software\NewKey" -Recurse
```
## 12. 发送电子邮件脚本
```bash
Send-MailMessage -To "recipient@example.com" -From "sender@example.com" -Subject "Test Email" -Body "This is a test email."
```
## 13. 下载文件脚本
```bash
Invoke-WebRequest -Uri "https://example.com/file.txt" -OutFile "C:\DownloadedFile.txt"
```
## 14. 解压压缩文件脚本
```bash
Expand-Archive -Path "C:\CompressedFile.zip" -Destination "C:\ExtractedFiles"
```
## 15. 压缩文件脚本
```bash
Compress-Archive -Path "C:\FilesToCompress" -Destination "C:\CompressedFile.zip"
```

## 16. 测试网络连通性

- 测试 IP 地址 Ping 值
- 测试 IP 地址端口是否打开
- 测试指定 URL 是否可访问

> test.ps1

```powershell
param(
    [string]$IPAddress,
    [int]$Port,
    [string]$URL
)

function Test-Ping {
    param(
        [string]$IP
    )
    $pingResult = Test-Connection -ComputerName $IP -Count 1 -ErrorAction SilentlyContinue
    if ($pingResult) {
        Write-Output "$IP 可达！"
    } else {
        Write-Output "$IP 不可达！"
    }
}

function Test-Port {
    param(
        [string]$IP,
        [int]$Port
    )
    $tcpClient = New-Object System.Net.Sockets.TcpClient
    try {
        $tcpClient.Connect($IP, $Port)
        Write-Output "$IP 的 $Port 端口可达！"
    } catch {
        Write-Output "$IP 的 $Port 端口不可达！"
    } finally {
        $tcpClient.Close()
    }
}

function Test-URL {
    param(
        [string]$URL
    )
    $webRequest = Invoke-WebRequest -Uri $URL -ErrorAction SilentlyContinue
    if ($webRequest) {
        Write-Output "URl可访问！"
    } else {
        Write-Output "URl不可访问"
    }
}

if ($IPAddress) {
    Test-Ping -IP $IPAddress
}
if ($Port) {
    if (-not $IPAddress) {
        Write-Error "测试端口连通性，请至少提供一个 IP 地址."
    } else {
        Test-Port -IP $IPAddress -Port $Port
    }
}
if ($URL) {
    Test-URL -URL $URL
}
```

```bash
.\test.ps1 -IPAdress 192.168.1.1 -Port 8001 -URL https://www.baidu.com
```

## 17、安装OpenSSH Server

- 当`程序与功能`里面不显示 OpenSSH服务端的安装程序时，可使用`管理员权限 Powershell` 手动安装

  ```bash
  # 查询
  Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'
  # 安装OpenSSH客户端
  Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
  # 安装OpenSSH服务端
  Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
  
  # 设置开机自启
  Set-Service -Name sshd -StartupType 'Automatic'
  # 启动sshd服务
  Start-Service sshd
  
  # 检查OpenSSH服务器的状态
  Get-Service -Name sshd
  #卸载服务端(需要重启)
  Remove-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
  ```

- 配置文件：**C:\ProgramData\ssh\sshd_config**

- 配置`authorized_keys`公钥登录

  ```bash
  # 允许公钥授权访问，确保条目不被注释
  PubkeyAuthentication yes
  # 指定认证授权文件存放路径
  AuthorizedKeysFile .ssh/authorized_keys
  
  # Match Group administrators
  # 注释掉默认授权文件位置
  # AuthorizedKeysFile __PROGRAMDATA__/ssh/administrators_authorized_keys
  ```

- 更改`C:\Users\用户名\.ssh\authorized_keys`文件权限

  - 属性->安全->高级->（左下角）**禁用继承**
  - 弹框选择“**将已继承的权限转换为此对象的显式权限**”
  - 在**权限条目**中删除除了**SYSTEM**和**当前用户**外的所有权限，然后应用并退出

  ```bash
  icacls.exe "C:\Users\用户\.ssh\authorized_keys" /inheritance:r /grant "Administrators:F" /grant "SYSTEM:F"
  ```

- 检查**OpenSSH Authentication Agent**服务是否正常运行。没有运行的话需要启动改服务。

- 检查防火墙是否放行 SSH 服务的端口 (默认端口22，建议修改)

  ```bash
  # 确认防火墙规则，一般在安装时会配置好
  Get-NetFirewallRule -Name *ssh*
  
  # 若安装时未添加防火墙规则"OpenSSH-Server-In-TCP"，则通过以下命令添加
  New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
  ```

- 重启服务：`restart-service sshd`
