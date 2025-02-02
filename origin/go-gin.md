# Golang的Web框架Gin使用总结

# 一、Gin基础设置

```bash
func main(){
	# 设置当前运行模式
	gin.SetMode(gin.ReleaseMode)
	
	r := gin.Default()
}
```

# 二、使用BasicAuth中间件限制指定接口

```go
router := gin.Default()

authorizedRoute = router.Group("/", gin.BasicAuth(gin.Accounts{
  "admin": "123456", //用户名：密码
  "root": "aaaaaaa",
}))

// Group函数注册了一个群组路由，gin.BasicAuth是中间件，参数gin.Accounts是一个map[string]string类型的映射，用来记录用户名和密码。

authorizedRoute.StaticFile("/favicon.ico", "./public/favicon.ico")

authorizedRoute.GET("/secrets", func(c *gin.Context) {
		// get user, it was set by the BasicAuth middleware
		user := c.MustGet(gin.AuthUserKey).(string)
		if secret, ok := secrets[user]; ok {
			c.JSON(http.StatusOK, gin.H{"user": user, "secret": secret})
		} else {
			c.JSON(http.StatusOK, gin.H{"user": user, "secret": "NO SECRET :("})
		}
})
```

# 三、重定向路由

```go
router := gin.Default()

router.GET("/", func(context *gin.Context) {
		context.Request.URL.Path = "/public"
		router.HandleContext(context)
})

// 或者重定向到外部连接
router.GET("/test", func(context *gin.Context) {
	context.Redirect(http.StatusMovedPermanently, "http://www.google.com/")
})

// 重定向POST请求
router.POST("/test", func(context *gin.Context) {
	context.Redirect(http.StatusFound, "/foo")
})
```

# 四、获取请求中的数据

## 1、打印请求Body

```go
router := gin.Default()

router.POST("/welcome", func(context *gin.Context) {
	requestBody, _ := io.ReadAll(context.Request.Body)
  println(string(requestBody))
})

router.POST("/test", func(context *gin.Context) {
	  var jsonBody map[string]interface{}
		if err := c.BindJSON(&jsonBody); err != nil {
			c.JSON(400, gin.H{"error": "Invalid JSON"})
			return
		}
    // 格式化并打印JSON
		formattedJSON, err := json.MarshalIndent(jsonBody, "", "  ")
    if err != nil {
			fmt.Println("JSON格式化错误:", err)
		} else {
			fmt.Println("格式化后的JSON体:\n", string(formattedJSON))
		}
		c.JSON(200, gin.H{"status": "OK"})
})
```

## 2、获取URL路径

### ①获取单个或多个请求URL路径

```go
router := gin.Default()

// 只会匹配“/user/john” 不会匹配 “/user/“ 或者 “/user“

router.GET("/user/:name", func(context *gin.Context) {
	name := context.Param("name")
	context.String(http.StatusOK, "Hello %s", name)
})

// However, this one will match /user/john/ and also /user/john/send
// If no other routers match /user/john, it will redirect to /user/john/
router.GET("/user/:name/*action", func(context *gin.Context) {
	name := context.Param("name")
	action := context.Param("action")
	message := name + " is " + action
	context.String(http.StatusOK, message)
})

// 对于每个匹配的请求，Context将保留路由定义
router.POST("/user/:name/*action", func(context *gin.Context) {
	context.FullPath() == "/user/:name/*action" // true
})

```

## 3、获取URL参数

### ①请求中的参数

> `/welcome?firstname=Jane&lastname=Doe`

```go
router := gin.Default()

router.GET("/welcome", func(context *gin.Context) {
	firstname := context.DefaultQuery("firstname", "默认值")
	lastname := context.Query("lastname") 
  // 也可以写成 context.Request.URL.Query().Get("lastname")
	context.String(http.StatusOK, "Hello %s %s", firstname, lastname)
})
```

### ②请求中的参数数组

`/post?ids[a]=1234&ids[b]=hello`

```go
router := gin.Default()

router.GET("/welcome", func(context *gin.Context) {
	ids := context.QueryMap("ids")
	context.String(http.StatusOK, "Hello %s %s", firstname, lastname)
})
```

## 4、获取POST请求表单数据

### ①表单的多个数据

> ```
> POST /post?id=1234&page=1 HTTP/1.1
> Content-Type: application/x-www-form-urlencoded
> 
> name=manu&message=this_is_great
> ```

