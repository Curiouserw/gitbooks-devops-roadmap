# Nginx非常规配置

# 一、设置访问日志JSON格式化

## 1、配置

```nginx
# ....全局配置省略..... ;

http {
		# ....HTTP模块其他配置省略..... ;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    # 设置日志格式
    log_format json_log  [escape=default|json|none] '{ "@timestamp": "$time_iso8601", '
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
                           '"up_resp_time": $upstream_response_time,'
                           '"request_time": $request_time,'
                           '"server_name": "$server_name"'
                        ' }';
    server {
        # .....虚拟主机其他配置省略.....
        set $app test ; # 设置变量app的值为“test”
        access_log  logs/host.access.log  json_log; # 设置访问日志按照“json_log”的格式进行输出
    }
}
```

## 2、escape参数

 `[escape=default|json|none]` 

- **default**：会把`“`或`\`转换为`\xXX`
- **json**：会把`“`转换为`\“`，会把`\`转换为`\\“`

参考：http://nginx.org/en/docs/http/ngx_http_log_module.html#log_format

## 3、日志格式可用变量

| 变量 | 含义 |
| ---- | ---- |
| $remote_addr | 记录客户端IP地址 |
| $remote_user | 记录客户端用户名 |
| $time_local | 记录通用的本地时间 |
| $time_iso8601 | 记录ISO8601标准格式下的本地时间 |
| $request | 记录请求的方法以及请求的http协议 |
| $status | 记录请求状态码(用于定位错误信息) |
| $body_bytes_sent | 发送给客户端的资源字节数，不包括响应头的大小 |
| $bytes_sent | 发送给客户端的总字节数 |
| $msec | 日志写入时间。单位为秒，精度是毫秒。 |
| $http_referer | 记录从哪个页面链接访问过来的 |
| $http_user_agent | 记录客户端浏览器相关信息 |
| $http_x_forwarded_for |记录客户端IP地址|
| $request_length | 请求的长度（包括请求行，请求头和请求正文）。 |
| $request_time | 请求花费的时间，单位为秒，精度毫秒 |

## 4、输出的JSON格式日志

```json
{
    "@timestamp": "2020-03-09T17:54:49+08:00",
    "app": "test",
    "remote_addr": "127.0.0.1",
    "referer": "-",
    "request": "GET /Empty-2C4G80G.ovf HTTP/1.1",
    "status": 200,
    "bytes": 7532,
    "agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.132 Safari/537.36",
    "x_forwarded": "33.100.12.1, 100.7.88.12",
    "up_addr": "-",
    "up_host": "-",
    "up_resp_time": 0.096,
    "request_time": 0.095,
    "server_name": ""
}
```

# 二、按天保留日志文件

```yaml
server{
    ...
		if ($time_iso8601 ~ '(\d{4}-\d{2}-\d{2})') {
       set $tttt $1;
    }
    access_log  logs/nginx-access-$tttt.log  main;
    ...
}
```

# 三、配置HTTP基础认证

Nginx 使用  `ngx_http_auth_basic_module`  模块支持 **HTTP基本身份验证** 功能 。

## 1、安装httpd-tools

```bash
yum install -y httpd-tools
apt install -y apache2-utils  
```

## 2、创建授权用户和密码

```bash
htpasswd -c -d /etc/nginx/basic-auth-pass-file test_user
```

htpasswd 其他操作参考：[htpasswd操作](linux-htpasswd.md)

## 3、配置nginx

```bash
server {
     # ...省略
    auth_basic   "登录认证";  
    auth_basic_user_file basic-auth-pass-file;

    # ...省略
    
    # 或者只设置某些URL进行登录认证
    # location /api {
    #    auth_basic "登录认证";
    #    auth_basic_user_file basic-auth-pass-file;
    #}
}
```

## 4、使用

```bash
# 浏览器中使用
直接在浏览器中输入地址, 会弹出用户密码输入框, 输入即可访问
# wget
wget --http-user=test_user --http-passwd=*** http://****
# curl
curl -u test_user:**** -O http://****
```

# 四、自定义日志格式引用自定义Header

自定日志格式时如果想获取请求中的自定义Header，nginx引用方式为`$http_headername`，如果header命名中包含`-`，需要转换成`_`。例如下面示例中引用开发自定义的`x-zz-app-info` 

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
                           '"up_resp_time": $upstream_response_time,'
                           '"request_time": $request_time,'
                           '"server_name": "$server_name",'
                           '"x-zz-app-info": "$http_x_zz_app_info"'
                        ' }';
  server {
      listen       80;
      server_name  localhost;
      set $app test;
      access_log  /var/log/nginx/host.access.log  json_log;

      location / {
          root   /usr/share/nginx/html;
          index  index.html index.htm;
      }
  }
```



