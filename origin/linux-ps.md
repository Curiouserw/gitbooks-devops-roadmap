# Linux ps

# 一、简介

在Linux系统中，ps(process status)命令常常用来捕获系统在某一时间的进程状态

## ps命令支持的语法格式：

- UNIX 风格，选项可以组合在一起，并且选项前必须有“-”连字符
- BSD 风格，选项可以组合在一起，但是选项前不能有“-”连字符
- GNU 风格的长选项，选项前有两个“-”连字符

## Linux进程状态: 


| 状态码 | 进程状态 | 含义 |
| ------- | ------- | ------- |
| R | 运行（runnable (on run queue)） | 正在运行或在运行队列中等待 |
| S | 中断（sleeping） | 休眠中, 受阻, 在等待某个条件的形成或接受到信号 |
| D | 不可中断（uninterruptible sleep ）(usually IO) | 收到信号不唤醒和不可运行, 进程必须等待直到有中断发生 |
| Z | 僵死（a defunct (”zombie”) process） | 进程已终止, 但进程描述符存在, 直到父进程调用wait4()系统调用后释放 |
| T | 停止（traced or stopped） | 进程收到SIGSTOP, SIGSTP, SIGTIN, SIGTOU信号后停止运行运行 |

## 显示进程信息格式

ps aux最初用到Unix Style中，而ps -ef被用在System V Style中，两者输出略有不同。现在的大部分Linux系统都是可以同时使用这两种方式的

au(x)输出格式

```bash
$ ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0 194364  7256 ?        Ss   09:27   0:25 /usr/lib/systemd/systemd ....
```

| USER   | PID    | %CPU                | %MEM          | VSZ                          | RSS                                            | TTY                                                          | STAT       | START        | TIME                        | COMMAND            |
| ------ | ------ | ------------------- | ------------- | ---------------------------- | ---------------------------------------------- | ------------------------------------------------------------ | ---------- | ------------ | --------------------------- | ------------------ |
| 用户名 | 进程ID | 进程占用的CPU百分比 | 占用CPU百分比 | 该进程使用的虚拟內存量（KB） | 该进程占用的固定內存量（KB）（驻留中页的数量） | 进程在那个终端上运行。<br/>若与终端无关，则显示? <br/>若为pts/0等，则表示由网络连接主机进程。 | 进程的状态 | 进程启动时间 | 该进程实际使用CPU运行的时间 | 所执行的命令及参数 |

ps -ef

```bash
$ ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 09:27 ?        00:00:25 /usr/lib/systemd/systemd ....
```

| UID    | PID    | PPID     | C             | STIME        | TTY                                                          | CMD                |
| ------ | ------ | -------- | ------------- | ------------ | ------------------------------------------------------------ | ------------------ |
| 用户ID | 进程ID | 父进程ID | 占用CPU百分比 | 进程启动时间 | 进程在那个终端上运行。<br>若与终端无关，则显示? <br>若为pts/0等，则表示由网络连接主机进程。 | 所执行的命令及参数 |

STAT状态位常见的状态字符

| 状态字符 | 含义|
| ------------- |----------|
| D | 无法中断的休眠状态（通常 IO 的进程） |
| R | 正在运行可中在队列中可过行的 |
| S | 处于休眠状态 |
| T | 停止或被追踪 |
| X | 死掉的进程 （基本很少见）|
| Z  | 僵尸进程 |
| < | 优先级高的进程 |
| N | 优先级较低的进程 |
| L | 有些页被锁进内存 |
| s | 进程的领导者（在它之下有子进程） |
| l | 多线程，克隆线程（使用 CLONE_THREAD, 类似 NPTL pthreads） |
| \+ | 位于后台的进程组 |

# 二、命令参数

> ps [options] [--help]

