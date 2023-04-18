# Filebeat常用配置及问题总结

# 一、常用配置

## 1、采集时添加字段和标签

```bash
filebeat.inputs:
- type: log
  paths:
  - /var/log/nginx/nginx-access.log
  exclude_files: ["_filebeat", ".gz$"]
  # 添加自定义的字段
  fields:
    env: test
  # 如果该选项设置为true，则新增fields成为顶级字段，而不是将其放在fields目录下。
  # 而且自定义的field会覆盖filebeat默认的field
  fields_under_root: true
  # 在tags字段中添加自定义的属性
  tags:
    nginx-access-log
```



## 2、删除自动添加的无用字段



```bash
processors:
- drop_fields:
    fields: ["agent", "input", "ecs"]
```



# 二、问题总结