# Proc文件系统

# 一、简介

Linux系统上的/proc目录是一种文件系统，即proc文件系统。与其它常见的文件系统不同的是，/proc是一种伪文件系统（也即虚拟文件系统），存储的是当前内核运行状态的一系列特殊文件，用户可以通过这些文件查看有关系统硬件及当前正在运行进程的信息，甚至可以通过更改其中某些文件来改变内核的运行状态。 

基于/proc文件系统如上所述的特殊性，其内的文件也常被称作虚拟文件，并具有一些独特的特点。例如，其中有些文件虽然使用查看命令查看时会返回大量信息，但文件本身的大小却会显示为0字节。此外，这些特殊文件中大多数文件的时间及日期属性通常为当前系统时间和日期，这跟它们随时会被刷新（存储于RAM中）有关。 

为了查看及使用上的方便，这些文件通常会按照相关性进行分类存储于不同的目录甚至子目录中，如/proc/scsi目录中存储的就是当前系统上所有SCSI设备的相关信息，/proc/N中存储的则是系统当前正在运行的进程的相关信息，其中N为正在运行的进程（可以想象得到，在某进程结束后其相关目录则会消失）。 

大多数虚拟文件可以使用文件查看命令如cat、more或者less进行查看，有些文件信息表述的内容可以一目了然，但也有文件的信息却不怎么具有可读性。不过，这些可读性较差的文件在使用一些命令如apm、free、lspci或top查看时却可以有着不错的表现。

# 二、常见文件

```bash
1      10824   11989  12474  15454  4283  5481  7368   9174       cpuinfo      kpagecount    softirqs
10     10827   11992  12476  15461  4288  5497  8      9328       crypto       kpageflags    stat
10005  10828   11993  12477  15462  4289  5541  8192   9362       devices      loadavg       swaps
10026  10829   11994  12479  15463  430   555   8204   9382       diskstats    locks         synobios
10136  10863   11995  12481  15464  4376  562   8214   9385       dma          mdstat        syno_cpu_arch
10137  10864   11996  12483  15538  4522  5698  8215   9386       driver       megaraid      syno_loadavg
10246  11      11997  12567  15577  4523  580   8235   9387       execdomains  meminfo       syno_platform
10295  11242   11998  12568  15610  4524  6025  8246   9388       fb           misc          synotty
10297  11274   11999  12777  15887  4651  6026  8251   9389       filesystems  modules       sys
10365  11297   12     12812  16     5     6027  8254   9394       fs           mounts        sysrq-trigger
10369  11328   12001  12873  16104  5007  603   8257   9397       interrupts   mpt           sysvipc
10371  11366   12002  12904  16124  5010  6057  8258   9400       iomem        mtrr          thread-self
10372  11394   12022  13     16131  5013  607   8455   9619       ioports      net           timer_list
10606  11400   12067  13471  16133  5016  612   8457   9620       irq          pagetypeinfo  tty
10738  11425   12104  13681  16139  5289  6149  9      acpi       kallsyms     partitions    uptime
10773  11704   12106  13685  16195  5394  6179  9075   buddyinfo  kcore        sched_debug   version
10777  11712   12314  14     16324  5398  6290  9077   bus        keys         schedstat     vmallocinfo
10821  11715   12420  15     16380  5405  690   9078   cgroups    key-users    scsi          vmstat
10822  11801   12470  15084  16384  5407  7     9079   cmdline    kmsg         self          zoneinfo
10823  11805   12472  15099  16387  5436  7342  9080   consoles   kpagecgroup  slabinfo
```

## 1、进程相关文件

`/proc`目录中包含许多以数字命名的子目录，这些数字表示系统当前正在运行进程的进程号，里面包含对应进程相关的多个信息文件。每个数字目录下是当前进程本身相关的信息文件。（以1号初始化进程为例）

