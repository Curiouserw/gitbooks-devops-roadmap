# Asuswrt-Merlin

# 一、简介

官网：https://www.asuswrt-merlin.net/

文档：https://github.com/RMerl/asuswrt-merlin.ng/wiki

2.5Ghz网卡：eth6

5Ghz 网卡：eth7

# 二、系统服务

## 1、dnsmasq

> cat /etc/dnsmasq.conf

```bash
```

> /jffs/configs/dnsmasq.conf.add

```bash
dhcp-option=tag:sideproxy,option:router,192.168.1.99
dhcp-host=A6:BE:BF:8C:AC:3D,set:sideproxy
log-facility=/var/log/dnsmasq.log
enable-tftp
log-queries
log-dhcp
tftp-root=/tmp/mnt/sd/tftp
dhcp-boot=pxelinux.0
```

> /var/lib/misc/dnsmasq.leases

```bash
# 检测配置文件语法
dnsmasq -C /jffs/configs/dnsmasq.conf.add --test
# 重启dnsmasq
service restart_dnsmasq && tail -f /var/log/dnsmasq.log |grep dhcp
```

```bash
echo "address=/test.top/127.0.0.1" >> /jffs/configs/dnsmasq.conf.add
service restart_dnsmasq
```

## 2、sshd-dropbear

```bash
-r keyfile      Specify hostkeys (repeatable)
		defaults:
		- rsa /etc/dropbear/dropbear_rsa_host_key
		- ecdsa /etc/dropbear/dropbear_ecdsa_host_key
		- ed25519 /etc/dropbear/dropbear_ed25519_host_key
		
		
/var/run/dropbear.pid

/usr/sbin/dropbear

nvram get sshd_authkeys
 
/root/.ssh/authorized_keys
```

## 3、iptables

加载comment模块

```bash
OS=$(uname -r)
insmod /lib/modules/${OS}/kernel/net/netfilter/xt_comment.ko

lsmod | grep -i comment
```

加载TPROXY 模块

```bash
OS=$(uname -r)
ls /lib/modules/${OS}/kernel/net/netfilter/*TPROXY*
modprobe xt_TPROXY
lsmod | grep -i TPROXY
```

## 4、自带DDNS 和 Let's Encrypt

```bash
nvram set ddns_enable_x="$(nvram get ddns_enable_x_bak)"
nvram set ddns_ipcheck="$(nvram get ddns_ipcheck_bak)"
nvram set ddns_server_x="$(nvram get ddns_server_x_bak)"
nvram set ddns_hostname_x="$(nvram get ddns_hostname_x_bak)"
nvram set le_enable="$(nvram get le_enable_bak)"
nvram set asus_ddns_service="Aliddns"
ddns_custom_updated 1
ddns_updated 
service restart_ddns_le 
service restart_ddns
```



## 5、格式化U盘

### ext4参数优化+4K对齐

最好在linux下操作完成后再插入路由器

```bash
# 彻底擦除磁盘开头（清除MBR和GPT签名）
dd if=/dev/zero of=/dev/sda bs=1M count=10 status=progress conv=fsync

parted /dev/sda --script \
  mklabel gpt \
  mkpart primary ext4 1MiB 100%

mkfs.ext4 \
  -O ^has_journal \
  -L sd \
  -E lazy_itable_init=0,lazy_journal_init=0 \
  -m 0 \
  /dev/sda1
```

插回路由器后验证是否4K对齐

```bash
blockdev --getss /dev/sda   # 逻辑扇区（一般 512,Start × 512 能被 4096 整除 → OK）
blockdev --getpbsz /dev/sda # 物理块（通常 4096）
cat /sys/block/sda/sda1/start 
cat /sys/block/sda/queue/physical_block_size
cat /sys/block/sda/queue/logical_block_size
```

## 6、其他

```bash
mke2fs -t ext4 -O ^has_journal /dev/sda
# 禁用日志：^has_journal
```

- amtm命令格式化U 盘

  U 盘标签跟之前保持一致，格式ext4

  ```bash
  umount /tmp/mnt/sd && \
  mke2fs -t ext4 -O ^has_journal /dev/sda && \
  reboot now
  ```

