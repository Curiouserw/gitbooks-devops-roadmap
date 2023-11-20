# Synology

# 一、HSynology安装

- https://cndl.synology.cn/download/DSM/release/6.2.3/25426/DSM_DS918%2B_25426.pat

- https://www.31du.cn/blog/ds3617.html

# 二、系统管理

群辉中的系统命令或者套间命令大部分是以syno开头。在DSM的 “套件中心” 安装的软件包是以`pkgctl-`为前缀命名的

```bash
# 列出系统运行的服务
synoservice synoservicecfg --list
# 重启系统服务
synoservice --restart pkgctl-Docker
```

# 三、其他操作

## 1、恢复卸载硬盘无法识别加入存储池的数据

**原因：**DSM 显示磁盘损坏。重启后直接不认磁盘。插入Windows 电脑也无法挂载磁盘。磁盘仅有部分坏块，使用 Diskgenues 无法加载文件系统，仅能查看出分区被 raid设置过。

**解决方案：**在 Linux 系统中直接挂载磁盘分区进行数据迁移。可使用`fdisk -l或cat /proc/mdstat或 lvs`查看磁盘设备路径。

至于之前DSM 设置加密过磁盘分区（需要保存有加密存储空间的恢复密钥文件）：

```bash
apt-get install cryptsetup
# 解码恢复密钥
base64 --decode ${加密存储空间的恢复密钥文件路径} > ${base64_decode_output_path}
# 测试恢复密钥是否正确
cryptsetup open --test-passphrase /dev/vgX/volume_Y -S 1 -d ${base64_decode_output_path}
# 使用解码的恢复密钥为加密存储空间解密
cryptsetup open --allow-discards /dev/vgX/volume_Y cryptvol_Y -S 1 -d ${base64_decode_output_path}
# 挂载磁盘分区
mount ${device_path} ${mount_point} -o ro
```

参考：https://kb.synology.cn/zh-cn/DSM/tutorial/How_can_I_recover_data_from_my_DiskStation_using_a_PC

## 2、扩充分区容量

- 群辉系统盘使用分区进行 raid 组盘的
- fdisk只支持MBR分区，MBR分区表最大支撑2T的磁盘，所以无法划分大于2T的分区。而parted工具可以划分单个分区大于2T的GPT格式的分区，也可以划分普通的MBR分区。

