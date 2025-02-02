# Nexus的OrientDB数据库操作

# 一、找回密码

在配置Nexus对接LDAP过程中，原先的用户（admin和原来创建的）也是可以登录的，为了测试只让LDAP用户登录，将安全域中的Local Authorizing Realm和Local Authenticating Realm给去掉了，导致admin用户登陆不上，LDAP上的用户能登陆，但没有管理Nexus的权限。换句话说，这个Nexus成了僵尸。如果Nexus的admin密码忘了,情形也类似。都需要修改Nexus数据库中的用户数据才能修改相关配置（Nexus是将用户、配置等相关信息存放在OrientDB 数据库中，而不是在配置文件中，所以改密码或者在没有权限的情况下改配置都是要操作OrientDB 中的数据）

1. Nexus版本：3.15.2
2. Nexus的数据目录是单独使用NFS类型的PV挂载的
3. openshift上Nexus容器的执行用户没有权限操作除持久化目录之外的目录。（以root用户起的Nexus进程可直接使用Nexus容器中的`/opt/nexus/lib/support/nexus-orient-console.jar`连接OrientDB ）

## 1. 进入控制台

```bash
java -jar /opt/nexus-3.15.2-01/lib/support/nexus-orient-console.jar
```

此时会进入OrientDB 控制台

```bash
# OrientDB console v.2.2.36 (build d3beb772c02098ceaea89779a7afd4b7305d3788, branch 2.2.x) https://www.orientdb.com
# Type 'help' to display all the supported commands.
orientdb>
#输入exit退出
```

## 2. 连接DB

```bash
orientdb> connect plocal:/root/test/db/security admin admin
```

## 3. 重置Realm

如果从活动列表中删除了缺省安全域，则缺省管理员用户(即使用户名密码正确)也无法进行身份验证

```bash
orientdb>  delete from realm
```

**Note**: 重置后。默认的安全域将被激活，任何自定义的安全域都将被删除。后续再UI界面进行添加

## 4.(可选) 重置admin用户密码

密码`admin123`加密后的`$shiro1$SHA-512$1024$NE+wqQq/TmjZMvfI7ENh/g==$V4yPw8T64UQ6GfJfxYq2hLsVrBY8D1v+bktfOxGdt4b/9BthpWPNUy/CBk6V9iA0nHpzYzJFWO8v/tZFtES8CA==`

```bash
orientdb>  update user SET password="$shiro1$SHA-512$1024$NE+wqQq/TmjZMvfI7ENh/g==$V4yPw8T64UQ6GfJfxYq2hLsVrBY8D1v+bktfOxGdt4b/9BthpWPNUy/CBk6V9iA0nHpzYzJFWO8v/tZFtES8CA==" UPSERT WHERE id="admin"
```

## 5.(可选) 重置admin为管理员

如果admin用户不在是“nx-admin”管理员角色

```bash 
orientdb>  select * from user_role_mapping where userID = "admin"
orientdb>  update user_role_mapping set roles = ["nx-admin"] where userID = "admin"  
orientdb>  select status from user where id = "admin"
orientdb> update user set status="active" upsert where id="admin"
```

## 6. 收尾

退出OrientDB 控制台，修改回临时数据目录所有文件的原始权限，重新将数据目录挂载到容器上，然后重启Nexus

Bazinga，admin用户能正常，又重新夺回Nexus的管理权限，所有的仓库配置和数据没有丢失，重新将LDAP的安全域加回去，一切恢复原样。

# 二、预写日志损坏导致无法启动

## 0、报错

```bash
*SYSTEM Felix - Framework listener delivery error.
com.orientechnologies.orient.core.exception.OStorageException: Cannot open local storage '/nexus-data/db/config' with mode=rw
```

orient 会检测到损坏并尝试重播预写日志中的事务，上述错误表明预写日志本身也已损坏。

## 1、删除wal数据

```
cp -r /data/nexus/nexus-data/db /data/nexus/nexus-data/db-backup
rm -f /data/nexus/nexus-data/db/config/*.wal
```

## 2、进入控制台

```bash
java -jar /opt/sonatype/nexus/lib/support/nexus-orient-console.jar
```

## 3、连接数据库文件

```bash
orientdb> connect plocal:/nexus-data/db/config/ admin admin
orientdb> repair database --fix-links
orientdb> rebuild index *
orientdb> disconnect 
orientdb> exit
```

# 附录：Nexus使用的OrientDB

## 1. Launching the OrientDB Console on Nexus 3.2.1 and Newer

**Nexus 3.2.1+ includes a single jar executable which can launch the OrientDB console.**

As the operating system user account that typically owns the Nexus Repository Manager process, start a terminal session on the host where Nexus is installed.
Change directories to your application directory.
Launch the console using the same version of Java executable that Nexus is using :
Unix

```shell
java -jar ./lib/support/nexus-orient-console.jar
```

Windows

```shell
java -jar lib\support\nexus-orient-console.jar
```

Mac

```shell
.install4j/jre.bundle/Contents/Home/jre/bin/java -jar ./lib/support/nexus-orient-console.jar
```

You should be presented with a command-line interface such as this, ready to accept your commands:

```shell
# OrientDB console v.2.2.16 www.orientdb.com
#Type 'help' to display all the supported commands.
orientdb>
# When you are done your commands, type exit to quit the console.
```

**The nexus-orient-console.jar sets up the correct classpath to successfully launch the console. Launching the jar from another location is not supported**

# 参考连接

1. https://support.sonatype.com/hc/en-us/articles/115002930827-Accessing-the-OrientDB-Console
2. https://support.sonatype.com/hc/en-us/articles/213467158-How-to-reset-a-forgotten-admin-password-in-Nexus-3-x
3. https://groups.google.com/a/glists.sonatype.com/g/nexus-users/c/64W_y9fDGzQ/m/o0FKRgiDAwAJ?pli=1
4. https://stackoverflow.com/questions/51776568/sonatype-nexus-repository-orientdb-cant-connect-to-database