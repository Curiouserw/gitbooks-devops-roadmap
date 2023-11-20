# Bark

Github: https://github.com/Finb/Bark

采用GB2312或GBK编码方式时，一个*中文字符占*2个*字节*；而采用UTF-8编码方式时，一个*中文字符*会占3个*字节*

加密字符有 14 个字节的限制

### 点击推送将跳转到url的地址（发送时，URL参数需要编码）

```bash
curl https://api.day.app/******/百度网址?url=https://www.baidu.com 
```

- 时效性通知

```bash
curl https://api.day.app/******/aa?level=timeSensitive
```

# 服务端

Github：https://github.com/Finb/bark-server

# Chrome 插件

Github：https://github.com/xlvecle/Bark-Chrome-Extension

Chrome应用商店：https://chrome.google.com/webstore/detail/bark/pmlkbdbpglkgbgopghdcmohdcmladeii

# 参考

- https://ios.sspai.com/post/68177
- https://github.com/xiebruce/bark-server-docker