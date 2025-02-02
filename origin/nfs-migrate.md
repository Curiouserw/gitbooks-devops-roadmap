# NAS数据迁移

# 一、简介

## 阿里云NAS简介

- 通用型NAS用于存储频繁访问的热数据，提供三种存储规格，分别为容量型、高级型和性能型。
- 通用型NAS：支持NFS v3和NFS v4.0

| **项目**         | **容量型**                                                   | **高级型**                                                   | **性能型**                                                   |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 吞吐（峰值）     | 初始读带宽150 MB/s，每GiB增加0.15 MB/s，读上限为10 GB/s，写上限为5 GB/s。 | 初始读带宽300 MB/s，每GiB增加0.3 MB/s，读上限为20 GB/s，写上限为5 GB/s。 | 初始读带宽600 MB/s，每GiB增加0.6 MB/s，读上限为20 GB/s，写上限为5 GB/s。 |
| IOPS             | 上限为15000。                                                | 上限为30000。                                                | 上限为30000。                                                |
| 平均单路4k读延迟 | 10 ms                                                        | 2 ms                                                         | 2 ms                                                         |
| 平均单路4k写延迟 | 10 ms                                                        | 2 ms                                                         | 2 ms                                                         |
| 容量             | 0~10 PiB                                                     | 0~1 PiB                                                      | 0~1 PiB                                                      |
| 扩容步长         | 4 KiB                                                        | 4 KiB                                                        | 4 KiB                                                        |
| 扩容方式         | 自动扩容                                                     | 自动扩容                                                     | 自动扩容                                                     |

## NFS服务端请求类型

- **文件操作请求**

  - CREATE：创建文件

  - REMOVE：删除文件

  - READ：读取文件内容

  - WRITE：写入文件内容

  - LOOKUP：查找目录中的文件或子目录

  - MKDIR：创建目录

  - RMDIR：删除目录

  - READDIR：读取目录内容

  - READDIRPLUS（NFSv3+）：扩展的目录读取操作，返回文件的额外属性信息

  - ACCESS：检查文件或目录的访问权限
- **属性操作请求**：用于获取或设置文件和目录的元数据信息

  - GETATTR：获取文件或目录的属性，例如大小、时间戳、权限等

  - SETATTR：设置文件或目录的属性，例如修改时间、访问权限等


-  **句柄相关请求**：句柄来标识文件和目录，以下请求与文件句柄操作相关

  - FSINFO：获取文件系统的信息，例如支持的属性和限制

  - FSSTAT：获取文件系统的统计信息，例如剩余空间、总容量等

  - PATHCONF：获取与路径相关的配置信息，例如最大文件名长度

  - STATFS：检查文件系统的状态和容量信息


- **文件锁定请求**：NFSv4 引入了状态化协议，NFSv4 特有。支持对文件的锁定操作

  - LOCK：对文件的某个范围加锁

  - LOCKT：测试某个范围是否可以加锁

  - LOCKU：释放文件的锁

  - OPEN：打开文件并获取锁

  - CLOSE：关闭文件并释放锁


- **数据缓存相关请求**：用于提高性能，通过管理缓存来减少重复操作

  - COMMIT：将之前的写操作从客户端的缓存写入到服务端的磁盘

  - NOOP（NFSv4 特有）：空操作，通常用于保持会话活跃


- **文件系统操作请求**：管理整个文件系统的操作

  - MOUNT：挂载远程文件系统（通过 RPC）

  - UMOUNT：卸载远程文件系统（通过 RPC）
  - EXPORT：列出服务端导出的目录
  - NULL：空操作，用于测试连接或服务是否可用


- **符号链接和硬链接请求**

  - SYMLINK：创建符号链接

  - READLINK：读取符号链接的目标路径

  - LINK：创建硬链接


- **其他请求类型**

  - RENAME：重命名文件或目录

  - REMOVE：删除文件

  - CLONE（部分实现支持）：克隆文件以节省存储空间


# 二、方案：读写分离+并发



## 1、挂载优化

- 内核升级至高于5.4

- 使用**NFSv3**协议挂载

  - **NFSv3**：无状态协议，性能高，但安全性较低。

  - **NFSv4**：状态管理，增强了安全性（如内置支持 Kerberos 身份验证）。服务端需要确保 NFSv4 相关配置正确

- 其他挂载参数

  - **rsize**：定义数据块的大小，用于客户端与文件系统之间读取数据。建议值：1048576。
  - **wsize**：定义数据块的大小，用于客户端与文件系统之间写入数据。建议值：1048576。

  - **hard**：在文件存储NAS暂时不可用的情况下，使用文件系统上某个文件的本地应用程序时会停止并等待至该文件系统恢复在线状态。建议启用
  - **timeo**：指定时长，单位为0.1秒，即NFS客户端在重试向文件系统发送请求之前等待响应的时间。建议值：600（60秒）。

  - **retrans**：NFS客户端重试请求的次数。建议值：2。
  - **noresvport**：在网络重连时使用新的TCP端口，保障在网络发生故障恢复时不会中断连接。建议启用该参数。

  https://help.aliyun.com/zh/nas/user-guide/mount-an-nfs-file-system-on-a-linux-ecs-instance?spm=a2c4g.11186623.0.0.5db964e8Id90bh#section-jyi-hyd-hbr

