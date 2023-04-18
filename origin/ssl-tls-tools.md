# SSL证书生成工具

# 一、简介

## 数字证书的标准

- **X.509版本号：**指出该证书使用了哪种版本的X.509标准，版本号会影响证书中的一些特定信息
  序列号：由CA给予每一个证书分配的唯一的数字型编号，当证书被取消时，实际上是将此证书序列号放入由CA签发的CRL（Certificate Revocation List证书作废表，或证书黑名单表）中。这也是序列号唯一的原因
- **签名算法标识符：**用来指定CA签署证书时所使用的签名算法，常见算法如RSA
- **签发者信息：**颁发证书的实体的 X.500 名称信息。它通常为一个 CA
- **证书的有效期：**证书起始日期和时间以及终止日期和时间；指明证书在这两个时间内有效。
- **主题信息：**证书持有人唯一的标识，在 Internet上应该是唯一的
- **发布者的数字签名：**这是使用发布者私钥生成的签名，以确保这个证书在发放之后没有被撰改过。
- **证书的公钥：**包括证书的公钥、算法(指明密钥属于哪种密码系统)的标识符和其他相关的密钥参数

## 数字证书的常见格式

- **.CSR(certificate signing request)文件**：证书请求文件，这个并不是证书，而是向证书颁发机构获得签名证书的申请文件。含有公钥信息。生成该文件时需要用到自己的私钥。

- **CER**：存放证书文件可以是二进制编码或者BASE64编码

- **CRT**：证书可以是DER编码，也可以是PEM编码，在linux系统中比较常见

- **pem**：该编码格式在RFC1421中定义，但他也同样广泛运用于密钥管理，实质上是 Base64 编码的二进制内容

  PEM 是明文格式的, 以 “`-----BEGIN CERTIFICATE-----` 开头，已 `-----END CERTIFICATE-----` 结尾”, 中间是经过 base64 编码的内容, Apache 和 NGINX 服务器偏向于使用这种编码格式，也是 openssl 默认采用的信息存放方式。PEM 其实就是把 DER 的内容进行了一次 base64 编码。

- **DER**：用于二进制DER编码的证书。这些证书也可以用CER或者CRT作为扩展名

- **JKS**：java的密钥存储文件,二进制格式,是一种 Java 特定的密钥文件格式， JKS的密钥库和私钥可以用不同的密码进行保护

- **p12/PFX**：包含所有私钥、公钥和证书。其以二进制格式存储，也称为 PFX 文件，在windows中可以直接导入到密钥区，密钥库和私钥用相同密码进行保护

## 证数参数含义

- **C：国家**（Country Name）
- **ST：** 省份（State or Province Name）
- **L：** 城市（Locality Name）
- **O：** 公司（Organization Name）
- **OU：** 部门（Organizational Unit Name）
- **CN：** 产品名（Common Name）
- **emailAddress：** 邮箱（Email Address）
- **req_distinguished_name ：**根据情况进行修改
- **alt_names：** 127.0.0.1修改为服务器实际的IP或DNS地址，例如：IP.1=127.0.0.1或DNS.1 =broker.xxx.com

# 二、OpenSSL

- OpenSSL是TLS/SSL协议的开源实现，提供开发库和命令行程序；

- OpenSSL一共实现了4种非对称加密算法，包括DH算法、RSA算法、DSA算法和椭圆曲线算法（EC）。DH算法一般用户密钥交换。RSA算法既可以用于密钥交换，也可以用于数字签名，当然，如果你能够忍受其缓慢的速度，那么也可以用于数据加密。DSA算法则一般只用于数字签名。

## 1.对称加密

