# Nvidia管理和监控命令行nvidia-smi

# 一、简介

nvidia-smi简称NVSMI， 基于NVIDIA Management Library （NVIDIA管理库），实现NVIDIA GPU设备的管理和监控功能。提供监控GPU使用情况和更改GPU状态的功能，主要支持Tesla, GRID, Quadro以及TitanX的产品，有限支持其他的GPU产品。是一个跨平台工具，它支持所有标准的NVIDIA驱动程序支持的Linux发行版以及从WindowsServer 2008 R2开始的64位的系统。该工具是N卡驱动附带的，只要安装好驱动后就会有它。

- Windows下程序位置：C:\Program Files\NVIDIACorporation\NVSMI\nvidia-smi.exe。
- Linux下程序位置：/usr/bin/nvidia-smi，由于所在位置已经加入PATH路径，可直接输入nvidia-smi运行

# 二、命令详解

```bash
$ nvidia-smi

Fri Nov 06 06:55:24 2020
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 442.50       Driver Version: 442.50       CUDA Version: 10.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name            TCC/WDDM | Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce RTX 206... WDDM  | 00000000:09:00.0 Off |                  N/A |
|  0%   38C    P8    17W / 215W |    616MiB /  8192MiB |      2%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|    0      1532    C+G   Insufficient Permissions                   N/A      |
|    0      1540    C+G   Insufficient Permissions                   N/A      |
|    0      2064    C+G   Insufficient Permissions                   N/A      |
|    0      2456    C+G   Insufficient Permissions                   N/A      |
|    0      6744    C+G   ...es\Google\Chrome\Application\chrome.exe N/A      |
|    0      6992    C+G   ...t_cw5n1h2txyewy\ShellExperienceHost.exe N/A      |
|    0      7224    C+G   ...x64__8wekyb3d8bbwe\Microsoft.Photos.exe N/A      |
|    0      9284    C+G   C:\Windows\explorer.exe                    N/A      |
|    0     10312    C+G   ...dows.Search_cw5n1h2txyewy\SearchApp.exe N/A      |
|    0     11272    C+G   ...w5n1h2txyewy\InputApp\TextInputHost.exe N/A      |
|    0     14020    C+G   C:\Softwares\Microsoft VS Code\Code.exe    N/A      |
+-----------------------------------------------------------------------------+
```



- **GPU**：GPU 编号；
- **Name**：GPU 型号；
- **Persistence-M**：持续模式的状态。持续模式虽然耗能大，但是在新的GPU应用启动时，花费的时间更少，这里显示的是off的状态；
- **Fan**：风扇转速，从0到100%之间变动，N/A表示没有风扇；
- **Temp**：温度，单位是摄氏度；
- **Perf**：性能状态，从P0到P12，P0表示最大性能，P12表示状态最小性能（即 GPU 未工作时为P0，达到最大工作限度时为P12）。
- **Pwr:Usage/Cap**：能耗；
- **Memory Usage**：显存使用率；
- **Bus-Id**：涉及GPU总线的东西，`domain:bus:device.function`；
- **Disp.A**：Display Active，表示GPU的显示是否初始化；
- **Volatile GPU-Util**：浮动的GPU利用率；
- **Uncorr. ECC**：Error Correcting Code， 是否开启错误检查和纠正技术，0/DISABLED, 1/ENABLED
- **Compute M**：计算模式，`0/DEFAULT, 1/EXCLUSIVE_PROCESS, 2/PROHIBITED`。



- **`nvidia-smi –q –u`** ：显示单元而不是GPU的属性

- **`nvidia-smi –q –i xxx`**：指定具体的GPU或unit信息

- **`nvidia-smi –q –f xxx`**：将查询的信息输出到具体的文件中，不在终端显示

- **`nvidia-smi –q –x`**：将查询的信息以xml的形式输出

- **`nvidia-smi -q –d xxx`**：指定显示GPU卡某些信息

  xxx参数可以为`MEMORY`, `UTILIZATION`, `ECC`, `TEMPERATURE`, `POWER`,`CLOCK`, `COMPUTE`, `PIDS`, `PERFORMANCE`, `SUPPORTED_CLOCKS`, `PAGE_RETIREMENT`,`ACCOUNTING`

- **`nvidia-smi –q –l xxx`**：动态刷新信息，按Ctrl+C停止，可指定刷新频率，以秒为单位

- **`nvidia-smi --query-gpu=gpu_name,gpu_bus_id,vbios_version --format=csv`**：选择性查询选项，可以指定显示的属性选项

  可查看的属性有：timestamp，driver_version，pci.bus，pcie.link.width.current等。（可查看nvidia-smi--help-query–gpu来查看有哪些属性）

## 参考：

1. https://blog.csdn.net/handsome_bear/article/details/80903477

   