- koolshare的USB2JFSS插件挂载JFSS分区到U盘

- amtm命令安装entware到U盘

- opkg install常用软件

  ```bash
  opkg install jq vim procps-ng-sysctl procps-ng-top procps-ng-watch procs bash nginx-ssl
  ```

- 插件根目录
- 插件根目录下的服务状态管理脚本：`stats.js/config.js`  --------软链---------> `/jffs/.koolshare/scripts/`
- 插件根目录下的icon文件：`icon-插件名.png`  --------软链---------> `/jffs/.koolshare/res`
- 插件根目录下的服务自启动脚本 ：`S99插件.sh` --------软链---------> `/jffs/.koolshare/init.d/`

# 三、系统 API

## 1、登录接口

- **URL：**`/login.cgi`
- **POST请求**
- **Header**
  - `Content-Type`: `application/x-www-form-urlencoded`
  - `Referer`: `https://192.168.1.1:8443/Main_Login.asp`
- **Body**
  - **form-urlencoded**
    - **login_authorization** : `"用户名:密码"格式的Base64编码`
- **响应**：需要保存 Cookie。后续其他请求都需要该 Cookie

```bash
routerhost="https://192.168.1.1:8443"
username="登录用户名"
password="登录用户密码"
userpwd_base64=$(echo "$username:$password" | base64)
curl -lk "$routerhost/login.cgi" \
  -c cookies.txt
  -H "Referer: $routerhost/Main_Login.asp" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "login_authorization=$userpwd_base64"
```

## 2、获取 dbus 接口

- **URL**：`/_api/dbus的key前缀` 
- **GET请求**
- **Cookie**
  - 需要登录接口的 **Cookie asus_s_token**

```bash
routerhost="https://192.168.1.1:8443"
curl -kl -XGET "$routerhost/_api/soft" \
	-b cookies.txt 
```

## 3、获取日志接口

- **URL**：`/_temp/日志文件名(一般是指在/tmp/upload/路径下的文件)`
- **GET请求**
- **Cookie**
  - 需要登录接口的 **Cookie asus_s_token**

```bash
routerhost="https://192.168.1.1:8443"
curl -kl -XGET "$routerhost/_temp/cronnotifytask.log"
```

## 4、执行脚本

- **URL：**`/_api/`

- **POST请求**

- **Header**

  - `Content-Type`: `application/json`

- **Body**

  - **JSON**

    > {
    >
    > ​    "id": **需要8位以上的uuid**,
    >
    > ​    "method": "脚本名，一般是指在**/koolshare/scripts/**下面的脚本",
    >
    > ​    "params": [ "传递给脚本参数，会作为脚本的第3位形参"  ]
    >
    > }

- **Cookie**

  - 需要登录接口的 **Cookie asus_s_token**

```bash
routerhost="https://192.168.1.1:8443"
ID=$(printf "%08d\n" $(( $(od -An -N4 -tu4 /dev/urandom) % 100000000 )))
curl -lk -X POST "$routerhost/_api/" \
  -b cookies.txt \
  --data-raw "{\"id\": $ID, \"method\": \"tes.sh\",\"params\": [1]}"
```

## 5、上传文件

```bash
routerhost="https://192.168.1.1:8443"
curl -lk -X POST "$routerhost/_upload" \
    -b cookies.txt \
    -H 'Connection: keep-alive' \
    -H 'Content-Type: multipart/form-data; boundary=--------------------------646083769051819578072890' \
    --form 'test.csv=@"/Users/test/test.csv"'
```

# 三、自启脚本与定时任务

## 1、开机自启脚本

https://github.com/RMerl/asuswrt-merlin.ng/wiki/User-scripts

