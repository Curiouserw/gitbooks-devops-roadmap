# Gitlab的部署与配置

# 一、Docker

Deb安装包下载地址：https://packages.gitlab.com/gitlab/

## ARM

官方gitlab-ce暂没有ARM 架构的镜像,所以由第三方的镜像进行部署

参考：https://about.gitlab.com/handbook/engineering/development/enablement/distribution/maintenance/arm.html#why-dont-you-compile-arm32-bit-packages-on-arm64-for-speed

GitHub：https://github.com/ulm0/gitlab   

DockerHub：https://hub.docker.com/r/ulm0/gitlab

```bash
mkdir -p /data/gitlab/data /data/gitlab/logs /data/gitlab/config && \
docker run -d \
--hostname 192.168.1.8 \
-e GITLAB_OMNIBUS_CONFIG="external_url 'http://192.168.1.1:38080';gitlab_rails['lfs_enabled'] = true; gitlab_rails['gitlab_shell_ssh_port'] = 30022 ; node_exporter['enable'] = true ;" \
-e RAILS_ENV="production" \
-e GITLAB_EMAIL_DISPLAY_NAME="Gitlab 13" \
-e GITLAB_EMAIL_FROM="*****@163.com" \
-e GITLAB_EMAIL_REPLY_TO="*****@163.com" \
-e GITLAB_EMAIL_SUBJECT_SUFFIX="Gitlab 13" \
-e GITLAB_ROOT_PASSWORD="*****" \
-p 38080:38080 \
-p 30022:22 \
--name gitlab \
--restart always \
--privileged \
-v /data/gitlab/config:/etc/gitlab \
-v /data/gitlab/logs:/var/log/gitlab \
-v /data/gitlab/data:/var/opt/gitlab \
ulm0/gitlab:13.2.6
```

手动构建新版本的arm gitlab docker镜像

```bash
git clone https://github.com/ulm0/gitlab.git gitlab-arm-docker
cd gitlab-arm-docker
echo "13.8.1" > VERSION
make build
```

针对raspberry的deb包下载地址：https://packages.gitlab.com/gitlab/raspberry-pi2

