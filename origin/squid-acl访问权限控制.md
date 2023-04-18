
# 一、ACL概念

Squid提供了强大的代理控制机制，通过合理设置ACL（Access Control List，访问控制列表）并进行限制，可以针对源地址、目标地址、访问的URL路径、访问的时间等各种条件进行过滤。

ACL访问控制的步骤：

1. 使用acl配置项定义需要控制的条件
2. 通过http_access配置项对已定义的列表做“允许”或“拒绝”访问的控制

# 二、ACL用法概述

## 1、定义ACL访问列表

定义格式：

```bash
acl aclname acltype string1… 
#acl 列表名称 列表类型 列表内容 ...
acl aclname acltype "File_Path"… 
#acl 列表名称 列表类型 "文件路径" ...
#当使用文件时，该文件的格式为每行包含一个条目。
```

常用的ACL列表类型:

- **`src`**：指明源地址
  - acl aclname src ip-address/netmask ... 客户ip地址
  - acl aclname src addr1-addr2/netmask ... 地址范围
- **`dst`**：指明目标地址，即客户请求的服务器的IP地址。语法为：
  - acl aclname dst ip-address/netmask ... 
- **`srcdomain`**：指明客户所属的域，Squid将根据客户IP反向查询DNS。语法为：
  - acl aclname srcdomain foo.com ...   
- **`dstdomain`**：指明请求服务器所属的域，由客户请求的URL决定。语法为：
  - acl aclname dstdomain foo.com ...   此处需要注意的是：如果用户使用服务器IP而非完整的域名时，Squid将进行反向的DNS解析来确定其完整域名，如果失败，就记录为“none”。
- **`time`**：指明访问时间。语法如下：
  - acl aclname time [day-abbrevs] [h1:m1-h2:m2][hh:mm-hh:mm]
  - 日期的缩写指代关系如下：
    - S：指代Sunday
    - M：指代Monday
    - T：指代Tuesday
    - W：指代Wednesday
    - H：指代Thursday
    - F：指代Friday
    - A：指代Saturday
    - 另外，h1：m1必须小于h2：m2，表达式为[hh：mm-hh：mm]。
- **`port`**：指定访问端
  - acl aclname port 80 70 21 ...
  - acl aclname port 0-1024 ... 指定一个端口范围
- **`method`**：指定请求方法。比如：
  - acl aclname method GET POST ...
- **`url_regex`**：URL规则表达式匹配，语法为：
  - acl aclname url_regex[-i] pattern
- **`urlpath_regex`**：URL-path规则表达式匹配，略去协议和主机名。其语法为：
  - acl aclname urlpath_regex[-i] pattern

**Notes：**

1. acltype可以是任一个在ACL中定义的名称。
2. 任何两个ACL元素不能用相同的名字。
3. 每个ACL由列表值组成。当进行匹配检测的时候，多个值由逻辑或运算连接；换句话说，任一ACL元素的值被匹配，则这个ACL元素即被匹配。
4. 并不是所有的ACL元素都能使用访问列表中的全部类型。
5. 不同的ACL元素写在不同行中，Squid将这些元素组合在一个列表中。

## 2、http_access访问控制列表使用访问控制

根据访问控制列表允许或禁止某一类用户访问。如果某个访问没有相符合的项目，则默认为应用最后一条项目的“非”。比如最后一条为允许，则默认就是禁止。通常应该把最后的条目设为“deny all”或“allow all”来避免安全性隐患。使用该访问控制列表要注意如下问题：

- 这些规则按照它们的排列顺序进行匹配检测，一旦检测到匹配的规则，匹配检测就立即结束。
- 访问列表可以由多条规则组成。
- 如果没有任何规则与访问请求匹配，默认动作将与列表中最后一条规则对应。
- 一个访问条目中的所有元素将用逻辑与运算连接（如下所示）：
- http_access Action声明1 AND 声明2 AND
- 多个http_access声明间用或运算连接，但每个访问条目的元素间用与运算连接。
- 列表中的规则总是遵循由上而下的顺序。


# 三、ACL示例


1. 允许网段10.0.0.124/24以及192.168.10.15/24内的所有客户机访问代理服务器，并且允许在文件/etc/squid/guest列出的客户机访问代理服务器，除此之外的客户机将拒绝访问本地代理服务器：

    ```bash
    acl clients src 10.0.0.124/24 192.168.10.15/24
    acl guests src “/etc/squid/guest”
    acl all src 0.0.0.0/0.0.0.0
    http_access allow clients
    http_access allow guests
    http_access deny all
    其中，文件“/etc/squid/guest”中的内容为：
    172.168.10.3/24
    210.113.24.8/16
    10.0.1.24/25
    ```

2. 允许域名为job.net、gdfq.edu.cn的两个域访问本地代理服务器，其他的域都将拒绝访问本地代理服务器：

    ```bash
    acl permitted_domain src job.net gdfq.edu.cn
    acl all src 0.0.0.0/0.0.0.0
    http_access allow permitted_domain
    http_access deny all
    ```

3. 使用正则表达式，拒绝客户机通过代理服务器访问包含有诸如“sexy”等关键字的网站：

    ```bash
    acl deny_url url_regex -i sexy
    http_access deny deny_url
    ```

4. 拒绝客户机通过代理服务器访问文件中指定IP或者域名的网站，其中文件/etc/squid/ deny_ip中存放有拒绝访问的IP地址，文件/etc/squid/deny_dns中存放有拒绝访问的域名：

    ```bash
    acl deny_ip dst “etc/squid/deny_ip”
    acl deny_dns dst “etc/squid/deny_dns”
    http_access deny deny_ip
    http_access deny deny_dns
    ```

5. 允许和拒绝指定的用户访问指定的网站，其中，允许客户1访问网站http://www.sina.com.cn，而拒绝客户2访问网站http://www.163.com：

    ```bash
    acl client1 src 192.168.0.118
    acl client1_url url_regex ^http://www.sina.com.cn
    acl client2 src 192.168.0.119
    acl client2_url url_regex ^http://www.163.com
    http_access allow client1 client1_url
    http_access deny client2 client2_url
    ```

6. 允许所有的用户在规定的时间内（周一至周四的8：30到20：30）访问代理服务器，只允许特定的用户（系统管理员，其网段为：192.168.10.0/24）在周五下午访问代理服务器，其他的在周五下午一点至六点一律拒绝访问代理服务器：

    ```bash
    acl allclient src 0.0.0.0/0.0.0.0
    acl administrator 192.168.10.0/24
    acl common_time time MTWH 8:30-20:30
    acl manage_time time F 13:00-18:00
    http_access allow allclient common_time
    http_access allow administrator manage_time
    http_access deny manage_time
    ```



