# 0、常用脚本参数

```bash
# 在"set -e"之后出现的代码，一旦出现了返回值非零，整个脚本就会立即退出，注：set +e表示关闭，-e选项，set -e表示重新打开-e选项。
set -e 

# 表示shell脚本执行时如果遇到不存在的变量会报错并停止执行。默认不加-u参数的情况下，shell脚本遇到不存在的变量不会报错，会继续执行。
set -u 

set -o pipefail #设置了这个选项以后，包含管道命令的语句的返回值，会变成最后一个返回非零的管道命令的返回值。
```

# 1、判断curl返回状态码

```bash
#!/bin/bash
response=$(curl -sL -o /dev/null -w %{http_code} https://baidu.com)
if [[ $response -ge 200 && $response -le 299 ]] ;then 
	echo 'check point success'
else 
	echo 'check point fail'
fi
```

# 2、读取文件中的配置到变量中

```bash
#!/bin/bash
# 配置文件中的配置项格式为key1=value1，一行一个配置项
while read line;do
	eval "$line"
done < /etc/openvpn/server/smtp.conf
echo $key1
```

# 3、根据console输入条件执行

```bash
#!/bin/bash
echo "选择以下功能:"
echo "   0) 功能0"
echo "   1) 功能1"
echo "   2) 功能2"
echo "   3) 功能3"
echo "   4) 功能4"
read -p "功能选项[4]: " option
until [[ -z "$option" || "$option" =~ ^[0-4]$ ]]; do
	read -p "$option为无效的选项，请重新输入功能选项: " option
done
case "$option" in
    0) 
    	echo "功能0已执行!" 
    ;;
		1)
			echo "功能1已执行!" 
      exit
    ;;
		2)
			echo "功能2已执行!" 
			exit
		;;
		3)
			echo "功能3已执行!"  
			exit
		;;
		# 默认选项
		4|"")
			echo "功能4已执行!" 
			exit
		;;
esac
```

# 4、将指定输出内容写入文件

```bash
{
echo "hahh"
echo "lalal"
} > /tmp/test
```

# 5、判断变量是否存在或为空

```bash
if [ -z ${var+x} ]; then 
	echo "var is unset"; 
else 
	echo "var is set to '$var'"; 
fi
```

参考：https://stackoverflow.com/questions/3601515/how-to-check-if-a-variable-is-set-in-bash

```bash
#!/bin/bash

# -z 选项用于检查变量是否为空字符串，如果是，则返回true
if [ -z "${VAR}" ]; then
    echo "VAR is empty"
fi

# -n 选项用于检查变量是否为非空字符串，如果是，则返回true。
if [ -n "${VAR}" ]; then
    echo "VAR is not empty"
fi
```

在测试变量是否为空时，最好使用双引号将变量括起来，以避免在变量为空时出现语法错误。

# 6、换算秒为分钟、小时

```bash
#!/bin/bash

a=60100
swap_seconds ()
{
    SEC=$1
    (( SEC < 60 )) && echo -e "持续时间: $SEC秒\c"
    (( SEC >= 60 && SEC < 3600 )) && echo -e "持续时间: $(( SEC / 60 ))分钟$(( SEC % 60 ))秒\c"
    (( SEC > 3600 )) && echo -e "持续时间: $(( SEC / 3600 ))小时$(( (SEC % 3600) / 60 ))分钟$(( (SEC % 3600) % 60 ))秒\c"
}

b=`swap_seconds $a`
echo $b
```

输出

```bash
持续时间: 16小时41分钟40秒
```

# 7、脚本命令行参数的传递与判断

```bash
#!/bin/bash

main() {
	if [[ $# == 1 ]]; then
        case $1 in
        "-h")
            echo "脚本使用方法: "
            echo "  ./gitlab-pipeline.sh git仓库名1 git仓库名2 ... tag名(tag命名规则为: *-v加数字)"
            exit
            ;;
        "--help")
            echo "脚本使用方法: "
            echo "  ./gitlab-pipeline.sh git仓库名1 git仓库名2 ... tag名(tag命名规则为: *-v加数字)"
            exit
            ;;
        *)
            echo "参数错误！"
            exit
            ;;
        esac
    fi
}

main $*
```



