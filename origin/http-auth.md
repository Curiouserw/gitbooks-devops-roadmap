# HTTP Auth认证

# 一、简介

[HTTP Authorization ](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Authorization)头是一个用于在客户端与服务器之间传送认证凭证的 HTTP 请求头。它允许用户代理（比如 web 浏览器）向服务器提供身份验证信息，以响应服务器的身份验证请求。这类请求一般出现在服务器需要验证请求者是否有权访问某个资源时。Authorization 头是 HTTP 协议中定义的标准字段之一，其值通常包含凭证信息，如用户名和密码，或者是一个令牌，如 OAuth 或 JWT（JSON Web Token）。

[RFC 7235](https://datatracker.ietf.org/doc/html/rfc7235) 定义了一个 HTTP 身份验证框架，服务器可以用来质询（[challenge](https://developer.mozilla.org/zh-CN/docs/Glossary/Challenge)）客户端的请求，客户端则可以提供身份验证凭据。

质询与响应的工作流程如下：

1. 服务器端向客户端返回 [`401`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status/401)（Unauthorized，未被授权的）响应状态码，并在 [`WWW-Authenticate`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/WWW-Authenticate) 响应标头提供如何进行验证的信息，其中至少包含有一种质询方式。
2. 之后，想要使用服务器对自己身份进行验证的客户端，可以通过包含凭据的 [`Authorization`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Authorization) 请求标头进行验证。
3. 通常，客户端会向用户显示密码提示，然后发送包含正确的 `Authorization` 标头的请求。



# 二、认证机制

## 1、基本认证Basic Authentication

基本认证是最简单的 HTTP 认证机制，它是在 [RFC 7617](https://datatracker.ietf.org/doc/html/rfc7617) 中规定的，通过发送一个经过 Base64 编码的`用户名:密码`字符串来作为凭据信息的。尽管实现简单，但基本认证通常不被认为是安全的，因为base64 算法是可逆的，故一般配合HTTPS/TLS 协议使用以确保安全性。

- **认证流程**:
  1. 服务器返回 `401 Unauthorized`，标头 `WWW-Authenticate: Basic realm="User Visible Realm"`。
  2. 客户端发送 `Authorization: Basic <base64(username:password)>`。

## 2、摘要认证Digest Authentication

摘要认证在安全性上比基本认证更高一步，因为它使用了 MD5 散列函数来传输密码。尽管比基本认证更为安全，但摘要认证也有一些安全漏洞，并且在现代 Web 应用中不如其他认证机制流行。

**认证流程**:

1. 服务器返回 `401` 及 `nonce`（随机数）等信息，要求客户端进行认证。

   ```bash
   HTTP/1.1 401 Unauthorized
   WWW-Authenticate: Digest
   realm="**请求URI的安全域**",			# 请求的URI
   qop="auth,auth-int",						# 保护质量，包含auth（默认的）和auth-int（增加了报文完整性检测）两种策略，（可以为空，但是）不推荐为空值
   nonce="**服务端的nonce**",
   opaque="*****"
   ```

2. 客户端计算 response=`MD5(MD5(用户名:realm值:密码):nonce值:MD5(method:请求的URI))`。默认为MD5算法。发送请求。

   ```bash
   Authorization: Digest
   username="Mufasa",　 
   realm="testrealm@host.com",
   nonce="服务端的nonce",
   uri="/index.html", 									    # 请求的URI
   qop=auth,                             
   nc=00000001,														# nonce计数器，是一个16进制的数值，表示同一nonce下客户端发送出请求的数量。以便检测重复的请求
   cnonce="**客户端随机数**", 						    # 是一个不透明的字符串值，双方都会使用，以避免用明文文本。都可以查验对方的身份，并对消息完整性提供保护
   response="6629fae49393a05397450978507c4ef1", 		# 最终的摘要信息 ha3
   opaque="5ccc069c403ebaf9f0171e9517f40e41"　 ←　服务器端质询响应信息
   ```

3. 服务器如果认证成功，则返回相应的资源。如果认证失败，则仍返回401状态，要求重新进行认证。

注意事项：

1. 避免将密码作为明文在网络上传递，相对提高了HTTP认证的安全性。
2. 当用户为某个realm首次设置密码时，服务器保存的是以用户名，realm，密码为基础计算出的哈希值（ha1），而非密码本身。
3. 如果qop=auth-int，在计算ha2时，除了包括HTTP方法，URI路径外，还包括请求实体主体，从而防止PUT和POST请求表示被人篡改。
4. 但是因为nonce本身可以被用来进行摘要认证，所以也无法确保认证后传递过来的数据的安全性。

## 3、Token 认证，如 Bearer 令牌（Token Authentication）

Token 认证是一种更为安全的认证方法，广泛用于当前的 Web 应用中，特别是在 RESTful API 认证中。一个常见的实施方式是使用 Bearer tokens，客户端发送一个密钥（token），它由服务器验证且通常是持有密钥用户识别令牌。

## 4、OAuth

OAuth 是一个用于授权的开放标准，它允许用户让第三方应用访问自己存储在另一服务提供商上的信息，而无需将用户名和密码直接提供给第三方应用。OAuth 可以用来提供认证（OAuth 2.0）和授权，例如允许应用代表用户去访问 Google 或 Facebook 上的数据。

## 5、API 密钥（API Keys）


API 密钥是由服务器预先生成的一组字符，客户端将其作为访问 API 资源的凭据。API 密钥通常作为请求的一部分发送，可以放在 URL、请求头或请求体中。虽然 API 密钥方便易用，但比较适用于对安全需求不是特别高的场景，且通常与其他手段（如限制 IP 地址）结合使用来增加安全性。

# 三、工具或配置



### [使用 Apache 限制访问和 basic 身份验证](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Authentication#使用_apache_限制访问和_basic_身份验证)

要对 Apache 服务器上的目录进行密码保护，你需要一个 `.htaccess` 和 a `.htpasswd` 文件。

该 `.htaccess` 文件格式通常看起来像这样：

```
AuthType Basic
AuthName "Access to the staging site"
AuthUserFile /path/to/.htpasswd
Require valid-user
```

该 `.htaccess` 文件引用一个 `.htpasswd` 文件，其中每行用冒号（`:`）分隔的用户名和密码。你不能看到真实的密码因为它们是[散列](https://httpd.apache.org/docs/2.4/misc/password_encryptions.html)（在这个例子中是使用了 MD5）。你可以命名 `.htpasswd` 文件为你所喜欢的名字，但是应该保证这个文件不被其他人访问。(Apache 通常配置阻止访问 `.ht*` 类的文件).

```
aladdin:$apr1$ZjTqBB3f$IF9gdYAGlMrs2fuINjHsz.
user2:$apr1$O04r.y2H$/vEkesPhVInBByJUkXitA/
```

### [nginx 访问限制和 basic 认证](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Authentication#nginx_访问限制和_basic_认证)

在 nginx 配置中，你需要指定一个要保护的 location 并且 `auth_basic` 指令提供密码保护区域的名称。

`auth_basic_user_file` 指令指定包含加密的用户凭据 `.htpasswd` 文件，就像上面的 apache 例子。

```
location /status {
    auth_basic           "Access to the staging site";
    auth_basic_user_file /etc/apache2/.htpasswd;
}
```

### [使用 URL 中的身份凭据进行的访问](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Authentication#使用_url_中的身份凭据进行的访问)

许多客户端同时支持避免弹出登录框，而是使用包含用户名和密码的经过编码的 URL，如下所示：

```
https://username:password@www.example.com/
```

**这种 URL 已被弃用**。在 Chrome 中，URL 中的 `username:password@` 部分甚至会因为安全原因而被[移除](https://bugs.chromium.org/p/chromium/issues/detail?id=82250#c7)。Firefox 则会检查该站点是否真的需要身份验证，假如不是，则会弹出一个警告窗口：你即将使用用户名 `username` 登录 `www.example.com` 站点，但是该站点不需要进行身份验证。这可能是在试图进行欺诈。