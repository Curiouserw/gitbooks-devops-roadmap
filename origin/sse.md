# SSE(Server-Sent Events)

# 一、简介概要

 Server-Sent Events（SSE）是一种基于 HTTP 的单向推送技术，允许服务端通过持久化的 HTTP 连接向客户端实时发送数据更新，本质上是使用 MIME 类型 `text/event-stream` 进行流式传输，同时客户端通过 JavaScript 的 EventSource 对象进行接收、解析与自动重连管理。相比传统的轮询和长轮询，SSE 简化了客户端逻辑、降低了连接数开销，并且与浏览器的事件模型天然契合，适合轻量级的实时通知、日志推送、行情更新等场景

## **1、HTTP 头与连接**

- **请求方式**
   客户端通过标准的 HTTP GET 请求向服务端指定的 SSE 端点发起连接，不需要额外的握手协议或升级流程。
- **响应头**
   服务端必须在响应中设置 `Content-Type: text/event-stream`，并通常会加入 `Cache-Control: no-cache` 以防缓存中断数据流；某些反向代理（如 Nginx）还需关闭缓冲（`X-Accel-Buffering: no`）以确保数据及时推送。

## **2、数据帧格式**

SSE 使用简单的纯文本格式定义事件帧（event frame），主要包含四种字段：

```
id: <event-id>          # 可选，设置 lastEventId  
event: <event-name>     # 可选，定义自定义事件类型  
data: <payload>         # 必需，可多行，表示事件主体  
retry: <milliseconds>   # 可选，指定下次重连时的延迟  
\n                      # 以空行分隔事件
```

- 每个字段以 `<field>:<value>\n` 形式出现，多行 `data:` 会被拼接成一个完整的事件体；
- 空行（双换行）标志一条事件的结束；
- `id` 字段会被客户端保存为 `lastEventId`，在重连时自动通过 `Last-Event-ID` 请求头告知服务端，便于断点续传

# 二、客户端EventSource

## 1、创建与事件监听

在前端，使用原生的 `EventSource` 对象即可建立 SSE 连接：

```ini
const source = new EventSource('/sse-endpoint');
source.onmessage = e => console.log('收到数据：', e.data);
source.onerror = e => console.error('连接错误', e);
```

- 构造函数 `new EventSource(url, { withCredentials: true|false })` 会自动发起持久化 GET 请求，并封装就绪状态 `readyState`（0=CONNECTING, 1=OPEN, 2=CLOSED）及事件回调。
- 默认情况下 SSE 遵循同源策略，如需跨域，需配合 CORS 设置响应头 `Access-Control-Allow-Origin` 等。

## 2、**自动重连机制**

- 当连接意外断开（网络波动、服务端关闭等）时，`EventSource` 会在默认 3 秒后自动尝试重连；
- 可通过服务端发送 `retry: <ms>` 字段动态调整客户端的重连等待时间；
- 重连请求会自动携带 `Last-Event-ID` 头，保证服务端从上次断点继续推送未接收的事件。

# 三、服务端示例代码

- **TLS 启动 (HTTPS)**
- **JWT 鉴权 (基于 Cookie)**
- **自定义 TokenBucket 限流**
- **心跳 (Heartbeat)**
- **Redis Pub/Sub 多实例水平扩展**
- **多种自定义事件类型**
- **Last-Event-ID 历史重放 (断线重连恢复)**
- **环形缓冲区缓存最近 N 条事件**
- **优雅停机**