# 五、禁用iframe嵌入使用

在Nginx中配置添加X-Frame-Options 响应头

- 在http配置里设置X-Frame-Options

  ```bash
  pid        /var/run/nginx.pid;
  http{
     include       /etc/nginx/mime.types;
  	 default_type  application/octet-stream;
     ....
     add_header X-Frame-Options SAMEORIGIN;
     ....
  }
  ```

- 在server配置里设置X-Frame-Options

  ```bash
  server{
  	listen 443;
  	....
  	add_header X-Frame-Options SAMEORIGIN;
  	...
  }
  ```

- **X-Frame-Options 响应头**：

  X-Frame-Options HTTP 响应头是用来给浏览器指示允许一个页面可否在 `<frame>,` `<iframe>` 或者 `<object>` 中展现的标记。网站可以使用此功能，来确保自己网站的内容没有被嵌到别人的网站中去，也从而避免了点击劫持 (clickjacking) 的攻击。

- **X-Frame-Options 有三个值:**

  - **DENY**：表示该页面不允许在 frame 中展示，即便是在相同域名的页面中嵌套也不允许。

  - **SAMEORIGIN**：表示该页面可以在相同域名页面的 frame 中展示。

  - **ALLOW-FROM uri**：表示该页面可以在指定来源的 frame 中展示。

参考：https://cloud.tencent.com/developer/article/1182220

# 六、镜像流量

## 1、简介

- nginx 1.13.4及后续版本内置了`ngx_http_mirror_module`模块，提供流量镜像复制的功能。

- 支持流量放大，做法为：配置多份相同镜像。

- 相比`tcp-copy`的优势：无需录制流量，实时可用；配置相当简单。

- 源站请求，直接原路返回；正常配置下，mirror请求不影响源站请求及响应，源站将流量复制到mirror站后，两者不再有任何交集。

- 使用`ngx_http_mirror_module`模块实现流量拷贝。例如将生产环境的流量拷贝复制到预上线环境或测试环境

  - 可以验证功能是否正常，以及服务的性能；

  - 用真实有效的流量请求去验证，又不用造数据，不影响线上正常访问；

  - 相比于灰度发布，镜像流量不会影响真实流量；

  - 可以用来排查线上问题；

  - 重构，假如服务做了重构，这也是一种测试方式

## 2、镜像请求复制配置

```nginx
server {
  listen 80;
  error_log  logs/nginx-error.log;
  access_log logs/nginx-access.log  main;
  location / {
    root  /var/www/html;
    index index.html index.htm;
    autoindex on;

    # 开起复制请求体
    mirror_request_body on;
    # 指定镜像uri为/mirror
    mirror /mirror;
  }
  location /mirror {
      # internal标志该location只为内部的重定向服务， 外部的调用请求则返回404
      internal;
    
     # 指定是否镜像请求body部分，此选项与proxy_request_buffering、fastcgi_request_buffering、scgi_request_buffering和 uwsgi_request_buffering冲突，一旦开启mirror_request_body为on，则请求自动缓存;
      proxy_pass_request_body on;
    
      proxy_set_header Host $host;
    
      # 设置镜像流量的头部
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    
      # 原始uri不会镜像，可以通过$request_uri变量取得原始请求的uri
      proxy_set_header X-Original-URI $request_uri;
    
      proxy_pass http://127.0.0.1:8080$request_uri;
  }
}

server {
  listen 8080;
  error_log logs/nginx-error-mirror.log;
  access_log logs/nginx-access-mirror.log  main;

  location / {
    root  /var/www/html;
    index index.html index.htm;
    autoindex on;   
  }
}
```

