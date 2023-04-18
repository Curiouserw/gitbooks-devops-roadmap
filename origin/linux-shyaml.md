# Linux下YAML文本处理工具shyaml
# 一、Overviews

通过 shyaml，可以直接获取键、值、键值对或对应的类型

# 二、安装

    pip install shyaml

# 三、语法

```bash
cat <file.yaml> | shyaml ACTION KEY [DEFAULT]
```

**ACTION**

- get-type：获取相应的类型
- get-value：获取值
- get-values{,-0}：对序列类型来说，获取值列表
- keys{,-0}：返回键列表
- values{,-0}：返回值列表
- key-values,{,-0}：返回键值对

**Note：**

1. 结果默认是加`\n`换行符，若用`-0`形式则以`NUL`字符填充
2. `KEY`为要查询的键，如不提供，则使用`DEFAULT`

# 四、示例

```yaml
---
idc_group:
  name: bx
  bx:
    news_bx: news_bx
    web3_bx: web3_php-fpm_bx
```

如果要获取`idc_group.name`的值则可以执行

```bash
cat file.yaml | shyaml get-value idc_group.name
```

想获取`idc_group.bx`的键值对可执行

```bash
cat file.yaml | shyaml key-values idc_group.bx
```

# 参考链接

1. https://www.linuxidc.com/Linux/2016-04/130403.htm
2. https://dev.to/vikcodes/yq-a-command-line-tool-that-will-help-you-handle-your-yaml-resources-better-8j9
