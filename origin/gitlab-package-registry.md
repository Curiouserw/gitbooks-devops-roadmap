# Gitlab的制品仓库

# 一、简介

# 二、配置

- Personal Access Token
- CI Job Token
- Deploy Token

# 三、通用文件仓库

Gitlab Docs：https://docs.gitlab.com/ee/user/packages/generic_packages/

### 开起



### 手动上传

```bash
curl --header "PRIVATE-TOKEN: <访问Token>" \
     --upload-file 要上传的文件 \
     "http://gitlab地址/api/v4/projects/仓库ID/packages/generic/包名/版本/文件名"
```

### Pipeline中上传

```bash
image: curlimages/curl:latest

stages:
  - upload
  - download
upload:
  stage: upload
  script:
    - 'curl --header "JOB-TOKEN: $CI_JOB_TOKEN" --upload-file path/to/file.txt "${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/packages/generic/my_package/0.0.1/file.txt"'
download:
  stage: download
  script:
    - 'wget --header="JOB-TOKEN: $CI_JOB_TOKEN" ${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/packages/generic/my_package/0.0.1/file.txt'
```

### 制品命令

- 包名
  - 只允许`[a-zA-Z0-9][._-]`
- 版本名
  - 格式：` X.Y.Z`,包含`[0-9][.]`，符合`/\A\d+\.\d+\.\d+\z/`正则表达式
- 文件名
  - 只允许`[a-zA-Z0-9][._-]`

# 四、Docker Image仓库

## 1、配置

Gitlab Docs ：https://docs.gitlab.com/ee/administration/packages/container_registry.html



# 五、Maven Package仓库

Gitlab Docs：https://docs.gitlab.com/ee/user/packages/maven_repository/

 `settings.xml`

```xml
<settings>
  <servers>
    <server>
      <id>gitlab-maven</id>
      <configuration>
        <httpHeaders>
          <property>
            <name>Private-Token / Deploy-Token / Job-Token </name>
            <value>YOUR_PERSONAL_ACCESS_TOKEN / YOUR_DEPLOY_TOKEN / ${env.CI_JOB_TOKEN}</value>
          </property>
        </httpHeaders>
      </configuration>
    </server>
  </servers>
</settings>
```

`pom.xml`

```xml
<repositories>
  <repository>
    <id>gitlab-maven</id>
    <url>https://gitlab.example.com/api/v4/projects/PROJECT_ID/packages/maven</url>
  </repository>
</repositories>
<distributionManagement>
  <repository>
    <id>gitlab-maven</id>
    <url>https://gitlab.example.com/api/v4/projects/PROJECT_ID/packages/maven</url>
  </repository>
  <snapshotRepository>
    <id>gitlab-maven</id>
    <url>https://gitlab.example.com/api/v4/projects/PROJECT_ID/packages/maven</url>
  </snapshotRepository>
</distributionManagement>
```

在gitlab的pipeline中pom.xml文件

```xml
<repositories>
  <repository>
    <id>gitlab-maven</id>
    <url>${env.CI_SERVER_URL}/api/v4/projects/${env.CI_PROJECT_ID}/packages/maven</url>
  </repository>
</repositories>
<distributionManagement>
  <repository>
    <id>gitlab-maven</id>
    <url>${env.CI_SERVER_URL}/api/v4/projects/${env.CI_PROJECT_ID}/packages/maven</url>
  </repository>
  <snapshotRepository>
    <id>gitlab-maven</id>
    <url>${env.CI_SERVER_URL}/api/v4/projects/${env.CI_PROJECT_ID}/packages/maven</url>
  </snapshotRepository>
</distributionManagement>
```

发布命令

```bash
mvn deploy

# 如果成功后会显示一下信息
Uploading to gitlab-maven: https://gitlab.example.com/api/v4/projects/PROJECT_ID/packages/maven/com/mycompany/mydepartment/my-project/1.0-SNAPSHOT/my-project-1.0-20200128.120857-1.jar
```









