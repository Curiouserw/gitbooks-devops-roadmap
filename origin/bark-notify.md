# Bark

# 一、简介

Github: https://github.com/Finb/Bark.

采用GB2312或GBK编码方式时，一个*中文字符占*2个*字节*；而采用UTF-8编码方式时，一个*中文字符*会占3个*字节*

点击推送将跳转到url的地址（发送时，URL参数需要编码）

```bash
curl https://api.day.app/******/百度网址?url=https://www.baidu.com 
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
BARK_KEY=1234 ./barkserver
```

## 3、配置nginx代理转发

```bash
server {
    listen 9443 ssl;
    server_name bark.test.com;
    ssl_certificate ssl/test.com/test.com.crt;
    ssl_certificate_key ssl/test.com/test.com.key;
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
curl http://127.0.0.1:8080/ping
curl http://127.0.0.1:8080/healthz
```

# 三、推送 API

## 通知类型

| 功能           | 描述                                                         | 参数      | 示例                                                         |
| -------------- | ------------------------------------------------------------ | --------- | ------------------------------------------------------------ |
| **时效性通知** | 可对通知设置中断级别，不设置为 active ，可选参数<br />active：系统立即亮屏显示通知。默认值 <br />timeSensitive：时效性通知，可在专注状态显示通知 <br />passive：仅将通知添加到通知列表，不会亮屏提醒 | **level** | `{"body": "我是重要警告", "sound": "birdsong","level": "critical","volume":"5"}` |
| 重要警告       | 重要警告会忽略静音设置和勿扰模式，通知都会提示并播放声音。音量参数 volume 不传默认值为 5，取值范围为 0-10 | **level** | `{"body": "我是重要警告", "sound": "birdsong","level": "critical","volume":"5"}` |
| 图片推送       | 再推送横幅与消息列表中显示图片，需传递图片 URL               | **image** | `{"body": "我是重要警告", "sound": "birdsong","level": "critical","volume":"5","image":"https://day.app/assets/images/avatar.jpg"}` |
| 消息复制       | 下拉推送，锁屏界面左滑查看推送时，可以先选择复制推送内容。携带 copy参数时，将只复制 copy 参数的值 | **copy**  | `{"body": "我是重要警告", "sound": "birdsong","level": "critical","volume":"5","copy":"请复制我"}` |
| 设置角标       | 可以为推送设置角标，角标可以是任意数字                       | **badge** | `{"body": "我是重要警告", "sound": "birdsong","level": "critical","volume":"5","badge":"666"}` |
| URL 推送       | 携带 URL 参数时，点击推送会跳转到这个 URL，支持跳转 URL Schema | **url**   | `{"body": "我是重要警告", "sound": "birdsong","level": "critical","volume":"5","url":"https://www.baidu.com"}` |
| 自定义推送图标 | 需 ios15 及以上，为推送设置自定义图标。设置的图标将替换为默认的 Bark 图标，会自动缓存在本机，相同的图标 URL 仅下载一次。 | **icon**  | `{"body": "我是重要警告", "sound": "birdsong","level": "critical","volume":"5","icon":"https://day.app/assets/images/avatar.jpg"}` |
| 持续响铃通知   | 持续播放响铃 30 秒，配置 level=critical 参数可在静音模式下持续响铃 | **call**  | `{"body": "我是重要警告", "sound": "birdsong","level": "critical","volume":"5","call":"1"}` |
| 消息分组       | 对消息进行分组，推送将按 group分组显示在通知中心中，也可以在历史消息列表中选择查看不同的群组 | **group** | `{"body": "我是重要警告", "sound": "birdsong","level": "critical","volume":"5","group":"监控通知"}` |

## 推送参数

