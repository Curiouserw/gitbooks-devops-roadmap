# ONVIF：

# 一、简介

ONVIF（Open Network Video Interface Forum）是一个开放的行业论坛，旨在促进基于 IP 的物理安全产品的互操作性。ONVIF 规范定义了一系列标准，使不同厂商的设备（如摄像头、录像机等）能够通过通用协议进行通信。

# 二、ONVIF 规范

## 1、核心规范

- **Core Specification**（核心规范）：定义了基本的设备管理和发现功能。
- **Profile S**：用于视频流传输。
- **Profile G**：用于录像存储和检索。
- **Profile T**：增强了 H.265 编码支持和更高级的视频分析。
- **Profile M**：用于元数据分析，包括 AI 识别、物体检测等高级功能。
- **Profile C**：用于门禁控制。
- **Profile D**：用于设备配置管理。

## 2、设备发现

ONVIF 设备通常使用 **WS-Discovery** 协议进行自动发现，客户端可以通过 UDP 组播搜索局域网内的 ONVIF 兼容设备。

## 3、认证与安全

ONVIF 设备通常支持HTTP 认证方式：

- **Basic Authentication（基本认证）**：采用 Base64 编码用户凭据，安全性较低。
- **Digest Authentication（摘要认证）**：使用 MD5 哈希计算并结合 nonce 机制，提供更安全的身份

部分设备也支持 WS-Security 进行更安全的认证。此外，ONVIF 也开始支持基于 OAuth 和 TLS 的安全通信。

curl命令调试：`curl --digest -u '用户名:密码' 'http://192.168.1.12/onvif/event_service'`

# 三、RTSP

ONVIF 设备通常支持 **RTSP**（Real Time Streaming Protocol）协议用于视频流传输。RTSP URL 的格式通常如下：

```bash
rtsp://<用户名>:<密码>@<IP地址>:<端口>/<流路径>
# 554 是默认 RTSP 端口
# /live 是流路径，具体路径可能因设备厂商而异
```

## 视频编码格式

- **H.264**（主流编码格式，兼容性好）
- **H.265**（更高效的压缩比）
- **MJPEG**（无压缩格式，适用于特定应用）
- **MPEG-4**（较老的标准，逐步淘汰）

# 四、云台控制

ONVIF 提供了云台（Pan-Tilt-Zoom，PTZ）控制接口，使客户端可以远程控制摄像头的方向和缩放。

## 1、移动方式

### 绝对移动Absolute

- **目标**：将摄像头移动到指定的绝对位置。
- **参数**：`pan`、`tilt` 和 `zoom` 分别表示摄像头在水平方向、垂直方向和缩放方向上的位置，取值范围分别为 (-1.0 到 1.0) 和 (0.0 到 1.0)。
- **步骤**
  - 创建 `AbsoluteMove` 请求对象，并将 `ProfileToken` 和 `Position`（包括 `PanTilt` 和 `Zoom`）传入。
  - 调用 `ptz_service.AbsoluteMove()` 方法执行移动命令。
- **返回值**：如果操作成功，返回 `True`，否则返回 `False`。

### 相对移动Relative

- **目标**：根据当前摄像头位置进行相对移动。
- **参数**：`pan_offset`、`tilt_offset` 和 `zoom_offset` 分别表示水平方向、垂直方向和缩放方向上的偏移量，取值范围也分别是 (-1.0 到 1.0) 和 (0.0 到 1.0)。
- **步骤**
  - 创建 `RelativeMove` 请求对象，并传入 `ProfileToken` 和 `Translation`（包括 `PanTilt` 和 `Zoom` 偏移量）。
  - 调用 `ptz_service.RelativeMove()` 方法执行相对移动命令。
- **返回值**：如果操作成功，返回 `True`，否则返回 `False`。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<SOAP-ENV:Envelope
    xmlns:SOAP-ENV="http://www.w3.org/2003/05/soap-envelope"
    xmlns:wsdl="http://www.onvif.org/ver20/ptz/wsdl"
    xmlns:tt="http://www.onvif.org/ver10/schema">
    <SOAP-ENV:Header>
        <Security s:mustUnderstand="1"
            xmlns="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-secext-1.0.xsd">
            <UsernameToken>
                <Username>{{username}}</Username>
                <Password Type="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-username-token-profile-1.0#PasswordDigest">{{digest}}</Password>
                <Nonce EncodingType="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-soap-message-security-1.0#Base64Binary">{{nonce}}</Nonce>
                <Created xmlns="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-utility-1.0.xsd">{{created}}</Created>
            </UsernameToken>
        </Security>
    </SOAP-ENV:Header>
    <SOAP-ENV:Body>
        <wsdl:RelativeMove>
            <wsdl:ProfileToken>{{profile}}</wsdl:ProfileToken>
            <wsdl:Translation>
                <tt:PanTilt x="0.5" y="0.3"/>
                <tt:Zoom x="0.2"/>
            </wsdl:Translation>
            <wsdl:Speed>
                <tt:PanTilt x="0.8" y="0.8"/>
                <tt:Zoom x="0.5"/>
            </wsdl:Speed>
        </wsdl:RelativeMove>
    </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

### 持续移动Continuous

- **目标**：根据指定的速度持续移动摄像头，直到停止命令。
- **参数**：`pan_speed`、`tilt_speed` 和 `zoom_speed` 分别表示水平方向、垂直方向和缩放方向上的速度，取值范围为 (-1.0 到 1.0) 和 (0.0 到 1.0)。`duration` 表示持续移动的时间（秒）。
- **步骤**
  - 创建 `ContinuousMove` 请求对象，并传入 `ProfileToken` 和 `Velocity`（包括 `PanTilt` 和 `Zoom` 速度）。
  - 调用 `ptz_service.ContinuousMove()` 方法开始持续移动。
  - 使用 `time.sleep(duration)` 控制移动持续的时间，然后调用 `stop()` 函数停止摄像头移动。
- **返回值**：如果操作成功，返回 `True`，否则返回 `False`。

### 缩放控制Zoom

控制摄像头的焦距进行缩放。

### 预设位Preset Position

存储摄像头的固定位置，支持快速调用。

### 巡航Patrol/Tour

按预设路线自动移动摄像头。

# WebRTC

https://webrtcforthecurious.com/zh/docs/01-what-why-and-how/

# 参考

https://github.com/linkease/onvif
