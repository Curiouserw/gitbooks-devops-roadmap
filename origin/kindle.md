# Kindle

# 一、简介



Kindle中的电子书主要以 .azw3 、 .mobi 或 .pdf 格式存放于 /mnt/us/documents/ 目录下。虽然可通过Amazon账户重新下载已购内容，但个性化排版、本地导入文档及批注信息无法自动恢复。

重点备份对象包括：
- 所有非Amazon来源的 .mobi 和 .pdf 文件；
- My Clippings.txt ——记录所有高亮、笔记和书签；
- collections.json ——书架分组信息；
- .mbp 数据库文件——存储每本书的阅读进度与元数据。



/etc/wpa_supplicant.conf     Wi-Fi配置

/var/local/appreg.db 应用注册数据库

9.1*12.3厘米

# 二、系统

## 1、搜索框命令

| 类别       | Kindle 搜索框命令    | 功能说明                                             |
| :------------- | :------------------ | :--------------------------------------------------- |
| **阅读与统计** | ;ReadingTimeReset | 重置本章阅读速度统计                                 |
|                | ;ReadingTimeOff   | 关闭本章剩余阅读时间显示                             |
|                | ;ReadingTimeOn    | 开启本章剩余阅读时间显示                             |
| **系统与屏幕** | ~ds               | **禁用屏幕保护程序和自动休眠**，实现常亮             |
|                | ~dm               | 导出系统日志到 /documents 目录                     |
|                | ;411              | 查看 Kindle 服务器信息                               |
|                | ;611              | 查看本地 WAN 口信息                                  |
|                | ;711              | 查看 WiFi 相关信息                                   |
| **重置与清理** | ;urst             | **完全删除用户分区内所有内容**（书籍、文档、日志等） |
|                | ;shpm 或 ;lzzl  | **将设备彻底恢复出厂设置**                           |
|                |                     |                                                      |



## 2、lipc-probe

lipc-probe

## 3、ipc-get-prop

## 4、电池信息gasgauge-info

```bash
gasgauge-info [-chklms] [-i <seconds>] <registers...>
-i s    实时显示(秒)
-c      显示电池剩余容量百分比
-s      显示电池剩余容量百分比，但是不带%
-v      显示电池电压（mV）
-l      显示电池charge current (load)
-k      显示电池温度
-m      显示电池可用容量(mAh)
```

# 三、eips

eips 是一款针对 Kindle 的定制电子墨水支持程序。

显示图片：`eips -g|-b  image_path [-w waveform -f -x xpos -y ypos -v]`

| 用途             | 命令示例                                                     |
| :--------------- | :----------------------------------------------------------- |
| **清除屏幕**     | eips -c                                                      |
| **显示文本**     | eips 'Hello, Kindle' 在屏幕显示字符串文本<br />eips 10 20 'Hello, Kindle'     在屏幕第10行第20列显示文本<br />eips 10 20 -h 'Hello, Kindle'  在屏幕第10行第20列高亮显示文本 |
| **显示图像**     | eips -g image.png                                            |
| **部分刷新**     | eips -g /mnt/us/linkss/screensavers/bg_ss03.png -x 50 -y 100 -v |
| **显示网格**     | eips -a 5                                                    |
| **显示屏幕信息** | eips -i<br />eips -i c                                       |
| **显示测试图案** | eips -q  棋盘格<br />eips -l  灰度<br />eips -p  图案<br />eips -r barcode 条形码 |

强制拉伸以适应目标尺寸，图片会强制拉伸填充新尺寸

> ffmpeg -i input.png -vf "scale=1072:1448" -pix_fmt gray output.png
>
> ​             -vf "scale=1072:1448" 使用视频过滤器缩放图片到1072×1448像素。
>
> ​             -pix_fmt gray 将图片像素格式设置为8位灰度。
>
> # 在FFmpeg内部，解码PNG时，2位或4位灰度等格式也会被转换到8位进行处理。

保持宽高比并自动填充边缘，图片会等比例缩放至完全包含在目标尺寸内，不足部分填充黑色。

> scale=1072:1448:pad=1072:1448:(ow-iw)/2:(oh-ih)/2:black


# 四、系统服务

LIPC是Kindle系统内的一种**轻量级进程间通信机制**。系统服务（如电源管理powerd、网络管理wifid等）会将自己的状态信息和可配置参数作为"**属性（Properties）**"暴露出来。

## 1、设备注册与认证服务

### 设备注册

amazonRegistrationService

| 属性                          | 权限(类型) | 说明               | 命令示例                                                     |
| :---------------------------- | :--------- | :----------------- | :----------------------------------------------------------- |
| primaryUserId                 | r Str      | 获取主用户ID       | lipc-get-prop com.lab126.amazonRegistrationService primaryUserId |
| isDeviceFreshUnregistered     | r Int      | 检查设备是否未注册 | lipc-get-prop com.lab126.amazonRegistrationService isDeviceFreshUnregistered |
| registerWithPreAuthorizedCode | w Str      | 使用预授权码注册   | lipc-set-prop com.lab126.amazonRegistrationService registerWithPreAuthorizedCode "code" |
| GetUserGivenName              | r Str      | 获取用户给定名称   | lipc-get-prop com.lab126.amazonRegistrationService GetUserGivenName |
| stopCBLSignIn                 | w Str      | 停止CBL登录        | lipc-set-prop com.lab126.amazonRegistrationService stopCBLSignIn "" |
| startCBLSignIn                | w Str      | 开始CBL登录        | lipc-set-prop com.lab126.amazonRegistrationService startCBLSignIn "" |
| deregister                    | w Int      | 注销设备           | lipc-set-prop com.lab126.amazonRegistrationService deregister 1 |
| GetDeviceName                 | r Str      | 获取设备名称       | lipc-get-prop com.lab126.amazonRegistrationService GetDeviceName |
| getRegisteredPhoneNumber      | r Str      | 获取注册电话号码   | lipc-get-prop com.lab126.amazonRegistrationService getRegisteredPhoneNumber |
| isRegistrationPending         | r Int      | 检查注册是否待处理 | lipc-get-prop com.lab126.amazonRegistrationService isRegistrationPending |
| GetUserName                   | r Str      | 获取用户名         | lipc-get-prop com.lab126.amazonRegistrationService GetUserName |
| registerWithToken             | w Str      | 使用令牌注册       | lipc-set-prop com.lab126.amazonRegistrationService registerWithToken "token" |
| registerSecondaryUser         | rw Has     | 注册次要用户       | lipc-set-prop com.lab126.amazonRegistrationService registerSecondaryUser '{"user": "data"}' |
| fetchPreRegistrationData      | w Str      | 获取预注册数据     | lipc-set-prop com.lab126.amazonRegistrationService fetchPreRegistrationData "" |
| isRegistrationPending         | r Int      | 检查注册挂起状态   | lipc-get-prop com.lab126.amazonRegistrationService isRegistrationPending |
| isRegistered                  | r Int      | 检查是否已注册     | lipc-get-prop com.lab126.amazonRegistrationService isRegistered |
| updateTimeZone                | w Str      | 更新时区           | lipc-set-prop com.lab126.amazonRegistrationService updateTimeZone "UTC" |
| signIn                        | w Str      | 登录               | lipc-set-prop com.lab126.amazonRegistrationService signIn "" |
| GetCOR                        | r Str      | 获取COR信息        | lipc-get-prop com.lab126.amazonRegistrationService GetCOR  |
| getRegisteredEmail            | r Str      | 获取注册邮箱       | lipc-get-prop com.lab126.amazonRegistrationService getRegisteredEmail |
| registerPrimaryUser           | rw Has     | 注册主用户         | lipc-set-prop com.lab126.amazonRegistrationService registerPrimaryUser '{"user": "data"}' |

