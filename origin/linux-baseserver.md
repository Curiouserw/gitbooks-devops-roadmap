# 一、NFS

## 1、服务端

### ①安装

```
yum install -y nfs-utils rpcbind ;\
systemctl enable nfs ;\
systemctl enable rpcbind ;\
systemctl start nfs ;\
systemctl start rpcbind
```

### ②配置

配置文件：`/etc/exports`

```bash
# 配置格式
要共享的目录 指定客户端IP地址1(权限) 指定客户端IP地址2(权限)

/NFS 172.16.2.0/24(ro,rw)
/NFS 172.16.2.0/24(ro,rw) 192.168.0.2(ro,rw)
```

限制挂载客户端 

```
指定ip地址的主机：192.168.0.100
指定子网中的所有主机：192.168.0.0/24 或 192.168.0.0/255.255.255.0
指定域名的主机：nfs.test.com
指定域中的所有主机：*.test.com
所有主机：*
```

设置挂载权限 

```
ro：共享目录只读；
rw：共享目录可读可写；
all_squash：所有访问用户都映射为匿名用户或用户组；
no_all_squash（默认）：访问用户先与本机用户匹配，匹配失败后再映射为匿名用户或用户组；
root_squash（默认）：将来访的root用户映射为匿名用户或用户组；
no_root_squash：来访的root用户保持root帐号权限；
anonuid=<UID>：指定匿名访问用户的本地用户UID，默认为nfsnobody（65534）；
anongid=<GID>：指定匿名访问用户的本地用户组GID，默认为nfsnobody（65534）；
secure（默认）：限制客户端只能从小于1024的tcp/ip端口连接服务器；
insecure：允许客户端从大于1024的tcp/ip端口连接服务器；
sync：将数据同步写入内存缓冲区与磁盘中，效率低，但可以保证数据的一致性；
async：将数据先保存在内存缓冲区中，必要时才写入磁盘；
wdelay（默认）：检查是否有相关的写操作，如果有则将这些写操作一起执行，这样可以提高效率；
no_wdelay：若有写操作则立即执行，应与sync配合使用；
subtree_check（默认） ：若输出目录是一个子目录，则nfs服务器将检查其父目录的权限；
no_subtree_check ：即使输出目录是一个子目录，nfs服务器也不检查其父目录的权限，这样可以提高效率；
```

### ③生效配置 

```
exportfs -a
```

### ④验证查看

```bash
showmount -e Server_IP
```

### ⑤查看运行状态

```bash
nfsstat

rpcinfo

netstat -tu -4
```

### ⑥查看挂载的客户端

```bash
# 查看除本机以外挂载的客户端
netstat -an | grep $(hostname -i):2049 |awk '{print $5}' |grep -v "$(hostname -i)"
```

## 2、客户端

### ①客户端安装

```bash
yum install -y nfs-utils rpcbind
```

### ②手动挂载

```bash
mount -t nfs -o 挂载参数 Server_IP:Share_Dir  Mount_Dir 

mount -t nfs -o vers=3,nolock,proto=tcp,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 \
192.168.1.7:/mnt/nfs /mnt/backup
```

### ③自动挂载

```bash
echo "Server_IP:Share_Dir Mount_dir nfs 权限 0 1" >> /etc/fstab
mount -a
```

### ④挂载参数

