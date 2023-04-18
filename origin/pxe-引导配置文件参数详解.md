
```yaml
default ks 　　　　#默认启动的是 'label ks' 中标记的启动内核
prompt 1          #显示 'boot: ' 提示符。为 '0' 时则不提示，将会直接启动 'default' 参数中指定的内容。
timeout 6 　　　　 #在用户输入之前的超时时间，单位为 1/10 秒。

display boot.msg  #显示某个文件的内容，注意文件的路径。默认是在/var/lib/tftpboot/ 目录下。也可以指定位类似 '/install/boot.msg'这样的，路径+文件名。

F1 boot.msg 　　　 #按下 'F1' 这样的键后显示的文件。
F2 options.msg 
F3 general.msg 
F4 param.msg 
F5 rescue.msg 


label linux       #'label' 指定你在 'boot:' 提示符下输入的关键字，比如boot: linux[ENTER]，这个会启动'label linux' 下标记的kernel 和initrd.img 文件。
  kernel vmlinuz  #kernel 参数指定要启动的内核。
  append initrd=initrd.img #append 指定追加给内核的参数，能够在grub 里使用的追加给内核的参数，在这里也都可以使用。
  
label text 
  kernel vmlinuz 
  append initrd=initrd.img text 
  
label ks 
  kernel vmlinuz 
  append ks=http://192.168.111.130/ks.cfg initrd=initrd.img    #告诉系统，从哪里获取ks.cfg文件 
  
label local 
  localboot 1 
  
label memtest86 
  kernel memtest 
  append -
```

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

1. https://wiki.centos.org/zh/HowTos/PXE/PXE_Setup/Menus
2. https://blog.51cto.com/4690837/2333165
3. https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/6/html/installation_guide/ch-boot-x86#sn-boot-menu-x86