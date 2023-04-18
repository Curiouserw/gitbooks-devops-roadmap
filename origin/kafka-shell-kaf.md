# kafka命令行工具kaf

# 一、简介

如果是使用kafka原生bin目录下的二进制命令的话，每一次命令要打好多参数，参数还无法自动补全，甚是麻烦，而GitHub中有个项目，可以像docker、kubectl命令一样，快速操作kafka。

GitHub地址：https://github.com/birdayz/kaf

# 二、安装配置

## 1、安装

### **Go**

```bash
go get -u github.com/birdayz/kaf/cmd/kaf
```

### **二进制**

直接在GitHub的releases页面下载对应操作系统的二进制文件到可执行路径下

### MacOS

```bash
brew tap birdayz/kaf
brew install kaf
```

## 2、配置

### ①命令行参数

```bash
Kafka Command Line utility for cluster management

Usage:
  kaf [command]

Available Commands:
  completion  Generate bash completion script for bash or zsh
  config      Handle kaf configuration
  consume     Consume messages
  group       Display information about consumer groups.
  groups      List groups
  help        Help about any command
  node        Describe and List nodes
  nodes       List nodes in a cluster
  produce     Produce record. Reads data from stdin.
  query       Query topic by key
  topic       Create and describe topics.
  topics      List topics

Flags:
  -b, --brokers strings          Comma separated list of broker ip:port pairs
  -c, --cluster string           set a temporary current cluster
      --config string            config file (default is $HOME/.kaf/config)
  -h, --help                     help for kaf
      --schema-registry string   URL to a Confluent schema registry. Used for attempting to decode Avro-encoded messages
  -v, --verbose                  Whether to turn on sarama logging

Use "kaf [command] --help" for more information about a command.
```

### ②命令行补全

**Bash Linux**

```
kaf completion bash > /etc/bash_completion.d/kaf
```

**Bash MacOS**

```
kaf completion bash > /usr/local/etc/bash_completion.d/kaf
```

**Zsh**

```
kaf completion zsh > "${fpath[1]}/_kaf"
```

**Fish**

```
kaf completion fish > ~/.config/fish/completions/kaf.fish
```

**Powershell**

```
Invoke-Expression (@(kaf completion powershell) -replace " ''\)$"," ' ')" -join "`n")
```

## 3、使用

### ①配置kafka连接

```bash
kaf config add-cluster local -b localhost:9092
```

连接配置会写在`~/.kaf/config`文件中

### ②选择对应的kafka连接

```bash
kaf config select-cluster
```

### ③列出kafka broker节点的详细信息

```
kaf node ls
```

### ④列出所有的Topic及其分区、副本信息

```
kaf topics
```

### ⑤列出指定Topic的详细信息

```
kaf topics describe test_topic
```

### ⑥列出所有的消费者组

```
kaf groups
```

### ⑦列出指定消费者组的详细信息

```
kafa group describe dispatcher
```

### ⑧从标准输入写消息到指定Topic

```
echo test | kaf produce test_topic
```

### ⑨消费指定Topic中的消息

```bash
kaf consume test_topic -f
```

