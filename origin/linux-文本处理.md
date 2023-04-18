# 一、awk

## 1、获取匹配关键字后的内容

```bash
awk '{ if(match($0,"关键字")) {print substr($0,RSTART+RLENGTH) }}'文件

#示例

    # 原始文本
    2018-07-31T09:33:08.160102Z 1 [Note] A temporary password isgenerated for root@localhost: oco4Pr&a!o;v

    # 命令
    awk '{ if(match($0,"root@localhost: ")) {print substr($0,RSTAR+RLENGTH) }}' test.log

    # 结果
    oco4Pr&a!o;v
```

## 2、去除文本中的空行

```bash
awk NF test.txt

# NF代表当前行的字段数，空行的话字段数为0,被awk解释为假，因此不进行输出。
```

## 3、获取匹配关键字后多少的位字符串

```bash
# 样本
a="Location: https://allinone.okd311.curiouser.com:8443/oauth/token/implicit#access_token=FBHwgR1jj2coLoYYfG9SdGUke9L9HmAU2IOI9GaMKrQ&expires_in=86400&scope=user%3Afull&token_type=Bearer"

# 获取"access_token="的值

# 方式一

    echo $a | grep "access_token=" |awk -F"access_token=" '/access_token=/{printf substr($2,0,43)}'

    # 结果： FBHwgR1jj2coLoYYfG9SdGUke9L9HmAU2IOI9GaMKrQ

# 方式二

    echo $a | grep "access_token=" |awk '{ if(match($0,"access_token=")) {print substr($0,RSTART+RLENGTH) }}'| awk -F '&' '{print $1}'

    # 结果： FBHwgR1jj2coLoYYfG9SdGUke9L9HmAU2IOI9GaMKrQ
```

## 4、使用多个分隔符分割字符串

```bash
# @, 空格和Tab都是字段分隔符
awk -F”[@ /t]" '{print $1 $2}'
```

## 5、为每行增加行号

```bash
awk '$0=NR":"$0' 文件名 > 新文件名

# $0表示原来每行的内容，
# NR表示行号，
# 双引号之间表示行号与原来内容之间的delimiter
```

# 二、sed

```bash
sed [-hnV][-e<script>][-f<script文件>][文本文件]

参数说明：
-e <script> 或 --expression=<script> 以选项中指定的script来处理输入的文本文件。
-f <script文件> 或 --file=<script文件> 以选项中指定的script文件来处理输入的文本文件。
-h 或 --help 显示帮助。
-n 或 --quiet或--silent 仅显示script处理后的结果。
-V 或 --version 显示版本信息。
```

**动作说明**

```bash
a ：新增， a 的后面可以接字串，而这些字串会在新的一行出现(目前的下一行)
c ：取代， c 的后面可以接字串，这些字串可以取代 n1,n2 之间的行！
d ：删除，因为是删除啊，所以 d 后面通常不接任何咚咚；
i ：插入， i 的后面可以接字串，而这些字串会在新的一行出现(目前的上一行)
p ：打印，亦即将某个选择的数据印出。通常 p 会与参数 sed -n 一起运行～
s ：取代，可以直接进行取代的工作哩！通常这个 s 的动作可以搭配正规表示法！例如 1,20s/old/new/g
```

## 0、替换环境变量的值到文件中

如果想替换字符串中的值为环境变量的值，可以使用将匹配规则使用`""`括起来，在`""`中直接引用环境变量

```bash
echo "test asdasdas \$TEST" > test.txt
export aaaaa=1234
sed -i -e "s/\$TEST/$aaaaa/g" test.txt
```

如果变量的值中包含特殊字符，例如`/`，与sed匹配模式的关键字符造成歧义。可使用`？`代替`/`

```bash
echo "test asdasdas \$TEST" > test.txt
export aaaaa=/1234
sed -i "s?\$TEST?$aaaaa?g" test.txt
```

## 1、新增内容到末尾行的末尾

```bash
sed '$ s/$/新增内容/'  file_path
```

## 2、去除文本中空行和开头"##"的行

```bash
sed '/^$/d;/^##/d'  file_path
```

## 3、去除文本中的空行

```bash
sed '/^\s*$/d' test.txt
```

## 4、多个匹配规则

```bash
sed -i -e '/hah/a lala\nhehe' -e '/lala/d' test
```

## 5、在查找到匹配行后的操作

```bash
sed -i '/hah/a lallalla' test   #在查找到匹配行后添加一行
sed -i '/hah/a lala\nhehe' test #在查找到匹配行后添加多行
sed -i '/hah/d' test #删除查找到匹配行
```

## 6、在查找匹配行的末首或末尾添加内容

```bash
sed -i '/ha/ s/^/la' test #在查找包含"ha"的行首追加"la","laha"
sed -i '/ha/ s/$/la' test #在查找包含"ha"的行末追加"la"，"hala"
```

## 7、去掉文本中开头带#号注释的行

```bash
sed -i -c -e '/^$/d;/^#/d'  file
```

## 8、去除文本中的换行符^M

