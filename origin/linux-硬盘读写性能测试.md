# DD硬盘命令详解

Linux下可使用dd命令来测试硬盘读写速度

# 一、语法简介

```bash
dd if=path/to/input_file of=/path/to/output_file bs=block_size count=number_of_blocks
```

参数

```bash
if=file 　　　　　　　　　　　　　　　　输入文件名，缺省为标准输入
of=file 　　　　　　　　　　　　　　　　输出文件名，缺省为标准输出
ibs=bytes 　　　　　　　　　　　　　　　一次读入 bytes 个字节(即一个块大小为 bytes 个字节)
obs=bytes 　　　　　　　　　　　　　　　一次写 bytes 个字节(即一个块大小为 bytes 个字节)
bs=bytes 　　　　　　　　　　　　　　　 同时设置读写块的大小为 bytes ，可代替 ibs 和 obs
cbs=bytes 　　　　　　　　　　　　　　　一次转换 bytes 个字节，即转换缓冲区大小
skip=blocks 　　　　　　　　　　　　　 从输入文件开头跳过 blocks 个块后再开始复制
seek=blocks      　　　　　　　　　　 从输出文件开头跳过 blocks 个块后再开始复制(通常只有当输出文件是磁盘或磁带时才有效)
count=blocks 　　　　　　　　　　　　　仅拷贝 blocks 个块，块大小等于 ibs 指定的字节数
conv=conversion[,conversion...]    用指定的参数转换文件。
iflag=FLAGS　　　　　　　　　　　　　　指定读的方式FLAGS，参见“FLAGS参数说明”
oflag=FLAGS　　　　　　　　　　　　　　指定写的方式FLAGS，参见“FLAGS参数说明”


#conv 转换参数：
ascii 　　　　　　　　　　　　　　　　　转换 EBCDIC 为 ASCII
ebcdic 　　　　　　　　　　　　     　 转换 ASCII 为 EBCDIC
ibm 　　　　　　　　　　　　　　　　　　转换 ASCII 为 alternate EBCDIC
block 　　　　　　　　　　　　　　　　 把每一行转换为长度为 cbs 的记录，不足部分用空格填充
unblock 　　　　　　　　　　　　　　　 使每一行的长度都为 cbs ，不足部分用空格填充
lcase 　　　　　　　　　　　　　　　　 把大写字符转换为小写字符
ucase 　　　　　　　　　　　　　　　　 把小写字符转换为大写字符
swab 　　　　　　　　　　　　　　　　  交换输入的每对字节
noerror 　　　　　　　　　　　　　　　 出错时不停止
notrunc 　　　　　　　　　　　　　　　 不截短输出文件。
sync 　　　　　　　　　　　　　　　　　 把每个输入块填充到ibs个字节，不足部分用空(NUL)字符补齐
```

FLAGS 参数说明：​

```bash
append -append  mode  (makes  sense  only  for output; conv=notrunc sug-gested)
direct　　　　　　　　　　　　　　　  读写数据采用直接IO方式
directory　　　　　　　　　　　　　　读写失败除非是directory
dsync　　　　　　　　　　　　　　　　 读写数据采用同步IO
sync　　　　　　　　　　　　　　　　　同上，但是针对是元数据
fullblock　　　　　　　　　　　　　　堆积满block（accumulate full blocks of input ）(iflag only)
nonblock　　　　　　　　　　　　　　 读写数据采用非阻塞IO方式
noatime　　　　　　　　　　　　　　　 读写数据不更新访问时间
```

# 二、DD磁盘读写测试

## 1、相关参数

- time有计时作用，dd用于复制，从if读出，写到of
- if=/dev/zero（产生字符）不产生IO，因此可以用来测试纯写速度
- 同理of=/dev/null（回收站、无底洞）不产生IO，可以用来测试纯读速度
- 将/tmp/test拷贝到/var则同时测试了读写速度
- bs是每次读或写的大小，即一个块的大小，count是读写块的数量

当写入到驱动盘的时候，我们简单的从无穷无用字节的源 /dev/zero 读取，当从驱动盘读取的时候，我们读取的是刚才的文件，并把输出结果发送到无用的 /dev/null。在整个操作过程中， DD 命令会跟踪数据传输的速度并且报告出结果。

## 2、测试磁盘写能力

```bash
time dd if=/dev/zero of=/testw.dbf bs=4k count=100000
```