### 身份认证

**IdentityService**

| 属性                  | 权限(类型) | 说明           | 命令示例                                                     |
| :-------------------- | :--------- | :------------- | :----------------------------------------------------------- |
| invalidateAccessToken | w Str      | 使访问令牌失效 | lipc-set-prop com.lab126.IdentityService invalidateAccessToken "" |
| refreshCache          | w Str      | 刷新缓存       | lipc-set-prop com.lab126.IdentityService refreshCache ""     |
| accessToken           | rw Has     | 访问令牌管理   | [*NOT SHOWN*]                                                |

### 设备认证

**DeviceAuthenticationService**

| 属性                             | 权限(类型) | 说明                      | 命令示例                                                     |
| :------------------------------- | :--------- | :------------------------ | :----------------------------------------------------------- |
| fetchAuthCookies                 | w Int      | 获取认证cookies           | lipc-set-prop com.lab126.DeviceAuthenticationService fetchAuthCookies 1 |
| cookies                          | r Str      | 读取cookies               | lipc-get-prop com.lab126.DeviceAuthenticationService cookies |
| installCredentials               | w Int      | 安装凭据                  | lipc-set-prop com.lab126.DeviceAuthenticationService installCredentials 1 |
| authcookies                      | r Str      | 读取认证cookies           | lipc-get-prop com.lab126.DeviceAuthenticationService authcookies |
| fetchAuthCookiesForRegionalStack | w Int      | 为区域堆栈获取认证cookies | lipc-set-prop com.lab126.DeviceAuthenticationService fetchAuthCookiesForRegionalStack 1 |

## 2、系统核心服务

### 电源管理

| 属性                       | 权限(类型) | 说明             | 命令示例                                                     |
| :------------------------- | :--------- | :--------------- | :----------------------------------------------------------- |
| state                      | r Str      | 电源状态         | lipc-get-prop com.lab126.powerd state                      |
| powerButton                | w Int      | 电源按钮操作     | lipc-set-prop com.lab126.powerd powerButton 1              |
| status                     | r Str      | 状态信息         | lipc-get-prop com.lab126.powerd status                     |
| touchScreenSaverTimeout    | w Int      | 触摸屏保超时     | lipc-set-prop com.lab126.powerd touchScreenSaverTimeout 300 |
| deviceRestart              | w Int      | 设备重启         | lipc-set-prop com.lab126.powerd deviceRestart 1            |
| flTurboBrightnessIntensity | r Int      | FL Turbo亮度强度 | lipc-get-prop com.lab126.powerd flTurboBrightnessIntensity |
| abortSuspend               | w Int      | 中止挂起         | lipc-set-prop com.lab126.powerd abortSuspend 1             |
| rtcWakeup                  | w Int      | RTC唤醒          | lipc-set-prop com.lab126.powerd rtcWakeup 1                |
| flMaxIntensity             | r Int      | FL最大强度       | lipc-get-prop com.lab126.powerd flMaxIntensity             |
| disableScreenOff           | rw Int     | 禁用屏幕关闭     | lipc-set-prop com.lab126.powerd disableScreenOff 1         |
| battStateInfo              | r Str      | 电池状态信息     | lipc-get-prop com.lab126.powerd battStateInfo              |
| flAuto                     | rw Int     | FL自动模式       | lipc-set-prop com.lab126.powerd flAuto 1                   |
| deferRestart               | w Int      | 延迟重启         | lipc-set-prop com.lab126.powerd deferRestart 1             |
| rtcWakeup2                 | w Str      | RTC唤醒2         | lipc-set-prop com.lab126.powerd rtcWakeup2 "time"          |
| flStartup                  | w Int      | FL启动           | lipc-set-prop com.lab126.powerd flStartup 1                |
| addSuspendLevels           | w Int      | 添加挂起级别     | lipc-set-prop com.lab126.powerd addSuspendLevels 1         |
| restartAlertResponse       | w Str      | 重启警报响应     | lipc-set-prop com.lab126.powerd restartAlertResponse "response" |
| logMask                    | rw Str     | 日志掩码         | lipc-set-prop com.lab126.powerd logMask "mask"             |
| logLevel                   | rw Str     | 日志级别         | lipc-set-prop com.lab126.powerd logLevel "level"           |
| deferSuspend               | w Int      | 延迟挂起         | lipc-set-prop com.lab126.powerd deferSuspend 1             |
| suspendGrace               | w Int      | 挂起宽限期       | lipc-set-prop com.lab126.powerd suspendGrace 1             |
| isLowTempState             | r Int      | 是否低温状态     | lipc-get-prop com.lab126.powerd isLowTempState             |
| isCharging                 | r Int      | 是否充电         | lipc-get-prop com.lab126.powerd isCharging                 |
| battTemperature            | r Int      | 电池温度         | lipc-get-prop com.lab126.powerd battTemperature            |
| battLevel                  | r Int      | 电池电量         | lipc-get-prop com.lab126.powerd battLevel                  |
| preventScreenSaver         | rw Int     | 防止屏保         | lipc-set-prop com.lab126.powerd preventScreenSaver 1       |
| wakeUp                     | w Int      | 唤醒设备         | lipc-set-prop com.lab126.powerd wakeUp 1                   |
| flRawIntensity             | r Int      | FL原始强度       | lipc-get-prop com.lab126.powerd flRawIntensity             |
| sendPendingResume          | w Int      | 发送待处理恢复   | lipc-set-prop com.lab126.powerd sendPendingResume 1        |
| shutdownAlertResponse      | w Str      | 关机警报响应     | lipc-set-prop com.lab126.powerd shutdownAlertResponse "response" |
| flIntensity                | rw Int     | FL强度           | lipc-set-prop com.lab126.powerd flIntensity 50             |

