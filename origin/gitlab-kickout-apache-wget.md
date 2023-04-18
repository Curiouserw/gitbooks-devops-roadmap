# 剥离Gitlab代码仓库中非业务代码文件

# 一、简介

​    不管使用 Jenkins、Drone CI还是 Gitlab Pipeline做 CICD 自动化，代码仓库的 根目录 总是会有一些流水线相关的配置文件。如果使用的 Git Flow不注意这个问题，会因为部署环境的不一致，这些文件在各个分支中会不一致。之后一个代码合并可就会代码冲突、合并错误，造成代码部署不到环境中。同时这些文件，对于开发者无关紧要，但是对于devops来说却是关键。所有要把这些配置文件从业务代码仓库中剥离出来，与业务代码解耦。做到开发无感知，同时又方便灵活配置。

​    Gitlab 从 12.6 版本开始，pipeline支持读取使用外部存储的`.gitlab-ci.yml`。这样就可将Gitlab 所有项目业务代码仓库中的容器化配置、k8s部署配置等非业务文件剥离出来，按照目录层级方式进行区分，放在一个静态文件服务器中，或者作为一个单独的应用代码仓库，放在到 Gitlab中，通过 Pipeline 脚本流程部署到 k8s 中的 **Apache2**服务容器中。配置好访问地址或域名，在其他业务代码仓库cicd脚本流程中需要的地方使用wget进行下载自己的 CICD 脚本文件或容器化配置。

**统一存储管理的优点：**

- 可以快速集中修改
- 不与业务代码耦合，防止各分支中这些文件差异引起的代码冲突。
- 所有文件进行git版本管理，变更有历史追溯

**统一存储管理防单点故障及安全措施**

- 跟普通业务代码应用仓库一样进行容器化部署
- 无状态，使用K8s部署多个实例
- 设置用户名密码访问下载，在Gitlab 保存密码（密码设置成保密隐藏变量）
- 限制 IP 地址段访问下载

# 二、流水线配置文件中心的部署

**以下示例操作时，是将流水线配置文件代码仓库部署到 k8s中 apace2容器中，统称为：`流水线配置文件中心`**

## 1. 目录层级

```bash
-- docker
  |-- config.sh
  |-- Dockerfile
  |-- k8s-application.tpl.yaml
-- .gitlab-ci.yml
-- application
    -- app1
      |-- .gitlab-ci.yml            
      |-- test
         |-- Dockerfile
         |-- k8s-deploymen.tpl.yaml
      |-- stg
         |-- Dockerfile
         |-- k8s-deploymen.tpl.yaml
      |-- prod
         |-- Dockerfile
         |-- k8s-deploymen.tpl.yaml
    -- app2
      |-- .gitlab-ci.yml
      |-- test
         |-- Dockerfile
         |-- k8s-deploymen.tpl.yaml
      |-- stg
         |-- Dockerfile
         |-- k8s-deploymen.tpl.yaml
      |-- prod
         |-- Dockerfile
         |-- k8s-deploymen.tpl.yaml
		....
```

## 2. docker/k8s-application.tpl.yaml

模板中的变量会在该仓库的Gitlab CICD 过程中提供。然后使用 config.sh替换该模板文件中的预置变量，生成最终能应用到 k8s 集群的文件

- **CI_PROJECT_NAME**：是该仓库的仓库名
- **IMAGE_NAME**： CICD 脚本命令拼成的镜像名
- **POD_NUM**： 在 k8s中该应用的部署个数

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: $CI_PROJECT_NAME
  namespace: tools
  labels:
    app: $CI_PROJECT_NAME
  annotations:
    kubernetes.io/change-cause: $IMAGE_NAME
