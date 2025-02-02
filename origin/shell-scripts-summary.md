# Shell脚本总结

# 一、简介

```bash
#!/bin/sh

# 脚本中的第一行的“#!”为Shebang字符串。通常出现在类Unix系统的脚本中第一行，作为前两个字符。在直接调用脚本时，系统的程序载入器会分析 Shebang 后的内容，将这些内容作为解释器指令，并调用该指令，将载有 Shebang 的文件路径作为该解释器的参数，执行脚本，从而使得脚本文件的调用方式与普通的可执行文件类似。例如，以指令#!/bin/sh开头的文件，在执行时会实际调用 /bin/sh 程序（通常是 Bourne shell 或兼容的 shell，例如 bash、dash 等）来执行。
# 如果脚本文件中没有#!这一行，那么执行时会默认采用当前Shell去解释这个脚本(即：SHELL环境变量）。
# 如果#!之后的解释程序是一个可执行文件，那么执行这个脚本时，它就会把文件名及其参数一起作为参数传给那个解释程序去执行。
# 如果#!指定的解释程序没有可执行权限，则会报错 bad interpreter: Permission denied。如果#!指定的解释程序不是一个可执行文件，那么指定的解释程序会被忽略，# 转而交给当前的SHELL去执行这个脚本。
# 如果#!指定的解释程序不存在，那么会报错 bad interpreter: No such file or directory。注意：#!之后的解释程序，需要写其绝对路径（如：#!/bin/bash），它是不会自动到环境变量PATH中寻找解释器的。要用绝对路径是因为它会调用系统调用execve，这可以用strace工具来查看。
# 脚本文件必须拥有可执行权限。可通过chmod +x [filename] 来添加可执行权限。
# 当然，如果你使用类似于 bash test.sh ，python train.py这样的命令来执行脚本，那么#!这一行将会被忽略掉，解释器当然是用命令行中显式指定的解释器。
```



# 二、变量

## 1、Shell内置变量

