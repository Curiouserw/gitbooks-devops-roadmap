# CIS基准

# 一、简介

互联网安全中心（CIS，Center for Internet Security）是一个非营利性组织。该组织针对操作系统、数据库、中间件等应用制定配置策略的基准（即CIS Benchmark），以改善企业和组织的网络安全性、合规性，提升安全建设和发展态势。

[CIS 基准](https://www.cisecurity.org/cis-benchmarks/)是安全配置系统的配置基线和最佳做法。 每则指导建议都参考了一个或多个 [CIS 控制措施](https://www.cisecurity.org/controls/)，可帮助组织改进其网络防御能力。 CIS 控制措施与许多已建立的标准和规章框架对应，包括 NIST 网络安全框架 (CSF) 和 NIST SP 800-53、ISO 27000 系列标准、PCI DSS、HIPAA 等等。

CIS 基准提供两个级别的安全设置：

- “级别 1”推荐可在任何系统上配置的基本安全要求，这些要求应很少或不会导致服务中断或功能降低。
- “级别 2”推荐需要更高安全性的环境安全设置，这些设置可能会导致功能降低。

## CIS 基准与法规合规性

CIS 基准与安全和数据隐私监管框架保持一致或一一“对应”，包括[ NIST（美国国家标准技术研究院）网络安全框架](https://www.ibm.com/cn-zh/topics/nist)、PCI DSS（支付卡行业数据安全标准）(PCI DSS)、HIPAA（健康保险可移植性和责任法案）以及 ISO/EIC 2700。 因此，在受这些类型法规约束的行业中运营的任何组织都可以通过遵循 CIS 基准，在合规性方面取得重大进展。 此外，CIS 控制措施和 CIS 强化映像有助于组织遵守 GDPR（欧盟的《通用数据保护条例》）。

## CIS 基准核心类别

- **1. 操作系统基准：**涵盖核心操作系统的安全配置，如 Microsoft Windows、Linux 和 Apple OSX。 其中包括有关本地和远程访问限制、用户档案、驱动程序安装协议和 Internet 浏览器配置的最佳实践指南。
- **2. 服务器软件基准：**涵盖广泛使用的服务器软件的安全配置，包括 Microsoft Windows Server、SQL Server、[VMware](https://www.ibm.com/cn-zh/topics/vmware)、[Docker](https://www.ibm.com/cn-zh/topics/docker) 和 [Kubernetes](https://www.ibm.com/cn-zh/topics/kubernetes)。 这些基准包括有关配置 Kubernetes PKI 证书、API 服务器设置、服务器管理控制、vNetwork 策略和存储限制的建议。
- **3. 云提供商基准：**为 Amazon Web Services (AWS)、Microsoft Azure、Google、IBM 和其他受欢迎的[公有云](https://www.ibm.com/cn-zh/topics/public-cloud)提供安全配置。 其中包括配置身份和访问管理 (IAM)、系统日志记录协议、网络配置和合规性保护的准则。
- **4. 移动设备基准：**解决移动操作系统（包括 iOS 和 Android）的问题，关注开发人员选项和设置、操作系统隐私配置、浏览器设置和应用权限等领域。
- **5. 网络设备基准**：为 Cisco、Palo Alto Networks、Juniper 等公司的网络设备和适用硬件提供一般及特定于供应商的安全配置指南。
- **6. 桌面软件基准：**涵盖一些最常用的桌面软件应用的安全配置，包括 Microsoft Office 和 Exchange Server、Google Chrome、Mozilla Firefox 和 Safari 浏览器。 这些基准侧重于电子邮件隐私和服务器设置、移动设备管理、默认浏览器设置和第三方软件拦截。
- **7. 多功能打印设备基准：**概述在办公室环境中配置多功能打印机的安全最佳实践，涵盖固件更新、TCP/IP 配置、无线访问配置、用户管理和文件共享等主题。

# 二、基线检测工具

**Kubernetes CIS Benchmark :** https://github.com/aquasecurity/kube-bench

**UNIX like Linux  CIS Benchmark：**https://github.com/CISOfy/Lynis

**Windows 安全基线检查工具：**

- https://www.microsoft.com/en-us/download/details.aspx?id=55319
- https://learn.microsoft.com/zh-cn/windows/security/operating-system-security/device-management/windows-security-configuration-framework/security-compliance-toolkit-10

## 参考：

- https://www.ibm.com/cn-zh/topics/cis-benchmarks