- **`/jffs/scripts/services-start：`**系统服务启动后，间接通过`/koolshare/bin/ks-services-start.sh`调用`/koolshare/init.d/`路径下**V**开头的脚本
- **`/jffs/scripts/services-stop：`**重启时系统服务关闭后，间接通过`/koolshare/bin/ks-services-stop.sh`调用`/koolshare/init.d/`路径下**T**开头的脚本。(间接通过`/opt/etc/init.d/rc.unslung`调用`/opt/etc/init.d/路径下T开头的脚本`,将stop参数传递过去)
- **`/jffs/scripts/wan-start：`**WAN网卡启动之后，间接通过`/koolshare/bin/ks-wan-start.sh`调用`/koolshare/init.d/`路径下**S**开头的脚本，将start参数传递过去。但此时可能拨号仍未成功，网络会不太稳定
- **`/jffs/scripts/nat-start：`**：在 NAT 规则（即端口转发等）应用于 NAT 表后调用，间接通过`/koolshare/bin/ks-nat-start.sh`脚本调用`/koolshare/init.d/`路径下**N**开头的脚本，将start_nat参数传递过去。可在此处放置自己的 NAT 表自定义规则
- **`/jffs/scripts/post-mount：`**  在磁盘分区挂载后，间接通过`/koolshare/bin/ks-mount-start.sh`调用`/koolshare/init.d/`路径下**M**开头的脚本，将`start`和`$1`参数传递过去。`$1`通常为挂载点路径，例如`/tmp/mnt/sd`
- **`/jffs/scripts/unmount：`**在磁盘分区卸载后，间接通过`/koolshare/bin/ks-unmount.sh`调用`/koolshare/init.d/`路径下**U**开头的脚本，将`$1`参数传递过去。`$1`通常为挂载点路径，例如`/tmp/mnt/sd`

```bash
开机自启脚本目录

-- /jffs/scripts/
   ｜---- post-mount         # 开机早期，每个分区挂载完成后执行，接受挂载点作为参数 $1
            |————> /koolshare/bin/ks-mount-start.sh start $1
                |————> find /koolshare/init.d/ -name 'M*' | sort -n
   ｜---- service-event      # 插件统一启动（services）脚本
            |————> /koolshare/bin/ks-services-start.sh
                |————> find /koolshare/init.d/ -name 'V*' | sort -n
   ｜---- wan-start	        # 当 WAN（互联网连接）建立成功后，可设置 DDNS、VPN、自动任务等
            |————> /koolshare/bin/ks-wan-start.sh start
                |————> find /koolshare/init.d/ -name 'S*' | sort -r 
   ｜---- nat-start          # 可设置 NAT（iptables）、端口转发、访问控制等
            |————> /koolshare/bin/ks-nat-start.sh start_nat
                |————> find /koolshare/init.d/ -name 'N*' | sort -n
   ｜---- services-stop      # 插件统一暂停（services）脚本
            |————> /koolshare/bin/ks-services-stop.sh
                |————> find /koolshare/init.d/ -name 'T*' | sort -rn
   ｜---- unmount            # 设备拔除/卸载时触发的脚本
            |————> /koolshare/bin/ks-unmount.sh
                |————> find /koolshare/init.d/ -name 'U*' | sort -n
```

## 2、Cru定时任务

```bash
# 查看定时任务
cru l
# 添加定时任务
cru a 任务名 '0 22 * * 5 sh /tmp/mnt/sd/backup.sh'
```

# 四、NVRAM配置管理

**NVRAM （Non-Volatile RAM）**是一种非易失性存储区域，用于保存**固件和系统核心配置**，即使断电也不会丢失。包括网络设置、无线配置、VPN、用户脚本开关等。

固化在 Flash 存储中（**ROM 区**），有空间限制（通常为 **64KB**，约 65,536 字节），不同型号可能略有差异。

修改后需 `nvram commit` 才会写入 Flash，否则重启后丢失。

```bash
nvram show                     # 显示所有变量（注意输出很多）
nvram get <key>               # 获取某个变量值
nvram set <key>=<value>       # 设置变量
nvram unset <key>             # 删除变量
nvram commit                  # 保存到闪存（否则只是临时修改）
```

**常用nvram配置**