```bash
dr-xr-xr-x root 0 Feb 4 09:34 attr
-r-------- root 0 Feb 4 09:34 auxv
-r--r--r-- root 0 Feb 4 09:20 cgroup
--w------- root 0 Feb 4 09:34 clear_refs
-r--r--r-- root 0 Feb 4 09:34 cmdline									# 启动当前进程的完整命令，但僵尸进程目录中的此文件不包含任何信息
-rw-r--r-- root 0 Feb 4 09:34 comm
-rw-r--r-- root 0 Feb 4 09:34 coredump_filter
lrwxrwxrwx root 0 Feb 4 09:34 cwd -> /								# 指向当前进程运行目录的一个符号链接
-r-------- root 0 Feb 4 09:34 environ				# 当前进程的环境变量列表，彼此间用空字符（NULL）隔开；变量用大写字母表示，其值用小写字母表示
lrwxrwxrwx root 0 Feb 4 09:34 exe -> /usr/sbin/init #指向启动当前进程的可执行文件（完整路径）的符号链接，通过/proc/N/exe可以启动当前进程的一个拷贝
dr-x------ root 0 Feb 4 09:34 fd   				 # 这是个目录，包含当前进程打开的每一个文件的文件描述符，这些文件描述符是指向实际文件的一个符号链接； 
dr-x------ root 0 Feb 4 09:34 fdinfo
-r-------- root 0 Feb 4 09:34 io
-r--r--r-- root 0 Feb 4 09:34 limits	# 当前进程所使用的每一个受限资源的软限制、硬限制和管理单元；此文件仅可由实际启动当前进程的UID用户读取
dr-x------ root 0 Feb 4 09:34 map_files
-r--r--r-- root 0 Feb 4 09:34 maps	# 当前进程关联到的每个可执行文件和库文件在内存中的映射区域及其访问权限所组成的列表
-rw------- root 0 Feb 4 09:34 mem		# 当前进程所占用的内存空间，由open、read和lseek等系统调用使用，不能被用户读取； 
-r--r--r-- root 0 Feb 4 09:34 mountinfo
-r--r--r-- root 0 Feb 4 09:34 mounts
-r-------- root 0 Feb 4 09:34 mountstats
dr-xr-xr-x root 0 Feb 4 09:34 net
dr-x--x--x root 0 Feb 4 09:34 ns
-rw-r--r-- root 0 Feb 4 09:34 oom_adj
-r--r--r-- root 0 Feb 4 09:34 oom_score
-rw-r--r-- root 0 Feb 4 09:34 oom_score_adj
-r-------- root 0 Feb 4 09:34 pagemap
-r-------- root 0 Feb 4 09:34 personality
lrwxrwxrwx root 0 Feb 4 09:34 root -> /  # 指向当前进程运行根目录的符号链接；在Unix和Linux系统上，通常采用chroot命令使每个进程运行于独立的根目录
-rw-r--r-- root 0 Feb 4 09:34 sched
-r--r--r-- root 0 Feb 4 09:34 schedstat
-r--r--r-- root 0 Feb 4 09:34 smaps
-r-------- root 0 Feb 4 09:34 stack
-r--r--r-- root 0 Feb 4 09:20 stat		# 当前进程的状态信息，包含一系统格式化后的数据列，可读性差，通常由ps命令使用
-r--r--r-- root 0 Feb 4 09:34 statm		# 当前进程占用内存的状态信息，通常以“页面”（page）表示
-r--r--r-- root 0 Feb 4 09:20 status	# 与stat所提供信息类似，但可读性较好，如下所示，每行表示一个属性信息
-r-------- root 0 Feb 4 09:34 syscall
dr-xr-xr-x root 0 Feb 4 09:34 task	# 包含由当前进程所运行的每一个线程的相关信息，每个线程的相关信息文件均保存在一个由线程号（tid）命名的目录中，这类似于其内容类似于每个进程目录中的内容；（内核2.6版本以后支持此功能） 
-r--r--r-- root 0 Feb 4 09:34 wchan
```

## 2、/proc/sys目录

