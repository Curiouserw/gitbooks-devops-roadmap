# 一、用户认证

Openshift通过OAuth进行用户的认证。在Openshift的master节点上运行着一个内置的OAuth服务对用户的请求进行认证检查。一旦OAuth服务器通过登录信息确认了用户的信息，OAuth服务器就返回用户的访问Token。通过这个Token，用户可以在有效的时间内对系统进行访问。

    #登录命令
    $ oc login -u 用户名
    ​
    #查看以哪个用户登录的
    $ oc whoami
    $ oc whoami -t 查看当前用户当前Session的Token
    ​
    #system:admin是集群默认的管理员，该用户是一个特殊用户，它不能通过用户名密码登录，它也没有Token。

作为身份验证的登录信息，如用户名密码，并非保存在Openshift集群中，而是保存在用户信息管理系统中，这些用户信息管理系统在Openshift中被称为Identity Provider。但Openshift并不提供用户信息管理系统，而是提供了不同的适配器连接不同的用户信息管理系统。通过配置，Openshift可以连接到以下用户信息管理系统：
- LADP（Lightweight Directory Access Protocol）
- 微软的活动目录（Active Directory）
- AllowALL
- DenyAll
- HTPasswd
- Github

        #查看当前Openshift集群支持的用户信息管理系统
        cat /etc/origin/master/master-config.yaml|grep provider -A 3
            provider:
            apiVersion: v1
            file: /etc/origin/master/htpasswd
            kind: HTPasswdPasswordIdentityProvider

        #Htpasswd是Apache提供的一个基于文本文件管理用户名密码的用户信息管理工具


Openshift的用户管理，在后台创建用户时，会同时创建一个User对象和Identity对象（该对象保存了用户来源哪一个Identity Provider及用户信息）。

    #查看集群中所有用户
    $oc get user
    NAME        UID                                    FULL NAME   IDENTITIES
    admin       a04e0467-c8e7-11e7-b9d9-5254ac31d0ec               htpasswd_auth:admin
    dev         1ffbda60-cb72-11e7-bd9b-5254c1caedf4               htpasswd_auth:dev

    #查看用户的Identity对象
    $oc get identity
    NAME                      IDP NAME        IDP USER NAME   USER NAME   USER UID
    htpasswd_auth:admin       htpasswd_auth   admin           admin       a04e0467-c8e7-11e7-b9d9-5254ac31d0ec
    htpasswd_auth:dev         htpasswd_auth   dev             dev         1ffbda60-cb72-11e7-bd9b-5254c1caedf4
    Openshift的用户组管理。用户组的信息来源有两个：一个是Identity Provider，二是通过用户在Openshift中定义的。
    #通过oadm groups命令在Openshift中对组及组成员进行管理
    $> oadm groups

    #添加用户到用户组
    $> oadm groups add-users group_name user_name

    #查看用户组
    $> oc get group

    #创建用户组
    $> oadm groups new group_name

    #删除组
    $> oc delete group group_name

# 二、用户权限管理

1. 用户角色权限管理
   1. 授予及撤销用户某种角色
        ```bash
        oc policy add-role-to-user view test
        oc policy remove-role-from-user view test
        ```
   2. 查看项目的角色绑定关系
        ```bash
        oc get rolebinding -n 项目名
        ```
   3. 授予某用户对某项目的某角色
        ```bash
        oc policy add-role-to-user view test -n test
        ```
   4. 查看角色绑定的规则
        ```bash
        oc describe clusterrole registry-viewer
        ```
2. 用户管理
    1. 新增用户
        ```bash
        ansible masters -m shell -a "htpasswd -b /etc/origin/master/htpasswd  test test"
        ```
    2. 查看已创建的用户
        ```bash
        oc get user
        或
        cat /etc/origin/master/htpasswd
        ```
    3. 删除用户
        ```bash
        oc delete user test
        ansible masters -m shell -a "htpasswd -D /etc/origin/master/htpasswd ha"
        ```
3. 用户组管理
    1. 创建用户组、添加用户到用户组
        ```bash
        oc adm groups new test
        oc adm groups add-users test 用户1 用户2 用户3
        ```
    2. 查看创建的用户组及组内的成员用户
        ```bash
        oc get group
        ```
    3. 删除用户组
        ```bash
        oc delete group test
        ```