spec:
  selector:
    matchLabels:
      app: $CI_PROJECT_NAME
  replicas: $POD_NUM
  template:
    metadata:
      labels:
        app: $CI_PROJECT_NAME
    spec:
      imagePullSecrets:
         - name: registry-pull-secret
      volumes:
        - name: apache2-80-conf
          configMap:
            name: $CI_PROJECT_NAME
            items:
            - key: apache2-80.conf
              path: 000-default.conf
        - name: apache2-80-auth-file
          configMap:
            name: $CI_PROJECT_NAME
            items:
            - key: apache2-80-auth-file
              path: .htpasswd
      containers:
        - name: app
          image: $IMAGE_NAME
          imagePullPolicy: IfNotPresent
          env:
            - name: K8S_POD_UID
              valueFrom:
                fieldRef:
                  fieldPath: metadata.uid
          volumeMounts:
            - name: apache2-80-conf
              mountPath: /etc/apache2/sites-enabled/000-default.conf
              subPath: 000-default.conf
            - name: apache2-80-auth-file
              mountPath: /etc/apache2/.htpasswd
              subPath: .htpasswd
          ports:
            - containerPort: 80
              protocol: TCP
              name: web
          livenessProbe:
            failureThreshold: 3
            initialDelaySeconds: 10
            periodSeconds: 10
            successThreshold: 1
            tcpSocket:
              port: 80
            timeoutSeconds: 2
          readinessProbe:
            failureThreshold: 3
            initialDelaySeconds: 10
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 2
            tcpSocket:
              port: 80
          resources: 
            limits:
              cpu: "500m"
              memory: 500Mi
            requests:
              cpu: "100m"
              memory: 100Mi
```

`configmap.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-docker-k8s-config
  namespace: tools
data:
  # http base auth基础认证密码文件内容
  apache2-80-auth-file: |
    cicd-user:"该处内容填充htpasswd工具生成的密码本"
  # Apache的配置文件，配置访问下载文件需要进行http基础认证
  apache2-80.conf: |
    <VirtualHost *:80>
        ServerAdmin localhost
        DocumentRoot /var/www/html
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        <Directory "/var/www/html">
            AuthType Basic
            AuthName "Restricted Content"
            AuthUserFile /etc/apache2/.htpasswd
            Require valid-user
        </Directory>
    </VirtualHost>
```

## 3. docker/Dockerfile

```dockerfile
FROM apache2:latest
COPY applications /var/www/html
RUN echo "$CI_COMMIT_SHORT_SHA" >/var/www/html/index.html
```

## 4. docker/config.sh

该脚本用于替换 Gitlab Runner 中的变量到模板文件预置的变量Key中

```bash
#!/bin/sh

if [ ! -n "$1" ]; then
  echo 'must input file name '
  exit 1
else
  cat $1 |
    sed 's/\$NAMESPACE'"/$NAMESPACE/g" |
    sed 's~\$IMAGE_NAME'"~$IMAGE_NAME~g" |
    sed 's~\$APPENV'"~$APPENV~g" |
    sed 's~\$POD_NUM'"~$POD_NUM~g" |
    sed 's~\$CI_PROJECT_NAME'"~$CI_PROJECT_NAME~g" |
    cat -
fi
exit 0
```

## 5. .gitlab-ci.yml

该应用的部署自动化Gitlab CICD脚本，脚本中的部分变量是在 Gitlab 通用 CICD 变量中保存的（设置的是值隐藏保密模式：会在输出时以星号代替）

- **HARBOR_TOKEN**：保存的是Harbor 拉取镜像账号的 Token
- **PIPELINE_DINGDING_ROBOT_TOKEN**：保存的是该应用部署失败时，发送钉钉通知机器人的Webhook Token

```yml
image: alpine:with-dockerctl

variables:
  DOCKER_HOST: tcp://localhost:2375
  DOCKER_DAEMON: |
    {
      "registry-mirrors": ["https://12345.mirror.aliyuncs.com"],
      "insecure-registries": ["harbor.curiouser.com"]
    }
services:
  - name: docker:18.09-dind
    command:
      - /bin/sh
      - -c
      - |
        mkdir -p /etc/docker || exit
        echo "$DOCKER_DAEMON" > /etc/docker/daemon.json || exit
        dockerd-entrypoint.sh || exit
stages:
  - build
  - deploy

#####################################################################################
###                               构建阶段脚本                                 			 ##
#####################################################################################
.k8s_build: &k8s_build
  - export RELEASE_NUM="$(date +%Y%m%d)-$(echo $CI_COMMIT_SHA | cut -c1-8)"
  - export IMAGE_NAME="harbor.curiouser.com/$CI_PROJECT_NAME/$APPENV:$RELEASE_NUM"
  - docker login harbor.curiouser.com -u gitlab-ci -p $HARBOR_TOKEN
  - docker build -t "harbor.curiouser.com/$CI_PROJECT_NAME/$APPENV:$RELEASE_NUM" . -f docker/Dockerfile
  - docker push "harbor.curiouser.com/$CI_PROJECT_NAME/$APPENV:$RELEASE_NUM"