## 3、镜像请求放大复制配置

```bash
server{
	...
	mirror /mirror;
  # 多加一份mirror，流量放大一倍
  mirror /mirror;
	...
}
```

参考：

- http://nginx.org/en/docs/http/ngx_http_mirror_module.html
- https://cloud.tencent.com/developer/article/1751894
- https://segmentfault.com/a/1190000039870391
- https://www.it610.com/article/1306183699749965824.htm
- https://blog.csdn.net/zfw_666666/article/details/124660479

# 七、限流

Nginx 提供两种限流方式，一是控制请求速率，二是控制并发连接数。

## 1、限制连接频率

### ①简介

nginx的`ngx_http_limit_conn_module` 模块提供了限制连接数的能力。主要是利用`limit_conn_zone`和`limit_conn`两个指令来限制单个IP的请求数。并非所有的连接都被计数，只有在服务器处理了请求并且已经读取了整个请求头时，连接才被计数。

### ②指令配置

- **limit_conn_zone：**`limit_conn_zone 【key】 【zone=name:size】`

  - **key**：定义限流对象，key可以为文本、变量或者二者的组合来指定要根据什么对象进行标识来达到限流。

    通常key为`$binary_remote_addr`，`$binary_remote_addr`是`$remote_addr（客户端IP）`的二进制格式，固定占用4个字节。即限定每个客户端IP的请求处理速率。binary_ 开头的目的是压缩内存占用量

    `$ remote_addr`变量的大小可以从7到15个字节不等。存储的状态在32位平台上占用32或64字节的内存，在64位平台上总是占用64字节。对于IPv4地址，`$ binary_remote_addr`变量的大小始终为4个字节，对于IPv6地址则为16个字节。存储状态在32位平台上始终占用32或64个字节，在64位平台上占用64个字节。一个兆字节的区域可以保持大约32000个32字节的状态或大约16000个64字节的状态。如果区域存储耗尽，服务器会将错误返回给所有其他请求。

  - **zone**：指定zone的名称和大小，用来存储访问的频次信息。

    name:10m表示一个大小为10M，名字为name的内存区域

    1M能存储16000 IP地址的访问信息，10M可以存储16W IP地址访问信息


- **limit_conn：**`limit_conn 【zone】 【number】`

  - **zone：**指定`limit_conn_zone`定义的zone名称，与上面limit_conn_zone 里的name对应。

  - **number：**指定最多持有的连接个数


- **limit_conn_status：**`limit_conn_status 【code】`
  - **code：** 指定超过limit的请求的HTTP返回状态码，默认值为503

- **limit_conn_log_level指令格式：** `limit_req_log_level info | notice | warn | error;`

```nginx
http{
  ...
  # 表示限制根据用户的IP地址来显示，设置存储地址为的内存大小10M
  limit_conn_zone $binary_remote_addr zone=perip:10m ;
  limit_conn_zone $server_name zone=perserver:10m ;
  ...
}
server{
  ...
  location / {
    # 表示同一个IP地址的客户端最多最多持有20个连接。
  	limit_conn perip 20;
    # 表示虚拟主机(server) 能同时处理100个并发连接总数。
    limit_conn perserver 100;
  }
  ...
}
```



## 2、限制请求频率

### ①简介

- nginx的`limit_req_module`模块可以控制速率限流，在一定的时间内只处理特定数量的请求。

- nginx的`limit_req_module`模块针对每个key限定请求处理的速率（rate），比较典型的是，按照客户端IP，限定此IP下请求的处理速率。
- nginx是采用漏桶算法实现控制速率限流。漏桶(Leaky Bucket)算法思路很简单,水(请求)先进入到漏桶里,漏桶以一定的速度出水(接口有响应速率),当水流入速度过大会直接溢出(访问频率超过接口响应速率),然后就拒绝请求,可以看出漏桶算法能强行限制数据的传输速率.

### ②指令配置

