# Filebeat的Nginx模块

# 一、简介

Filebeat的Nginx Module模块可直接用来处理Nginx标准格式的访问日志和错误日志。

# 二、启用Nginx模块

## 1. 加载Modules

在`/etc/filebeat/filebeat.yml`中配置加载Modules

```yaml
filebeat.config.modules:
  path: ${path.config}/modules.d/*.yml
  reload.enabled: true
  reload.period: 10s
```

指定特殊的filebeat全局配置文件来配置加载Modules

```bash
filebeat -c /etc/filebeat/filebeat-kong.yaml modules enable nginx
```

## 2. 启用Nginx 模块

```bash
filebeat modules enable nginx
```

# 三、配置Nginx模块变量参数

## 1.  配置Nginx 模块变量参数的方式

- Nginx module的配置文件`/etc/filebeat/modules.d/nginx.yml`

  ```yaml
  - module: nginx
    # 设置Nginx访问日志
    access:
      input:
    	  close_eof: true
      enabled: true
      # 日志文件路径。如果为空，默认根据操作系统版本自行选择日志文件路径.
      var.paths: [ "/var/log/nginx/access.log", "/var/log/nginx/admin_access.log" ]
    # 设置Nginx错误日志
    error:
      enabled: true
      # 日志文件路径。如果为空，默认根据操作系统版本自行选择日志文件路径.
      var.paths: ["/var/log/nginx/error.log"]
  ```

- 在运行时

  ```bash
  filebeat -e \
  --modules nginx \
  -M "nginx.access.var.paths=[ "/var/log/nginx/access.log", "/var/log/nginx/admin_access.log" ]" \
  -M "nginx.error.var.paths=["/var/log/nginx/error.log"]" \
  -M "nginx.access.input.close_eof=true"
  ```

# 四、Nginx模块配置文件详解

## 1. Nginx模块配置目录结构

Nginx模块中所有的配置文件在`/usr/share/filebeat/module/nginx`路径下

```bash
/usr/share/filebeat/module/nginx
├── access
│   ├── config
│   │   └── nginx-access.yml
│   ├── ingest
│   │   └── default.json
│   └── manifest.yml
├── error
│   ├── config
│   │   └── nginx-error.yml
│   ├── ingest
│   │   └── pipeline.json
│   └── manifest.yml
└── module.yml
```

## 2. module.yml

```yaml
dashboards:
- id: 55a9e6e0-a29e-11e7-928f-5dbe6f6f5519
  file: Filebeat-nginx-overview.json
- id: 046212a0-a2a1-11e7-928f-5dbe6f6f5519
  file: Filebeat-nginx-logs.json
- id: ML-Nginx-Access-Remote-IP-Count-Explorer
  file: ml-nginx-access-remote-ip-count-explorer.json
- id: ML-Nginx-Remote-IP-URL-Explorer
  file: ml-nginx-remote-ip-url-explorer.json
```

## 3. access/manifest.yml

```yml
module_version: "1.0"

var:
  - name: paths
    default:
      - /var/log/nginx/access.log*
    os.darwin:
      - /usr/local/var/log/nginx/access.log*
    os.windows:
      - c:/programdata/nginx/logs/*access.log*

ingest_pipeline: ingest/default.json
input: config/nginx-access.yml

machine_learning:
- name: response_code
  job: machine_learning/response_code.json
  datafeed: machine_learning/datafeed_response_code.json
  min_version: 5.5.0
- name: low_request_rate
  job: machine_learning/low_request_rate.json
  datafeed: machine_learning/datafeed_low_request_rate.json
  min_version: 5.5.0
- name: remote_ip_url_count
  job: machine_learning/remote_ip_url_count.json
  datafeed: machine_learning/datafeed_remote_ip_url_count.json
  min_version: 5.5.0
- name: remote_ip_request_rate
  job: machine_learning/remote_ip_request_rate.json
  datafeed: machine_learning/datafeed_remote_ip_request_rate.json
  min_version: 5.5.0
- name: visitor_rate
  job: machine_learning/visitor_rate.json
  datafeed: machine_learning/datafeed_visitor_rate.json
  min_version: 5.5.0

requires.processors:
- name: user_agent
  plugin: ingest-user-agent
- name: geoip
  plugin: ingest-geoip
```

## 4. access/config/nginx-access.yml   

```
type: log
paths:
{{ range $i, $path := .paths }}
 - {{$path}}
{{ end }}
exclude_files: [".gz$"]
processors:
- add_locale: ~
```

## 5.  access/ingest/default.json   