```bash
-a  显示所有终端机下执行的进程，除了阶段作业领导者之外。
 a  显示现行终端机下的所有进程，包括其他用户的进程。
-A  显示所有进程。
-c  显示CLS和PRI栏位。
 c  列出进程时，显示每个进程真正的指令名称，而不包含路径，参数或常驻服务的标示。
-C<指令名称> 　指定执行指令的名称，并列出该指令的进程的状况。
-d 　显示所有进程，但不包括阶段作业领导者的进程。
-e 　此参数的效果和指定"A"参数相同。
 e 　列出进程时，显示每个进程所使用的环境变量。
-f 　显示UID,PPIP,C与STIME栏位。
 f 　用ASCII字符显示树状结构，表达进程间的相互关系。
-g<群组名称> 　此参数的效果和指定"-G"参数相同，当亦能使用阶段作业领导者的名称来指定。
 g 　显示现行终端机下的所有进程，包括群组领导者的进程。
-G<群组识别码> 　列出属于该群组的进程的状况，也可使用群组名称来指定。
 h 　不显示标题列。
-H 　显示树状结构，表示进程间的相互关系。
-j或j 　采用工作控制的格式显示进程状况。
-l或l 　采用详细的格式来显示进程状况。
 L 　列出栏位的相关信息。
-m或m 　显示所有的执行绪。
 n 　以数字来表示USER和WCHAN栏位。
-N 　显示所有的进程，除了执行ps指令终端机下的进程之外。
-p<进程识别码> 　指定进程识别码，并列出该进程的状况。
 p<进程识别码> 　此参数的效果和指定"-p"参数相同，只在列表格式方面稍有差异。
 r 　只列出现行终端机正在执行中的进程。
-s<阶段作业> 　指定阶段作业的进程识别码，并列出隶属该阶段作业的进程的状况。
 s 　采用进程信号的格式显示进程状况。
 S 　列出进程时，包括已中断的子进程资料。
-t<终端机编号> 　指定终端机编号，并列出属于该终端机的进程的状况。
 t<终端机编号> 　此参数的效果和指定"-t"参数相同，只在列表格式方面稍有差异。
-T 　显示现行终端机下的所有进程。
-u<用户识别码> 　此参数的效果和指定"-U"参数相同。
 u 　以用户为主的格式来显示进程状况。
-U<用户识别码> 　列出属于该用户的进程的状况，也可使用用户名称来指定。
 U<用户名称> 　列出属于该用户的进程的状况。
 v 　采用虚拟内存的格式显示进程状况。
-V或V 　显示版本信息。
-w或w 　采用宽阔的格式来显示进程状况。　
 x 　显示所有进程，不以终端机来区分。
 X 　采用旧式的Linux i386登陆格式显示进程状况。
-y 配合参数"-l"使用时，不显示F(flag)栏位，并以RSS栏位取代ADDR栏位。
```

# 三、示例

## 1. 查看CPU占用最多的前10个进程

```bash
ps auxw|head -1;ps auxw|sort -rn -k3|head -10 
```

## 2.显示所有当前进程

> ps -ax

 **-a**参数，-a 代表 **all**。

**-x**参数会显示没有控制终端的进程

## 3. 查看虚拟内存使用最多的前10个进程

> ps auxw | head -1 ; ps auxw|sort -rn -k5|head -10

## 4.  根据用户名过滤进程

> ps -u root

```bash
$ ps -u root
  PID TTY          TIME CMD
    1 ?        00:00:25 systemd
    2 ?        00:00:00 kthreadd
    3 ?        00:00:07 ksoftirqd/0
    6 ?        00:00:02 kworker/u40:0
    8 ?        00:00:05 migration/0
```

## 5. 根据内存使用来升序排序

>  ps -aux --sort -pmem 

```bash
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
1000130+ 13437 16.7  9.2 13303016 5703272 ?    Ssl  09:29 131:50 /usr/lib/jvm/jre/bin/java ....
1000540+ 25444  8.7  4.1 9575924 2533824 ?     Ssl  09:30  68:56 /usr/share/elasticsearch/ .....
```

## 6. 根据 CPU 使用来升序排序

>  ps -aux --sort -pcpu

```bash
$ 
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root      7602 48.8  0.3 4883792 231504 ?      Ssl  09:29 385:36 /usr/bin/hyperkube kubelet ....
root      3282 25.0  1.4 4151700 895280 ?      Ssl  09:28 198:12 openshift start master api 
```

## 7. 格式化输出 指定用户（真实或有效的UID）创建的进程

> ps -U pid_user -u pid_user  u

**-U** 参数按真实用户 **ID(RUID)** 筛选进程，它会从用户列表中选择真实用户名或 ID。真实用户即实际创建该进程的用户

**-u** 参数用来筛选有效用户 **ID（EUID）**

**u** 参数用来决定以针对用户的格式输出，由 **User, PID, %CPU, %MEM, VSZ, RSS, TTY, STAT, START, TIME 和 COMMAND这几列组成**

```bash
$ pid_user=ceph && ps -U $pid_user -u $pid_user  u
USER  PID %CPU %MEM    VSZ   RSS TTY  STAT START   TIME COMMAND
ceph  1474  0.2  0.0 398604 40772 ?   Ssl  09:27   2:10 /usr/bin/ceph-mon -f --cluster ceph --id ...
ceph  1478  0.0  0.0 393520 34540 ?   Ssl  09:27   0:21 /usr/bin/ceph-mds -f --cluster ceph --id ...
ceph  1845  0.1  0.3 1104056 219284 ? Ssl  09:27   0:59 /usr/bin/ceph-osd -f --cluster ceph --id ...
```

# 参考

1. https://blog.csdn.net/ShiXueTanLang/article/details/80781089
2. https://www.cnblogs.com/jiqing9006/p/10036676.html
3. https://www.cnblogs.com/xiangtingshen/p/10920236.html
4. http://www.sohu.com/a/238434503_100180425
5. https://www.cnblogs.com/fps2tao/p/10313337.html