与/proc下其它文件的“只读”属性不同的是，管理员可对`/proc/sys`子目录中的许多文件内容进行修改以更改内核的运行特性，事先查看某文件是否“可写入”。写入操作通常使用类似于`echo DATA > /path/to/your/filename`的格式进行。需要注意的是，即使文件可写，其一般也不可以使用编辑器进行编辑。 

## 3、其他目录文件

- **/proc/apm**：高级电源管理（APM）版本信息及电池相关状态信息，通常由apm命令使用； 

- **/proc/buddyinfo**：用于诊断内存碎片问题的相关信息文件； 

- **/proc/cmdline**：在启动时传递至内核的相关参数信息，这些信息通常由lilo或grub等启动管理工具进行传递； 

  ```bash
  syno_hdd_powerup_seq=1 HddHotplug=0 syno_hw_version=DS918+ vender_format_version=2 console=ttyS0,115200n8 withefi quiet syno_hdd_detect=0 root=/dev/md0 sn=1780PDN998701 mac1=001132112233 mac2=001132112231 netif_num=2
  ```


- **/proc/cpuinfo**：处理器的相关信息的文件； 

- **/proc/crypto**：系统上已安装的内核使用的密码算法及每个算法的详细信息列表； 

  ```bash
  name : crc32c
  driver : crc32c-generic
  module : kernel
  priority : 0
  type : digest
  blocksize : 32
  digestsize : 4
  …………
  ```


- **/proc/devices**：系统已经加载的所有块设备和字符设备的信息，包含主设备号和设备组（与主设备号对应的设备类型）名； 

  ```bash
  Character devices:
  1 mem
  4 /dev/vc/0
  4 tty
  4 ttyS
  …………
  Block devices:
  1 ramdisk
  2 fd
  8 sd
  …………
  ```


- **/proc/diskstats**：每块磁盘设备的磁盘I/O统计信息列表；（内核2.5.69以后的版本支持此功能） 

- **/proc/dma**：每个正在使用且注册的ISA DMA通道的信息列表； 


- **/proc/execdomains**：内核当前支持的执行域（每种操作系统独特“个性”）信息列表； 


- **/proc/fb**：帧缓冲设备列表文件，包含帧缓冲设备的设备号和相关驱动信息； 

- **/proc/filesystems**：当前被内核支持的文件系统类型列表文件，被标示为nodev的文件系统表示不需要块设备的支持；通常mount一个设备时，如果没有指定文件系统类型将通过此文件来决定其所需文件系统的类型；

  ```bash
  nodev sysfs
  nodev rootfs
  nodev proc
  iso9660
  ext3
  …………
  ```


- **/proc/interrupts**：X86或X86_64体系架构系统上每个IRQ相关的中断号列表；多路处理器平台上每个CPU对于每个I/O设备均有自己的中断号； 

  ```bash
              CPU0       CPU1       CPU2       CPU3
     0:         11          0          0          0   IO-APIC    2-edge      timer
     3:          0          0          0          0   IO-APIC    3-edge      serial
     4:        206          0          0          0   IO-APIC    4-edge      serial
     8:          0          0          0          0   IO-APIC    8-fasteoi   rtc0
     9:          0          0          0          0   IO-APIC    9-fasteoi   acpi
    87:       7594      15750       1492       9913   PCI-MSI 311296-edge      0000:00:13.0
    88:       3137          0      67388          0   PCI-MSI 1048576-edge      0000:02:00.0
    89:         34          0          0          0   PCI-MSI 327680-edge      xhci_hcd
    90:          1          0          0          0   PCI-MSI 1572864-edge      eth0
    91:         67          0         13      38093   PCI-MSI 1572865-edge      eth0-rx-0
    92:       1058        715        235        881   PCI-MSI 1572866-edge      eth0-rx-1
    93:       5258          0        470       1026   PCI-MSI 1572867-edge      eth0-tx-0
    94:       1949          0          0      36357   PCI-MSI 1572868-edge      eth0-tx-1
    95:        166          0          0          0   PCI-MSI 32768-edge      i915
   NMI:        102         96         80         81   Non-maskable interrupts
   LOC:    3399022    3276720    3075216    3057331   Local timer interrupts
   SPU:          0          0          0          0   Spurious interrupts
   PMI:        102         96         80         81   Performance monitoring interrupts
  ```