#####################################################################################
###                                 部署阶段脚本                                     ##
#####################################################################################
.k8s_deploy: &k8s_deploy
  - export RELEASE_NUM="$(date +%Y%m%d)-$(echo $CI_COMMIT_SHA | cut -c1-8)"
  - export IMAGE_NAME=harbor.curiouser.com/$CI_PROJECT_NAME/$APPENV:$RELEASE_NUM
  - cd docker && ./config.sh ./k8s-application.tpl.yaml > k8s-application.yaml
  - cat k8s-application.yaml
  - kubectl apply -f k8s-application.yaml
  - kubectl -n $NAMESPACE rollout status --timeout=100s deployment/$CI_PROJECT_NAME || exit_code=$?
  - |
    if [ $exit_code -ne 0 ];then
      apk add curl jq --no-cache > /dev/null;
      ROLLBACK_ID=$(kubectl -n $NAMESPACE rollout undo deployment/$CI_PROJECT_NAME -ojson | jq -r '.status.observedGeneration') ;
      curl -s https://oapi.dingtalk.com/robot/send?access_token="$PIPELINE_DINGDING_ROBOT_TOKEN" -H 'Content-Type: application/json' -d '{"msgtype": "markdown","markdown": {"title": "Gitlab流水线部署失败","text": "['$CI_PROJECT_NAME']('$CI_PROJECT_URL'/-/tree/'$CI_BUILD_REF_NAME')的'$APPENV'环境第['$CI_PIPELINE_ID']('$CI_PIPELINE_URL')号流水线'$CI_JOB_STAGE'阶段失败，已回滚至最近一个稳定版本'$ROLLBACK_ID'，请检查相关错误！"},"at": {"isAtAll": true}}' > /dev/null;
      exit 1;
    fi

#####################################################################################
#                     								CICD    触发条件  							               #
#  在 master 分支代码有代码表更时触发，将构建、部署阶段脚本任务分配至 stg-k8s-runner 上运行   #
#  		同时设置了一系列环境变量，设置了最后部署阶段要手动触发																  #
#####################################################################################
build-prd:
  stage: build
  variables:
    APPENV: prod
  retry:
    max: 2
    when:
      - always
  script: *k8s_build
  only:
    - master
  tags:
    - stg-k8s-runner

deploy-prd:
  stage: deploy
  image: harbor.curiouser.com/kubectl-stg:latest
  variables:
    NAMESPACE: tools
    APPENV: prod
    POD_NUM: 2
  script: *k8s_deploy
  only:
    - master
  when: manual
  tags:
    - stg-k8s-runner
```

## 6. 配置域名（可选）

在 k8s中的设置一个 Ingress 域名，域名绑定内网IP地址至内网DNS ，确保 CICD 过程中 Pipeline 所在环境（容器或服务器） 能访问到。方法有很多种，配置Ingress域名只是一种方法。

# 三、如何使用？

其他代码仓库中如何在自己的 CICD 过程中从`流水线配置文件中心`下载自己的配置文件

## 1、gitlab 仓库 设置

<img src="../assets/gitlabsetup-app-configcenter.jpg" style="zoom:67%;" />

> http://gitlab-ci:$APP_K8S_CONFIG_CENTER@app-k8s-config-center.curiouser.com/$CI_PROJECT_NAME/.gitlab-ci.yml

## 2、Gitlab pipeline配置文件

> 自动化配置文件中心代码仓库中的：applicatlion/app1/.gitlab-ci.yml

```yaml
image: php:7.4-alpine-with-dockerctl
variables:
  DOCKER_HOST: tcp://localhost:2375
  DOCKER_DAEMON: |
    {
      "registry-mirrors": ["https://12345.mirror.aliyuncs.com"],
      "insecure-registries":["harbor.curiouser.com"]
    }

services:
  - name: docker:18.09-dind
    command:
      - /bin/sh
      - -c
      - |
        mkdir -p /etc/docker || exit
        echo "$DOCKER_DAEMON" > /etc/docker/daemon.json || exit
        dockerd-entrypoint.sh || exit
stages:
  - scan-code
  - build
  - deploy
  