- 修改内核默认NFS请求数量限制，2 --> 128

  ```bash
  if (lsmod | grep sunrpc); then
    (modinfo sunrpc | grep tcp_max_slot_table_entries) && sysctl -w sunrpc.tcp_max_slot_table_entries=128
    (modinfo sunrpc | grep tcp_slot_table_entries) && sysctl -w sunrpc.tcp_slot_table_entries=128
  fi
  
  (modinfo sunrpc | grep tcp_max_slot_table_entries) && echo "options sunrpc tcp_max_slot_table_entries=128" >> /etc/modprobe.d/sunrpc.conf
  (modinfo sunrpc | grep tcp_slot_table_entries) && echo "options sunrpc tcp_slot_table_entries=128" >> /etc/modprobe.d/sunrpc.conf
  ```


```bash
mount -t nfs \
   -o vers=3,nolock,proto=tcp,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport \
   nas.aliyuncs.com:/ /mnt
```

- https://help.aliyun.com/zh/nas/user-guide/other-issues?spm=a2c4g.11186623.0.0.7d361f34qw1GxY#section-o97-87k-mdw

- https://help.aliyun.com/zh/nas/user-guide/faq-about-the-performance-of-nas-file-systems?spm=a2c4g.11186623.0.0.23aa3898UuKxL8#section-urf-b97-l23



```bash
telnet $mount_point.$region.nas.aliyuncs.com 2049
telnet $mount_point.$region.nas.aliyuncs.com 111
```

## 2、统计NFS 文件数据

文件目录结构

```bash
/ns-data
|---文件
|---文件夹1
    |-----文件夹1-1
    |-----文件夹1-2
    |----- .....
|---文件夹2
    |-----文件夹2-1
    |-----文件夹2-2
    |----- .....
```



|             | du -sh | find+6个并发du                               |
| ----------- | ------ | -------------------------------------------- |
| 总耗时/s    | 90     | 92<br />(10.44s user 90.35s system 109% cpu) |
| NFS挂载版本 | v4     | v4                                           |
| Access      | 155    | 522                                          |
| GetAttr     | 4.91k  | 12.59k                                       |
| LookUP      | 4.59K  | 11.04k                                       |
| GetFH       | 4.56k  | 11.04k                                       |
| ReadDir     | 274    | 740                                          |
| PutFH       | 5.01k  | 13.33K                                       |

```bash
#!/bin/bash
# 获取脚本所在路径
SCRIPT_DIR=$(cd $(dirname ${BASH_SOURCE[0]}); pwd)
# 设置统计时间
statistic_ts=$(date "+%Y%m%d-%H%M%S")
# 设置统计结果文件
results_file=$SCRIPT_DIR"/$statistic_ts-summary.txt"
splitresults_dir=$SCRIPT_DIR"/$statistic_ts-results/"
if [[ -d $splitresults_dir ]];then rm -rf $splitresults_dir ; else mkdir -p $splitresults_dir;fi
# 清空统计结果文件
> "$results_file"

cat nas-output.csv| awk -F', ' -v summary="$results_file"  -v splitresults_dir="$splitresults_dir" '
BEGIN {
    # 定义大小范围名称和计数变量
    ranges[1] = "1M";           limits[1] = 1024
    ranges[2] = "1M-5M";        limits[2] = 5120
    ranges[3] = "5M-10M";       limits[3] = 10240
    ranges[4] = "10M-20M";      limits[4] = 20480
    ranges[5] = "20M-50M";      limits[5] = 51200
    ranges[6] = "50M-100M";     limits[6] = 102400
    ranges[7] = "100M-150M";    limits[7] = 153600
    ranges[8] = "150M-200M";    limits[8] = 204800
    ranges[9] = "200M-500M";    limits[9] = 512000
    ranges[10] = "500M-800M";   limits[10] = 819200
    ranges[11] = "800M-1000M";  limits[11] = 1024000
    ranges[12] = "1G-3G";       limits[12] = 3072000
    ranges[13] = "3G";          limits[13] = 1e12
    for (i = 1; i <= 13; i++) count[i] = 0 
}
{
    # 读取大小 (单位KB) 和路径
    size = $3
    path = $2
    sub(".*/", "", path)
    # 按范围统计并记录路径
    for (i = 1; i <= 13; i++) {
        if (size <= limits[i]) {
            count[i]++
            # 设置当前输出文件，文件名格式为 "<范围名称>_<分文件编号>.txt"
            if (line_count[i] == 10000) {
                file_count[i]++
                line_count[i] = 0
            }
            # 记录路径到当前范围文件
            range_file = splitresults_dir ranges[i] "_" file_count[i] ".txt"
            print path >> range_file
            line_count[i]++
            break
        }
    }
}

END {
    # 输出统计概要到 summary 文件
    print "范围               目录数量" > summary
    print "---------------------------" >> summary
    for (i = 1; i <= 13; i++) {
        printf "%-15s %10d\n", ranges[i], count[i] >> summary 
    }
}
'
```

