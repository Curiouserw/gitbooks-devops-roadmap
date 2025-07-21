# Shell 脚本的资源限制

# 一、用户级别ulimit

ulimit 是Shell 内置 ，用来限制当前 shell 进程及其子进程可用的系统资源上限，主要限制内存、文件大小、CPU 时间、打开文件数等。是由操作系统内核资源管理提供的机制，不是实时限制，而是进程请求资源时的硬限制。

| 资源名称           | ulimit 参数 | 含义                   |
| ------------------ | ----------- | ---------------------- |
| CPU 时间（秒）     | `-t`        | 每个进程最多运行多少秒 |
| 文件大小（KB）     | `-f`        | 可创建的最大文件大小   |
| 数据段大小（KB）   | `-d`        | 数据段最大值（heap）   |
| 堆栈大小（KB）     | `-s`        | 栈大小                 |
| 虚拟内存大小（KB） | `-v`        | 虚拟内存限制           |
| 打开的文件数       | `-n`        | 最大文件描述符数量     |
| 子进程数           | `-u`        | 用户可用的最大进程数   |
| 文件锁数量         | `-x`        | 文件锁数量限制         |
| core 文件大小      | `-c`        | core dump 最大限制     |

使用示例

```bash
#!/bin/bash
# 限制 CPU 时间为 60 秒
ulimit -t 60
# 限制最大文件大小为 10 MB
ulimit -f 10240
# 限制最大打开文件数为 256
ulimit -n 256
# 限制子进程数为 50
ulimit -u 50
...
```


```bash
# 限制当前 shell 及子进程最多使用 512MB 虚拟内存
ulimit -v $((512 * 1024))

# 限制进程最多运行 600 秒 CPU 时间（即 10 分钟）
ulimit -t 600

# 查看所有限制
ulimit -a
```

特点

- 限制的是 资源总量，不是实时的 CPU 占用率限制
- CPU 时间限制的是进程在 CPU 上真正消耗的时间，达到后会收到 SIGXCPU 信号，进程默认被终止。
- 超过内存限制时进程会申请失败或被操作系统杀死（OOM）

是一种“软限制”和“硬限制”结合的机制，软限制可以动态调整，但不能超过硬限制

# 二、实时限制cpulimit—CPU 使用率限制

cpulimit 是一个独立的用户空间工具，实时限制指定进程的 CPU 使用率，通过向进程发送 SIGSTOP/SIGCONT 信号来暂停和恢复进程，从而控制其 CPU 使用率。不是操作系统内核机制，而是利用信号控制进程调度实现限制。

主要功能

- 限制某个进程或进程树的 CPU 使用率（如限制为 30%，50% 等）

- 支持指定进程名、PID 或通过命令启动进程

- 支持限制包括子进程在内的整个进程树（-i 参数）

- 只控制 CPU 使用，不限制内存或其他资源
- 是一个 外部监控工具，通过信号暂停/恢复进程，达到控制 CPU 使用率的效果
- 可以在进程运行时附加限制，不需要进程自身支持
- 对 CPU 限制粒度较细，但不支持内存、IO、文件描述符等限制
- 不改变进程内核级调度优先级，只用暂停方式控制进程执行时间

| 参数 | 含义                         |
| ---- | ---------------------------- |
| `-l` | 最大 CPU 百分比（如 20、50） |
| `-p` | 指定 PID                     |
| `-e` | 指定可执行文件名             |
| `--` | 后面跟执行命令               |

使用示例

```bash
# 限制进程名为 myprocess 的进程最多使用 50% CPU：
cpulimit -l 50 -e myprocess

# 限制 PID 12345
cpulimit -l 30 -p 12345

# 用 cpulimit 启动脚本，限制 40% CPU
cpulimit -l 40 -- bash myscript.sh

# 限制同时包含所有子进程
cpulimit -l 40 -i -- bash myscript.sh
```



# 三、cgroups

`cgroups` 是 Linux 内核提供的功能，用于限制、记录和隔离进程组的资源使用，如 CPU、内存、磁盘 IO 等。适用于容器、后台服务、Shell 脚本的资源控制。

常见控制子系统（子控制器）

| 控制器    | 功能                                   |
| --------- | -------------------------------------- |
| `cpu`     | 控制 CPU 时间分配（如限制 CPU 使用率） |
| `memory`  | 限制使用内存、设置 OOM 行为            |
| `blkio`   | 限制块设备 I/O 速率                    |
| `cpuset`  | 指定使用哪些 CPU 核和内存节点          |
| `devices` | 限制设备访问权限                       |
| `pids`    | 限制进程数                             |

- 创建 cgroup 分组

  ```bash
  cgcreate -g cpu,memory:/myscript
  # 这将在/sys/fs/cgroup/cpu/myscript和/sys/fs/cgroup/memory/myscript下创建目录。
  ```

- 设置限制

  ```bash
  # 限制 CPU 占用（20%）
  cgset -r cpu.cfs_quota_us=20000 myscript
  cgset -r cpu.cfs_period_us=100000
  
  # 限制内存为 100MB
  cgset -r memory.limit_in_bytes=104857600 myscript
  ```

- 使用 `cgexec` 运行脚本

  ```bash
  cgexec -g cpu,memory:myscript ./yourscript.sh
  ```

- 配置文件方式（用于服务/系统启动）

  > `/etc/cgconfig.conf`

  ```ini
  group myscript {
      cpu {
          cpu.cfs_period_us = 100000;
          cpu.cfs_quota_us = 20000;
      }
      memory {
          memory.limit_in_bytes = 104857600;
      }
  }
  ```
  `systemctl start cgconfig`启动配置

📝 注意

- `cgroups v1` 和 `v2` 不完全兼容。`v2` 更现代、更统一。大多数现代发行版（如 Fedora 31+，Ubuntu 22.04）默认用 `cgroups v2`。
- 若使用 Docker 或 systemd，可以直接通过 systemd 单元文件或 Docker 参数控制资源限制。

