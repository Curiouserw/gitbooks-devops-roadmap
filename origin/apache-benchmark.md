# Apache Benchmark

# 一、简介

ab命令会创建多个并发访问线程，模拟多个访问者同时对某一URL地址进行访问。它的测试目标是基于URL的，因此，它既可以用来测试apache的负载压力，也可以测试nginx、lighthttp、tomcat、IIS等其它Web服务器的压力。ab命令对发出负载的计算机要求很低，它既不会占用很高CPU，也不会占用很多内存。但却会给目标服务器造成巨大的负载，其原理类似CC攻击。自己测试使用也需要注意，否则一次上太多的负载。可能造成目标服务器资源耗完，严重时甚至导致死机。

- **吞吐率**：服务器并发处理能力的量化描述，单位是reqs/s，指的是某个并发用户数下单位时间内处理的请求数。某个并发用户数下单位时间内能处理的最大请求数，称之为最大吞吐率。
- **并发连接数**：某一时刻服务器所接受的请求数（会话数）。
- **并发用户数**：某一时刻服务器所接受的连接数，一个用户可能同时产生多个连接。
- **用户平均请求等待时间**：总请求数 / 并发用户数。
- **服务器平均请求等待时间**：处理完成所有请求数所花费的时间 / 总请求数。
- **PV**： 即page view，页面浏览量，用户每一次对网站中的每个页面访问均被记录1次。用户对同一页面的多次刷新，访问量累计。
- **UV**：即Unique visitor，独立访客，通过客户端的cookies实现。即同一页面，客户端多次点击只计算一次，访问量不累计。
- **IP**： 即Internet Protocol，本意本是指网络协议，在数据统计这块指通过ip的访问量。即同一页面，客户端使用同一个IP访问多次只计算一次，访问量不累计。
- **TPS**：即Transactions Per Second的缩写，每秒处理的事务数目。一个事务是指一个客户机向服务器发送请求然后服务器做出反应的过程。客户机在发送请求时开始计时，收到服务器响应后结束计时，以此来计算使用的时间和完成的事务个数，最终利用这些信息作出的评估分。
- **QPS**：即Queries Per Second的缩写，每秒能处理查询数目。是一台服务器每秒能够相应的查询次数，是对一个特定的查询服务器在规定时间内所处理流量多少的衡量标准。QPS=并发量 / 平均响应时间
- **RPS**：即Requests Per Second的缩写，每秒能处理的请求数目。等效于QPS

# 二、安装

Linux

```bash
yum install httpd-tools
```

Macos

```bash
自带
```

# 三、命令参数

```bash
-n requests    执行的请求数，即一共发起多少请求。
-c concurrency 请求并发数。
-t timelimit  测试所进行的最大秒数。其内部隐含值是-n 50000，它对服务器的测试限制在一个固定的总时间以内。默认没有时间限制。
-s timeout    指定每个请求的超时时间，默认是30秒。
-b windowsize 指定tcp窗口的大小，单位是字节。
-B address    指定在发起连接时绑定的ip地址是什么。
-p postfile   指定要POST的文件，同时要设置-T参数。
-u putfile    指定要PUT的文件，同时要设置-T参数。
-T content-type  指定使用POST或PUT上传文本时的文本类型，默认是'text/plain'。
-v verbosity    设置详细模式等级。
-w    将结果输出到html的表中。
-i    使用HEAD方式代替GET发起请求。
-y attributes    以表格方式输出时，设置html表格tr属性。 
-z attributes    以表格方式输出时，设置html表格th或td属性。
-C attribute    添加cookie,比如'Apache=1234'。（可重复）
-H attribute    为请求追加一个额外的头部，比如'Accept-Encoding: gzip'。（可重复）
-A attribute    对服务器提供BASIC认证信任。用户名和密码由一个:隔开，并以base64编码形式发送。无论服务器是否需要(即,是否发送了401认证需求代码)，此字符串都会被发送。
-P attribute    对一个中转代理提供BASIC认证信任。用户名和密码由一个:隔开，并以base64编码形式发送。无论服务器是否需要(即, 是否发送了401认证需求代码)，此字符串都会被发送。
-X proxy:port   指定代理服务器的IP和端口。
-V              打印版本信息。
-k              启用HTTP KeepAlive功能，即在一个HTTP会话中执行多个请求。默认时，不启用KeepAlive功能。
-d              不显示"percentage served within XX [ms] table"的消息(为以前的版本提供支持)。
-q          如果处理的请求数大于150，ab每处理大约10%或者100个请求时，会在stderr输出一个进度计数。此-q标记可以抑制这些信息。
-g filename    把所有测试结果写入一个'gnuplot'或者TSV(以Tab分隔的)文件。此文件可以方便地导入到Gnuplot,IDL,Mathematica,Igor甚至Excel中。其中的第一行为标题。
-e filename    产生一个以逗号分隔的(CSV)文件，其中包含了处理每个相应百分比的请求所需要(从1%到100%)的相应百分比的(以微妙为单位)时间。由于这种格式已经“二进制化”，所以比'gnuplot'格式更有用。
-r              当收到错误时不要退出。
-h              输出帮助信息
-Z ciphersuite  指定SSL/TLS密码套件
-f protocol     指定SSL/TLS协议(SSL3, TLS1, TLS1.1, TLS1.2 or ALL)
```

