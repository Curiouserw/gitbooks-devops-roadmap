# 硬盘格式分区及文件系统

# MBR和GPT

## 创建MBR 分区

- **MacOS diskutil**

  ```bash
  # macOS 下（diskutil 交互方式，生成空分区）
  # 创建新 MBR 分区（ext4 起点 2048）
  sudo diskutil partitionDisk /dev/disk4 MBR "Free Space" TEMP 0g
  # TEMP 占位，不要关心文件系统
  # 确保分区起始扇区 = 2048（macOS 默认会 1MiB 对齐）
  ```

- linux

## 清除MBR和GPT

- **MacOS diskutil**

  ```bash
  # 删除 GPT、MBR、EFI分区,只留下一个“裸盘”
  sudo diskutil eraseDisk free none /dev/disk4
  ```

- **dd**

  ```bash
  dd if=/dev/zero of=/dev/sda bs=1M count=10 status=progress conv=fsync
  ```

# ext4

- mkfs.ext4

  ```bash
  mkfs.ext4 \
    -O ^has_journal \
    -L sd \
    -E lazy_itable_init=0,lazy_journal_init=0 \
    -m 0 \
    /dev/sda1
    
  # ^has_journal        禁用ext4的日志功能journal
  # lazy_itable_init=0  立即初始化 inode 表（避免后台慢初始化）
  # lazy_journal_init=0 虽然没日志，但防止残留相关延迟行为
  # -m 0                不保留 5% root 空间（路由器不需要）
  ```
  
  ```bash
  # 禁用索引（减少写入）
  tune2fs -O ^dir_index /dev/sda1
  ```

# NTFS

```bash
# mkntfs命令是ntfs-3g项目的用户态工具
/usr/sbin/mkntfs -f -L sd /dev/sda1

# -L <label>	卷标，等同 Windows 卷标
# -v  显示版本  
# -q  安静模式  脚本中常用
# -f  快速格式  强烈建议
# --debug  调试输出  一般不用
```

# FAT32/FAT16

```bash
mkdosfs -F 32 -s 64 -n sd /dev/sda1

# -f <n>  FAT 表数量（默认 2）
# -r <n>  根目录项数（FAT12/16）
# -s <n>  每簇扇区数（USB盘可调）
# -S <n>  扇区大小（⚠️ 一般不动）
# -R <n>  保留扇区数（少用）
# -n <label>  卷标
# -i <id>  Volume ID（十六进制）
```

# 分区

- fdisk

  ```bash
  (
  echo o
  echo n
  echo p
  echo 1
  echo 2048
  echo
  echo
  echo w
  ) | fdisk /dev/sda
  
  # p       # 打印当前分区表
  # n       # 新建分区，如果存在旧分区，此时先 d 删除分区
  # 1       # 分区号
  # 2048    # 设置新分区起始扇区为 2048（1MiB 对齐，保证 4K 对齐）
  # <回车> # 使用剩余空间
  # t 83    # 设置类型为 Linux
  # w       # 写入表并退出
  ```

- parted

  ```bash
  parted /dev/sda --script \
    mklabel gpt \
    mkpart primary ext4 1MiB 100%
    
  # 1 MiB = 2048 × 512B
  ```

# 验证

- 查看标签和 uuid：`blkid /dev/sda1`
- 查看内核态是否 4K 对齐：
  - `cat /sys/block/sda/sda1/start`
  - `blockdev --getalignoff /dev/sda1`
  - `tune2fs -l /dev/sda1 | grep -i journal`
- 查看ext4日志设置和 inode保留空间：`tune2fs -l /dev/sda1 | grep -E 'has_journal|Reserved block'`
-  查看USB电流限制： `/sys/bus/usb/devices/usb*/power/current_limit`
- 查看逻辑扇区大小 (一般为512,Start×512能被4096整除OK) ：
  - `blockdev --getss /dev/sda`
  - `cat /sys/block/sda/queue/logical_block_size`
- 查看物理块大小（通常 4096）：
  - `blockdev --getpbsz /dev/sda `
  - `cat /sys/block/sda/queue/physical_block_size`

# 性能测试

```bash
dd if=/dev/zero of=/tmp/mnt/sd/test.bin bs=1M count=100 conv=fdatasync
```

# 挂载

- 挂载参数

  ```bash
  mount -o remount,noatime,nodiratime,commit=600 /tmp/mnt/sd
  
  noatime,nodiratime,commit=600
  # noatime,nodiratime	禁止访问时间写入
  # commit=600	每 10 分钟刷一次 metadata
  #（无日志）	commit 只影响 metadata cache
  
  noatime,nodiratime      # 不更新访问时间，减少写入
  data=writeback          # 写回模式，性能最佳
  commit=60               # 60秒同步一次（默认5秒）
  barrier=0               # 禁用屏障（适合带电容的U盘）
  errors=remount-ro       # 错误时只读挂载
  ```

- 查看挂载状态：`lsblk -o NAME,PHY-SEC,LOG-SEC,MOUNTPOINT /dev/sda`

# 重新读取分区表

```bash
partprobe /dev/sda
blockdev --rereadpt /dev/sda
echo 1 > /sys/block/sda/device/rescan
```
