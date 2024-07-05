# Bark

# 一、简介

Github: https://github.com/Finb/Bark

采用GB2312或GBK编码方式时，一个*中文字符占*2个*字节*；而采用UTF-8编码方式时，一个*中文字符*会占3个*字节*

### 点击推送将跳转到url的地址（发送时，URL参数需要编码）

```bash
curl https://api.day.app/******/百度网址?url=https://www.baidu.com 
```

- 时效性通知

```bash
curl https://api.day.app/******/aa?level=timeSensitive
```

# 二、编译部署服务端

Github：https://github.com/Finb/bark-server

## 1、编译

```bash
brew install go-task/tap/go-task
git clone https://github.com/Finb/bark-server
cd bark-server
task linux_armv8
scp dist/bark-server_linux_armv8 目标路径
```

## 2、启动

```bash
export BARK_DEVICE_TOKEN=******
BARK_KEY=1234 ./bark-server_linux_armv8
```

## 3、配置nginx代理转发

```bash
server {
    listen 9443 ssl;
    server_name bark.test.com;
    ssl_certificate ssl/test.top/test.com.crt;
    ssl_certificate_key ssl/test.top/test.com.key;
    ssl_session_timeout 1d;
    ssl_session_cache shared:MozSSL:5m;
    ssl_session_tickets off;
    ssl_protocols TLSv1.3;
    ssl_prefer_server_ciphers off;
    add_header Strict-Transport-Security "max-age=63072000" always;
    ssl_stapling on;
    ssl_stapling_verify on;
    set $app bark ;
    error_log  /var/logs/nginx/nginx-bark-error.log;
    access_log /var/logs/nginx/nginx-bark-access.log  json_log;
    
    location / {
        log_not_found on;
        proxy_pass http://127.0.0.1:8080;
        proxy_read_timeout 300;
        proxy_connect_timeout 300;
        proxy_redirect off;
        proxy_set_header Host              $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Real-IP         $remote_addr;
    }
}
```

## 4、iptables 放行

```bash
iptables -I OVPNSI -p tcp --dport 9443 -j ACCEPT ;
```

## 5、测试

```bash
curl https://bark.test.com:9443/ping
```

# 三、推送 API

## 1、加密推送

```bash
#!/bin/bash
bark_server_host='https://bark.test.com:9443'
deviceKey='手机里面的APP的deviceKey'

key='APP'
iv='******'

sedcipherednotify(){
    key=$(printf $key | xxd -ps -c 200)
    iv=$(printf $iv | xxd -ps -c 200)
    notify_content_json='{"body": "测试测试测试测试测试测试测试测试测试测试测试", "sound": "birdsong"}'
    ciphertext=$(echo -n "$notify_content_json" | openssl enc -aes-128-cbc -K $key -iv $iv | base64 | tr -d '\r\n')
    curl -i -L -X POST \
      --resolve bark.test.com:9443:192.168.1.1
      --data-urlencode "ciphertext=$ciphertext" \
      --data-urlencode "title=路由器通知" \
      --data-urlencode "iv=ZDM....2NzYwY1W" \
      $bark_server_host/$deviceKey
}

sedcipherednotify
```

# 四、客户端

## 1、Chrome 插件

Github：https://github.com/xlvecle/Bark-Chrome-Extension

Chrome应用商店：https://chrome.google.com/webstore/detail/bark/pmlkbdbpglkgbgopghdcmohdcmladeii

# 参考

- https://ios.sspai.com/post/68177
- https://github.com/xiebruce/bark-server-docker