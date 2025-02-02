# ModSecurity3 + OWASP CRS

# 一、ModSecurity简介

**ModSecurity是一个开源的跨平台Web应用程序防火墙（WAF）引擎，用于Apache，IIS和Nginx，由Trustwave的SpiderLabs开发。作为WAF产品，ModSecurity专门关注HTTP流量，当发出HTTP请求时，ModSecurity检查请求的所有部分，如果请求是恶意的，它会被阻止和记录。**

完美兼容nginx，是nginx官方推荐的WAF

支持OWASP规则

3.0版本比老版本更新更快，更加稳定，并且得到了nginx、Inc和Trustwave等团队的积极支持



中文社区：http://www.modsecurity.cn/



## ModSecurity的功能

-  **SQL Injection (SQLi)：**阻止SQL注入
-  **Cross Site Scripting (XSS)：**阻止跨站脚本攻击
-  **Local File Inclusion (LFI)：**阻止利用本地文件包含漏洞进行攻击
-  **Remote File Inclusione(RFI)：**阻止利用远程文件包含漏洞进行攻击
-  **Remote Code Execution (RCE)：**阻止利用远程命令执行漏洞进行攻击
-  **PHP Code Injectiod：**阻止PHP代码注入
-  **HTTP Protocol Violations：**阻止违反HTTP协议的恶意访问
-  **HTTPoxy：**阻止利用远程代理感染漏洞进行攻击
-  **Shellshock：**阻止利用Shellshock漏洞进行攻击
-  **Session Fixation：**阻止利用Session会话ID不变的漏洞进行攻击
-  **Scanner Detection：**阻止黑客扫描网站
-  **Metadata/Error Leakages：**阻止源代码/错误信息泄露
-  **Project Honey Pot Blacklist：**蜜罐项目黑名单
-  **GeoIP Country Blocking：**根据判断IP地址归属地来进行IP阻断

# 二、OWASP CRS(CoreRuleSet)

## 1、简介

OWASP® CRS 是一组通用攻击检测规则，可与 ModSecurity 或兼容的 Web 应用程序防火墙一起使用。 CRS 旨在保护 Web 应用程序免受各种攻击（包括 OWASP 十大攻击），并尽量减少误报。 CRS 提供针对许多常见攻击类别的保护。

**OWASP CRS  Github：**https://github.com/coreruleset/coreruleset

**OWASP CRS  文档：**https://coreruleset.org/docs/deployment/

**OWASP CRS  Docker：**https://hub.docker.com/r/owasp/modsecurity-crs

## 2、CRS 规则生效的原理





## 3、CRS 相关定义

### 3.1 偏执级别



### 3.2 异常阈值

### 3.3 自带的规则集