```go
package main

import (
    "context"          // 上下文管理，用于 Redis 订阅和请求取消
    "encoding/json"   // JSON 编解码
    "fmt"              // 格式化 I/O
    "log"              // 日志输出
    "net/http"         // HTTP 服务
    "os"               // 操作系统接口，用于信号捕获
    "os/signal"        // 信号通知
    "sync"             // 互斥锁和读写锁
    "syscall"          // 系统调用，如 SIGINT/SIGTERM
    "time"             // 时间相关

    "github.com/dgrijalva/jwt-go"    // JWT 解析与验证库
    "github.com/gin-contrib/cors"    // Gin 的 CORS 中间件
    "github.com/gin-gonic/gin"       // Gin Web 框架
    "github.com/go-redis/redis/v8"   // Redis 客户端
)

// ==== 全局配置 & 变量 ====
var (
    JWTSecret   = []byte("your-256-bit-secret") // JWT HMAC 签名密钥，请替换成安全值

    // Redis 相关配置
    redisAddr   = "localhost:6379"             // Redis 地址
    redisTopic  = "events"                     // 订阅/发布的频道名称
    ctx         = context.Background()           // Redis 操作的根上下文
    rdb         = redis.NewClient(&redis.Options{Addr: redisAddr})

    // 历史消息环形缓冲区，用于 Last-Event-ID 重放
    history      = NewHistoryBuffer(200)          // 缓存最近 200 条事件
    nextEventID  = int64(1)                      // 全局自增事件 ID
)

// ==== 自定义限流：TokenBucket 实现 ====
// TokenBucket 维护一个容量有限的令牌桶，用于限流
// capacity: 桶最大容量，rate: 每秒放入的令牌数
type TokenBucket struct {
    capacity  int64       // 最大令牌数
    rate      float64     // 每秒添加令牌数
    tokens    float64     // 当前可用令牌数
    lastToken time.Time   // 上次令牌更新的时间
    mtx       sync.Mutex  // 互斥锁，保护并发访问
}

// NewTokenBucket 构造函数，初始化令牌桶状态
func NewTokenBucket(capacity int64, rate float64) *TokenBucket {
    return &TokenBucket{
        capacity:  capacity,
        rate:      rate,
        tokens:    float64(capacity),     // 初始填满
        lastToken: time.Now(),            // 记录初始化时间
    }
}

// Allow 尝试从令牌桶中获取一个令牌，返回 true 则允许通过，否则拒绝
func (tb *TokenBucket) Allow() bool {
    tb.mtx.Lock()
    defer tb.mtx.Unlock()

    // 计算自上次更新后应补充的令牌数
    now := time.Now()
    elapsed := now.Sub(tb.lastToken).Seconds()
    tb.tokens += elapsed * tb.rate            // 增加令牌
    if tb.tokens > float64(tb.capacity) {
        tb.tokens = float64(tb.capacity)      // 上限检查
    }
    tb.lastToken = now                        // 更新最后时间戳

    // 如果有足够的令牌，则扣减并允许请求
    if tb.tokens >= 1 {
        tb.tokens--
        return true
    }
    return false                              // 否则拒绝
}

// LimitHandler 返回一个 Gin 中间件，使用 TokenBucket 对请求进行限流
func LimitHandler(maxConn int64) gin.HandlerFunc {
    tb := NewTokenBucket(maxConn, float64(maxConn))  // rate = capacity/sec
    return func(c *gin.Context) {
        if !tb.Allow() {
            log.Println("[RATE] Too many requests, dropping connection")
            c.String(http.StatusServiceUnavailable, "Too many requests")
            c.Abort()
            return
        }
        c.Next()                                // 允许继续处理
    }
}

// ==== 历史缓冲区：用于断线重连恢复 ==== 
// SSEMessage 保存单条 SSE 事件信息
type SSEMessage struct {
    ID    int64  `json:"id"`              // 事件 ID
    Event string `json:"event"`          // 自定义事件类型
    Data  string `json:"data"`           // 事件数据
}

// HistoryBuffer 是一个简单的环形缓冲区，存储最近 capacity 条 SSEMessage
type HistoryBuffer struct {
    buf      []SSEMessage    // 底层切片
    capacity int             // 最大容量
    mu       sync.RWMutex    // 读写锁
}

// NewHistoryBuffer 创建新的环形缓冲区，容量 cap
func NewHistoryBuffer(cap int) *HistoryBuffer {
    return &HistoryBuffer{buf: make([]SSEMessage, 0, cap), capacity: cap}
}

// Add 将新消息添加到缓冲，若已满则删除最旧一条
func (h *HistoryBuffer) Add(msg SSEMessage) {
    h.mu.Lock()
    defer h.mu.Unlock()
    if len(h.buf) == h.capacity {
        h.buf = h.buf[1:]
    }
    h.buf = append(h.buf, msg)
}

// Since 返回所有 ID > lastID 的历史消息，供重连客户端重放
func (h *HistoryBuffer) Since(lastID int64) []SSEMessage {
    h.mu.RLock()
    defer h.mu.RUnlock()
    var out []SSEMessage
    for _, m := range h.buf {
        if m.ID > lastID {
            out = append(out, m)
        }
    }
    return out
}

// ==== JWT 鉴权中间件 ==== 
// authMiddleware 在 SSE 端点和发布接口之前校验 token
func authMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        path := c.Request.URL.Path
        // 跳过无需鉴权的路由
        if path == "/" || path == "/login" || path == "/login.html" {
            return
        }
        // 从 Cookie 获取 JWT
        tokenStr, err := c.Cookie("token")
        if err != nil {
            log.Printf("[AUTH] Missing token cookie: %v", err)
            c.AbortWithStatusJSON(http.StatusUnauthorized, gin.H{"error": "unauthorized"})
            return
        }
        // 解析并验证 JWT
        token, err := jwt.Parse(tokenStr, func(t *jwt.Token) (interface{}, error) {
            if _, ok := t.Method.(*jwt.SigningMethodHMAC); !ok {
                return nil, fmt.Errorf("unexpected signing method: %v", t.Header["alg"])
            }
            return JWTSecret, nil
        })
        if err != nil || !token.Valid {
            log.Printf("[AUTH] Invalid token: %v", err)
            c.AbortWithStatusJSON(http.StatusUnauthorized, gin.H{"error": "unauthorized"})
            return
        }
        // 从 claims 提取用户名并存入上下文
        claims, ok := token.Claims.(jwt.MapClaims)
        if !ok || claims["username"] == nil {
            log.Println("[AUTH] Invalid token claims")
            c.AbortWithStatusJSON(http.StatusUnauthorized, gin.H{"error": "unauthorized"})
            return
        }
        c.Set("username", claims["username"])
        log.Printf("[AUTH] User %v authenticated", claims["username"])
    }
}

// ==== 登录 & JWT 生成 ====
// handleLogin 验证 用户名/密码，生成 JWT 并写入 HttpOnly Cookie
func handleLogin(c *gin.Context) {
    var form struct{
        Username string `form:"username" json:"username" binding:"required"`
        Password string `form:"password" json:"password" binding:"required"`
    }
    if err := c.ShouldBind(&form); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }
    // TODO: 替换为实际的密码验证逻辑
    if form.Password != "password" {
        c.JSON(http.StatusUnauthorized, gin.H{"error": "invalid credentials"})
        return
    }
    // 构建 JWT 并签名
    token := jwt.NewWithClaims(jwt.SigningMethodHS256, jwt.MapClaims{
        "username": form.Username,
        "exp":      time.Now().Add(time.Hour).Unix(),
    })
    ts, err := token.SignedString(JWTSecret)
    if err != nil {
        c.JSON(http.StatusInternalServerError, gin.H{"error": "token generation failed"})
        return
    }
    // 写 HttpOnly Cookie，前端无法通过 JS 读取
    c.SetCookie("token", ts, 3600, "/", "", false, true)
    c.JSON(http.StatusOK, gin.H{"status": "ok"})
}

// ==== 发布接口 handlePublish ====
// 接受 JSON 格式 {event, data}，将其发布到 Redis 频道
func handlePublish(c *gin.Context) {
    var body struct{ Event, Data string }
    if err := c.BindJSON(&body); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }
    payload, _ := json.Marshal(body)
    if err := rdb.Publish(ctx, redisTopic, payload).Err(); err != nil {
        c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
        return
    }
    c.Status(http.StatusNoContent)
}

// ==== SSE 端点 sseHandler ====
// 处理 SSE 客户端连接，支持历史重放、实时推送、心跳和限流
func sseHandler(c *gin.Context) {
    log.Println("[SSE] New connection")
    w := c.Writer
    req := c.Request

    // 设置 SSE 必需的响应头
    w.Header().Set("Content-Type", "text/event-stream")
    w.Header().Set("Cache-Control", "no-cache")
    w.Header().Set("Connection", "keep-alive")
    flusher, ok := w.(http.Flusher)
    if !ok {
        c.Status(http.StatusInternalServerError)
        return
    }

    // 重连支持：读取客户端 Last-Event-ID 并重放历史消息
    lastID := int64(0)
    if hdr := req.Header.Get("Last-Event-ID"); hdr != "" {
        fmt.Sscanf(hdr, "%d", &lastID)
        log.Printf("[SSE] Last-Event-ID=%d", lastID)
    }
    for _, msg := range history.Since(lastID) {
        fmt.Fprintf(w, "id: %d event: %s data: %s", msg.ID, msg.Event, msg.Data)
    }
    flusher.Flush()

    // 订阅 Redis 频道，监听实时消息
    sub := rdb.Subscribe(ctx, redisTopic)
    defer sub.Close()
    ch := sub.Channel()

    // 心跳定时器：每 30s 发送注释行，防止代理超时关闭连接
    hb := time.NewTicker(30 * time.Second)
    defer hb.Stop()

    // 本地限流器：避免单连接过快推送导致阻塞
    tb := NewTokenBucket(10, 1) // 桶容量 10，速率 1 token/sec

    for {
        select {
        case raw := <-ch:
            // 本地限流
            if !tb.Allow() {
                log.Println("[RATE] drop event")
                continue
            }
            // 解析发布的 JSON 数据
            var inc struct{ Event, Data string }
            if err := json.Unmarshal([]byte(raw.Payload), &inc); err != nil {
                log.Printf("[SSE] invalid payload: %v", err)
                continue
            }
            // 构造 SSEMessage 并加入历史缓冲
            msg := SSEMessage{ID: nextEventID, Event: inc.Event, Data: inc.Data}
            nextEventID++
            history.Add(msg)

            // 推送给客户端
            fmt.Fprintf(w, "id: %d event: %s data: %s", msg.ID, msg.Event, msg.Data)
            flusher.Flush()
            log.Printf("[SSE] sent id=%d event=%s", msg.ID, msg.Event)

        case <-hb.C:
            // 心跳注释行
            fmt.Fprintf(w, ": ping")
            flusher.Flush()
            log.Println("[SSE] heartbeat sent")

        case <-req.Context().Done():
            log.Println("[SSE] client disconnected")
            return
        }
    }
}

func main() {
    // 优雅停机：监听 SIGINT/SIGTERM
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)

    // Gin 模式与中间件
    gin.SetMode(gin.ReleaseMode)
    router := gin.New()
    router.Use(gin.Logger(), gin.Recovery(), cors.Default())

    // 静态文件 & 登录页面
    router.LoadHTMLFiles("static/login.html")
    router.GET("/login.html", func(c *gin.Context) { c.HTML(http.StatusOK, "login.html", nil) })
    router.POST("/login", handleLogin)

    // 发布接口 & SSE 端点：均需鉴权
    router.POST("/publish", authMiddleware(), handlePublish)
    router.GET("/events", authMiddleware(), LimitHandler(10), sseHandler)

    // 启动 HTTPS 服务
    certFile := "path/to/cert.pem"  // 替换为实际路径
    keyFile := "path/to/key.pem"   // 替换为实际路径
    srv := &http.Server{Addr: ":8443", Handler: router}

    go func() {
        log.Println("[HTTP] HTTPS listening on :8443")
        if err := srv.ListenAndServeTLS(certFile, keyFile); err != nil && err != http.ErrServerClosed {
            log.Fatalf("HTTPS listen error: %v", err)
        }
    }()

    <-quit
    log.Println("[HTTP] Shutdown initiated")
    ctxShut, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()
    if err := srv.Shutdown(ctxShut); err != nil {
        log.Fatalf("Server Shutdown: %v", err)
    }
    log.Println("[HTTP] Server exited gracefully")
}
```



# 参考

- https://developer.mozilla.org/zh-CN/docs/Web/API/Server-sent_events/Using_server-sent_events
- https://blog.csdn.net/xuejianxinokok/article/details/136830066
- https://www.zixi.org/archives/517.html
- https://juejin.cn/post/7197116653196001340
- https://wangdoc.com/webapi/server-sent-events
- https://blog.yuanpei.me/posts/3175881014/