```json
{
    "description": "Pipeline for parsing Nginx access logs. Requires the geoip and user_agent plugins.",
    "processors": [
        {
            "grok": {
                "field": "message",
                "patterns": [
                    "\"?(?:%{IP_LIST:nginx.access.remote_ip_list}|%{DATA:source.address}) - %{DATA:user.name} \\[%{HTTPDATE:nginx.access.time}\\] \"%{DATA:nginx.access.info}\" %{NUMBER:http.response.status_code:long} %{NUMBER:http.response.body.bytes:long} \"%{DATA:http.request.referrer}\" \"%{DATA:user_agent.original}\""
                ],
                "pattern_definitions": {
                    "IP_LIST": "%{IP}(\"?,?\\s*%{IP})*"
                },
                "ignore_missing": true
            }
        },
        {
            "grok": {
                "field": "nginx.access.info",
                "patterns": [
                    "%{WORD:http.request.method} %{DATA:url.original} HTTP/%{NUMBER:http.version}",
                    ""
                ],
                "ignore_missing": true
            }
        },
        {
            "remove": {
                "field": "nginx.access.info"
            }
        },
        {
            "split": {
                "field": "nginx.access.remote_ip_list",
                "separator": "\"?,?\\s+",
                "ignore_missing": true
            }
        },
        {
            "split": {
                "field": "nginx.access.origin",
                "separator": "\"?,?\\s+",
                "ignore_missing": true
            }
        },
        {
            "set": {
                "field": "source.ip",
                "value": ""
            }
        },
        {
            "script": {
                "lang": "painless",
                "source": "boolean isPrivate(def dot, def ip) { try { StringTokenizer tok = new StringTokenizer(ip, dot); int firstByte = Integer.parseInt(tok.nextToken());       int secondByte = Integer.parseInt(tok.nextToken());       if (firstByte == 10) {         return true;       }       if (firstByte == 192 && secondByte == 168) {         return true;       }       if (firstByte == 172 && secondByte >= 16 && secondByte <= 31) {         return true;       }       if (firstByte == 127) {         return true;       }       return false;     } catch (Exception e) {       return false;     }   }   try {    ctx.source.ip = null;    if (ctx.nginx.access.remote_ip_list == null) { return; }    def found = false;    for (def item : ctx.nginx.access.remote_ip_list) {        if (!isPrivate(params.dot, item)) {            ctx.source.ip = item;            found = true;            break;        }    }    if (!found) {     ctx.source.ip = ctx.nginx.access.remote_ip_list[0];   }} catch (Exception e) { ctx.source.ip = null; }",
                "params": {
                    "dot": "."
                }
            }
        },
        {
            "remove": {
                "field": "source.ip",
                "if": "ctx.source.ip == null"
            }
        },
        {
            "convert": {
                "field": "source.ip",
                "target_field": "source.address",
                "type": "string",
                "ignore_missing": true
            }
        },
        {
            "remove": {
                "field": "message"
            }
        },
        {
            "rename": {
                "field": "@timestamp",
                "target_field": "event.created"
            }
        },
        {
            "date": {
                "field": "nginx.access.time",
                "target_field": "@timestamp",
                "formats": [
                    "dd/MMM/yyyy:H:m:s Z"
                ],
                "on_failure": [{"append": {"field": "error.message", "value": "{{ _ingest.on_failure_message }}"}}]
            }
        },
        {
            "remove": {
                "field": "nginx.access.time"
            }
        },
        {
            "user_agent": {
                "field": "user_agent.original"
            }
        },
        {
            "geoip": {
                "field": "source.ip",
                "target_field": "source.geo",
                "ignore_missing": true
            }
        },
        {
            "geoip": {
                "database_file": "GeoLite2-ASN.mmdb",
                "field": "source.ip",
                "target_field": "source.as",
                "properties": [
                    "asn",
                    "organization_name"
                ],
                "ignore_missing": true
            }
        },
        {
            "rename": {
                "field": "source.as.asn",
                "target_field": "source.as.number",
                "ignore_missing": true
            }
        },
        {
            "rename": {
                "field": "source.as.organization_name",
                "target_field": "source.as.organization.name",
                "ignore_missing": true
            }
        }
    ],
    "on_failure": [
        {
            "set": {
                "field": "error.message",
                "value": "{{ _ingest.on_failure_message }}"
            }
        }
    ]
}
```

## 6. error/manifest.yml

```yml
module_version: "1.0"
var:
  - name: paths
    default:
      - /var/log/nginx/error.log*
    os.darwin:
      - /usr/local/var/log/nginx/error.log*
    os.windows:
      - c:/programdata/nginx/logs/error.log*
ingest_pipeline: ingest/pipeline.json
input: config/nginx-error.yml
```