```go
router := gin.Default()

router.POST("/post", func(context *gin.Context) {
		name := context.PostForm("name")
		message := context.PostForm("message")
		fmt.Printf("name: %s; message: %s", name, message)
})
```

### ②表单的数组数据：

>```
>POST /post?ids[a]=1234&ids[b]=hello HTTP/1.1
>Content-Type: application/x-www-form-urlencoded
>
>names[first]=thinkerou&names[second]=tianou
>```

```go
router := gin.Default()

router.POST("/post", func(context *gin.Context) {
	names := context.PostFormMap("names")
	fmt.Printf("names: %v", names)
})
```

# 五、文件上传

## 1、单个文件上传

```go
router := gin.Default()
	// Set a lower memory limit for multipart forms (default is 32 MiB)
router.MaxMultipartMemory = 8 << 20  // 8 MiB
router.POST("/upload", func(context *gin.Context) {
	file, _ := context.FormFile("file")
	log.Println(file.Filename)
	// Upload the file to specific dst.
	context.SaveUploadedFile(file, dst)
	context.String(http.StatusOK, fmt.Sprintf("'%s' uploaded!", file.Filename))
})
```

> ```
> curl -X POST http://localhost:8080/upload \
>   -F "file=@/Users/curiouser/test.txt" \
>   -H "Content-Type: multipart/form-data"
> ```

## 2、多个文件上传

```go
router := gin.Default()
// Set a lower memory limit for multipart forms (default is 32 MiB)
router.MaxMultipartMemory = 8 << 20  // 8 MiB
router.POST("/upload", func(context *gin.Context) {
	// Multipart form
	form, _ := context.MultipartForm()
	files := form.File["upload[]"]

	for _, file := range files {
		log.Println(file.Filename)

		// Upload the file to specific dst.
		context.SaveUploadedFile(file, dst)
	}
	context.String(http.StatusOK, fmt.Sprintf("%d files uploaded!", len(files)))
})
```

> ```bash
> curl -X POST http://localhost:8080/upload \
>     -F "upload[]=@/Users/curiouser/test1.txt" \
>     -F "upload[]=@/Users/curiouser/test2.txt" \
>     -H "Content-Type: multipart/form-data"
> ```



# 六、绑定请求数据到结构体中

## 1、绑定请求路径到结构体中

> ```
> GET /thinkerou/987fbc97-4bed-5078-9f07-9141ba07c9f3
> GET /thinkerou/not-uuid
> ```

```go
type Person struct {
	ID string `uri:"id" binding:"required,uuid"`
	Name string `uri:"name" binding:"required"`
}

route := gin.Default()
route.GET("/:name/:id", func(context *gin.Context) {
		var person Person
		if err := context.ShouldBindUri(&person); err != nil {
			context.JSON(400, gin.H{"msg": err})
			return
		}
})
```

## 2、绑定请求参数到结构体中

> ```
> GET  /testing?name=curiouser&address=xyz&birthday=1993&createTime=123&unixTime=15622
> ```

```go
type Person struct {
        Name       string    `form:"name"`
        Address    string    `form:"address"`
        Birthday   time.Time `form:"birthday" time_format:"2006-01-02" time_utc:"1"`
        CreateTime time.Time `form:"createTime" time_format:"unixNano"`
        UnixTime   time.Time `form:"unixTime" time_format:"unix"`
}

route := gin.Default()
route.GET("/testing",func(context *gin.Context) {
	var person Person
    if context.ShouldBind(&person) == nil {
          log.Println(person.Name)
          log.Println(person.Address)
          log.Println(person.Birthday)
          log.Println(person.CreateTime)
          log.Println(person.UnixTime)
    }
})
```

## 3、绑定请求Header到结构体中

> ```bash
> curl -H "rate:300" -H "domain:music" 127.0.0.1:8080/test
> ```

```go
type testHeader struct {
	Rate   int    `header:"Rate"`
	Domain string `header:"Domain"`
}

route := gin.Default()
route.GET("/test", func(context *gin.Context) {
		h := testHeader{}
		if err := context.ShouldBindHeader(&h); err != nil {
			context.JSON(200, err)
		}
})
```

## 4、绑定HTML复选框中的数据到结构体

> ```html
> <form action="/test" method="POST">
>     <p>Check some colors</p>
>     <label for="red">Red</label>
>     <input type="checkbox" name="colors[]" value="red" id="red">
>     <label for="green">Green</label>
>     <input type="checkbox" name="colors[]" value="green" id="green">
>     <label for="blue">Blue</label>
>     <input type="checkbox" name="colors[]" value="blue" id="blue">
>     <input type="submit">
> </form>
> ```

