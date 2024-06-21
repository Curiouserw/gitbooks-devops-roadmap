# 一、推荐目录结构

```bash
/main.go 入口文件
/config 配置
/router 路由
  |—— api.go
  |-- web.go
/handler 请求处理器
  |-- user.go
/middleware 中间件
  |-- jwtToken.go
/model 模型
  |-- user.go
/test 测试
```

# 二、变量声明

## 1、变量范围

- 函数内定义的变量称为局部变量
- 函数外定义的变量称为全局变量
- 函数定义中的变量称为形式参数

## 2、可导出与不可导出变量规则

变量的可见性由其标识符的首字母是否大写来确定。

- **Exported（可导出）变量：** 如果变量的标识符（名字）以大写字母开头，那么它是可导出的，可以在其他包中访问。可导出的变量是包外可见的。

- **Unexported（不可导出）变量：** 如果变量的标识符以小写字母开头，那么它是不可导出的，只能在当前包中访问。不可导出的变量对于其他包是不可见的。

可见性规则适用于所有的标识符，不仅仅是变量，还包括函数、类型等。这个规则有助于封装和隐藏内部实现，使得包的使用更加清晰和可维护。

在同一个包内，无论变量是否可导出，都可以直接访问。但在不同的包中，只能访问可导出的变量。

# 三、类型转换

## struct与json之间的互转

```go
type Test struct {
  A string `json:"a`
  B int `json:"b"`
}
```

## struct转json

```go
var t Test
t.A="测试"
t.B=2

jsonBytes, err := json.Marshal(t)
if err != nil {
	fmt.Println(err)
}
fmt.Println(string(jsonBytes))
```

## json转struct

```bash
jsonStr := `{
					"a": "测试",
          "b": 2
        }`
var t Test
json.Unmarshal([]byte(jsonStr), &t)
fmt.Println(t)
```

## 整型转 string

```go
var number int = 80
strconv.Itoa(number)
```

## bytes转 string

```go
var b byte
string(b)
```



# 四、主函数的初始化

```go
var (
	router *gin.Engine
)

func init (){
  router = gin.Default()
}

func main(){
  router.GET("/",func(context *gin.Context) {})
}
```

# 五、数组

## ①遍历

```go
var test_array = [5]float32{1000.0, 2.0, 3.4, 7.0, 50.0}
for _,a := range test_array {
	fmt.Println(a)
}

var test_array = [3]string{"test1","test2"}
for _,a := range test_array {
	fmt.Println(a)
}
```



# 六、**golang程序初始化**

golang程序初始化先于main函数执行，由runtime进行初始化，初始化顺序如下：

1. 初始化导入的包（包的初始化顺序并不是按导入顺序（“从上到下”）执行的，runtime需要解析包依赖关系，没有依赖的包最先初始化，与变量初始化依赖关系类似，参见[golang变量的初始化](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s/PGDzMaYznZVuDiO6V-zYDw)）；
2. 初始化包作用域的变量（该作用域的变量的初始化也并非按照“从上到下、从左到右”的顺序，runtime解析变量依赖关系，没有依赖的变量最先初始化，参见[golang变量的初始化](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s/PGDzMaYznZVuDiO6V-zYDw)）；
3. 执行包的init函数；

变量初始化->init()->main()



# 七、获取系统环境变量

## 方式一：env

```go
type config struct {
    App     string
    Port    int      `default:"8000"`
    IsDebug bool     `env:"DEBUG"`
    Hosts   []string `slice_sep:","`
    Timeout time.Duration
    Redis struct {
        Version string `sep:""` // no sep between `CONFIG` and `REDIS`
        Host    string
        Port    int
    }
    MySQL struct {
        Version string `default:"5.7"`
        Host    string
        Port    int
    }
}
```

## 方式二：os.Getenv

```bash
var (
	key1 string
	key2 string
	appConfig = &config.AppConfig{
		AppID:          os.Getenv("APP_NAME"),
		Cluster:        os.Getenv("APP_ENV"),
	}
)

ausername="test"
awk '!/^;/&&!/^#/&&$1=="'${ausername}'"{print $2;exit}' psw-file
```

# 八、读取json格式配置文件

> config.json

```json
{
  "host": "192.168.1.1",
  "ports": {
    "22": "ssh",
    "3306": "mysql",
    "6379": "redis",
    "9091": "kafka"
  }
}
```

> main.go

```go
package main

import (
	"encoding/json"
	"fmt"
	"os"
)

// AppConfig 结构体定义了应用程序的配置结构
type AppConfig struct {
	Host     string            `json:"host"`
	PortsMap map[string]string `json:"ports"`
}
// 全局变量，存储应用程序配置
var appconfig = AppConfig{}

func main() {
  // 读取配置文件内容
	configfileData, err := os.ReadFile("config.json")
	if err != nil {
		fmt.Println("当前目录下未发现配置文件config.json，本次启动将使用默认值。可创建配置，格式为json")
	} else {
		// 解析 JSON
		err = json.Unmarshal(configfileData, &appconfig)
		if err != nil {
			fmt.Println("当前目录下的config.json无法解析，本次启动不使用其中设置。格式为json，请进行检查。")
		} else {
			// 检查参数是否存在
			if appconfig.Host == "" && appconfig.PortsMap == nil {
				println("当前目录下config.json没有配置host和ports。本次启动将使用默认值。格式为json，请进行检查。")
			}
			// 检查参数是否缺少
			if appconfig.Host == "" {
				println("当前目录下config.json没有配置host 。本次启动将使用默认值。格式为json，请进行检查。")
			}
			if appconfig.PortsMap == nil {
				println("当前目录下config.json没有配置ports，本次启动将使用默认值。格式为json，请进行检查。")
			}
		}
	}
  
	fmt.Printf("服务器地址: %s\n", appconfig.Host)
	for port, service := range appconfig.PortsMap {
		fmt.Printf("  %s: %s\n", service, port)
	}
}
```



# 九、其他

1. 在 Go 语言中，map 是无序的，即使按照某种顺序插入数据，也不能保证遍历时是按照插入的顺序。最终的 `map` 遍历结果仍然是无序的。

# 十、struct 嵌套

对象继承在 Golang嵌套。

```go
type Address struct {
    Street     string
    City       string
    PostalCode string
}

type Person struct {
    Name    string
    Age     int
    Address // 匿名字段
}

p := Person{
    Name: "Alice",
    Age:  30,
    Address: Address{
        Street:     "123",
        City:       "Any",
        PostalCode: "12345",
    }
}
print(json.Marshal(p))
```

# 十一、异常

```go
err.
```