## 7. error/config/nginx-error.yml

```yml 
type: log
paths:
{{ range $i, $path := .paths }}
 - {{$path}}
{{ end }}
exclude_files: [".gz$"]
processors:
- add_locale: ~
```

## 8.  error/ingest/pipeline.json   

```json
{
  "description": "Pipeline for parsing the Nginx error logs",
  "processors": [{
    "grok": {
      "field": "message",
      "patterns": [
        "%{DATA:nginx.error.time} \\[%{DATA:log.level}\\] %{NUMBER:process.pid:long}#%{NUMBER:process.thread.id:long}: (\\*%{NUMBER:nginx.error.connection_id:long} )?%{GREEDYDATA:message}"
      ],
      "ignore_missing": true
    }
  }, {
    "rename": {
      "field": "@timestamp",
      "target_field": "event.created"
    }
  }, {
    "date": {
      "field": "nginx.error.time",
      "target_field": "@timestamp",
      "formats": ["yyyy/MM/dd H:m:s"],
      "ignore_failure": true
    }
  }, {
    "date": {
      "if": "ctx.event.timezone != null",
      "field": "nginx.error.time",
      "target_field": "@timestamp",
      "formats": ["yyyy/MM/dd H:m:s"],
      "timezone": "{{ event.timezone }}",
      "on_failure": [{"append": {"field": "error.message", "value": "{{ _ingest.on_failure_message }}"}}]
    }
  }, {
    "remove": {
      "field": "nginx.error.time"
    }
  }],
  "on_failure" : [{
    "set" : {
      "field" : "error.message",
      "value" : "{{ _ingest.on_failure_message }}"
    }
  }]
}
```

# 五、示例：

## 1. Filebeat Nginx模块配置采集API网关Kong日志

**Kong日志数据采集处理流程**：`kong节点 + filbeat ----> Kubernetes上的Logstash ----> Kubernetes上的Elasticsearch`

Kong使用了Nginx作为基础组件，它的日志也主要是Nginx格式的日志，分为两种：访问日志和错误日志。它的Nginx是安装了Lua模块的，而Lua模块的错误日志和Nginx的错误日志混合在一起。Lua的错误日志格式有的是多行。这就造成整个Nginx错误日志中既有单行错误日志，又有多行错误日志。

直接使用Filebeat的Nginx模块采集日志文件。对于标准格式的Kong访问日志是没有问题的，关键点是错误日志，要修改filebeat的Nginx模块对错误日志文件进行多行采集，设置过滤关键词，将关键词之间的多行合并为一个采集事件。

Kong的日志输出目录：`/usr/local/kong/logs`。目录下有两种格式的日志文件

1. Nginx标准日志格式的访问日志文件：`/usr/local/kong/logs/admin_access.log` `/usr/local/kong/logs/access.log`

    ```bash
    172.17.18.169 - - [21/Oct/2019:11:47:42 +0800] "GET /oalogin.php HTTP/1.1" 494 46 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.120 Safari/537.36"
    ```

2. 带有Lua模块Nginx的错误日志文件 ：`/usr/local/kong/logs/error.log`
    ```bash
    2019/10/21 10:58:56 [warn] 14716#0: *17345670 [lua] reports.lua:70: log(): [reports] unknown request scheme: http while logging request, client: 172.17.18.169, server: kong, request: "GET / HTTP/1.1", host: "172.17.18.169"
    2019/10/21 10:59:05 [warn] 14717#0: *17346563 [lua] reports.lua:70: log(): [reports] unknown request scheme: http while logging request, client: 172.17.18.169, server: kong, request: "GET /routes HTTP/1.1", host: "172.17.18.169"
    2019/10/21 11:00:09 [error] 14716#0: *17348732 lua coroutine: runtime error: don't know how to respond to POST
    stack traceback:
    coroutine 0:
            [C]: ?
    coroutine 1:
            [C]: in function 'resume'
            /usr/local/share/lua/5.1/lapis/application.lua:397: in function 'handler'
            /usr/local/share/lua/5.1/lapis/application.lua:130: in function 'resolve'
            /usr/local/share/lua/5.1/lapis/application.lua:167: in function </usr/local/share/lua/5.1/lapis/application.lua:165>
            [C]: in function 'xpcall'
            /usr/local/share/lua/5.1/lapis/application.lua:173: in function 'dispatch'
            /usr/local/share/lua/5.1/lapis/nginx.lua:230: in function 'serve'
            /usr/local/share/lua/5.1/kong/init.lua:1113: in function 'admin_content'
            content_by_lua(nginx-kong.conf:190):2: in main chunk, client: 172.17.18.169, server: kong_admin, request: "POST /routes/smsp-route HTTP/1.0", host: "local.api.kong.curouser.com:80"
    2019/10/21 11:06:38 [warn] 14713#0: *17362982 [lua] reports.lua:70: log(): [reports] unknown request scheme: http while logging request, client: 172.17.18.169, server: kong, request: "GET /upstream HTTP/1.1", host: "172.17.18.169"
    ```

