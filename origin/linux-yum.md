# YUM详解

# 一、Overviews

Yum（全称为 Yellow dog Updater, Modified）是一个在Fedora和RedHat以及CentOS中的包管理器。基于RPM包管理，能够从指定的服务器自动下载RPM包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软件包，无须繁琐地一次次下载、安装。

Yum的关键之处是要有可靠的repository，顾名思义这就是软件的仓库，它可以是http或者ftp站点，也可以是本地的软件池，但是必须包含rpm的header，rmp的header包括了rmp的各种信息，包括描述、功能、提供的文件、依赖性等，正是收集了这些信息，才能自动化的完成余下的任务。

repo文件是yum源（软件仓库）的配置文件，通常一个repo文件定义了一个或者多个软件仓库的细节内容

RPM包的名称规则示例：ttpd-manual- 2.0.40-21.i386.rpm，ttp-manual是软件包的名称，2是主版本号；0是次版本号；40是修正号；21是编译的次数；i386是适合的平台

# 二、YUM命令详解

yum的命令形式一般是如下：yum [选项] [参数] [package ...]

**选项**

```bash
  -h, --help            显示此帮助消息并退出
  -t, --tolerant        忽略错误
  -C, --cacheonly       完全从系统缓存运行，不升级缓存
  -c [config file], --config=[config file]  配置文件路径
  -R [minutes], --randomwait=[minutes]  命令最长等待时间
  -d [debug level], --debuglevel=[debug level]  调试输出级别
  --showduplicates      在 list/search 命令下，显示源里重复的条目
  -e [error level], --errorlevel=[error level]  错误输出级别
  --rpmverbosity=[debug level name]  RPM 调试输出级别
  -q, --quiet           静默执行
  -v, --verbose         详尽的操作过程
  -y, --assumeyes       回答全部问题为是
  --assumeno            回答全部问题为否
  --version             显示 Yum 版本然后退出
  --installroot=[path]  设置安装根目录
  --enablerepo=[repo]   启用一个或多个软件源(支持通配符)
  --disablerepo=[repo]  禁用一个或多个软件源(支持通配符)
  -x [package], --exclude=[package]  采用全名或通配符排除软件包
  --disableexcludes=[repo] 禁止从主配置，从源或者从任何位置排除
  --disableincludes=[repo] disable includepkgs for a repo or for everything
  --obsoletes           更新时处理软件包取代关系
  --noplugins           禁用 Yum 插件
  --nogpgcheck          禁用 GPG 签名检查
  --disableplugin=[plugin]  禁用指定名称的插件
  --enableplugin=[plugin]   启用指定名称的插件
  --skip-broken         忽略存在依赖关系问题的软件包
  --color=COLOR         配置是否使用颜色
  --releasever=RELEASEVER  在 yum 配置和 repo 文件里设置 $releasever 的值
  --downloadonly        仅下载而不更新
  --downloaddir=DLDIR   指定一个其他文件夹用于保存软件包
  --setopt=SETOPTS      设置任意配置和源选项
  --bugfix              Include bugfix relevant packages, in updates
  --security            Include security relevant packages, in updates
  --advisory=ADVS, --advisories=ADVS  Include packages needed to fix the given advisory, in updates
  --bzs=BZS             Include packages needed to fix the given BZ, in updates
  --cves=CVES           Include packages needed to fix the given CVE, in updates
  --sec-severity=SEVS, --secseverity=SEVS  Include security relevant packages matching the severity, in updates
```

**参数**

