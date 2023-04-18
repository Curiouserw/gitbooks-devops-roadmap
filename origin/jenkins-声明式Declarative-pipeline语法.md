
# Jenkins声明式Declarative Pipeline

# 一、语法结构

Jenkins 2.5新加入的pipeline语法

声明式pipeline 基本语法和表达式遵循 groovy语法，但是有以下例外：

- 声明式pipeline 必须包含在固定格式的pipeline{}中
- 每个声明语句必须独立一行， 行尾无需使用分号
- 块(Blocks{}) 只能包含章节(Sections),指令（Directives）,步骤(Steps),或者赋值语句
- 属性引用语句被视为无参数方法调用。 如input()

一个声明式Pipeline中包含的元素

- pipeline：声明这是一个声明式的pipeline脚本
- agent：指定要执行该Pipeline的节点（job运行的slave或者master节点）
- stages：阶段集合，包裹所有的阶段（例如：打包，部署等各个阶段）
- stage：阶段，被stages包裹，一个stages可以有多个stage
- steps：步骤,为每个阶段的最小执行单元,被stage包裹
- post：执行构建后的操作，根据构建结果来执行对应的操作

示例：

```groovy
pipeline{
    // 指定pipeline在哪个slave节点上允许
    agent { label 'jdk-maven' }
    // 指定pipeline运行时的一些配置
    option {
        timeout(time: 1, unit: 'HOURS')
    }
    // 自定义的参数
    parameters {
        string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
        text(name: 'BIOGRAPHY', defaultValue: '', description: 'Enter some information about the person')
        booleanParam(name: 'TOGGLE', defaultValue: true, description: 'Toggle this value')
        choice(name: 'CHOICE', choices: ['One', 'Two', 'Three'], description: 'Pick something')
        password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'Enter a password')
    }
    // 自定义的环境变量
    environment {
        Gitlab_Deploy_KEY = credentials('gitlab-jenkins-depolykey')
    }
    // 定义pipeline的阶段任务
    stages {
        stage ("阶段1任务：拉代码") {
            steps {
                // 拉代码的具体命令
            }
        }
        stage ("阶段2任务：编译代码") {
            steps {
                // 编译代码的具体命令
            }
        }
        stage ("阶段3任务：扫描代码") {
            steps {
                // 拉代码的具体命令
            }
        }
        stage ("阶段4任务：打包代码") {
            steps {
                // 打包代码的具体命令
            }
        }
        stage ("阶段5任务：构建推送Docker镜像") {
            steps {
                // 构建推送Docker镜像的具体命令
            }
        }
        stage ("阶段6任务：部署镜像") {
            steps {
                // 部署镜像的具体命令
            }
        }
    }
    post {
        success {
            // 当pipeline构建状态为"success"时要执行的事情
        }
        always {
            // 无论pipeline构建状态是什么都要执行的事情
        }
    }
}

```

# 二、章节Sections

## 1、agent（必须）

指定整个Pipeline或特定阶段是在Jenkins Master节点还是Jenkins Slave节点上运行。可在顶级`pipeline`块和每个`stage`块中使用（在`顶层pipeline{}`中是必须定义的 ，但在阶段Stage中是可选的）

参数（以下参数值在`顶层pipeline{}`和`stage{}`中都可使用）：

- any：在任何可用的节点上执行Pipeline或Stage
- none：当在`顶层pipeline{}`中应用时，将不会为整个Pipeline运行分配全局代理，并且每个`stage`部分将需要包含其自己的`agent`部分
- label
- node
- docker
- dockerfile
- kubernetes

公用参数：

- label
- customWorkspace
- reuseNode
- args

## 2、post

定义在Pipeline运行或阶段结束时要运行的操作。具体取决于Pipeline的状态

支持pipeline运行状态:

- always：无论Pipeline运行的完成状态如何都要运行
- changed：只有当前Pipeline运行的状态与先前完成的Pipeline的状态不同时，才能运行
- fixed：整个pipeline或者stage相对于上一次失败或不稳定Pipeline的状态有改变。才能运行
- regression：
- aborted：只有当前Pipeline处于“中止”状态时，才会运行，通常是由于Pipeline被手动中止（通常在具有灰色指示的Web UI 中表示）
- failure：仅当当前Pipeline处于“失败”状态时才运行（通常在Web UI中用红色指示表示）
- success：仅当当前Pipeline在“成功”状态时才运行（通常在具有蓝色或绿色指示的Web UI中表示）
- unstable：只有当前Pipeline在不稳定”状态，通常由测试失败，代码违例等引起，才能运行（通常在具有黄色指示的Web UI中表示）
- unsuccessful：
- cleanup：无论Pipeline或stage的状态如何，在跑完所有其他的`post`条件后运行此条件下 的`post`步骤。

