# 一. 字符串的截取

| 表达式                                    | 含义                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| `${#string}`                              | $string的字符个数                                            |
| `${string:position}`                      | 在\$string中, 从位置$position开始提取子串                    |
| `${string:position:length}`               | 在\$string中, 从位置$position开始提取长度为$length的子串     |
| `${string#substring}`                     | 从 变量\$string的开头, 删除最短匹配$substring的子串          |
| `${string##substring}`                    | 从 变量\$string的开头, 删除最长匹配$substring的子串          |
| `${string%substring}`                     | 从 变量\$string的结尾, 删除最短匹配$substring的子串          |
| `${string%%substring}`                    | 从 变量\$string的结尾, 删除最长匹配$substring的子串          |
| `${string/substring/replacement}`         | 使用\$replacement, 来代替第一个匹配的$substring              |
| `${string//substring/replacement}`        | 使用\$replacement, 代替所有匹配的$substring                  |
| `${string/#substring/replacement}`        | 如果\$string的前缀匹配$substring, 那么就用$replacement来代替匹配到的$substring |
| `${string/%substring/replacement}`        | 如果\$string的后缀匹配$substring, 那么就用$replacement来代替匹配到的$substring |
| `expr match "$string" '$substring'`       | 匹配\$string开头的$substring* 的长度                         |
| `expr "$string" : '$substring'`           | 匹 配\$string开头的$substring* 的长度                        |
| `expr index "$string" $substring`         | 在\$string中匹配到的$substring的第一个字符出现的位置         |
| `expr substr $string $position $length`   | 在\$string中 从位置$position开始提取长度为$length的子串      |
| `expr match "$string" '\($substring\)'`   | 从\$string的 开头位置提取$substring*                         |
| `expr "$string" : '\($substring\)'`       | 从\$string的 开头位置提取$substring*                         |
| `expr match "$string" '.*\($substring\)'` | 从\$string的 结尾提取$substring*                             |
| `expr "$string" : '.*\($substring\)'`     | 从\$string的 结尾提取\$substring*                            |



## 1. \# 号从左边开始，删除第一次匹配到条件的左边字符，保留右边字符

```bash
# 样本: a="docker.io/openshift/origin-metrics-cassandra:v3.9"
b=${a#*/};echo $b
# 结果：openshift/origin-metrics-cassandra:v3.9
```

## 2. \## 号从左边开始，删除最后一次匹配到条件的左边字符，保留右边字符

```bash
# 样本: a="docker.io/openshift/origin-metrics-cassandra:v3.9"
b=${a##*/};echo $b
# 结果：origin-metrics-cassandra:v3.9
```

## 3. %号从右边开始，删除第一次匹配到条件的右边内容，保留左边字符（不保留匹配条件）

```bash
# 样本: a="docker.io/openshift/origin-metrics-cassandra:v3.9"   
b=${a%/*};echo $b
# 结果：docker.io/openshift
```

## 4. %% 号从右边开始，删除最后一次匹配到条件的右边内容，保留左边字符（不保留匹配条件）

```bash
# 样本: a="docker.io/openshift/origin-metrics-cassandra:v3.9"   
b=${a%%/*};echo $b
# 结果：docker.io
```

## 5. 从左边第几个字符开始，及字符的个数

```bash 
# 样本: a="docker.io/openshift/origin-metrics-cassandra:v3.9"   
b=${a:0:5};echo $b
# 结果：docke
```

## 6. 从左边第几个字符开始，一直到结束

```bash
# 样本: a="docker.io/openshift/origin-metrics-cassandra:v3.9"   
b=${a:7};echo $b
# 结果：io/openshift/origin-metrics-cassandra:v3.9
```

## 7. 从右边第几个字符开始，向右截取 length 个字符。

```bash
# 样本: a="docker.io/openshift/origin-metrics-cassandra:v3.9"   
b=${a:0-8:5};echo $b
# 结果：dra:v
```

## 8. 从右边第几个字符开始，一直到结束

```bash
# 样本: a="docker.io/openshift/origin-metrics-cassandra:v3.9"   
b=${a:0-8};echo $b
# 结果：dra:v3.9
```

## 9. 截取字符串中的ip

```bash
# 样本: a="当前 IP：123.456.789.172  来自于：中国 上海 上海  联通"
b=${a//[!0-9.]/};echo $b

或者
echo $a | grep -o -E "[0-9]+\.[0-9]+\.[0-9]+\.[0-9]*"

# 结果：123.456.789.172
```

## 10.提取字符串中的数字

```bash
echo "test-v1.1.0" | tr -cd '[0-9.]'    # 输出1.1.0
aa="test-v1.1.0" | echo ${aa//[!0-9.]/} # 输出1.1.0
```

