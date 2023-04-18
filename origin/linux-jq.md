# Linux下JSON文本处理工具jq
# 一、Overviews

jq 是一款命令行下处理 JSON 数据的工具。其可以接收标准输入，命令管道或者文件中的 JSON 数据，经过一系列的过滤器(filters)和表达式的转后形成我们需要的数据结构并将结果输出到标准输出中。jq 的这种特性使我们可以很容易地在 Shell 脚本中调用它。

文档说明：https://stedolan.github.io/jq/manual/

工具网站：https://jqplay.org/

# 二、安装

```bash
yum install -y epel-release ;\
yum install -y jq
```

# 三、jq命令参数

```bash
jq [options] <jq filter> [file...]

options:
-c    使输出紧凑，而不是把每一个JSON对象输出在一行。;
-n    不读取任何输入，过滤器运行使用null作为输入。一般用作从头构建JSON数据。;
-e    set the exit status code based on the output;
-s    读入整个输入流到一个数组(支持过滤);
-r    如果过滤的结果是一个字符串，那么直接写到标准输出（去掉字符串的引号）;
-R    read raw strings, not JSON texts;
-C    打开颜色显示;
-M    关闭颜色显示;
-S    sort keys of objects on output;
--tab    use tabs for indentation;
--arg a v    jq 通过该选项提供了和宿主脚本语言交互的能力。该选项将值(v)绑定到一个变量(a)上。在后面的 filter 中可以直接通过变量引用这个值。例如，filter '.$a'表示查询属性名称等于变量 a 的值的属性。;
  --argjson a v  set variable $a to JSON value <v>;
  --slurpfile a f set variable $a to an array of JSON texts read from <f>;
```

# 四、示例

## 样本数据

```json
{"snapshots": [{"snapshot": "AAA-api-frame-2019-02-08-2019.02.24","uuid": "aMhlAmhrqng","version_id": 7050100,"version": "7.5.1","indices": ["AAA-api-frame-2019-02-08"],"include_global_state": false,"state": "SUCCESS","start_time": "2019-02-24T08:53:01.193Z","end_time": "2019-02-24T08:53:01.593Z","duration_in_millis": 402,"failures": [],"shards": {"total": 1,"failed": 0,"successful": 1}},{"snapshot": "BBB-api-frame-2019-02-09-2019.02.24","uuid": "Wp5MBOFWJA","version_id": 7050199,"version": "7.5.1","indices": ["BBB-api-frame-2019-0209"],"include_global_state": false,"state": "SUCCESS","start_time": "2019-02-24T11:04:56.063Z","end_time": "2020-02-24T11:04:56.463Z","duration_in_millis": 399,"failures": [],"test": "hah","shards": {"total": 1,"failed": 0,"successful": 1}}]}
```

```json
{
  "snapshots": [
    {
      "snapshot": "AAA-api-frame-2019-02-08-2019.02.24",
      "uuid": "aMhlAmhrqng",
      "version_id": 7050100,
      "version": "7.5.1",
      "indices": [
        "AAA-api-frame-2019-02-08"
      ],
      "include_global_state": false,
      "state": "SUCCESS",
      "start_time": "2019-02-24T08:53:01.193Z",
      "end_time": "2019-02-24T08:53:01.593Z",
      "duration_in_millis": 402,
      "failures": [],
      "shards": {
        "total": 1,
        "failed": 0,
        "successful": 1
      }
    },
    {
      "snapshot": "BBB-api-frame-2019-02-09-2019.02.24",
      "uuid": "Wp5MBOFWJA",
      "version_id": 7050199,
      "version": "7.5.1",
      "indices": [
        "BBB-api-frame-2019-0209"
      ],
      "include_global_state": false,
      "state": "SUCCESS",
      "start_time": "2019-02-24T11:04:56.063Z",
      "end_time": "2020-02-24T11:04:56.463Z",
      "duration_in_millis": 399,
      "failures": [],
      "test": "hah",
      "shards": {
        "total": 1,
        "failed": 0,
        "successful": 1
      }
    }
  ]
}
```

## 1、输出控制

### ①美化输出

```bash
$ jq -r '.' test.json
```

### ②换行与不换行输出

示例数据

```json
[{"id": 16176,"iid": 7},{"id": 16173,"iid": 4}]
```

默认遍历数组中一个对象属性时会换行显示

```bash
$ cat test.json | jq -r '.[] | .id , .iid'
16176
7
16173
4
```

jq 加`-j`参数，可不换行输出

```bash
$ cat test.json | jq -jr '.[] | .id , .iid'
161767161734
```

或者

```bash
$ cat test.json | jq -r '.[] | "\(.id) , \(.iid)"'
16176 , 7
16173 , 4
```

### ③输出额外信息

示例数据还使用上一个

````bash
$ cat test.json | jq -r '.[] | " id: \(.id) , iid: \(.iid)"'
 id: 16176 , iid: 7
 id: 16173 , iid: 4

$ cat test.json | jq -jr '.[] | " \"" , "IID: " , .iid , " ID: " , .id ,"\""  '
 "IID: 7 ID: 16176" "IID: 4 ID: 16173"
````

### ④格式化输出

```bash
$ cat test.json | jq -r '[.id,.iid] as [$id,$iid] | "\($id) -|- \($iid)"'
1 -|- 11
22 -|- 21
```

### ⑤以Key=value的形式输出

```bash
jq -r '.snapshots[].shards|to_entries[]|"\(.key | ascii_upcase)=\(.value)"' test.json
TOTAL=1
FAILED=0
SUCCESSFUL=1
```