| 内置变量    | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| $?          | 上一条命令执行状态  0 代表执行成功，1代表执行失败            |
| \$0~\$9     | 位置参数1-9                                                  |
| \${10}      | 位置参数10                                                   |
| \$#         | 位置参数个数                                                 |
| \$$         | 脚本进程的PID                                                |
| \$-         | 传递到脚本中的标识                                           |
| \$!         | 运行在后台的最后一个作业的进程ID(PID)                        |
| \$_         | 之前命令的最后一个参数                                       |
| \$@         | 传递给脚本或函数的所有参数。被双引号(" ")包含时，与 \$* 稍有不同 |
| \$*         | 传递给脚本或函数的所有参数                                   |
| \$0         | 脚本的文件名                                                 |
| ${@: -1}    | 传递给脚本或函数的最后一个参数                               |
| ${@:1:$#-1} | 传递给脚本或函数除最后一个参数以外的所有参数                 |

**`$* 和 $@ 的区别`**

`$*` 和 `$@` 都表示传递给函数或脚本的所有参数，不被双引号(" ")包含时，都以`"$1" "$2" … "$n"` 的形式输出所有参数。但是当它们被双引号`(" ")`包含时，`"$*"` 会将所有的参数作为一个整体，以`"$1 $2 …\$n"`的形式输出所有参数；`"$@"` 会将各个参数分开，以`”\$1” "\$2" … "$n"` 的形式输出所有参数。

## 2、变量的定义、赋值

**①将命令输出赋值变量**

```bash
var=`shell命令`  # `是反引号
var=$(shell命令) 
var='
line 1
line 2
line 3
'
```

**②读取标准输入赋值给变量**

```bash
read -p "请输入一个字符： " key
echo $key
```

**③read赋值变量**

```bash
read aaa bbb <<< "11 22"

# 变量切割赋值
cc="11 22"
read aaa bbb <<< $cc
```

## 3、变量的引用

**①基础引用**

```bash
$var
${var}
${var:defaultvalue}
```

**②变量的引用默认值**

| 表达式          | 含义                                                      |
| --------------- | --------------------------------------------------------- |
| ${var_DEFAULT}  | 如果var没有被声明, 那么就以$DEFAULT作为其值               |
| ${var=DEFAULT}  | 如果var没有被声明, 那么就以$DEFAULT作为其值               |
| ${var:-DEFAULT} | 如果var没有被声明, 或者其值为空, 那么就以$DEFAULT作为其值 |
| ${var:=DEFAULT} | 如果var没有被声明, 或者其值为空, 那么就以$DEFAULT作为其值 |
| ${var+OTHER}    | 如果var声明了, 那么其值就是$OTHER, 否则就为null字符串     |
| ${var:+OTHER}   | 如 果var被设置了, 那么其值就是$OTHER, 否则就为null字符串  |
| ${var?ERR_MSG}  | 如果var没 被声明, 那么就打印$ERR_MSG                      |
| ${var:?ERR_MSG} | 如果var没 被设置, 那么就打印$ERR_MSG                      |
| ${!varprefix*}  | 匹配之前所有以varprefix开头进行声明的变量                 |
| ${!varprefix@}  | 匹配之前所有以varprefix开头进行声明的变量                 |

**③用变量值作为新变量名**

```bash
$ name=test
$ test_p=123
$ echo `eval echo '$'"$name""_p"`
123
```

或者

```bash
$ var="world"
$ declare "hello_$var=value"
$ echo $hello_world
value
```

或者（ `bash` 4.3+）

```bash
$ hello_world="value"
$ var="world"
$ declare -n ref=hello_$var
$ echo $ref
value
```

或者

```bash
$ hello_world="value"
$ var="world"
$ ref="hello_$var"
$ echo ${!ref}
value
```

参考：https://github.com/dylanaraps/pure-bash-bible#variables

```bash
name_1=aa
name_2=bbb
for i in ${!name_@} ;do  #  ${!name_@}仅限在sh、 bash中使用
    echo "\$i为当前变量名：" $i
    echo "\${!i}当前变量名的值：" ${!i}
    echo "\${i/name/name_var}可替换当前变量名中的name为name_var: " ${i/name/name_var}
done
```

## 4、变量的数值运算

①加减乘除

```bash
#样本数据
a=120
b=110

((c=$a+$b))    #结果：230
((d=$a-$b))    #结果：10
((e=$a*$b))    #结果：13200
((f=$a/$b))    #结果：1

c=$((a+b))     #结果：220
d=$((a-b))     #结果：20
e=$((a*b))     #结果：12000
f=$((a/b))     #结果：1

c=`expr a+b`        #结果：220
d=`expr $a - $b`    #结果：20
e=`expr $a \* $b`   #结果：12000
f=`expr $a / $b`    #结果：1
```

②自增

```bash
a=1

#第一种整型变量自增方式
a=$(($a+1))
echo $a

#第二种整型变量自增方式
a=$[$a+1]
echo $a

#第三种整型变量自增方式
a=`expr $a + 1`
echo $a

#第四种整型变量自增方式
let a++
echo $a

#第五种整型变量自增方式
let a+=1
echo $a

#第六种整型变量自增方式
((a++))
echo $a
```

## 5、数值变量的判断

```bash
-gt    大于，如[ $a -gt $b ]
-lt    小于，如[ $a -lt $b ]
-eq    等于，如[ $a -eq $b ]
-ne    不等于，如[ $a -ne $b ]
-ge    大于等于，如[ $a -ge $b ]
-le    小于等于 ，如 [ $a -le $b ]
<     小于(需要双括号),如:(($a < $b))
<=    小于等于(需要双括号),如:(($a <= $b))
>     大于(需要双括号),如:(($a > $b))
>=    大于等于(需要双括号),如:(($a >= $b))
```

## 6、变量的处理

①变量输出多行变一行并追加字符

```bash
$ echo $a

1

2

3

$ echo $a | tr '\n' ',’

1,2,3,
```

②位数截取

```bash
a=1110418197875

# 截去后三位,要求只取"1110418197875"

# 方式1: 数值运算
b=$((a/1000))

# 方式2：字符截取（将数值变量当成字符串来处理）
c=${a:0:-3}
```

## 7、数组

**①定义赋值**

```bash
tests=('a1a' 'b2b' 'c3c')
```

**②切割字符串为数组**

```bash
test="a,b-,d"
# bash中的 read 读取输入为数组的参数为 -a
IFS='-' read -a tests <<< "$test"
# zsh中的 read 读取输入为数组的参数为 -A
IFS='-' read -A tests <<< "$test"

# 数组索引，bash是从0开始 ，zsh中索引是从1开始。
echo "数组第一个元素: ${tests[0]} 数组第二个元素: ${tests[1]}" 

# 输出： 数组第一个元素: a,b 数组第二个元素: ,d
```

**③输出**

```bash
# 数组索引，bash是从0开始 ，zsh中索引是从1开始。
tests=('a1a' 'b2b' 'c3c')
echo ${tests[1]}  # bash输出为"b2b", zsh则输出"a1a"

# 输出，bash只显示第一个"a1a"，zsh显示所有元素
echo $tests
```

**④遍历循环**

```bash
# 切割其他变量出为数组变量
a=$(echo "a,b-,d")
IFS='-' read -A -r tests <<< "$a"  # 以逗号为分割符，切割成"a,b"和",d"作为数组变量元素

# 通用(bash/sh/zsh)的遍历循环方法。。区别是zsh中输出在一行中，sh和bash中分行输出
tests=('a1a' 'b2b' 'c3c')
for test in ${tests[@]} ; do
  echo "test: $test" 
done
```

## 8、多行文本变量

**①定义赋值、引用、输出**

```bash
tests='a1a
b2b
c3c
'

echo $tests 
# a1a b2b c3c

echo "$tests"
# a1a
# b2b
# c3c

# 总共是有四行的输出，最后一个是空行
```

**②遍历循环**

```bash
# zsh中的方法
for test in ${(f)tests}; do
  echo "测试: "$test
done

# bash中的方法
for test in ${tests}; do
  echo "测试: $test"
done
```

# 三、条件判断

## 1、条件判断的语法

- `[[ ]]` 
  - 是 Bash 的扩展条件表达式，不调用外部命令，处理速度更快。

- `[]`
  -  `test` 命令的一个别名，调用外部命令进行条件测试。
  - 语法严格，通常需要在各个参数之间留空格。
  - 支持的功能有限，许多运算符需要使用反斜杠进行转义（例如，逻辑运算符 `-o` 和 `-a`）。
  - 只支持简单的字符串、数字比较和文件测试。

## 2、文件目录的判断

| [ -a FILE ]         | 如果 FILE 存在则为真。                                       |
| ------------------- | ------------------------------------------------------------ |
| [ -b FILE ]         | 如果 FILE 存在且是一个块文件则返回为真。                     |
| [ -c FILE ]         | 如果 FILE 存在且是一个字符文件则返回为真。                   |
| [ -d FILE ]         | 如果 FILE 存在且是一个目录则返回为真。                       |
| [ -e FILE ]         | 如果 指定的文件或目录存在时返回为真。                        |
| [ -f FILE ]         | 如果 FILE 存在且是一个普通文件则返回为真。                   |
| [ -g FILE ]         | 如果 FILE 存在且设置了SGID则返回为真。                       |
| [ -h FILE ]         | 如果 FILE 存在且是一个符号符号链接文件则返回为真。（该选项在一些老系统上无效） |
| [ -k FILE ]         | 如果 FILE 存在且已经设置了冒险位则返回为真。                 |
| [ -p FILE ]         | 如果 FILE 存并且是命令管道时返回为真。                       |
| [ -r FILE ]         | 如果 FILE 存在且是可读的则返回为真。                         |
| [ -s FILE ]         | 如果 FILE 存在且大小非0时为真则返回为真。                    |
| [ -u FILE ]         | 如果 FILE 存在且设置了SUID位时返回为真。                     |
| [ -w FILE ]         | 如果 FILE 存在且是可写的则返回为真。（一个目录为了它的内容被访问必然是可执行的） |
| [ -x FILE ]         | 如果 FILE 存在且是可执行的则返回为真。                       |
| [ -O FILE ]         | 如果 FILE 存在且属有效用户ID则返回为真。                     |
| [ -G FILE ]         | 如果 FILE 存在且默认组为当前组则返回为真。（只检查系统默认组） |
| [ -L FILE ]         | 如果 FILE 存在且是一个符号连接则返回为真。                   |
| [ -N FILE ]         | 如果 FILE 存在 and has been mod如果ied since it was last read则返回为真。 |
| [ -S FILE ]         | 如果 FILE 存在且是一个套接字则返回为真。                     |
| [ FILE1 -nt FILE2 ] | 如果 FILE1 比 FILE2 新, 或者 FILE1 存在但是 FILE2 不存在则返回为真。 |
| [ FILE1 -ot FILE2 ] | 如果 FILE1 比 FILE2 老, 或者 FILE2 存在但是 FILE1 不存在则返回为真。 |
| [ FILE1 -ef FILE2 ] | 如果 FILE1 和 FILE2 指向相同的设备和节点号则返回为真。       |

## 3、命令执行状态的判断

```bash
if [ command ]; then
    符合该条件执行的语句
fi
```

```bash
if [ command ]; then
    command执行返回状态为0要执行的语句
else
    command执行返回状态为1要执行的语句
fi
```

```bash
if [ command1 ]; then
    command1执行返回状态为0要执行的语句
elif [ command2 ]; then
    command2执行返回状态为0要执行的语句
else
    command1和command2执行返回状态都为1要执行的语句
fi
```

**`PS`**: [ command ]，command前后要有空格



# 四、字符串的处理

## 1、截取

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

**①\#号从左边开始，删除第一次匹配到条件的左边字符，保留右边字符**

```bash
# 样本: a="docker.io/openshift/origin-metrics-cassandra:v3.9"
b=${a#*/};echo $b
# 结果：openshift/origin-metrics-cassandra:v3.9
```

**②\##号从左边开始，删除最后一次匹配到条件的左边字符，保留右边字符**

```bash
# 样本: a="docker.io/openshift/origin-metrics-cassandra:v3.9"
b=${a##*/};echo $b
# 结果：origin-metrics-cassandra:v3.9
```

**③%号从右边开始，删除第一次匹配到条件的右边内容，保留左边字符（不保留匹配条件）**

```bash
# 样本: a="docker.io/openshift/origin-metrics-cassandra:v3.9"   
b=${a%/*};echo $b
# 结果：docker.io/openshift
```

**④ %%号从右边开始，删除最后一次匹配到条件的右边内容，保留左边字符（不保留匹配条件）**

```bash
# 样本: a="docker.io/openshift/origin-metrics-cassandra:v3.9"   
b=${a%%/*};echo $b
# 结果：docker.io
```

**⑤从左边第几个字符开始，及字符的个数**

```bash 
# 样本: a="docker.io/openshift/origin-metrics-cassandra:v3.9"   
b=${a:0:5};echo $b
# 结果：docke
```

**⑥从左边第几个字符开始，一直到结束**

```bash
# 样本: a="docker.io/openshift/origin-metrics-cassandra:v3.9"   
b=${a:7};echo $b
# 结果：io/openshift/origin-metrics-cassandra:v3.9
```

**⑦从右边第几个字符开始，向右截取 length 个字符**

```bash
# 样本: a="docker.io/openshift/origin-metrics-cassandra:v3.9"   
b=${a:0-8:5};echo $b
# 结果：dra:v
```

**⑧从右边第几个字符开始，一直到结束**

```bash
# 样本: a="docker.io/openshift/origin-metrics-cassandra:v3.9"   
b=${a:0-8};echo $b
# 结果：dra:v3.9
```

**⑨截取字符串中的ip**

```bash
# 样本: a="当前 IP：123.456.789.172  来自于：中国 上海 上海  联通"
b=${a//[!0-9.]/};echo $b

或者
echo $a | grep -o -E "[0-9]+\.[0-9]+\.[0-9]+\.[0-9]*"
# 结果：123.456.789.172
```

**⑩提取字符串中的数字**

```bash
echo "test-v1.1.0" | tr -cd '[0-9.]'    # 输出1.1.0
aa="test-v1.1.0" | echo ${aa//[!0-9.]/} # 输出1.1.0
```

## 2、包含判断

**样本数据**

```
a="test"
b="curiouser"
c="test hahah devops"
```

**①通过grep来判断**

    if `echo $c |grep -q $a` ;then
        echo "$c" " ----包含--- " "$a"
    else
        echo "$c" " ----不包含--- " "$a"
    fi

**②字符串运算符**

    if [[ $c =~ $a ]] ;then
        echo "$c" " ----包含--- " "$a"
    else
        echo "$c" " ----不包含--- " "$a"
    fi

**③用通配符*号**

用通配符*号代替str1中非str2的部分，如果结果相等说明包含，反之不包含

    if [[ $c == *$a* ]] ;then
        echo "$c" " ----包含--- " "$a"
    else
        echo "$c" " ----不包含--- " "$a"
    fi

**④利用替换**

    if [[ ${c/$a//} == $c ]] ;then
        echo "$c" " ----不包含--- " "$a"
    else
        echo "$c" " ----包含--- " "$a"
    fi

# 五、循环

## 1、for循环

**①数字性循环**

```bash
#!/bin/bash
for((i=1;i<=10;i++));
do
    echo $(expr $i \* 3 + 1);
done
```

```bash
#!/bin/bash
for i in $(seq 1 10)
do
    echo $(expr $i \* 3 + 1);
done
```

```bash
#!/bin/bash
for i in {1..10}
do
    echo $(expr $i \* 3 + 1);
done
```

```bash
#!/bin/bash
awk 'BEGIN{for(i=1; i<=10; i++) print i}'
```

**②字符性循环**

```bash
#!/bin/bash
for i in `ls`;
do
    echo $i is file name\! ;
done
```

```bash
#!/bin/bash
for i in $* ;
do
    echo $i is input chart\! ;
done
```

```bash
#!/bin/bash
for i in f1 f2 f3 ;
do
    echo $i is appoint ;
done
```

```bash
#!/bin/bash
list="rootfs usr data data2"

for i in $list;
do
    echo $i is appoint ;
done
```

**③路径查找**

```bash
#!/bin/bash
for file in /proc/*;
do
    echo $file is file path \! ;
done
```

```bash
#!/bin/bash
for file in $(ls *.sh)
do
    echo $file is file path \! ;
done
```

## 2、While循环

```bash
while condition ; do
    statements ...
done 
```

**`Note:`** 和if一样，condition可以有一系列的statements组成，值是最后的statment的exit status

## 3、Util循环

```bash
until [condition-is-true] ; do 
  statements ... 
done 
```

**`Note:`** 执行statements，直至command正确运行。在循环的顶部判断条件,并且如果条件一直为false那就一直循环下去

# 六、自定义函数

## 格式

```bash
[ function ] 函数名 [()]
{
    action;
    [return int;]
}

# 1.函数在被调用前先声明好
# 2.function关键字可有无
```

## 调用

```bash
$ 函数名()
$ 函数名(参数1,参数2)
$ 函数名 参数1 参数2
$ (函数名 参数1 参数2)
```

# 七、文本读取

## 1. while+read循环读取

①while+read的基础语法语句

```
while IFS= read -r line; do
    echo "$line"
done < input.txt
```

**说明：**

- `IFS=`：避免对行首和行尾的空白字符进行截断。
- `-r`：防止反斜杠（`\`）被解释为转义字符。

优势：

- 简洁、直接，适用于处理大多数文件。
- 使用 `IFS=` 可以确保读取的行内容不被意外截断。

**②读取并分割每一行**

```bash
# 例如需要同时获取行号，可以使用 `cat -n` 或 `nl`
nl -ba input.txt | while read -r line_num line_content; do
    echo "Line $line_num: $line_content"
done

# 或者通过自增变量
line_num=0
while IFS= read -r line; do
    line_num=$((line_num + 1))
    echo "Line $line_num: $line"
done < input.txt
```

**③从管道读取文件**

```bash
cat input.txt | while IFS= read -r line; do
    echo "$line"
done
```

**注意：**

由于子进程的作用域，`while` 中的变量不能传递到主脚本中。如果需要保留变量值，可以使用如下方式：

```
while IFS= read -r line; do
    lines+=("$line")
done < <(cat input.txt)

# 输出数组中的内容
for l in "${lines[@]}"; do
    echo "$l"
done
```

**④文件描述符优化**

```
exec 3< input.txt
while IFS= read -r line <&3; do
    echo "$line"
done
exec 3<&-
```

**优势：**

- 可以显式管理文件打开和关闭。
- 在复杂脚本中避免与标准输入冲突。

## 2. mapfile读取到数组

如果文件不太大，可以用 `mapfile` 将所有行读入数组：

```bash
mapfile -t lines < input.txt

# 遍历数组
for line in "${lines[@]}"; do
    echo "$line"
done
```

**优势：**

- 速度快，代码简单。
- 适合需要随机访问或并行处理的场景。

缺点：

- 一次性加载文件到内存，文件非常大时可能导致内存不足。

## 3. 注意

- **小文件**（几百行以内）：`while read` 和 `mapfile` 差异不大。
- **大文件**（几万行以上）：`while read` 更适合逐行处理，而 `mapfile` 适合一次性操作。

# 八 、并发处理

## 注意

- 在某些情况下，并发执行（使用&符号）可能会因为系统资源的限制而导致并发执行的效果不明显，甚至比串行执行更慢。
- 适合并发处理的场景
  - 逻辑复杂，耗时比较长的
  - 程序需要对外交互的，耗时比较长的。例如连接数据库，请求访问外部系统的

## 示例

例如要同时测试多个服务器 IP的端口是否开起，响应时间多少。

```bash
#!/bin/bash
# 功能函数，测试服务器端口是否打开和响应时间。tcping可以测试端口的响应时间

# 函数，测试服务器端口和响应时间
pingMultServer(){
    # 使用tcping测试端口响应时间
    tcp_ping=`tcping -c 2 $1 $2 2>/dev/null | grep "Average" | awk -F"[='ms'.]" '{print $14}'`   
    if [ "$tcp_ping" ];then
        return $tcp_ping
    else
        # 如果tcping不可用，使用常规ping
        o_ping=`ping -W 2 -c 2 $1 2>/dev/null | grep "round-trip" | awk -F '[=|/ |.]' '{print $10}'`
        [ "$o_ping" ] && return "$o_ping" || return 0
    fi
}
# 原始服务器信息
orign_servers='192.168.1.1:8001,192.168.1.1:8002,192.168.1.1:8443,192.168.1.1:9443'

# 存储ping结果的数组
ping=()
# 将原始服务器信息解析成服务器数组
IFS=',' read -a servers <<< "$orign_servers"
# servers=('192.168.1.1:8080' '192.168.1.1:8081' '192.168.1.1:8082' '192.168.1.1:8083')
for server in ${servers[@]}; do
    # 解析每个服务器的子部分
    IFS=':' read -a serverinfo <<< "$server"
    # 并发地调用pingMultServer函数
    pingMultServer ${serverinfo[0]} ${serverinfo[1]}  &
    pids+=($!)
done

# 等待并发进程完成，创建空数组以接收并发进程的返回值
for pid in "${pids[@]}"; do
    wait "$pid"
    ping+=("$?")
done
# 构建ping结果字符串
pingstr=""
for i in "${!ping[@]}"; do
    pingstr+="\"${ping[i]}\","
done

echo "[ ${pingstr%,} ]" 
```

| 类型     | user  | System | total  |
| -------- | ----- | ------ | ------ |
| 并发执行 | 1.01s | 0.46s  | 5.17s  |
| 串行执行 | 0.97s | 0.39s  | 38.97s |

> **send_email $1 $client $client_random_password /etc/openvpn/client/$client.ovpn**

>  **send_email *$1* $client $clientcnname $client_random_password /etc/openvpn/client/$client.ovpn**