Windows下保存的文本文件，上传到Linux/Unix下后总会在末尾多了一个换行符^M，导致一些xml、ini、sh等文件读取错误

```bash
sed 's/^M//' 原文件>新文件

# 注意，^M = Ctrl v + Ctrl m，而不是手动输入^M
```

## 9、每行前后添加空行

每行后面添加一行空行：

```
sed G tmp
```

 每行前面添加一行空行：

```
sed '{x;p;x;}' tmp
```

每行后面添加两行空行：

```
sed 'G;G' tmp
```

 每行前面添加两行空行：

```
sed '{x;p;x;x;p;x;}' tmp
```

每行后面添加三行空行：

```
sed 'G;G;G' tmp
```

 每行前面添加三行空行：

```
sed '{x;p;x;x;p;x;x;p;x}' tmp
```

依次类推，添加几行空行，就有几个G或者x;p;x

## 10、如果行后有空行，则删除，然后每行后面添加空行

```
sed '/^$/d;G' tmp
```

## 11、在匹配行前后添加空行

如果一行里面有shui这个单词，那么在他后面会添加一个空行

```
sed '/shui/G' tmp 
```

 如果一行里面有shui这个单词，那么在他前后各添加一个空行

```
sed '/shui/{x;p;x;G}' tmp
```

如果一行里面有shui这个单词，那么在他前面添加一个空行

```
sed '/shui/{x;p;x;}' tmp 
```

在第一行前面添加空行，想在第几行，命令中的1就改成几

```
sed '1{x;p;x;}' tmp 
```

在第一行后面添加空行，想在第几行，命令中的1就改成几

```
sed '1G' tmp 
```

## 12、每几行后面添加一个空行

每两行后面增加一个空行

```
 sed 'N;/^$/d;G' file.txt
```

每两行前面添加一个空行

```
 sed 'N;/^$/d;{x;p;x;}' tmp
```

每三行后面增加一个空行

```
 sed 'N;N;/^$/d;G' file.txt
```

 每三行前面增加一个空行

```
 sed 'N;N;/^$/d;{x;p;x;}' tmp
```

## 13、以x为开头或以x为结尾的行前后添加空行

以xi为开头的行后面添加空行

```
 sed '/^xi/G;' tmp
```

 以xi为结尾的行前面添加空行

```
 sed '/^xi/{x;p;x;}' tmp
```

以xi为结尾的行后面添加空行

```
 sed '/xi$/G;' tmp
```

 以xi为结尾的行后面添加空行

```
 sed '/xi$/{x;p;x;}' tmp
```

# 三、grep

```bash
grep [OPTION]... PATTERN [FILE]...

-r 是递归查找
-n 是显示行号
-R 查找所有文件包含子目录
-i 忽略大小写
-l 只列出匹配的文件名
-L 列出不匹配的文件名
-w 只匹配整个单词，而不是字符串的一部分
-C 匹配的上下文分别显示[number]行
```

## 1、统计某文件夹下文件的个数

```bash
ls -l /data|grep "^-"|wc -l   #不包含子目录
ls -lR /data|grep "^-"|wc -l  #不含子目录
```

## 2、统计某文件夹下目录的个数

```bash
ls -l /data |grep "^ｄ"|wc -l  #不包含子目录
ls -lR /data|grep "^ｄ"|wc -l  #包含子目录
```

## 3、统计某目录(包含子目录)下的所有某种类型的文件

```bash
ls -lR /data|grep txt|wc -l
```

## 4、去除文本中的空行

```bash
grep -v '^\s*$' test.txt
```

## 5、多关键词匹配

```bash
grep pattern1 | pattern2 files ：显示匹配 pattern1 或 pattern2 的 

grep pattern1 files | grep pattern2 ：显示既匹配 pattern1 又匹配 pattern2 的行。

grep -E '关键词1|关键词2' 
```

## 6、xargs配合grep查找

```bash
find -type f -name '*.php'|xargs grep 'GroupRecord'
```

## 7、查找路径下含有某字符串的所有文件

```bash
grep -rn "hello,world!" *
```

## 8、完全匹配关键词

```bash
grep -Fx 关键词
```

## 9、精准匹配

```bash
grep -w 关键词 
# 不加-w是默认情况
```

## 10、显示文件非#注释内容

```bash
# 显示文件非#注释内容
grep -v ^[[:space:]]*# 文本文件

# # 显示文件非#注释内容，不显示空行
egrep -v '#|^$' filename
```

## 11、删除空行并重定向到新文件

```bash
grep . filename > newfilename
```

# 四、egrep

## 1、只显示文本中的非空行和非注释行

```bash
egrep -v '^$|#'  file_path
```

# 五、cut

## 1、获取硬盘某个分区的UUID号追加到fstab

```bash
blkid | grep /dev/sdb5 | cut -d ' ' -f 2 >>/etc/fstab;sed -i '$ s/$/ data ext4 defaults 0 0/' /etc/fstab
```

# 六、wc 

## 1、统计某个目录下某种文件内总共多少行

