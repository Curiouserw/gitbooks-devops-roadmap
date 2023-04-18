# Git常用操作

## 1、删除远程仓库分支

```bash
git push 远程仓库别名 :远程仓库中的分支
```

## 2、克隆远程仓库指定分支到本地指定路径

```bash
git clone <Git_URL> -b <branch_name> <指定路径>
```

## 3、拉取远程仓库指定分支到本地仓库的特定分支

```bash
git fetch <remote_repo_shortname> 远程仓库中分支名:本地分支名
#使用该方式会在本地新建分支，但是不会自动切换到该本地分支，需要手动checkout切换分支。
git remote update ;\
git checkout -b local_branch <remote_repo_shortname>/<remote_branch>
#使用该方式会在本地新建分支，并自动切换到该本地分支。采用此种方法建立的本地分支会和远程分支建立映射关系。
```

## 4、Git代理设置

```bash
# 设置使用HTTP类型的代理
git config --global https.proxy http://127.0.0.1:1080
git config --global https.proxy https://127.0.0.1:1080

# 取消代理的设置
git config --global --unset http.proxy
git config --global --unset https.proxy

# 设置使用socks5类型的代理
git config --global http.proxy 'socks5://127.0.0.1:1081'
git config --global https.proxy 'socks5://127.0.0.1:1081'

# 查看代理设置
git config --global --get http.proxy
git config --global --get https.proxy
```

## 5、查看单个文件修改历史

```bash
git log --pretty=oneline 文件名

	5096d69f*** handle merge file conflict
	a7593501*** fix: 同步 master docker 文件夹所有文件

git show  a7593501***
```

## 6、修改已提交的Commint

```bash
git commit --amend
```

## 7、HTTP方式设置记住用户名和密码

设置记住密码（默认15分钟）

```
git config --global credential.helper cache
```

设置时间，例如设置一个小时之后失效

```
git config credential.helper 'cache --timeout=3600'
```

长期存储密码

```
git config --global credential.helper store
```

增加远程地址的时候设置密码

```
http://yourname:password@git.oschina.net/name/project.git
```

## 8、Git tag使用

```bash
# 查看tag
git tag

# 打某个分支的tag
git tag -a 0.0.0_b1 -m "test tag"

# 将本地tag推送到远程仓库中
git push origin --tags

# 删除本地tag
git tag -d 0.0.0_b3

# 删除远程仓库中的tag
git push origin :tags/0.0.0_b1
```

## 9、获取最近一次提交的commit id

```bash
# 获取完整commit id
git rev-parse HEAD

# 获取8位commit id
git rev-parse --short HEAD
```

## 10、commit回退

```bash
# 进回退到最近一个的上一个commit
git reset --hard HEAD^

# 回退到指定commit。commit的ID可残缺地写
git reset --hard <commit id>

# 查看已回退commit的历史，并恢复回退的commit
git reflog
git reset --hard <commit id>
```

## 11、本地分支Merge

```bash
git checkout master
# 当前所处master分支，以下命令是将develop分支合并到master分支
git merge develop
```

## 12、解决合并冲突

```bash
git diff --name-only --diff-filter=U
```

## 13、对比差异

```bash
git diff master..develop
git diff master...
```

## 14、显示commit的author和committer

关于Author和Committer的区别，请参考：https://stackoverflow.com/questions/6755824/what-is-the-difference-between-author-and-committer-in-git

```bash
# 查看最近一个commit的author和committer
git log --format="%an %cn" -n 1

# 查看commit Hash后的文件内容
git cat-file -p <commit id>
```

## 15、清除历史commit创建新分支

```bash
# 当某一个分支积累了太多commit历史信息，管理起来比较麻烦。或者某些commit信息包含敏感想要清除掉。
git checkout --orphan <new branch>
git add .
git commit -am "reset commit info"
git push origin
```

## 16、设置当个仓库的用户名邮箱地址

在项目根目录下进行单独配置

```bash
git config user.name "用户名"
git config user.email "邮箱地址"
git config --list
```

也可以直接在项目根目录下的`.git/config`文件中直接追加以下内容

```ini
[user]
	name = 用户名
	email = 邮箱地址
```

## 17、git push参数与Git alias

```bash
git push \
	-o merge_request.create \
	-o merge_request.target=develop \
	-o merge_request.title=test-push-option \
	-o merge_request.assign=tester \
	-o merge_request.description='test git push option' \
  -o ci.skip
```

使用git Alias 快速设置参数

```bash
git config --global alias.mwps "push -o merge_request.create -o merge_request.target=develop -o merge_request.merge_when_pipeline_succeeds"

git mwps origin <local-branch-name>
```

参考：https://docs.gitlab.com/ee/user/project/push_options.html#push-options-for-merge-requests

# Git问题总结

- `fatal: I don't handle protocol 'http'`
  - 原因：`.git/config`中的远程仓库URL可能有乱码