```bash
nvram get wan0_ipaddr / wan0_realip_ip								#		查看WAN网接口的外网IP地址
nvram get custom_clientlist														# 	查看静态DHCP绑定（IP-MAC-名称）列表。以>分隔的多组IP、MAC、主机名三元组
nvram get dhcp_staticlist															# 	与 custom_clientlist 类似，用于 DHCP 静态绑定
nvram get lan_ipaddr																	# 	查看路由器 LAN IP（默认：192.168.1.1）
nvram get wan0_proto																	# 	查看WAN 连接类型（如：dhcp, pppoe）
nvram get wan_pppoe_username / wan_pppoe_passwd				# 	查看PPPoE用户名/密码
nvram get wl0_ssid / wl1_ssid													# 	查看无线 SSID（wl0_ssid: 2.4G / wl1_ssid: 5G）
nvram get jffs2_scripts																# 	查看是否启用 jffs 用户脚本（1 为启用）
nvram get usb_enable																	# 	查看是否启用 USB 功能（1 为启用）
nvram get ntp_server0/ntp_server1											#   查看NTP时间服务器地址
nvram get router_name																	# 	查看主机名称
nvram get odmpid																			# 	查看当前设备的具体型号
nvram get extendno																		# 	固件构建扩展编号
nvram get firmver																			# 	固件版本号
nvram get buildno																			# 	构建版本号
nvram get asus_mfg																		# 	出厂标识（通常为 1）
nvram get sshd_authkeys

nvram savefile 
# 所有的 ENV 会保存在/data/nvramdefault.txt
```

> 注意：可以先`nvram savefile` 将配置保存到文件，然后grep搜索相关配置。例如：`grep "ppoe" /data/nvramdefault.txt`查找PPOE相关配置

# 五 、DBUS键值数据库

## 1、DBUS简介

Asuswrt-Merlin 中的 `DBUS` 实际上是基于 `skipd` 的轻量键值数据库（KV Store），用来支持运行时插件通信和参数管理。

它**不是 Linux D-Bus**，与桌面系统中的 `org.freedesktop.DBus` 完全无关。

- 持久化数据（如插件设置）：`/jffs/ksdb/`（如使用了 jffs 持久化机制）

**常见用途：**

| 场景           | 示例键                                      |
| -------------- | ------------------------------------------- |
| 插件开关       | `softcenter_module_skynet_enable=1`         |
| 状态标志       | `skynet_running=1`                          |
| 自定义脚本参数 | `myscript_interval=3600`                    |
| 插件版本管理   | `softcenter_module_diversion_version=5.2.3` |

## 2、命令用法

```bash
dbus get <key>                     # 获取变量
dbus set <key>=<value>             # 设置变量
dbus remove <key>                  # 删除变量
dbus show                          # 列出所有键值对
dbus list <prefix>                 # 列出指定前缀的键（推荐）
```

## 3、**DBus数据备份与恢复**

建议进行逻辑备份。物理文件备份的话，文件中有太多的无用数据。

`````bash
dbus listall > /mnt/sd/backup/dbus-backup
`````

```bash
killall skipd ; rm -f /jffs/ksdb/log && skipd -d /jffs/ksdb/ &
while read line; do
  key=$(echo "$line" | cut -d= -f1)
  value=$(echo "$line" | cut -d= -f2-)
  dbus set "$key=$value"
done < /mnt/sd/backup/dbus-backup
```



# 六、UI插件中心KoolShare

## 1、插件DBus设置

```txt
dbus set softcenter_module_插件名_description="插件描述"
dbus set softcenter_module_插件名_install="1"
dbus set softcenter_module_插件名_name="插件名"
dbus set softcenter_module_插件名_title="插件标签页名"
dbus set softcenter_module_插件名_version="1.0"
```

设置以上`DBus key`之后，`KoolShare` 软件中心的前端首页就会显示已安装该插件。但是会提示缺少图标。按照第2步上传好图标，点击图标就会跳转到第 3 步的前端静态页面了

## 2、插件静态资源文件

```bash
# 前端页面图标
/www/res/icon-插件名.png
```

## 3、软件中心显示前端页面

```bash
/koolshare/webs/Module_插件名.asp
```

