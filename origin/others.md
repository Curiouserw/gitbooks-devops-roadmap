# 零散汇总

# 一、软件版本周期

α、β、λ 常用来表示软件测试过程中的三个阶段。

- α 是第一阶段，一般只供内部测试使用；
- β是第二个阶段，已经消除了软件中大部分的不完善之处，但仍有可能还存在缺陷和漏洞，一般只提供给特定的用户群来测试使用；

- λ是第三个阶段，此时产品已经相当成熟，只需在个别地方再做进一步的优化处理即可上市发行。

## 开发期

- Alpha(α)：预览版，或者叫内部测试版；一般不向外部发布，会有很多Bug；一般只有测试人员使用。

- Beta(β)：测试版，或者叫公开测试版；这个阶段的版本会一直加入新的功能；在 Alpha版之后推出。

- RC(Release Candidate)：最终测试版本；可能成为最终产品的候选版本，如果未出现问题则可发布成为正式版本

多数开源软件会推出两个RC版本，最后的 RC2 则成为正式版本。

## 完成期

- Stable：稳定版；来自预览版本释出使用与改善而修正完成。

- GA(General Availability)：正式发布的版本；在国外都是用GA来说明release版本的。

- RTM(Release to Manufacturing)：给生产商的release版本；RTM版本并不一定意味着创作者解决了软件所有问题；仍有可能向公众发布前更新版本。

​       另外一种RTM的称呼是RTW（Release To Web），表示正式版本的软件发布到Web网站上供客户免费下载。

- RTL(Retail)：零售版；是真正的正式版，正式上架零售版。

以Windows 7为例，RTM版与零售版的版本号是一样的。

## 其他表述

- OEM(Original Equipment Manufacturer)：原始设备制造商；是给计算机厂商随着计算机贩卖的，也就是随机版；只能随机器出货，不能零售。只能全新安装，不能从旧有操作系统升级。包装不像零售版精美，通常只有一面CD和说明书(授权书)。

- RVL：号称是正式版，其实RVL根本不是版本的名称。它是中文版/英文版文档破解出来的。

- EVAL：而流通在网络上的EVAL版，与“评估版”类似，功能上和零售版没有区别。

## 参考：

1. https://blog.csdn.net/waynelu92/java/article/details/73604172

# 二、armel、armhf和arm64的区别

出于低功耗、封装限制等种种原因，以前的一些ARM处理器没有独立的硬件浮点运算单元，需要手写软件来实现浮点运算。随着技术发展，现在高端的ARM处理器基本都具备了硬件执行浮点操作的能力。这样，新旧两种架构之间的差异，就产生了两个不同的嵌入式应用程序二进制接口（EABI）——软浮点与矢量浮点（VFP）。但是软浮点（soft float）和硬浮点（hard float）之间有向前兼容却没有向后兼容的能力，也就是软浮点的二进制接口（EABI）仍然可以用于当前的高端ARM处理器。

- armel：是arm eabi little endian的缩写。eabi是软浮点二进制接口，这里的e是embeded，是对于嵌入式设备而言。

- armhf：是arm hard float的缩写。

- arm64：64位的arm默认就是hf的，因此不需要hf的后缀。

- armel和armhf的区别体现在浮点运算上，它们在进行浮点运算时都会使用fpu，但是armel传参数用普通寄存器，而armhf传参数用的是fpu的寄存器，因此armhf的浮点运算性能更高。

# 三、VBS获取出口 IP 并发送邮件

```vbscript
Private Function getRouterIP()
	Dim http
	Set http = CreateObject("Msxml2.ServerXMLHTTP")
	http.open "GET", "http://myip.ipip.net/", False
	http.send
	responseStatus = http.status
	responseBody = http.responseBody
	Set http = nothing
	If responseStatus= 200 Then
	    dim objstream 
	    set objstream = CreateObject("adodb.stream")
	    objstream.Type = 2 
	    objstream.Open 
	    objstream.WriteText responseBody , 1 
	    objstream.Position = 0 
	    objstream.Charset = "UTF-8" 
	    objstream.Position = 2 
	    ip =objstream.ReadText 
	    objstream.close 
	    Set objstream = Nothing
	    getRouterIP = ip
	Else
		MsgBox responseStatus
	End If
End Function

Sub sendMail()
	Const schema = "http://schemas.microsoft.com/cdo/configuration/"  
	Set CDO = CreateObject("CDO.Message")
	With CDO.Configuration.Fields
		.Item(schema & "sendusing") = 2									'发送端口		
    .Item(schema & "smtpserver") = "smtp.163.com"		'smtp服务器
		.Item(schema & "smtpauthenticate") = 1					'是否需要提供用户名和密码，0是不提供 
		.Item(schema & "sendusername") = "*****"				'发送者邮箱帐号'
		.Item(schema & "sendpassword") = "*****"				'发送者邮箱密码
		.Item(schema & "smtpserverport") = 25						'SMTP服务器端口
		.Item(schema & "smtpusessl") = True
		.Item(schema & "smtpconnectiontimeout") = 60
		.Update
	End With
	CDO.From = "************@163.com"									'发送者邮件地址'
	CDO.To = "************@163.com"										'接收者邮件地址'
	CDO.Subject = now() & "-" & "You Know That"				'邮件主题'
	CDO.TextBody = getRouterIP 												'邮件内容'
	CDO.Send																					'发送邮件执行'
End Sub

sendMail()
```