> **openssl enc -ciphername [-in filename] [-out filename] [-pass arg] [-e] [-d] [-a/-base64] [-A] [-k password] [-kfile filename] [-K key] [-iv IV] [-S salt] [-salt] [-nosalt] [-z] [-md] [-p] [-P] [-bufsize number] [-nopad] [-debug] [-none]  [-engine id]**
>
> -in <file>       输入文件
> -out <file>    输出文件
> -pass <arg>    密码
> -e                      encrypt 加密操作
> -d                      decrypt 解密操作
> -a/-base64      base64 encode/decode, depending on encryption flag 是否将结果base64编码
> -k                      passphrase is the next argument
> -kfile                 passphrase is the first line of the file argument
> -md                  指定密钥生成的摘要算法 默认MD5
> -S                      salt in hex is the next argument 用于加盐加密
> -K/-iv                key/iv in hex is the next argument 加密所需的key和iv向量
> -[pP]                 print the iv/key (then exit if -P) 是否需要在控制台输出生成的 key和iv向量
> -bufsize <n>   buffer size 读写文件的I/O缓存，一般不需要指定
> -nopad            disable standard block padding 禁止标准填充
> -engine e         use engine e, possibly a hardware device 指定三方加密设备
> Cipher Types  以下是部分算法，我们可以选择用哪种算法加密
> 				-aes-128-cbc    -aes-128-cbc-hmac-sha1  -aes-128-cfb
>
> ​    		 -aes-128-cfb1   -aes-128-cfb8     -aes-128-ctr
>
>   			-aes-128-ecb     -aes-128-gcm     -aes-128-ofb   ......

```bash
# 原始文本
echo "1234567890abc" > plain.txt

# 加密文本
openssl enc -aes-128-cbc -pbkdf2 -in plain.txt -out encrypt.txt 

# 解密文本
openssl aes-128-cbc -d -pbkdf2 -in encrypt.txt -out encrypt_decrypt.txt
```

## 2.常规操作

###  ①生成私钥

> **openssl genrsa [-out filename] [-passout arg] [-des] [-des3] [-idea] [numbits]**
>
> **选项说明：**
> **-out filename       ：将生成的私钥保存至filename文件，若未指定输出文件，则为标准输出。**
> **-numbits                ：指定要生成的私钥的长度，默认为1024。该项必须为命令行的最后一项参数。**
> **-des|-des3|-idea  ：指定加密私钥文件用的算法，这样每次使用私钥文件都将输入密码，太麻烦所以很少使用。**
> **-passout args        ：加密私钥文件时，传递密码的格式，如果要加密私钥文件时单未指定该项，则提示输入密码。传递密码的args的格式见一下格式。**
> 　　**pass:password   ：password表示传递的明文密码**
> 　　**env:var         ：从环境变量var获取密码值**
> 　　**file:filename   ：filename文件中的第一行为要传递的密码。若filename同时传递给"-passin"和"-passout"选项，则filename的第一行为"-passin"的值，第二行为"-passout"的值**
> 　　**stdin                   ：从标准输入中获取要传递的密码**

```bash
openssl genrsa -out private.key 2048
```

### ②私钥生成公钥

公钥提取自私钥

```bash
openssl rsa -pubout -in private.key -out public.key
```

### ③公钥加密

```bash
openssl rsautl -encrypt -inkey public.key  -pubin -in msg.txt  -out rsa_msg.txt
```

### ④私钥解密

```bash
openssl rsautl -decrypt -inkey private.key -in rsa_msg.txt -out dersa_msg.txt
```

### ⑤数字签名: 私钥加密公钥解密

如果是用私钥进行加密，公钥解密叫做数字签名，因为私钥只有一份，用公钥解密出来验证确认是你用这个私钥做的签名，这就是签名和验证。

```bash
openssl dgst -sign private.key -sha256 -out sign.msg.txt msg.txt 
```

公钥验证签名，需要对比源文件验证：

```bash
openssl dgst -verify public.key  -sha256 -signature sign.msg.txt msg.txt
```

### ⑥查看证书内容

```bash
# 查看证书内容
openssl x509 -noout -text -in server.crt

# 查看证书的过期时间
openssl x509 -in ca.crt -noout -dates

# 查看证书的subject和颁发者
openssl x509 -in server.crt -subject -issuer -noout
```