```bash
# 使用 parted 查看所有磁盘分区的详细情况以及其上的文件系统
$ parted -l
Model: SSD 128GB (scsi)
Disk /dev/sdb: 128GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:
  Number  Start   End     Size    File system     Name        Flags
   1      1049kB  2551MB  2550MB  ext4            Linux RAID  raid
   2      2551MB  4699MB  2147MB  linux-swap(v1)  Linux RAID  raid
   3      4699MB  4766MB  67.1MB  fat32           primary     boot, esp
   4      4766MB  4832MB  66.1MB  fat16           primary
   5      4832MB  128GB   123GB                   Linux RAID  raid

Model: TOSHIBA HDWD110 (scsi)
Disk /dev/sdf: 1000GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:
  Number  Start   End     Size    Type     File system  Flags
   1      1049kB  2551MB  2550MB  primary               raid
   2      2551MB  4699MB  2147MB  primary               raid
   3      4832MB  1000GB  995GB   primary               raid
# ---------Raid组---------
Model: Linux Software RAID Array (md)
Disk /dev/md0: 2550MB
Sector size (logical/physical): 512B/512B
Partition Table: loop
Disk Flags:
  Number  Start  End     Size    File system  Flags
   1      0.00B  2550MB  2550MB  ext4
# ---------Raid组，分区/dev/md1，交互分区文件系统swap---------
Model: Linux Software RAID Array (md)
Disk /dev/md1: 2147MB
Sector size (logical/physical): 512B/512B
Partition Table: loop
Disk Flags:
  Number  Start  End     Size    File system     Flags
   1      0.00B  2147MB  2147MB  linux-swap(v1)
# ---------Raid组，分区/dev/md2，文件系统btrfs ---------
Model: Linux Software RAID Array (md)
Disk /dev/md2: 123GB
Sector size (logical/physical): 512B/512B
Partition Table: loop
Disk Flags:
  Number  Start  End     Size    File system  Flags
   1      0.00B  9.63GB  9.63GB  btrfs
# ---------Raid组，分区/dev/md3，文件系统btrfs---------
Model: Linux Software RAID Array (md)
Disk /dev/md3: 995GB
Sector size (logical/physical): 512B/512B
Partition Table: loop
Disk Flags:
  Number  Start  End    Size   File system  Flags
   1      0.00B  995GB  995GB  btrfs
 
# 查看分区挂载情况
$ df -mh
Filesystem      Size  Used Avail Use% Mounted on
/dev/md0        2.3G  1.4G  793M  65% /
none            3.9G     0  3.9G   0% /dev
/tmp            3.9G  1.2M  3.9G   1% /tmp
/run            3.9G  3.7M  3.9G   1% /run
/dev/shm        3.9G  4.0K  3.9G   1% /dev/shm
none            4.0K     0  4.0K   0% /sys/fs/cgroup
cgmfs           100K     0  100K   0% /run/cgmanager/fs
/dev/md2        9.3G  2.0G  6.9G  22% /volume1
/dev/md3        890G  740G  151G  84% /volume2

# 从上述命令得知群辉的用户文件系统所在分区/dev/md2挂载到/volume1。要进行扩容的分区正是/volume1。
# 使用RAID磁盘阵列管理命令查看raid分区的详细情况
$ mdadm -D /dev/md2
/dev/md2:
        Version : 1.2
  Creation Time : Sat Apr 20 14:27:24 2019
     Raid Level : raid1
     Array Size : 10094080 (9.63 GiB 10.34 GB)
  Used Dev Size : 10094080 (9.63 GiB 10.34 GB)
   Raid Devices : 1
  Total Devices : 1
    Persistence : Superblock is persistent
    Update Time : Mon Nov 13 13:11:43 2023
          State : clean
 Active Devices : 1
Working Devices : 1
 Failed Devices : 0
  Spare Devices : 0
           Name : Synology:2
           UUID : 4a0fadf4:7e52e674:f2202100:91789994
         Events : 2137
    Number   Major   Minor   RaidDevice State
       0       8       21        0      active sync   /dev/sdb5
# 从上述命令得知分区所在磁盘为/dev/sdb。使用 fdisk 查看磁盘的分区详细情况
$ fdisk /dev/sdb
Command (m for help): p
Disk /dev/sdb: 119.2 GiB, 128035676160 bytes, 250069680 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 0D7DE78D-A648-4E1D-8FE2-E9FB24CAFA38

Device       Start      End  Sectors  Size Type
/dev/sdb1     2048  4982527  4980480  2.4G Linux RAID
/dev/sdb2  4982528  9176831  4194304    2G Linux RAID
/dev/sdb3  9177088  9308159   131072   64M EFI System
/dev/sdb4  9308160  9437183   129024   63M Linux filesystem
/dev/sdb5  9437184 29627391 20190208  9.6G Linux RAID

# 查看得知还有 250069680-20190208=229879472 个扇区没有使用，大约117698289664*512=117698289664 Bytes个字节大小，约117698289664/1024/1024/1024=109.61507416 GB大小
```

扩容分区容量（命令中的5，对应/dev/sdb5）

```bash
parted /dev/sdb resizepart 5 100%
```

在群辉存储池管理中进行手动扩容。

```bash
$ df -mh
Filesystem      Size  Used Avail Use% Mounted on
/dev/md0        2.3G  1.4G  793M  65% /
none            3.9G     0  3.9G   0% /dev
/tmp            3.9G  1.2M  3.9G   1% /tmp
/run            3.9G  3.7M  3.9G   1% /run
/dev/shm        3.9G  4.0K  3.9G   1% /dev/shm
none            4.0K     0  4.0K   0% /sys/fs/cgroup
cgmfs           100K     0  100K   0% /run/cgmanager/fs
/dev/md2        111G  1.9G  109G   2% /volume1
/dev/md3        890G  740G  151G  84% /volume2
```