- **/proc/iomem**：每个物理设备上的记忆体（RAM或者ROM）在系统内存中的映射信息； 

  ```bash
  00000100-00000fff : reserved
  00001000-0003efff : System RAM
  0003f000-0003ffff : ACPI Non-volatile Storage
  00040000-0009ffff : System RAM
  000a0000-000bffff : PCI Bus 0000:00
  000c0000-000dffff : PCI Bus 0000:00
    000c0000-000c7fff : Video ROM
  ```


- **/proc/ioports**：当前正在使用且已经注册过的与物理设备进行通讯的输入-输出端口范围信息列表；如下面所示，第一列表示注册的I/O端口范围，其后表示相关的设备； 

  ```bash
  0000-001f : dma1
  0020-0021 : pic1
  0040-0043 : timer0
  0050-0053 : timer1
  0060-006f : keyboard
  ```


- **/proc/kallsyms**：模块管理工具用来动态链接或绑定可装载模块的符号定义，由内核输出；（内核2.5.71以后的版本支持此功能）；通常这个文件中的信息量相当大；


- **/proc/kcore**：系统使用的物理内存，以ELF核心文件（core file）格式存储，其文件大小为已使用的物理内存（RAM）加上4KB；这个文件用来检查内核数据结构的当前状态，因此，通常由GBD通常调试工具使用，但不能使用文件查看命令打开此文件； 

- **/proc/kmsg**：此文件用来保存由内核输出的信息，通常由/sbin/klogd或/bin/dmsg等程序使用，不要试图使用查看命令打开此文件； 


- **/proc/locks**：保存当前由内核锁定的文件的相关信息，包含内核内部的调试数据；每个锁定占据一行，且具有一个惟一的编号；如下输出信息中每行的第二列表示当前锁定使用的锁定类别，POSIX表示目前较新类型的文件锁，由lockf系统调用产生，FLOCK是传统的UNIX文件锁，由flock系统调用产生；第三列也通常由两种类型，ADVISORY表示不允许其他用户锁定此文件，但允许读取，MANDATORY表示此文件锁定期间不允许其他用户任何形式的访问； 

  ```bash
  1: POSIX  ADVISORY  WRITE 20975 00:1f:9017052 0 EOF
  2: POSIX  ADVISORY  WRITE 20975 00:1f:9016783 0 EOF
  3: POSIX  ADVISORY  WRITE 20975 00:1f:9016653 0 EOF
  4: POSIX  ADVISORY  WRITE 20975 00:1f:9016501 0 EOF
  5: POSIX  ADVISORY  WRITE 20975 00:1f:9016442 0 EOF
  6: POSIX  ADVISORY  WRITE 20975 00:1f:9016361 0 EOF
  7: POSIX  ADVISORY  WRITE 20975 00:1f:9016114 0 EOF
  8: POSIX  ADVISORY  WRITE 20975 00:1f:9016042 0 EOF
  9: POSIX  ADVISORY  WRITE 20975 00:1f:9015963 0 EOF
  10: POSIX  ADVISORY  WRITE 20975 00:1f:28255 0 EOF
  ```


- **/proc/mdstat**：保存RAID相关的多块磁盘的当前状态信息，在没有使用RAID机器上，其显示`unused devices: <none>`： 

  ```bash
  Personalities : [linear] [raid0] [raid1] [raid10] [raid6] [raid5] [raid4]
  md4 : active raid1 sdf5[0]
        483555456 blocks super 1.2 [1/1] [U]
  
  md2 : active raid1 sdb3[0]
        10094080 blocks super 1.2 [1/1] [U]
  
  md3 : active raid1 sde3[0]
        971940544 blocks super 1.2 [1/1] [U]
  
  md1 : active raid1 sdb2[0] sde2[1] sdf2[2]
        2097088 blocks [16/3] [UUU_____________]
  
  md0 : active raid1 sdb1[0] sde1[1] sdf1[2]
        2490176 blocks [16/3] [UUU_____________]
  unused devices: <none>
  ```