```asp
<!DOCTYPE html
	PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<html xmlns:v>
<head>
	<meta http-equiv="X-UA-Compatible" content="IE=Edge" />
	<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
	<meta HTTP-EQUIV="Pragma" CONTENT="no-cache">
	<meta HTTP-EQUIV="Expires" CONTENT="-1">
	<link rel="shortcut icon" href="images/favicon.png">
	<link rel="icon" href="images/favicon.png">
	<title>Testplugin</title>
	<link rel="stylesheet" type="text/css" href="index_style.css" />
	<link rel="stylesheet" type="text/css" href="form_style.css" />
	<link rel="stylesheet" type="text/css" href="usp_style.css" />
	<link rel="stylesheet" type="text/css" href="css/element.css">
	<link rel="stylesheet" type="text/css" href="res/softcenter.css">
	<script type="text/javascript" src="/state.js"></script>
	<script type="text/javascript" src="/popup.js"></script>
	<script type="text/javascript" src="/help.js"></script>
	<script type="text/javascript" src="/js/jquery.js"></script>
	<script type="text/javascript" src="/general.js"></script>
	<script type="text/javascript" language="JavaScript" src="/js/table/table.js"></script>
	<script type="text/javascript" language="JavaScript" src="/client_function.js"></script>
	<script type="text/javascript" src="/res/softcenter.js"></script>
	<style>
		.show-btn1:hover,
		.show-btn2:hover,
		.active {
			border: 1px solid #2f3a3e;
			background: #2f3a3e;
		}
		.ks_btn {
			border: 1px solid #222;
			font-size: 10pt;
			color: #fff;
			padding: 5px 5px 5px 5px;
			border-radius: 5px 5px 5px 5px;
			width: 14%;
			background: linear-gradient(to bottom, #003333 0%, #000000 100%);
		}
		.ks_btn:hover,
		{
      border: 1px solid #222;
      font-size: 10pt;
      color: #fff;
      padding: 5px 5px 5px 5px;
      border-radius: 5px 5px 5px 5px;
      width: 14%;
      background: linear-gradient(to bottom, #27c9c9 0%, #279fd9 100%);
		}
	
	</style>
	<script>
		var dbus = {};
		var _responseLen;
		var noChange = 0;
		var x = 5;
		var params_inp = ['testplugin_pid'];
		var params_chk = ['testplugin_enable'];
		
		function init() {
			show_menu(menu_hook);
			get_dbus_data();
		}

		function conf2obj() {
			for (var i = 0; i < params_inp.length; i++) {
				E(params_inp[i]).value = dbus[params_inp[i]];
			}
			for (var i = 0; i < params_chk.length; i++) {
				if (dbus[params_chk[i]]) {
					E(params_chk[i]).checked = dbus[params_chk[i]] == "1";
				}
			}
		}

		function get_dbus_data() {
			$.ajax({
				type: "GET",
				url: "/_api/testplugin",
				dataType: "json",
				cache: false,
				async: false,
				success: function (data) {
					dbus = data.result[0];
					conf2obj();
					E("testplugin_pid").innerHTML = dbus["testplugin_pid"];
				},
				error: function (XmlHttpRequest, textStatus, errorThrown) {
					console.log(XmlHttpRequest.responseText);
					alert("skipd数据读取错误，请用在chrome浏览器中按F12键后，在console页面获取错误信息！");
				}
			});
		}
		function menu_hook() {
			tabtitle[tabtitle.length - 1] = new Array("", "testplugin");
			tablink[tablink.length - 1] = new Array("", "Module_testplugin.asp");
		}
		function reload_Soft_Center() {
			location.href = "/Module_Softcenter.asp";
		}
	</script>
</head>

<body onload="init();">
  <div id="TopBanner"></div>
	<div id="Loading" class="popup_bg"></div>
	<table class="content" align="center" cellpadding="0" cellspacing="0">
		<tr>
			<td width="17">&nbsp;</td>
			<td valign="top" width="202">
				<div id="mainMenu"></div>
				<div id="subMenu"></div>
			</td>
			<td valign="top">
				<div id="tabMenu" class="submenuBlock"></div>
				<table width="98%" border="0" align="left" cellpadding="0" cellspacing="0" style="display: block;">
					<tr>
						<td align="left" valign="top">
							<div>
								<table width="760px" border="0" cellpadding="5" cellspacing="0" bordercolor="#6b8fa3"
									class="FormTitle" id="FormTitle">
									<tr>
										<td bgcolor="#4D595D" colspan="3" valign="top">
											<div>&nbsp;</div>
											<div style="float:left;" class="formfonttitle" style="padding-top: 12px">
												Testplugin</div>
											<div style="float:right; width:15px; height:25px;margin-top:10px"><img
													id="return_btn" onclick="reload_Soft_Center();" align="right"
													style="cursor:pointer;position:absolute;margin-left:-30px;margin-top:-25px;"
													title="返回软件中心" src="/images/backprev.png"
													onMouseOver="this.src='/images/backprevclick.png'"
													onMouseOut="this.src='/images/backprev.png'"></img></div>
											<div style="margin:30px 0 10px 5px;" class="splitLine"></div>
											<div style="margin:5px 0px 0px 0px;">
												<table width="100%" border="1" align="center" cellpadding="4"
													cellspacing="0" bordercolor="#6b8fa3" class="FormTable">
													<tr id="switch_tr">
														<th>
															<label>开启Barknotify</label>
														</th>
														<td colspan="2">
															<div class="switch_field" style="display:table-cell">
																<label for="testplugin_enable">
																	<input id="testplugin_enable" class="switch"
																		type="checkbox" style="display: none;">
																	<div class="switch_container">
																		<div class="switch_bar"></div>
																		<div class="switch_circle transition_style">
																			<div></div>
																		</div>
																	</div>
																</label>
															</div>
														</td>
													</tr>
													<tr id="testplugin_pid_tr">
														<th>进程ID</th>
														<td>
															<span id="testplugin_pid"></span>
														</td>
													</tr>
												</table>
											</div>
										</td>
									</tr>
								</table>
							</div>
						</td>
					</tr>
				</table>
			</td>
			<td width="10" align="center" valign="top"></td>
		</tr>
	</table>
	<div id="footer"></div>
</body>
</html>
```