## 3、stages（必须）

- 至少包含一个用于执行任务的`stage`指令
- `pipeline{ }`中只能有一个`stages{}`

## 4、steps（必须）

- 在`stage`指令中至少包含一个用于执行命令的`steps`

# 三、Jenkins中的变量

## 变量的来源

- Jenkins内置的环境变量
  - 构建任务相关的变量
  - 构建状态相关的变量

- 插件提供的环境变量

- pipeline中environment指令定义的变量

- 脚本自定义的变量

## 变量的引用

- $变量名
- ${变量名}
- ${env.变量名}





## 变量的处理

- ${变量名[0..7]}
- 变量名.take(8)
- ${变量名.replace(' and counting', '')}



The issue here is caused by the way Jenkins interprets `$var` inside `sh` block:

- if you use `"double quotes"`, `$var` in `sh "... $var ..."` will be interpreted as Jenkins variable;
- if you use `'single quotes'`, `$var` in `sh '... $var ...'` will be interpreted as shell variable.



## 参考

1. https://stackoverflow.com/questions/16943665/how-to-get-git-short-hash-in-to-a-variable-in-jenkins-running-on-windows-2008
2. https://stackoverflow.com/questions/44007034/conditional-environment-variables-in-jenkins-declarative-pipeline/53771302

# 四、指令Directives



## 1、Environment环境变量

environment{…},使用键值对来定义一些环境变量并赋值。它的作用范围，取决environment{…}所写的位置。写在顶层环境变量，可以让所有stage下的step共享这些变量；也可以单独定义在某一个stage下，只能供这个stage去调用变量，其他的stage不能共享这些变量。一般来说，我们基本上上定义全局环境变量，如果是局部环境变量，我们直接用def关键字声明就可以，没必要放environment{…}里面。

同时，environment{…}支持`credentials()` 方法来访问预先在Jenkins保存的凭据，并赋值给环境变量

`credentials()` 支持的凭据类型：

- Secret Text

- Secret File

- Username and password：使用`变量名_USR` and `变量名_PSW` 来获取其中的用户名和Password

  ```groovy
  pipeline {
      agent any
      stages {
          stage('Example Username/Password') {
              environment {
                  SERVICE_CREDS = credentials('my-prefined-username-password')
              }
              steps {
                  sh 'echo "Service user is $SERVICE_CREDS_USR"'
                  sh 'echo "Service password is $SERVICE_CREDS_PSW"'
                  sh 'curl -u $SERVICE_CREDS https://myservice.example.com'
              }
          }
      }
  }
  ```

- SSH with Private Key

  ```groovy
  pipeline {
      agent any
      stages {
          stage('Example Username/Password') {
              environment {
                  SSH_CREDS = credentials('my-prefined-ssh-creds')
              }
              steps {
                  sh 'echo "SSH private key is located at $SSH_CREDS"'
                  sh 'echo "SSH user is $SSH_CREDS_USR"'
                  sh 'echo "SSH passphrase is $SSH_CREDS_PSW"'
              }
          }
      }
  }
  ```

## 2、Parameters参数

- `pipeline{ }`中只能有一个`parameters{}`

### 参数定义格式

```groovy
parameters {
  参数类型(name: '参数名', defaultValue: '默认值', description: '描述')
}
```

### 参数类型

- string
- text
- boobleanParam
- choice
- password

### 参数调用格式：`${params.参数名}`

### 示例：
```groovy
pipeline {
    agent any
    parameters {
        string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
        text(name: 'BIOGRAPHY', defaultValue: '', description: 'Enter some information about the person')
        booleanParam(name: 'TOGGLE', defaultValue: true, description: 'Toggle this value')
        choice(name: 'CHOICE', choices: ['One', 'Two', 'Three'], description: 'Pick something')
        password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'Enter a password')
    }
    stages {
        stage('Example') {
            steps {
                echo "Hello ${params.PERSON}"
                echo "Biography: ${params.BIOGRAPHY}"
                echo "Toggle: ${params.TOGGLE}"
                echo "Choice: ${params.CHOICE}"
                echo "Password: ${params.PASSWORD}"
            }
        }
    }
}
```