- **limit_req_zone：**`limit_req_zone【key】 【zone=name:size】【rate=rate】`

  - **key**：定义限流对象，key可以为文本、变量或者二者的组合来指定要根据什么对象进行标识来达到限流。

    通常key为`$binary_remote_addr`，`$binary_remote_addr`是`$remote_addr（客户端IP）`的二进制格式，固定占用4个字节。即限定每个客户端IP的请求处理速率。binary_ 开头的目的是压缩内存占用量。

    `$ remote_addr`变量的大小可以从7到15个字节不等。存储的状态在32位平台上占用32或64字节的内存，在64位平台上总是占用64字节。对于IPv4地址，`$ binary_remote_addr`变量的大小始终为4个字节，对于IPv6地址则为16个字节。存储状态在32位平台上始终占用32或64个字节，在64位平台上占用64个字节。一个兆字节的区域可以保持大约32000个32字节的状态或大约16000个64字节的状态。如果区域存储耗尽，服务器会将错误返回给所有其他请求。

  - **zone**：指定zone的名称和大小，用来存储访问的频次信息。

    name:10m表示一个大小为10M，名字为name的内存区域

    1M能存储16000 IP地址的访问信息，10M可以存储16W IP地址访问信息

  - **rate**：用于设置最大访问速率，表示允许相同标识的客户端的访问频次，单位是每秒几次，r/s。

    `rate=2r/s` 表示每秒最多处理2个请求。

    Nginx 实际上以毫秒为粒度来跟踪请求信息，因此 2r/s 实际上是限制：每100毫秒处理两个请求。这意味着，自上一个请求处理完后，若后续100毫秒内又有请求到达，将拒绝处理该请求。


- **limit_req：** `limit_req【zone=name】【burst=number】【nodelay】`

  - **zone=name：** 指定`limit_req_zone`定义的zone名称，与上面limit_req_zone 里的name对应。

  - **burst=number：**指定burst的容量大小，默认为0。

    burst译为突发、爆发，表示在超过设定的处理速率后能额外处理的请求数。**即允许指定个数的请求进行等待处理**，对于后续**过量**的请求，将会被拒绝并返回503错误。如果burst=20 ，若同时有21个请求到达，Nginx 会处理第一个请求，剩余20个请求将放入队列，然后每隔100ms从队列中获取一个请求进行处理。若请求数大于21，将拒绝处理多余的请求，直接返回503.

  - **nodelay：**如果设置，会在瞬时提供处理(burst + rate)个请求的能力，请求超过（burst + rate）的时候就会直接返回503，永远不存在请求需要等待的情况。如果没有设置，则所有请求会依次等待排队。

    nodelay参数允许请求在排队的时候就立即被处理，也就是说只要请求能够进入burst队列，就会立即被后台worker处理，请注意，这意味着burst设置了nodelay时，系统瞬间的QPS可能会超过rate设置的阈值。nodelay参数要跟burst一起使用才有作用。


- **limit_req_log_level：** `limit_req_log_level info | notice | warn | error;`
- **limit_req_status：**`limit_req_status code;`

```nginx
http{
  ...
  limit_req_zone $binary_remote_addr zone=myReqLimit:10m rate=5r/s;
  ...
}
server{
  ...
  location / {
  	limit_req zone=myReqLimit;  
    # 如果有时正常流量突然增大，超出的请求将被拒绝，无法处理突发流量，可以结合 burst 参数使用来解决该问题
    limit_req zone=myReqLimit burst=4 nodelay;
    # burst=4 nodelay 表示平均每秒允许不超过2个请求，突发不超过4个请求，并且处理突发4个请求的时候，没有延迟，等到完成之后，按照正常的速率处理。
  }
  ...
}
```



## 参考

- https://nginx.org/en/docs/http/ngx_http_limit_req_module.html
- https://nginx.org/en/docs/http/ngx_http_limit_conn_module.html
- https://www.cnblogs.com/biglittleant/p/8979915.html
- https://www.cnblogs.com/daijiting/p/14909190.html
- https://blog.csdn.net/lt326030434/article/details/122989403
- https://segmentfault.com/a/1190000022530919



# 八、gzip压缩功能

Nginx支持对指定类型的文件进行压缩然后再传输给客户端，而且压缩还可以设置压缩比例，压缩后的文件大小将比源文件显著变小，这样有助于降低出口带宽的利用率