### ⑦去除key文件密码

```bash
openssl rsa -in server.key -out server.key
```

## 3. 自建CA签名证书

自签名证书分为

- 自签名私有证书（无法被吊销）
- 自签名CA证书（可以被吊销）

### ①生成CA根证书

>  生成CA私钥（*.key*）-->   生成CA证书请求（*.csr*）-->  自签名得到根证书（*.crt*）

- 生成CA认证机构的证书密钥key：`openssl genrsa -des3 -out ca.key 4096`

- 用私钥ca.key生成CA认证机构的证书ca.crt

  ```bash
  openssl req -x509 -new -nodes -sha256 \
    -key ca.key \
    -days 36500 \
    -subj "/C=CN/ST=Shanghai/L=Shanghai/O=Curiouser/OU=devops/CN=test.curiouser.com/CN=localhost/CN=127.0.0.1" \
    -out ca.crt
  ```

- 查看证书内容：`openssl x509 -in ca.crt -noout -text`

### ②用自签根证书给用户证书签名

>  生成私钥（*.key*）-->   生成证书请求（*.csr*）-->   用CA根证书签名得到证书（*.crt*）

```bash
# 生成网站的密钥server.key
openssl genrsa -des3 -out server.key 4096
# 生成网站证书的请求文件server.csr
openssl req -new \
  -subj "/C=CN/ST=Shanghai/L=Shanghai/O=Curiouser/OU=devops/CN=test.curiouser.com/CN=localhost/CN=127.0.0.1" \
  -key server.key \
  -out server.csr
```

cert.ext

```ini
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = localhost
IP.2 = 127.0.0.1
DNS.3 = test.curiouser.com
```

```bash
# 使用虚拟的CA认证机构的证书ca.crt，来对自己网站的证书请求文件server.csr进行处理，生成签名后的证书server.crt
openssl x509 -req  \
  -CAcreateserial \
  -CA ca.crt \
  -CAkey ca.key \
  -CAserial serial \
  -extfile cert.ext \
  -in server.csr \
  -out server.crt \
  -days 3650
```

使用CA验证一下证书是否通过

```bash
 openssl verify -CAfile ca.crt server.crt
 # server.crt: OK
```

# 三、Let's Encrypt

Let's Encrypt是一个于2015年三季度推出的数字证书认证机构，旨在以自动化流程消除手动创建和安装证书的复杂流程，并推广使万维网服务器的加密连接无所不在，为安全网站提供免费的SSL/TLS证书。

Let's Encrypt由互联网安全研究小组（缩写ISRG）提供服务。主要赞助商包括电子前哨基金会、Mozilla基金会、Akamai以及思科。2015年4月9日，ISRG与Linux基金会宣布合作。

用以实现新的数字证书认证机构的协议被称为自动证书管理环境（ACME）。GitHub上有这一规范的草案，且提案的一个版本已作为一个Internet草案发布。

Let's Encrypt宣称这一过程将十分简单、自动化并且免费

# 四、ACME.sh

简单来说acme.sh 实现了 acme 协议, 可以从 let‘s encrypt 生成免费的证书。
acme.sh 有以下特点：

- 一个纯粹用Shell（Unix shell）语言编写的ACME协议客户端。
- 完整的ACME协议实施。 支持ACME v1和ACME v2 支持ACME v2通配符证书
- 简单，功能强大且易于使用。你只需要3分钟就可以学习它。
- Let's Encrypt免费证书客户端最简单的shell脚本。
- 纯粹用Shell编写，不依赖于python或官方的Let's Encrypt客户端。
- 只需一个脚本即可自动颁发，续订和安装证书。 不需要root/sudoer访问权限。
- 支持在Docker内使用，支持IPv6

Github：https://github.com/acmesh-official/acme.sh



```bash
curl  https://get.acme.sh | sh
```

# 五、Certbot

https://certbot.eff.org/

```bash
wget https://dl.eff.org/certbot-auto
chmod a+x ./certbot-auto
./certbot-auto --help
```

