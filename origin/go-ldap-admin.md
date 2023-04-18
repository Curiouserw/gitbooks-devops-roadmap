# Go LDAP Admin

# 一、简介

基于Go+Vue实现的openLDAP后台管理项目

Github：https://github.com/eryajf/go-ldap-admin

官网：http://ldapdoc.eryajf.net/

提供在线体验地址如下：

| 分类          | 地址                                                         | 用户名                    | 密码   |
| ------------- | ------------------------------------------------------------ | ------------------------- | ------ |
| go-ldap-admin | [http://demo-go-ldap-admin.eryajf.net](http://demo-go-ldap-admin.eryajf.net/) | admin                     | 123456 |
| phpLdapAdmin  | http://demo-go-ldap-admin.eryajf.net:8091/                   | cn=admin,dc=eryajf,dc=net | 123456 |

# 二、安装部署

## 1、docker

```bash
git clone https://github.com/eryajf/go-ldap-admin.git
cd docs/docker-compose
docker-compose up -d
```

- 本地进行访问：http://localhost:8090，用户名/密码：`admin`/`123456`

- 如果想要访问PhpLdapAdmin，则可访问：http://localhost:8091，用户名/密码：`cn=admin,dc=eryajf,dc=net`/`123456`