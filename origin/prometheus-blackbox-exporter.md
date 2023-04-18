# Blackbox exporter

# 一、简介

黑盒监控即以用户的身份测试服务的外部可见性，常见的黑盒监控包括`HTTP 探针`、`TCP 探针` 等用于检测站点或者服务的可访问性，以及访问效率等。

黑盒监控相较于白盒监控最大的不同在于黑盒监控是以故障为导向当故障发生时，黑盒监控能快速发现故障，而白盒监控则侧重于主动发现或者预测潜在的问题。一个完善的监控目标是要能够从白盒的角度发现潜在问题，能够在黑盒的角度快速发现已经发生的问题。

[Blackbox Exporter](https://github.com/prometheus/blackbox_exporter) 是 Prometheus 社区提供的官方黑盒监控解决方案，其允许用户通过：`HTTP`、`HTTPS`、`DNS`、`TCP` 以及 `ICMP` 的方式对网络进行探测

GitHub：https://github.com/prometheus/blackbox_exporter



# 二、安装

## 1、二进制

```bash
blackbox_exporter_version=0.18.0 && \
wget https://github.com/prometheus/blackbox_exporter/releases/download/v$blackbox_exporter_version/blackbox_exporter-$blackbox_exporter_version.linux-amd64.tar.gz && \
tar -zxvf blackbox_exporter-$blackbox_exporter_version.linux-amd64.tar.gz -C /opt  && \
rm -f blackbox_exporter-$blackbox_exporter_version.linux-amd64.tar.gz  && \
ln -s /opt/blackbox_exporter-0.18.0.linux-amd64 /opt/blackbox_exporter && \
echo -e "export BLACKBOX_EXPORTER_HOEM=/opt/blackbox_exporter\nexport PATH=\$PATH:\$BLACKBOX_EXPORTER_HOEM" >> /etc/profile && \
source /etc/profile && \
blackbox_exporter --help
```

**命令参数**

```bash
usage: blackbox_exporter [<flags>]

Flags:
  -h, --help                     Show context-sensitive help (also try --help-long and --help-man).
   --config.file="blackbox.yml"  Blackbox exporter configuration file.
   --web.listen-address=":9115"  The address to listen on for HTTP requests.
   --timeout-offset=0.5          Offset to subtract from timeout in seconds.
   --config.check                If true validate the config file and then exit.
   --history.limit=100           The maximum amount of items to keep in the history.
   --web.external-url=<url>      The URL under which Blackbox exporter is externally reachable (for example, if Blackbox exporter is served via a reverse proxy). Used for generating relative and absolute links back to Blackbox exporter itself. If the URL has a path portion, it will be used to prefix all HTTP endpoints served by Blackbox exporter. If omitted, relevant URL components will be derived automatically.
   --web.route-prefix=<path>     Prefix for the internal routes of web endpoints. Defaults to path of --web.external-url.
   --log.level=info              Only log messages with the given severity or above. One of: [debug, info, warn, error]
   --log.format=logfmt           Output format of log messages. One of: [logfmt, json]
   --version                     Show application version.
```

**启动**

```bash
nohup blackbox_exporter  --config.file=配置文件路径 --其他参数 > /var/log/blackbox_exporter.log 2>&1 &
```

## 2、Docker

```bash
docker run --rm -d \
  -p 9115:9115 \
  --name blackbox_exporter \
  -v `配置文件路径`:/config \
  prom/blackbox-exporter:master --config.file=/config/blackbox.yml
```

## 3、Kubernetes

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blackbox
  namespace: monitoring
  labels:
    app: blackbox-exporter
spec:
  replicas: 2
  revisionHistoryLimit: 3
  selector:
    matchLabels:
      app: blackbox-exporter
  strategy:
    rollingUpdate:
      maxSurge: 30%
      maxUnavailable: 30%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: blackbox-exporter
    spec:
      containers:
      - image: prom/blackbox-exporter:master
        name: blackbox-exporter
        args:
        - --config.file=/etc/blackbox_exporter/blackbox.yml # ConfigMap 中的配置文件
        - --log.level=info  # 日志级别，可以把级别调到 error
        ports:
        - containerPort: 9115
          name: http
        volumeMounts:
        - name: config
          mountPath: /etc/blackbox_exporter
      volumes:
      - name: config
        configMap:
          name: blackbox-config
      nodeSelector:
        role: monitoring
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: blackbox-config
  namespace: monitoring
data:
  blackbox.yml: |-
    modules:
      http_2xx:
        prober: http
        timeout: 10s
        http: 
          valid_status_codes: [0,200] 
      baidu-header:
        prober: http
        timeout: 10s
        http:
          valid_status_codes: [0,200] 
          method: GET
          headers:
            Access-Token: ***
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: blackbox-exporter
  name: blackbox-exporter
  namespace: monitoring
spec:
  ports:
  - name: http
    port: 9115
    targetPort: http
  selector:
    app: blackbox-exporter
```



# 三、配置文件

配置文件详解：https://github.com/prometheus/blackbox_exporter/blob/master/CONFIGURATION.md

示例配置文件：https://github.com/prometheus/blackbox_exporter/blob/master/example.yml

## 1、配置文件结构

```bash

```



## 2、配置值类型

- `<boolean>`: 布尔值，可选 `true` | `false`
- `<int>`: 整型值
- `<duration>`: 与正则表达式[0-9] +（ms | [smhdwy]）匹配的持续时间
- `<filename>`: 当前工作目录中的有效路径
- `<string>`: 字符串
- `<secret>`: 包含密码的常规字符串，例如密码
- `<regex>`: 正则表达式

## 3、HTTP探针配置

```bash
  # 此探针接受的响应状态代码。默认为2xx。
  [ valid_status_codes: <int>, ... | default = 2xx ]

  # 此探针接受的HTTP版本。
  [ valid_http_versions: <string>, ... ]

  # The HTTP method the probe will use.
  [ method: <string> | default = "GET" ]

  # 为探针设置的HTTP标头。
  headers:
    [ <string>: <string> ... ]

  # 用于解压缩响应的压缩算法（gzip，br，deflate，identity）。如果指定了“ Accept-Encoding”标头，则必须使压缩算法，表示使用此选项是可接受的。例如可以使用`compression：gzip`和`Accept-Encoding：br，gzip`或`Accept-Encoding：br; q = 1.0，gzip; q = 0.9`。 gzip是
  # 可接受的质量低于br的质量不会使配置无效，因为您可能会测试服务器即使请求也不会返回br编码的内容。在另一方面，“ compression：gzip”和“ Accept-Encoding：br，identity”不是有效的配置，因为您要求不返回gzip，并尝试解压缩服务器返回的任何内容都可能会失败。[压缩：<string> |默认=“”]
  [ compression: <string> | default = "" ]

  # 探针是否将遵循任何重定向
  [ no_follow_redirects: <boolean> | default = false ]

  # 如果存在SSL，则探测失败
  [ fail_if_ssl: <boolean> | default = false ]

  # 如果不存在SSL，则探测失败。
  [ fail_if_not_ssl: <boolean> | default = false ]

  # 如果响应内容与正则表达式匹配，则探测失败
  fail_if_body_matches_regexp:
    [ - <regex>, ... ]

  # 如果响应内容与正则表达式不匹配，则探测失败
  fail_if_body_not_matches_regexp:
    [ - <regex>, ... ]

  # 如果响应头与正则表达式匹配，则探测失败。对于具有多个值的标头，如果至少一个匹配，则失败。
  fail_if_header_matches:
    [ - <http_header_match_spec>, ... ]

  # 如果响应头与正则表达式不匹配，则探测失败。对于具有多个值的标头，如果一个也不匹配，则失败。
  fail_if_header_not_matches:
    [ - <http_header_match_spec>, ... ]

  # 为此探针配置TLS协议
  tls_config:
    [ <tls_config> ]

  # 为此探针配置HTTP基本身份验证凭据。
  basic_auth:
    [ username: <string> ]
    [ password: <secret> ]
    [ password_file: <filename> ]

  # 为此探针配置访问目标的Bearer token 
  [ bearer_token: <secret> ]

  # 为此探针配置访问目标的Bearer token文件
  [ bearer_token_file: <filename> ]

  # 用于连接到目标的HTTP代理服务器。
  [ proxy_url: <string> ]

  # 为此探针配置IP协议（ip4，ip6）
  [ preferred_ip_protocol: <string> | default = "ip6" ]
  [ ip_protocol_fallback: <boolean> | default = true ]

  # 为此探针配置访问目标的HTTP请求主体。
  body: [ <string> ]
```

匹配Header的正则表达式配置

```yaml
header: <string>,
regexp: <regex>,
[ allow_missing: <boolean> | default = false ]
```

## 4、TCP探针配置

```bash
# The IP protocol of the TCP probe (ip4, ip6).
[ preferred_ip_protocol: <string> | default = "ip6" ]
[ ip_protocol_fallback: <boolean | default = true> ]

# The source IP address.
[ source_ip_address: <string> ]

# The query sent in the TCP probe and the expected associated response.
# starttls upgrades TCP connection to TLS.
query_response:
  [ - [ [ expect: <string> ],
        [ send: <string> ],
        [ starttls: <boolean | default = false> ]
      ], ...
  ]

# Whether or not TLS is used when the connection is initiated.
[ tls: <boolean | default = false> ]

# Configuration for TLS protocol of TCP probe.
tls_config:
  [ <tls_config> ]
```

## 5、DNS探针配置

```bash
# The IP protocol of the DNS probe (ip4, ip6).
[ preferred_ip_protocol: <string> | default = "ip6" ]
[ ip_protocol_fallback: <boolean | default = true> ]

# The source IP address.
[ source_ip_address: <string> ]

[ transport_protocol: <string> | default = "udp" ] # udp, tcp

# Whether to use DNS over TLS. This only works with TCP.
[ dns_over_tls: <boolean | default = false> ]

# Configuration for TLS protocol of DNS over TLS probe.
tls_config:
  [ <tls_config> ]

query_name: <string>

[ query_type: <string> | default = "ANY" ]
[ query_class: <string> | default = "IN" ]

# List of valid response codes.
valid_rcodes:
  [ - <string> ... | default = "NOERROR" ]

validate_answer_rrs:

  fail_if_matches_regexp:
    [ - <regex>, ... ]

  fail_if_all_match_regexp:
    [ - <regex>, ... ]

  fail_if_not_matches_regexp:
    [ - <regex>, ... ]

  fail_if_none_matches_regexp:
    [ - <regex>, ... ]

validate_authority_rrs:

  fail_if_matches_regexp:
    [ - <regex>, ... ]

  fail_if_all_match_regexp:
    [ - <regex>, ... ]

  fail_if_not_matches_regexp:
    [ - <regex>, ... ]

  fail_if_none_matches_regexp:
    [ - <regex>, ... ]

validate_additional_rrs:

  fail_if_matches_regexp:
    [ - <regex>, ... ]

  fail_if_all_match_regexp:
    [ - <regex>, ... ]

  fail_if_not_matches_regexp:
    [ - <regex>, ... ]

  fail_if_none_matches_regexp:
    [ - <regex>, ... ]
```

## 6、ICMP探针配置

```bash
# The IP protocol of the ICMP probe (ip4, ip6).
[ preferred_ip_protocol: <string> | default = "ip6" ]
[ ip_protocol_fallback: <boolean | default = true> ]

# The source IP address.
[ source_ip_address: <string> ]

# Set the DF-bit in the IP-header. Only works with ip4, on *nix systems and
# requires raw sockets (i.e. root or CAP_NET_RAW on Linux).
[ dont_fragment: <boolean> | default = false ]

# The size of the payload.
[ payload_size: <int> ]
```

## 7、TLS相关配置

```bash
# Disable target certificate validation.
[ insecure_skip_verify: <boolean> | default = false ]

# The CA cert to use for the targets.
[ ca_file: <filename> ]

# The client cert file for the targets.
[ cert_file: <filename> ]

# The client key file for the targets.
[ key_file: <filename> ]

# Used to verify the hostname for the targets.
[ server_name: <string> ]
```



# 四、其他

## 1、

# 参考：

1. https://www.qikqiak.com/post/blackbox-exporter-on-prometheus/