## 4、供前端页面调用的脚本

该脚本会被前端页面的API接口调用。一般请求时会带有形参，`$1`一般为请求ID ，`$2`一般为功能标识位，`$3`一般为功能参数。

例如登录后的用户，请求ID为10000，请求了该脚本，`$2`为3，3代表获取脚本的功能标识，`$3`为100，表示获取脚本日志最后100行。

所以实现脚本时，判断`$2`为3时，就根据`$3`获取最新100行日志文件，然后将`$1`传递给`http_response`函数进行结果返回

```bash
/koolshare/scripts/插件名.sh
```

```bash
#!/opt/bin/bash
source /koolshare/scripts/base.sh
plugin_name="插件名"
start(){ }
stop(){ }
# $1 的值为请求ID，$2的值为功能标志 ，$3 是WEBUI传入的 JSON 字符串
case "$2" in
1)
    # 用于WEBUI来启动服务
    http_response "$1"
    stop ;
    start ;
    ;;
2)
    # 用于WEBUI来关闭服务
    http_response "$1"
    stop
    ;;
3)  
		# 用于WEBUI获取插件进程PID
    json_str=$3
    pluginname=$(echo "$json_str" | jq -r '.pluginname')
    # 校验必填项
    if [[ "$pluginname" == "null" && "$pluginname" != "$plugin_name" ]]; then
        http_response "$1\", \"success\": \"false\", \"error\": \"缺少字段"
        exit 1
    fi
    pid=$(pidof $plugin_name 2> /dev/null )
    if [ $pid ];then
        http_response "$1\", \"pid\": $pid,  \"success\": \"true" ;
    else
        http_response "$1\", \"pid\": 0, \"success\": \"false" ;
    fi
    ;;
*)
    http_response '{"result": "unknown", "data": [], "success": "false"}'
    ;;
esac
```

## 5、插件开机自启的脚本

该脚本会被系统脚本带形参的方式调用。例如带`start`作为`$1`调用

```bash
/koolshare/init.d/S99插件名.sh
```



# 七、终端插件中心AMTM



# 八、应用脚本

## 1、获取硬件温度

https://www.snbforums.com/threads/how-to-read-the-temperature-without-merlin.80167/

https://www.snbforums.com/threads/script-to-monitor-temperatures-on-rt-ac68u.68775/

```bash
# ARMv7 CPU
cat /proc/dmu/temperature
# ARMv8 CPU
cat /sys/class/thermal/thermal_zone0/temp | awk '{print $1 / 1000}'
# 2.4GHz
wl -i `nvram get wl0_ifname` phy_tempsense | awk '{print $1 / 2 + 20}'
# 5GHz
wl -i `nvram get wl1_ifname` phy_tempsense | awk '{print $1 / 2 + 20}'
```

