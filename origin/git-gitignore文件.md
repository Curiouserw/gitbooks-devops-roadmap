# git的.gitignore文件

# 一、简介

一般来说每个Git项目中都需要一个“.gitignore”文件，这个文件的作用就是告诉Git哪些文件不需要添加到版本管理中。

# 二、用法

```
# 此为注释 – 将被 Git 忽略
*.a       # 忽略所有 .a 结尾的文件
!lib.a    # 但 lib.a 除外
/TODO     # 仅仅忽略项目根目录下的 TODO 文件，不包括 subdir/TODO
build/    # 忽略 build/ 目录下的所有文件
doc/*.txt # 会忽略 doc/notes.txt 但不包括 doc/server/arch.txt
```

## 注意

1. 这样没有扩展名的文件在Windows下不太好创建，方法：创建一个文件，文件名为：“.gitignore.”，注意前后都有一个点。保存之后系统会自动重命名为“.gitignore”。
2. 假设我们只有过滤规则没有添加规则，那么我们就需要把/mtk/目录下除了one.txt以外的所有文件都写出来！
3. 如果你不慎在创建.gitignore文件之前就push了项目，那么即使你在.gitignore文件中写入新的过滤规则，这些规则也不会起作用，Git仍然会对所有文件进行版本管理。简单来说，出现这种问题的原因就是Git已经开始管理这些文件了，所以你无法再通过过滤规则过滤它们。所以一定要养成在项目开始就创建.gitignore文件的习惯，否则一旦push，处理起来会非常麻烦

# 三、样本

[参考https://github.com/github/gitignore/blob/master/Maven.gitignore](https://github.com/github/gitignore/blob/master/Maven.gitignore)

```yaml
target/
pom.xml.tag
pom.xml.releaseBackup
pom.xml.versionsBackup
pom.xml.next
release.properties
dependency-reduced-pom.xml
buildNumber.properties
.mvn/timing.properties
.mvn/wrapper/maven-wrapper.jar

### IDES ###
### STS ###
.apt_generated
.classpath
.factorypath
.project
.settings
.springBeans
.sts4-cache

### IntelliJ IDEA ###
.idea
*.iws
*.iml
*.ipr

### NetBeans ###
/nbproject/private/
/nbbuild/
/dist/
/nbdist/
/.nb-gradle/
/build/

### VS Code ###
.vscode/

### OS ###
.DS_Store
```

