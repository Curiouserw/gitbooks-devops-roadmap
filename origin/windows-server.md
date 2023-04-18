# Windows Server 管理

# 一、Windows Server 2012 R2 找回管理员密码

使用系统ISO光盘启动，在安装界面打开CMD（Shift+F10打开），替换放大镜程序 `C:\WINDOWS\system32\magnify.exe`为` C:\WINDOWS\system32\cmd.exe`。重启，在锁屏桌面打开放大镜(实际打开的是CMD)，使用`net`命令重制管理员密码。

1. 进BIOS设置开机从系统ISO光盘启动，启动进入系统安装程序后按`Shift+F10`进入cmd

   ![](../assets/windows-server-0.png)

2. 在cmd中替换放大镜程序，然后重启系统(从原系统盘启动)

   X是光盘路径，C盘是系统保留的100M的那个分区，D盘才是我们真正意义上的系统盘（如果有保留的话）

   ```bash
   # 备份原始放大镜程序
   copy D:\WINDOWS\system32\magnify.exe D:\WINDOWS\system32\magnify.exe.bak
   # 替换放大镜程序
   copy D:\WINDOWS\system32\cmd.exe D:\WINDOWS\system32\magnify.exe
   ```

3. 在登录页面打开放大镜，然后就在弹出的CMD中使用`net`命令重置管理员用户

   ```bash
   net user Administrator newpassword
   ```

4. 然后就可以使用新密码进行登录啦

5. 重制密码登录后再进行1～2操作，恢复原始放大镜程序。



**参考**

1. https://blog.csdn.net/njuptxiao/article/details/85098666
2. https://blog.51cto.com/ilyncsteven/2107216