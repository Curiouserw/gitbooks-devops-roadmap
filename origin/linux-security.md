# Linux 安全架构

# 一、SSH相关

## 1、只允许某用户从指定IP地址登陆

    sed -i '$a AllowUsers CR@192.168.1.12 root@192.168.1.12' /etc/ssh/sshd_config ;\
    systemctl restart sshd

## 2、设置SSH空闲超时退出时间

`/etc/ssh/sshd_config`

    #ClientAliveInterval 0
    #ClientAliveCountMax 3
    修改成
    ClientAliveInterval 30    #（每30秒往客户端发送会话请求，保持连接）
    ClientAliveCountMax 3     #（去掉注释即可，3表示重连3次失败后，重启SSH会话）

`systemctl restart sshd`

## 3、 限制登陆访问尝试的验证次数

    MaxAuthTries 20

## 4、允许root用户登录

    sed -i 's/PermitRootLogin no/PermitRootLogin yes/g'  /etc/ssh/sshd_config ;\
    systemctl restart sshd

## 5、设置登录方式

```bash
# AuthorizedKeysFile   .ssh/authorized_keys   //公钥公钥认证文件
# PubkeyAuthentication yes   //可以使用公钥登录
# PasswordAuthentication no  //不允许使用密码登录
```

## 6、禁止使用空白密码用户访问

```bash
PermitEmptyPasswords no
```

## 7、程序用户禁止登录

- 不创建家目录
- 不允许 ssh登录

```bash
useradd -s /usr/sbin/nologin test
```

## 8、SSH登录事件通知至ntfy

`/etc/pam.d/sshd`

```bash
session optional pam_exec.so /usr/local/bin/ntfy-ssh-login.sh
```

`ntfy-ssh-login.sh`

```bash
#!/bin/bash
TOPIC_URL=http://test.curiouser.top:18070/ssh-notify
if [ "${PAM_TYPE}" = "open_session" ]; then
  curl -H tags:warning -H prio:high -d "SSH login to $(hostname): ${PAM_USER} from ${PAM_RHOST}" "${TOPIC_URL}"
fi
```

部署私有的ntfy

```bash
ntfy_server:
  image: 'binwiederhier/ntfy'
  restart: always
  container_name: ntfy_server
  command: serve --config /etc/ntfy/server.yml --cache-file /var/cache/ntfy/cache.db --listen-http :18070
  ports:
    - '18070:18070'
  volumes:
    - '/data/ntfy/data:/var/cache/ntfy'
    - '/data/ntfy/config:/etc/ntfy'
    - '/data/ntfy/data:/var/lib/ntfy/'
```

# 二、OS系统设置

## 1、访问控制配置文件的权限设置

```bash
chown root:root /etc/passwd /etc/shadow /etc/group /etc/gshadow
chmod 0644 /etc/group
chmod 0644 /etc/passwd
chmod 0400 /etc/shadow
chmod 0400 /etc/gshadow
```

## 2、密码安全策略

`/etc/pam.d/password-auth`

```bash
password    requisite     pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type=  difok=1 minlen=8 ucredit=-1 lcredit=-1 dcredit=-1 
```

- difok= 定义新密码中必须要有几个字符和旧密码不同
- minlen=新密码的最小长度
- ucredit= 新密码中可以包含的大写字母的最大数目。-1 至少一个
- lcredit=新密码中可以包含的小写字母的最大数
- dcredit=定新密码中可以包含的数字的最大数目

注：这个密码强度的设定只对"普通用户"有限制作用，root用户无论修改自己的密码还是修改普通用户的时候，不符合强度设置依然可以设置成功

## 3、用户帐号限制

- `/etc/login.defs`配置文件是设置用户帐号限制的文件，可配置密码的最大过期天数，密码的最大长度约束等内容。

- 该文件里的配置对root用户无效。此文件中的配置与 `/etc/passwd` 和 `/etc/shadow` 文件中的用户信息有冲突时，系统会以`/etc/passwd` 和 `/etc/shadow` 为准。
- `/etc/login.defs` 文件用于在Linux创建用户时，对用户的一些基本属性做默认设置，例如指定用户 UID 和 GID 的范围，用户的过期时间，密码的最大长度，等等。

