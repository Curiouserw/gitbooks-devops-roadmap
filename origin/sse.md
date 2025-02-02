# SSE(Server-Sent Events)

# 一、简介

SSE 是基于传统的 HTTP 协议实现的，采用了长轮询（long-polling）机制。客户端通过向服务器发送一个 HTTP 请求，服务器保持连接打开并周期性地向客户端发送数据。

SSE 通过 EventSource 对象来实现，在客户端可以通过监听 onmessage 事件来接收服务器端发送的数据。

# 参考

- https://developer.mozilla.org/zh-CN/docs/Web/API/Server-sent_events/Using_server-sent_events
- https://blog.csdn.net/xuejianxinokok/article/details/136830066
- https://www.zixi.org/archives/517.html
- https://juejin.cn/post/7197116653196001340
- https://wangdoc.com/webapi/server-sent-events
- https://blog.yuanpei.me/posts/3175881014/