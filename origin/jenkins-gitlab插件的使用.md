# Jenkins Gitlab插件的使用

# 一、Overviews

Jenkins的Gitlab插件能接收gitlab仓库代码事件触发的Webhook来触发Jenkins Job/Pipeline的构建，并且能将构建状态同步到Gitlab中.

插件Github：https://github.com/jenkinsci/gitlab-plugin

![](../assets/jenkins-gitlab插件的使用-0.png)

**利用此插件可实现如下效果：**

![](../assets/jenkins-gitlab插件的使用-00.png)

![](../assets/jenkins-gitlab插件的使用-01.png)

# 二、安装

Jenkins -> 系统管理 ->插件管理->可用插件->搜索"gitlab"

插件下载地址：https://plugins.jenkins.io/gitlab-plugin

# 三、配置

## 1、Gitlab发送Webhook到Jenkins时的安全认证配置

Gitlab代码仓库配置web hook时需要Jenkins Gitlab插件的Token,来加密验证webhook的安全性

### 方式一：全局性的认证Token

  ① Jenkins创建新用户（授予Job/Build权限即可）

  ② 获取新用户的User ID和API Token

  ③ Gitlab配置web hook填写URL时，使用`http://USERID:APITOKEN@JENKINS_URL/project/YOUR_JOB`即可（`Secret Token`可忽略）

### 方式二(推荐)：单独项目Jenkins Job的认证Token

  ![](../assets/jenkins-gitlab插件的使用-1.png)
  在Gitlab中配置Webhook时，将图中的Token填入即可

### 方式三(不推荐)：不使用认证

  ① Manage Jenkins -> Configure System -> `GitLab` section -> 取消勾选 "Enable authentication for '/project' end-point"

  ![](../assets/jenkins-gitlab插件的使用-2.png)

## 2、Jenkins回写构建状态到Gitlab时需要的认证授权配置

Jenkins通过Gitlab API将构建状态回写到对应代码仓库时，需要有权限在代码仓库中创建评论，更改状态等

### 步骤一

  ① Gitlab创建新用户（建议命名用户名时尽量见名知意，例如：Jenkins）

  ② 登陆新用户，获取`Access Tokens`,Token权限只要`api`即可(及时复制Token,只显示一次)

   ![](../assets/jenkins-gitlab插件的使用-3.png)

  ③ 在Jenkins中创建`GitLab API token`类型的凭据，在`API token`字段中保存上一步获取的Token

  ④ 在Jenkins Gitlab插件中配置gitlab Server相关信息

   ![](../assets/jenkins-gitlab插件的使用-4.png)

### 步骤二

  对应代码仓库配置Jenkins Job时要将该Token所属的用户加入members中，授予Developers角色或有更高权限的角色

# 五、Jenkins Job中配置Webhook

## 1、配置Gitlab Buid Trigger

   ![](../assets/jenkins-gitlab插件的使用-12.png)

## 2、配置构建后Gitlab回写信息的动作

- 添加构建后动作"Pushlish build status to Gitlab"

   ![](../assets/jenkins-gitlab插件的使用-5.png)

   之后会在Gitlab代码仓库的`CI/CD-->Pipeline`和Merge Request中看到构建状态图标

    ![](../assets/jenkins-gitlab插件的使用-6.png)

- 添加构建后动作"Add note with build status on Gitlab merge request"

   ![](../assets/jenkins-gitlab插件的使用-7.png)

   之后会根据构建状态的不同在Gitlab的Merge Request中添加不同的comment评论

   ![](../assets/jenkins-gitlab插件的使用-8.png)

- 添加构建后动作"Add Vote for build status on Gitlab merge request"

   ![](../assets/jenkins-gitlab插件的使用-9.png)

   之后会根据构建状态的不同在Gitlab的Merge Request中显示不同的投票

   ![](../assets/jenkins-gitlab插件的使用-10.png)

- 添加构建后动作"Accept Gitlab merge request on success"

   ![](../assets/jenkins-gitlab插件的使用-11.png)

   该动作会在build构建成功后在Gitla上自动同意接受Merge Request

- 不添加构建后动作

  Gitlab的事件只会触发Jenkins构建，不会回写任何信息到gitlab对应代码仓库中

# 五、Gitlab 代码仓库配置Webhook

详见：[gitlab-配置代码仓库事件触发器Webhook](gitlab-配置代码仓库事件触发器Webhook.md)

# 六、获取Webhook HTTP请求信息的变量

对于Gitlab发送过来的Webhook HTTP POST请求信息，可直接通过以下变量来获取。

- gitlabBranch
- gitlabSourceBranch
- gitlabActionType
- gitlabUserName
- gitlabUserEmail
- gitlabSourceRepoHomepage
- gitlabSourceRepoName
- gitlabSourceNamespace
- gitlabSourceRepoURL
- gitlabSourceRepoSshUrl
- gitlabSourceRepoHttpUrl
- gitlabMergeRequestTitle
- gitlabMergeRequestDescription
- gitlabMergeRequestId
- gitlabMergeRequestIid
- gitlabMergeRequestState
- gitlabMergedByUser
- gitlabMergeRequestAssignee
- gitlabMergeRequestLastCommit
- gitlabMergeRequestTargetProjectId
- gitlabTargetBranch
- gitlabTargetRepoName
- gitlabTargetNamespace
- gitlabTargetRepoSshUrl
- gitlabTargetRepoHttpUrl
- gitlabBefore
- gitlabAfter
- gitlabTriggerPhrase
