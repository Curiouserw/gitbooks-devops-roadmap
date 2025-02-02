# 攻防基础

# 一、简介

# 二、靶机环境

## 1、Vulhub

https://github.com/vulhub/vulhub

Vulhub是一个面向大众的开源漏洞靶场，无需docker知识，简单执行一条命令即可编译、运行一个完整的漏洞靶场镜像。

```bash
# 下载项目
wget https://github.com/vulhub/vulhub/archive/master.zip -O vulhub-master.zip
unzip vulhub-master.zip
cd vulhub-master
# 进入某一个漏洞/环境的目录
cd flask/ssti
# 自动化编译环境
docker compose build
# 启动整个环境
docker compose up -d
```

每个环境目录下都有相应的说明文件，请阅读该文件，进行漏洞/环境测试。测试完成后，删除整个环境

```bash
docker compose down -v
```

## 2、Vulnhub

https://www.vulnhub.com/

# 工具

## 1、[afrog](https://github.com/zan8in/afrog)：漏洞扫描工具

## 2、[**fscan**](https://github.com/shadow1ng/fscan) ：内网综合扫描工具

# ATT&CK

**MITRE ATT&CK**，全称是`MITRE Adversarial Tactics Techniques and Common Knowledge`，即**入侵者战术、技术和共有知识库**，是以[骇客](https://zh.wikipedia.org/wiki/駭客)的视角，针对[网络攻击](https://zh.wikipedia.org/wiki/網絡攻擊)入侵进行分类和说明的指南，由非营利组织MITRE所创建。

**MITRE**：https://www.mitre.org/（中文网站：https://attack.seccmd.net/）

MITRE（The MITRE Corporation）是一家**非营利性的美国研究和技术组织**，成立于1958年，最初从麻省理工学院林肯实验室分离出来。该组织在众多关键领域为美国政府提供系统工程、研究开发和信息技术支持服务，尤其以网络安全、国防技术、航空系统、医疗保健、生物识别技术和公共政策分析等方面的研究与应用闻名。

- 在网络安全领域，MITRE创建并维护了`Common Vulnerabilities and Exposures (CVE) 编号系统`，这是一个国际标准，用于标识已知的安全漏洞和 exp。
- MITRE还开发了`ATT&CK（Adversarial Tactics, Techniques & Common Knowledge）框架`，这是一种行业广泛采用的方法，用于描述对手在网络攻击中的战术、技术和过程（TTPs）

# 参考

- https://www.ddosi.org/att/