```go
type colorsForm struct {
    Colors []string `form:"colors[]"`
}

route := gin.Default()
route.GET("/test", func(context *gin.Context) {
	var foo colorsForm
    context.ShouldBind(&foo)
    context.JSON(200, gin.H{"color": foo.Colors})
})
```

## 5、绑定表单数据到结构体

> ```bash
> curl -X POST -v --form name=user --form "avatar=@./avatar.png" http://localhost:8080/profile
> ```

```go
type ProfileForm struct {
	Name   string                `form:"name" binding:"required"`
	Avatar *multipart.FileHeader `form:"avatar" binding:"required"`
	// or for multiple files
	// Avatars []*multipart.FileHeader `form:"avatar" binding:"required"`
}

router := gin.Default()
router.POST("/profile", func(context *gin.Context) {
	// you can bind multipart form with explicit binding declaration:
	// c.ShouldBindWith(&form, binding.Form)
	// or you can simply use autobinding with ShouldBind method:
	var form ProfileForm
	// in this case proper binding will be automatically selected
	if err := context.ShouldBind(&form); err != nil {
		context.String(http.StatusBadRequest, "bad request")
		return
	}
	err := context.SaveUploadedFile(form.Avatar, form.Avatar.Filename)
	if err != nil {
		context.String(http.StatusInternalServerError, "unknown error")
		return
	}
	// db.Save(&form)
	context.String(http.StatusOK, "ok")
})

```

## 6、绑定请求体到结构体

```go
type Test struct {
	ID                   int    `json:"id"`
	AppID                string `json:"appId"`
}

router := gin.Default()
router.POST("/welcome", func(context *gin.Context) {
    var a Test
	if err := context.ShouldBind(&a); err != nil {
		context.String(http.StatusBadRequest, "bad request")
		return
	}
})
```

# 七、使用中间件

## 1、全局设置中间件

```go
router := gin.Default()

router.Use(gin.Logger())
router.Use(gin.Recovery())
```

## 2、单个路由设置中间件

```go
router := gin.Defalut()

authorized := router.Group("/login")
authorized.Use(AuthRequired())
// 简短写法
// authorized = router.Group("/login",gin.BasicAuth(gin.Accounts{
//		"admin": "111",
// ))

// 或者
router.GET("/benchmark", MyBenchLogger(), benchEndpoint)
```

# 八、设置接口支持跨域

```go
router.GET("/login",func(context *gin.Context) {
		context.Writer.Header().Set("Access-Control-Allow-Origin", "*")
		context.Header("Access-Control-Allow-Origin", "*") // 设置允许访问所有域
		context.Header("Access-Control-Allow-Methods", "POST, GET, OPTIONS, PUT, DELETE,UPDATE")
		context.Header("Access-Control-Allow-Headers", "Authorization, Content-Length, X-CSRF-Token, Token,session,X_Requested_With,Accept, Origin, Host, Connection, Accept-Encoding, Accept-Language,DNT, X-CustomHeader, Keep-Alive, User-Agent, X-Requested-With, If-Modified-Since, Cache-Control, Content-Type, Pragma,token,openid,opentoken")
		context.Header("Access-Control-Expose-Headers", "Content-Length, Access-Control-Allow-Origin, Access-Control-Allow-Headers,Cache-Control,Content-Language,Content-Type,Expires,Last-Modified,Pragma,FooBar")
		context.Header("Access-Control-Max-Age", "172800")
		context.Header("Access-Control-Allow-Credentials", "false")
		context.Set("content-type", "application/json")  //设置返回格式是json
}
```

参考：https://www.cnblogs.com/you-men/p/14054348.html

# 九、压缩响应数据

常见浏览器支持的压缩算法：**gzip, deflate, br(Brotli), zstd**

具体各个浏览器的支持详情参考：https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Encoding



## 1、Gzip压缩

gzip是GNUzip的缩写，最早用于UNIX系统的文件压缩。HTTP协议上的gzip编码是一种用来改进web应用程序性能的技术，web服务器和客户端（浏览器）必须共同支持gzip。目前主流的浏览器，Chrome,firefox,IE等都支持该协议。常见的服务器如Apache，Nginx，IIS同样支持gzip。

gzip压缩比率在3到10倍左右

Go gin gzip的实现: https://github.com/gin-contrib/gzip

> go get github.com/gin-contrib/gzip

