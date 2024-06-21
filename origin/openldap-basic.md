# OpenLDAP

# 一、简介

LDAP 具有两个国家标准，分别是X.500 和LDAP。OpenLDAP 是基于X.500 标准的，而且去除了X.500 复杂的功能并且可以根据自我需求定制额外扩展功能，但与X.500 也有不同之处，例如OpenLDAP 支持TCP/IP 协议等，目前TCP/IP 是Internet 上访问互联网的协议。

# 二、基础概念

**DN（Distinguished Name，区别名称）**：DN 是条目在 LDAP 中的唯一标识符，它表示了条目在 LDAP 树中的位置。DN 的格式类似于文件系统中的路径

**Base DN**：Base DN 是在进行 LDAP 搜索时指定的起始点，搜索将从该点开始递归地向下进行。例如，如果 Base DN 设置为 `dc=example,dc=com`，那么搜索将从这个节点开始。

# 二、安装

## 1、 二进制

Ubuntu / Debian

```bash
apt update && DEBIAN_FRONTEND=noninteractive apt install -y slapd ldap-utils gosa-schema
```

https://www.linuxquestions.org/questions/linux-server-73/script-install-slapd-with-admin-ldap-password-4175426002/

https://superuser.com/questions/1786748/how-to-skip-the-password-typing-when-installing-library

## 2、Docker/k8s

https://github.com/osixia/docker-openldap

# 三、初始化配置LDAP

- `/etc/ldap/slapd.d/`: 根据slapd.conf配置信息生成的文件
- `/etc/ldap/schema/`: OpenLDAP的schema存放目录
- `/var/lib/ldap/`: OpenLDAP数据目录

日志级别

| **Level** | **Keyword**    | **Description**                                         |
| --------- | -------------- | ------------------------------------------------------- |
| -1        | any            | enable all debugging                                    |
| 0         |                | no debugging                                            |
| 1         | (0x1 trace)    | trace function calls                                    |
| 2         | (0x2 packets)  | debug packet handling                                   |
| 4         | (0x4 args)     | heavy trace debugging                                   |
| 8         | (0x8 conns)    | connection management                                   |
| 16        | (0x10 BER)     | print out packets sent and received                     |
| 32        | (0x20 filter)  | search filter processing                                |
| 64        | (0x40 config)  | configuration processing                                |
| 128       | (0x80 ACL)     | access control list processing                          |
| 256       | (0x100 stats)  | stats log connections/operations/results                |
| 512       | (0x200 stats2) | stats log entries sent                                  |
| 1024      | (0x400 shell)  | print communication with shell backends                 |
| 2048      | (0x800 parse)  | print entry parsing debugging                           |
| 16384     | (0x4000 sync)  | syncrepl consumer processing                            |
| 32768     | (0x8000 none)  | only messages that get logged whatever log level is set |