```bash
check          检查 RPM 数据库问题
check-update   检查是否有可用的软件包更新
clean          删除缓存数据
deplist        列出软件包的依赖关系
distribution-synchronization 已同步软件包到最新可用版本
downgrade      降级软件包
erase          从系统中移除一个或多个软件包
fs             Acts on the filesystem data of the host, mainly for removing docs/lanuages for minimal hosts.
fssnapshot     Creates filesystem snapshots, or lists/deletes current snapshots.
groups         显示或使用、组信息
help           显示用法提示
history        显示或使用事务历史
info           显示关于软件包或组的详细信息
install        向系统中安装一个或多个软件包
langavailable  Check available languages
langinfo       List languages information
langinstall    Install appropriate language packs for a language
langlist       List installed languages
langremove     Remove installed language packs for a language
list           列出一个或一组软件包
load-transaction 从文件名中加载一个已存事务
makecache      创建元数据缓存
provides       查找提供指定内容的软件包
reinstall      覆盖安装软件包
repo-pkgs      将一个源当作一个软件包组，这样我们就可以一次性安装/移除全部软件包。
repolist       显示已配置的源
search         在软件包详细信息中搜索指定字符串
shell          运行交互式的 yum shell
swap           Simple way to swap packages, instead of using shell
update         更新系统中的一个或多个软件包
update-minimal Works like upgrade, but goes to the 'newest' package match which fixes a problem that affects your system
updateinfo     Acts on repository update information
upgrade        更新软件包同时考虑软件包取代关系
version        显示机器和/或可用的源版本。
```

**常用命令**

清除缓存
```bash
yum clean [headers, packages, metadata, dbcache, plugins, expire-cache, rpmdb, all]

headers--清除缓存目录(/var/cache/yum)下的 headers
packages--清除缓存目录(/var/cache/yum)下的软件包
all--清除所有缓存
```

**yum update与yum upgrade的区别**

* yum update  只更新软件，不更新内核
* yum upgrade 升级所有包，不改变软件设置和系统设置，系统版本升级，内核不改变

# 三、repo文件

```properties
[serverid]
#serverid是用于区别各个不同的repository，必须有一个独一无二的名称。若重复了，是前面覆盖后面--还是反过来呢？？？用enabled 测试是后面覆盖前面

name=Some name for this server
#name，是对repository的描述，支持像$releasever $basearch这样的变量; name=Fedora Core $releasever - $basearch - Released Updates

baseurl=url://path/to/repository/
# 1. 格式: baseurl=url://server1/path/to/repository/, url支持的协议有 http:// ftp:// file://三种。
# 2. baseurl后可以跟多个url，你可以自己改为速度比较快的镜像站，但#baseurl只能有一个
# 3. 其中url指向的目录必须是这个repository header目录的上一级，它也支持$releasever $basearch这样的变量。

gpgcheck=1

exclude=gaim

failovermethod=priority
#failovermethode有两个选项roundrobin和priority，意思分别是有多个url可供选择时，yum选择的次序，roundrobin是随机选择，如果连接失 败则使用下一个，依次循环，priority则根据url的次序从第一个开始。如果不指明，默认是roundrobin。

enabled=[1 or 0]
# 1. 当某个软件仓库被配置成 enabled=0 时，yum 在安装或升级软件包时不会将该仓库做为软件包提供源。使用这个选项，可以启用或禁用软件仓库。
# 2. 通过 yum 的 --enablerepo=[repo_name] 和 --disablerepo=[repo_name] 选项，或者通过 PackageKit 的"添加/删除软件"工具，也能够方便地启用和禁用指定的软件仓库


变量解释：
# $releasever  发行版的版本，从[main]部分的distroverpkg获取，如果没有，则根据redhat-release包进行判断。
# $arch        cpu体系，如i686,athlon等
# $basearch    cpu的基本体系组，如i686和athlon同属i386，alpha和alphaev6同属alpha。
```

# 四、yum-fastestmirror插件

yum-fastestmirror插件，它会自动选择最快的mirror。它的配置文件/etc/yum/pluginconf.d/fastestmirror.conf，yum镜像的速度测试记录文件/var/cache/yum/x86_64/7/timedhosts.txt**

**禁用插件配置**

1. 修改插件的配置文件

```bash
sed -i 's/enabled=1/enabled=0/' /etc/yum/pluginconf.d/fastestmirror.conf
enabled = 1//由1改为0，禁用该插件
```

2. 修改yum的配置文件

```bash
# sed -i 's/plugins=1/plugins=0/' /etc/yum.conf
plugins=1   //改为0，不使用插件
```

# 五、YUM源的创建

## 1、使用Nexus的YUN格式仓库作为YUM镜像源

详见：[Nexus中yum仓库的配置与使用](origin/nexus-yum仓库的配置与使用.md)

## 2、Createrepo创建本地YUM镜像源