- **/proc/meminfo**：系统中关于当前内存的利用状况等的信息，常由free命令使用；可以使用文件查看命令直接读取此文件，其内容显示为两列，前者为统计属性，后者为对应的值； 

  ```bash
  MemTotal:        8058608 kB
  MemFree:          138636 kB
  MemAvailable:     484704 kB
  Buffers:            6000 kB
  Cached:           590724 kB
  SwapCached:        68220 kB
  Active:          4024568 kB
  Inactive:        3306388 kB
  Active(anon):    3719356 kB
  Inactive(anon):  3106256 kB
  Active(file):     305212 kB
  Inactive(file):   200132 kB
  Unevictable:        2528 kB
  Mlocked:            2528 kB
  SwapTotal:       6930348 kB
  SwapFree:        6119948 kB
  Dirty:               624 kB
  Writeback:             0 kB
  AnonPages:       6706328 kB
  Mapped:           223620 kB
  Shmem:             91104 kB
  Slab:             208120 kB
  SReclaimable:     113072 kB
  SUnreclaim:        95048 kB
  KernelStack:       28144 kB
  PageTables:        81552 kB
  NFS_Unstable:          0 kB
  Bounce:                0 kB
  WritebackTmp:          0 kB
  CommitLimit:    10959652 kB
  Committed_AS:   21421636 kB
  VmallocTotal:   34359738367 kB
  VmallocUsed:           0 kB
  VmallocChunk:          0 kB
  DirectMap4k:       14124 kB
  DirectMap2M:     8263680 kB
  ```


- **/proc/mounts**：在内核2.4.29版本以前，此文件的内容为系统当前挂载的所有文件系统，在2.4.19以后的内核中引进了每个进程使用独立挂载名称空间的方式，此文件则随之变成了指向/proc/self/mounts（每个进程自身挂载名称空间中的所有挂载点列表）文件的符号链接；/proc/self是一个独特的目录。如下所示，其中第一列表示挂载的设备，第二列表示在当前目录树中的挂载点，第三点表示当前文件系统的类型，第四列表示挂载属性（ro或者rw），第五列和第六列用来匹配/etc/mtab文件中的转储（dump）属性； 

  ```bash
  /dev/md0 / ext4 rw,relatime,barrier,data=ordered 0 0
  none /dev devtmpfs rw,nosuid,noexec,relatime,size=4016084k,nr_inodes=1004021,mode=755 0 0
  none /dev/pts devpts rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000 0 0
  none /proc proc rw,nosuid,nodev,noexec,relatime 0 0
  none /sys sysfs rw,nosuid,nodev,noexec,relatime 0 0
  /tmp /tmp tmpfs rw,relatime 0 0
  /run /run tmpfs rw,nosuid,nodev,relatime,mode=755 0 0
  /dev/shm /dev/shm tmpfs rw,nosuid,nodev,relatime 0 0
  none /sys/fs/cgroup tmpfs rw,relatime,size=4k,mode=755 0 0
  cgmfs /run/cgmanager/fs tmpfs rw,relatime,size=100k,mode=755 0 0
  cgroup /sys/fs/cgroup/cpuset cgroup rw,relatime,cpuset,release_agent=/run/cgmanager/agents/cgm-release-agent.cpuset,clone_children 0 0
  cgroup /sys/fs/cgroup/cpu cgroup rw,relatime,cpu,release_agent=/run/cgmanager/agents/cgm-release-agent.cpu 0 0
  cgroup /sys/fs/cgroup/cpuacct cgroup rw,relatime,cpuacct,release_agent=/run/cgmanager/agents/cgm-release-agent.cpuacct 0 0
  cgroup /sys/fs/cgroup/blkio cgroup rw,relatime,blkio,release_agent=/run/cgmanager/agents/cgm-release-agent.blkio 0 0
  cgroup /sys/fs/cgroup/memory cgroup rw,relatime,memory,release_agent=/run/cgmanager/agents/cgm-release-agent.memory 0 0
  cgroup /sys/fs/cgroup/devices cgroup rw,relatime,devices,release_agent=/run/cgmanager/agents/cgm-release-agent.devices 0 0
  cgroup /sys/fs/cgroup/freezer cgroup rw,relatime,freezer,release_agent=/run/cgmanager/agents/cgm-release-agent.freezer 0 0
  none /proc/bus/usb devtmpfs rw,nosuid,noexec,relatime,size=4016084k,nr_inodes=1004021,mode=755 0 0
  none /sys/kernel/debug debugfs rw,relatime 0 0
  securityfs /sys/kernel/security securityfs rw,relatime 0 0
  /dev/md2 /volume1 btrfs rw,relatime,synoacl,space_cache=v2,auto_reclaim_space,metadata_ratio=50,subvolid=257,subvol=/@syno 0 0
  /dev/vg1000/lv /volume3 btrfs rw,relatime,relatime_period=30,synoacl,space_cache=v2,auto_reclaim_space,metadata_ratio=50,subvolid=257,subvol=/@syno 0 0
  ```