因为/dev//zero是一个伪设备，它只产生空字符流，对它不会产生IO，所以，IO都会集中在of文件中，of文件只用于写，所以这个命令相当于测试磁盘的写能力。命令结尾添加oflag=direct将跳过内存缓存，添加oflag=sync将跳过hdd缓存。

## 3、测试磁盘读能力

```bash
time dd if=/dev/sdb of=/dev/null bs=4k
```

因为/dev/sdb是一个物理分区，对它的读取会产生IO，/dev/null是伪设备，相当于黑洞，of到该设备不会产生IO，所以，这个命令的IO只发生在/dev/sdb上，也相当于测试磁盘的读能力。（Ctrl+c终止测试）

## 4、测试同时读写能力

```bash
time dd if=/dev/sdb of=/testrw.dbf bs=4k
```

在这个命令下，一个是物理分区，一个是实际的文件，对它们的读写都会产生IO（对/dev/sdb是读，对/testrw.dbf是写），假设它们都在一个磁盘中，这个命令就相当于测试磁盘的同时读写能力。

## 5、测试纯写入性能

```bash
time dd if=/dev/zero of=test bs=8k count=10000 oflag=direct
```

## 6、测试纯读取性能

```bash
dd if=test of=/dev/null bs=8k count=10000 iflag=direct
```

注意：dd 只能提供一个大概的测试结果，而且是连续 I/O 而不是随机 I/O，理论上文件规模越大，测试结果越准确。 同时，iflag/oflag 提供 direct 模式，direct 模式是把写入请求直接封装成 I/O 指令发到磁盘，非 direct 模式只是把数据写入到系统缓存就认为 I/O 成功，并由操作系统决定缓存中的数据什么时候被写入磁盘。

# 三、DD刻录功能相关

## 1、复制磁盘所有扇区数据到文件

```bash
sudo dd if=/dev/sdX of=/path/to/output/file bs=4M
```

## 2、刻录磁盘文件到U盘

```bash
sudo dd if=netboot.xyz.img of=/dev/rdiskN bs=1m
# 如果遇到Invalid number '1m', 说明用的是GNU的dd，那参数 bs=1M
```

# 四、磁盘读写测试结果

## 磁盘数据

| 品牌     | 类型 | 容量  | 主控 | 存储芯片 | 缓存 | 接口    | 接入方式 | 接入系统   |
| -------- | ---- | ----- | ---- | -------- | ---- | ------- | -------- | ---------- |
| Kingston | SD   | 256GB |      |          |      | USB 3.2 | USB 3.0  | AsusMerlin |
| Coolfish | SSD  | 128GB |      |          |      |         | USB 2.0  | AsusMerlin |
| Surma    | SSD  | 128GB |      |          |      |         | PCIE     | Synology   |
| TOSHIBA  | HDD  | 1TB   |      |          |      |         | SATA     | Synology   |

## 缓存写入性能测试

**`time dd if=/dev/zero of=test bs=8k count=100000`**

| 写入磁盘 | 写入大小                 | 总耗时    | 平均速率  | 系统耗时 |
| -------- | ------------------------ | --------- | --------- | -------- |
| Kingston | 819200000 bytes(819 MB） | 4.7764 s  | 172 MB/s  | 1.39s    |
| Coolfish | 819200000 bytes(819 MB） | 16.6004 s | 49.3 MB/s | 1.05s    |
| Surma    | 819200000 bytes(819 MB） | 1.57491 s | 520 MB/s  | 1.49s    |
| TOSHIBA  | 819200000 bytes(819 MB） | 1.72799 s | 474 MB/s  | 1.47s    |

## 直接写入性能测试

**`time dd if=/dev/zero of=test bs=8k count=100000 oflag=direct`**

| 写入磁盘 | 写入大小                 | 总耗时    | 平均速率  | 系统耗时 |
| -------- | ------------------------ | --------- | --------- | -------- |
| Kingston | 819200000 bytes(819 MB） | 9.4353 s  | 86.8 MB/s | 3.45s    |
| Coolfish | 819200000 bytes(819 MB） | 29.6246 s | 27.7 MB/s | 1.58s    |
| Surma    | 819200000 bytes(819 MB） | 13.4049 s | 61.1 MB/s | 6.88s    |
| TOSHIBA  | 819200000 bytes(819 MB） | 41.5395 s | 19.7 MB/s | 14.63s   |

# 参考链接

1. http://www.360doc.com/content/15/0906/17/8737500_497292503.shtml