| 参数            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| **title**       | 推送标题                                                     |
| **subtitle**    | 推送副标题                                                   |
| **body**        | 推送内容                                                     |
| **device_key**  | 设备key                                                      |
| **device_keys** | key 数组，用于批量推送                                       |
| **level**       | 推送中断级别。 <br />critical: 重要警告, 在静音模式下也会响铃 <br />active：默认值，系统会立即亮屏显示通知 <br />timeSensitive：时效性通知，可在专注状态下显示通知。<br /> passive：仅将通知添加到通知列表，不会亮屏提醒。 |
| **volume**      | 重要警告的通知音量，取值范围：0-10，不传默认值为5            |
| **badge**       | 推送角标，可以是任意数字                                     |
| **call**        | 传"1"时，通知铃声重复播放                                    |
| **autoCopy**    | 传"1"时， iOS14.5以下自动复制推送内容，iOS14.5以上需手动长按推送或下拉推送 |
| **copy**        | 复制推送时，指定复制的内容，不传此参数将复制整个推送内容。   |
| **sound**       | 可以为推送设置不同的铃声                                     |
| **icon**        | 为推送设置自定义图标，设置的图标将替换默认Bark图标。 图标会自动缓存在本机，相同的图标 URL 仅下载一次。 |
| **group**       | 对消息进行分组，推送将按group分组显示在通知中心中。 也可在历史消息列表中选择查看不同的群组。 |
| **ciphertext**  | 加密推送的密文                                               |
| **isArchive**   | 传 1 保存推送，传其他的不保存推送，不传按APP内设置来决定是否保存。 |
| **url**         | 点击推送时，跳转的URL ，支持URL Scheme 和 Universal Link     |
| **action**      | 传 "none" 时，点击推送不会弹窗                               |

## 1、推送加密消息

- 构建带有参数与内容的json文本 

  ```bash
  notify_content_json='{
    "url": "https://www.baidu.com",
    "body": "测试Test",
    "sound": "birdsong",
    "volume":"6"
  }'
  ```

- 使用openssl加密json文本

  ```bash
  scrtkey="客户端加密推送设置里的key"
  scrtiv="客户端加密推送设置里的IV"
  key=$(printf $scrtkey | xxd -ps -c 200)
  iv=$(printf $scrtiv | xxd -ps -c 200)
  ciphertext=$(echo -n "$notify_content_json" | openssl enc -aes-128-cbc -K $key -iv $iv | base64 | tr -d '\n\r')
  ```

- 发送

  ```bash
  curl -s -X POST \
    --resolve bark.test.com:443:192.168.1.1 \
    --data-urlencode "ciphertext=$ciphertext" \
    -H "Authorization: Basic Barkserver的HTTP Basic认证base64编码Token" \
    --data-urlencode "title=通知主题" \
    "http://127.0.0.1:8080/设备key" -o /dev/null
  ```


## 2、推送明文消息

```bash
deviceKey='客户端devicekey'
notify_content_json='{"body":"测试测试","title":"test","sound":"birdsong","level":"critical"}'
curl -s -X POST --retry 1 --connect-timeout 2 \
	-H 'Content-Type: application/json; charset=utf-8' \
	-d "$notify_content_json" \
  -H "Authorization: Basic Barkserver的HTTP Basic认证base64编码Token" \
  http://127.0.0.1:8080/$deviceKey -o /dev/null
```

## 3、批量推送明文消息

- 批量推送仅支持 push接口的post json请求
- 需 bark-server 更新至 v2.1.9

```bash
notify_content_json='{"body":"批量明文通知","title":"batchtest","sound":"birdsong","level":"critical","device_keys": ["客户端devicekey","客户端devicekey"]}'

curl -s -X POST --retry 1 --connect-timeout 2 \
	-H 'Content-Type: application/json; charset=utf-8' \
	-d "$notify_content_json" \
  -H "Authorization: Basic Barkserver的HTTP Basic认证base64编码Token" \
  "http://127.0.0.1:8080/push" -o /dev/null
```



## 4、批量推送密文消息

```bash
scrtkey="客户端加密推送设置里的key"
scrtiv="客户端加密推送设置里的IV"
key=$(printf $scrtkey | xxd -ps -c 200)
iv=$(printf $scrtiv | xxd -ps -c 200)

notify_content='{"body": "批量加密通知", "title": "通知测试", "group": "通知测试" ,"sound": "birdsong"}'
ciphertext=$(echo -n "$notify_content" | openssl enc -aes-128-cbc -K $key -iv $iv | /opt/bin/base64 | tr -d '\n')
devices_json='["客户端devicekey","客户端devicekey"]'
notify_ciphered_content="{ \"ciphertext\": \"$ciphertext\", \"device_keys\": $json_device_keys }"
curl -s -X POST --retry 1 --connect-timeout 2 \
	-H "Content-Type: application/json" \
  -H "Authorization: Basic Barkserver的HTTP Basic认证base64编码Token" \
	-d "$notify_ciphered_content" \
	"http://127.0.0.1:8080/push"
```



# 四、客户端

## 1、Chrome 插件

Github：https://github.com/xlvecle/Bark-Chrome-Extension

Chrome应用商店：https://chrome.google.com/webstore/detail/bark/pmlkbdbpglkgbgopghdcmohdcmladeii

# 参考

- https://ios.sspai.com/post/68177
- https://github.com/xiebruce/bark-server-docker