- **/proc/modules**：当前装入内核的所有模块名称列表，可以由lsmod命令使用，也可以直接查看；如下所示，其中第一列表示模块名，第二列表示此模块占用内存空间大小，第三列表示此模块有多少实例被装入，第四列表示此模块依赖于其它哪些模块，第五列表示此模块的装载状态（Live：已经装入；Loading：正在装入；Unloading：正在卸载），第六列表示此模块在内核内存（kernel memory）中的偏移量；

  ```bash
  xt_ipvs 2202 0 - Live 0xffffffffa0fe0000
  ip_vs_rr 1447 0 - Live 0xffffffffa0fdc000
  ip_vs 127371 3 xt_ipvs,ip_vs_rr, Live 0xffffffffa0fb3000
  xt_mark 1317 0 - Live 0xffffffffa0faf000
  iptable_mangle 1656 0 - Live 0xffffffffa0fab000
  br_netfilter 13589 0 - Live 0xffffffffa0fa3000
  bridge 55340 1 br_netfilter, Live 0xffffffffa0f8d000
  stp 1693 1 bridge, Live 0xffffffffa0f89000
  aufs 194415 0 - Live 0xffffffffa0f4d000
  macvlan 13776 0 - Live 0xffffffffa0f45000
  veth 5094 0 - Live 0xffffffffa0f40000
  xt_conntrack 3401 5 - Live 0xffffffffa0f3c000
  xt_addrtype 2893 1 - Live 0xffffffffa0f38000
  ipt_MASQUERADE 1213 25 - Live 0xffffffffa0f34000
  xt_REDIRECT 1486 0 - Live 0xffffffffa0f30000
  ```


- **/proc/partitions**：块设备每个分区的主设备号（major）和次设备号（minor）等信息，同时包括每个分区所包含的块（block）数目

  ```bash
  major minor  #blocks  name
  
     1        0     655360 ram0
     1        1     655360 ram1
     1        2     655360 ram2
     1        3     655360 ram3
     1        4     655360 ram4
     1        5     655360 ram5
     1        6     655360 ram6
     1        7     655360 ram7
     1        8     655360 ram8
     1        9     655360 ram9
     1       10     655360 ram10
     1       11     655360 ram11
     1       12     655360 ram12
     1       13     655360 ram13
     1       14     655360 ram14
     1       15     655360 ram15
     8       16   15638616 sdb
     8       17    2490240 sdb1
     8       18    2097152 sdb2
     8       19   10095104 sdb3
     8       20      65536 sdb4
     8       21      64512 sdb5
  ```