```bash
find .  -name "*.java" -print | xargs cat | wc -l
```

# 七、find

- 精确查找
- 实时查找 遍历（慢）
- 支持查找条件较

格式：`find [选项]... [查找路径] [查找条件] [处理动作]`

- 查找路径：指定具体目标路径；默认为当前目录

- 查找条件：可以对文件名、大小、类型、权限等标准进行查找；默认为找出指定路径下的所有文件

- 处理动作：对符合条件的文件做操作，默认输出至屏幕（print）

- 查找条件

  | 关键字          | 说明                                                         |
  | --------------- | ------------------------------------------------------------ |
  | -name           | 根据目标文件的名称进行查找,允许使用“*”及“?”通配符必须加上"     " |
  | -iname          | 不区分名字的大小写                                           |
  | -size           | 根据目标文件的大小进行查找一般使用“＋”、“-”号设置超过或小于指定的大小作为查找条件常用的容量单位包括 kB（注意 k 是小写）、MB、GB(在没有+  -的情况下写的越小越好) |
  | -user           | 根据文件是否属于目标用户进行查找                             |
  | -type           | 根据文件的类型进行查找文件类型包括普通文件（f）、目录（d）、块设备文件（b）、字符设备文件（c）等 |
  | -a              | 和，可以省略                                                 |
  | -o              | 或                                                           |
  | -inum           | 根据文件inode号查找                                          |
  | -perm           | 按文件权限查找                                               |
  | -maxdepth level | 将你的文件以分级的形式查找                                   |
  | -mindepth level | 将你的文件以分级的形式查找                                   |
  | -mtime          | 时间                                                         |
  | -empty          | 空目录                                                       |
  | -nouser         | 无主用户，用户被删除                                         |

- 处理动作

  | 选项    | 作用             |
  | ------- | ---------------- |
  | -print  | 默认，不需要输入 |
  | -ls     | 长格式           |
  | -delete | 删除             |
  | -ok     | 执行一次询问一次 |
  | -exec   | 直接处理，不询问 |

## 1、显示指定路径的文件，排除文件夹

```bash
find . -name '*' -type f -print
```

## 2、显示指定路径的文件，排除指定文件夹

```bash
find . -name '*' ! -path "./.git/*" -type f -print
```

# 八、dos2unix

dos2unix是将Windows格式文件转换为Unix、Linux格式的实用命令。Windows格式文件的换行符为\r\n ,而Unix&Linux文件的换行符为\n. dos2unix命令其实就是将文件中的\r\n 转换为\n。而unix2dos则是和dos2unix互为孪生的一个命令，它是将Linux&Unix格式文件转换为Windows格式文件的命令。

## 安装

```bash
yum install dos2unix -y

#会安装dos2Unix、unix2dos、unix2mac这三条命令
```

## 用法

```bash
dos2unix [options] [-c convmode] [-o file ...] [-n infile outfile ...]

-h      显示命令dos2unix联机帮助信息。
-k      保持文件时间戳不变
-q      静默模式，不输出转换结果信息等
-V      显示命令版本信息
-c      转换模式
-o      在源文件转换，默认参数
-n      保留原本的旧档，将转换后的内容输出到新档案.默认都会直接在原来的文件上修改，
```

## 1、一次转换多个文件

```bash
$> dos2unix filename1 filename2 filename3
```

## 2、默认情况下会在源文件上进行转换，如果需要保留源文件，那么可以使用参数-n 

```bash
dos2unix -n oldfilename newfilename
```

## 3、保持文件时间戳不变

```bash
$ ls -lrt dosfile
 -rw-r--r-- 1 root root 67 Dec 26 11:46 dosfile

$ dos2unix dosfile
dos2unix: converting file dosfile to UNIX format ...

$ ls -lrt dosfile
 -rw-r--r-- 1 root root 65 Dec 26 11:58 dosfile

$ dos2unix -k dosfile
dos2unix: converting file dosfile to UNIX format ...

$ ls -lrt dosfile
 -rw-r--r-- 1 root root 65 Dec 26 11:58 dosfile
```

## 4、静默模式格式化文件

```bash
unix2dos -q dosfile
```

# 八、Perl

## 1、统计文件的行数和字符数

```bash
perl -lne '$i++; $in += length($_); END { print "$i lines, $in characters"; }' 文本文件
```

## 2、转换tab为空格

> 1t = 2sp

```bash
perl -p -i -e 's/\t/  /g' 文本文件
```

## 3、搜索并替换字符

```bash
perl -i -pe's/SEARCH/REPLACE/' 文本文件
```

# 九、vscode

## 1、搜索空行并进行替换

按下ctrl+h键进行正则匹配：`^\s*(?=\r?$)\n`，然后进行替换操作

## 2、插入递增数字

- 安装插件`increment-selection`

- 使用 `option+cmd+向下键`，进行多行编辑
- 再 `cmd+ option + i` 插入数字

- 默认起始数字是0， 如果想改变起始数字，在多行编辑前将第一项改为想要的数字，多行选中后再使用快捷键。
