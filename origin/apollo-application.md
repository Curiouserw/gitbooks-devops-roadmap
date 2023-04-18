# 应用集成Apollo

# 一、Java

# 二、PHP

# 三、Golang

**Apollo Go SDK**: https://github.com/apolloconfig/agollo

**Apollo Go SDK文档**：https://pkg.go.dev/github.com/zouyx/agollo?utm_source=godoc

设置系统环境变量

```bash
export APP_NAME=test-app
export APP_ENV=dev
export APOLLO_ADDR=http://127.0.0.1:8080
export APP_NS=application
export APOLLO_SECRET=
```



```go
package main

import (
	"fmt"
	"github.com/apolloconfig/agollo/v4"
	"github.com/apolloconfig/agollo/v4/env/config"
)

var (
	key1 string
	key2 string
	appConfig = &config.AppConfig{
		AppID:          os.Getenv("APP_NAME"),
		Cluster:        os.Getenv("APP_ENV"),
		IP:             os.Getenv("APOLLO_ADDR"),
		NamespaceName:  os.Getenv("APP_NS"),
		IsBackupConfig: true,
		Secret:         os.Getenv("APOLLO_SECRET"),
	}
)

func init() {
	client, _ := agollo.StartWithConfig(func() (*config.AppConfig, error) {
		return appConfig, nil
	})
	fmt.Println("初始化Apollo配置成功")

	key1 = client.GetStringValue("key1", "")
	key2 = client.GetStringValue("key2", "")
}

func main() {
	fmt.Println("main")
	fmt.Println(key1)
	fmt.Println(key2)
}
```
