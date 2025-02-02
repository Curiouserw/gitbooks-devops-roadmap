# Gitlab版本升级

# 一、简介

gitlab的版本规则是：`主版本(Major).次版本(Minor).补丁版本(Patch)`

- 主版本会在每年的五月22左右发布

- 次版本会在每月22号左右发布
- 补丁版本会不定时发布

版本升级说明

- 在同一个主要版本下，补丁版本和次要版本之间跳转是安全的。例如
  - `12.7.5` -> `12.10.5`
  - `11.3.4` -> `11.11.1`
  - `12.0.4` -> `12.0.12`
  - `11.11.1` -> `11.11.8`
- 而主版本升级，需要考虑版本是否可向后兼容与数据迁移，原则上先升级到当前主版本的最大次版本，然后在升级到下个主要版本最小次版本，依次往最新的版本进行升级

说明文档：

- https://docs.gitlab.com/ee/policy/maintenance.html
- https://docs.gitlab.com/ee/update/README.html#version-specific-upgrading-instructions

# 二、升级前准备

## 1、确定版本升级路径

使用官方提供的版本升级工具：https://gitlab-com.gitlab.io/support/toolbox/upgrade-path/ 

​							https://gitlab.com/gitlab-com/support/toolbox/upgrade-path

可查看最新的版本升级路径文档：https://docs.gitlab.com/ee/update/index.html#upgrade-paths