**修改Nginx模块采集错误日志文件的方式**

Filebeat的安装，Nignx模块启用，模块参数配置等操作步骤省略。这里只写针对Nginx错误日志配置进行的修改。

编辑`/usr/share/filebeat/module/nginx/error/config/nginx-error.yml`

```yml
# ===========================修改前的======================================
type: log
paths:
{{ range $i, $path := .paths }}
 - {{$path}}
{{ end }}
exclude_files: [".gz$"]

processors:
- add_locale: ~
# ===========================修改后的======================================
type: log
paths:
{{ range $i, $path := .paths }}
 - {{$path}}
{{ end }}
exclude_files: [".gz$"]
multiline.pattern: '^[0-9]{4}/[0-9]{2}/[0-9]{2} [0-9]{2}:[0-9]{2}:[0-9]{2}'
multiline.negate: true
multiline.match: after  
```

Logstash针对FIlebeat发送过来的日志事件进行分割处理的Pipelines

```json
#=======================接收Filebeat发送过来的日志事件====================
input {
  beats {
    id => "logstash_kong_beats"
    port => 5044
  }
}
#=======================过滤、拆分、转换日志事件==============================
filter {
  if [fileset][name] == "access" {
    grok {
       match => { "message" => ["%{IPORHOST:[nginx][access][remote_ip]} - %{DATA:[nginx][access][user_name]} \[%{HTTPDATE:[nginx][access][time]}\] \"%{WORD:[nginx][access][method]} %{DATA:[nginx][access][url]} HTTP/%{NUMBER:[nginx][access][http_version]}\" %{NUMBER:[nginx][access][response_code]} %{NUMBER:[nginx][access][body_sent][bytes]} \"%{DATA:[nginx][access][referrer]}\" \"%{DATA:[nginx][access][agent]}\""] }
       remove_field => "message"
     }
     mutate {
      add_field => { "read_timestamp" => "%{@timestamp}" }
     }
     date {
             match => [ "[nginx][access][time]", "dd/MMM/YYYY:H:m:s Z" ]
             remove_field => "[nginx][access][time]"
     }
     useragent {
             source => "[nginx][access][agent]"
             target => "[nginx][access][user_agent]"
             remove_field => "[nginx][access][agent]"
     }
     geoip {
             source => "[nginx][access][remote_ip]"
             target => "[nginx][access][geoip]"
           }
     }
   else if [fileset][name] == "error" {
     grok {
       match => { "message" => ["%{DATA:[nginx][error][time]} \[%{DATA:[nginx][error][level]}\] %{NUMBER:[nginx][error][pid]}#%{NUMBER:[nginx][error][tid]}: (\*%{NUMBER:[nginx][error][connection_id]} )?%{GREEDYDATA:[nginx][error][message]}"] }
       remove_field => "message"
     }
       mutate {
         rename => { "@timestamp" => "read_timestamp" }
       }
       date {
         match => [ "[nginx][error][time]", "YYYY/MM/dd H:m:s" ]
         remove_field => "[nginx][error][time]"
       }
     }
}
#=======================根据日志事件类型的不同输出到不同elasticsearch索引中====================
output {
  if [fileset][name] == "access" {
    elasticsearch {
      id => "logstash_kong_access_log"
      hosts => ["elasticsearch.elk.svc"]
      index => "kong-accesslog-%{+YYYY.MM.dd}"
      document_type => "_doc"
      http_compression => true
      template_name => "logstash-logger"
      user => "logstash-user"
      password => "logstash-password"
    }
  }else if [fileset][name] == "error"{
    elasticsearch {
      id => "logstash_kong_error_log"
      hosts => ["elasticsearch.elk.svc"]
      index => "kong-errorlog-%{+YYYY.MM.dd}"
      document_type => "_doc"
      http_compression => true
      template_name => "logstash-curiouser"
      user => "logstash-user"
      password => "logstash-password"
     }
  }
}
```