scan-code:
  stage: scan-code
  image:
    name: harbor.curiouser.com/sonarscanner-cli:4.3.0.2102
    entrypoint: [""]
  variables:
    GIT_DEPTH: 0
  script:
    - sonar-scanner
        -Dsonar.host.url=http://sonarqube.curiouser.com
        -Dsonar.login=$SONARSCANNER_TOKEN
        -Dsonar.qualitygate.wait=true
        -Dsonar.qualitygate.timeout=500
        -Dsonar.projectName=$CI_PROJECT_NAME
        -Dsonar.projectKey=$CI_PROJECT_NAME
        -Dsonar.projectVersion=$CI_COMMIT_SHA
        -Dsonar.gitlab.project_id=$CI_PROJECT_ID
        -Dsonar.gitlab.commit_sha=$CI_COMMIT_SHORT_SHA
        -Dsonar.gitlab.ref_name=$CI_COMMIT_REF_NAME
        -Dsonar.links.scm=$CI_PROJECT_URL
        -Dsonar.links.ci=$CI_PIPELINE_URL
  rules:
    - if: $CI_PIPELINE_SOURCE == 'merge_request_event' && $CI_MERGE_REQUEST_TARGET_BRANCH_NAME == 'release'
  tags:
    - stg-k8s-runner

############################### K8s build scripts ###########################################################

.k8s_build: &k8s_build_scripts
  - sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories  && apk update && apk add --no-cache wget git
  - wget -q -nv -np -r -l=1 -nd --http-user=gitlab-ci --http-password=$APP_K8S_CONFIG_CENTER -R "index.html*" -P docker http://app-k8s-config-center.curiouser.com/$CI_PROJECT_NAME/$APPENV/
  - export RELEASE_NUM="$(date +%Y%m%d)-$(echo $CI_COMMIT_SHA | cut -c1-8)"
  - composer install --ignore-platform-reqs --no-dev --no-interaction -o
  - docker login harbor.curiouser.com -u gitlab-ci -p $HARBOR_TOKEN
  - docker build -t "harbor.curiouser.com/$CI_PROJECT_NAME/$APPENV:$RELEASE_NUM" . -f docker/Dockerfile
  - docker push "harbor.curiouser.com/$CI_PROJECT_NAME/$APPENV:$RELEASE_NUM"

############################### K8s Deploy scripts ###########################################################
.k8s_deploy: &k8s_deploy_scripts
  - export RELEASE_NUM="$(date +%Y%m%d)-$(echo $CI_COMMIT_SHA | cut -c1-8)"
  - export IMAGE_NAME=harbor.curiouser.com/$CI_PROJECT_NAME/$APPENV:$RELEASE_NUM
  - sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories  && apk update && apk add --no-cache wget git
  - wget -q -nv -np -r -l=1 -nd --http-user=gitlab-ci --http-password=$APP_K8S_CONFIG_CENTER -R "index.html*" -P docker http://app-k8s-config-center.curiouser.com/$CI_PROJECT_NAME/$APPENV/
  - chmod +x docker/config.sh && cd docker && ./config.sh ./k8s-deployment.tpl.yaml > k8s-application.yaml
  - ./config.sh ./k8s-cronjob.tpl.yaml > k8s-cronjob.yaml
  - cat k8s-application.yaml
  - kubectl apply -f k8s-application.yaml
  - kubectl -n $NAMESPACE rollout status --timeout=280s deployment/$CI_PROJECT_NAME || exit_code=$?
  - |
    if [ $exit_code -ne 0 ];then
      apk add curl jq --no-cache > /dev/null;
      ROLLBACK_ID=$(kubectl -n $NAMESPACE rollout undo deployment/$CI_PROJECT_NAME -ojson | jq -r '.status.observedGeneration') ;
      curl -s https://oapi.dingtalk.com/robot/send?access_token="$PIPELINE_DINGDING_ROBOT_TOKEN" -H 'Content-Type: application/json' -d '{"msgtype": "markdown","markdown": {"title": "Gitlab流水线部署失败","text": "['$CI_PROJECT_NAME']('$CI_PROJECT_URL'/-/tree/'$CI_BUILD_REF_NAME')的'$APPENV'环境第['$CI_PIPELINE_ID']('$CI_PIPELINE_URL')号流水线'$CI_JOB_STAGE'阶段失败，已回滚至最近一个稳定版本'$ROLLBACK_ID'，请检查相关错误！"},"at": {"isAtAll": true}}' > /dev/null;
      exit 1;
    fi