### 进程监控

**pmond**

| 属性                 | 权限(类型) | 说明         | 命令示例                                              |
| :------------------- | :--------- | :----------- | :---------------------------------------------------- |
| mem_warn             | w Str      | 内存警告     | lipc-set-prop com.lab126.pmond mem_warn "warning"   |
| kill                 | w Str      | 终止进程     | lipc-set-prop com.lab126.pmond kill "process"       |
| system_memory_status | r Str      | 系统内存状态 | lipc-get-prop com.lab126.pmond system_memory_status |
| restart              | w Str      | 重启进程     | lipc-set-prop com.lab126.pmond restart "process"    |
| stop                 | w Str      | 停止进程     | lipc-set-prop com.lab126.pmond stop "process"       |
| logLevel             | rw Str     | 日志级别     | lipc-set-prop com.lab126.pmond logLevel "level"     |
| summary              | r Str      | 摘要信息     | lipc-get-prop com.lab126.pmond summary              |
| logMask              | rw Str     | 日志掩码     | lipc-set-prop com.lab126.pmond logMask "mask"       |
| start_monitoring     | w Int      | 开始监控     | lipc-set-prop com.lab126.pmond start_monitoring 1   |
| stop_monitoring      | w Int      | 停止监控     | lipc-set-prop com.lab126.pmond stop_monitoring 1    |
| heartbeat_start      | w Str      | 心跳开始     | lipc-set-prop com.lab126.pmond heartbeat_start ""   |
| start                | w Str      | 启动进程     | lipc-set-prop com.lab126.pmond start "process"      |
| mem_limit            | w Str      | 内存限制     | lipc-set-prop com.lab126.pmond mem_limit "limit"    |
| heartbeat_stop       | w Str      | 心跳停止     | lipc-set-prop com.lab126.pmond heartbeat_stop ""    |

### 应用框架kaf

| 属性                | 权限(类型) | 说明           | 命令示例                                              |
| :------------------ | :--------- | :------------- | :---------------------------------------------------- |
| dumpHeap            | w Str      | 转储堆内存     | lipc-set-prop com.lab126.kaf dumpHeap ""            |
| callInspector       | w Str      | 调用检查器     | lipc-set-prop com.lab126.kaf callInspector ""       |
| ondemand            | rw Has     | 按需服务       | lipc-get-prop com.lab126.kaf ondemand               |
| timezoneId          | r Str      | 时区ID         | lipc-get-prop com.lab126.kaf timezoneId             |
| callCVM             | w Str      | 调用CVM        | lipc-set-prop com.lab126.kaf callCVM ""             |
| currentDisplayMode  | rw Int     | 当前显示模式   | lipc-set-prop com.lab126.kaf currentDisplayMode 1   |
| allocHeap           | w Int      | 分配堆内存     | lipc-set-prop com.lab126.kaf allocHeap 1024         |
| purgeSoftRefs       | w Str      | 清除软引用     | lipc-set-prop com.lab126.kaf purgeSoftRefs ""       |
| heapUsed            | r Int      | 已用堆内存     | lipc-get-prop com.lab126.kaf heapUsed               |
| logLevel            | rw Str     | 日志级别       | lipc-set-prop com.lab126.kaf logLevel "level"       |
| dumpHeapWithoutGC   | w Str      | 无GC转储堆     | lipc-set-prop com.lab126.kaf dumpHeapWithoutGC ""   |
| delayFrameworkStart | rw Int     | 延迟框架启动   | lipc-set-prop com.lab126.kaf delayFrameworkStart 1  |
| frameworkStarted    | r Int      | 框架已启动     | lipc-get-prop com.lab126.kaf frameworkStarted       |
| logMask             | rw Str     | 日志掩码       | lipc-set-prop com.lab126.kaf logMask "mask"         |
| dumpHeapUsage       | r Str      | 转储堆使用情况 | lipc-get-prop com.lab126.kaf dumpHeapUsage          |
| dumpNativeHeapStats | w Str      | 转储原生堆统计 | lipc-set-prop com.lab126.kaf dumpNativeHeapStats "" |

## 3、网络连接服务

### WiFi管理
wifid