# 8、检测docker 容器的启动状态


```bash
# 第一步：判断镜像是否存在
if [ `docker images --format {{.Repository}}:{{.Tag}} |grep -Fx 192.168.1.7:32772/applications/$CI_PROJECT_NAME:${CI_COMMIT_SHORT_SHA};echo $?` -eq 0 ];then
   docker pull 192.168.1.7:32772/applications/$CI_PROJECT_NAME:$CI_COMMIT_SHORT_SHA ;
fi;
# 第二步：判断是否已经有重名的容器在运行或者处在其他状态。重名的，先删掉，在启动；不重名的直接启动
if [ `docker ps -a --format {{.Names}} |grep -Fx $CI_PROJECT_NAME > /dev/null ;echo $?` -eq 0 ] ;then
   docker rm -f $CI_PROJECT_NAME ;
   docker run -d --name $CI_PROJECT_NAME -p 30088:8080 192.168.1.7:32772/applications/$CI_PROJECT_NAME:$CI_COMMIT_SHORT_SHA ;
else
   docker run -d --name $CI_PROJECT_NAME -p 30088:8080 192.168.1.7:32772/applications/$CI_PROJECT_NAME:$CI_COMMIT_SHORT_SHA ;
fi;
# 第三步：循环五次判断容器的监控检测是否处于什么状态。健康状态就直接退出循环，不健康显示健康检查日志，正在启动的直接显示。处于其他状态的直接显示状态
n=0;
while true ;do
  container_state=`docker inspect --format='{{json .State.Health.Status}}' $CI_PROJECT_NAME`;
  case $container_state in
    '"starting"' )
      echo "应用容器正在启动！";
    ;;
    '"healthy"' )
      echo "应用容器已启动，状态健康！";
    break;
    ;;
    '"unhealthy"' )
      echo "应用容器健康检测失败！";
      docker inspect --format='{{json .State.Health.Log}}' $CI_PROJECT_NAME;
    ;;
    * )
      echo "未知的状态:$container_state";
    ;;
  esac;
  sleep 1s;
  n=$(($n+1));
  if [ $n -eq 5 ];
     then break ;
  fi ;
done
```

# 9、检查常见系统命令是否安装

```bash
check_command() {
	if ! command -v ifconfig >/dev/null 2>&1; then
		echo -e "\033[31mifconfig命令不存在，正在下载安装！\033[0m"
		if os="ubuntu"; then
			apt install -y net-tools >/dev/null 2>&1
		elif os="centos"; then
			yum install -y net-tools >/dev/null 2>&1
		elif os="fedora"; then
			dnf install -y net-tools >/dev/null 2>&1
		fi
	elif ! command -v ip >/dev/null 2>&1; then
		echo -e "\033[31mip命令不存在，正在下载安装！\033[0m"
		if os="ubuntu"; then
			apt install -y iproute2 >/dev/null 2>&1
		elif os="centos"; then
			yum install -y iproute2 >/dev/null 2>&1
		elif os="fedora"; then
			dnf install -y iproute2 >/dev/null 2>&1
		fi
	elif ! command -v curl >/dev/null 2>&1; then
		echo -e "\033[31mcurl命令不存在，正在下载安装！\033[0m"
		if os="ubuntu"; then
			apt install -y curl >/dev/null 2>&1
		elif os="centos"; then
			yum install -y curl >/dev/null 2>&1
		elif os="fedora"; then
			dnf install -y curl >/dev/null 2>&1
		fi
	elif ! command -v wget >/dev/null 2>&1; then
		echo -e "\033[31mawk命令不存在，正在下载安装！\033[0m"
		if os="ubuntu"; then
			apt install -y wget >/dev/null 2>&1
		elif os="centos"; then
			yum install -y wget >/dev/null 2>&1
		elif os="fedora"; then
			dnf install -y wget >/dev/null 2>&1
		fi
	elif ! command -v tail >/dev/null 2>&1; then
		echo -e "\033[31mcoreutils命令不存在，正在下载安装！\033[0m"
		if os="ubuntu"; then
			apt install -y coreutils >/dev/null 2>&1
		elif os="centos"; then
			yum install -y coreutils >/dev/null 2>&1
		elif os="fedora"; then
			dnf install -y coreutils >/dev/null 2>&1
		fi
	elif ! command -v sed >/dev/null 2>&1; then
		echo -e "\033[31msed命令不存在，正在下载安装！\033[0m"
		if os="ubuntu"; then
			apt install -y sed >/dev/null 2>&1
		elif os="centos"; then
			yum install -y sed >/dev/null 2>&1
		elif os="fedora"; then
			dnf install -y sed >/dev/null 2>&1
		fi
	elif ! command -v grep >/dev/null 2>&1; then
		echo -e "\033[31mgrep命令不存在，正在下载安装！\033[0m"
		if os="ubuntu"; then
			apt install -y grep >/dev/null 2>&1
		elif os="centos"; then
			yum install -y grep >/dev/null 2>&1
		elif os="fedora"; then
			dnf install -y grep >/dev/null 2>&1
		fi
	fi
}
```