##################################### 测试环境构建部署触发条件 #####################################
test_build:
  stage: build
  variables:
    APPENV: test
  script: *k8s_build_scripts
  only:
    - develop
  tags:
    - stg-k8s-runner
test_deploy:
  stage: deploy
  image: harbor.curiouser.com/kubectl-stg:latest
  variables:
    NAMESPACE: application-test
    APPENV: test
    POD_NUM: 1
  script: *k8s_deploy_scripts
  only:
    - develop
  tags:
    - stg-k8s-runner
##################################### 测试环境构建部署触发条件 #####################################
##################################### STG环境构建部署触发条件 #####################################
staging_build:
  stage: build
  variables:
    APPENV: stg
  script: *k8s_build_scripts
  only:
    - release
  tags:
    - stg-k8s-runner
staging_deploy:
  stage: deploy
  image: harbor.curiouser.com/kubectl-stg:latest
  variables:
    NAMESPACE: application-stg
    APPENV: stg
    POD_NUM: 1
  script: *k8s_deploy_scripts
  only:
    - release
  tags:
    - stg-k8s-runner
##################################### STG环境构建部署触发条件 #####################################
##################################### 生产环境构建部署触发条件 #####################################
prod_build:
  stage: build
  variables:
    APPENV: prod
  script: *k8s_build_scripts
  only:
    - tags
  except:
    - release
    - develop
  tags:
    - prod-k8s-runner
prod_deploy:
  stage: deploy
  image: harbor.curiouser.com/kubectl-prod:latest
  variables:
    NAMESPACE: application-prd
    APPENV: prod
    POD_NUM: 3
  script: *k8s_deploy_scripts
  when: manual
  only:
    - tags
  except:
    - release
    - develop
  tags:
    - prod-k8s-runner
##################################### 生产环境构建部署触发条件 #####################################
```

# 四、疑问

## 1. 所有配置文件都放到 Gitlab 仓库里，为什么不直接使用，还要放到Apache2中？

Gitlab Pipeline 支持git 协议拉取其他仓库中的流水线配置文件，但是使用 git获取，就会拉取流水线配置文件中心中所有的项目流水线文件(使用 git的sparsecheckout比较麻烦)。不安全，也不够快。同时也无法完美解决谁有权限拉取的问题。

## 2. 为什么用Apache2，而不用 Nginx ？为什么用 wget，不用 curl ?

当时同时测试了Nginx和 Apache2。但是忘了 Nginx 是因为什么原因没有用。有在测试时遇到问题的或者我想起了，评论区见。

curl命令无法配置成下载整个文件夹

# 五、附录

## 1、wget命令参数详解

```bash
wget -q -nv -np -r -l=1 -nd --http-user=cicd-user --http-password=$APP_K8S_CONFIG_CENTER_TOKEN -R "index.html*" -P docker http://app-k8s-config-center.curiouser.com/test-project/test/

# -q 静默下载，不显示输出
# -nv, --no-verbose 关闭详尽输出，但不进入安静模式。
# -np, --no-parent 不追溯至父目录。
# -r 指定递归下载
# -l 最大递归深度( inf 或 0 代表无限制，即全部下载)。
# -nd, --no-directories 不创建服务器上文件所在的目录路径
# -P 指定文件存储目录 
# -R 指定不下载的文件列表,逗号分隔
# --http-user=USER        设置 http 用户名
# --http-password=PASS    设置 http 密码
```

## 2、Gitlab CICD Pipeline详解

参考本人个人博客：https://gitbook.curiouser.top/origin/gitlab-runner.html?h=gitlab%20

# 参考

- https://docs.gitlab.com/ee/ci/pipelines/settings.html

# 声明

1. **创作不易，转载请附上原文出处链接及本声明。**

2. **本文章同时会发布至个人博客站点。**

3. **如果本文章能帮到你，请至[个人博客GitHub 仓库](https://github.com/Curiouserw/gitbooks-devops-roadmap)点个 Star，或者 Buy me a Coffee。**

   <img src="https://gitbook.curiouser.top/assets/wechat-donate.jpg" style="zoom:50%;" />