| 设置项                   | 含义                                                         |
| ------------------------ | ------------------------------------------------------------ |
| MAIL_DIR /var/spool/mail | 创建用户时，系统会在目录 /var/spool/mail 中创建一个用户邮箱，比如 lamp 用户的邮箱是 /var/spool/mail/lamp。 |
| PASS_MAX_DAYS 99999      | 密码有效期，99999 是自 1970 年 1 月 1 日起密码有效的天数，相当于 273 年，可理解为密码始终有效。 |
| PASS_MIN_DAYS 0          | 表示自上次修改密码以来，最少隔多少天后用户才能再次修改密码，默认值是 0。 |
| PASS_MIN_LEN 5           | 指定密码的最小长度，默认不小于 5 位，但是现在用户登录时验证已经被 PAM 模块取代，所以这个选项并不生效。 |
| PASS_WARN_AGE 7          | 指定在密码到期前多少天，系统就开始通过用户密码即将到期，默认为 7 天。 |
| UID_MIN 500              | 指定最小 UID 为 500，也就是说，添加用户时，默认 UID 从 500 开始。注意，如果手工指定了一个用户的 UID 是 550，那么下一个创建的用户的 UID 就会从 551 开始，哪怕 500~549 之间的 UID 没有使用。 |
| UID_MAX 60000            | 指定用户最大的 UID 为 60000。                                |
| GID_MIN 500              | 指定最小 GID 为 500，也就是在添加组时，组的 GID 从 500 开始。 |
| GID_MAX 60000            | 用户 GID 最大为 60000。                                      |
| CREATE_HOME yes          | 指定在创建用户时，是否同时创建用户主目录，yes 表示创建，no 则不创建，默认是 yes。 |
| UMASK 077                | 用户主目录的权限默认设置为 077。                             |
| USERGROUPS_ENAB yes      | 指定删除用户的时候是否同时删除用户组，准备地说，这里指的是删除用户的初始组，此项的默认值为 yes。 |
| ENCRYPT_METHOD SHA512    | 指定用户密码采用的加密规则，默认采用 SHA512，这是新的密码加密模式，原先的 Linux 只能用 DES 或 MD5 加密。 |

## 4、用户登录次数限制



## 5、系统超时退出

```bash
# 600表示超过600秒无操作即断开连接
echo "export TMOUT=600" >> /etc/profile
source /etc/profile
```

# 三、shell安全

## 1、历史命令安全

禁止shell的history记录密码相关的环境变量

> Bash
>
> ```bash
> # 开起不记录模式
> set +o history
> # 关闭不记录模式
> set -o history
> ```
>
> ZSH
>
> ```bash
> setopt HIST_IGNORE_SPACE
> 
> # 之后命令前面加一个空格即可不被记录
>  command
> ```

# 四、Fail2Ban

Fail2Ban 是一款入侵防御软件，可以保护服务器免受暴力攻击。 它是用 Python 编程语言编写的。

Fail2Ban 基于auth 日志文件工作，默认情况下它会扫描所有 auth 日志文件，如 /var/log/auth.log、
/var/log/apache/access.log 等，并禁止带有恶意标志的IP，比如密码失败太多，寻找漏洞等等标志。

通常，Fail2Ban 用于更新防火墙规则，用于在指定的时间内拒绝 IP 地址。 它也会发送邮件通知。

Fail2Ban 为各种服务提供了许多过滤器，如 ssh、apache、nginx、squid、named、mysql、nagios 等。

Fail2Ban 能够降低错误认证尝试的速度，但是它不能消除弱认证带来的风险。

这只是服务器防止暴力攻击的安全手段之一。

```bash
# Debian / Ubuntu
apt install fail2ban

```

ssh

```bash
[sshd]

enabled = true
port    = ssh
backend = systemd
maxretry = 3
findtime = 300
bantime = 3600
ignoreip = 127.0.0.1

# enabled  – 是否启用该规则
# port     – 要监听的端口。例如：ssh的22端口
# backend  –  指定用于获取文件修改的后端。由于所有现代 Linux 系统都依赖于 systemd 的日志服务，因此我们将其指定为后端。
# maxretry – 某个 IP 在被禁止之前尝试失败的次数。
# findtime – “maxretry”登录失败将导致禁止的时间范围（以秒为单位）。我们指定了 300 秒，即 5 分钟。
# bantime  – IP 应保持禁止状态的持续时间（以秒为单位）。在我们的例子中，我们设置了 3600 秒，这意味着在接下来的一小时内，来自该 IP 地址的任何后续请求（不仅仅是到 SSH 端口）都将被阻止。
# ignoreip – 忽略的IP地址白名单。可确保给定的IP地址，即使超过“maxretry”中指定的失败尝试次数，也不会被阻止。
```

```bash
fail2ban-client status sshd
```

```bash
iptables -L -n
```

```bash
fail2ban-client unban --all
fail2ban-client unban <ip-address>
```

参考：https://linuxiac.com/how-to-protect-ssh-with-fail2ban/