## 3、Options选项

- `pipeline{ }`中只能有一个`options{}`

- buildDiscarder
- checkoutToSubdirectory
- disableConcurrentBuilds
- disableResume
- newContainerPerStage
- overrideIndexTriggers
- preserveStashes
- quietPeriod
- retry
- skipDefaultCheckout
- skipStagesAfterUnstable
- timeout
- timestamps
- parallelsAlwaysFailFast

## 4、Triggers触发器

- `pipeline{ }`中只能有一个`triggers {}`

触发器类型

- cron
- pollSCM
- upstream

Jenkins的Cron语法

## 5、Stage阶段(至少有一个)

- 包含在`stages{}`中
- 至少有一个

## 6、Tools工具

- 包含在`pipeline{}`或`stage{}`

支持的工具：

- Maven
- JDK
- Gradle

## 7、Input用户输入

## 8、When条件

内置条件：

- **branch**
  
  Execute the stage when the branch being built matches the branch pattern given, for example: when { branch 'master' }. Note that this only works on a multibranch Pipeline.

- **buildingTag**

  Execute the stage when the build is building a tag. Example: when { buildingTag() }

- **changelog**
  
  Execute the stage if the build’s SCM changelog contains a given regular expression pattern, for example: when { changelog '.*^\\[DEPENDENCY\\] .+$' }

- **changeset**

  Execute the stage if the build’s SCM changeset contains one or more files matching the given string or glob. Example: when { changeset "**/*.js" }

  By default the path matching will be case insensitive, this can be turned off with the caseSensitive parameter, for example: when { changeset glob: "ReadMe.*", caseSensitive: true }

- **changeRequest**

  Executes the stage if the current build is for a "change request" (a.k.a. Pull Request on GitHub and Bitbucket, Merge Request on GitLab or Change in Gerrit etc.). When no parameters are passed the stage runs on every change request, for example: when { changeRequest() }.

  By adding a filter attribute with parameter to the change request, the stage can be made to run only on matching change requests. Possible attributes are id, target, branch, fork, url, title, author, authorDisplayName, and authorEmail. Each of these corresponds to a CHANGE_* environment variable, for example: when { changeRequest target: 'master' }.

  The optional parameter comparator may be added after an attribute to specify how any patterns are evaluated for a match: EQUALS for a simple string comparison (the default), GLOB for an ANT style path glob (same as for example changeset), or REGEXP for regular expression matching. Example: when { changeRequest authorEmail: "[\\w_-.]+@example.com", comparator: 'REGEXP' }

- **environment**

  Execute the stage when the specified environment variable is set to the given value, for example: when { environment name: 'DEPLOY_TO', value: 'production' }

- **equals**

  Execute the stage when the expected value is equal to the actual value, for example: when { equals expected: 2, actual: currentBuild.number }

- **expression**

  Execute the stage when the specified Groovy expression evaluates to true, for example: when { expression { return params.DEBUG_BUILD } } Note that when returning strings from your expressions they must be converted to booleans or return null to evaluate to false. Simply returning "0" or "false" will still evaluate to "true".

- **tag**

  Execute the stage if the TAG_NAME variable matches the given pattern. Example: when { tag "release-*" }. If an empty pattern is provided the stage will execute if the TAG_NAME variable exists (same as buildingTag()).

  The optional parameter comparator may be added after an attribute to specify how any patterns are evaluated for a match: EQUALS for a simple string comparison, GLOB (the default) for an ANT style path glob (same as for example changeset), or REGEXP for regular expression matching. For example: when { tag pattern: "release-\\d+", comparator: "REGEXP"}

- **not**

  Execute the stage when the nested condition is false. Must contain one condition. For example: when { not { branch 'master' } }

- **allOf**

  Execute the stage when all of the nested conditions are true. Must contain at least one condition. For example: when { allOf { branch 'master'; environment name: 'DEPLOY_TO', value: 'production' } }

- **anyOf**

  Execute the stage when at least one of the nested conditions is true. Must contain at least one condition. For example: when { anyOf { branch 'master'; branch 'staging' } }

- **triggeredBy**

  Execute the stage when the current build has been triggered by the param given. For example:

  - when { triggeredBy 'SCMTrigger' }
  - when { triggeredBy 'TimerTrigger' }
  - when { triggeredBy 'UpstreamCause' }
  - when { triggeredBy cause: "UserIdCause", detail: "vlinde" }

# 未完待整理更新