Nginx对文件的压缩功能是依赖于模块 `ngx_http_gzip_module`,默认是内置模块。

>  配置适用上下文：`http，server，location`

```nginx
# 启用或禁用gzip压缩，默认关闭。
gzip on | off;

# 压缩比由低到高从1到9，默认为1
gzip_comp_level level;

# 禁用IE6 gzip功能
gzip_disable "MSIE [1-6]\.";

# gzip压缩的最小文件，小于设置值的文件将不会压缩
gzip_min_length 1k;

# 启用压缩功能时，协议的最小版本，默认HTTP/1.1
gzip_http_version 1.0 | 1.1;

# 指定Nginx服务需要向服务器申请的缓存空间的个数和大小,平台不同,默认:32 4k或者16 8k;
gzip_buffers number size; 

# 指明仅对哪些类型的资源执行压缩操作;默认为gzip_types text/html，不用显示指定，否则出错。当gzip开启时，只对指定的“MIME”类型的响应使用gzip，但无论如何“text/html;”类型一定会压缩。
gzip_types mime-type ...;

# 默认值为“off”；如果nginx开启了“gzip”、“gunzip”、“gzip_static”，且在当前请求中应用，那么是否在响应的中添加"Vary:Accept-Encoding" header；这个header可以帮助我们（以及浏览器）判定请求是否使用gzip。
gzip_vary on | off;

# 预压缩，即直接从磁盘找到对应文件的gz后缀的式的压缩文件返回给用户，无需消耗服务器CPU
# 注意: 来自于ngx_http_gzip_static_module模块
gzip_static on | off;
```



# 九、大文件传输配置

| 配置项 | 含义 |
| --- | --- |
| **client_max_body_size** | 设置请求体允许的最大体积，默认1m。若超过所设定的大小，返回413错误。 |
| **client_header_timeout** |等待客户端发送一个请求头的超时时间，若超过所设定的大小，返回408错误。 |
| **client_body_timeout** |设置读取请求体的超时时间，若超过所设定的大小，返回413错误。 |
| **proxy_connect_timeout** |http请求无法立即被容器(tomcat, netty等)处理，被放在nginx的待处理池中等待被处理。此参数为等待的最长时间，默认为60秒，官方推荐最长不要超过75秒。 |
| **proxy_read_timeout** |http请求被容器(tomcat, netty等)处理后，nginx会等待处理结果，也就是容器返回的response。即为服务器响应时间，Nginx等待的最长时间。默认60秒。 |
| **proxy_send_timeout** |设置后端向Nginx返回响应时的超时时间，默认60秒。 |

参考：

- https://blog.csdn.net/weixin_38306434/article/details/86491905

- https://zhuanlan.zhihu.com/p/82548284

# 十、设置IP黑白名单

## 1、简介

- Nginx的**ngx_http_access_module**模块可以根据IP地址进行放行或限制。默认加载此模块。核心指令有`allow/deny`

- **allow/deny指令格式：**`allow|deny address|CIDR|all;`

  - **address:**

    > deny 1.2.3.4;       # 屏蔽单个ip访问
    > allow 1.2.3.4;       # 允许单个ip访问

  - **CIDR:**

    > deny 1.0.0.0/8;         # 屏蔽整个段即从1.0.0.1到1.255.255.254的IP段
    > deny 1.2.0.0/16;      #  屏蔽IP段即从1.2.0.1到1.2.255.254的IP段
    > deny 1.2.3.0/24;      #  屏蔽即从1.2.3.1到1.2.3.254的IP段

  - **all：**

    > deny all;       # 屏蔽所有ip访问
    > allow all;      # 允许所有ip访问

- `access/deny`配置规则生效顺序：依次检查规则，直到找到第一个匹配项。类似于iptables规则

- `access/deny`指令配置范围
  - 在http范围的黑白单中，访问该服务器的所有服务都要被黑名单过滤。
  - 在server范围黑白名单中，只有访问该http服务器的当前server服务时，才会被黑名单过滤。
  - 在location范围黑白名单中，针对当前转发才会被黑名单过滤。

