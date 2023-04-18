# Tcpdump

# 一、简介

tcpdump就是：dump the traffic on a network，根据使用者的定义对网络上的[数据包](https://baike.baidu.com/item/数据包)进行截获的包分析工具。

tcpdump是一个用于截取网络分组，并输出分组内容的工具。凭借强大的功能和灵活的截取策略，使其成为类UNIX系统下用于网络分析和问题排查的首选工具

tcpdump 支持针对网络层、协议、主机、网络或端口的过滤，并提供and、or、not等逻辑语句来帮助你去掉无用的信息

tcpdump需要有**root**权限。

tcpdump使用了独立于系统的libpcap的接口。libpcap是linux平台下的网络数据包捕获函数包

tcpdump调用libpcap的接口在linux系统链路层抓包。而linux本身指定的许多访问控制规则都是基于三层或三层以上的过滤规则，所以tcpdump**可以抓取过滤规则之前的数据包**。

tcpdump可以使用带-w参数的tcpdump 截获数据并保存到文件中，然后再使用Wireshark进行解码分析

tcpdump工作在网卡的**混杂模式**



# 二、使用

## 1、命令格式

```bash
tcpdump [-aAbdDefhHIJKlLnNOpqStuUvxX#] [ -B size ] [ -c count ]
		[ -C file_size ] [ -E algo:secret ] [ -F file ] [ -G seconds ]
		[ -i interface ] [ -j tstamptype ] [ -M secret ] [ --number ]
		[ -Q in|out|inout ]
		[ -r file ] [ -s snaplen ] [ --time-stamp-precision precision ]
		[ --immediate-mode ] [ -T type ] [ --version ] [ -V file ]
		[ -w file ] [ -W filecount ] [ -y datalinktype ] [ -z postrotate-command ]
		[ -Z user ] [ expression ]
```

## 2、命令选项

### ①抓包选项：

**`-A`**:   以ASCII码方式显示每一个数据包(不会显示数据包中链路层头部信息). 在抓取包含网页数据的数据包时, 可方便查看数据(nt: 即Handy for capturing web pages).

**`-C  file-size`**:   (nt: 此选项用于配合-w file 选项使用) 该选项使得tcpdump 在把原始数据包直接保存到文件中之前, 检查此文件大小是否超过file-size. 如果超过了, 将关闭此文件,另创一个文件继续用于原始数据包的记录. 新创建的文件名与-w 选项指定的文件名一致, 但文件名后多了一个数字.该数字会从1开始随着新创建文件的增多而增加. file-size的单位是百万字节`(nt: 这里指1,000,000个字节,并非1,048,576个字节, 后者是以1024字节为1k, 1024k字节为1M计算所得, 即1M=1024 ＊ 1024 ＝ 1,048,576)`

**`-c`**：指定要抓取的包数量。

**`-d`** :  以容易阅读的形式,在标准输出上打印出编排过的包匹配码, 随后tcpdump停止.(nt | rt: human readable, 容易阅读的,通常是指以ascii码来打印一些信息. compiled, 编排过的. packet-matching code, 包匹配码,含义未知, 需补充)

**`-dd`**:  以C语言的形式打印出包匹配码.

**`-ddd`**:  以十进制数的形式打印出包匹配码(会在包匹配码之前有一个附加的'count'前缀).

**`-i interface`**：指定tcpdump需要监听的接口。默认会抓取第一个网络接口

**`-N`**:  不列出域名

**`-n`**：对地址以数字方式显式，否则显式为主机名，也就是说-n选项不做主机名解析。

**`-nn`**：除了-n的作用外，还把端口显示为数值，否则显示端口服务名。

**`-P`**：指定要抓取的包是流入还是流出的包。可以给定的值为`in、out和inout`，默认为"inout"。

**`-p`**:   不让网络界面进入混杂模式

**`-O`**:  不将数据包编码最佳化

**`-s len`**：设置tcpdump的数据包抓取长度为len。

​		如果不设置默认将会是65535字节。对于要抓取的数据包较大时，长度设置不够可能会产生包截断，若出现包截断，输出行中会出现"[|proto]"的标志(proto实际会显示为协议名)。但是抓取len越长，包的处理时间越长，并且会减少tcpdump可缓存的数据包的数量，从而会导致数据包的丢失，所以在能抓取我们想要的包的前提下，抓取长度越小越好。

**`-S`**： 打印TCP 数据包的顺序号时, 使用绝对的顺序号, 而不是相对的顺序号.

​		相对顺序号可理解为, 相对第一个TCP 包顺序号的差距,比如, 接受方收到第一个数据包的绝对顺序号为232323, 对于后来接收到的第2个,第3个数据包, tcpdump会打印其序列号为1, 2分别表示与第一个数据包的差距为1 和 2. 而如果此时-S 选项被设置, 对于后来接收到的第2个, 第3个数据包会打印出其绝对顺序号:232324, 232325

**`-y datalinktype`**：设置tcpdump 只捕获数据链路层协议类型是datalinktype的数据包

### ②输出选项：

**`-e`**：输出的每行中都将包括数据链路层头部信息，例如源MAC和目标MAC。

**`-q`**：快速打印输出。即打印很少的协议相关信息，从而输出行都比较简短。

**`-t`**：在每行输出中不打印时间戳

**`-tt`**：不对每行输出的时间进行格式处理(nt: 这种格式一眼可能看不出其含义, 如时间戳打印成1261798315)

**`-ttt`**：tcpdump 输出时, 每两行打印之间会延迟一个段时间(以毫秒为单位)

**`-tttt`**：在每行打印的时间戳之前添加日期的打印

**`-X`**：输出包的头部数据，会以16进制和ASCII两种方式同时输出。

**`-XX`**：输出包的头部数据，会以16进制和ASCII两种方式同时输出，更详细。

**`-v`**：当分析和打印的时候，产生详细的输出。

**`-vv`**：产生比-v更详细的输出。

**`-vvv`**：产生比-vv更详细的输出。

### ③其他功能性选项：

**`-D`**：列出可用于抓包的接口。将会列出接口的数值编号和接口名，它们都可以用于`"-i"`后。

**`-F`**：从文件中读取抓包的表达式。若使用该选项，则命令行中给定的其他表达式都将失效。

**`-w`**：将抓包数据输出到文件中而不是标准输出。可以同时配合”-G time"选项使得输出文件每time秒就自动切换到另一个文件。可通过"-r"选项载入这些文件以进行分析和打印。

**`-r`**：从给定的数据包文件中读取数据。使用"-"表示从标准输入中读取

## 3、条件表达式定义过滤规则

表达式用于决定哪些数据包将被过滤打印. 如果不给定条件表达式, 网络上所有被捕获的包都会被打印,否则, 只有满足条件表达式的数据包被打印

**表达式格式：**

**`tcpdump 命令选项 proto dir type `**

- **proto(protocol协议类型的过滤器)**：根据协议进行过滤

  - tcp：
  - udp：
  - icmp：
  - ether：
  - fddi：
  - tr：
  - wlan：
  - ip：
  - ip6：
  - arp：
  - rarp：
  - decnet：
  - 若未给定协议类型，则匹配所有可能的类型

- **dir（direction数据流向类型的过滤器）**：根据数据流向进行过滤

  - src：
  - dst： 
  - src  or dst：
  - src and dst：

-  **`type（关键词类型的过滤器）`**：

  - host： 
- net： 
  - port： 
  - portrange： 
  - ether： 
  - gateway： 
  
  例如：host 192.168.201.128  ,  net 128.3, port 20, portrange 6000-6008’

## 4、**过滤规则的组合**

**表达式单元之间可以使用操作符`" and / && / or / || / not / ! "`进行连接，从而组成复杂的条件表达式。**

- **`and`**：所有的条件都需要满足，也可以表示为 **`&&`**
- **`or`**：只要有一个条件满足就可以，也可以表示为 **`||`**
- **`not`**：取反，也可以使用 **`!`**

而在单个过滤器里，常常会判断一条件是否成立，这时候，就要使用下面两个符号

- **`=`**：判断二者相等
- **`==`**：判断二者相等
- **`!=`**：判断二者不相等

使用括号"()"可以改变表达式的优先级，但需要注意的是括号会被shell解释，所以应该使用反斜线""转义为"()"，在需要的时候，还需要包围在引号中。

当你使用这两个符号时，tcpdump 还提供了一些关键字的接口来方便我们进行判断，比如

- **`if`**：表示网卡接口名
- **`proc`**：表示进程名
- **`pid`**：表示进程id
-  **`svc`**：表示 service class
- **`dir`**：表示方向，in 和 out
- **`eproc`**：表示 effective process name
- **`epid`**：表示 effective process ID

# 三、抓包解读

## 输出格式

```bash
$ tcpdump -S -e -nn -i eth0 dst 192.168.1.7 and port 5000

06:24:16.887580 dc:a6:32:c4:88:d2 > 00:11:32:11:22:33, ethertype IPv4 (0x0800), length 338: 192.168.1.18.54832 > 192.168.1.7.5000: Flags [P.], seq 2391754816:2391755088, ack 1379501343, win 2051, options [nop,nop,TS val 238444658 ecr 19745762], length 272
```

第一列：时分秒毫秒 21:26:49.013621

第二列：源MAC地址

第三列：目标MAC地址

第四列：网络协议及其长度

第五列：源IP地址.端口号

第六列：目标IP地址.端口号

第七列：数据包内容，包括Flags 标识符，seq 号，ack 号，win 窗口，数据长度 length，其中 [P.] 表示 PUSH 标志位为 1，

## Flags 标识符

- **[S]** : SYN（开始连接）
- **[P]** : PSH（推送数据）
- **[F]** : FIN （结束连接）
- **[R]** : RST（重置连接）
- **[.]** : 没有 Flag，由于除了 SYN 包外所有的数据包都有ACK，所以一般这个标志也可表示 ACK

# 四、示例

## 1、**找出一段时间内发包数最多的 IP**

```bash
$ tcpdump -nnn -t -c 200 | cut -f 1,2,3,4 -d '.' | sort | uniq -c | sort -nr | head -n 20

200 packets captured
244 packets received by filter
0 packets dropped by kernel
    103 IP 192.168.1.18
     72 IP 220.64.11.92
     25 IP 192.168.1.7
```

## 2、抓取 HTTP GET 请求

```bash
$ tcpdump -s 0 -A -vv 'tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x47455420'

$ tcpdump -vv -A -l -i6 | grep 'GET'
```

## 3、抓取 HTTP POST请求

```bash
$ tcpdump -s 0 -A -vv 'tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x504f5354'

$ tcpdump -vv -A -l -i6 | grep 'POST'
```

## 4、抓取 HTTP请求中的 User-Agent和Host

```bash
$ tcpdump -nn -A -i6 -l | egrep -i 'User-Agent:|Host:'
```

## 5、抓取 HTTP请求中的主机名和路径

```bash
$ tcpdump -i6 -v -n -l | egrep -i "POST /|GET /|Host:"

.Ke..B]'POST /webapi/entry.cgi HTTP/1.1
Host: 192.168.1.7:5000
```

## 5、**抓取 HTTP 有效数据包**

抓取 80 端口的 HTTP 有效数据包，排除 TCP 连接建立过程的数据包（SYN / FIN / ACK）

```bash
$ tcpdump 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'
```

# 参考

1. https://baijiahao.baidu.com/s?id=1671144485218215170&wfr=spider&for=pc
2. https://www.jianshu.com/p/d9162722f189