# 10、检查系统网络

```bash
check_network() {
    ping -c 4 114.114.114.114 >/dev/null
    if [ ! $? -eq 0 ]; then
        echo -e "\033[31mIP地址无法ping通，请检查网络连接！！！\033[0m"
        exit
    fi
    ping -c 4 www.baidu.com >/dev/null
    if [ ! $? -eq 0 ]; then
        echo -e "\033[31m域名无法Ping通，请检查DNS配置！！！\033[0m"
        exit
    fi
    curl -s --retry 2 --connect-timeout 2 www.baidu.com >/dev/null
    if [ ! $? -eq 0 ]; then
        echo -e "\033[31m域名无法Ping通，请检查DNS配置！！！\033[0m"
        exit
    fi
}
```

# 11、发送钉钉通知

```bash
function sendDingDing(){
  Ding_Webhook_Token='钉钉机器人的WebHook Token'
  Ding_Webhook_keyword="可用作消息标题"
  send_result=$(curl -sS \
    -XPOST https://oapi.dingtalk.com/robot/send?access_token="$Ding_Webhook_Token" \
    -H 'Content-Type: application/json' \
    -d '{"msgtype": "markdown","markdown": {"title": "'$Ding_Webhook_Keyword'","text": "'"$*"'"},"at": {"isAtAll": true}}')
  if [ $(echo $send_result | jq -r '.errcode') -eq 0 ]; then
    echo "发送成功:  \n" $*" "
  else
    echo "发送失败:  \n" $*"\n"
    echo $send_result
  fi
}

sendDingDing  "$(date +"%F %H:%M:%S") 需要告警的消息"
```

`.zshrc`

```bash
function sendDingDing() {
  setopt no_nomatch
  Ding_Webhook_Token='钉钉机器人的WebHook Token'
  Ding_Webhook_Keyword='可用作消息标题'
  send_result=$(curl -sS \
    -XPOST https://oapi.dingtalk.com/robot/send?access_token="$Ding_Webhook_Token" \
    -H 'Content-Type: application/json' \
    -d '{"msgtype": "markdown","markdown": {"title": "'$Ding_Webhook_Keyword'","text": "'"$*"'"},"at": {"isAtAll": true}}')
  if [ $(echo $send_result | jq -r '.errcode') -eq 0 ]; then
    echo "发送成功:  \n" $*" "
  else
    echo "发送失败:  \n" $*"\n"
    echo $send_result
  fi
}
function kfdd() {
  sendDingDing "$(date +"%F %H:%M:%S") "$*" "
}
```

# 12、清理保持备份个数

清理指定目录北备份文件，并保持指定个数。

```bash
function clean_backups (){
  if [ $# -gt 2 ];then
    echo "参数错误"
    exit 1
  else
    BAK_HOME=$1
    keepNum=$2
    fileNum=$(find ${BAK_HOME} -maxdepth 1 -type d -ctime -1 ! -path $BAK_HOME |wc -l)
    if [ $fileNum -gt $keepNum ] ;then
      find ${BAK_HOME} -maxdepth 1 -type d -ctime -1 ! -path $BAK_HOME | head -n `expr $fileNum - $keepNum` | xargs rm -rf
    fi
  fi
}
# 调用
clean_backups "./backups" 2 
```