## 2、配置

### ①白名单模式

```nginx
server{
  location / {
       allow 123.13.123.12;
       allow 23.53.32.1/100;
       deny  all;
  }
}
```

### ②黑名单模式

```nginx
server{
  location / {
      # 黑名单设置，deny后面接限制的IP，为什么不加allow all? 因为这个默认是开启的 
      deny 123.13.123.12;
  }
}
```

### ③读取黑白IP地址配置文件

- IP地址配置文件如果有变更，需要reload重新加载配置
- 黑白IP地址配置文件中的规则与指令配置的规则匹配顺序是先匹配配置文件中

```nginx
server{
  location / {
      # 默认位置路径为当前server指令配置文件路径下
      include whitelist.txt ;
      deny all ;
  }
}

# 文件/etc/nginx/conf.d/whitelist.txt内容如下：
# allow 192.168.1.123;
# allow 192.168.2.123;

server{
  location / {
      # 默认位置路径为当前server指令配置文件路径下
      include blockip.txt ;
      # 默认是allow all;
  }
}

# 文件/etc/nginx/conf.d/blockip.txt内容如下：
# deny 192.168.1.123;
# deny 192.168.2.123;
```

## 参考

- https://nginx.org/en/docs/http/ngx_http_access_module.html
- https://www.jb51.net/article/255998.htm

# 十一、屏蔽私密文件

```bash
location ~ (.git|.gitattributes|.gitignore|.svn) {
    deny all;
}
```

# 十二、跳转WWW

```bash
server {
    listen 80;
    # 配置正常的带www的域名
    server_name www.test.curiouser.com;
    root /var/www/html/test;
    location / {
        try_files $uri $uri/ /index.html =404;
    }
}
server {
    # 这个要放到下面，
    # 将不带www的 test.curiouser.com 永久性重定向到  https://www.test.curiouser.com
    server_name test.curiouser.com;
    rewrite ^(.*) https://www.test.curiouser.com$1 permanent;
}
```

# 十三、Nginx与PHP FastCGI

- Nginx本身不能处理PHP，它只是个web服务器，当接收到请求后，如果是php请求，则发给php解释器处理，并把结果返回给客户端

- **Nginx与php-fpm之间的请求处理流程：**

  - 请求发送到nginx服务

  - nginx收到请求，通过fast-cgi协议，将请求数据发送到php-fpm进程管理器

  - php-fpm进程管理器将任务发给子进程（fork的子worker进程）

  - worker进程中的php解释器执行php文件来处理请求

  - php解释器处理好之后再通过fast-cgi协议将处理结果发给Nginx服务器

  - nginx将结果返回到客户端

- **Nginx与php-fpm之间有2种通信方式：**

  - **TCP socket：**通过 `ip:port`的方式进行通信
    - 可以将nginx和php-fpm分布在不同的服务器上
    - tcp socket：需要重新打包、拆包、计算校验、应答等
    - 缺点：需要有多余的tcp开销
    - 优点：高并发下，保证通信的正确性和完整性

  - **Unix socket**：通过 php启动生成的socket文件进行通信
    - nginx和php-fpm就只能部署在一台机器上
    - 优点：效率比tcp高，只是将数据进行进程间的拷贝，减少不必要的tcp开销
    - 缺点：高并发时不稳定，产生大量长时缓存，数据可能出错不返回异常

```nginx
server {
  location / {
    try_files $uri /index.php?$args;
  }
  location ~ \.php$ {
    fastcgi_split_path_info ^(.+\.php)(/.+)$;
    fastcgi_pass 127.0.0.1:9000;              # fastcgi进程监听的IP地址和端口
    fastcgi_index index.php; 
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_param PATH_INFO $fastcgi_path_info;
  }
}
```

# 十四、Rewrite重写

- Nginx通过ngx_http_rewrite_module 模块支持URL重写、支持if条件判断，但不支持else
- rewrite功能就是，使用nginx提供的全局变量或自己设置的变量，结合正则表达式和标志位实现url重写以及重定向

- rewrite只能放在server{},location{},if{}中，并且默认只能对域名后边的除去传递的参数外的字符串起作用，

