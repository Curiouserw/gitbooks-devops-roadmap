# ESXI升级

# 一、简介

升级ESXi的方法有一下几种，任君挑选：

- 从 CD、DVD 或 USB 驱动器使用交互式图形用户界面 (GUI) 安装程序
- 脚本式升级
- 使用 esxcli行界面 (CLI)
- vSphere Auto Deploy
- vSphere Update Manager

# 二 、CLI命令行升级

## 第一步：检查硬件是否符合

检查服务器的硬件是否符合升级版本

CPU兼容性：https://compatibilityguide.broadcom.com/search?program=cpu&persona=live&column=cpuSeries&order=asc

博通新的兼容性检查网站只支持 7.0 以上版本的

## 第二步：下载升级包

操作步骤省略

## 第三步：上传升级包到数据存储器

操作步骤省略

## 第四步：查看升级包信息

SSH 登录到 ESXI 服务器，使用`esxcli software sources profile list`查看升级文件的版本信息

```bash
esxcli software sources profile list -d /vmfs/volumes/hdd/Images/VMware-VMvisor-Installer-7.0.0.update03-23794027.x86_64-Dell_Customized-A24.zip

# Name                       Vendor     Acceptance Level  Creation Time        Modification Time
# -------------------------  ---------  ----------------  -------------------  -------------------
# DEL-ESXi-703_23794027-A24  Dell Inc.  PartnerSupported  2024-09-30T08:09:36  2024-09-30T08:09:36
```

## 第五步：进入维护模式

操作步骤省略。补充命令行命令：`vicfg-hostops --server=server_name --operation enter`

## 第六步：模拟升级

```bash
esxcli software profile update -p DEL-ESXi-703_23794027-A24 --dry-run -d /vmfs/volumes/hdd/Images/VMware-VMvisor-Installer-7.0.0.update03-23794027.x86_64-Dell_Customized-A24.zip
```

## 第七步：升级

```bash
esxcli software profile update -p DEL-ESXi-703_23794027-A24 -d /vmfs/volumes/hdd/Images/VMware-VMvisor-Installer-7.0.0.update03-23794027.x86_64-Dell_Customized-A24.zip
```

## 第八步：重启

操作步骤省略

## 第九步：退出维护模式

操作步骤省略

# 附录、升级失败或错误操作

## 1、CPU不兼容报错

模拟升级时报错

```bash
 [HardwareError]
 Hardware precheck of profile ESXi-7.0U3s-24585291-standard failed with errors: <CPU_SUPPORT ERROR: The CPU in this host is not supported by ESXi 7.0.3. Please refer to the VMware Compatibility Guide (VCG) for the list of supported CPUs.>
 Please refer to the log file for more details.
```

哦豁，放弃吧少年，你的主机硬件不支持升级到该版本。

## 2、CPU未来不受支持

有针对CPU的告警提示，提醒目前使用的CPU可能在未来不受支持。使用--no-hardware-warning选项直接忽略。

```bash
[HardwareError]
Hardware precheck of profile ESXi-7.0U3s-24585291-standard failed with warnings : <CPU_SUPPORT WARNING: The CPU in this host may not be supported in future ESXi releases. Please plan accordingly.>

Apply --no-hardware-warning option to ignore the warnings and proceed with the transaction.
Please refer to the log file for more details.
```

## 3、移除冲突VIB

如果主机或安装程序 ISO 映像包含的 VIB 会引发冲突和阻止升级。

- 从主机中移除引发冲突的 VIB 并重试升级。

  - 如果使用的是 Update Manager，选择此选项以在修复过程中移除第三方软件模块。

  - 使用的是esxcli还可从主机移除引发冲突的 VIB

    ```bash
    # 确定在主机上安装的 VIB。
    esxcli --server=server_name software vib list
    
    # 移除 VIB。
    esxcli --server=服务器名称 software vib remove --vibname=名称
    # 通过以下形式之一指定要移除的一个或多个 VIB：
    # 名称
    # 名称：版本
    # 供应商：名称
    # 供应商：名称：版本
    ```

- 使用vSphere ESXi Image Builder CLI创建可解决冲突的自定义安装程序 ISO 映像。

# 参考

- https://techdocs.broadcom.com/cn/zh-cn/vmware-cis/vsphere/vsphere/6-7/esxi-upgrade-6-7/about-vsphere-upgrade/overview-of-the-upgrade-process/overview-of-the-esxi-host-upgrade-process.html
- https://www.dell.com/support/manuals/en-us/vmware-esxi-7.x/vmware_7.0_icg_pub/dell-custom-add-on-7.0-u3-package-a24?guid=guid-39663553-01ca-437f-b6e7-c3e5b3a5585c&lang=en-us