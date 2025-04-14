# 常用工具

# 一、时间的处理

## 1、按照指定格式输出

```go
println(time.Now().Format("2006-01-02 15:04:05"))
```

## 2、输出毫秒级别时间戳

```go
cts := fmt.Sprintf("%d", time.Now().UnixNano()/1e6)
```

# 二、字符串的处理

## 1、分割字符串

**多分割符分割字符串**

```bash
目标字符串 := strings.FieldsFunc("待处理字符串", func(r rune) bool {
		return r == '/' || r == '?'
	})[3]
```

**单个分隔符分割字符串**

```bash
str="aaaaaaaa\r\nBBBBBBBBBBB\r\n1111"
目标字符串 := strings.Split(string(res[0:len(res)]), "\r\n")
# 目标字符串类型为字符串数组
```

## 2、判断字符串前缀是否包含指定字符

```go
str="aaaaaaaa\r\nBBBBBBBBBBB\r\n1111"
res := strings.HasPrefix(s, "aaa")
// res为布尔值
```

# 三、命令行参数

```go
import ("flag" )
var (
	omhost   string
	omport   string
	ompasswd string
)

func init (){
  flag.StringVar(&omhost, "host", "", "OpenVPN服务端地址")
	flag.StringVar(&omport, "port", "", "OpenVPN服务端管理端口，默认为空")
	flag.StringVar(&ompasswd, "passwd", "", "OpenVPN服务端管理端口密码")
	flag.Parse()
}

func main(){
	if omhost == "" && omport == "" {
		fmt.Println("请在启动命令后添加'-host','-port'参数设置")
		os.Exit(0)
	} else if omhost == "" {
		fmt.Println("请在启动命令后添加'-host'参数设置IP地址")
		os.Exit(0)
	} else if omport == "" {
		fmt.Println("请在启动命令后添加'-port'参数设置管理端口号")
		os.Exit(0)
	}
}
```

# 四、本地文件操作

## 1、判断文件是否存在

```go
func PathExists(path string) (bool, error) {
	/*
	   判断文件或文件夹是否存在
	   如果返回的错误为nil,说明文件或文件夹存在
	   如果返回的错误类型使用os.IsNotExist()判断为true,说明文件或文件夹不存在
	   如果返回的错误为其它类型,则不确定是否在存在
	*/
	_, err := os.Stat(path)
	if err == nil {
		return true, nil
	}
	if os.IsNotExist(err) {
		return false, nil
	}
	return false, err
}
func main{
	isFileExist, err := PathExists("文件路径")
	if isFileExist == true && err == nil {
		// 文件存在时toDo
  }else{
    // 文件不存在时toDo
  }
```

# 五、下载操作

## 1、文件下载

```go
func main{
	resp, err := http.Get("文件URL")
	if err != nil {
		return err,""
	}
	defer resp.Body.Close()
	out, err := os.Create("文件本地存储路径")
	if err != nil {
		return err, ""
	}
	defer out.Close()
	// 然后将响应流和文件流对接起来
	_, err = io.Copy(out, resp.Body)
	if err != nil {
		return err
	}else {
		return nil
	}
}
```

# 六、钉钉验签机器人通知

```go
func sign(secret string,cts string) string {
	sign := fmt.Sprintf("%s\n%s", cts, secret)
	signData := computeHmacSha256(sign, secret)
	encodeURL := url.QueryEscape(signData)
	return fmt.Sprintf("&timestamp=%s&sign=%s", cts, encodeURL)
}

func computeHmacSha256(message string, secret string) string {
	key := []byte(secret)
	h := hmac.New(sha256.New, key)
	h.Write([]byte(message))
	return base64.StdEncoding.EncodeToString(h.Sum(nil))
}

func DingDingNotify(msg string)  {
	cts := fmt.Sprintf("%d", time.Now().UnixNano()/1e6)
	var dingDingRebotToken string="自定义机器人的Secret"
	dsign:=genSignedURL("自定义机器人验签设置的秘钥",cts)
	dd_robot_api_addr ,err :=url.Parse("https://oapi.dingtalk.com/robot/send?access_token="+dingDingRebotToken+"&timestamp="+cts+"&sign=" + dsign)
	if err != nil{
		panic(err)
	}
	ddmsgtmp := `{"at": {"isAtAll": true}, "text": {"content":"`+msg+`"},"msgtype":"text"}`
	res, err := http.Post(dd_robot_api_addr.String(), "application/json",bytes.NewReader([]byte(ddmsgtmp)))
	if err !=nil{
		panic(err)
		println(time.Now().Format("2006-01-02 15:04:05")+" 钉钉通知消息发送失败，状态码为："+res.Status)
	}else {
		println(time.Now().Format("2006-01-02 15:04:05")+" 钉钉通知消息已发送，状态码为："+res.Status)
	}
}
```

# 七、编译优化

## 1、删除调试符

默认情况下go编译出的程序在运行出错时会输出自己在哪个线程哪个文件哪个函数哪行出的错。这些信息属于DWARF信息。在编译二进制文件时，添加参数可以去除掉。（go > 1.7）

```bash
go build -ldflags '-s -w' main.go

# -s disable symbol table
# -w disable DWARF generation
```

## 2、混淆编译运行时的环境变量

```bash
export GO111MODULE=on
export GOROOT="/usr/local/go"
export ACTUAL_GOPATH="实际GOPAH"
export TMP_GOPATH="/tmp/go"
export GOBIN="$GOROOT/bin"
export CGO_LDFLAGS="-g -O2"
[ -z ${GOPATH+x} ] && [ -h $TMP_GOPATH ] && ln -s $ACTUAL_GOPATH $TMP_GOPATH && export GOPATH="/tmp/go"
export GOENV=$TMP_GOPATH/env
export GOCACHE=$TMP_GOPATH/caches
[[ ! $PATH =~ $GOPATH ]] && export PATH=$PATH:$GOROOT/bin
```

## 3、使用upx压缩二进制包

```bash
# 安装
brew instal upx 
yum install upx -y

# 压缩
upx --brute 二进制文件
# --brute 尝试所有压缩算法
```

# 八、Debug工具

目前 golang生态圈里的主流调试工具有 GDB、LLDB 和 Delve 几种调试器。其中 GDB 是最早支持的调试工具，LLDB 是 macOS 系统推荐的标准调试工具。但是 GDB 和 LLDB 对 Go 语言的专有特性都缺乏很大支持，例如gdb 只能做到最基本的变量打印，却理解不了 golang 的一些特殊类型，比如 channel，map，slice 等，gdb 原生是无法调适 goroutine 协程的， 因为这是用户态的调度单位，gdb 只能理解线程，所以只能通过 python 脚本（src/runtime/runtime-gdb.py）的扩展，把协程结构按照链表输出。而只有 Delve 是专门为 Go 语言设计开发的调试工具。而且 Delve 本身也是采用 Go 语言开发，对 Windows 平台也提供了一样的支持。

```bash
sudo go install github.com/go-delve/delve/cmd/dlv@latest 
dlv verison

```
