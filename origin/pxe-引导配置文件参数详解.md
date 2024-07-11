



PXE 配置文档：https://wiki.syslinux.org/wiki/index.php?title=Config



## pxelinux搜索配置文件的顺序

```yaml
Client 192.168.1.25:49152 /tftpboot/pxelinux.cfg/564d4f19-fe3b-5ae0-f81a-ed57dfa1f0ae, No Such File/No Access
# pxelinux.cfg/01-客户机网卡MAC地址
Client 192.168.1.25:49153 /tftpboot/pxelinux.cfg/01-aa-bb-cc-dd-ee-ff, No Such File/No Access
# pxelinux.cfg/C0A80119 为 192.168.1.25的16进制表示（可通过syslinux软件包提供的gethostip命令将10进制的IP转换为16进制）
Client 192.168.1.25:49154 /tftpboot/pxelinux.cfg/C0A80119, No Such File/No Access
Client 192.168.1.25:49155 /tftpboot/pxelinux.cfg/C0A8011, No Such File/No Access
Client 192.168.1.25:49156 /tftpboot/pxelinux.cfg/C0A801, No Such File/No Access
Client 192.168.1.25:49157 /tftpboot/pxelinux.cfg/C0A80, No Such File/No Access
Client 192.168.1.25:49158 /tftpboot/pxelinux.cfg/C0A8, No Such File/No Access
Client 192.168.1.25:49159 /tftpboot/pxelinux.cfg/C0A, No Such File/No Access
Client 192.168.1.25:49160 /tftpboot/pxelinux.cfg/C0, No Such File/No Access
Client 192.168.1.25:49161 /tftpboot/pxelinux.cfg/C, No Such File/No Access
Client 192.168.1.25:49162 /tftpboot/pxelinux.cfg/default, 4 Blocks Served


Client 192.168.1.25:49163 /tftpboot/boot.msg, No Such File/No Access
Client 192.168.1.25:49164 /tftpboot/vesamenu.c32, 110 Blocks Served
Client 192.168.1.25:49165 /tftpboot/pxelinux.cfg/default, 4 Blocks Served
Client 192.168.1.25:49166 /tftpboot/splash.jpg, No Such File/No Access
Client 192.168.1.25:49167 /tftpboot/Ubuntu/20.04.6/vmlinuz, 9718 Blocks Served
Client 192.168.1.25:49168 /tftpboot/Ubuntu/20.04.6/initrd, 62731 Blocks Served
```







```ini
# 默认启动的是 'label ks' 中标记的启动内核
default ks
# 显示 'boot: ' 提示符。为 '0' 时则不提示，将会直接启动 'default' 参数中指定的内容。
prompt 1
# 在用户输入之前的超时时间，单位为 1/10 秒，60则为6秒
timeout 60
# 显示某个文件的内容，注意文件的路径。默认是在/var/lib/tftpboot/ 目录下。也可以指定位类似 '/install/boot.msg'这样的，路径+文件名。
display boot.msg  
# 按下F1键后显示的文件，可以设置F1~F12
F1 boot.msg
F2 options.msg
F3 general.msg
F4 param.msg
F5 rescue.msg

# label指定你在 'boot:' 提示符下输入的关键字，比如boot: linux[ENTER]，这个会启动'label linux'下标记的kernel和initrd.img 文件。
label linux 
  menu label ^1> Install CentOS 7.9.2009 x86_64
  menu default
  # kernel 指定启动内核文件的路径，路径是相对于/var/lib/tftpboot/的。仅支持从 TFTP 协议服务中加载该文件，不支持HTTP协议等其他协议服务。
  kernel images/CentOS-7/vmlinuz-7.9
  # append: 指定追加给内核的参数，能够在grub 里使用的追加给内核的参数，在这里也都可以使用。
  # initrd: 指定初始RAM磁盘文件的位置。默认根路径：/var/lib/tftpboot/。不支持HTTP协议文件路径。
  # ks： 指定CentOS/Redhat发行版本的无人值守 kickstart配置文件的路径。支持从NFS, HTTPS, HTTP, FTP协议服务中获取该文件
  append initrd=images/CentOS-7/initrd-7.9.img ks=http://192.168.1.1/CentOS/7.9.2009.cfg
  
label text 
  kernel vmlinuz 
  append initrd=initrd.img text 
  
label ks 
  kernel vmlinuz 
  # 告诉系统，从哪里获取ks.cfg文件 
  append 
  
label local 
  localboot 1 
  
label memtest86 
  kernel memtest 
  append -
```





格式：

> menu color <Item> <ANSI Seq.> <foreground> <background> <shadow type>

```bash
menu color border      30;44      #00000000 #00000000 none  菜单边框   透明
menu color title       1;36;44    #FFFFFFFF #00000000 none  顶部标题文字颜色   白色
menu color sel         30;37      #FFFF0000 #00000000 none  选定子菜单颜色     红色
menu color unsel       37;44      #FFFFFFFF #00000000 none  未选定子菜单颜色   白色
menu color help        37;40      #FFFFFFFF #00000000 none  菜单注释文字颜色   白色
menu color cmdline     37;40      #FFFF0000 #00000000 none  TAB命令行颜色      红色
menu color msg07       37;40      #FFFFFF00 #00000000 none  F1显示颜色         黄
menu color timeout     37;40      #FFFF0000 #00000000 none  剩余时间秒数颜色   红色
menu color timeout_msg 37;40      #FFFFFF00 #00000000 none  剩余时间字体颜色   黄
menu color hotkey      1;37;44    #ffffff00 #00000000 none  快捷键颜色         黄
menu color hotsel      1;7;37;40  #FFFF0000 #00000000 none  选定快捷键颜色     红色
```





示例

```yaml
default vesamenu.c32
prompt 0
timeout 60
display boot.msg
menu background splash.jpg
menu title ########## Curiouser PXE Boot Menu ##########

label CentOS7.5.1804
  menu label ^1> Install CentOS 7.5.1804 x86_64
  kernel vmlinuz-7.5
  append initrd=initrd-7.5.img text ks=http://192.168.1.80/CentOS7.5.1804.cfg

label CentOS7.7.1908
  menu label ^2> Install CentOS 7.7.1908 x86_64
  menu default
  kernel vmlinuz-7.7
  append initrd=initrd-7.7.img text ks=http://192.168.1.80/CentOS7.7.1908.cfg
```



https://github.com/zhangguanzhang/Installer/blob/master/docs/pxe.md

# 参考

1. https://docs.centos.org/en-US/centos/install-guide/pxe-server/#chap-installation-server-setup
2. https://blog.51cto.com/4690837/2333165
3. https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/6/html/installation_guide/ch-boot-x86#sn-boot-menu-x86
4. https://github.com/zhangguanzhang/Installer/blob/master/docs/pxe.md