| 属性                        | 权限(类型) | 说明               | 命令示例                                                     |
| :-------------------------- | :--------- | :----------------- | :----------------------------------------------------------- |
| certificateData             | rw Has     | 证书数据           |                                                              |
| deleteCertificate           | w Str      | 删除证书           | lipc-set-prop com.lab126.wifid deleteCertificate "cert"      |
| signalStrength              | r Str      | 信号强度           | lipc-get-prop com.lab126.wifid signalStrength                |
| cmIntfInfo                  | rw Has     | CM接口信息         |                                                              |
| enable                      | rw Int     | 启用WiFi           | lipc-set-prop com.lab126.wifid enable 1                      |
| hotSpotDBDownloadStatus     | w Int      | 热点数据库下载状态 | lipc-set-prop com.lab126.wifid hotSpotDBDownloadStatus 1     |
| 711                         | r Str      | 711信息            | lipc-get-prop com.lab126.wifid 711                           |
| macSecret                   | r Str      | MAC密钥            | lipc-get-prop com.lab126.wifid macSecret                     |
| cmConnect                   | w Str      | CM连接             | lipc-set-prop com.lab126.wifid cmConnect "network"           |
| cmCheckConnection           | w Str      | CM检查连接         | lipc-set-prop com.lab126.wifid cmCheckConnection ""          |
| connMetadata                | rw Has     | 连接元数据         |                                                              |
| cmIntfInUse                 | r Int      | 使用中的CM接口     | lipc-get-prop com.lab126.wifid cmIntfInUse                   |
| manufacturerCode            | r Str      | 制造商代码         | lipc-get-prop com.lab126.wifid manufacturerCode              |
| installCertificate          | rw Has     | 安装证书           | lipc-set-prop com.lab126.wifid installCertificate '{"cert": "data"}' |
| setRegDomCC                 | w Str      | 设置区域域CC       | lipc-set-prop com.lab126.wifid setRegDomCC "CC"              |
| macAddress                  | r Str      | MAC地址            | lipc-get-prop com.lab126.wifid macAddress                    |
| profileData                 | rw Has     | 配置文件数据       | lipc-get-prop com.lab126.wifid profileData                   |
| netConfig                   | rw Has     | 网络配置           | lipc-get-prop com.lab126.wifid netConfig                     |
| cmDisconnect                | w Str      | CM断开连接         | lipc-set-prop com.lab126.wifid cmDisconnect ""               |
| logMask                     | rw Str     | 日志掩码           | lipc-set-prop com.lab126.wifid logMask "mask"                |
| logLevel                    | rw Str     | 日志级别           | lipc-set-prop com.lab126.wifid logLevel "level"              |
| cmNWProperties              | rw Has     | CM网络属性         | lipc-get-prop com.lab126.wifid cmNWProperties                |
| createProfile               | rw Has     | 创建配置文件       | lipc-set-prop com.lab126.wifid createProfile '{"profile": "data"}' |
| scanState                   | r Str      | 扫描状态           | lipc-get-prop com.lab126.wifid scanState                     |
| cmConnMode                  | w Str      | CM连接模式         | lipc-set-prop com.lab126.wifid cmConnMode "mode"             |
| feelingLuckyProfile         | r Str      | 幸运配置文件       | lipc-get-prop com.lab126.wifid feelingLuckyProfile           |
| deleteProfile               | w Str      | 删除配置文件       | lipc-set-prop com.lab126.wifid deleteProfile "profile"       |
| currentEssid                | rw Has     | 当前ESSID          |                                                              |
| cmState                     | r Str      | CM状态             | lipc-get-prop com.lab126.wifid cmState                       |
| createNetConfig             | rw Has     | 创建网络配置       | lipc-set-prop com.lab126.wifid createNetConfig '{"config": "data"}' |
| scan                        | w Str      | 扫描网络           | lipc-set-prop com.lab126.wifid scan ""                       |
| scanList                    | rw Has     | 扫描列表           |                                                              |
| profileCount                | r Int      | 配置文件计数       | lipc-get-prop com.lab126.wifid profileCount                  |
| resetSavePasswordPreference | w Int      | 重置保存密码首选项 | lipc-set-prop com.lab126.wifid resetSavePasswordPreference 1 |

### 连接管理器cmd

| 属性                | 权限(类型) | 说明        | 命令示例                                                     |
| :------------------ | :--------- | :---------- | :----------------------------------------------------------- |
| ensureConnection    | w Str      | 确保连接    | lipc-set-prop com.lab126.cmd ensureConnection ""             |
| activeInterface     | r Str      | 活跃接口    | lipc-get-prop com.lab126.cmd activeInterface                 |
| logLevel            | rw Str     | 日志级别    | lipc-set-prop com.lab126.cmd logLevel "level"                |
| logMask             | rw Str     | 日志掩码    | lipc-set-prop com.lab126.cmd logMask "mask"                  |
| interfaceProperties | rw Has     | 接口属性    | lipc-get-prop com.lab126.cmd interfaceProperties             |
| disableInterfaces   | rw Has     | 禁用接口    | lipc-set-prop com.lab126.cmd disableInterfaces '{"interface": "wifi"}' |
| wirelessEnable      | rw Int     | 启用无线    | lipc-set-prop com.lab126.cmd wirelessEnable 1                |
| availableInterfaces | rw Has     | 可用接口    |                                                              |
| smsRecvRequired     | rw Int     | SMS接收要求 | lipc-set-prop com.lab126.cmd smsRecvRequired 1               |

## 4、存储管理服务

### 卷管理

| 属性                         | 权限(类型) | 说明                 | 命令示例                                                     |
| :--------------------------- | :--------- | :------------------- | :----------------------------------------------------------- |
| userstoreIsAvailable         | r Int      | 用户存储是否可用     | lipc-get-prop com.lab126.volumd userstoreIsAvailable       |
| userstoreTotalSpace          | r Int      | 用户存储总空间       | lipc-get-prop com.lab126.volumd userstoreTotalSpace        |
| userstoreDeferUnmountRequest | w Int      | 用户存储延迟卸载请求 | lipc-set-prop com.lab126.volumd userstoreDeferUnmountRequest 1 |
| logLevel                     | rw Str     | 日志级别             | lipc-set-prop com.lab126.volumd logLevel "level"           |
| logMask                      | rw Str     | 日志掩码             | lipc-set-prop com.lab126.volumd logMask "mask"             |
| disableMounting              | rw Int     | 禁用挂载             | lipc-set-prop com.lab126.volumd disableMounting 1          |
| recreateUserstoreResponse    | w Str      | 重新创建用户存储响应 | lipc-set-prop com.lab126.volumd recreateUserstoreResponse "response" |
| userstoreReadyToUnMount      | w Int      | 用户存储准备卸载     | lipc-set-prop com.lab126.volumd userstoreReadyToUnMount 1  |
| useUsbForNetwork             | rw Int     | 使用USB进行网络      | lipc-set-prop com.lab126.volumd useUsbForNetwork 1         |
| driveModeState               | r Int      | 驱动器模式状态       | lipc-get-prop com.lab126.volumd driveModeState             |
| userstoreFreeSpace           | r Int      | 用户存储可用空间     | lipc-get-prop com.lab126.volumd userstoreFreeSpace         |

## 5、阅读相关服务

### 阅读设置