# 六、CFSSL

CFSSL是CloudFlare开源的一款PKI/TLS工具。 CFSSL 包含一个命令行工具 和一个用于 签名，验证并且捆绑TLS证书的 HTTP API 服务。 使用Go语言编写。

CFSSL包括：

- 一组用于生成自定义 TLS PKI 的工具
- `cfssl`程序，是CFSSL的命令行工具
- `multirootca`程序是可以使用多个签名密钥的证书颁发机构服务器
- `mkbundle`程序用于构建证书池
- `cfssljson`程序，从`cfssl`和`multirootca`程序获取JSON输出，并将证书，密钥，CSR和bundle写入磁盘

PKI借助数字证书和公钥加密技术提供可信任的网络身份。通常，证书就是一个包含如下身份信息的文件：

- 证书所有组织的信息
- 公钥
- 证书颁发组织的信息
- 证书颁发组织授予的权限，如证书有效期、适用的主机名、用途等
- 使用证书颁发组织私钥创建的数字签名

需要安裝`CFSSL`工具，这将会用來建立 TLS Certificates

Github： https://github.com/cloudflare/cfssl

下载地址： https://pkg.cfssl.org/

```bash
export CFSSL_URL="https://pkg.cfssl.org/R1.2"
wget "${CFSSL_URL}/cfssl_linux-amd64" -O /usr/local/bin/cfssl
wget "${CFSSL_URL}/cfssljson_linux-amd64" -O /usr/local/bin/cfssljson
chmod +x /usr/local/bin/cfssl /usr/local/bin/cfssljson
```

# 七、easy-rsa

## 1. 安装

```bash
yum install easy-rsa
apt install easy-rsa
apk add easy-rsa
brew install easy-rsa
docker run -it --rm cmd.cat/easyrsa easyrsa --help
```

## 2、命令

- **初始化pki：**`easyrsa init-pki`
- **创建CA：**`easyrsa build-ca`
- **生成服务器证书请求：**`easyrsa gen-req server nopass`

# 八、使用操作

## 1. Nginx

```bash
server {
    listen 443 ssl;
    ssl_certificate      cert/cert-file-name.pem; # 证书文件
    ssl_certificate_key  cert/cert-file-name.key; # 证书私钥文件
    ssl_session_timeout 5m;                       # 会话缓存过期时间
    ssl_protocols TLSv1.1 TLSv1.2 TLSv1.3;        # 表示使用的TLS协议的类型
    ssl_prefer_server_ciphers on;                 # 设置协商加密算法时，优先使用服务端的加密套件
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4; #表示使用的加密套件的类型。
    server_name test.curiouser.com;
    set $app test;
    if ($time_iso8601 ~ '(\d{4}-\d{2}-\d{2})') {
        set $tttt $1;
    }
    error_log  /var/log/nginx/nginx-test-443-error.log;
    access_log  /var/log/nginx/nginx-tes1t-443-access-$tttt.log  main;
    location / {
      root ~/test;
      index index.html index.htm;
      autoindex on;
    }
}
# 将所有HTTP请求通过rewrite指令重定向到HTTPS。
server {
    listen 80;
    server_name test.curiouser.com;  # 需要将yourdomain替换成证书绑定的域名。
    rewrite ^(.*)$ https://$host$1;  # 将所有HTTP请求通过rewrite指令重定向到HTTPS。
    location / {
        index index.html index.htm;
    }
}
```

## 2. Apache

