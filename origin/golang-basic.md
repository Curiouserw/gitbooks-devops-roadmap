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

- 函数内定义的变量称为局部变量
- 函数外定义的变量称为全局变量
- 函数定义中的变量称为形式参数

# 三、struct与json之间的互转

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

# 五、数组的遍历

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

## 方式一

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

## 方式二

```bash
var (
	key1 string
	key2 string
	appConfig = &config.AppConfig{
		AppID:          os.Getenv("APP_NAME"),
		Cluster:        os.Getenv("APP_ENV"),
	}
)
```