| 参数                         | 说明                                                         | 参数建议                                                     |
| ---------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| soft/hard                    | 软[挂载](https://so.csdn.net/so/search?q=挂载&spm=1001.2101.3001.7020)方式挂载系统，若NFS请求超时，则客户端向调用程序返回错误；如果使用硬连接方式则客户端一直重新请求直至成功。默认为hard。 | 对于关键数据业务，不希望业务由于网络延迟或服务器服务重启或短暂的过载等情况而中断，建议挂载时使用hard参数；对于非关键数据业务，希望客户端程序能尽快响应，可以使用soft参数。 |
| timeo=n                      | 客户端重传请求前等待时间。对于基于TCP的NFS服务，默认等待重传时间为60s。使用TCP协议时，NFS Client不执行任何超时backoff。对于UDP协议，client使用一个合适的算法，为常用的请求类型estimate合适的超时时间。但对不常用的请求类型使用timeo设置。如果timeo没有设置，不常用的请求类型1.1秒以后重试。在每次重发后，NFS Client会将timeout时间加倍，直到最大的60秒。 |                                                              |
| retrans=n                    | 客户端返回错误前的重传次数。默认为重传3次。retrans与soft参数一起使用时才有效。 | timeo、retrans这两个参数选择主要取决于网络性能。对于网络吞吐量小，延时高，抖动高，丢包率大的情况，建议将timeo，retrans两个参数值设置大一些。对于网络吞吐量大，延时低，抖动低，丢包率小的情况，建议将timeo，retrans两个参数值设置小一些。具体设置值因网络状况而定。 |
| resize=n                     | 每个READ命令字向服务器读取文件的最大字节数。实际数据小于或等于此值。resize必须是1024倍数的正整数，小于1024时自动设为4096，大于1048576时自动设为1048576。默认时，服务器和客户端进行协商后设置。 | 通常使用默认值，由客户端和服务器协商设置。对于拥塞的低速网络，可以将该值调小，向服务器发送较短的请求包来提高NFS性能。对于高速网络，可以将该值增大，减少向服务器发送的请求包，获得性能的提升。 |
| wsize=n                      | 每个WRITE命令字向服务器写入文件的最大字节数。实际数据小于或等于此值。resize必须是1024倍数的正整数，小于1024时自动设为4096，大于1048576时自动设为1048576。默认时，服务器和客户端进行协商后设置。 | 通常使用默认值，由客户端和服务器协商设置。对于拥塞的低速网络，可以将该值调小，向服务器发送较小的请求包来提高NFS性能。对于高速网络，可以将该值增大，减少向服务器发送的请求包，获得性能的提升。 |
| async/sync                   | 同步、异步挂载，客户端默认异步（async）。对于异步挂载，客户端下发的写数据会先缓存在内存中，达到一定大小或者其他条件（与客户端设置，状态有关），再一起发往服务端。而同步挂载，每次下发的写数据马上发到服务端。 | 建议用默认的异步挂载，可以提高业务性能。对于客户端应用层来说，完全感知不到数据是否发往服务端，只能感知数据已经写成功。同步挂载要求每个请求立即发到服务端，增加了请求连接、发送次数，对于小io场景性能差异很明显（io越小，差异越大，一般都能达到几倍甚至更大的差异）。<br/> 注：对于异步挂载，可能在查看io性能显现为性能波动。在客户端写入缓存期间iops可能就低，因为数据还未发到服务端。但是性能是完全正常的，应该说比同步更好。如果是要要追求界面上的平稳，那就建议改为同步挂载。（补充：追求一次小io的数据全部刷到服务端稳定存储时间小，不仅要求客户端同步挂载，还需要服务端共享也是同步导出，否则需要加上手动commit刷盘——flush）。 |
| acregmin/acregmax            | 设置NFS客户端缓存普通文件属性的最短时间和最长时间，单位为秒。超过此时间后对其进行更新。默认最短时间是3s，最长时间是60s。 |                                                              |
| acdirmin/acdirmax            | 设置NFS客户端缓存目录属性的最短时间和最长时间，单位为秒。超过此时间后对其进行更新。默认最短时间是3s，最长时间是60s。 |                                                              |
| ac/noac                      | 设置是否缓存文件属性。为了提高性能，NFS客户端缓存文件属性（默认ac），然后每隔一段时间去检查文件属性后更新。在缓存有效期内，客户端不检测服务器上文件属性是否改变。默认为ac。 | 当服务器上共享文件的属性频繁地被多个客户端改变时，建议使用noac选项，或者使用ac并配合使用较小的`acregmin/acregmax/acdirmin/acdirmax`设置，这样就能获得较好的属性一致性。当服务器上共享文件的属性不会被频繁改变时，例如文件共享为只读，或者网络性能较好，建议使用默认的ac选项，然后根据实际的网络状况来调整`acregmin/acregmax/acdirmin/acdirmax`设置。 |
| Actimeo                      | 将acregmin/acregmax/acdirmin/acdirmax四个参数设置为相同时间，单位为秒。 |                                                              |
| bg/fg                        | 设置挂载失败后的行为方式。默认的fg方式将立刻退出返回错误状态，bg方式是退出前将产生一个子进程在后台继续尝试挂载。 |                                                              |
| sharecache/nosharecache      | 设置客户端并发挂载同一文件系统时数据缓存和属性缓存的共享方式。设置为sharecache时，多个挂载共享共享同一缓存。设为nosharecache时，每个挂载各有一个缓存。默认为sharecache。 | 该参数用于客户端多次挂载同一共享目录的情况，建议使用默认的sharecache选项。 |
| resvport/noresvport          | 设置连接服务器是否使用保密源端口。默认的resvport设置保密端口，noresvport设置为非保密端口。内核2.6.28及以后版本支持。 |                                                              |
| lookupcache=mode             | 设置内核管理给定挂载点的目录项缓存方式。其中包括all/none/pos几种方式。客户端缓存LOOKUP命令字请求结果。如果请求的目录项在服务器上，则返回结果为正，否则为负。all的管理方式是在父目录缓存的属性失效前客户端缓存这两种目录项；pos是在父目录缓存的属性失效前客户端缓存结果为正的查询结果，总是重新验证结果为负的查询结果。none总是重新验证目录缓存项。默认为all管理方式，内核2.6.28及以后版本支持。 | LOOKUP命令字的作用是将文件名转换文件句柄。对于多个客户端经常创建或删除文件的情况，建议使用none。其它情况选用all或者pos。 |
| intr/nointr                  | 设置是否允许信号中断挂载点的文件操作。如果指定intr，当NFS操作被信号中断时系统返回EINTR。指定nointr，信号不会中断NFS文件操作。默认为nointr。指定intr时，通常同时使用soft选项，防止数据损坏。内核2.6.25及以后不再支持。 |                                                              |
| cto/nocto                    | 设置是否使用“关闭打开”缓存一致的特性。通常客户端打开文件时检查是否存在以及是否有权限，当关闭文件时刷新更改。当设置为nocto时，客户端使用非标准的试探来检查服务器上文件是否改变，对于只读和文件更改较少时情形有助于提高性能 | 对于文件内容改变很少的情况，如服务器提供只读共享权限（文件系统以RO权限导出）给客户使用，建议使用nocto选项，这样可以提高性能。对于文件内容经常改变，客户端对文件缓存一致性要求较高，建议使用cto参数。 |
| proto=transport  (NFS2/NFS3) | 客户端向服务器发起传输请求使用的协议，可以为UDP或者TCP。未指定时，mount命令选择服务器支持的协议。 |                                                              |
| tcp/udp                      | 等价于proto=tcp和proto=udp选项。                             | 在不稳定的复杂网络环境中建议使用tcp参数，在稳定的网络下可以使用udp参数。NFSv3/NFSv4支持tcp/udp，NFSv2只支持udp。 |
| port=n                       | 指定服务器NFS服务端口。如果NFS服务端口不在port上，则mount请求失败。未指定或设为0，mount命令根据服务器的rpcbind服务选择服务端口。 |                                                              |
| mountport=n                  | 指定服务器上mountd的端口。如果mountd服务端口不在port上，则mount请求失败。未指定或设为0，mount命令根据服务器的rpcbind服务选择服务端口。该参数用于即使防火墙屏蔽rpcbind协议也能正常mount到服务器。 |                                                              |
| mountproto=transport         | 客户端向服务器发起MNT请求时和UMNT使用的协议，可以为udp或者tcp。该参数用于防火墙屏蔽特定的协议也能正常mount到服务器。 |                                                              |
| mounthost=name               | 设置开启mountd主机名。未指定时，mount命令认为mountd服务和NFS服务在同一主机上。 |                                                              |
| mountvers=n                  | 设置连接服务器mountd时的RPC版本号。未指定时使用与请求的NFS版本相适应的版本号。该参数用于多个NFS服务运行于同一远程主机上 |                                                              |
| namlen=n                     | 设置挂载路径名的最大长度。未指定时，通过与服务器协商设定。大多数情况为255字节 |                                                              |
| nfsvers/vers=n               | 设置使用NFS服务的NFS协议版本号。当文件系统为nfs时，Linux客户端支持NFS2和NFS3。如果不支持请求的版本，mount失败。未指定时，客户端先尝试使用NFS3，若失败再与服务器进行协商。 |                                                              |
| lock/nolock                  | 选择是否使用NLM协议在服务器上锁文件。当选择nolock选项时，锁对于同一主机的应用有效，对不同主机不受锁的影响。默认为lock |                                                              |
| acl/noacl                    | 设置是否在挂载时使用NFSACL协议。NFSACL协议不属于标准NFS协议，而是Solaris上的实现。未指定时，客户端与服务器协商检查服务器是否支持，如果支持则使用NFSACL |                                                              |
| rdirplus/nordirplus          | 设置是否使用NFS3的REAADDIRPLUS请求。默认为readdirplus        |                                                              |
| proto=transpro （NFS4）      | 客户端向服务器发起传输请求使用的协议，可以为UDP或者TCP。未指定时，选用TCP。 |                                                              |
| port=n                       | 指定服务器NFS服务端口。如果NFS服务端口不在port上，则mount请求失败。未指定时，客户端使用NFS标准的2049号端口。指定为0时，客户端选用服务器rpcbind服务支持的端口。 |                                                              |
| clientaddr=n.n.n.n           | 指定一个IPv4的主机地址使服务器能执行NFS4的回调请求。未指定时，mount命令尝试自己发现一个合适的回调地址。 |                                                              |



## 3、卸载出现“设备正忙”的挂载

```bash
# 找到哪个进程正在使用挂载点的文件
lsof | grep '挂载点' 
# 查看使用挂载点文件的进程命令，如果无关紧要，可以直接杀掉
kill -9 对应的进程号
```



# 二、FTP的虚拟用户

 vsftpd服务器同时支持匿名用户、本地用户和虚拟用户三类用户账号，使用虚拟用户账号可以提供集中管理的FTP根目录，方便了管理员的管理，同时将用于FTP登录的用户名、密码与系统用户账号区别开，进一步增强了FTP服务器的安全性。

- 设置不使用ftp所在主机的系统用户及匿名用户登录
- 限制登录的虚拟用户只在指定的目录下操作
- 方便添加虚拟用户

## 1、安装

```bash
yum install -y vsftpd
vsftpd -version
```

## 2、配置

创建系统用户，所有的虚拟用户都对应到这个用户

```bash
useradd -d /data/ftp/ -s /sbin/nologin vftpuser 
```

创建虚拟用户的账号密码文本

```bash
sehll >vi /etc/vsftpd/vuser.txt
testuser
1234test
#奇数行是用户名，偶数是密码

#生成虚拟数据库文件（*如果db_load没有安装，yum install db4-utils db4-devel db4-4.3安装才能使用。）
db_load –T –t hash –f /etc/vsftpd/vuser.txt  /etc/vsftpd/vuser.db
#修改虚拟数据库文件vuser.db的权限为 700
chmod   700  /etc/vsftpd/vuser.db  

#配置PAM文件，目的是对客户端进行验证.编辑/etc/pam.d/vsftpd文件，注释所有内容，后添加：
vi /etc/pam.d/vsftpd
auth                 required     pam_userdb.so   db=/etc/vsftpd/vuser  
account              required     pam_userdb.so   db=/etc/vsftpd/vuser  
#不能写成db=/etc/vsftpd/vuser.db
```

配置vsftp的配置文件`/etc/vsftpd/vsftpd.conf`

```bash
anonymous_enable=NO       #是否允许匿名用户登录
local_enable=YES           #是否允许vsftp所在主机的本地用户登录
listen=YES
listen_ipv6=NO
guest_enable=YES           #激活虚拟账户  
guest_username=vftpuser    #把虚拟账户绑定为系统账户vftpuser   
pam_service_name=vsftpd    #使用PAM验证，指定PAM配置文件，文件已经在/etc/pam.d/存在（第二步配置的）
user_config_dir=/etc/vsftpd/vsftpd_user_conf  #设置虚拟用户的配置文件目录，配置文件名与虚拟用户名同名
write_enable=NO
```

配置虚拟用户的配置文件`/etc/vsftpd/vsftpd_user_conf/testuser`

```bash
anon_world_readable_only=NO        #浏览FTP目录
anon_upload_enable=YES            #允许上传
anon_mkdir_write_enable=YES        #建立和删除目录  
anon_other_write_enable=YES    #改名和删除文件
local_root=/data/ftp/test   #指定虚拟用户在系统用户下面的路径，限制虚拟用户家目录，虚拟用户登录后主目录
write_enable=YES                #启用/禁止用户的写权限
allow_writeable_chroot=YES
download_enable=NO							#设置只能上传不能下载
cmds_denied=DELE								#禁用掉删除DELE命令
#每行配置项结尾不能有空格
```

创建配置文件中设置的目录并设置相关权限

```bash
mkdir -p /data/ftp/test ;\
chown -R vftpuser:vftpuser /data/ftp/test ;\
chmod -R 770 /data/ftp/test
```

## 3、启动验证

```bash
systemctl start vsftpd
systemctl status vsftpd -l

ftp 127.0.0.1
> ls
> put local_file_path ftp_file_path
```

## 4、配置中出现的错误

### ①`refusing to run with writable root inside chroot ()`

限定了用户不能跳出其主目录之后，使用该用户登录FTP时往往会遇到这个错误

```
500 OOPS: vsftpd: refusing to run with writable root inside chroot ()
```

从2.3.5之后，vsftpd增强了安全检查，如果用户被限定在了其主目录下，则该用户的主目录不能再具有写权限了！如果检查发现还有写权限，就会报该错误。

 要修复这个错误，可以用命令`chmod a-w /home/user`去除用户主目录的写权限（注意把目录替换成你自己的）。或者你可以在vsftpd_user_conf下的虚拟用户配置文件中增加下列一项：`allow_writeable_chroot=YES`

### ②`530 login incrrect`无法登陆。 

查看日志`tail -f /var/log/secure或者systemctl status vsftpd -l`

```
PAM unable to dlopen(/lib/security/pam_userdb.so): /lib/security/pam_userdb.so: cannot open shared object file: No such file or directory
```

原来pam_userdb.so在`/lib64/security/pam_userdb.so`,修改`/etc/pam.d/vsftpd`

```bash
#将下列内容
auth    required      /lib/security/pam_userdb.so     db=/etc/vsftpd/vuser
account required      /lib/security/pam_userdb.so     db=/etc/vsftpd/vuser
#替换成
auth    required      /lib64/security/pam_userdb.so     db=/etc/vsftpd/vuser
account required      /lib64/security/pam_userdb.so     db=/etc/vsftpd/vuser
#或者
auth    required     pam_userdb.so   db=/etc/vsftpd/vuser  
account required     pam_userdb.so   db=/etc/vsftpd/vuser 
```

 保存重启vsftpd服务。

### ③`425 Security: Bad IP connecting`

当使用非21端口进行端口转发连接的话，会出现上述情况。

解决方案：

```bash
1.#vim /etc/vsftpd/vsftpd.conf 
2.添加：pasv_promiscuous=YES 
3.保存后退出 
4.重启vsftpd #service vsftpd restart

#pasv_promiscuous选项参数说明：
此选项激活时，将关闭PASV模式的安全检查。该检查确保数据连接和控制连接是来自同一个IP地址。小心打开此选项。此选项唯一合理的用法是存在于由安全隧道方案构成的组织中。默认值为NO。 
合理的用法是：在一些安全隧道配置环境下，或者更好地支持FXP时(才启用它)。
```

 