获取硬件温度输出Prometheus Metrics格式

```bash
#!/bin/bash
PROM_STAT_FILE=/tmp/muprome/temp.prom
# 获取 CPU 温度，设置异常处理
if CPUS_temp=$(cat /sys/class/thermal/thermal_zone0/temp 2>/dev/null); then
    CPUS_temp_celsius=$(echo "scale=1; $CPUS_temp / 1000" | bc)
else
    CPUS_temp_celsius=0
fi
# 获取 2.5GHz WiFi 温度，设置异常处理
if NET25_temp=$(wl -i eth6 phy_tempsense 2>/dev/null | awk '{print $1 / 2 + 20}'); then
    :
else
    NET25_temp=0
fi
# 获取 5GHz WiFi 温度，设置异常处理
if NET5S_temp=$(wl -i eth7 phy_tempsense 2>/dev/null | awk '{print $1 / 2 + 20}'); then
    :
else
    NET5S_temp=0
fi
{
  # 输出设备名称和温度
  echo "node_hwmon_chip_names{chip=\"CPU\", chip_name=\"CPU\"} 1"
  echo "node_hwmon_temp_celsius{chip=\"CPU\", chip_name=\"CPU\"} $CPUS_temp_celsius"

  echo "node_hwmon_chip_names{chip=\"2.5GHz WiFi\", chip_name=\"2.5GHz WiFi\"} 1"
  echo "node_hwmon_temp_celsius{chip=\"2.5GHz WiFi\", chip_name=\"2.5GHz WiFi\"} $NET25_temp"

  echo "node_hwmon_chip_names{chip=\"5GHz WiFi\", chip_name=\"5GHz WiFi\"} 1"
  echo "node_hwmon_temp_celsius{chip=\"5GHz WiFi\", chip_name=\"5GHz WiFi\"} $NET5S_temp"
} > "$PROM_STAT_FILE"
```



# 附录

## 1、OpenVPN 相关信息

/usr/sbin/openvpn --version

>OpenVPN 2.6.5 arm-buildroot-linux-gnueabi [SSL (OpenSSL)] [LZO] [LZ4] [EPOLL] [MH/PKTINFO] [AEAD]
>library versions: OpenSSL 1.1.1u  30 May 2023, LZO 2.10

```
/tmp/etc/openvpn/server1
├── ca.crt
├── ca.key
├── ccd
├── client.ovpn
├── config.ovpn
├── dh.pem
├── fw_nat.sh
├── fw.sh
├── ovpn-down -> /sbin/rc
├── ovpn-up -> /sbin/rc
├── server.crt
├── server.key
├── status
└── vpn-watchdog1.sh
```

## 2、获取设备信息的脚本

```bash
#!/bin/sh
# Check if the comm command is installed
if ! command -v comm &> /dev/null; then
  echo "The comm command is not installed.  Install Now"
  opkg install coreutils-comm
fi

# Get the current time
now=$(date +"%Y-%m-%d %H:%M:%S")

# Get the list of connected devices
devices=$(brctl showmacs br0 | awk '{print $2}')

# Check if the list of devices has changed
if [ ! -f /tmp/connected_devices ]; then
  # This is the first time the script is running, so save the list of devices to a file
  echo "$devices" > /tmp/connected_devices
else
  # Get the list of devices that were connected previously
  previous_devices=$(cat /tmp/connected_devices)

  # Compare the list of devices that are connected now to the list of devices that were connected previously
  new_devices=$(comm -23 <(echo "$devices") <(echo "$previous_devices"))

  # Send a notification to DingTalk for each new device
  for device in $new_devices; do
    curl -X POST https://oapi.dingtalk.com/robot/send \
      -H "Content-Type: application/json" \
      -d "{\"msgtype\": \"text\", \"text\": {\"content\": \"New device connected to bridge br0: $device ($now)\"}}"
  done

  # Update the list of connected devices
  echo "$devices" > /tmp/connected_devices
fi
```



# 参考：

- https://github.com/decoderman/amtm/tree/master