| 属性          | 权限(类型) | 说明         | 命令示例                                                 |
| :------------ | :--------- | :----------- | :------------------------------------------------------- |
| getFontStyles | r Str      | 获取字体样式 | lipc-get-prop com.lab126.reader.qa getFontStyles       |
| lineSpacing   | rw Int     | 行间距       | lipc-set-prop com.lab126.reader.qa lineSpacing 2       |
| fontSize      | rw Int     | 字体大小     | lipc-set-prop com.lab126.reader.qa fontSize 3          |
| fontBoldness  | rw Int     | 字体粗细     | lipc-set-prop com.lab126.reader.qa fontBoldness 1      |
| fontStyle     | rw Str     | 字体样式     | lipc-set-prop com.lab126.reader.qa fontStyle "regular" |

### 注释管理

**whisperstore**

| 属性                            | 权限(类型) | 说明                 | 命令示例                                                     |
| :------------------------------ | :--------- | :------------------- | :----------------------------------------------------------- |
| get_all_annotation              | rw Has     | 获取所有注释         |                                                              |
| add_or_update_annotation_edit   | rw Has     | 添加或更新注释       | lipc-set-prop com.lab126.whisperstore add_or_update_annotation_edit |
| delete_annotation_edit          | rw Has     | 删除注释             | lipc-set-prop com.lab126.whisperstore delete_annotation_edit |
| ingest_lpr_sidecar              | rw Has     | 导入LPR边车数据      | lipc-set-prop com.lab126.whisperstore ingest_lpr_sidecar     |
| clean_sync_table_on_book_delete | rw Has     | 删除书籍时清理同步表 | lipc-set-prop com.lab126.whisperstore clean_sync_table_on_book_delete |
| ingest_all_annotation           | rw Has     | 导入所有注释         | lipc-set-prop com.lab126.whisperstore ingest_all_annotation  |

## 6、应用程序管理

### 应用程序管理器

| 属性                  | 权限(类型) | 说明             | 命令示例                                                     |
| :-------------------- | :--------- | :--------------- | :----------------------------------------------------------- |
| close                 | w Int      | 关闭应用         | lipc-set-prop com.lab126.appmgrd close 1                   |
| backward              | w Int      | 后退             | lipc-set-prop com.lab126.appmgrd backward 1                |
| logallapps            | w Str      | 记录所有应用     | lipc-set-prop com.lab126.appmgrd logallapps ""             |
| seekBackToView        | w Str      | 回到视图         | lipc-set-prop com.lab126.appmgrd seekBackToView "view"     |
| activeContext         | rw Str     | 活动上下文       | lipc-get-prop com.lab126.appmgrd activeContext             |
| saveContextToView     | w Str      | 保存上下文到视图 | lipc-set-prop com.lab126.appmgrd saveContextToView "view"  |
| oobeAppCrashResponse  | w Str      | OOBE应用崩溃响应 | lipc-set-prop com.lab126.appmgrd oobeAppCrashResponse "response" |
| loadresult            | w Str      | 加载结果         | lipc-set-prop com.lab126.appmgrd loadresult "result"       |
| kppBackButtonPress    | w Int      | KPP后退按钮按下  | lipc-set-prop com.lab126.appmgrd kppBackButtonPress 1      |
| backButtonState       | r Int      | 后退按钮状态     | lipc-get-prop com.lab126.appmgrd backButtonState           |
| backgroundAppLoaded   | w Str      | 后台应用加载     | lipc-set-prop com.lab126.appmgrd backgroundAppLoaded "app" |
| peekHistoryView       | r Str      | 查看历史视图     | lipc-get-prop com.lab126.appmgrd peekHistoryView           |
| backgroundAppUnLoaded | w Str      | 后台应用卸载     | lipc-set-prop com.lab126.appmgrd backgroundAppUnLoaded "app" |
| activeView            | r Str      | 活动视图         | lipc-get-prop com.lab126.appmgrd activeView                |
| startdefault          | w Int      | 启动默认应用     | lipc-set-prop com.lab126.appmgrd startdefault 1            |
| activeAppPid          | r Int      | 活动应用PID      | lipc-get-prop com.lab126.appmgrd activeAppPid              |
| forward               | w Int      | 前进             | lipc-set-prop com.lab126.appmgrd forward 1                 |
| logMask               | rw Str     | 日志掩码         | lipc-set-prop com.lab126.appmgrd logMask "mask"            |
| logLevel              | rw Str     | 日志级别         | lipc-set-prop com.lab126.appmgrd logLevel "level"          |
| clearAllContext       | w Int      | 清除所有上下文   | lipc-set-prop com.lab126.appmgrd clearAllContext 1         |
| defer                 | w Str      | 延迟操作         | lipc-set-prop com.lab126.appmgrd defer "action"            |
| savecontext           | w Str      | 保存上下文       | lipc-set-prop com.lab126.appmgrd savecontext "context"     |
| peekAppHistory        | r Str      | 查看应用历史     | lipc-get-prop com.lab126.appmgrd peekAppHistory            |
| pauseresult           | w Str      | 暂停结果         | lipc-set-prop com.lab126.appmgrd pauseresult "result"      |
| unloadresult          | w Str      | 卸载结果         | lipc-set-prop com.lab126.appmgrd unloadresult "result"     |
| load                  | w Str      | 加载应用         | lipc-set-prop com.lab126.appmgrd load "app"                |
| activeApp             | r Str      | 活动应用         | lipc-get-prop com.lab126.appmgrd activeApp                 |
| stop                  | w Str      | 停止应用         | lipc-set-prop com.lab126.appmgrd stop "app"                |
| loghistory            | w Str      | 记录历史         | lipc-set-prop com.lab126.appmgrd loghistory ""             |
| startView             | w Str      | 启动视图         | lipc-set-prop com.lab126.appmgrd startView "view"          |
| goresult              | w Str      | 执行结果         | lipc-set-prop com.lab126.appmgrd goresult "result"         |
| runresult             | w Str      | 运行结果         | lipc-set-prop com.lab126.appmgrd runresult "result"        |
| popAppHistory         | r Str      | 弹出应用历史     | lipc-get-prop com.lab126.appmgrd popAppHistory             |
| start                 | w Str      | 启动应用         | lipc-set-prop com.lab126.appmgrd start "app"               |
| peekAppHistoryURI     | r Str      | 查看应用历史URI  | lipc-get-prop com.lab126.appmgrd peekAppHistoryURI         |

