



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





# 参考

1. https://docs.centos.org/en-US/centos/install-guide/pxe-server/#chap-installation-server-setup
2. https://blog.51cto.com/4690837/2333165
3. https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/6/html/installation_guide/ch-boot-x86#sn-boot-menu-x86