```go
import  "github.com/gin-contrib/gzip"

// 全局路由使用gzip压缩响应数据
router.Use(gzip.Gzip(gzip.DefaultCompression))

// 根据正则排除的路由不使用 gzip压缩响应数据
router.Use(gzip.Gzip(gzip.DefaultCompression, gzip.WithExcludedPathsRegexs([]string{".*"})))
// 排除指定路径的路由不使用 gzip压缩响应数据
router.Use(gzip.Gzip(gzip.DefaultCompression, gzip.WithExcludedPaths([]string{"/api/"})))
```

- https://blog.rexskz.info/trip-for-finding-golang-memory-leak.html
- https://blog.hi917.com/detail/57.html

## 2、Brotli压缩

Brotli 是谷歌2015 年推出的开源无损压缩算法，它通过变种的LZ77 算法、Huffman 编码以及二阶文本建模等方式进行数据压缩，比常见的Gzip更高效。

**注意：Brotli 压缩算法在浏览器下，接口只有部署在 https 请求下生效，因为在http请求头Accept-Encoding中默认是没有 br的，只有gzip,deflate ,同时浏览器禁止在http请求中修改Accept-Encoding Header。参考：https://developer.mozilla.org/zh-CN/docs/Glossary/Forbidden_header_name。但是可以使用 Curl命令 、PostMan等其他工具进行测试**

### ①Google基于C的go实现

-  Github : https://github.com/google/brotli/tree/master/go

- 压缩率设置：`0 ~ 11`。设置默认为 5。

```go
import (
	"bytes"
	"net/http"
	"strings"
	"github.com/gin-gonic/gin"
	"github.com/google/brotli/go/cbrotli"
)

func BrotliCompress() gin.HandlerFunc {
	return func(c *gin.Context) {
		// 检查客户端是否支持Brotli压缩
		if strings.Contains(c.Request.Header.Get("Accept-Encoding"), "br") {
			writer := &brotliWriter{ResponseWriter: c.Writer, buf: &bytes.Buffer{}}
			c.Writer = writer
			c.Next()
			writer.compress()
		} else {
			c.Next()
		}
	}
}

type brotliWriter struct {
	gin.ResponseWriter
	buf *bytes.Buffer
}

func (b *brotliWriter) Write(data []byte) (int, error) {
	return b.buf.Write(data)
}

func (b *brotliWriter) compress() {
	b.Header().Set("Content-Encoding", "br")
	b.Header().Del("Content-Length")

	// 使用Brotli压缩
	writer := cbrotli.NewWriter(b.ResponseWriter, cbrotli.WriterOptions{Quality: 5})
	defer writer.Close()

	_, err := writer.Write(b.buf.Bytes())
	if err != nil {
		http.Error(b.ResponseWriter, "Failed to compress response", http.StatusInternalServerError)
		return
	}
}

func main(){
  router := gin.Default()
  // 指定路由使用 BR 压缩响应数据
  router.POST("/getlogs", BrotliCompress(), func(c *gin.Context) {
     ...
  }
}
```

测试

```bash
curl -v -H "Accept-Encoding: br" https://localhost:8443
```

### ②纯Go实现

- Github：https://github.com/andybalholm/brotli

```go
import (
	"bytes"
	"strings"
	"github.com/andybalholm/brotli"
	"github.com/gin-gonic/gin"
)

func BrotliCompress() gin.HandlerFunc {
	return func(c *gin.Context) {
		if strings.Contains(c.Request.Header.Get("Accept-Encoding"), "br") {
			writer := &brotliWriter{ResponseWriter: c.Writer, buf: &bytes.Buffer{}}
			c.Writer = writer
			c.Next()
			writer.compress()
		} else {
			c.Next()
		}
	}
}

type brotliWriter struct {
	gin.ResponseWriter
	buf *bytes.Buffer
}

func (b *brotliWriter) Write(data []byte) (int, error) {
	return b.buf.Write(data)
}

func (b *brotliWriter) compress() {
	b.Header().Set("Content-Encoding", "br")
	b.Header().Del("Content-Length")
	writer := brotli.NewWriter(b.ResponseWriter)
	defer writer.Close()
	writer.Write(b.buf.Bytes())
}

func main(){
  router := gin.Default()
  // 指定路由使用 BR 压缩响应数据
  router.POST("/getlogs", BrotliCompress(), func(c *gin.Context) {
     ..
  }
}
```



参考：https://blog.csdn.net/qq_34556414/article/details/109112165