## 7、用户界面服务

### UI框架

| 属性                  | 权限(类型) | 说明               | 命令示例                                                     |
| :-------------------- | :--------- | :----------------- | :----------------------------------------------------------- |
| disableEnablePillow   | w Str      | 禁用/启用Pillow    | lipc-set-prop com.lab126.pillow disableEnablePillow "disable" |
| dismissChrome         | w Str      | 隐藏Chrome界面     | lipc-set-prop com.lab126.pillow dismissChrome ""             |
| customDialog          | w Str      | 自定义对话框       | lipc-set-prop com.lab126.pillow customDialog "dialog"        |
| applicationWindow     | w Str      | 应用窗口           | lipc-set-prop com.lab126.pillow applicationWindow "window"   |
| logLevel              | rw Str     | 日志级别           | lipc-set-prop com.lab126.pillow logLevel "level"             |
| activityIndicator     | w Str      | 活动指示器         | lipc-set-prop com.lab126.pillow activityIndicator "show"     |
| debugInfo             | w Str      | 调试信息           | lipc-set-prop com.lab126.pillow debugInfo ""                 |
| logMask               | rw Str     | 日志掩码           | lipc-set-prop com.lab126.pillow logMask "mask"               |
| displayChrome         | w Str      | 显示Chrome界面     | lipc-set-prop com.lab126.pillow displayChrome ""             |
| interrogatePillow     | w Str      | 查询Pillow状态     | lipc-set-prop com.lab126.pillow interrogatePillow ""         |
| interrogatePillowHash | rw Has     | 查询Pillow状态哈希 | lipc-get-prop com.lab126.pillow interrogatePillowHash        |
| pillowAlert           | w Str      | 显示警告           |                                                              |

### 窗口管理器

| 属性                           | 权限(类型) | 说明               | 命令示例                                                     |
| :----------------------------- | :--------- | :----------------- | :----------------------------------------------------------- |
| liglDebugParams                | w Str      | LIGL调试参数       | lipc-set-prop com.lab126.winmgr liglDebugParams "params"   |
| defineWipeCurve                | w Str      | 定义擦除曲线       | lipc-set-prop com.lab126.winmgr defineWipeCurve "curve"    |
| setAccOrientation              | rw Str     | 设置加速计方向     | lipc-set-prop com.lab126.winmgr setAccOrientation "portrait" |
| extendAfterDamageTimeout       | w Int      | 扩展损坏超时后     | lipc-set-prop com.lab126.winmgr extendAfterDamageTimeout 1000 |
| epdcMode                       | rw Str     | EPDC模式           | lipc-set-prop com.lab126.winmgr epdcMode "mode"            |
| lightboxMode                   | rw Int     | 灯箱模式           | lipc-set-prop com.lab126.winmgr lightboxMode 1             |
| debugInfo                      | rw Str     | 调试信息           | lipc-get-prop com.lab126.winmgr debugInfo                  |
| extendDamageTimeout            | w Int      | 扩展损坏超时       | lipc-set-prop com.lab126.winmgr extendDamageTimeout 1000   |
| chromeState                    | rw Int     | Chrome状态         | lipc-set-prop com.lab126.winmgr chromeState 1              |
| getAllWindows                  | w Str      | 获取所有窗口       | lipc-set-prop com.lab126.winmgr getAllWindows ""           |
| orientation                    | r Str      | 屏幕方向           | lipc-get-prop com.lab126.winmgr orientation                |
| liglFlash                      | w Str      | LIGL闪烁           | lipc-set-prop com.lab126.winmgr liglFlash ""               |
| timerRemainingTime             | rw Int     | 计时器剩余时间     | lipc-set-prop com.lab126.winmgr timerRemainingTime 1000    |
| accelerometer                  | r Str      | 加速计数据         | lipc-get-prop com.lab126.winmgr accelerometer              |
| focusChrome                    | w Str      | 聚焦Chrome         | lipc-set-prop com.lab126.winmgr focusChrome ""             |
| setPreference                  | w Str      | 设置首选项         | lipc-set-prop com.lab126.winmgr setPreference "pref"       |
| fakeKeyEvent                   | w Str      | 模拟按键事件       | lipc-set-prop com.lab126.winmgr fakeKeyEvent "key"         |
| winmgrReady                    | r Int      | 窗口管理器就绪     | lipc-get-prop com.lab126.winmgr winmgrReady                |
| clearTrigger                   | w Str      | 清除触发器         | lipc-set-prop com.lab126.winmgr clearTrigger "trigger"     |
| liglPause                      | rw Int     | LIGL暂停           | lipc-set-prop com.lab126.winmgr liglPause 1                |
| isScreenSaverLayerWindowActive | r Int      | 屏保层窗口是否活跃 | lipc-get-prop com.lab126.winmgr isScreenSaverLayerWindowActive |
| grip_enabled                   | rw Int     | 握持启用           | lipc-set-prop com.lab126.winmgr grip_enabled 1             |
| visibleWindows                 | w Str      | 可见窗口           | lipc-set-prop com.lab126.winmgr visibleWindows ""          |
| createTrigger                  | w Str      | 创建触发器         | lipc-set-prop com.lab126.winmgr createTrigger "trigger"    |
| getActiveAppTitle              | r Str      | 获取活跃应用标题   | lipc-get-prop com.lab126.winmgr getActiveAppTitle          |
| eatTapMode                     | w Int      | 吞噬点击模式       | lipc-set-prop com.lab126.winmgr eatTapMode 1               |
| logMask                        | rw Str     | 日志掩码           | lipc-set-prop com.lab126.winmgr logMask "mask"             |
| logLevel                       | rw Str     | 日志级别           | lipc-set-prop com.lab126.winmgr logLevel "level"           |
| passwordDialogEnabled          | rw Int     | 密码对话框启用     | lipc-set-prop com.lab126.winmgr passwordDialogEnabled 1    |
| orientationLock                | rw Str     | 方向锁定           | lipc-set-prop com.lab126.winmgr orientationLock "portrait" |
| ASRMode                        | rw Int     | ASR模式            | lipc-set-prop com.lab126.winmgr ASRMode 1                  |
| refreshOnTurn                  | rw Str     | 翻页刷新           | lipc-set-prop com.lab126.winmgr refreshOnTurn "enable"     |
| fakeTap                        | w Str      | 模拟点击           | lipc-set-prop com.lab126.winmgr fakeTap "x,y"              |
| pageTurnkeyConfig              | rw Str     | 翻页键配置         | lipc-set-prop com.lab126.winmgr pageTurnkeyConfig "config" |
| activeDialogCount              | r Int      | 活跃对话框计数     | lipc-get-prop com.lab126.winmgr activeDialogCount          |
| enableDispalyEvents            | rw Int     | 启用显示事件       | lipc-set-prop com.lab126.winmgr enableDispalyEvents 1      |