### ⑥压缩输出

```bash
jq -c '.' test.json
```

## 2、访问属性值

### ①输出属性的值

```bash
$ jq -r '.snapshots[].snapshot' test.json

$ jq -r '.snapshots[].snapshot,.snapshots[].end_time' test.json

# 如果属性名中有空格，需要加双引号
$ jq -r '."with space"' test.json
$ jq -r '"\(."@timestamp") \(.end_time)"'  test.json
```

### ②批量访问属性值

```bash
$ jq -r '.snapshots[] | [.snapshot,.end_time] test.json
```

## 3、操作属性值

### ①取值赋予变量

```bash
$ cat test.json | jq -r '[.id,.iid] as [$id,$iid] | "\($id)|\($iid)"'
```

## 4、JSON数组的操作

### ①遍历访问数组

```bash
$ jq -r '.snapshots[]' test.json

$ jq -r '.snapshots[] | .snapshot' test.json

$ jq -r '.snapshots[].snapshot' test.json
```

### ②按索引访问数组

获取snapshot的index

```bash
$ jq -r '.snapshots[0]' test.json

$ jq -r '.snapshots[1].indices[0]' test.json
```

### ③数组切片

只取数组指定位置的值

```bash
# 从0开始到第一个
$ jq -r '.snapshots[0:1]' test.json

# 从头开始到第一个
$ jq -r '.snapshots[:1]' test.json

# 倒数一个到最后一个
$ jq -r '.snapshots[-1:]' test.json
```

## 5、应用其他环境变量

```bash
my_var="hello world"
jq -r --arg my_var "$my_var" '.foo | "\($my_var) \(.bar)"' input.json
```

## 6、函数操作

### ①keys：获取有哪些属性

```bash
$ jq -r '.snapshots[] | keys' test.json
```

### ②length：获取属性的个数

```bash
$ jq -r '.snapshots[] | length' test.json
```

### ③min/max：大小的比较

```bash
$ jq -r '[.snapshots[].indices[0]] | min' test.json

$ jq -r '[.snapshots[].indices[0]] | max' test.json
```

### ④ select：筛选过滤

```bash
$ jq -r '.snapshots[] | select(.duration_in_millis < 400)' test.json

# 如果属性值不是整型，而是字符型。例如 duration_in_millis: "300"
$ jq -r '.snapshots[] | select(.duration_in_millis < "400") | .end_time , .version_id' test.json
$ jq -r '.snapshots[] | select(.duration_in_millis < "400") | "\(.end_time) \(.version_id)"' test.json

$ jq -r '.snapshots[] | select(.duration_in_millis < 400 and .state=="SUCCESS" )' test.json

# 精准匹配
$ jq -r ' .snapshots[] | .[] | select( .uuid == "Wp5MBOFWJA" )  ' test.json
# 精准匹配：反向选择
$ jq -r ' .snapshots[] | .[] | select( .uuid | . != "Wp5MBOFWJA") | "\($id)|\($iid)"' test.json

# 模糊匹配
jq '.[] |= map(select(.source | contains("/log/app/stg/pressure-server/"))' filebeat-registry.json
```

### ⑤select：正则表达式筛选过滤

```bash
$ jq -r '.snapshots[] | select(.snapshot|test("^BBB.*") ) | .version_id' test.json
```

### ⑥del：删除属性

```bash
$ jq -r 'del(.snapshots[].version)' test.json

# 删除匹配到的属性
$ jq -r 'del( .snapshots[] | select(.uuid == "Wp5MBOFWJA"))'  test.json > test-deled.json
```

### ⑦map：map属性值进行操作

判断属性值是否存在

```bash
$ jq -r '.snapshots | map(has("snapshot"))' test.json
```

操作数值类型的属性值

```bash
$ jq -r '.snapshots | map(.duration_in_millis+2)' test.json
```

### ⑧unique：去重属性值

```bash
$ jq -r '.snapshots | map(.state) | unique' test.json
```

### ⑨重组json结构

```bash
$ jq -r '.snapshots | map(.) | .[] | {"快照名": .snapshot,"快照的索引": .indices}' test.json

{
  "快照名": "AAA-api-frame-2019-02-08-2019.02.24",
  "快照的索引": [
    "AAA-api-frame-2019-02-08"
  ]
}
{
  "快照名": "BBB-api-frame-2019-02-09-2019.02.24",
  "快照的索引": [
    "BBB-api-frame-2019-0209"
  ]
}
```

### ⑩if else 函数判断

```bash
$ jq -r ' .snapshots[] 
| if .snapshot == "" then .value |= "值为空" else . end
| "\(.id)= \(.iid)" '
```



# 参考链接

1. https://www.baeldung.com/linux/jq-command-json
2. https://www.ibm.com/developerworks/cn/linux/1612_chengg_jq/index.html?ca=drs-&utm_source=tuicool&utm_medium=referral
3. https://stackoverflow.com/questions/56692037/filter-empty-and-or-null-values-with-jq
3. https://justcode.ikeepstudying.com/2018/02/shell%EF%BC%9A%E6%97%A0%E6%AF%94%E5%BC%BA%E5%A4%A7%E7%9A%84shell%E4%B9%8Bjson%E8%A7%A3%E6%9E%90%E5%B7%A5%E5%85%B7jq-linux%E5%91%BD%E4%BB%A4%E8%A1%8C%E8%A7%A3%E6%9E%90json-jq%E8%A7%A3%E6%9E%90-json/