# 四、测试结果详解

> ab -n 1000 -c 10 https://www.baidu.com/

```bash
# 测试服务器的名字
Server Software:      
# 请求的URL主机名  
Server Hostname:        www.baidu.com
#web服务器监听的端口
Server Port:            443
SSL/TLS Protocol:       TLSv1.2,ECDHE-RSA-AES128-GCM-SHA256,2048,128
Server Temp Key:        ECDH P-256 256 bits
TLS Server Name:        www.baidu.com
# 请求的URL中的根绝对路径
Document Path:          /
# HTTP响应数据的正文长度
Document Length:        227 bytes

# 并发用户数，这是ab命令中设置的-c参数
Concurrency Level:      10
# 所有这些请求被处理完成所花费的总时间
Time taken for tests:   12.537 seconds
# 总请求数量，这是ab命令中设置的-n参数
Complete requests:      1000
# 失败的请求数，这里的失败是指请求的连接服务器、发送数据、接收数据等环节发生异常，以及无响应后超时的情况。对于超时时间的设置可以用ab的-t参数。如果接受到的http响应数据的头信息中含有2xx以外的状态码，则会在测试结果显示另一个名为“Non-2xx responses”的统计项，用于统计这部分请求数，这些请求并不算是失败的请求。
Failed requests:        0
#写入错误
Write errors:           0
# 总的网络传输量(字节数)，包含http的头信息等。使用ab的-v参数即可查看详细的http头信息。
Total transferred:      1110922 bytes
# HTML内容传输量，实际的页面传递字节数。也就是减去了Total transferred中http响应数据中头信息的长度。
HTML transferred:       227000 bytes
# 每秒处理的请求数，服务器的吞吐量，等于：Complete requests / Time taken for tests(QPS)
Requests per second:    79.76 [#/sec] (mean)
# 用户平均请求等待时间
Time per request:       125.369 [ms] (mean)
# 服务器平均处理时间
Time per request:       12.743 [ms] (mean, across all concurrent requests)
# 平均每秒网络上的流量,即每秒收到的速率
Transfer rate:          86.54 [Kbytes/sec] received
 
# 压力测试时的连接处理时间。-- min最小值、mean平均值、[+/-sd]方差、median中位数、maxz最大值
Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:       49   95  25.8    89      201          # socket链路建立消耗，代表网络状况好
Processing:    12   29  14.9    25      131          # 写入缓冲区消耗+链路消耗+服务器消耗
Waiting:       12   26  12.7    23      130          # 写入缓冲区消耗+链路消耗+服务器消耗+读取数据消耗
Total:         67  124  31.3    116     245          # 单个事务总时间

Percentage of the requests served within a certain time (ms)
  50%    116
  66%    127
  75%    136
  80%    142
  90%    169
  95%    192
  98%    213
  99%    227
 100%    245 (longest request)
# 整个场景中所有请求的响应情况。在场景中每个请求都有一个响应时间，其中50％的用户响应时间小于119毫秒，66％的用户响应时间小于141毫秒，最大的响应时间小于281毫秒
```



# 五、实操

## 1、测试试socket出现打开文件过多

```bash
socket: Too many open files (24)
```

解决：

```bash
ulimit -n
ulimit -n 204800
```

## 2、测试网址要带URL

```bash
# 不带路径则无法测试
https://www.baidu.com
# 以下路径可以测试
https://www.baidu.com/
```