```bash
basedn='dc=devops,dc=test,dc=top' \
rootuserdn='cn=admin,'$basedn \
ldap_admin_passw=`specialchars='-_=#' && \
astring=$(echo $(date +%s)$RANDOM | md5sum | base64 | tr -dc A-Za-z0-9 | head -c 16 ) && \
randomchar=${specialchars:$RANDOM % ${#specialchars}:1} && \
randompos=$(( $RANDOM % ( ${#astring} + 1 ) ))  && \
echo ${astring:0:$randompos}${randomchar}${astring:$randompos}` && \
echo "==============ldap 管理员admin密码: "$ldap_admin_passw"==============" && \
ldap_admin_passw_sha=`slappasswd -h {SHA} -s $ldap_admin_passw` && \
cp /usr/share/slapd/slapd.init.ldif /etc/ldap/slapd.init.ldif && \
sed -i -e "/nis.ldif/a include: file:///etc/ldap/schema/gosa/rfc2307bis.ldif" \
  -e "/nis.ldif/d" \
  -e "s|olcRootPW:.*|olcRootPW: $ldap_admin_passw_sha|g" \
  -e "s/olcRootDN:.*/olcRootDN: $rootuserdn/g" \
  -e "s/olcSuffix:.*/olcSuffix: $basedn/g" \
  /etc/ldap/slapd.init.ldif && \
slapadd -F "/etc/ldap/slapd.d/" -b "cn=config" -l /etc/ldap/slapd.init.ldif && \
chown -R openldap:openldap /etc/ldap/slapd.d && \
systemctl restart slapd && \
ldapsearch -LLL -Y external -H ldapi:/// -b cn=schema,cn=config -s one dn && \
ldapwhoami -x -D "$rootuserdn" -w $ldap_admin_passw
```

# 四、创建组织结构

```bash
ldapadd -x -D "$rootuserdn" -w $ldap_admin_passw <<EOF
dn: $basedn
objectClass: top
objectClass: domain
dc: devops

dn: ou=systems,$basedn
objectClass: organizationalUnit
objectClass: top
ou: systems

dn: cn=openvpn,ou=systems,$basedn
objectClass: groupOfNames
objectClass: top
cn: openvpn
member: cn=test,ou=users,$basedn

dn: ou=users,$basedn
objectClass: organizationalUnit
objectClass: top
ou: users

dn: cn=test,ou=users,$basedn
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
objectClass: posixAccount
objectClass: top
cn: test
gidNumber: 10001
homeDirectory: /home/test
sn:: 546L5rC45LmQ
uid: 10001
uidNumber: 10001
EOF

ldapsearch -x -b "$basedn" -D "$rootuserdn" -w $ldap_admin_passw \
   -s base "(objectclass=domain)"
```





# 五、开启Overlay功能

Overlays是OpenLDAP的一种软件功能模块，它通过给其他功能提供一些钩子(*Hook*)来实现类似backend的功能。memberof和refint都是一种Overlay。        

## 1、memberOf与Refint功能模块

- **MemberOf**

  Member是连接Group和Entry的桥梁。

  如果某个组中通过 member 属性新增了一个用户，OpenLDAP 便会自动在该用户上创建一个 memberOf 属性，其值为该组的 dn。遗憾的是，OpenLDAP 默认并不启用这个特性，因此我们需要通过相关的配置开启它。

- **Refint**

  Refint全称是`Referential Integrity`，可以理解为“引用一致性”，它用于支持动态更新。在配置memberof的时候，会有以下一条配置`olcMemberOfRefInt: TRUE`。该行配置会启用`Referential Integrity`。当配置refint之后，那么修改用户信息后。用户对应组的属性会被动态更新。

## 2、加载配置模块

```bash
ldapadd -x -D "$rootuserdn" -w $ldap_admin_passw <<EOF
# Load modules
dn: cn=module,cn=config
cn: module
objectClass: olcModuleList
olcModuleLoad: memberof
olcmoduleload: refint
olcModulePath: /usr/lib/ldap/

#memberof overlay
dn: olcOverlay=memberof,olcDatabase={1}mdb,cn=config
objectClass: olcMemberOf
objectClass: olcOverlayConfig
objectClass: olcConfig
objectClass: top
olcOverlay: memberof
olcMemberOfDangling: ignore
olcMemberOfRefInt: TRUE
olcMemberOfGroupOC: groupOfUniqueNames
olcMemberOfMemberAD: uniqueMember
olcMemberOfMemberOfAD: memberOf

#refint overlay
dn: olcOverlay=refint,olcDatabase={1}mdb,cn=config
objectClass: olcOverlayConfig
objectClass: olcRefintConfig
olcOverlay: refint
olcRefintAttribute: owner
olcRefintAttribute: manager
olcRefintAttribute: uniqueMember
olcRefintAttribute: member
olcRefintAttribute: memberOf
EOF
```



```bash
# cn=module 的名称请查看ls -l /etc/ldap/slapd.d/cn\=config/ |grep module 如果没有直接写module ，Docker中默认为module{0};
# olcDatabase={1}mdb 请确认ls -l /etc/ldap/slapd.d/cn=config/ |grep olcDatabase 的名称，docker中默认的为{1}mdb,Centos rpm 安装默认为{2}hdb。

bash -c 'cat >/etc/ldap/01-memberOf.ldif << EOF
# Load memberof module
dn: cn=module{0},cn=config
changetype: modify
add: olcModuleLoad
olcModuleLoad: memberof
olcModuleLoad: refint

# Backend memberOf overlay
dn: olcOverlay=memberof,olcDatabase={1}mdb,cn=config
changetype: add
objectClass: olcConfig
objectClass: olcMemberOf
objectClass: olcOverlayConfig
objectClass: top
olcOverlay: memberof
olcMemberOfDangling: ignore
olcMemberOfRefInt: TRUE
# groupOfNames: 一个成员可以属于多个组的情况 ; groupOfUniqueNames: 每个成员只能属于一个组
olcMemberOfGroupOC: groupOfNames
olcMemberOfMemberAD: member
olcMemberOfMemberOfAD: memberOf
EOF'


bash -c 'cat >/etc/ldap/02-refint.ldif << EOF

# Backend refint overlay
dn: olcOverlay=refint,olcDatabase={1}mdb,cn=config
changetype: add
objectClass: olcOverlayConfig
objectClass: olcRefintConfig
objectClass: top
olcOverlay: refint
olcRefintAttribute: memberof member manager owner uniqueMember
EOF'


bash -c 'cat >/etc/ldap/03-index.ldif << EOF
# Add indexes
dn: olcDatabase={1}mdb,cn=config
changetype:  modify
replace: olcDbIndex
olcDbIndex: uid eq
olcDbIndex: mail eq
olcDbIndex: memberOf eq
olcDbIndex: entryCSN eq
olcDbIndex: entryUUID eq
olcDbIndex: objectClass eq
EOF'


for f in $(find /etc/ldap -mindepth 1 -maxdepth 1 -type f -name 0\*.ldif | sort); do  
	if grep -iq changetype $f ; then 
		ldapmodify -Y EXTERNAL -Q -H ldapi:/// -f $f ; 
	else
		ldapadd -Y EXTERNAL -Q -H ldapi:/// -f $f ; 
	fi;
done

# 用于检索配置树的根节点，即 cn=config 的 DN。
ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=config dn

# 用于检索 olcDatabase={1}mdb,cn=config 下的 memberof overlay 的配置信息。
ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b olcOverlay={0}memberof,olcDatabase={1}mdb,cn=config
ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b olcOverlay={1}refint,olcDatabase={1}mdb,cn=config
```





```bash

# 用于检索 olcDatabase={1}mdb,cn=config 的 DN，即数据库配置的 DN。
ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b olcDatabase={1}mdb,cn=config dn

# 检索 memberof overlay 的 DN，但在 OpenLDAP 中，overlay的DN通常不是一个单独的节点，而是与数据库关联的属性
ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b "olcOverlay={0}memberof,olcDatabase={1}mdb,cn=config" dn

# 查看 memberof overlay 的配置
ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b "olcOverlay={0}memberof,olcDatabase={1}mdb,cn=config"
```

## 3、测试memberof与Refint的功能

### ①创建测试用户与用户组测试数据

```bash
ldapadd -x -D "cn=admin,dc=devops,dc=test,dc=top" -w $ldap_admin_passw <<EOF
dn: cn=testgroup1,dc=devops,dc=test,dc=top
objectClass: top
objectClass: groupOfNames
cn: testgroup
description: 测试组-1
member: cn=testuser1,dc=devops,dc=test,dc=top

dn: cn=testgroup2,dc=devops,dc=test,dc=top
objectClass: top
objectClass: groupOfNames
cn: testgroup
description: 测试组-2
member: cn=testuser2,dc=devops,dc=test,dc=top

dn: cn=testuser1,dc=devops,dc=test,dc=top
objectClass: top
objectClass: person
objectClass: organizationalPerson
cn: testuser1
sn: testuser1

dn: cn=testuser2,dc=devops,dc=test,dc=top
objectClass: top
objectClass: person
objectClass: organizationalPerson
cn: testuser2
sn: testuser2
EOF
```

### ② 测试查询

```bash
ldapsearch -x -LLL -H ldap://localhost -b "cn=testuser2,dc=devops,dc=test,dc=top" memberOf

ldapsearch -x -H ldap:/// -b "cn=test,ou=users,dc=devops,dc=test,dc=top" -D "cn=admin,dc=devops,dc=test,dc=top" -w $ldap_admin_passw  memberOf
```

# 五、导入 RFC2307Bis

1. **功能差异：**
   - NIS 模式是为了向后兼容 UNIX 系统中使用的 NIS 服务而设计的。它定义了一组属性和对象类，用于存储与用户、组、主机等相关的信息。
   - RFC2307bis 模式是为了与 RFC 2307 中定义的 LDAP 网络信息服务（NIS）模式保持兼容，并添加了一些额外的属性和对象类，以便更好地支持 UNIX 环境中的用户和组信息的存储和检索。
2. **属性和对象类的差异：**
   - NIS 模式包含了一些与 NIS 相关的属性和对象类，如 `nisNetgroup`、`nisMap`、`nisObject` 等。
   - RFC2307bis 模式在 NIS 模式的基础上扩展了属性和对象类，如添加了 `posixAccount`、`posixGroup`、`shadowAccount` 等，以更好地支持 UNIX 环境中的用户和组管理。
3. **应用场景：**
   - NIS 模式通常用于那些仍在使用 NIS 服务的旧系统或需要与 NIS 服务进行交互的系统。
   - RFC2307bis 模式更适用于需要更现代化的 UNIX 用户和组管理功能的系统，它提供了更多的属性和对象类，使得用户和组信息的存储更加灵活和完善。

总的来说，RFC2307bis 模式可以看作是对 NIS 模式的扩展和改进，提供了更多功能和更好的兼容性，因此在现代化的 UNIX 环境中更常见和推荐使用。



```bash
cp /etc/ldap/add/rfc2307bis.* /etc/ldap/schema/
mkdir -p /tmp/schema
slaptest -f /etc/ldap/add/rfc2307bis.conf -F /tmp/schema
```

查询已导入的 schema

```bash
ldapsearch -LLL -Y external -H ldapi:/// -b cn=schema,cn=config -s one dn
```

测试

```bash
$ ldapadd -W -H ldap:/// -D 'cn=Manager,dc=example,dc=com' <<LDIF
# Create a new organization
dn: dc=example,dc=com
objectClass: dcObject
objectClass: organization
dc: example
o: My Organization
description: The Example organization

# Create role for the directory manager
dn: cn=Manager,dc=example,dc=com
objectClass: organizationalRole
cn: Manager
description: Directory Manager

# Create unit for groups
dn: ou=groups,dc=example,dc=com
objectClass: top
objectClass: organizationalUnit
ou: groups
description: Organizational unit for user groups

# Create unit for people
dn: ou=people,dc=example,dc=com
objectClass: top
objectClass: organizationalUnit
ou: people
description: Organizational unit for user accounts

# Create a test user in the people unit
dn: uid=test,ou=people,dc=example,dc=com
objectClass: top
objectClass: person
objectClass: posixAccount
uid: test
cn: Test User
sn: Test
uidNumber: 10001
gidNumber: 100
homeDirectory: /home/test
loginShell: /bin/bash
LDIF
```



https://bubblesorted.raab.link/content/replace-nis-rfc2307-rfc2307bis-schema-openldap

https://unofficialaciguide.com/2019/07/31/ldap-schemas-for-aci-administrators-rfc2307-vs-rfc2307bis/

# 六、常用命令

### ① 查看openLDAP的版本

```bash
bashslapd -VV
```

### ② 查看已启用的模块

```bash
ldapsearch -LLL -Y EXTERNAL -H ldapi:/// -b  cn=config | grep -i module
slapcat -n 0 | grep olcModuleLoad
```

### ③ 查询数据库配置

```bash
ldapsearch -LLL -Y EXTERNAL -H ldapi:/// -b cn=config olcDatabase | grep mdb
    # 输出
    #     SASL/EXTERNAL authentication started
    #     SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
    #     SASL SSF: 0
    #     dn: olcDatabase={1}mdb,cn=config
    #     olcDatabase: {1}mdb
```

### ④ 查看module模块路径

```bash
ldapsearch -LLL -Y EXTERNAL -H ldapi:/// -b cn=config | grep -i module

# Centos: /usr/lib64/openldap
# Ubuntu: 
#    memberof: /usr/lib/ldap/memberof.la
#    refint: /usr/lib/ldap/refint.la
```

### ⑤ 测试配置文件

```bash
slaptest -u 
```

### ⑥ 导入基本的Schema

```bash
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/ldap/schema/cosine.ldif
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/ldap/schema/nis.ldif
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/ldap/schema/inetorgperson.ldif
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/ldap/schema/ppolicy.ldif
```

### ⑦ 生成密码

```bash
slappasswd -h {SHA} -s 密码 
```

# 六、用户管理

## 1、新增只读用户

```bash
basedn='dc=devops,dc=test,dc=top' \
rootuserdn='cn=admin,'$basedn \
LDAP_READONLY_USER_PW=`specialchars='-_=#' && \
astring=$(echo $(date +%s)$RANDOM | md5sum | base64 | tr -dc A-Za-z0-9 | head -c 16 ) && \
randomchar=${specialchars:$RANDOM % ${#specialchars}:1} && \
randompos=$(( $RANDOM % ( ${#astring} + 1 ) ))  && \
echo ${astring:0:$randompos}${randomchar}${astring:$randompos}` && \
echo "==============ldap 只读用户名密码: "$ldap_admin_passw"==============" && \
LDAP_READONLY_USER_PW_SHA=`slappasswd -h {SHA} -s $LDAP_READONLY_USER_PW`

ldapadd -x -D $rootuserdn -w $ldap_admin_passw <<EOF
dn: cn=readonly,${basedn}
cn: readonly
objectClass: simpleSecurityObject
objectClass: organizationalRole
description: LDAP read only user
userPassword: ${LDAP_READONLY_USER_PW_SHA}
EOF
```



## 2、创建用户

```bash
dn: cn=uservpn,dc=bufx,dc=gov,dc=cn
cn: uservpn
sn: vpn
userPassword: {SSHA}abZPY3cWgMMtnNqtFyuX64xq7Mm0TNFd
objectClass: inetOrgPerson
objectClass: organizationalPerson
```



# 十一、卸载备份

```bash
systemctl stop slapd
rm -rf /etc/ldap/slapd.d/* /etc/ldap/slapd.init.ldif /var/lib/ldap/* /etc/ldap/0*.ldif
```



# 参考

- https://www.cnblogs.com/somata/p/OPENLDAPServerConfigAndPostManagement.html
- https://cloud.tencent.com/developer/article/1932586
- https://www.brianshowalter.com/blog/installing-configuring-openldap
- https://www.adimian.com/blog/how-to-enable-memberof-using-openldap/
- https://www.openldap.org/doc/admin25/guide.html
- https://www.openldap.org/software/release/announce_lts.html
- https://www.cnblogs.com/eagle6688/p/16990393.html
- https://tylersguides.com/guides/openldap-memberof-overlay/
- https://lework.github.io/2019/07/18/ldap/
- https://mayanbin.com/posts/enable-memberof-in-openldap
- https://github.com/osixia/docker-openldap/blob/master/image/service/slapd/assets/config/bootstrap/ldif/03-memberOf.ldif
- https://blog.csdn.net/qq_23191379/article/details/106867730
- https://blog.361way.com/ldap-adduser-grants/2825.html
- https://blog.csdn.net/xyy511/article/details/90521086
- https://blog.buubiu.com/open-ldap-%E5%88%9B%E5%BB%BA%E5%8F%AA%E8%AF%BB%E7%94%A8%E6%88%B7/
- https://www.cnblogs.com/husbandmen/p/13307381.html 