```bash
<VirtualHost *:443>     
    ServerName   #修改为申请证书时绑定的域名。                    
    DocumentRoot  /data/www/hbappserver/public          
    SSLEngine on   
    SSLProtocol all -SSLv2 -SSLv3 # SSL协议支持的协议，去掉不安全的协议。
    SSLCipherSuite HIGH:!RC4:!MD5:!aNULL:!eNULL:!NULL:!DH:!EDH:!EXP:+MEDIUM   # 加密套件
    SSLHonorCipherOrder on
    SSLCertificateFile      cert/domain_name_public.crt # 证书文件
    SSLCertificateKeyFile   cert/domain_name.key        # 证书密钥文件
    SSLCertificateChainFile cert/domain_name_chain.crt  # 证书链文件；证书链开头如果有#字符，请删除。
</VirtualHost>

# 设置HTTP请求自动跳转HTTPS。
<VirtualHost *:80>     
    ServerName   #修改为申请证书时绑定的域名。                    
    DocumentRoot  /data/www/hbappserver/public  
    # 设置HTTP请求自动跳转HTTPS
    RewriteEngine on
    RewriteCond %{SERVER_PORT} !^443$
    RewriteRule ^(.*)$ https://%{SERVER_NAME}$1 [L,R]
</VirtualHost>
```

## 3. Linux发行版导入自签证书

### **CentOS/Redhat/Fedora**

```bash
yum install -y ca-certificates

cp ca.crt /usr/local/share/ca-certificates/
update-ca-certificates
# 或者
cp *.crt /etc/pki/ca-trust/source/anchors/
update-ca-trust extract
```

### Ubuntu/Debian

```bash
apt-get install ca-certificates

cp ca.crt /usr/local/share/ca-certificates/
update-ca-certificates
# 或者
cp cacert.pem /usr/share/ca-certificates
dpkg-reconfigure ca-certificates
```

### Alpine

```bash
apk add --no-cache ca-certificates
mv my.crt /usr/local/share/ca-certificates/
update-ca-certificate
```

## 4. 命令行工具使用

- **Curl**

  ```bash
  curl --cacert ./openssl/ca.crt  -v https://test.curiouser.com/test
  ```

## 5. 浏览器使用

### ①浏览器Firefox

<img src="../assets/openssl-selfca-import-firefox-1.jpg?lastModify=1668568032" alt="img" style="zoom:33%;" />

<img src="../assets/openssl-selfca-import-firefox-2.jpg?lastModify=1668568032" alt="img" style="zoom:33%;" />

### ②MacOS 

- 将自签CA的证书拖进：`钥匙串-->系统-->证书`中即可导入

<img src="../assets/opensll-selfca-macos-import.jpg?lastModify=1668568032" alt="img" style="zoom:33%;" />

- 信任

<img src="../assets/opensll-selfca-macos-import-2.jpg?lastModify=1668568032" alt="img" style="zoom:33%;" />

# 九、证书格式转换

以下证书格式之间是可以互相转换的。
![img](/Users/curiouser/code/gitbooks-devops-roadmap/assets/http-ssl-format-transf.jpg)

## 1. 将JKS格式证书转换成PFX格式

使用JDK中自带的Keytool工具，将JKS格式证书文件转换成PFX格式。例如将 server.jks证书文件转换成 server.pfx证书文件：

```bash
keytool -importkeystore -srckeystore D:\server.jks -destkeystore D:\server.pfx
        -srcstoretype JKS -deststoretype PKCS12
```

## 2. 将PFX格式证书转换为JKS格式

使用JDK中自带的Keytool工具，将PFX格式证书文件转换成JKS格式。例如将 server.pfx证书文件转换成 server.jks证书文件：

```bash
keytool -importkeystore -srckeystore D:\server.pfx -destkeystore D:\server.jks
        -srcstoretype PKCS12 -deststoretype JKS
```

## 3. 将PEM/KEY/CRT格式证书转换为PFX格式

