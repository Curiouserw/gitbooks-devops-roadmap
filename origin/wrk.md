# 命令行测试工具wrk

# 一、简介

GitHub地址：https://github.com/wg/wrk



常见的测试工具有：Apache ab, Apache JMeter (互联网公司用的较多)，LoadRunner 等

wrk 是一款针对 Http 协议的基准测试工具，它能够在单机多核 CPU 的条件下，使用系统自带的高性能 I/O 机制，如 epoll，kqueue 等，通过多线程和事件模式，对目标机器产生大量的负载。

> PS: 其实，wrk 是复用了 redis 的 ae 异步事件驱动框架，准确来说 ae 事件驱动框架并不是 redis 发明的, 它来至于 Tcl 的解释器 jim, 这个小巧高效的框架, 因为被 redis 采用而被大家所熟知。

# 二、安装

## 安装

- **源码编译/Linux**

  ```bash
  git clone https://github.com/wg/wrk.git wrk && \
  cd wrk && \
  make && \
  cp wrk /usr/local/bin 
  ```

- **MacOS**

  ```bash
  brew install wrk
  ```

- **Docker**

  Dockerfile

  alpine镜像的软件源中有wrk包，版本为：4.1.0

  ```bash
  FROM alpine:3.11.5
  
  RUN sed -i "s/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g" /etc/apk/repositories \
      && apk add --no-cache wrk
  
  VOLUME [ "/data" ]
  WORKDIR /data
  ENTRYPOINT ["wrk"]
  ```

  Makefile

  ```bash
  IMAGE_BASE = tools
  IMAGE_NAME = wrk
  IMAGE_VERSION = v1
  all: build
  build:
  		docker build --rm -f Dockerfile -t ${IMAGE_BASE}/${IMAGE_NAME}:${IMAGE_VERSION} .
  ```

  使用命令

  ```bash
  # 本地使用docker启动wrk容器
  docker run -it tools/wrk:v1 -t10 -c400 -d30s --latency https://www.baidu.com
  
  docker run -it -v .:/data tools/wrk:v1 -s test.lua -c400 -d30s --latency https://www.baidu.com
  
  # 在K8S集群中临时启动一个pod 
  kubectl run -n test --rm --generator=run-pod/v1 --image tools/wrk:v1 -i -- wrk -t10 -c400 -d30s --latency https://www.baidu.com
  ```

# 三、命令详解

```bash
wrk <参数选项> <HTTP服务URL>                            

# 参数选项：                                  
#    -c, --connections <N>  跟服务器建立并保持的TCP连接数量  
#    -d, --duration    <T>  压测时间           
#    -t, --threads     <N>  使用多少个线程进行压测                                                         
#    -s, --script      <S>  指定Lua脚本路径       
#    -H, --header      <H>  为每一个HTTP请求添加HTTP头      
#        --latency          在压测结束后，打印延迟统计信息   
#        --timeout     <T>  超时时间     
#    -v, --version          打印正在使用的wrk的详细版本信息
                                                      
# <N>代表数字参数，支持国际单位 (1k, 1M, 1G)
# <T>代表时间参数，支持时间单位 (2s, 2m, 2h)
```

# 四、压测报告解读

示例：对本地Nginx的Web服务进行10个线程、400个请求连接、长达30s的压力测试

```bash
wrk -t10 -c400 -d30s --latency http://localhost:8080/
```

30 秒压测过后，生成如下压测报告：

```bash
Running 30s test @ http://localhost:8080/
  10 threads and 400 connections（共12个测试线程，400个连接）
               （平均值） （标准差）  （最大值）（正负一个标准差所占比例）
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    （延迟）
    Latency    22.09ms    4.40ms 115.67ms   90.98%
    (每秒请求数)
    Req/Sec     1.09k   456.65     2.08k    59.60%
  Latency Distribution （延迟分布）
     50%   21.14ms
     75%   22.36ms
     90%   24.83ms
     99%   42.64ms
  326894 requests in 30.03s, 269.96MB read   (32.03s内处理了326894个请求，耗费流量269.96MB)
  Socket errors: connect 159, read 132, write 0, timeout 0  (发生错误数)
Requests/sec:  10884.16	(QPS 10884.16,即平均每秒处理请求数为10884.16)
Transfer/sec:      8.99MB (平均每秒流量8.99MB)
```

# 五、使用 Lua 脚本进行复杂测试

wrk可以通过编写 Lua 脚本的方式，在运行压测命令时，通过参数 `--script` 来指定 Lua 脚本，来进行Post请求

等复杂测试

wrk 支持在三个阶段对压测进行个性化，分别是启动阶段、运行阶段和结束阶段。每个测试线程，都拥有独立的Lua 运行环境。

**启动阶段:**

```lua
function setup(thread)
```