`8.11.Z` --> `8.12.0` --> `8.17.7` --> `9.5.10` --> `10.8.7` --> `11.11.8` --> `12.0.12` --> `12.1.17` --> `12.10.14` --> `13.0.14` --> `13.1.11` -- > [latest `13.Y.Z`](https://about.gitlab.com/releases/categories/releases/)

| 当前版本 | 目标版本 | 支持的版本升级路径                                           |
| :--------- | :------- | :----------------------------------------------------------- |
| `14.6.2` | `15.1.0` | `14.6.2` -> `14.9.5` -> `14.10.5` -> `15.0.2` -> `15.1.0` |
| `14.6.2` | `15.0.0` | `14.6.2` -> `14.9.5` -> `14.10.5` -> `15.0.2` |
| `13.9.2` | `14.1.8` | `13.9.2` -> `13.12.15` -> `14.0.12` -> `14.1.8` |
| `12.9.2` | `13.4.3`   | `12.9.2` -> `12.10.14` -> `13.0.14` -> `13.4.3`              |
| `11.5.0` | `13.2.10`  | `11.5.0` -> `11.11.8` -> `12.0.12` -> `12.1.17` -> `12.10.14` -> `13.0.14` -> `13.2.10` |
| `11.3.4` | `12.10.14` | `11.3.4` -> `11.11.8` -> `12.0.12` -> `12.1.17` -> `12.10.14` |
| `10.4.5` | `12.9.5`   | `10.4.5` -> `10.8.7` -> `11.11.8` -> `12.0.12` -> `12.1.17` -> `12.9.5` |
| `9.2.6` | `12.2.5`   | `9.2.6` -> `9.5.10` -> `10.8.7` -> `11.11.8` -> `12.0.12` -> `12.1.17` -> `12.2.5` |
| `8.13.4` | `11.3.4`   | `8.13.4` -> `8.17.7` -> `9.5.10` -> `10.8.7` -> `11.3.4`     |

已实践的版本升级路线：

`11.6.1 ----> 11.11.0-ce.0(关键点) ----> 12.0.1-ce.0(关键点) ----> 12.8.0-ce.0 ----> 12.10.0(关键点) ----> 13.0.0-ce.0(关键点) ----> latest(13.8.3)`

## 2、确定升级步骤

升级的步骤取决于Gitlab的部署方式

- `二进制包安装`
  - 参考文档：https://docs.gitlab.com/omnibus/update/
- `源码编译安装`
  - 参考文档：https://docs.gitlab.com/ee/update/upgrading_from_source.html
- `Docker方式安装`
  - 参考文档：https://docs.gitlab.com/omnibus/docker/README.html#update
- `Helm方式安装`
  - 参考文档：https://docs.gitlab.com/charts/installation/upgrade.html#steps

# 三、Docker升级步骤

以升级`14.5.3 -> 14.9.5 -> 14.10.5 -> 15.0.5 -> 15.1.4`为例

`/data/gitlab/docker-compose.yml`

```yaml
web:
  image: 'gitlab/gitlab-ce:14.5.3-ce.0'
  restart: always
  container_name: gitlab
  hostname: 'gitlab.curiouser.com'
  environment:
    GITLAB_OMNIBUS_CONFIG: |
      external_url 'http://gitlab.curiouser.com'
      gitlab_rails['gitlab_shell_ssh_port'] = 30022
  ports:
    - '80:80'
    - '30022:22'
  volumes:
    - '/data/gitlab/config:/etc/gitlab'
    - '/data/gitlab/logs:/var/log/gitlab'
    - '/data/gitlab/data:/var/opt/gitlab'
```

## 1、备份数据和镜像

- 备份docker-compose整个目录

  ```bash
  mkdir /data/gitlab-backup-20220805
  cp -r /data/gitlab/ /data/gitlab-backup-20220805
  ```

- 备份docker镜像

  ```bash
  docker save -o /data/gitlab-backup-20220805/gitlab-ce-14-5-3-ce.tar gitlab/gitlab-ce:14.5.3-ce.0
  ```

## 2、修改docker-compose文件中的镜像版本

```ini
# 14.5.3 -> 14.9.5
sed -i "s/image\: \'gitlab\/gitlab-ce:14.5.3-ce.0\'/image\: \'gitlab\/gitlab-ce\:14.9.5-ce.0\'/g" docker-compose.yml

# 14.9.5 -> 14.10.5
sed -i "s/image\: \'gitlab\/gitlab-ce:14.9.5-ce.0\'/image\: \'gitlab\/gitlab-ce\:14.10.5-ce.0\'/g" docker-compose.yml

# 14.10.5 -> 15.0.5
sed -i "s/image\: \'gitlab\/gitlab-ce:14.10.5-ce.0\'/image\: \'gitlab\/gitlab-ce\:15.0.5-ce.0\'/g" docker-compose.yml

# 15.0.5 -> 15.1.4
sed -i "s/image\: \'gitlab\/gitlab-ce:15.0.5-ce.0\'/image\: \'gitlab\/gitlab-ce\:15.1.4-ce.0\'/g" docker-compose.yml
```

## 3、重启容器

```bash
cd /data/gitlab
docker-compose up -d && docker-compose logs -f
```

## 4、检查后台迁移任务是否完成

**必须确认后台迁移任务均为0时，才可以开始后续的下一个版本的升级**

- **方式一**：在Web页面的`管理中心–> 监控 –> 后台任务`中查看队列中的迁移任务

- **方式二：**

  - GitLab EE > 14.0

    ```bash
    gitlab-psql -c 'select job_class_name,table_name, column_name, job_arguments from
    batched_background_migrations where status <> 3;'
    ```

  - GitLab EE=12.9  < 14.0

    ```bash
    gitlab-rails runner -e production 'puts Gitlab::BackgroundMigration.remaining'
    ```

参考：https://docs.gitlab.com/ee/update/index.html#batched-background-migrations

## 5、重复2~4步升级至指定版本

- 每一次替换升级，都需要容器重启至可访问，后台迁移任务检查为0时方可操作下一个版本的升级

## 6、验证

- 使用账号密码看是否能正常登陆？
- 本地使用git客户端看是否能正常拉取推送代码？
- 测试各个环境的gitlab runner，提交代码看pipeline能否正常运行和部署？
- 测试钉钉通知是否正常？
- 验证SonarQube是否可以通过Gitlab认证进行登录？
- 验证新增功能

## 7、启用新功能

```bash
# 进入容器
docker exec -it gitlab bash
# 进入GitLab Rails 控制台
gitlab-rails console -e production
# 执行命令以启动新功能
Feature.enable(:performance_analytics)
```

# 四、升级失败的处理

利用**`备份数据文件`**和**`镜像文件`**重新恢复最初状态

