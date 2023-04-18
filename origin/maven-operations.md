# Maven常见操作

# 1、设置代理

## 方式一：settings.xml中配置

```xml
<settings>
...省略...
  <proxies>
    <proxy>
      <active>true</active>
      <protocol>http</protocol>
      <host>proxy.somewhere.com</host>
      <port>8080</port>
      <username>proxyuser</username>
      <password>somepassword</password>
      <nonProxyHosts>www.google.com|*.somewhere.com</nonProxyHosts>
    </proxy>
  </proxies>

...省略...
</settings>
```

## 方式二：设置环境变量

```bash
export MAVEN_OPTS="-DsocksProxyHost=127.0.0.1 -DsocksProxyPort=$PORT"

# 或者
-Dhttp.proxyHost=<host> -Dhttp.proxyPort=<port> -Dhttps.proxyHost=<host> -Dhttps.proxyPort=<port>
```

