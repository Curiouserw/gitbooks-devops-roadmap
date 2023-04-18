# Nginx日志写入Kafka

# 一、简介

## 方案

- **Nginx module**：编译集成第三方kafka相关模块，可直接将日志发送到kafka
- **Nginx + 第三方应用**：结合第三方应用将Nginx日志发送到kafka
  - `tail | kafkacat ———> kafka`
  - `rsyslog ———> kafka`
  - `Nginx stdout |  k8s 第三方operator ———> kafka`

# 二、Nginx module -> kafka

```bash
openssl_version=1.1.1 && \
nginx_version=1.18.0 && \
mkdir compile-dir && cd compile-dir && \
curl -s -# https://www.openssl.org/source/openssl-$openssl_version.tar.gz | tar zxf - -C ./ && \
curl -s -# https://nginx.org/download/nginx-$nginx_version.tar.gz | tar zxf - -C ./ && \
cd nginx-$nginx_version && \
./configure \
  --prefix=/opt/nginx-1.18.0 \
  --user=nginx \
  --group=nginx \
  --modules-path=/opt/nginx-1.18.0/modules \
  --sbin-path=/opt/nginx-1.18.0/sbin/nginx \
  --error-log-path=/opt/nginx-1.18.0/logs/error.log \
  --http-log-path=/opt/nginx-1.18.0/logs/access.log \
  --conf-path=/opt/nginx-1.18.0/conf/nginx.conf \
  --pid-path=/opt/nginx-1.18.0/nginx.pid \
  --lock-path=/opt/nginx-1.18.0/nginx.lock \
  --with-pcre \
  --with-openssl=../openssl-1.1.1 \
  --with-http_stub_status_module \
  --with-http_realip_module \
  --with-http_addition_module \
  --with-http_sub_module \
  --with-http_dav_module \
  --with-http_flv_module \
  --with-http_mp4_module \
  --with-http_gunzip_module \
  --with-http_gzip_static_module \
  --with-http_random_index_module \
  --with-http_secure_link_module \
  --with-http_stub_status_module \
  --with-http_auth_request_module \
  --with-http_xslt_module=dynamic \
  --with-http_image_filter_module=dynamic \
  --with-http_geoip_module=dynamic \
  --with-http_image_filter_module \
  --with-http_v2_module \
  --with-http_slice_module \
  --with-threads \
  --with-stream \
  --with-stream_ssl_module \
  --with-stream_ssl_preread_module \
  --with-stream_realip_module \
  --with-stream_geoip_module=dynamic \
  --with-mail \
  --with-mail_ssl_module \
  --with-compat && \
make && \
make install
```



参考：

- https://github.com/brg-liuwei/ngx_kafka_module
- https://github.com/kaltura/nginx-kafka-log-module

# 三、tail | kafkacat -> kafka

参考：

- https://github.com/edenhill/kafkacat

# 四、nginx -> rsyslog -> kafka

## 1、Prerequisite

- Nginx >=  v1.7.1 （之后才支持 syslog 的方式处理日志）
- Rsyslog >= v8.7.0 （数据需要通过 Rsyslog 的 omkafka 模块写入到 Kafka，omkafka 在 Rsyslog 的 v8.7.0+ 版本才支持）
- rsyslog的`omkafka`模块`rsyslog-kafka`已安装

## 2、rsyslog配置

> vi /etc/rsyslog.dc/rsyslog_nginx_kafka_cluster.conf

```bash
module(load="imudp")
input(type="imudp" port="514")

# nginx access log ==> rsyslog server(local) ==> kafka
module(load="omkafka")

template(name="nginx-rsyslog-kafka" type="string" string="%msg%")

if $inputname == "imudp" then {
    if ($programname == "nginx-rsyslog") then
        action(type="omkafka"
            template="nginx-rsyslog-kafka"
            broker=["localhost:9092"]
            topic="test"
            partitions.auto="on"
            confParam=[
                "socket.keepalive.enable=true"
            ]
        )
}

:rawmsg, contains, "nginx-rsyslog" ~
```

> 启动rsyslog

```bash
rsyslogd
```

## 3、Nginx配置

```bash
http {
	log_format json_log '{ "@timestamp": "$time_iso8601", '
                           '"app": "$app", '
                           '"remote_addr": "$remote_addr", '
                           '"referer": "$http_referer", '
                           '"request": "$request", '
                           '"status": $status, '
                           '"bytes": $body_bytes_sent, '
                           '"agent": "$http_user_agent", '
                           '"x_forwarded": "$http_x_forwarded_for", '
                           '"up_addr": "$upstream_addr",'
                           '"up_host": "$upstream_http_host",'
                           '"up_resp_time": "$upstream_response_time",'
                           '"request_time": "$request_time",'
                           '"server_name": "$server_name",'
                           '"x-zz-app-info": "$http_x_zz_app_info"'
                        ' }';
  server {
      listen       80;
      server_name  localhost;
      # set $app test;
      # access_log  /var/log/nginx/host.access.log  json_log;
      access_log syslog:server=localhost,facility=local7,tag=nginx-rsyslog,severity=info main;

      location / {
          root   /usr/share/nginx/html;
          index  index.html index.htm;
      }
  }
```

## 4、测试

> nginx的日志 ———> rsyslog —————> Kafka
>
> wrk -t10 -c500 -d30s --latency http://127.0.0.1:80
>
> 30.1秒发送了38861个请求，QPS大约1291，Transfer/sec: 1.05MB 



> nginx的日志 ————> 本地文件
>
> wrk -t10 -c500 -d30s --latency http://127.0.0.1:80
>
> 30.07秒发送了115156个请求，QPS大约3829，Transfer/sec: 3.10MB

nginx的日志通过网络发送到rsyslog再到Kafka，比落盘形成文件，QPS小了三倍。同时压测完成到所有日志进kafka有15秒延迟

**因素**：

- 网络消耗
- 磁盘性能可能导致kafka写入慢

**参考**：

- http://zhang-jc.github.io/2019/03/15/%E4%BD%BF%E7%94%A8-Rsyslog-%E5%B0%86-Nginx-Access-Log-%E5%86%99%E5%85%A5-Kafka/

# 五、Nginx stdout -> k8s 第三方operator-> kafka

参考：

- https://banzaicloud.com/docs/one-eye/logging-operator/quickstarts/kafka-nginx/

