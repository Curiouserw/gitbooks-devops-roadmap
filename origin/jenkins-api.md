# Jenkins API 

# 一、简介



Jenkins API支持以下3种格式：

- XML
- JSON并支持JSONP跨域访问
- Python

 https://jenkins-host/api/ 



Jenkins API没有统一的入口，而是采用“…/api/” 的REST API样式，其中”…” 表示Jenkins资源的URL



API类型	说明
JobsAPI	任务管理（任务信息、创建、修改）
PluginManagerAPI	插件管理（插件信息、安装插件）
QueueAPI	任务队列相关（队列状态）
StatisticsAPI	Jenkins统计信息
CrumbIssuerAPI	系统哈希值信息（用于防御CSRF攻击）
SystemAPI	Jenkins系统状态（版本、路径）



`Jenkins` 使用 `Baisc Auth` 的权限验证方式，需要传入 `username` 和 `api token` 。 

但在 `Job` 的远程触发中，可以设置用于远程触发的 `token` (在 `Job` 的配置页面设置)，这样在触发 `Job` 时就不需要传入 `Basic Auth` 了。
远程触发的 `token` 使用 `urlencode` 的方式放在请求的 `body` 中，其原始数据为： `token=` 

- Basic Auth

  `curl -X POST /view//job//build --user :`

- Token

  `curl -X POST /view//job//build --data-urlencode token=`



_class": "org.jenkinsci.plugins.workflow.job.WorkflowJob"

"_class": "hudson.model.FreeStyleProject"



# 二、API

## 1. 创建Job



## 2. 创建视图



## 3. 触发Jo构建



## 4. 