## 8、数据传输与同步

### 系统更新

| 属性                         | 权限(类型) | 说明                   | 命令示例                                                  |
| :--------------------------- | :--------- | :--------------------- | :-------------------------------------------------------- |
| refreshCache                 | w Str      | 刷新缓存               | lipc-set-prop com.lab126.ota refreshCache ""              |
| isBootAfterSilentOTA         | r Int      | 是否静默OTA后启动      | lipc-get-prop com.lab126.ota isBootAfterSilentOTA         |
| shouldShowReleaseNotesOnHome | r Int      | 是否在主页显示发布说明 | lipc-get-prop com.lab126.ota shouldShowReleaseNotesOnHome |
| isFirstBootAfterUpdate       | r Int      | 是否更新后首次启动     | lipc-get-prop com.lab126.ota isFirstBootAfterUpdate       |
| getUpdateVersion             | r Str      | 获取更新版本           | lipc-get-prop com.lab126.ota getUpdateVersion             |
| isBootAfterUpdate            | r Int      | 是否更新后启动         | lipc-get-prop com.lab126.ota isBootAfterUpdate            |
| isUpdateAvailable            | r Int      | 检查更新可用性         | lipc-get-prop com.lab126.ota isUpdateAvailable            |
| autoInstall                  | rw Int     | 自动安装               | lipc-set-prop com.lab126.ota autoInstall 1                |
| startUpdate                  | w Int      | 开始更新               | lipc-set-prop com.lab126.ota startUpdate 1                |
| logLevel                     | rw Str     | 日志级别               | lipc-set-prop com.lab126.ota logLevel "level"             |
| scanUserstoreForUpdates      | w Int      | 扫描用户存储更新       | lipc-set-prop com.lab126.ota scanUserstoreForUpdates 1    |
| logMask                      | rw Str     | 日志掩码               | lipc-set-prop com.lab126.ota logMask "mask"               |
| isUpdatePending              | r Str      | 更新是否挂起           | lipc-get-prop com.lab126.ota isUpdatePending              |
| checkForOTA                  | w Str      | 检查OTA更新            | lipc-set-prop com.lab126.ota checkForOTA ""               |
| transferProgressNotification | rw Has     | 传输进度通知           |                                                           |
| subscribeForProgressUpdate   | w Str      | 订阅进度更新           |                                                           |

## 9、系统工具与监控

### 设备策略管理

