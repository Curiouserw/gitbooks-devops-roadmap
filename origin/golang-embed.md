# Golang embed嵌入静态资源文件

# 一、简介

一般编译出来的可执行二进制文件都是单个的文件，非常适合复制和部署。在实际使用中，除了二进制文件，可能还需要一些配置文件，或者静态文件，比如html模板、静态的图片、CSS、javascript等文件。如果这些文件也能打进到二进制文件中，只需复制、按照单个的可执行文件即可。

一些开源的项目很久以前就开始做这方面的工作，比如[gobuffalo/packr](https://github.com/gobuffalo/packr)、[markbates/pkger](https://github.com/markbates/pkger)、[rakyll/statik](https://github.com/rakyll/statik)、[knadh/stuffbin](https://github.com/knadh/stuffbin)等。

自 Go 1.16 版本开始引入。它提供了一种将静态文件或整个目录嵌入到 Go 二进制文件中的机制，以便在运行时访问这些文件，而无需依赖外部文件系统。

`embed` 包定义了 `FS` 接口，表示嵌入的文件系统。通过 `FS` 接口，可以使用 `ReadFile` 和 `ReadDir` 等方法来读取嵌入的文件或目录的内容。支持嵌入整个目录，而不仅仅是单个文件。

# 二、功能

## 1、嵌入多个文件

```go
//go:embed test.txt test2.txt
var f embed.FS
```

或者

```go
//go:embed test1.txt
//go:embed test2.txt
var f embed.FS
```

## 2、嵌入文件为字符串变量

### ①嵌入单个文件内容为字符串变量

```go
//go:embed hello.txt
var s filecontent
func main() {
    fmt.Println(filecontent)
}
```

### ②嵌入同一个文件为多个字符串变量

```go
//go:embed test.txt
var test1content string
//go:embed test.txt
var test2content string
func main() {
    fmt.Println(test1content)
    fmt.Println(test2content)
}
```

### ③嵌入文件内容为可见变量和不可见变量

```go
//go:embed hello.txt
var filecontent string
//go:embed hello.txt
var Filecontent string
func main() {
   fmt.Println(filecontent)
}
```

## 2、嵌入文件为字节数组

把单个文件的内容嵌入为字节数组slice of byte。

```go
//go:embed hello.txt
var filecontext []byte
func main() {
    fmt.Println(filecontext)
}
```

## 3、嵌入文件夹

```go
//go:embed statics
var f embed.FS
func main() {
    indexdata, _ := f.ReadFile("statics/index.html")
    favicondata, _ = f.ReadFile("statics/favicon.ico")
}
```

## 4、嵌入相对路径文件夹路径

- 相对路径的根路径是go源文件所在的文件夹

- 支持使用双引号`"`或者反引号的方式应用到嵌入的文件名或者文件夹名或者模式名上

```go
//go:embed "te st1.txt" `test-1.txt`
var f embed.FS
func main() {
    data, _ := f.ReadFile("te st1.txt")
    fmt.Println(string(data))
}
```

## 5、匹配模式嵌入

```go
//go:embed statics/*
var f embed.FS
func main() {
    indexdata, _ := f.ReadFile("statics/index.html")
    favicondata, _ = f.ReadFile("statics/js/jquery.js")
}
```

不支持绝对路径、不支持路径中包含`.`和`..`。如果想嵌入go源文件所在的路径，使用`*`

```go
//go:embed *
var f embed.FS
func main() {
    indexdata, _ := f.ReadFile("statics/index.html")
    jquerydata, _ = f.ReadFile("statics/js/jquery.js")
}
```

# 三、示例

```go
package main

import (
    "embed"
    "fmt"
    "github.com/gin-gonic/gin"
    "net/http"
    "path"
)

// 创建一个全局的 Gin 引擎实例
var router = gin.Default()

// 使用 embed.FS 类型声明一个变量 f，用于指定要嵌入的文件或目录。目的是将静态文件或目录嵌入到可执行文件中
//
//go:embed statics/*
var f embed.FS

// serveEmbeddedFile 从嵌入文件系统中读取文件并响应客户端
func serveEmbeddedFile(context *gin.Context, relativePath string) {
    filePath := path.Join("statics", relativePath)
    if fileContent, err := f.ReadFile(filePath); err == nil {
        // 根据文件类型设置响应头
        context.Data(http.StatusOK, http.DetectContentType(fileContent), fileContent)
    } else {
        context.String(http.StatusInternalServerError, fmt.Sprintf("Error reading file %s", relativePath))
    }
}

func main() {
    // 处理根路径的请求，返回嵌入的 index.html 文件内容
    router.GET("/", func(context *gin.Context) {
        serveEmbeddedFile(context, "index.html")
    })

    // 处理 /js/:filename 路径的请求，返回嵌入的 JavaScript 文件内容
    router.GET("/js/:filename", func(context *gin.Context) {
        filename := context.Param("filename")
        serveEmbeddedFile(context, path.Join("js", filename))
    })

    // 启动 Gin 服务器，监听端口 9091
    if err := router.Run(":9091"); err != nil {
        fmt.Println("Error starting server:", err)
    }
}

```

# 参考

- https://colobu.com/2021/01/17/go-embed-tutorial/
