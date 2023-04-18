# 一、简介

kickstart文件中各部分(section)要遵循一定的顺序。每个部分中的项(Item)并不需要按照一定的顺序排列，除非有其他要求。各部分的顺序如下：

- 命令部分
- %packages部分
- %pre, %post, 以及%traceback部分 -- 这些部分的顺序可以任意排列
- %packages, %pre, %post以及%traceback部分需要以%end结束。
- 不要求的项(Item)可以被省略。
- 省略任何一个被要求的项将会导致安装程序向用户询问相关的问题，就像典型安装过程向用户询问那样。一旦用户给出了答案，安装过程将会继续自动进行，除非又遇到缺失的项。
- 以(#)开头的行作为注释行被忽略。
- 如果在kickstart安装中使用了不推荐的命令、选项或者语法，警告日志将会被记录到anaconda日志中。因为在一个或者两个发行版之间这些不推荐的项经常会被删掉，所以检查安装日志以确保没有使用这些项非常必要。当使用ksvalidator的时候，这些不推荐的项会导致错误。
- 如果选项后接等号（=），则必须指定一个值

**引用磁盘的特殊说明：**

```yaml
Kickstart一直通过设备节点名(例如 sda)来引用磁盘。Linux内核采用了更加动态的方法，设备名并不会在重启时保持不变。因此，这会使得在Kickstart脚本中引用磁盘变得复杂。为了满足稳定的设备命名，你可以在项(Item)中使用/dev/disk代替设备名。例如，你可以使用：
part / --fstype=ext4 --onpart=/dev/disk/by-path/pci-0000:00:05.0-scsi-0:0:0:0-part1
part / --fstype=ext4 --onpart=/dev/disk/by-id/ata-ST3160815AS_6RA0C882-part1
来代替：
part / --fstype=ext4 --onpart=sda1
这种方式提供了对磁盘的持久引用，因而比仅仅使用sda更加有意义。 这在大的存储环境中特别有意义。你也可以使用类似于shell的入口来应用磁盘。这种方式主要用来简化大的存储环境中clearpart以及ignoredisk命令的使用。例如，为了替代：
ignoredisk --drives=sdaa,sdab,sdac
你可以使用如下的入口：
ignoredisk --drives=/dev/disk/by-path/pci-0000:00:05.0-scsi-*
最后，如果想要在任何地方引用已经存在的分区或者文件系统（例如，在part --ondisk=中），你可以通过文件系统标签(label)或者UUID来进行。例如：
part /data --ondisk=LABEL=data
part /misc --ondisk=UUID=819ff6de-0bd6-4bf4-8b72-dbe41033a85b
```

# 二、必需选项

## bootloader ：指明引导程序(bootloader)如何被安装。引申Bootloader相关概念：CentOS系统启动流程

```yaml
# 建议给Bootloader设置密码以防止黑客修改系统启动项或者不授权登录系统  
--append= 指定内核参数，要指定多个参数，使用空格分隔。引导程序默认的参数是"rhgb quiet"。举例: "bootloader --location=mbr --append="hdd=ide-scsi ide=nodma"
--boot-drive= 指定安装bootloader到哪个磁盘上
--leavebootorder  防止安装程序更改 EFI 或者 ISeries/PSeries 系统中的现有可引导映像。
--driveorder= 指定在 BIOS 引导顺序中的首选驱动器。例如: bootloader --driveorder=sda,hda
--location= 指定引导记录的写入位置（在大多数情况下不需要指定这个选项），有效值如下
               1.mbr  默认选项。具体要看该驱动器是使用主引导记录（MBR）还是 GUID 分区表（GPT）方案：
                 a.在使用 GPT 格式化的磁盘中，这个选项会在 BIOS 引导分区中安装 stage 1.5 引导装载程序。
                 b.在使用 MBR 格式化的磁盘中，会在 MBR 与第一个分区之间的空白空间中安装 stage 1.5。
               2.partition  在包含内核的分区的第一个扇区中安装引导装载程序
               3.none -不安装引导装载程序。
--password= 如果使用GRUB2,则会将使用这个选项指定的密码设定为引导装载程序密码.这应用来限制对GRUB2 shell的访问,并可以跳过任意内核选项.如果指定密码,GRUB2还会询问用户名。该
            用户名总是root
--iscrypted 通常当使用 --password= 选项指定引导装载程序密码时，会将其以明文方式保存在 Kickstart 文件中。如果要加密此密码，可使用这个选项和一个加密的密码。
            请使用 grub2-mkpasswd-pbkdf2 命令生成加密的密码，输入要使用的密码，并将该命令的输出结果（以 grub.pbkdf2 开头的哈希符号）复制到 Kickstart 文件中
--timeout= 指定引导装载程序引导默认选项前等待的时间（以秒为单位）。
--default= 设定引导装载程序配置中的默认引导映像。
--extlinux 使用 extlinux 引导装载程序而不是 GRUB2。这个选项只能用于支持 extlinux 的系统。
--disabled 这个选项是 --location=none 的加强版。--location=none 只是简单地禁用 bootloader 安装，而 --disabled 则不仅禁用 bootloader 安装，也会禁用 bootloader 软件
           包的安装，从而节省了空间。
```

## keyboard：设置系统键盘类型

```yaml
--vckeymap= # 指定VConsole应该使用的字符映射表。<keymap>是字符映射表的文件名，和/usr/lib/kbd/keymaps目录下的文件名除去".map.gz"后相同。
--xlayouts=<layout1>,<layout2>,...,<layoutN> #指定 X 布局列表，该列表可使用逗号分开，无空格。接受与 setxkbmap(1) 相同格式的值，可以是 layout 格式（比如 cz），也可
                                             #以是 layout (variant) 格式（比如 cz (qwerty)）。
--switch=<layout1>,<layout2>,...,<layoutN> #指定布局切换选项（在多个键盘布局间切换的快捷方式）列表。必须使用逗号分开多个选项，无空格。接受值与 setxkbmap(1) 格式相同。
​
#示例使用 --xlayouts= 选项设置两个键盘布局（English (US) 和 Czech (qwerty)），并允许使用 Alt+Shift 在二者之间进行切换：
#  keyboard --xlayouts=us,'cz (qwerty)' --switch=grp:alt_shift_toggle
```

## lang：设置在安装过程中使用的语言以及系统的缺省语言

```yaml
示例：lang en_US
文本模式的安装过程不支持某些语言(主要是中文,日语,韩文和印度的语言).如果用lang命令指定这些语言中的一种,安装过程仍然会使用英语,但是系统会缺省使用指定的语言.
```

## part 或者 partition：在系统上创建一个分区。（install模式必须）

```yaml
part  <mntpoint>|swap|pv.id|rdid.id  options
   1.mntpoint:挂载点，是在创建普通分区时指定新分区挂载位置的项；挂载点需要格式正. 例如 /, /usr, /home
   2.swap 分区将被用作交换分区。为了自动决定交换分区的大小，可以使用--recommended选项。
   3.raid.<id> 表示创建的分区类型为raid型；必须用id号进行唯一区别；
   4.pv.<id> 表示所创建的分区类型为LVM型；必须用唯一id号进行区别；
  --size= 最小分区大小(MB)。这里可以指定一个整数值如500.不要在后面加MB。
  --grow 告诉分区增长以填满可用空间(如果有的话)，或者填满设置的最大值。注意，--grow并不支持RAID卷在上的分区。
  --maxsize= 分区被设置为grow时的最大分区大小(MB)。指定一个整数值，不要在后面加上MB。
  --noformat 告诉安装程序不格式化分区，和--onpart一起使用。
  --onpart= or --usepart= 把分区放在已经存在的设备上。使用"--onpart=LABEL=name"或者"--onpart=UUID=name" 来通过各自的标签(label)或uuid来指定一个分区。Stop (medium 
                        size).png  Anaconda也许会以特殊的顺序创建分区，所以使用标签比只用分区名要安全些
  --ondisk= or --ondrive= 强制在特定的磁盘上创建。
  --asprimary 强制分区作为主分区，否则会导致分区失败。
  --fsprofile= 为在该分区上创建文件系统的程序指定使用类型。使用类型定义了创建文件系统时各种各样的调整参数。为了让该选项能起作用，文件系统必须支持使用类型的概念，而且必须有
               一个配置文件列出所有可用的类型。对于ext2/3/4，配置文件位于/etc/mke2fs.conf。
  --fstype= 为分区设置文件系统类型。有效值包括ext4,ext3,ext2,btrfs,swap以及vfat。其它文件系统是否有效取决于传递给anaconda使能其它文件系统的命令行参数。
  --fsoptions= 为挂载文件系统指定自由格式的字符串选项。该字符串将会被拷贝到安装系统的/etc/fstab文件中并且应该被引号括起来。
  --label= 指定分区上创建的文件系统标签。如果所指定的标签已经被其它文件系统使用，新的标签将会被创建。
  --recommended 自动决定分区大小。
  --onbiosdisk= 强制在BIOS发现的特定磁盘上创建分区。
  --encrypted 说明该分区应该被加密
  --passphrase= 指定加密分区时指定的密码短语。如果没有上述--encrypted选项，该选项不起任何作用。如果没有密码短语被指定，将会使用系统范围内的默认密码短语，如果没有默认的，
                安装器会停下来提醒。
  --escrowcert=<url> 从<url>加载X.509认证。存储用证书加密过的数据加密密钥。只在--encrypted指定时有效。
  --backuppassphrase 只在--escrowcert指定时相关。除了存储数据加密密钥之外，产生一个随机密码短语并将其添加到该分区。然后使用--escrowcert指定的证书加密并存储于/root。如果
                     不止一个LUKS卷使用--backuppassphrase，它们将共享该密码短语。
例：
part  /boot  --fstype=“ext3” --size=100
part  swap  --fstype=“swap” –size=512  
part  /  --bytes-pre-inode=4096  --fstype=“ext4”--size=10000
part  /data    --onpart=/dev/sdb1  --noformat
part  raid.100  --size=2000
part  pv.100     --size=1000
```

## auth/authconfig：设置系统的授权验证选项。它只是authconfig程序的封装，因而所有被authconfig程序识别的选项都可以应用于auth命令。想要获取完整的列表，请参考authconfig手册。默认情况下，密码一般会被加密但并不会放在shadow文件中。

```yaml
--enablemd5,每个用户口令都使用md5加密.
--enablenis,启用NIS支持.在缺省情况下,--enablenis使用在网络上找到的域.域应该总是用--nisdomain=选项手工设置.
--nisdomain=,用在NIS服务的NIS域名.
--nisserver=,用来提供NIS服务的服务器(默认通过广播).
--useshadow或--enableshadow,使用屏蔽口令.
--enableldap,在/etc/nsswitch.conf启用LDAP支持,允许系统从LDAP目录获取用户的信息(UIDs,主目录,shell 等等).要使用这个选项,必须安装nss_ldap软件包.也必须用--ldapserver=和--ldapbasedn=指定服务器和base DN(distinguished name).
 --enableldapauth,把LDAP作为一个验证方法使用.这启用了用于验证和更改密码的使用LDAP目录的pam_ldap模块.要使用这个选项,必须安装nss_ldap软件包.也必须用--ldapserver=和--ldapbasedn=指定服务器和base DN.
--ldapserver=,如果指定了--enableldap或--enableldapauth,使用这个选项来指定所使用的LDAP服务器的名字.这个选项在/etc/ldap.conf文件里设定.
--ldapbasedn=,如果指定了--enableldap或--enableldapauth,使用这个选项来指定用户信息存放的LDAP目录树里的DN.这个选项在/etc/ldap.conf文件里设置.
--enableldaptls,使用TLS(传输层安全)查寻.该选项允许LDAP在验证前向LDAP服务器发送加密的用户名和口令.
--enablekrb5,使用Kerberos 5验证用户.Kerberos自己不知道主目录,UID或shell.如果启用了Kerberos,必须启用LDAP,NIS,Hesiod或者使用/usr/sbin/useradd命令来使这个工作站获知用户的帐号.如果使用这个选项,必须安装pam_krb5软件包.
--krb5realm=,工作站所属的Kerberos 5领域.
--krb5kdc=,为领域请求提供服务的KDC.如果的领域内有多个KDC,使用逗号(,)来分隔它们.
--krb5adminserver=,领域内还运行kadmind的KDC.该服务器处理改变口令以及其它管理请求.如果有不止一个KDC,该服务器必须是主KDC.
--enablehesiod,启用Hesiod支持来查找用户主目录,UID 和 shell.在网络中设置和使用 Hesiod 的更多信息,可以在 glibc 软件包里包括的/usr/share/doc/glibc-2.x.x/README.hesiod里找到.Hesiod是使用DNS记录来存储用户,组和其他信息的 DNS 的扩展.
--hesiodlhs,Hesiod LHS("left-hand side")选项在/etc/hesiod.conf里设置.Hesiod 库使用这个选项来决定查找信息时搜索DNS的名字,类似于LDAP对 base DN的使用.
--hesiodrhs,Hesiod RHS("right-hand side")选项在/etc/hesiod.conf里设置.Hesiod 库使用这个选项来决定查找信息时搜索DNS的名字,类似于LDAP对base DN的使用.
--enablesmbauth,启用对SMB服务器(典型的是Samba或Windows服务器)的用户验证.SMB验证支持不知道主目录,UID 或 shell.如果启用SMB,必须通过启用LDAP,NIS,Hesiod或者用/usr/sbin/useradd命令来使用户帐号为工作站所知.要使用这个选项,必须安装pam_smb软件包.
--smbservers=,用来做SMB验证的服务器名称.要指定不止一个服务器,用逗号(,)来分隔它们.
--smbworkgroup=,SMB服务器的工作组名称.
--enablecache,启用nscd服务.nscd服务缓存用户,组和其他类型的信息.如果选择在网络上用NIS,LDAP或hesiod分发用户和组的信息,缓存就尤其有用.
```

## rootpw：设置系统root账号的密码

```yaml
rootpw [--iscrypted|--plaintext] [--lock] password
--iscrypted #如果该选项存在,口令就会假定已被加密.可使用以下命令生成用随机盐值进行sha512加密后的密码
            # python -c 'import crypt,getpass;pw=getpass.getpass();print(crypt.crypt(pw) if (pw==getpass.getpass("Confirm: ")) else exit())'
--plaintext # 使用不加密的密码
--lock      #锁定root用户，root用户将无法登陆Console
```

# 三、可选选项

## install/upgrade

```yaml
install：默认安装方法。必须从 cdrom、harddrive、nfs、liveimg 或者 url（用于 FTP、HTTP、或者 HTTPS 安装）中指定安装类型
upgrade: 升级现有系统.
```

## text/graphical：kickstart安装模式

```yaml
text：在文本模式下执行kickstart安装.
graphical： 在图形模式下执行kickstart安装.kickstart安装默认在图形模式下安装.
```

## cdrom/harddrive/url/nfs/liveimg：指定安装类型

```yaml
cdrom: 从系统上的第一个光盘驱动器CD-ROM/DVD中安装.
harddrive [--biospart=<bios partition> | --partition=<partition>] [--dir=<directory>]: 
    # 从本地驱动器上包含ISO镜像的目录安装，该驱动器必须是vfat或者ext2文件系统。除了改目录之外，还需要以后面的方式提供install.img。
    # 一种方式是由boot.iso启动，另一种是在ISO镜像相同的目录中创建一个images/目录，然后将install.img放在那里。
    --biospart= 安装用到的BIOS分区（例如82p2）。
    --partition= 安装用到的硬盘分区
    --dir= 包含ISO镜像和images/install.img的目录
    # 例如:harddrive --partition=hdb2 --dir=/tmp/install-tree
    
nfs --server=<hostname> --dir=<directory> [--opts=<nfs options>]
    # 从指定的NFS服务器安装.
    --server=,指定NFS服务器（主机名或者IP）。
    --dir=,包含安装树的variant目录的目录.
    --opts=,用于挂载NFS输出的Mount选项(可选).
    # 例如:nfs --server=nfsserver.example.com --dir=/tmp/install-tree
    
url --url=<url>|--mirrorlist=<url> [--proxy=<proxy url>] [--noverifyssl] 
    # 通过FTP或HTTP从远程服务器上的安装树中安装.
    --url= 安装用到的URL。支持的协议有HTTP, HTTPS, FTP, file等。
    --mirrorlist= 安装用到的镜像URL。在该URL中完成$releasever和$basearch的变量替换
    --proxy= 指定安装时用到的HTTP/HTTPS/FTP代理。参数的各个部分用实际值来代替
    --noverifyssl 对于HTTPS服务器上的目录树，不用检查服务器的证书以及服务器的主机名匹配证书的域名
    # 例如:url --url http://<server>/<dir>或url --url ftp://<username>:<password>@<server>/<dir>
    
liveimg --url=<url> [--proxy=<proxyurl>] [--checksum=<sha256>] [--noverifyssl] 
     #使用磁盘映像而不是软件包安装。映像文件可以是取自实时 ISO 映像的 squashfs.img 文件，压缩 tar 文件（.tar、.tbz、.tgz、.txz、.tar.bz2、.tar.gz 或者 .tar.xz）
     #或者安装介质可以挂载的任意文件系统。支持的文件系统为 ext2、ext3、ext4、vfat 和 xfs。
     # Anaconda预期该镜像包含完成系统安装所需的实用程序。因此，创建磁盘镜像最好的方法是使用livemedia-creator。
     # 如果该镜像包含/LiveOS/*.img（这是squashfs.img的构成），LiveOS中的第一个*.img将会被挂载，并用来安装目标系统。
    --url= 执行安装的位置。支持的协议为 HTTP、HTTPS、FTP 和 file。
    --proxy=[protocol://][username[:password]@]host[:port] 指定在安装时用到的HTTP/HTTPS/FTP代理。参数的各个部分用实际值来代替。
    --checksum= 可选，镜像文件的sha256校验和。
    --noverifyssl 对于HTTPS服务器上的目录树，不用检查服务器的证书以及服务器的主机名匹配证书的域名。
```

## reboot/poweroff/shutdown/halt：安装完成后做什么操作

```yaml
reboot：重启（缺省选项）
poweroff：关闭系统并断电.通常,在手工安装过程中,anaconda会显示一条信息并等待用户按任意键来重新启动系统.
shutdown：关闭系统.
halt：halt选项基本和shutdown -h命令相同.
```

## services：设置禁用或允许列出的服务

```yaml
--disabled 设置服务为禁用
--enabled  启动服务
例：services --disabled autid,cups,smartd,nfslock  服务之间用逗号隔开，不能有空格
```

## selinux： 在系统里设置SELinux状态.在anaconda里,SELinux缺省为enforcing.

```yaml
selinux [--disabled|--enforcing|--permissive]
  --enforcing,启用SELinux,实施缺省的targeted policy. 注:如果kickstart文件里没有selinux选项,SELinux将被启用并缺省设置为--enforcing.
  --permissive,输出基于SELinux策略的警告,但实际上不执行这个策略.
  --disabled,在系统里完全地禁用 SELinux.
```

## user：在系统上创建新用户

```yaml
user --name=<username> [--groups=<list>] [--homedir=<homedir>] [--password=<password>] [--iscrypted] [--shell=<shell>] [--uid=<uid>]
  --name=,提供用户的名字.这个选项是必需的.
  --groups=,除了缺省的组以外,用户应该属于的用逗号隔开的组的列表.
  --homedir=,用户的主目录.如果没有指定,缺省为/home/<username>.
  --password=,新用户的密码.如果没有指定,这个帐号将缺省被锁住.
  --iscrypted=,所提供的密码是否已经加密？
  --shell=,用户的登录shell.如果不提供,缺省为系统的缺省设置.
  --uid=,用户的UID.如果未提供,缺省为下一个可用的非系统 UID.
```

## network：配置系统的网卡信息

```yaml
--device= 指定要使用network命令配置或者激活的设备。能够以和 ksdevice启动选项相同的方式指定。例如：network --bootproto=dhcp --device=eth0 对于第一个network命令，如果选项没有被指定，它默认是 1)ksdevice启动选项, 2)为了获得kickstart而激活的设备,或者 3)UI上的选择框。对于如下的network命令，需要--device选项。
--ip= 网络接口的IP地址。
--ipv6= 网络接口的IPv6地址。可以是<IPv6 address>[/<prefix length>]形式的静态地址，例如，3ffe:ffff:0:1::1/128(如果前缀被省略，会被假定为64)，"auto"地址分配基于动态的邻居发现协议，而"dhcp"会使用DHCPv6协议。
--gateway= 默认网关，是一个IPv4或者IPv6地址。
--nodefroute 组织设备抓取默认路由。在安装器中使用--activate选项激活其它设备时非常有用，从F16起。
--nameserver= 主域名服务器，是一个IP地址。多个域名服务器必须由逗号分隔。
--nodns 并不配置DNS服务器。
--netmask= 安装系统的网络掩码。
--hostname= 安装系统的主机名。
--ethtool= 指定将要传递给ethtool程序的设备附加的低级别设置。
--essid= 无线网网络ID。
--wepkey= 无线网WEP加密密钥。
--wpakey= 无线网WPA加密密钥(从F16起)。
--onboot= 是否在启动时使能该设备。
--dhcpclass= DHCP类别。
--mtu= 设备的MTU。
--noipv4 在该设备上禁用IPv4。
--noipv6 在该设备上禁用IPv6。
--bondslaves 使用该选项指定的网卡作为多网卡绑定的从网卡，虚拟出的网卡的名字由--device指定。例如--bondslaves=eth0,eth1。自Fedora 19开始。
--bondopts 为--bondslaves和--device选项指定的绑定接口指定一个逗号分隔的参数列表。例如：--bondopts=mode=active-backup,primary=eth1。如果一个选项本身以逗号作为分隔符，那么使用分号作为选项之间的分隔符。
--vlanid 使用--device指定的设备作为父设备来创建的vlan设备的Id(802.1q标签)。例如，network --device=eth0 --vlanid=171将会创建vlan设备eth0.171。从Fedora 19起。
```

## logging：该命令控制anaconda安装过程中的错误日志，并不影响安装系统。

```yaml
--host=,发送日志信息到给定的远程主机,这个主机必须运行配置为可接受远程日志的syslogd进程.
--port=,如果远程的syslogd进程没有使用缺省端口,这个选项必须被指定.
--level=,debug,info,warning,error或critical中的一个.指定tty3上显示的信息的最小级别.然而,无论这个级别怎么设置,所有的信息仍将发送到日志文件.
```

## zerombr ：清除mbr信息，会同时清空系统用原有分区表

## clearpart：在建立新分区前清空系统上原有的分区表，默认不删除分区

```yaml
--all      擦除系统上原有所有分区；
--drives    删除指定驱动器上的分区
--initlabel    初始化磁盘卷标为系统架构的默认卷标
--linux        擦除所有的linux分区
--none（default）不移除任何分区
​
例：clearpart  --drives=hda,hdb --all  --initlabel
```

## eula：使用这个选项以非用户互动方式接受终端用户许可证协议（End User License Agreement，EULA）。指定这个选项可防止 Initial Setup 在完成安装并第一次重启系统时提示您接受该许可证。

```yaml
--agreed（强制） - 接受 EULA。必须总是使用这个选项，否则 eula 命令就毫无意义。
```

## ignoredisk:指定安装程序忽略指定的磁盘。如果您使用自动分区并希望忽略某些磁盘的话，这就很有用

```
--only-use - 指定安装程序要使用的磁盘列表。忽略其他所有磁盘。
例如1：要在安装过程使用磁盘 sda，并忽略所有其他磁盘：ignoredisk --only-use=sda
    要包括不使用 LVM 的多路经设备：
    ignoredisk --only-use=disk/by-id/dm-uuid-mpath-2416CD96995134CA5D787F00A5AA11017
    要包括使用 LVM 的多路径设备：
    ignoredisk --only-use=disk/by-id/scsi-58095BEC5510947BE8C0360F604351918
--interactive - 允许手动导航高级存储页面。

```

## repo：配置用于软件包安装来源的额外的yum库.可以指定多个repo行.

```yaml
repo --name=repoid [--baseurl=<url>|--mirrorlist=url] [options]
  --name= - 该库的 id。这个选项是必选项。如果库名称与另一个之前添加的库冲突，则会忽略它。因为这个安装程序使用预先配置的库列表，就是说您无法添加名称与预先配置的库相同的库。
  --baseurl= - 程序库的 URL。这里不支持 yum 库配置文件中使用的变量。可以使用这个选项，也可以使用 --mirrorlist，但不能同时使用这两个选项。
  --mirrorlist= - URL 指向该程序库的一组镜像。这里不支持 yum 库配置文件中使用的变量。可以使用这个选项，也可以使用 --baseurl，但不能同时使用这两个选项。
  --install - 将所安装系统提供的存储库配置保存在/etc/yum.repos.d/目录中。不使用这个选项，在 Kickstart 文件中配置的程序库只能在安装过程中使用，而无法在安装的系统中使用。
  --cost= - 为这个库分配的 cost 整数值。如果多个库提供同样的软件包，这个数字就是用来规定那个库优先使用，cost 较低的库比 cost 较高的库优先。
  --excludepkgs= - 逗号分开的软件包名称列表，同时一定不能从这个存储库中提取该软件包名称。如果多个库提供同样的软件包，且要保证其来自某个特定存储库。
                  可接受完整软件包名称（比如 publican）和 globs（比如 gnome-*）。
  --includepkgs= - 逗号分开的软件包名称列表，同时一定要从这个存储库中提取 glob。如果多个存储库提供同样的软件包，且要保证其来自某个特定存储库，这个选项就很有用了。
  --proxy=[protocol://][username[:password]@]host[:port] - 指定只有这个存储库使用的 HTTP/HTTPS/FTP 代理服务器。
                                                          这个设置不会影响其他存储库，也不会影响将 install.img 附加到 HTTP 安装的方法。
  --ignoregroups=true - 组成安装树时使用这个选项，且对安装过程本身没有影响。它告诉组合工具在镜像树时不要查看软件包组信息，这样就不会镜像大量无用数据。
  --noverifyssl - 连接到 HTTPS 服务器时禁止 SSL 验证。
```

## interactive：在安装过程中使用kickstart文件里提供的信息,但允许检查和修改给定的值.将遇到安装程序的每个屏幕以及kickstart文件里给出的值.

## autostep：通常 Kickstart 安装会跳过不必要的页面。这个选项可让安装程序浏览所有页面，并摘要显示每个页面。部署系统时不应使用这个选项，因为它会干扰软件包安装

```yaml
--autoscreenshot 在安装的每一步均截屏。这些截屏将在安装过程中保存在 /tmp/anaconda-screenshots 中，并在安装完成后保存在 /root/anaconda-screenshots 中。
```

## sshpw：安装过程中是否开启SSH与安装进程进行交互与监控

```yaml
sshpw --username=name password [--iscrypted|--plaintext] [--lock]
--username 
--iscrypted 
--plaintext 
--lock 
```

## autopart：自动生成分区（autopart 选项不能与 part/partition, raid、logvol 或者 volgroup 在同样的 Kickstart 文件中一同使用。）

```yaml
--type= - 选择您要使用的预先定义的自动分区方案之一。可接受以下值：
    lvm: LVM 分区方案。
    btrfs: Btrfs 分区方案。
    plain: 不附带 LVM 或者 Btrfs 的常规分区。
    thinp: LVM 精简分区方案。
--fstype= - 选择可用文件系统类型之一。可用值为 ext2、ext3、ext4、xfs 和 vfat。默认系统为 xfs。有关使用这些文件系统的详情，请查看 第 6.14.4.1.1 节 “文件系统类型”。
--nolvm - 不使用 LVM 或者 Btrfs 进行自动分区。这个选项等同于 --type=plain。
--encrypted - 加密所有分区。这等同于在手动图形安装过程的起始分区页面中选中 加密分区 复选框。
        # 注意：加密一个或多个分区时，Anaconda 尝试收集 256 字节熵，以保证对分区安全加密与安装系统互动可加速此进程（使用键盘输入或移动鼠标）。
        # 如果要在虚拟机中安装系统，则可添加 virtio-rng 设备（虚拟随机数生成器），
--passphrase= - 为所有加密设备提供默认的系统范围内的密码短语。
--escrowcert=URL_of_X.509_certificate - 将所有加密卷数据加密密码保存在 /root 中，使用来自 URL_of_X.509_certificate 指定的 URL 的 X.509 证书加密。每个加密卷的密码都作为单独的文件保存。只有指定 --encrypted 时这个选项才有意义。
--backuppassphrase - 为每个加密卷添加随机生成的密码短语。将这些密码保存在 /root 目录下的独立文件中，使用 --escrowcert 指定的 X.509 证书加密。只有指定 --escrowcert 时这个选项才有意义。
--cipher= - 如果指定 Anaconda 默认 aes-xts-plain64 无法满足需要，则可以指定要使用的加密类型。这个选项必须与 --encrypted 选项一同使用，单独使用无效。 《Red Hat Enterprise Linux 7 安全指南》中有可用加密类型列表，但 Red Hat 强烈推荐您使用 aes-xts-plain64 或者 aes-cbc-essiv:sha256。
```

## firewall：配置系统防火墙选项

```yaml
firewall –enable|--disable  [ --trust ] <device> [ --port= ]
--enable   拒绝回应输出要求的进入连接，比如 DNS 答复或 DHCP 请求。如果需要访问在这台机器中运行的服务，可以选择通过防火墙允许具体的服务。
--disable 不配置任何iptables防御规则；
--trust 在这里列出设备,比如em1,允许所有流量通过该防火墙进出那个设备.要列出一个以上的设备,请使用--trust em1 --trust em2。不要使用逗号分开的格式，比如 --trust em1, em2。
--port   可以用端口:协议（port:protocal）格式指定允许通过防火墙的端口。例如，如果想允许 IMAP 通过您的防火墙，可以指定 imap:tcp。还可以具体指定端口号码，要允许 UDP 分组
         在端口 1234 通过防火墙，输入 1234:udp。要指定多个端口，用逗号将它们隔开。
--service= 这个选项提供允许服务通过防火墙的高级方法。有些服务（比如 cups、avahi 等等）需要开放多个端口，或者另外有特殊配置方可工作。
           您应该使用 --port 选项指定每个具体端口，或者指定 --service= 并同时打开它们       
incoming - 使用以下服务中的一个或多个来替换，从而允许指定的服务通过防火墙。
--ssh
--smtp
--http
--ftp
​
示例：firewall --enable --trust eth0  --trust eth1  --port=80:tcp
```

## group:在系统中生成新组。如果某个使用给定名称或者 GID 的组已存在，这个命令就会失败。另外，该 user 命令可用来为新生成的用户生成新组

```yaml
group --name=name [--gid=gid]
--name= - 提供组名称。
--gid= - 组的 UID。如果未提供，则默认使用下一个可用的非系统 GID。
```

## volgroup：创建逻辑卷组vg

```yaml
volgroup name partition [options]
​
--noformat - 使用现有卷组，且不进行格式化。
--useexisting - 使用现有卷组并重新格式化。如果使用这个选项，请勿指定 partition。例如：volgroup rhel00 --useexisting --noformat
--pesize= - 以 KiB 为单位设定卷组物理扩展大小。默认值为 4096 (4 MiB)，最小值为 1024 (1 MiB)。
--reserved-space= - 以 MB 为单位指定在卷组中预留的未使用空间量。只适用于新生成的卷组。
--reserved-percent= - 指定卷组中预留未使用空间的比例。只适用于新生成的卷组。
​
注意1 ：不要在逻辑卷和卷组名称中使用小横线（-）。如果使用这个字符，会完成安装，但 /dev/mapper/ 目录列出这些卷和卷组时，小横线会加倍。例如：某个卷组名为 volgrp-01，包含名        为 logvol-01 逻辑卷，该逻辑卷会以 /dev/mapper/volgrp--01-logvol--01 列出。这个限制只适用于新创建的逻辑卷和卷组名。如果您使用 --noformat 选项重复使用现有名称，        它们的名称就不会更改。
注意2: 应该先创建分区，然后创建逻辑卷组，再创建逻辑卷。例如：
       part pv.01 --size 10000
       volgroup volgrp pv.01 
       logvol / --vgname=volgrp --size=2000 --name=root
```

## logvol：创建逻辑卷lv

```yaml
logvol mntpoint --vgname=name --name=name [options]
  mntpoint — 是该分区挂载的位置，且必须是以下格式之一：
    1. /path 例如：/ 或者 /home
    2.swap 该分区被用作交换空间。要自动决定 swap 分区的大小，使用 --recommended 选项：swap --recommended 使用 --hibernation 选项自动决定 swap 分区的大小，同时还允许       您的系统有附加空间以便可以休眠：swap --hibernation分配的分区大小将与 --recommended 加上系统 RAM 量相等。
这些选项如下所示：
--noformat- 使用现有逻辑卷且不要对其进行格式化。
--useexisting  使用现有逻辑卷并重新格式化它。
--fstype=  为逻辑卷设置文件系统类型。有效值有：xfs、ext2、ext3、ext4、swap 和 vfat。
--fsoptions=  指定在挂载文件系统时所用选项的自由格式字符串。将这个字符串复制到安装的系统的 /etc/fstab 中，并使用括号括起来。
--mkfsoptions=  指定要提供的附加参数，以便在这个分区中建立文件系统。没有对任何参数列表执行任何操作，因此必须使用可直接为 mkfs 程序提供的格式。
                就是说可使用逗号分开或双引号分开的多个选项，要看具体文件系统。
--label=  为逻辑卷设置标签。
--grow  会让逻辑卷使用所有可用空间（若有），或使用设置的最大值（如果指定了最大值）。必须给出最小值，可使用 --percent= 选项或 --size= 选项。
--size=  以 MB 单位的逻辑卷大小。这个选项不能与 --percent= 选项一同使用。
--percent=  考虑任何静态大小逻辑卷时的逻辑卷大小，作为卷组中剩余空间的百分比。这个选项不能与 --size= 选项一同使用。
            #重要:创建新逻辑卷时，必须使用 --size= 选项静态指定其大小，或使用 --percent= 选项指定剩余可用空间的百分比。不能再同一逻辑卷中同时使用这些选项。
--maxsize= - 当将逻辑卷被设置为可扩充时以 MB 为单位的最大值。在这里指定一个整数值，如500（不要在数字后添加单位）。
--recommended - 创建 swap 逻辑卷时可采用这个选项，以根据您的系统硬件自动决定这个卷的大小。
--resize - 重新定义逻辑卷大小。如果使用这个选项，则必须还指定 --useexisting 和 --size。
--encrypted - 指定该逻辑卷应该用 --passphrase= 选项提供的密码进行加密。如果没有指定密码短语，安装程序将使用 autopart --passphrase 命令指定默认系统级密码，
               如果没有设定默认密码则会停止安装，并提示输入密码短语。
        # 注意:加密一个或多个分区时，Anaconda 尝试收集 256 字节熵，以保证对分区安全加密与安装系统互动可加速此进程（使用键盘输入或移动鼠标）。如果要在虚拟机中安装系统，则可
        # 添加 virtio-rng 设备（虚拟随机数生成器）
--passphrase= - 指定在加密这个逻辑卷时要使用的密码短语。必须与 --encrypted 选项一同使用，单独使用这个选项无效。
--cipher= - 指定如果对 Anaconda 默认 aes-xts-plain64 不满意时要使用的加密类型。这个选项必须与 --encrypted 选项一同使用，单独使用无效。
            推荐使用 aes-xts-plain64 或者 aes-cbc-essiv:sha256。
--escrowcert=URL_of_X.509_certificate 将所有加密卷数据加密密钥作为文件保存在 /root 中，使用来自 URL_of_X.509_certificate 指定的 URL 的 X.509 证书加密。每个加密卷                                           的密钥都作为单独的文件保存。只有指定 --encrypted 时这个选项才有意义。
--backuppassphrase - 为每个加密卷添加随机生成的密码短语。将这些密码保存在 /root 目录下的独立文件中，使用 --escrowcert 指定的 X.509 证书加密。只有指定 --escrowcert 时                        这个选项才有意义。
--thinpool - 创建精简逻辑卷。（使用 none 挂载点）。
--metadatasize=size - 为新的精简池设备指定元数据大小（单位 MiB）。
--chunksize=size - 为新的精简池设备指定块大小（单位 KiB）。
--thin - 创建精简逻辑卷。（要求使用 --poolname）
--poolname=name - 指定在其中创建精简逻辑卷的精简池名称。需要 --thin 选项。
--profile=name  指定与精简逻辑卷配合使用的配置文件名称。如果使用此选项，还要用于给定逻辑的卷元数据中包含该名称。默认情况下，可使用的配置文件为在 /etc/lvm/profile 目录中定                    义的 default 和 thin-performance。详情请查看 lvm(8) 手册页。
--cachepvs= - 用逗号分开的物理卷列表，应作为这个卷的缓存使用。
--cachemode= - 指定应使用哪种模式缓存这个逻辑卷 - 可以是 writeback，也可以是 writethrough。
         #注意:有关缓存的逻辑卷及其模式的详情，请查看 lvmcache(7) 手册页。
--cachesize= - 附加到该逻辑卷的缓存大小，单位为 MiB。这个选项需要 --cachepvs= 选项。
​
注意1: 应该先创建分区，然后创建逻辑卷组，再创建逻辑卷以占据逻辑组里剩余的 90% 空间。例如：：
      part pv.01 --size 1 --grow
      volgroup myvg pv.01
      logvol / --vgname=myvg --name=rootvol --percent=90
```

## timezone：设置系统的时区

```yaml
timezone  [ --utc ]  <timezone>
例：timezone  --utc  Asia/Shanghai
```

## 软件包的选择

1. 在 Kickstart 文件中使用 %packages 命令列出要安装的软件包。
2. 可以根据环境、组或者其软件包名称指定软件包。
    - 安装程序定义包含相关软件包的几个环境和组。有关环境和组列表请查看安装光盘中的 repodata/*-comps-variant.architecture.xml 文件。
    - *-comps-variant.architecture.xml 文件包含描述可用环境（使用 <environment> 标签标记）和组（<group> 标记）的结构。每个组都有一个 ID、用户可见性数值、名称、描述和软件包列表。如果未安装选择该组，那么就会安装该软件包列表中标记为 mandatory 的软件包；如果未明确指定，也会安装标记为 default 的软件包，而标记为 optional 的软件包必须在明确指定后方可安装。
    - 您可以使用 ID（<id> 标签）或者名称（<name> 标签）指定软件包组或者环境。
3. %packages 部分必须以 %end 命令结尾。
4. 除组外，您还要指定要安装的整体环境
   
    ```bash
    %packages
    @^Infrastructure Server
    %end
    ```

5. 指定组，每个条目一行，以 @ 符号开始，接着是空格，然后是完整的组名或 *-comps-variant.architecture.xml 中指定的组 id。
   
    ```bash
    %packages
    @X Window System
    @Desktop
    @Sound and Video
    %end
    ```

6. 根据名称指定独立软件包，每行一个条目。您可以在软件包名称中使用星号（*）作为通配符。
   
    ```bash
    %packages
    sqlite
    curl
    aspell
    docbook*
    %end
    ```

7. 使用小横线（-）开头指定安装中不使用的软件包或组。

    ```bash
    %packages
    -@Graphical Internet
    -autofs
    -ipa*fonts
    %end
    ```

## 常用软件包选择选项

```yaml
以下选项可用于 %packages。要使用这个选项，请将其添加到软件包选择部分的开始。例如：
%packages --multilib --ignoremissing
​
--default 安装默认软件包组。这与在互动安装过程中的 软件包选择 页面中没有其他选择时要安装的软件包组对应。
--excludedocs 不要安装软件包中的任何文档。大多数情况下，这样会排除一般安装在 /usr/share/doc* 目录中的所有文件，但要排除的具体文件取决于各个软件包。
--ignoremissing 忽略所有在这个安装源中缺少的软件包、组及环境，而不是暂停安装询问是应该放弃还是继续安装。
--instLangs= 指定要安装的语言列表。注：这与软件包组等级选择不同。这个选项不会告诉您应该安装哪些软件包组，而是通过设置 RPM 宏控制应该安装独立软件包中的哪些事务文件。
--multilib 为 multilib 软件包配置已安装的系统（即允许在 64 位系统中安装 32 位软件包），并安装在这一部分指定的软件包。通常在 AMD64 和 Intel 64 系统中，只安装用于整个架构
           （标记为 x86_64）的软件包以及用于所有架构（标记为 noarch）软件包。使用这个选项时，将自动安装用于 32 位 AMD 系统 Intel（标记为 i686）的软件包。这只适用于在 
           %packages 部分明确指定的软件包。对于那些仅作为相依性安装而没有在 Kickstart 文件中指定的软件包，将只安装其所需架构版本，即使有更多可用架构也是如此。
--nocore  禁用默认总被安装的 @Core 软件包组。禁用 @Core 软件包组应只用于创建轻量级的容器；用 --nocore 安装桌面或服务器系统将导致系统不可用。
```

## 具体软件包组参数项

```yaml
这个列表中的选项只用于单一软件包组。不是在 Kickstart 文件的 %packages 命令中使用，而是在组名称中添加条目。例如：
%packages
@Graphical Internet --optional
%end
​
--nodefaults 只安装该组的强制软件包，不是默认选择。
--optional  除安装默认选择外，还要安装在 *-comps-variant.architecture.xml 文件组定义中标记为自选的软件包。
注：有些软件包组，比如 Scientific Support，没有指定任何强制或默认软件包 - 只有自选软件包。在这种情况下必须使用 --optional 选项，否则不会安装这个组中的任何软件包。
```

## 安装前脚本
1. ks.cfg文件被解析后马上加入要运行的命令.这个部分必须处于kickstart文件的最后(在命令部分之后)而且
2. 必须用%pre命令开头，%end结尾.
3. 可以在%pre部分访问网络；然而,此时命名服务还未被配置,所以只能使用IP地址.
4. 预安装脚本不会在 chroot 环境中运行
    ```bash
    --interpreter= 定义脚本运行解释器.常用的有:/usr/bin/sh, /usr/bin/bash, and /usr/bin/python.
    --erroronfail 如果脚本失败则显示出错信息并暂停安装。该出错信息可让您进入记录失败原因的位置。
    --log= 记录脚本运行过程中的信息到指定路径的文件中。例如：%pre --log=/mnt/sysimage/root/ks-pre.log
    ```

### 示例

```yaml
%pre
#!/bin/sh
hds=""
mymedia=""
for file in /proc/ide/h* do
mymedia=`cat $file/media`
if [ $mymedia == "disk" ] ; then
hds="$hds `basename $file`"
fi
done
set $hds
numhd=`echo $#`
drive1=`echo $hds | cut -d' ' -f1`
drive2=`echo $hds | cut -d' ' -f2`
​
#Write out partition scheme based on whether there are 1 or 2 hard drives
if [ $numhd == "2" ] ; then
#2 drives
echo "#partitioning scheme generated in %pre for 2 drives" > /tmp/part-include
echo "clearpart --all" >> /tmp/part-include
echo "part /boot --fstype xfs --size 75 --ondisk hda" >> /tmp/part-include
echo "part / --fstype xfs --size 1 --grow --ondisk hda" >> /tmp/part-include
echo "part swap --recommended --ondisk $drive1" >> /tmp/part-include
echo "part /home --fstype xfs --size 1 --grow --ondisk hdb" >> /tmp/part-include
else
#1 drive
echo "#partitioning scheme generated in %pre for 1 drive" > /tmp/part-include
echo "clearpart --all" >> /tmp/part-include
echo "part /boot --fstype xfs --size 75" >> /tmp/part-include
echo "part swap --recommended" >> /tmp/part-include
echo "part / --fstype xfs --size 2048" >> /tmp/part-include
echo "part /home --fstype xfs --size 2048 --grow" >> /tmp/part-include
fi
%end
```

## 安装后脚本

post-install 脚本是在 chroot 环境里运行的.因此,某些任务如从安装介质复制脚本或RPM将无法执行.

```yaml
--nochroot    允许指定想在chroot环境之外运行的命令
--interpreter 定义脚本运行解释器.常用的有:/usr/bin/sh, /usr/bin/bash, and /usr/bin/python.
--erroronfail 如果脚本失败则显示出错信息并暂停安装。该出错信息可让您进入记录失败原因的位置。
--log=        记录脚本运行过程中的信息到指定路径的文件中。例如：%pre --log=/mnt/sysimage/root/ks-pre.log
```

### 示例

```yaml
%post --nochroot
cp /etc/resolv.conf /mnt/sysimage/etc/resolv.conf
%end
```

# 四、示例

```yaml
**全新安装系统**
install
#在文本模式下安装
text
#指定安装过程中语言为英语
lang en_US.UTF-8
#指定键盘类型为US布局
keyboard us
​
auth  --useshadow  --passalgo=sha512
#
url --url="http://192.168.1.80/CentOS7"
#设置root用户密码
rootpw --iscrypted $1$6/87AF3n$eczKeiNRBv7H.GXnur1Ld/
#开启SELinux
selinux --enforcing
#关闭防火墙
firewall --disabled
#设置网络信息。网卡1设置手动获取IP，不初始化IPV6,不设置为默认路由。网卡2设置DHCP获取IP，不初始化IPV6,设置为默认路由。
network  --bootproto=static --ip=192.168.10.6 --device=enp0s3 --activate --nodefroute --noipv6 --nameserver=114.114.114.114 --netmask=24 --gateway=192.168.10.1
network  --bootproto=dhcp --device=enp0s8 --activate --nameserver=114.114.114.114 --noipv6
#设置主机名
network  --hostname=test
#安装完成后重启
reboot
#设置时区为上海的时区
timezone Asia/Shanghai --isUtc --nontp
#将BootLoader安装在sda磁盘上
bootloader --location=mbr --boot-drive=sda
#安装时清理sda磁盘上所有的分区
clearpart --all --drives=sda
#将SSH和NetworkManager设置为开机自启动
services --enabled=NetworkManager,sshd
firstboot --enable
#只使用sda磁盘
ignoredisk --only-use=sda
#对sda磁盘进行分区。单位是MB
part /boot --fstype="xfs" --ondisk=sda --size=200
part / --fstype="xfs" --ondisk=sda --size=30720
part /opt --fstype="xfs" --ondisk=sda --size=10240
part /var --fstype="xfs" --grow --ondisk=sda --size=1
​
%packages
@^minimal
@core
%end
#安装后要执行的脚本
%post --interpreter=/bin/bash --log=/root/post-install.log
mkdir /etc/yum.repos.d/bak
mv /etc/yum.repos.d/C* /etc/yum.repos.d/bak
cat >> /etc/yum.repos.d/ustc.repo << EOF
[base]
name=CentOS-\$releasever - Base
#mirrorlist=http://mirrorlist.centos.org/?release=\$releasever&arch=\$basearch&repo=os
baseurl=https://mirrors.ustc.edu.cn/centos/\$releasever/os/\$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
#released updates
[updates]
name=CentOS-\$releasever - Updates
# mirrorlist=http://mirrorlist.centos.org/?release=\$releasever&arch=\$basearch&repo=updates
baseurl=https://mirrors.ustc.edu.cn/centos/\$releasever/updates/\$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
#additional packages that may be useful
[extras]
name=CentOS-\$releasever - Extras
# mirrorlist=http://mirrorlist.centos.org/?release=\$releasever&arch=\$basearch&repo=extras
baseurl=https://mirrors.ustc.edu.cn/centos/\$releasever/extras/\$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-\$releasever - Plus
# mirrorlist=http://mirrorlist.centos.org/?release=\$releasever&arch=\$basearch&repo=centosplus
baseurl=https://mirrors.ustc.edu.cn/centos/\$releasever/centosplus/\$basearch/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
EOF
yum clean all > /dev/null
yum makecache > /dev/null
yum install -y tree vim telnet nc unzip git net-tools wget bind-utils > /dev/null
echo "Set HOSTNAME test"
echo "Disabled SELinux and Firewall"
echo "/dev/sda /boot xfs 200MB"
echo "/dev/sda / xfs 30G"
echo "/dev/sda /opt xfs 10G"
echo "/dev/sda /var xfs RemainingCapacity"
echo "Make Yum Repository To USE USTC Yum Repository "
echo "Installed Tools : tree vim telnet nc unzip git net-tools wget bind-utils"
echo "    #######################" >> /etc/motd
echo "    # Keep Your Curiosity #" >> /etc/motd
echo "    #######################" >> /etc/motd
%end
```

# 五、验证KS文件的语法正确性

```bash
yum install -y pykickstart
ksvalidator ks文件
```

# 六、比较OS不同版本间的KS语法差异

```bash
ksverdiff -f RHEL6 -t RHEL7

# -f 指定要比较的第一个发行本，-t 指定要比较的最后一个发行本

```

# 参考连接

★★★★★： https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/html/installation_guide/sect-kickstart-syntax
★★★★★：https://docs.centos.org/en-US/centos/install-guide/Kickstart2/#sect-kickstart-commands
★★★★★ https://blog.csdn.net/yanghua1012/article/details/80426659