在脚本文件中实现 setup 方法，wrk 就会在测试线程已经初始化，但还没有启动的时候调用该方法。wrk会为每一个测试线程调用一次 setup 方法，并传入代表测试线程的对象 thread 作为参数。setup 方法中可操作该 thread 对象，获取信息、存储信息、甚至关闭该线程。

```lua
thread.addr             - get or set the thread's server address
thread:get(name)        - get the value of a global in the thread's env
thread:set(name, value) - set the value of a global in the thread's env
thread:stop()           - stop the thread
```

**运行阶段：**

```lua
function init(args)
function delay()
function request()
function response(status, headers, body)
```

- `init(args)`: 由测试线程调用，只会在进入运行阶段时，调用一次。支持从启动 wrk 的命令中，获取命令行参数；
- `delay()`： 在每次发送请求之前调用，如果需要定制延迟时间，可以在这个方法中设置；
- `request()`: 用来生成请求, 每一次请求都会调用该方法，所以注意不要在该方法中做耗时的操作；
- `response(status, headers, body)`: 在每次收到一个响应时被调用，为提升性能，如果没有定义该方法，那么wrk不会解析 `headers` 和 `body`；

**结束阶段：**

```bash
function done(summary, latency, requests)
```

done() 方法在整个测试过程中只会被调用一次，我们可以从给定的参数中，获取压测结果，生成定制化的测试报告。

**自定义 Lua 脚本中可访问的变量以及方法：**

变量：wrk

```lua
wrk = {
    scheme  = "http",
    host    = "localhost",
    port    = 8080,
    method  = "GET",
    path    = "/",
    headers = {},
    body    = nil,
    thread  = <userdata>,
  }
```

以上定义了一个 `table` 类型的全局变量，修改该 `wrk` 变量，会影响所有请求。

方法：

1. `wrk.fomat`
2. `wrk.lookup`
3. `wrk.connect`

上面三个方法解释如下：

```lua
function wrk.format(method, path, headers, body)

    wrk.format returns a HTTP request string containing the passed parameters
    merged with values from the wrk table.
    # 根据参数和全局变量 wrk，生成一个 HTTP rquest 字符串。

function wrk.lookup(host, service)

    wrk.lookup returns a table containing all known addresses for the host
    and service pair. This corresponds to the POSIX getaddrinfo() function.
    # 给定 host 和 service（port/well known service name），返回所有可用的服务器地址信息。

function wrk.connect(addr)

    wrk.connect returns true if the address can be connected to, otherwise
    it returns false. The address must be one returned from wrk.lookup().
    # 测试给定的服务器地址信息是否可以成功创建连接
```

# 六、通过 Lua 脚本压测示例

**调用 POST 接口：**

```lua
wrk.method = "POST"
wrk.body   = "foo=bar&baz=quux"
wrk.headers["Content-Type"] = "application/x-www-form-urlencoded"
```

注意: wrk 是个全局变量，这里对其做了修改，使得所有请求都使用 POST 的方式，并指定了 body 和 Content-Type头。

**自定义每次请求的参数：**

```lua
request = function()
   uid = math.random(1, 10000000)
   path = "/test?uid=" .. uid
   return wrk.format(nil, path)
end
```

在 request 方法中，随机生成 1~10000000 之间的 uid，并动态生成请求 URL.

**每次请求前，延迟 10ms:**

```lua
function delay()
   return 10
end
```

**请求的接口需要先进行认证，获取 token 后，才能发起请求，咋办？**

```lua
token = nil
path  = "/auth"

request = function()
   return wrk.format("GET", path)
end

response = function(status, headers, body)
   if not token and status == 200 then
      token = headers["X-Token"]
      path  = "/test"
      wrk.headers["X-Token"] = token
   end
end
```

上面的脚本表示，在 token 为空的情况下，先请求 `/auth` 接口来认证，获取 token, 拿到 token 以后，将 token 放置到请求头中，再请求真正需要压测的 `/test` 接口。

**压测支持 HTTP pipeline 的服务：**

```lua
init = function(args)
   local r = {}
   r[1] = wrk.format(nil, "/?foo")
   r[2] = wrk.format(nil, "/?bar")
   r[3] = wrk.format(nil, "/?baz")

   req = table.concat(r)
end

request = function()
   return req
end
```

通过在 init 方法中将三个 HTTP请求拼接在一起，实现每次发送三个请求，以使用 HTTP pipeline。



# 参考

1. https://www.cnblogs.com/quanxiaoha/p/10661650.html
2. [https://yeqown.github.io/2018/01/16/wrk%E6%80%A7%E8%83%BD%E6%B5%8B%E8%AF%95%E5%B7%A5%E5%85%B7%E4%BD%BF%E7%94%A8%E6%80%BB%E7%BB%93/](https://yeqown.github.io/2018/01/16/wrk性能测试工具使用总结/)