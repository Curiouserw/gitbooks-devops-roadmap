# YUM仓库的配置与使用

# 一、Overviews

 之前搭建内网YUM源仓库都是使用HTTP和createrepo做的，每次还要手动去同步外网镜像源最新的RPM包，繁琐耗时。最近发现新版的Nexus已经有可以做YUM源的功能，能像Maven仓库那样，没有相关资源的时候去外网拉，有的话就不去外网啦。即节省存储，又加快速度，还能自动更新RPM版本。

- `Group类型仓库`
  - yum
    - yum-ustc
    - yum-ansible
    - yum-cloudera5
- `Proxy类型仓库`
  - yum-ustc：http://mirrors.ustc.edu.cn/
  - yum-tuna：https://mirrors.tuna.tsinghua.edu.cn/
  - yum-163：http://mirrors.163.com/
  - yum-ansible：https://releases.ansible.com/ansible/rpm/releasepel-7-x86_64/
  - yum-cloudera5：https://archive.cloudera.com/cdh5/
- `Hosted类型仓库`
  - yum-hosted（"Repodata Depth"设置为“1”）


# 二、手动上传RPM包到Hosted仓库

上传RPM包到Hosted仓库，相关的RPM元信息文件夹repodata会自动生成，不再需要createrepo生成啦（如果没有自动生成，点击仓库的Rebuild Index，然后稍等即可）

```bash
$ ls -l | grep ^[^d] | awk '{print $9}'
mysql-community-client-5.7.19-1.el7.x86_64.rpm
mysql-community-common-5.7.19-1.el7.x86_64.rpm
mysql-community-devel-5.7.19-1.el7.x86_64.rpm
mysql-community-embedded-5.7.19-1.el7.x86_64.rpm
mysql-community-embedded-compat-5.7.19-1.el7.x86_64.rpm
mysql-community-embedded-devel-5.7.19-1.el7.x86_64.rpm
mysql-community-libs-5.7.19-1.el7.x86_64.rpm
mysql-community-libs-compat-5.7.19-1.el7.x86_64.rpm
mysql-community-minimal-debuginfo-5.7.19-1.el7.x86_64.rpm
mysql-community-server-5.7.19-1.el7.x86_64.rpm
mysql-community-server-minimal-5.7.19-1.el7.x86_64.rpm
mysql-community-test-5.7.19-1.el7.x86_64.rpm

#一次只能上传一个RPM文件，
#为了一个Host仓库存放不同软件的不同版本，上传时需要指定上传到Hosted仓库的哪个目录下，以文件夹名来区分当前上传的RPM包是哪个版本的。
$ for i in `ls *rpm` ;do curl -v --user 'admin:admin123' --upload-file $i http://nexus-ip地址:8081/repository/yum-nexus-MySQL/mysql5.7.19/;done
```

# 三、YUM仓库的使用

```properties
[centos]
name=Nexus Yum
baseurl=http://nexus-ip地址:8081/repository/yum-nexus/centos/$releasever/os/$basearch/
enabled=1
gpgcheck=0

[centos-extras]
name=Nexus Yum Extras
baseurl=http://nexus-ip地址:8081/repository/yum-nexus/centos/$releasever/extras/$basearch/
enabled=1
gpgcheck=0

[epel]
name=Nexus Epel
baseurl=http://nexus-ip地址:8081/repository/yum-nexus/epel/$releasever/$basearch/
enabled=1
gpgcheck=0

[openshift]
name=Nexus openshift 3.11
baseurl=http://nexus-ip地址:8081/repository/yum-nexus/centos/$releasever/paas/$basearch/openshift-origin311/
enabled=1
gpgcheck=0

[mysql5.7]
name=Nexus MySQL 5.7.19
baseurl=http://nexus-ip地址:8081/repository/yum-nexus/mysql5.7.19
enabled=1
gpgcheck=0
```

# 注意

1. 不要代理多个外网的YUM源。例如同时代理缓存网易的163镜像源和清华大学的镜像源，这些外网镜像源中的RPM都大差无几，配置多个的话，使用group去代理拉RPM的时候，会优先选择group中靠前的外网代理镜像源。有时（偶尔其中一个外网镜像源不能用）会出现相同的RPM存在于每一个proxy仓库中，浪费存储。