| 规则组名称                                                   | **说明**                         |
| ------------------------------------------------------------ | -------------------------------- |
| **[REQUEST-911-METHOD-ENFORCEMENT](https://learn.microsoft.com/zh-cn/azure/web-application-firewall/ag/application-gateway-crs-rulegroups-rules?tabs=drs21#crs911-30)** | 锁定方法（PUT、PATCH）           |
| **[REQUEST-913-SCANNER-DETECTION](https://learn.microsoft.com/zh-cn/azure/web-application-firewall/ag/application-gateway-crs-rulegroups-rules?tabs=drs21#crs913-32)** | 防范端口和环境扫描程序           |
| **[REQUEST-920-PROTOCOL-ENFORCEMENT](https://learn.microsoft.com/zh-cn/azure/web-application-firewall/ag/application-gateway-crs-rulegroups-rules?tabs=drs21#crs920-32)** | 防范协议和编码问题               |
| **[REQUEST-921-PROTOCOL-ATTACK](https://learn.microsoft.com/zh-cn/azure/web-application-firewall/ag/application-gateway-crs-rulegroups-rules?tabs=drs21#crs921-32)** | 防范标头注入、请求走私和响应拆分 |
| **[REQUEST-930-APPLICATION-ATTACK-LFI](https://learn.microsoft.com/zh-cn/azure/web-application-firewall/ag/application-gateway-crs-rulegroups-rules?tabs=drs21#crs930-32)** | 防范文件和路径攻击               |
| **[REQUEST-931-APPLICATION-ATTACK-RFI](https://learn.microsoft.com/zh-cn/azure/web-application-firewall/ag/application-gateway-crs-rulegroups-rules?tabs=drs21#crs931-32)** | 防范远程文件包含 (RFI) 攻击      |
| **[REQUEST-932-APPLICATION-ATTACK-RCE](https://learn.microsoft.com/zh-cn/azure/web-application-firewall/ag/application-gateway-crs-rulegroups-rules?tabs=drs21#crs932-32)** | 防范远程代码执行攻击             |
| **[REQUEST-933-APPLICATION-ATTACK-PHP](https://learn.microsoft.com/zh-cn/azure/web-application-firewall/ag/application-gateway-crs-rulegroups-rules?tabs=drs21#crs933-32)** | 防范 PHP 注入攻击                |
| **[REQUEST-941-APPLICATION-ATTACK-XSS](https://learn.microsoft.com/zh-cn/azure/web-application-firewall/ag/application-gateway-crs-rulegroups-rules?tabs=drs21#crs941-32)** | 防范跨站点脚本攻击               |
| **[REQUEST-942-APPLICATION-ATTACK-SQLI](https://learn.microsoft.com/zh-cn/azure/web-application-firewall/ag/application-gateway-crs-rulegroups-rules?tabs=drs21#crs942-32)** | 防范 SQL 注入攻击                |
| **[REQUEST-943-APPLICATION-ATTACK-SESSION-FIXATION](https://learn.microsoft.com/zh-cn/azure/web-application-firewall/ag/application-gateway-crs-rulegroups-rules?tabs=drs21#crs943-32)** | 防范会话固定攻击                 |
| **[REQUEST-944-APPLICATION-ATTACK-JAVA](https://learn.microsoft.com/zh-cn/azure/web-application-firewall/ag/application-gateway-crs-rulegroups-rules?tabs=drs21#crs944-32)** | 防范 JAVA 攻击                   |
|                                                              |                                  |

### 3.4 请求处理阶段

- **请求头阶段**

  Apache完成读取请求头（post-read-request阶段）后立即处理此阶段中的规则。在这个时候还没有读取请求体，所以这个时候对所有请求参数并不是都可用。如果需要提前运行规则（在Apache对请求执行某些操作之前），在请求体读取之前执行某些操作，您需确定是否应该缓存请求体，或者决定如何将规则置于此阶段（例如，是否将其解析为XML）。

- **请求体阶段**

  请求体阶段是通用的输入分析阶段，大多数面向应用程序的规则都放在这里。在这个阶段是可以收到所有的请求参数，当然这个前提是请求主体是已经读取的。在此阶段ModSecurity支持请求正文阶段的三种编码类型：

  - **application/x-www-form-urlencoded** 用于传输表单数据
  - **multipart/form-data** 用于文件上传
  - **text/xml** 用于传递XML数据

- **响应头阶段**

  此阶段发生在响应头被发送回客户端之前。如果要在此之前检查响应，以及是否要使用响应头来决定是否要缓存响应体，可以讲将规则配置为此阶段。需要注意的是，某些响应状态码（例如404）由于在Apache请求周期的早期处理，因此此类访问是无法触发ModSecurity的规则。此外，还有一些响应头由Apache在稍后的挂钩（例如日期，服务器和连接）中添加，ModSecurity也无法触发或检查。这应该在代理设置中或阶段5（日志记录）中进行配置。

- **响应体阶段**

  响应体阶段是通用输出分析阶段。此时的响应体是应该要已经被缓存的，然后运行规则检查响应体。在此阶段，可以检查出站HTML来判断是否有信息泄露，是否包含错误消息或是失败的身份验证文本。

- **记录阶段**

  该阶段在日志记录发生之前运行。进入此阶段的规则只会影响日志记录的执行方式。此阶段可用于检查Apache记录的错误消息。但是不能在此阶段拒绝/阻断连接，因为已经太晚了。此阶段还允许检查在阶段3或者阶段4期间不可用的其他响应头。需要注意的是，在这个阶段不要将拒绝/阻断操作继承到规则中，因为这样本身是一个配置的错误。

### 3.5 严重等级对应异常分数

| 严重程度     | 默认异常分数 |
| :----------- | :----------- |
| **CRITICAL** | 5            |
| **ERROR**    | 4            |
| **WARNING**  | 3            |
| **NOTICE**   | 2            |







- https://www.nginx-cn.net/blog/f5-nginx-modsecurity-waf-transitioning-to-eol/
- https://docs.redhat.com/zh_hans/documentation/red_hat_jboss_core_services/2.4.57/html/red_hat_jboss_core_services_modsecurity_guide/ref_example-of-a-complex-modsecurity-rule_assembly_creating-modsecurity-rules
- http://www.modsecurity.cn/practice/post/9.html

# 三、ModSecurity安装配置集成

## 1、编译安装

## 2、配置

## 3、集成 Nginx

## 4、集成 CRS

## 5、自定义配置

http://www.modsecurity.cn/chm/index.html

## 6、远程加载CRS规则集

在安装成功ModSecurity之后，使用`SecRemoteRules`指令，将远程规则文件添加到modsecurity.conf文件中即可，如下所示：

```bash
Include /usr/local/nginx/conf/modsecurity/crs-setup.conf
SecRemoteRules some-key https://rules.modsecurity.cn/rules/customize.conf
Include /usr/local/nginx/conf/modsecurity/rules/*.conf
```

上述规则表示，将OWASP的规则放置在本地，然后远程服务器放置了一个customize.conf的规则文件，文件中主要放置一些更新比较频繁的自定义规则，此时请注意！之所以将加载远程规则的语句放置在第二位，是因为首先要加载OWASP规则的默认配置，即crs-setup.conf。

some-key意味着可以定义一个key值，远程服务器可以根据key值，来判断要返回哪些规则内容，或是否返回规则内容，当然，随便定义一个key值，远程服务器不进行任何判断也可以。



也可以直接将OWASP的所有规则全部放置在远程服务器当中，然后通过以下方式进行加载（内容需复制到modsecurity.conf末尾）：

```bash
SecRemoteRules 123 https://rules.modsecurity.cn/crs-setup.conf
SecRemoteRules 123 https://rules.modsecurity.cn/rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf
SecRemoteRules 123 https://rules.modsecurity.cn/rules/REQUEST-901-INITIALIZATION.conf
SecRemoteRules 123 https://rules.modsecurity.cn/rules/REQUEST-903.9001-DRUPAL-EXCLUSION-RULES.conf
.................
```

但此时请注意，由于OWASP规则中除了规则文件外，还有一些后缀为`.data`的数据文件。如果`.data`数据文件也放置在了远程服务器中，那么引用这些数据的规则（通过pmFromFile指令进行引用），也需要把路径改为远程服务器的路径，如REQUEST-913-SCANNER-DETECTION.conf文件中ID为913100的规则，原内容如下：

```bash
SecRule REQUEST_HEADERS:User-Agent "@pmFromFile scanners-user-agents.data" \
    "id:913100,\
    phase:2,\
    block,\
    capture,\
    t:none,t:lowercase,\
    msg:'Found User-Agent associated with security scanner',\
    logdata:'Matched Data: %{TX.0} found within %{MATCHED_VAR_NAME}: %{MATCHED_VAR}',\
    tag:'application-multi',\
    tag:'language-multi',\
    tag:'platform-multi',\
    tag:'attack-reputation-scanner',\
    tag:'paranoia-level/1',\
    tag:'OWASP_CRS',\
    tag:'OWASP_CRS/AUTOMATION/SECURITY_SCANNER',\
    tag:'WASCTC/WASC-21',\
    tag:'OWASP_TOP_10/A7',\
    tag:'PCI/6.5.10',\
    ver:'OWASP_CRS/3.2.0',\
    severity:'CRITICAL',\
    setvar:'tx.anomaly_score_pl1=+%{tx.critical_anomaly_score}',\
    setvar:'ip.reput_block_flag=1',\
    setvar:'ip.reput_block_reason=%{rule.msg}',\
    expirevar:'ip.reput_block_flag=%{tx.reput_block_duration}'"
```

此时就需要修改scanners-user-agents.data数据文件的路径，即`"@pmFromFile scanners-user-agents.data"`，需要改成`"@pmFromFile https://rules.modsecurity.cn/rules/scanners-user-agents.data"`



使用SecRemoteRules后，WEB服务在启动时，ModSecurity会访问配置的HTTPS路径来读取远程文件中的规则，而与此同时，请求头中会增加三个参数：`ModSec-unique-id`、`ModSec-status`、`ModSec-key`

```bash
GET /plain-text-rules.txt HTTP/1.1
User-Agent: modesecurity
Host: www.yourserver.com
Accept: */*
ModSec-unique-id: e781ee72979d16cd91ed89c716dc087f195d7cb8
ModSec-status: 2.9.3,(null),1.5.2/1.5.2,8.43/8.43 2019-02-23,Lua 5.1,2.9.9,e781ee72979d16cd91ed89c716dc087f195d7cb8
ModSec-key: some-key
```

其中，ModSec-key的值就是定义的key值，在实际中将其设置为了some-key



**注意**

- 目前只有ModSecurity V3版本支持配置多个SecRemoteRules指令，其他版本只能配置一条SecRemoteRules指令，也就是说，如果使用的不是ModSecurity V3版本，所有规则都需要放置在一个文件中，且规则必须一行一个，不能像OWASP规则那样，一条规则可以通过"\"来拆分成多行；
- ModSecurity 2.9.3无法使用SecRemoteRules指令，ModSecurity 2.9.2在与Apache搭配使用时可使用SecRemoteRules指令。

# 四、ModSecurity日志详解

## 1、日志配置

> modsecurity.conf

```ini
SecAuditEngine On|Off|RelevantOnly
SecAuditLog /path/to/audit.log
SecAuditLogParts ABIJDEFHZ
SecAuditLogType Serial|Concurrent 
SecAuditLogStorageDir /path/to/storage/dir 
SecAuditLogRelevantStatus "^(?:5|4(?!04))"
```

- **SecAuditEngine：**是否开启日志审计
  - **Off：** 禁用审核日志
  - **On：**记录所有事务，调试时有用。如果要了解黑客是如何利用漏洞进行入侵的，还是记录所有的日志。
  - **RelevantOnly：** 仅记录`指定HTTP状态码`的访问，与SecAuditLogRelevantStatus搭配使用。
- **SecAuditLogRelevantStatus：**如果 `SecAuditEngine` 设置为 `RelevantOnly`，则此指令控制应记录哪些 HTTP 响应状态代码。它是基于正则表达式的。上述值将记录所有 5xx 和 4xx 响应，不包括 404。
- **SecAuditLogParts：**设置日志记录的内容，建议配置为`ABCDEFHZ`，每字母代表的内容详见日志格式中的配置。当记录请求体时，建议使用C来记录请求体，不然日志会记录完整的请求体中的内容。在ModSecurity v3版本中，使用IJ无效，只能使用C来记录请求体。
- **SecAuditLogType：**日志记录方式
  - **Serial ：** 将日志记录到单个日志文件中，与SecAuditLog搭配使用。如果使用串行方式，服务器内网站过多或访问量过大时，日志文件会越来越大，不利于内容的检索
  - **Concurrent(推荐) ：**并行记录文件，每一次访问都会生出一个单独文件进行记录，与SecAuditLogStorageDir搭配使用

- **SecAuditLogStorageDir：**设置日志保存的目录，当SecAuditLogType设置为Concurrent时方可生效，注意，请确保配置的目录存在并赋予777权限，否则将无法记录日志
- **SecAuditLog：**设置日志保存的文件路径，注意，请确保文件所在目录存在并赋予777权限，否则将无法记录日志；

## 2、日志格式

| Section | Description                                                  |
| ------- | ------------------------------------------------------------ |
| A       | 日志标题（强制）                                             |
| B       | 请求 headers                                                 |
| C       | 请求 body                                                    |
| D       | 暴露                                                         |
| E       | 响应 body                                                    |
| F       | 响应 headers                                                 |
| G       | 暴露                                                         |
| H       | 日志trailer，其中包含附加数据                                |
| I       | Compact request body alternative (to part C), which excludes files |
| J       | 上传文件的信息                                               |
| K       | 包含与事务匹配的所有规则的列表                               |
| Z       | 日志最终边界（强制）                                         |

```bash
---ICmPEb5c---A--[04/Oct/2017:21:45:15 +0000] 150715351558.929952 141.212.122.16 64384 141.212.122.16 80
---ICmPEb5c---B--
GET / HTTP/1.1
Host: 54.183.57.254
User-Agent: Mozilla/5.0 zgrab/0.x
Accept-Encoding: gzip

---ICmPEb5c---D--

---ICmPEb5c---F--
HTTP/1.1 200
Server: nginx/1.13.4
Date: Wed, 04 Oct 2017 21:45:15 GMT
Content-Type: text/html
Connection: keep-alive

---ICmPEb5c---H--
ModSecurity: Warning. Matched "Operator `Rx' with parameter `^[d.:]+$' against variable `REQUEST_HEADERS:Host' (Value: `54.183.57.254' ) [file "/root/owasp
-v3/rules/REQUEST-920-PROTOCOL-ENFORCEMENT.conf"] [line "733"] [id "920350"] [rev "2"] [msg "Host header is a numeric IP address"] [data "54.183.57.254"] [s
everity "4"] [ver "OWASP_CRS/3.0.0"] [maturity "9"] [accuracy "9"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-prot
ocol"] [tag "OWASP_CRS/PROTOCOL_VIOLATION/IP_HOST"] [tag "WASCTC/WASC-21"] [tag "OWASP_TOP_10/A7"] [tag "PCI/6.5.10"] [ref "o0,13v21,13"]

---ICmPEb5c---I--

---ICmPEb5c---J--

---ICmPEb5c---Z--
```

# 四、模拟测试



不同ModSecurity版本的Nginx的并发性能测试结果 ：http://www.modsecurity.cn/practice/post/25.html







# 参考

- https://xieshaohu.wordpress.com/2022/10/09/%E5%9F%BA%E4%BA%8E-nginx-%E7%9A%84-waf-%E9%85%8D%E7%BD%AE%E6%96%B9%E6%B3%95%E4%B9%8Bmodsecurity/
- http://www.modsecurity.cn/practice/post/9.html
- http://www.modsecurity.cn/practice/post/29.html
- https://www.nginx-cn.net/blog/f5-nginx-modsecurity-waf-transitioning-to-eol/
- https://www.freebuf.com/sectool/211354.html
- https://www.freebuf.com/articles/web/279881.html
- https://www.freebuf.com/articles/network/362991.html
- https://www.freebuf.com/articles/web/338654.html
- http://www.modsecurity.cn/practice/post/9.html
- https://www.freebuf.com/articles/web/227217.html
- https://www.freebuf.com/articles/web/320836.html
- https://www.f5.com/company/blog/nginx/modsecurity-logging-and-debugging