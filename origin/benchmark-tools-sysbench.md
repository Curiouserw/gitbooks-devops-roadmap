# 基准测试工具SysBench

# 一、简介

SysBench是一个模块化的、跨平台、多线程基准测试工具，主要用于评估测试各种不同系统参数下的数据库负载情况。它主要包括以下几种方式的测试：
1. cpu性能
2. 磁盘io性能
3. 调度程序性能
4. 内存分配及传输速度
5. POSIX线程性能
6. 数据库性能(OLTP基准测试)   

目前sysbench主要支持 [MySQL](http://www.oschina.net/p/mysql),[pgsql](http://www.oschina.net/p/postgresql),[oracle](http://www.oschina.net/p/oracle10g) 这3种数据库。

GitHub：https://github.com/akopytov/sysbench

# 二、安装

## 1、包管理器安装

### Linux

- Debian/Ubuntu

  ```bash
  curl -s https://packagecloud.io/install/repositories/akopytov/sysbench/script.deb.sh | sudo bash
  sudo apt -y install sysbench
  ```

- RHEL/CentOS:

  ```bash
  curl -s https://packagecloud.io/install/repositories/akopytov/sysbench/script.rpm.sh | sudo bash
  sudo yum -y install sysbench
  ```

- Fedora:

  ```bash
  curl -s https://packagecloud.io/install/repositories/akopytov/sysbench/script.rpm.sh | sudo bash	
  sudo dnf -y install sysbench
  ```

- Arch Linux:

  ```bash
  sudo pacman -Suy sysbench
  ```

### MacOS

```bash
# Add --with-postgresql if you need PostgreSQL support
brew install sysbench
```

## 2、**源码安装**

### ①安装依赖

#### Debian/Ubuntu

```bash
apt -y install make automake libtool pkg-config libaio-dev
# For MySQL support
apt -y install libmysqlclient-dev libssl-dev
# For PostgreSQL support
apt -y install libpq-dev
```

#### RHEL/CentOS

```bash
yum -y install make automake libtool pkgconfig libaio-devel
# For MySQL support, replace with mysql-devel on RHEL/CentOS 5
yum -y install mariadb-devel openssl-devel
# For PostgreSQL support
yum -y install postgresql-devel
```

#### Fedora

```bash
dnf -y install make automake libtool pkgconfig libaio-devel
# For MySQL support
dnf -y install mariadb-devel openssl-devel
# For PostgreSQL support
dnf -y install postgresql-devel
```

#### macOS

Assuming you have Xcode (or Xcode Command Line Tools) and Homebrew installed:

```bash
brew install automake libtool openssl pkg-config
# For MySQL support
brew install mysql
# For PostgreSQL support
brew install postgresql
# openssl is not linked by Homebrew, this is to avoid "ld: library not found for -lssl"
export LDFLAGS=-L/usr/local/opt/openssl/lib 
```

### ②编译安装

```bash
./autogen.sh
# Add --with-pgsql to build with PostgreSQL support
./configure
make -j
make install

# The above will build sysbench with MySQL support by default. If you have MySQL headers and libraries in non-standard locations (and no `mysql_config` can be found in the `PATH`), you can specify them explicitly with `--with-mysql-includes` and `--with-mysql-libs` options to `./configure`.

# To compile sysbench without MySQL support, use `--without-mysql`. If no database drivers are available database-related scripts will not work, but other benchmarks will be functional.
```

## 3、Docker

### Dockerfile

```bash
FROM debian:latest

RUN sed -i 's/deb.debian.org/mirrors.aliyun.com/g' /etc/apt/sources.list \
    && apt update \
    && apt -y install curl \
    && curl -s https://packagecloud.io/install/repositories/akopytov/sysbench/script.deb.sh | bash \
    && apt update \
    && apt -y install sysbench redis-tools dumb-init \
    && rm -rf /var/lib/apt/lists/*

ENTRYPOINT ["dumb-init", "--"]
```

### Makefile

```bash
REGISTRY = 192.168.1.60
IMAGE_REPO = tools
IMAGE_NAME = mid-bf-tools
IMAGE_VERSION = $(v)
all: build push
build:
		docker build --rm -f Dockerfile -t ${REGISTRY}/${IMAGE_REPO}/${IMAGE_NAME}:${IMAGE_VERSION} .
push:
		docker push ${REGISTRY}/${IMAGE_REPO}/${IMAGE_NAME}:${IMAGE_VERSION}
```

### Kubernetes 

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: sysbench
  name: sysbench
spec:
  containers:
  - image: 192.168.1.60/tools/mid-bf-tools:v1
    name: sysbench
    command:
    - sysbench
    - --db-driver=mysql
    - --report-interval=2
    - --mysql-table-engine=innodb
    - --oltp-table-size=100000
    - --oltp-tables-count=24
    - --threads=64
    - --time=99999
    - --mysql-host=galera
    - --mysql-port=3306
    - --mysql-user=sbtest
    - --mysql-password=password
    - /usr/share/sysbench/tests/include/oltp_legacy/oltp.lua
    - run
  restartPolicy: Never
```

第三方的docker镜像：https://hub.docker.com/r/severalnines/sysbench

# 三、使用方法

## 命令语法

```bash
sysbench [选项]... [testname] [命令]
```

```bash
通用选项:
--threads=N                     # 线程的数量，默认是1
--events=N                      # 限制的最大事件数量，默认是0，不限制
--time=N                        # 整个测试执行的时间 [10]
--forced-shutdown=STRING        number of seconds to wait after the --time limit before forcing shutdown, or 'off' to disable [off]
--thread-stack-size=SIZE        size of stack per thread [64K]
--rate=N                        average transactions rate. 0 for unlimited rate [0]
--report-interval=N             periodically report intermediate statistics with a specified interval in seconds. 0 disables intermediate reports [0]
--report-checkpoints=[LIST,...] dump full statistics and reset all counters at specified points in time. The argument is a list of comma-separated values representing the amount of time in seconds elapsed from start of test when report checkpoint(s) must be performed. Report checkpoints are off by default. []
--debug[=on|off]                print more debugging info [off]
--validate[=on|off]             perform validation checks where possible [off]
--help[=on|off]                 print help and exit [off]
--version[=on|off]              print version and exit [off]
--config-file=FILENAME          File containing command line options
--tx-rate=N                     deprecated alias for --rate [0]
--max-requests=N                deprecated alias for --events [0]
--max-time=N                    deprecated alias for --time [0]
--num-threads=N                 deprecated alias for --threads [1]

Pseudo-Random Numbers Generator options:
  --rand-type=STRING random numbers distribution {uniform,gaussian,special,pareto} [special]
  --rand-spec-iter=N number of iterations used for numbers generation [12]
  --rand-spec-pct=N  percentage of values to be treated as 'special' (for special distribution) [1]
  --rand-spec-res=N  percentage of 'special' values to use (for special distribution) [75]
  --rand-seed=N      seed for random number generator. When 0, the current time is used as a RNG seed. [0]
  --rand-pareto-h=N  parameter h for pareto distribution [0.2]

Log options:
  --verbosity=N verbosity level {5 - debug, 0 - only critical messages} [3]

  --percentile=N       percentile to calculate in latency statistics (1-100). Use the special value of 0 to disable percentile calculations [95]
  --histogram[=on|off] print latency histogram in report [off]

General database options:

  --db-driver=STRING  specifies database driver to use ('help' to get list of available drivers) [mysql]
  --db-ps-mode=STRING prepared statements usage mode {auto, disable} [auto]
  --db-debug[=on|off] print database-specific debug information [off]


Compiled-in database drivers:
  mysql - MySQL driver

mysql options:
  --mysql-host=[LIST,...]          MySQL server host [localhost]
  --mysql-port=[LIST,...]          MySQL server port [3306]
  --mysql-socket=[LIST,...]        MySQL socket
  --mysql-user=STRING              MySQL user [sbtest]
  --mysql-password=STRING          MySQL password []
  --mysql-db=STRING                MySQL database name [sbtest]
  --mysql-ssl[=on|off]             use SSL connections, if available in the client library [off]
  --mysql-ssl-cipher=STRING        use specific cipher for SSL connections []
  --mysql-compression[=on|off]     use compression, if available in the client library [off]
  --mysql-debug[=on|off]           trace all client library calls [off]
  --mysql-ignore-errors=[LIST,...] list of errors to ignore, or "all" [1213,1020,1205]
  --mysql-dry-run[=on|off]         Dry run, pretend that all MySQL client API calls are successful without executing them [off]

Compiled-in tests:
  fileio - File I/O test
  cpu - CPU performance test
  memory - Memory functions speed test
  threads - Threads subsystem performance test
  mutex - Mutex performance test
```

### command

command是sysbench要执行的命令，包括以下三种：

```bash
prepare 为测试提前准备数据
run 执行正式的测试
cleanup 在测试完成后对数据库进行清理
```

### testname

```bash
可以是内置的fileio，memory等，也可以是lua脚本或者是lua脚本的路径
```

testname 指定了要进行的测试，在老版本的sysbench中，可以通过--test参数指定测试的脚本；

而在新版本中，--test参数已经声明为废弃，可以不使用--test，而是直接指定脚本。测试时使用的脚本为lua脚本，可以使用sysbench自带脚本，也可以自己开发。

## 测试fileio命令帮助

```bash
sysbench --test=fileio help

fileio options:
  --file-num=N                  代表生成测试文件的数量，默认为128。
  --file-block-size=N           测试时所使用文件块的大小，如果想磁盘针对innodb存储引擎进行测试，可以将其设置为16384，
  																即innodb存储引擎页的大小。默认为16384。
  --file-total-size=SIZE        创建测试文件的总大小，默认为2G大小。
  --file-test-mode=STRING       文件测试模式，包含：seqwr(顺序写), seqrewr(顺序读写), seqrd(顺序读), rndrd(随机读), 
  																rndwr(随机写), rndrw(随机读写)。
  --file-io-mode=STRING         文件操作的模式，sync（同步）,async（异步）,fastmmap（快速mmap）,slowmmap（慢速mmap），
  																默认为sync同步模式。
  --file-extra-flags=[LIST,...] list of additional flags to use to open files {sync,dsync,direct} []
  --file-fsync-freq=N           执行fsync()函数的频率。fsync主要是同步磁盘文件，因为可能有系统和磁盘缓冲的关系。 
  																0代表不使用fsync函数。默认值为100。
  --file-fsync-all[=on|off]     每执行完一次写操作，就执行一次fsync。默认为off。
  --file-fsync-end[=on|off]     在测试结束时执行fsync函数。默认为on。
  --file-fsync-mode=STRING      文件同步函数的选择，同样是和API相关的参数，由于多个操作系统对于fdatasync支持不同，
  																因此不建议使用fdatasync。默认为fsync。
  --file-merged-requests=N      大多情况下，合并可能的IO的请求数，默认为0不合并。
  --file-rw-ratio=N             测试时的读写比例，默认时为1.5，即可3：2。

```

##  测试memory命令帮助

```bash
sysbench --test=memory help

memory options:
  --memory-block-size=SIZE    测试内存块的大小，默认为1K
  --memory-total-size=SIZE    数据传输的总大小，默认为100G
  --memory-scope=STRING       内存访问的范围，包括全局和本地范围，默认为global
  --memory-oper=STRING        内存操作的类型，包括read, write, none，默认为write
  --memory-access-mode=STRING 内存访问模式，包括seq,rnd两种模式，默认为seq

```

## 测试threads命令帮助

```bash
$ sysbench --test=threads help

threads options:
  --thread-yields=N 指定每个请求的压力，默认为1000
  --thread-locks=N  指定每个线程的锁数量，默认为8

```

## **测试mutex命令帮助**

```bash
sysbench --test=mutex help

mutex options:
  --mutex-num=N   数组互斥的总大小。默认是4096
  --mutex-locks=N 每个线程互斥锁的数量。默认是50000
  --mutex-loops=N 内部互斥锁的空循环数量。默认是10000
```

## 注意

1. sysbench 1.0版本以上，默认不支持`sysbench --test=oltp `，使用`sysbench oltp_read_write`代替

   参考：https://github.com/akopytov/sysbench/issues/281



# 四、示例及结果解读

## 示例1、测试MySQL读写性能

### 准备数据

手动在数据库里面创建对应的DB

```bash
sysbench oltp_read_write \
 --db-driver=mysql \
 --mysql-host=localhost \
 --mysql-db=sysbench \
 --mysql-user=root \
 --mysql-password \
 --rand-type=uniform \
 --table_size=1000000 --tables=2 --threads=1 --events=0 \
 prepare
```

以上命令会在`sysbench`库中创建两张包含一百万条记录的表`sbtest1`和`sbtest2`

### 开始测试

```bash
sysbench oltp_read_write \
--threads=16 \
--histogram=on  \
--report-interval=10 \
--mysql-host=localhost \
--mysql-db=sysbench \
--mysql-user=root \
--mysql-password= \
run
```

### 测试结果解读

```bash
Running the test with following options:
Number of threads: 16
Initializing random number generator from current time

Initializing worker threads...

Threads started!

[ 10s ] thds: 16 tps: 1690.78 qps: 33849.48 (r/w/o: 23700.00/6765.32/3384.16) lat (ms,95%): 29.72 err/s: 1.00 reconn/s: 0.00
Latency histogram (values are in milliseconds)
       value  ------------- distribution ------------- count
       3.020 |                                         1
       3.130 |                                         1
       3.187 |*                                        9
       3.245 |**                                       20
       3.304 |**                                       24
       3.364 |****                                     46
       3.425 |******                                   75
       3.488 |***********                              141
       3.551 |****************                         208
       3.615 |********************                     258
       3.681 |************************                 310
       3.748 |***************************              345
       3.816 |*******************************          398
       3.885 |********************************         414
       3.956 |********************************         412
       4.028 |************************************     459
       4.101 |**************************************** 512
       4.176 |**************************************   488
       4.252 |************************************     457
       4.329 |**************************************   486
       4.407 |*************************************    470
       4.487 |********************************         407
       4.569 |*********************************        422
       4.652 |*********************************        417
       4.737 |********************************         411
       4.823 |*****************************            365
       4.910 |****************************             360
       4.999 |*************************                324
       5.090 |*************************                322
       5.183 |**************************               327
       5.277 |*********************                    270
       5.373 |***********************                  289
       5.470 |******************                       230
       5.570 |*******************                      240
       5.671 |*****************                        218
       5.774 |****************                         206
       5.879 |***************                          190
       5.986 |**************                           174
       6.095 |************                             154
       6.205 |************                             156
       6.318 |************                             149
       6.433 |*************                            160
       6.550 |************                             155
       6.669 |**********                               125
       6.790 |**********                               126
       6.913 |*********                                115
       7.039 |*********                                119
       7.167 |*********                                118
       7.297 |********                                 100
       7.430 |*********                                114
       7.565 |********                                 96
       7.702 |*******                                  84
       7.842 |*******                                  93
       7.985 |*******                                  88
       8.130 |*******                                  88
       8.277 |*******                                  87
       8.428 |******                                   71
       8.581 |******                                   71
       8.737 |*****                                    68
       8.895 |******                                   79
       9.057 |*****                                    58
       9.222 |******                                   73
       9.389 |*****                                    63
       9.560 |****                                     49
       9.734 |*****                                    70
       9.910 |****                                     51
      10.090 |***                                      42
      10.274 |***                                      41
      10.460 |*****                                    58
      10.651 |*****                                    64
      10.844 |****                                     51
      11.041 |****                                     51
      11.242 |****                                     53
      11.446 |****                                     48
      11.654 |***                                      44
      11.866 |****                                     55
      12.081 |****                                     57
      12.301 |****                                     55
      12.524 |****                                     52
      12.752 |*****                                    58
      12.984 |****                                     50
      13.219 |****                                     48
      13.460 |****                                     46
      13.704 |***                                      43
      13.953 |****                                     54
      14.207 |******                                   77
      14.465 |***                                      43
      14.728 |****                                     56
      14.995 |*****                                    62
      15.268 |****                                     57
      15.545 |****                                     53
      15.828 |****                                     51
      16.115 |****                                     52
      16.408 |****                                     51
      16.706 |****                                     47
      17.010 |****                                     46
      17.319 |***                                      33
      17.633 |***                                      44
      17.954 |*****                                    66
      18.280 |***                                      37
      18.612 |***                                      41
      18.950 |***                                      44
      19.295 |****                                     46
      19.645 |****                                     45
      20.002 |***                                      44
      20.366 |**                                       30
      20.736 |***                                      37
      21.112 |***                                      38
      21.496 |***                                      42
      21.886 |***                                      33
      22.284 |***                                      35
      22.689 |***                                      35
      23.101 |**                                       28
      23.521 |***                                      35
      23.948 |***                                      35
      24.384 |**                                       26
      24.827 |**                                       25
      25.278 |**                                       31
      25.737 |**                                       30
      
 			# ....省略....
      
      87.564 |                                         6
      89.155 |*                                        9
      90.775 |                                         5
      92.424 |                                         6
      94.104 |                                         6
      95.814 |                                         6
      97.555 |                                         4
      99.327 |                                         5
     101.132 |*                                        7
     102.969 |                                         2
     104.840 |                                         6
     106.745 |                                         5
     108.685 |                                         2
     110.659 |                                         4
     112.670 |                                         3
     114.717 |*                                        8
     116.802 |                                         5
     118.924 |                                         3
     121.085 |                                         1
     123.285 |                                         6
     125.525 |*                                        7
     127.805 |                                         4
     130.128 |                                         1
     132.492 |                                         1



SQL statistics:
    queries performed:
        read:                            237118	# 总select数量
        write:                           67723  # 总update、insert、delete语句数量
        other:                           33864  # commit、unlock tables以及其他mutex的数量
        total:                           338705  
    transactions:                        16927  (1690.41 per sec.)	# TPS
    queries:                             338705 (33824.71 per sec.)	# QPS
    ignored errors:                      10     (1.00 per sec.)			# 忽略的错误
    reconnects:                          0      (0.00 per sec.)			# 重新连接

General statistics:
    total time:                          10.0122s	# 测试的总时间
    total number of events:              16927		# 总的事件数，一般和TPS一样

Latency (ms):
         min:                                    2.99	# 最小响应时间
         avg:                                    9.46	# 平均响应时间
         max:                                  132.49	# 最大响应时间
         95th percentile:                       29.72	# 95%的响应时间是这个数据
         sum:                               160046.87

Threads fairness:
    events (avg/stddev):           1057.9375/8.09
    execution time (avg/stddev):   10.0029/0.00

```

### 清理测试数据

```bash
sysbench oltp_read_write \
 --db-driver=mysql \
 --mysql-host=localhost \
 --mysql-db=sysbench \
 --mysql-user=root \
 --mysql-password \
 cleanup
```

