# 摄像头：视频+音频+云台

# 一、简介

# 二、基础知识

# 三、视频流RTSP

# 四、云台控制ONVIF

## 2、移动方式

### Absolute绝对移动

- **目标**：将摄像头移动到指定的绝对位置。
- **参数**：`pan`、`tilt` 和 `zoom` 分别表示摄像头在水平方向、垂直方向和缩放方向上的位置，取值范围分别为 (-1.0 到 1.0) 和 (0.0 到 1.0)。
- **步骤**
  - 创建 `AbsoluteMove` 请求对象，并将 `ProfileToken` 和 `Position`（包括 `PanTilt` 和 `Zoom`）传入。
  - 调用 `ptz_service.AbsoluteMove()` 方法执行移动命令。
- **返回值**：如果操作成功，返回 `True`，否则返回 `False`。

### Relative相对移动

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



### Continuous续移动

- **目标**：根据指定的速度持续移动摄像头，直到停止命令。
- **参数**：`pan_speed`、`tilt_speed` 和 `zoom_speed` 分别表示水平方向、垂直方向和缩放方向上的速度，取值范围为 (-1.0 到 1.0) 和 (0.0 到 1.0)。`duration` 表示持续移动的时间（秒）。
- **步骤**
  - 创建 `ContinuousMove` 请求对象，并传入 `ProfileToken` 和 `Velocity`（包括 `PanTilt` 和 `Zoom` 速度）。
  - 调用 `ptz_service.ContinuousMove()` 方法开始持续移动。
  - 使用 `time.sleep(duration)` 控制移动持续的时间，然后调用 `stop()` 函数停止摄像头移动。
- **返回值**：如果操作成功，返回 `True`，否则返回 `False`。

# 参考

https://github.com/linkease/onvif
