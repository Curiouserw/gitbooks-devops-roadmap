# SSL与TLS

# 一、简介

- **SSL(Secrue Socket Layer 安全套接层)**：

  SSL(Secure Socket Layer 安全套接层)是基于HTTPS下的一个协议加密层，最初是由网景公司（Netscape）研发，后被IETF（The Internet Engineering Task Force - 互联网工程任务组）标准化后写入（RFCRequest For Comments 请求注释），RFC里包含了很多互联网技术的规范！

  起初是因为HTTP在传输数据时使用的是明文（虽然说POST提交的数据时放在报体里看不到的，但是还是可以通过抓包工具窃取到）是不安全的，为了解决这一隐患网景公司推出了SSL安全套接字协议层，SSL协议位于TCP/IP协议与各种应用层协议之间，为数据通讯提供安全支持。SSL协议可分为两层： SSL记录协议（SSL Record Protocol）：它建立在可靠的传输协议（如TCP）之上，为高层协议提供数据封装、压缩、加密等基本功能的支持。 SSL握手协议（SSL Handshake Protocol）：它建立在SSL记录协议之上，用于在实际的数据传输开始前，通讯双方进行身份认证、协商加密算法、交换加密密钥等。

- **TLS(Transport Layer Security安全传输层协议)**

  由于HTTPS的推出受到了很多人的欢迎，在SSL更新到3.0时，IETF对SSL3.0进行了标准化，并添加了少数机制(但是几乎和SSL3.0无差异)，标准化后的IETF更名为TLS1.0(Transport Layer Security 安全传输层协议)，可以说TLS就是SSL的新版本3.1，并同时发布“RFC2246-TLS加密协议详解”



# 二、HTTP请求SSL证书认证流程







# 三、SSL证书

## 1、证书类型

| 证书类型     | 适用网站类型               | 公信等级 | 认证强度                                   | 安全性 | 支持的证书品牌                                               |
| :----------- | :------------------------- | :------- | :----------------------------------------- | :----- | :----------------------------------------------------------- |
| DV域名型     | 个人网站                   | 一般     | CA机构审核个人网站真实性、不验证企业真实性 | 一般   | DigiCert、GeoTrust、GlobalSign、<br/>vTrus（国产）、WoSign（国产） |
| OV企业型     | 政府组织、企业、教育机构等 | 高       | CA机构审核组织及企业真实性                 | 高     | DigiCert）、GeoTrust、<br/>GlobalSign、CFCA（国产）、vTurs   |
| EV企业增强型 | 大型企业、金融机构等       | 最高     | 严格认证                                   | 最高   | DigiCert、GeoTrust、CFCA（国产）                             |

## 2、主流Web服务软件支持的证书

一般来说，主流的Web服务软件，通常都基于OpenSSL和Java两种基础密码库。

- Tomcat、Weblogic、JBoss等Web服务软件，一般使用Java提供的密码库。通过Java Development Kit （JDK）工具包中的Keytool工具，生成Java Keystore（JKS）格式的证书文件。
- Apache、Nginx等Web服务软件，一般使用OpenSSL工具提供的密码库，生成PEM、KEY、CRT等格式的证书文件。
- IBM的Web服务产品，如Websphere、IBM Http Server（IHS）等，一般使用IBM产品自带的iKeyman工具，生成KDB格式的证书文件。
- 微软Windows Server中的Internet Information Services（IIS）服务，使用Windows自带的证书库生成PFX格式的证书文件。

## 3、证书文件

- *.DER或*.CER文件： 这样的证书文件是二进制格式，只含有证书信息，不包含私钥。
- *.CRT文件： 这样的证书文件可以是二进制格式，也可以是文本格式，一般均为文本格式，功能与 *.DER及*.CER证书文件相同。
- *.PEM文件： 这样的证书文件一般是文本格式，可以存放证书或私钥，或者两者都包含。 *.PEM 文件如果只包含私钥，一般用*.KEY文件代替。
- *.PFX或*.P12文件： 这样的证书文件是二进制格式，同时包含证书和私钥，且一般有密码保护。

您也可以使用记事本直接打开证书文件。如果显示的是规则的数字字母，例如：

```sql
—–BEGIN CERTIFICATE—–
MIIE5zCCA8+gAwIBAgIQN+whYc2BgzAogau0dc3PtzANBgkqh......
—–END CERTIFICATE—–
```

那么，该证书文件是文本格式的。

- 如果存在`——BEGIN CERTIFICATE——`，则说明这是一个证书文件。
- 如果存在`—–BEGIN RSA PRIVATE KEY—–`，则说明这是一个私钥文件。