- **/proc/slabinfo** :在内核中频繁使用的对象（如inode、dentry等）都有自己的cache，即slab pool，而/proc/slabinfo文件列出了这些对象相关slap的信息


- **/proc/stat**：实时追踪自系统上次启动以来的多种统计信息；

  - “cpu”行后的八个值分别表示以1/100（jiffies）秒为单位的统计值（包括系统运行于用户模式、低优先级用户模式，运系统模式、空闲模式、I/O等待模式的时间等）； 
  - “intr”行给出中断的信息，第一个为自系统启动以来，发生的所有的中断的次数；然后每个数对应一个特定的中断自系统启动以来所发生的次数； 
  - “ctxt”给出了自系统启动以来CPU发生的上下文交换的次数。 
  - “btime”给出了从系统启动到现在为止的时间，单位为秒； 
  - “processes (total_forks) 自系统启动以来所创建的任务的个数； 
  - “procs_running”：当前运行队列的任务的数目； 
  - “procs_blocked”：当前被阻塞的任务的数目；

  ```bash
  cpu  223139 48 44015 1586826 69922 0 2477 0 0 0
  cpu0 56431 23 11065 395817 17523 0 649 0 0 0
  cpu1 54109 10 11450 397915 17649 0 596 0 0 0
  cpu2 55572 10 10954 397559 16719 0 695 0 0 0
  cpu3 57026 3 10544 395533 18031 0 535 0 0 0
  intr 12065678 11 0 0 0 206 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 31886 66307 34 1 32453 2413 5989 32388 166 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 
  ctxt 20744413
  btime 1612399810
  processes 37878
  procs_running 1
  procs_blocked 0
  softirq 10803666 2 4776615 1052 818173 113621 0 2279 1548289 0 3543635
  ```

- **/proc/swaps**：当前系统上的交换分区及其空间利用信息，如果有多个交换分区的话，则会每个交换分区的信息分别存储于/proc/swap目录中的单独文件中，而其优先级数字越低，被使用到的可能性越大

  ```bash
  Filename				Type		Size	  Used	Priority
  /dev/md1      partition	2097084	0     	-1
  /dev/zram0    partition	1208316	207996	1
  /dev/zram1    partition	1208316	207660	1
  /dev/zram2    partition	1208316	207612	1
  /dev/zram3    partition	1208316	207976	1
  ```


- **/proc/uptime**：系统上次启动以来的运行时间，如下所示，其第一个数字表示系统运行时间，第二个数字表示系统空闲时间，单位是秒； 

  ```bash
  4578.13 14964.34
  ```


- **/proc/version**：当前系统运行的内核版本号

  ```bash
  Linux version 4.4.59+ (root@build3) (gcc version 4.9.3 20150311 (prerelease) (crosstool-NG 1.20.0) ) #24922 SMP PREEMPT Thu Mar 28 11:07:03 CST 2019
  ```


- **/proc/vmstat**：当前系统虚拟内存的多种统计数据，信息量可能会比较大，这因系统而有所不同，可读性较好（2.6以后的内核支持此文件）

  ```bash
  nr_anon_pages 22270
  nr_mapped 8542
  nr_file_pages 47706
  nr_slab 4720
  nr_page_table_pages 897
  nr_dirty 21
  nr_writeback 0
  …………
  ```


- **/proc/zoneinfo**：内存区域（zone）的详细信息列表

  ```bash
  Node 0, zone DMA
  pages free 1208
  min 28
  low 35
  high 42
  active 439
  inactive 1139
  scanned 0 (a: 7 i: 30)
  spanned 4096
  present 4096
  nr_anon_pages 192
  nr_mapped 141
  nr_file_pages 1385
  nr_slab 253
  nr_page_table_pages 2
  nr_dirty 523
  nr_writeback 0
  nr_unstable 0
  nr_bounce 0
  protection: (0, 0, 296, 296)
  pagesets
  all_unreclaimable: 0
  prev_priority: 12
  start_pfn: 0
  …………
  ```

  

# 参考

- http://bbs.chinaunix.net/thread-2002769-1-1.html