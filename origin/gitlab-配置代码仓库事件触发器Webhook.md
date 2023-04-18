# Gitlab 代码仓库配置事件Webhoook触发器

# 一、Overviews

Webhook与异步编程中"订阅-发布模型"非常类似，一端触发事件，一端监听执行。

WebHook就是一个接收HTTP POST（或GET，PUT，DELETE）的URL

通常来说，WebHook通过HTTP协议将请求数据发送到外部系统后，就不再处理响应数据啦！

# 二、Gitlab事件触发器配置Webhook

当Gitlab中代码仓库发生了一些事件后，可设置触发器发送http通知到外部系统。通常配置Jenkins上的Webhook

1. 配置Gitlab的网络限制

   ![](../assets/gitlab-配置webhook-1.png)
    否则后续发送webhook请求时会报错"Requests to the local network are not allowed",原因详见：[附录 ](#附录：gitlab发送webhook请求时报错原因)

2. 配置代码代码仓库的Webhook

    ![配置代码代码仓库的Webhook](../assets/gitlab-配置webhook-2.png)

3. 配置完可以模拟一些事件进行测试

   ![](../assets/gitlab-配置webhook-3.png)

4. 查看详细的Webhook请求信息
   
   ![](../assets/gitlab-配置webhook-4.png)
   ![](../assets/gitlab-配置webhook-5.png)

# 附录：gitlab发送webhook请求时报错原因

```yaml
If you have non-GitLab web services running on your GitLab server or within its local network, these may be vulnerable to exploitation via Webhooks.

With Webhooks, you and your project maintainers and owners can set up URLs to be triggered when specific things happen to projects. Normally, these requests are sent to external web services specifically set up for this purpose, that process the request and its attached data in some appropriate way.

Things get hairy, however, when a Webhook is set up with a URL that doesn't point to an external, but to an internal service, that may do something completely unintended when the webhook is triggered and the POST request is sent.

Because Webhook requests are made by the GitLab server itself, these have complete access to everything running on the server (http://localhost:123) or within the server's local network (http://192.168.1.12:345), even if these services are otherwise protected and inaccessible from the outside world.

If a web service does not require authentication, Webhooks can be used to trigger destructive commands by getting the GitLab server to make POST requests to endpoints like "http://localhost:123/some-resource/delete".

To prevent this type of exploitation from happening, starting with GitLab 10.6, all Webhook requests to the current GitLab instance server address and/or in a private network will be forbidden by default. That means that all requests made to 127.0.0.1, ::1 and 0.0.0.0, as well as IPv4 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16 and IPv6 site-local (ffc0::/10) addresses won't be allowed.

This behavior can be overridden by enabling the option "Allow requests to the local network from hooks and services" in the "Outbound requests" section inside the Admin area under Settings (/admin/application_settings):

```