## 3、并发压缩至本地磁盘

```bash
#!/bin/bash
# 设置要读取的文件名
startts=$(date "+%s")
startdate=$(date "+%Y%m%d%H%M%S")
input_file_base="/data/backup-nas"
input_file="200M-500M.txt"  # 包含文件夹名的文本文件
backup_base="/data/nas-3000M"
cd /mnt/nas/
log_file="$input_file_base/compressed-$input_file.log"
debug_log="/home/deployer/backup-nas/debug.log"
> "$log_file"
> "$debug_log"
batch_counter=0  # 记录批次数
batch_size=2   # 每批次处理的文件夹数

echo "======$startdate开始并行压缩$input_file_base/$input_file 中的文件,文件夹======" > $log_file
# # 定义压缩函数
compress_folder() {
    folder="$1"
    parent_folder="${folder%%[-_]*}"
    {
        echo "开始压缩文件夹: $folder"
        tar -czf "$backup_base/${parent_folder}/${folder}.tar.gz" "$folder"
    }  >> "$log_file" 2>&1
}
# 从文本文件读取文件夹名，每次并发处理50个
while true; do
    # 每批次读取10个文件夹，并提前创建父文件夹
    folders=()  # 存储当前批次的文件夹
    for ((i = 0; i < $batch_size ; i++)); do
        # 直接读取文件夹名称，文件末尾时退出循环
        if ! read -r folder <&3; then
            break  # 跳出整个 while 循环
        fi
        folders+=("$folder")
        parent_folder="${folder%%[-_]*}"
        if [[ ! -d $backup_base/$parent_folder ]]; then
            mkdir -p $backup_base/$parent_folder
        fi
    done

    # 如果当前批次没有文件夹，结束循环
    if [ ${#folders[@]} -eq 0 ]; then
        break
    fi

    # 增加批次计数，并记录当前批次信息
    batch_counter=$((batch_counter + 1))
    echo "开始处理第$batch_counter批次，包含 ${#folders[@]} 个文件夹。" >> $debug_log
    
    # 并发压缩当前批次的文件夹
    for folder in "${folders[@]}"; do
        compress_folder "$folder" &
    done
    wait
done 3< "$input_file_base/$input_file"  # 将输入文件重定向到文件描述符 3

enddate=$(date "+%Y%m%d%H%M%S")
endts=$(date "+%s")
spenttime=$((endts-startts))
mv "$input_file_base/$input_file" "$input_file_base/compressed-$input_file"
echo "======$enddate 已完成并行压缩, 共计耗时：$spenttime 秒 ======" >> $log_file
echo "======耗时$spenttime秒，总共处理了 $batch_counter 个批次。======" >> $debug_log

```

## 3、串行删除源文件夹

```bash
#!/bin/bash
# 设置要读取的文件名
input_file_base="/home/deployer/backup-nas/20241113-170103-results"
input_file="done-1M_4.txt"  # 包含文件夹名的文本文件
startdate=$(date "+%Y%m%d%H%M%S")
log_file="/home/deployer/backup-nas/20241113-170103-results/deleted-$input_file.log"
# > "$log_file"
echo "======$startdate 开始删除已备份的文件夹======" >> "$log_file"

cd /mnt/nas/his-backup/his-xml/
# 每次处理的行数
batch_size=10
# 检查文件是否存在
if [[ ! -f "$input_file_base/$input_file" ]]; then
  echo "文件 $input_file_base/$input_file 不存在。"  >> "$log_file"
  exit 1
fi

startts=$(date "+%s")
# 读取文件并分批处理
while IFS= read -r line || [[ -n "$line" ]]; do
  if [[ -d "$line" ]]; then
    # 将存在的目录追加到临时数组
    batch+=("$line")
  else
    echo "$(date '+%Y%m%d%H%M%S')警告：目录 $line 不存在，跳过。" >> "$log_file"
  fi
  
  # 如果数组长度达到 batch_size，执行删除操作
  if [[ ${#batch[@]} -eq $batch_size ]]; then
    # echo "删除以下目录："
    # printf '%s\n' "${batch[@]}"  # 打印要删除的目录
    rm -rf "${batch[@]}"        # 执行删除操作
    echo "${batch[@]}" >> "$log_file"
    batch=()                    # 清空数组
  fi
done < "$input_file_base/$input_file"

# 处理剩余的行（不足 batch_size）
if [[ ${#batch[@]} -gt 0 ]]; then
  rm -rf "${batch[@]}"
  echo "${batch[@]}"  >> "$log_file"
fi

mv "$input_file_base/$input_file" "$input_file_base/deleted-$input_file"
endts=$(date "+%s")
enddate=$(date "+%Y%m%d%H%M%S")

spenttime=$((endts-startts))
echo "======耗时$spenttime秒，在$enddate 删除操作完成。========="  >> "$log_file"
```