- **rewrite 执行顺序**

  - 执行 server 块里面的 rewrite 指令

  - 执行 location 匹配

  - 执行选定的 location 中的 rewrite 指令

- **语法格式：** `rewrite <regex> <replacement> [flag]`

  - **regex** ：表示正则匹配规则

  - **replacement** ：表示跳转后的内容

  - **flag** ：表示 rewrite 支持的 flag 标记
    - last ：本条规则匹配完成后，继续向下匹配新的location URI规则，一般用在 server 和 if 中
    - break ：本条规则匹配完成即终止，不再匹配后面的任何规则，一般使用在 location 中
    - redirect：返回302临时重定向，浏览器地址会显示跳转后的URL地址
    - permanent：返回301永久重定向，浏览器地址栏会显示跳转后的URL地址。

- **rewrite 示例**

  - **基于域名跳转**

    ```bash
    server {
      listen       80;
      server_name  www.test.curiouser.com;                             #域名修改  
      charset utf-8;
      access_log  /var/log/nginx/www_test_curiouser_com.log;    #日志修改
      location / {                                         #添加域名重定向
        if ($host = 'www.test.curiouser.com'){              #$host为rewrite全局变量，代表请求主机头字段或主机名
          rewrite ^/(.*)$ http://www.test1.curiouser.com/$1 permanent;  #$1为正则匹配的内容，即域名后边的字符串
        }
        root   html;
        index  index.html index.htm;
      }
    }
    ```

  - **基于客户端 IP 访问跳转**

    ```nginx
    server {
      listen       80;
      server_name  www.curiouser.com;    #域名修
      charset utf-8;
      access_log  log/nginx/www-curiouser-com-access.log;    #日志修改
    
      # 设置是否合法的IP标记
      set $rewrite true;              #设置变量$rewrite，变量值为boole值true
      
      # 判断是否为合法IP
      if ($remote_addr = "192.168.1.8"){     #当客户端IP为192.168.1.8时，将变量值设为false，不进行重写
        set $rewrite false;
      }
      
      # 除了合法IP，其它都是非法IP，进行重写跳转维护页面
      if ($rewrite = true){           #当变量值为true时，进行重写
        rewrite (.+) /re.html;    #将域名后边的路径重写成/re.html后转发，例如www.curiouser.com/re.html
      }
      location = /re.html {
        root /var/www/html;         #网页返回/var/www/html/re.html的内容
      }
      location / {
        root   html;
        index  index.html index.htm;
      }
    }
    ```

  参考：

  - https://zhuanlan.zhihu.com/p/359801091
  - https://blog.51cto.com/u_14842009/5146017

# 十五、代理 WebSocket

官方文档：https://nginx.org/en/docs/http/websocket.html

```bash
map $http_upgrade $connection_upgrade { 
  # 如果$http_upgrade 不为 '' (空)， 则$connection_upgrade 为 upgrade 。
	default upgrade; 
	# 如果$http_upgrade 为 '' (空)， 则 $connection_upgrade 为 close。
	'' close; 
} 
server {
  listen 80;
  server_name example.com;
    
  location /socket.io {
    # 将请求转发到websocket服务上
    proxy_pass http://192.168.1.1:8084;
    # 使用 HTTP 1.1 协议进行代理。
    proxy_http_version 1.1;
    # 设置 Upgrade 请求头，其值与客户端请求中的 Upgrade 头相同。
    proxy_set_header Upgrade $http_upgrade; 
    # 设置 Connection 请求头为 "upgrade"，表明这是一个 WebSocket 连接。
    proxy_set_header Connection $connection_upgrade;
    
    proxy_redirect off;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr; 
    proxy_read_timeout 3600s; 
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
	}
}
```

参考：https://www.cnblogs.com/binghe001/p/14752404.html

# 十六、支持 Brotli压缩算法

目前仅在NGINX Plus版本中支持。参考：https://docs.nginx.com/nginx/admin-guide/dynamic-modules/brotli/

手动编译支持：https://github.com/google/ngx_brotli





参考：

- https://www.cnblogs.com/-wenli/p/13594882.html
- https://cloud.tencent.com/developer/article/1501009
