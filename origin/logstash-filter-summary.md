# Logstash常用filter实现的功能

# 1、截取带有文件路径字段中的文件名

```bash
filter{
 grok {
    match => {
      "[log][file][path]" => "%{GREEDYDATA}/%{GREEDYDATA:app}-access.log"
    }
  }
}
```

# 2、删除json字段

```bash
filter{
  mutate {
      remove_field => [ "@timestamp" , "headers" , "response.data"]
      gsub => ["message", "\\\", ""]
    }
}
```