| 属性                              | 权限(类型) | 说明                  | 命令示例                                                     |
| :-------------------------------- | :--------- | :-------------------- | :----------------------------------------------------------- |
| isPopularHighlightsDisabled       | r Int      | 热门标注是否禁用      | lipc-get-prop com.lab126.dpmManager isPopularHighlightsDisabled |
| parentalControlsHint              | rw Str     | 家长控制提示          | lipc-set-prop com.lab126.dpmManager parentalControlsHint "hint" |
| isSettingsMenuDisabled            | r Int      | 设置菜单是否禁用      | lipc-get-prop com.lab126.dpmManager isSettingsMenuDisabled   |
| isParentalPasswordSet             | r Int      | 家长密码是否设置      | lipc-get-prop com.lab126.dpmManager isParentalPasswordSet    |
| isActiveContentDisabled           | r Int      | 活动内容是否禁用      | lipc-get-prop com.lab126.dpmManager isActiveContentDisabled  |
| isExperimentalMenuDisabled        | r Int      | 实验菜单是否禁用      | lipc-get-prop com.lab126.dpmManager isExperimentalMenuDisabled |
| isDiscoveryDisabled               | rw Int     | 发现功能是否禁用      | lipc-set-prop com.lab126.dpmManager isDiscoveryDisabled 1    |
| logMask                           | rw Str     | 日志掩码              | lipc-set-prop com.lab126.dpmManager logMask "mask"           |
| isChangeLanguageDisabled          | r Int      | 更改语言是否禁用      | lipc-get-prop com.lab126.dpmManager isChangeLanguageDisabled |
| isToggleWirelessDisabled          | r Int      | 切换无线是否禁用      | lipc-get-prop com.lab126.dpmManager isToggleWirelessDisabled |
| wipeAfterMaxPasscodeAttempts      | r Int      | 最大密码尝试后擦除    | lipc-get-prop com.lab126.dpmManager wipeAfterMaxPasscodeAttempts |
| isChangeDictionaryDisabled        | r Int      | 更改字典是否禁用      | lipc-get-prop com.lab126.dpmManager isChangeDictionaryDisabled |
| isFactoryResetDisabled            | r Int      | 恢复出厂设置是否禁用  | lipc-get-prop com.lab126.dpmManager isFactoryResetDisabled   |
| logLevel                          | rw Str     | 日志级别              | lipc-set-prop com.lab126.dpmManager logLevel "level"         |
| secondaryProxyHost                | r Str      | 二级代理主机          | lipc-get-prop com.lab126.dpmManager secondaryProxyHost       |
| isTurnOnPasscodeDisabled          | r Int      | 开启密码是否禁用      | lipc-get-prop com.lab126.dpmManager isTurnOnPasscodeDisabled |
| setParentalControlsPassword       | rw Has     | 设置家长控制密码      | lipc-set-prop com.lab126.dpmManager setParentalControlsPassword |
| getControlStatus                  | r Int      | 获取控制状态          | lipc-get-prop com.lab126.dpmManager getControlStatus         |
| minPasscodeLength                 | r Int      | 最小密码长度          | lipc-get-prop com.lab126.dpmManager minPasscodeLength        |
| devicePolicyId                    | r Str      | 设备策略ID            | lipc-get-prop com.lab126.dpmManager devicePolicyId           |
| isArchivedItemsDisabled           | rw Int     | 归档项目是否禁用      | lipc-set-prop com.lab126.dpmManager isArchivedItemsDisabled 1 |
| isSocialNetworksDisabled          | r Int      | 社交网络是否禁用      | lipc-get-prop com.lab126.dpmManager isSocialNetworksDisabled |
| isToggleAnnotationsBackupDisabled | r Int      | 切换注释备份是否禁用  | lipc-get-prop com.lab126.dpmManager isToggleAnnotationsBackupDisabled |
| isKindleStoreDisabled             | rw Int     | Kindle商店是否禁用    | lipc-set-prop com.lab126.dpmManager isKindleStoreDisabled 1  |
| isTurnOffPasscodeDisabled         | r Int      | 关闭密码是否禁用      | lipc-get-prop com.lab126.dpmManager isTurnOffPasscodeDisabled |
| showAppLockedAlert                | rw Has     | 显示应用锁定警告      | lipc-set-prop com.lab126.dpmManager showAppLockedAlert       |
| isMp3PlayerDisabled               | r Int      | MP3播放器是否禁用     | lipc-get-prop com.lab126.dpmManager isMp3PlayerDisabled      |
| isWikipediaLookupDisabled         | rw Int     | Wikipedia查找是否禁用 | lipc-set-prop com.lab126.dpmManager isWikipediaLookupDisabled 1 |
| isAnnotationsBackupDisabled       | r Int      | 注释备份是否禁用      | lipc-get-prop com.lab126.dpmManager isAnnotationsBackupDisabled |
| isPublicNotesDisabled             | r Int      | 公共笔记是否禁用      | lipc-get-prop com.lab126.dpmManager isPublicNotesDisabled    |
| isMinPasscodeLengthEnabled        | r Int      | 最小密码长度是否启用  | lipc-get-prop com.lab126.dpmManager isMinPasscodeLengthEnabled |
| contactInfo                       | r Str      | 联系信息              | lipc-get-prop com.lab126.dpmManager contactInfo              |
| isBrowserDisabled                 | rw Int     | 浏览器是否禁用        | lipc-set-prop com.lab126.dpmManager isBrowserDisabled 1      |
| isManageWifiSettingsDisabled      | r Int      | 管理WiFi设置是否禁用  | lipc-get-prop com.lab126.dpmManager isManageWifiSettingsDisabled |
| secondaryProxyExcludedHosts       | r Str      | 二级代理排除主机      | lipc-get-prop com.lab126.dpmManager secondaryProxyExcludedHosts |
| isTogglePopularHighlightsDisabled | r Int      | 切换热门标注是否禁用  | lipc-get-prop com.lab126.dpmManager isTogglePopularHighlightsDisabled |
| maxPasscodeAttempts               | r Int      | 最大密码尝试次数      | lipc-get-prop com.lab126.dpmManager maxPasscodeAttempts      |
| isEditPersonalInfoDisabled        | r Int      | 编辑个人信息是否禁用  | lipc-get-prop com.lab126.dpmManager isEditPersonalInfoDisabled |
| notifyDeviceControlsStateChange   | w Str      | 通知设备控制状态变更  | lipc-set-prop com.lab126.dpmManager notifyDeviceControlsStateChange "state" |
| isDeregisterDisabled              | r Int      | 注销是否禁用          | lipc-get-prop com.lab126.dpmManager isDeregisterDisabled     |
| isEditDeviceNameDisabled          | r Int      | 编辑设备名称是否禁用  | lipc-get-prop com.lab126.dpmManager isEditDeviceNameDisabled |
| isEditPasscodeDisabled            | r Int      | 编辑密码是否禁用      | lipc-get-prop com.lab126.dpmManager isEditPasscodeDisabled   |
| secondaryProxyPort                | r Str      | 二级代理端口          | lipc-get-prop com.lab126.dpmManager secondaryProxyPort       |
| isTogglePublicNotesDisabled       | r Int      | 切换公共笔记是否禁用  | lipc-get-prop com.lab126.dpmManager isTogglePublicNotesDisabled |
| validateParentalControlsPassword  | rw Has     | 验证家长控制密码      | lipc-set-prop com.lab126.dpmManager validateParentalControlsPassword |

## 10、其他服务

### FreeTime服务

| 属性                             | 权限(类型) | 说明             | 命令示例                                                     |
| :------------------------------- | :--------- | :--------------- | :----------------------------------------------------------- |
| isDownloadInProgress             | r Int      | 下载是否进行中   | lipc-get-prop com.lab126.freetime isDownloadInProgress     |
| refreshCache                     | w Str      | 刷新缓存         | lipc-set-prop com.lab126.freetime refreshCache ""          |
| launchProfile                    | w Str      | 启动配置文件     | lipc-set-prop com.lab126.freetime launchProfile "profile"  |
| setParentalPin                   | w Str      | 设置家长PIN      | lipc-set-prop com.lab126.freetime setParentalPin "pin"     |
| getWordDefinition                | rw Has     | 获取单词定义     | lipc-get-prop com.lab126.freetime getWordDefinition        |
| subscriptionStatus               | r Int      | 订阅状态         | lipc-get-prop com.lab126.freetime subscriptionStatus       |
| validateParentalPin              | w Str      | 验证家长PIN      | lipc-set-prop com.lab126.freetime validateParentalPin "pin" |
| rejectSubscriptionOffer          | w Str      | 拒绝订阅优惠     | lipc-set-prop com.lab126.freetime rejectSubscriptionOffer "" |
| emitClaimEvent                   | w Str      | 发出声明事件     | lipc-set-prop com.lab126.freetime emitClaimEvent "event"   |
| updateChildrenSubscriptionStatus | w Str      | 更新儿童订阅状态 | lipc-set-prop com.lab126.freetime updateChildrenSubscriptionStatus "status" |
| getAllowlistedBooks              | rw Has     | 获取允许列表书籍 | lipc-get-prop com.lab126.freetime getAllowlistedBooks      |
| getChildProfiles                 | r Str      | 获取儿童配置文件 | lipc-get-prop com.lab126.freetime getChildProfiles         |
| allowlist                        | w Str      | 添加到允许列表   | lipc-set-prop com.lab126.freetime allowlist "book"         |