使用 [OpenSSL工具](https://www.openssl.org/)，将KEY格式密钥文件和CRT格式公钥文件转换成PFX格式证书文件。例如，将您的KEY格式密钥文件（server.key）和CRT格式公钥文件（server.crt）拷贝至OpenSSL工具安装目录，使用OpenSSL工具执行以下命令将证书转换成 server.pfx证书文件：

```bash
openssl pkcs12 -export -out server.pfx -inkey server.key -in server.crt
```

## 4. 将PFX转换为PEM/KEY/CRT

使用 [OpenSSL工具](https://www.openssl.org/)，将PFX格式证书文件转化为KEY格式密钥文件和CRT格式公钥文件。例如，将您的PFX格式证书文件拷贝至OpenSSL安装目录，使用OpenSSL工具执行以下命令将证书转换成server.pem证书文件KEY格式密钥文件（server.key）和CRT格式公钥文件（server.crt）：

- `openssl pkcs12 -in server.pfx -nodes -out server.pem`
- `openssl rsa -in server.pem -out server.key`
- `openssl x509 -in server.pem -out server.crt`

**说明** 此转换步骤是专用于通过Keytool工具生成私钥和CSR申请证书文件的，并且通过此方法您可以在获取到PEM格式证书公钥的情况下分离私钥。在您实际部署数字证书时，请使用通过此转换步骤分离出来的私钥和您申请得到的公钥证书匹配进行部署。

## 5、其他转换

- **PEM 转为 DER：**`openssl x509 -in cacert.crt -outform der -out cacert.der`

- **DER 转为 PEM：**`openssl x509 -in cert.crt -inform der -outform pem -out cacert.pem`

- **CER 转化为 PEM：**`openssl x509  -in <cert_name>.cer -out <cert_name>.pem  -outform PEM`

- **将x509转换为PEM：**`openssl x509 -in certificatename.cer -outform PEM -out certificatename.pem`

- **PKCS7转换为PEM：**`openssl pkcs7 -print_certs -in certificatename.p7b -out certificatename.pem`

- **PEM转换为P7B：**`openssl crl2pkcs7 -nocrl -certfile certificatename.pem -out certificatename.p7b -certfile CACert.cer`

- **pfx转换为PEM：**`openssl pkcs12 -in certificatename.pfx -out certificatename.pem`

- **PFX to PKCS#8：**

  - PFX to PEM：`openssl pkcs12 -in certificatename.pfx -nocerts -nodes -out certificatename.pem`
  - PEM to PKCS8：`openSSL pkcs8 -in certificatename.pem -topk8 -nocrypt -out certificatename.pk8`

- **P7B to PFX：**

  PKCS#7或P7B格式以Base64 ASCII格式存储，文件扩展名为.p7b或.p7c。一个P7B文件只包含证书和连锁证书（中间CA），不包含私钥。支持P7B文件的最常见的平台是Microsoft Windows和Java Tomcat。

  - P7B to CER ：`openssl pkcs7 -print_certs -in certificatename.p7b -out certificatename.cer`
  - CER and Private Key to PFX：`openssl pkcs12 -export -in certificatename.cer -inkey privateKey.key -out certificatename.pfx -certfile  cacert.cer`

# 十、其他操作

## 1. openssl命令行获取服务器SSL证书

```bash
openssl s_client -showcerts -connect {HOSTNAME}:{PORT} </dev/null 2>/dev/null|openssl x509 -outform PEM > www.test.com.ssl.pem
```

```bash
openssl s_client -connect {HOSTNAME}:{PORT} -showcerts
```

参考：

1. https://superuser.com/questions/97201/how-to-save-a-remote-server-ssl-certificate-locally-as-a-file

# 参考

- https://blog.csdn.net/yehuozhili/article/details/105214221
- https://blog.csdn.net/liwei16611/article/details/83686674
- https://www.cnblogs.com/even160941/p/16068449.html
- https://juejin.cn/post/7163440404480655367#comment
- https://www.cnblogs.com/handsomeBoys/p/6556336.html
- https://blog.csdn.net/caijiwyj/article/details/106840234
- https://www.jianshu.com/p/2676cdc688f9
- https://blog.csdn.net/xcndafad/article/details/122138791
- https://ningyu1.github.io/site/post/51-ssl-cert/
- https://blog.csdn.net/weixin_30439131/article/details/97570820
- https://www.cnblogs.com/haolb123/p/16553055.html
- https://www.helay.net/bible/detail2022103071.html