- 将CentOS版本系统镜像中的Packages并上传到主机上的某一目录下
- 安装createrepo用来创建软件包的索引。或者将系统镜像中repodata目录放到rpm包路径下

  ```bash
   yum install createrepo -y
   createrepo /data/localrepo/Office      会在创建repodata索引文件夹
  ```

- 在/etc/yum.repos.d/目录下创建repo文件local.repo

  ```properties
  [local]
  name=Local Yum Office Repository                      仓库名
  baseurl=file:///data/localrepo/Office                 仓库中rpm包存放路径
  gpgcheck=1                                            是否检查GPG-KEY，0为不检查，1为检查
  gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7   GPG-KEY的存放路径
  enabled=1                                             设置为1表示启用本Repo
  ```

- Createrepo命令参数详解

  ```bash
  -u  --baseurl <url>      指定Base URL的地址
  -o --outputdir <url>      指定元数据的输出位置
  -x --excludes <packages>      指定在形成元数据时需要排除的包
  -i --pkglist <filename>      指定一个文件，该文件内的包信息将被包含在即将生成的元数据中，格式为每个包信息独占一行，不含通配符、正则，以及范围表达式。
  -n --includepkg      通过命令行指定要纳入本地库中的包信息，需要提供URL或本地路径。
  -q --quiet      安静模式执行操作，不输出任何信息。
  -g --groupfile <groupfile>      指定本地软件仓库的组划分，范例如下：createrepo -g comps.xml /path/to/rpms注意：组文件需要和rpm包放置于同一路径下。
  -v --verbose      输出详细信息。
  -c --cachedir <path>      指定一个目录，用作存放软件仓库中软件包的校验和信息。
      当createrepo在未发生明显改变的相同仓库文件上持续多次运行时，指定cachedir会明显提高  其性能。
  --update      如果元数据已经存在，且软件仓库中只有部分软件发生了改变或增减，
      则可用update参数直接对原有元数据进行升级，效率比重新分析rpm包依赖并生成新的元数据要  高很多。
  -p --pretty      以整洁的格式输出xml文件。
  -d --database      该选项指定使用SQLite来存储生成的元数据，默认项。
  ```

## 3、HTTPD+Createrepo创建YUM镜像源

第二种方法创建的镜像源只能在本地使用，要是能在局域网中提供公共的服务，需要一个能提供HTTP服务的容器，可使用HTTPD（又称Apache），步骤省略。

# 六、Reposync同步YUM远程仓库的安装包

## 1、安装

```bash
yum install yum-utils -y
```

## 2、命令参数

```bash
Usage:
    Reposync is used to synchronize a remote yum repository to a local 
    directory using yum to retrieve the packages.
    /usr/bin/reposync [options]
Options:
  -h, --help            show this help message and exit
  -c CONFIG, --config=CONFIG config file to use (defaults to /etc/yum.conf)
  -a ARCH, --arch=ARCH  act as if running the specified arch (default: current arch, note: does not override $releasever. x86_64 is a superset for i*86.)
  --source              operate on source packages
  -r REPOID, --repoid=REPOID secify repo ids to query, can be specified multiple  times (default is all enabled)
  -e CACHEDIR, --cachedir=CACHEDIR    directory in which to store metadata
  -t, --tempcache       Use a temp dir for storing/accessing yum-cache
  -d, --delete          delete local packages no longer present in repository
  -p DESTDIR, --download_path=DESTDIR  Path to download packages to: defaults to current dir
  --norepopath          Don't add the reponame to the download path. Can only  be used when syncing a single repository (default is  to add the reponame)
  -g, --gpgcheck        Remove packages that fail GPG signature checking after  downloading
  -u, --urls            Just list urls of what would be downloaded, don't  download
  -n, --newest-only     Download only newest packages per-repo
  -q, --quiet           Output as little as possible
  -l, --plugins         enable yum plugin support
  -m, --downloadcomps   also download comps.xml
  --download-metadata   download all the non-default metadata
```

## 3、示例

```bash
$> bash -c 'cat > ceph.repo <<EOF
[Ceph]
name=Ceph packages for \$basearch
baseurl=http://download.ceph.com/rpm-jewel/el7/\$basearch
enabled=1
gpgcheck=1
type=rpm-md
gpgkey=https://download.ceph.com/keys/release.asc
priority=1
[Ceph-noarch]
name=Ceph noarch packages
baseurl=http://download.ceph.com/rpm-jewel/el7/noarch
enabled=1
gpgcheck=1
type=rpm-md
gpgkey=https://download.ceph.com/keys/release.asc
priority=1
EOF' ;\
  reposync --repoid=Ceph-noarch -p /data/ -n -c ceph.repo  
#  --repoid指定的是RepoID
#  -p指定的是存储的目录
#  -n指定下载最新的包
#  -c指定的是repo文件的路径，默认的是/etc/yum.repo.d目录下的repo文件
```

# 七、下载软件RPM包以及其依赖包

## 1、yum插件yumdownloadonly

```bash
yum install yum-plugin-downloadonly && 
yum install --downloadonly --downloaddir=/root/httpd httpd
```

## 2、使用yum-utils的命令yumdownloader

```bash
yum install -y yum-utils && \
yumdownloader --resolve docker-ce-20.10.5
```



# 附录：常见软件源

## 1、VirtualBox

```properties
[virtualbox]
name=Oracle Linux / RHEL / CentOS-$releasever / $basearch - VirtualBox
baseurl=http://download.virtualbox.org/virtualbox/rpm/el/$releasever/$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://www.virtualbox.org/download/oracle_vbox.asc
```

## 2、Nginx

```properties
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/OS/OSRELEASE/$basearch/
gpgcheck=0
enabled=1
```

## 3、EPEL

```properties
[epel]
name=Extra Packages for Enterprise Linux 7 - $basearch
#baseurl=http://download.fedoraproject.org/pub/epel/7/$basearch
metalink=https://mirrors.fedoraproject.org/metalink?repo=epel-7&arch=$basearch
failovermethod=priority
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
```

## 4、Ceph

```properties
[Ceph]
name=Ceph packages for $basearch
baseurl=http://download.ceph.com/rpm-jewel/el7/$basearch
enabled=1
gpgcheck=1
type=rpm-md
gpgkey=https://download.ceph.com/keys/release.asc
priority=1
[Ceph-noarch]
name=Ceph noarch packages
baseurl=http://download.ceph.com/rpm-jewel/el7/noarch
enabled=1
gpgcheck=1
type=rpm-md
gpgkey=https://download.ceph.com/keys/release.asc
priority=1
```

## 5、ELK Stack

```properties
[ELK-Stack-5.x]
name=ELK Stack repository for 5.x packages
baseurl=https://mirrors.tuna.tsinghua.edu.cn/elasticstack/5.x/yum/
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
[ELK-Stack-6.x]
name=ELK Stack repository for 6.x packages
baseurl=https://mirrors.tuna.tsinghua.edu.cn/elasticstack/6.x/yum/
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```

## 6、MySQL

```properties
[MySQL-Community-5.6]
name=MySQL Community 5.6
baseurl=https://mirrors.ustc.edu.cn/mysql-repo/yum/mysql-5.6-community/el/7/x86_64/
enabled=1
gpgcheck=0
[MySQL-Community-5.7]
name=MySQL Community 5.7
baseurl=https://mirrors.ustc.edu.cn/mysql-repo/yum/mysql-5.7-community/el/7/x86_64/
enabled=1
gpgcheck=0
[MySQL-Community-8.0]
name=MySQL Community 8.0
baseurl=https://mirrors.ustc.edu.cn/mysql-repo/yum/mysql-8.0-community/el/7/x86_64/
enabled=1
gpgcheck=0
[MySQL-Community-Connectors]
name=MySQL Community  Connectors
baseurl=http://mirrors.ustc.edu.cn/mysql-repo/yum/mysql-connectors-community/el/7/x86_64/
enabled=1
gpgcheck=0
[MySQL-Community-Tools]
name=MySQL Community  Tools
baseurl=https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql-tools-community-el7/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
```

## 7、MongoDB

```properties
bash -c 'cat > /etc/yum.repos.d/mongoDb.repo <<EOF
[mongodb-3.6]
name=MongoDB Repository
baseurl=http://mirrors.tuna.tsinghua.edu.cn/mongodb/yum/el7-3.6/
gpgcheck=0
enabled=1
EOF'
```
