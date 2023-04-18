# ElasticSearch索引的快照、清理策略

# 一、简介

当使用ES存储接入的应用日志时，日志索引会日益增多。而真实的日志查询需求一般是要求半月可查，存储半年到一年。应用每天产生的日志普遍会存到对应ES中以当天日期命名的索引中。查询时根据需求，最多查询半月对应索引里的数据。至于超过半月以上的日志索引数据，可快照文件，存储到文件系统中。在有特殊场景需求时进行快照恢复进行查询。这样减小ES的索引压力，提高查询效率。

**需求**

- 将半月以上的日志索引快照成文件，存储到快照仓库中
- 删除已快照的日志索引
- 定时检测创建超过15天的索引并快照、清理
- 钉钉通知快照后删除的索引名称
- 脚本执行错误时告警

# 二、基于API的Shell脚本

## 1、Shell脚本

- 通过环境变量设置参数
- 脚本执行完成后发送钉钉通知，显示脚本涉及到的ES索引
- 集成Sentry告警，每当脚本执行出错时将时间发送至Sentry，再由Sentry进行邮件告警
- 可使用Linux cron工具或K8S上的cornjob定时每天早上1点执行该脚本

```bash
#!/bin/bash
export SENTRY_DSN=http://*****@sentry.okd.curiouser.com/28
eval "$(sentry-cli bash-hook)"

elasticsearch_host=${ELASTICSEARCH_HOST:192.168.1.2}
elasticsearch_username=${ELASTICSEARCH_USERNAME:cronjob}
elasticsearch_password=${ELASTICSEARCH_PASSWORD:******}
elasticsearch_index_expiry_day=${ELASTICSEARCH_INDEX_EXPIRY_DAY:15}
elasticsearch_exclude_index=${ELASTICSEARCH_EXCLUDE_INDEX:.*}
elasticsearch_snapshots_repository=${ELASTICSEARCH_SNAPSHOTS_REPOSITORY:***}

elasticsearch_index_expiry_sec=$((elasticsearch_index_expiry_day*86400))
elasticsearch_url="http://${elasticsearch_host}:9200"
allIndex=`curl -s -u ${elasticsearch_username}:${elasticsearch_password} -XGET "${elasticsearch_url}/_cat/indices/_all?h=index"`
excludeIndex=`curl -s -u ${elasticsearch_username}:${elasticsearch_password} -XGET "${elasticsearch_url}/_cat/indices/${elasticsearch_exclude_index}/?h=i"`
indices=`echo -e "$allIndex\n""$excludeIndex" |sort -n |uniq -u`

for i in $indices ;
do
  createdateincludemesc=`curl -s -u ${elasticsearch_username}:${elasticsearch_password} -XGET "${elasticsearch_url}/_cat/indices/$i?h=cd"` ;
  createdate=$((createdateincludemesc/1000))
  currentdate=`date +%s`
  durationtime=$((currentdate-createdate)) ;
  if [ $durationtime -gt $elasticsearch_index_expiry_sec ] ;then
     snapshotsIndices=$i"\n"${snapshotsIndices}
  fi
done

for i in `echo -e $snapshotsIndices` ;
do
  if [ `curl -o /dev/null -w "%{http_code}\n" -s -u ${elasticsearch_username}:${elasticsearch_password} -XPUT "${elasticsearch_url}/_snapshot/${elasticsearch_snapshots_repository}/%3C$i-%7Bnow%2Fd%7D%3E?wait_for_completion=true" -H 'Content-Type: application/json' -d'{"indices": "'$i'","ignore_unavailable": true,"include_global_state": false}'` = 200 ] ;then
    if [ `curl -o /dev/null -w "%{http_code}\n" -s -u ${elasticsearch_username}:${elasticsearch_password} -XDELETE "${elasticsearch_url}/$i"` = 200 ] ;then
      echo -e "The Index $i \t have been snapshoted to repository and deleted !" ;
    else
      echo "$i failed to delete " ;
    fi
  else
    echo "$i failed to snapshot " ;
  fi
done

curl -s -o /dev/null 'https://oapi.dingtalk.com/robot/send?access_token=*****' \
  -H 'Content-Type: application/json' \
  -d '{"msgtype": "text",
       "text": {"content": "已成功将以下'"$elasticsearch_index_expiry_day"'天之前的索引进行了快照：\n'"$snapshotsIndices"'"}
  }'
```

## 2、脚本部署执行

### ①Linux的Cronjob

```bash
echo "0 1 * * * /opt/es-index-snapshots.sh" > /etc/crontab
```

### ②Kubernetes的cronjob

#### 1、构建Cronjob镜像

Dockerfile

```bash
FROM centos:7
RUN curl -sL https://sentry.io/get-cli/ | bash
ADD ./es-index-snapshots.sh /usr/sbin/es-index-snapshots.sh
Entrypoint ["/bin/sh","-c"]
CMD ["/usr/sbin/es-index-snapshots.sh"]
```

```bash
docker build -t es-index-snapshots:v1 .
```

#### 2、k8s资源声明文件

es-index-snapshots-cronjob.yml

```bash
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: es-index-snapshots-cronjob
  namespace: logging
spec:
  concurrencyPolicy: Allow
  failedJobsHistoryLimit: 1
  schedule: 0 1 * * *
  startingDeadlineSeconds: 200
  successfulJobsHistoryLimit: 3
  suspend: false
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - env:
            - name: TZ
              value: Asia/Shanghai
            - name: ELASTICSEARCH_HOST
              value: *.logging.svc
            - name: ELASTICSEARCH_USERNAME
              value: cronjob
            - name: ELASTICSEARCH_PASSWORD
              value: "***""
            - name: ELASTICSEARCH_INDEX_EXPIRY_DAY
              value: "15"
            - name: ELASTICSEARCH_EXCLUDE_INDEX
              value: .*
            - name: ELASTICSEARCH_SNAPSHOTS_REPOSITORY
              value: "***"
            image: es-index-snapshots:v1
            imagePullPolicy: Always
            name: es-index-snapshots-cronjob
            resources:
              limits:
                cpu: 600m
                memory: 800Mi
              requests:
                cpu: 300m
                memory: 500Mi
            terminationMessagePath: /dev/termination-log
            terminationMessagePolicy: File
          dnsPolicy: ClusterFirst
          imagePullSecrets:
          - name: harbor-secrets
          restartPolicy: OnFailure
          schedulerName: default-scheduler
          securityContext: {}
          terminationGracePeriodSeconds: 30

```

```bash
kubectl apply -f es-index-snapshots-cronjob.yml
```

# 三、基于 Python版本SDK的脚本

## 1、Python脚本

- 脚本依据索引的创建时间进行处理的。例如设置快照删除15天以前的索引，判断计算的期限是以索引的创建时间算起的15天
- 定时检测将指定期限以上的索引快照成文件，存储到快照仓库中，然后删除
- 钉钉通知进行处理的索引名称
- 脚本执行错误时进行Sentry告警

```bash
#!/usr/bin/python3
# -*- coding: UTF-8 -*-

import json,time,requests,sentry_sdk
from elasticsearch import Elasticsearch

es = Elasticsearch(
    ["192.168.1.6","192.168.1.7"],
    # es用户角色权限要求：集群权限：monitor、create_snapshot 索引权限：*(所有索引) monitor、delete_index
    http_auth=('es用户名', 'es用户密码'),
    scheme="http",
    port=9200,
    http_compress=True
)

app_index_retain_day=15
nginx_index_retain_day=15

# Sentry DSN
sentry_sdk.init(dsn='http://*****:*****@sentry.Curiouser.com/12')

# 钉钉机器人Token
dingding_webhook_token="*****"

# 获取所有索引
def getAllIndex():
    return es.cat.indices('*', h='index,cd', format='json', s='index')

# 将获取到的所有索引去除"."开头的、名字异常的或想排除的
def getExcludeSystemAndAberrantIndex():
    return list(filter(lambda x: (not ( x['index'].startswith('.') or '%{[app]}' in x['index'] or x['index'].startswith('gitlab-production') or x['index'].startswith('jaeger') )), getAllIndex()))

# 获取应用日志索引
def getAppIndex():
    return list(filter(lambda x: ( not ('nginx' in x['index'] or 'mysql-slowlog' in x['index'] )), getExcludeSystemAndAberrantIndex()))

# 获取Nginx日志索引
def getNginxIndex():
    return list(filter(lambda x: ( 'nginx' in x['index'] ), getExcludeSystemAndAberrantIndex()))

# 获取MySQL慢日志索引
def getMysqlSlowQueryLogIndex():
    return list(filter(lambda x: ('mysql-slowlog' in x['index']), getAllIndex()))

# Snapshots索引
def snapshotIndex(index):
    index_body = {"indices": index }
    print(index)
    return es.snapshot.create(body=index_body,repository='NAS-NFS-Snapshots-Repository', wait_for_completion='true', request_timeout=300, snapshot= index+'-snapshoted-'+ time.strftime('%m-%d') )

# 删除索引
def deleteIndex(index):
    es.indices.delete(index=index)

# 钉钉通知
def dingdingNotification(token,msg,day):

    url = "https://oapi.dingtalk.com/robot/send?access_token="+token
    headers = { "Content-Type": "application/json", "Charset": "UTF-8" }
    # 构建请求数据，post请求
    data = {
        "msgtype": "text",
        "text": {
            "content": msg+"\n"
        },
        "at": {
            "isAtAll": 'true'
        }
    }

    if not requests.post(url, data=json.dumps(data), headers=headers) :
        print("发送钉钉通知失败！")
        sentry_sdk.capture_exception(Exception("发送钉钉通知失败！"))
# 将创建日志超过指定天数的日志索引快照到存储仓库中，然后删除
def snapshotAndDeleteAppIndex(type,day):

    if type == 'app' :
        snapshoted_deleted_app_indices=[]
        for i in getAppIndex():
            cts=time.time()
            if ( (cts - int(i["cd"])/1000) ) > day*86400 :
                 if 'SUCCESS' in snapshotIndex(i["index"])['snapshot']["state"]:
                    deleteIndex(i['index'])
                    print(i['index']+ "已在ES中快照并删除！")
                    snapshoted_deleted_app_indices.append(i['index'])
                 else:
                    print("应用日志索引："+i['index']+"快照失败")
                    sentry_sdk.capture_exception(Exception("应用日志索引："+i['index']+"快照失败"))
                    continue
        if snapshoted_deleted_app_indices :
            Notification_Context="[索引快照清理任务]\n成功将以下"+str(day)+"天之前的应用日志索引进行了快照\n"+"\n".join(str(i) for i in  snapshoted_deleted_app_indices)
            dingdingNotification(dingding_webhook_token,Notification_Context,day)
        else:
            Notification_Context = "[索引快照清理任务]\n没有超过"+ str(day)+"天的应用日志索引需要被快照删除！"
            dingdingNotification(dingding_webhook_token, Notification_Context, day)
    elif type == 'nginx' :
        snapshoted_deleted_nginx_indices = []
        for i in getNginxIndex():
            cts=time.time()
            if ( (cts - int(i["cd"])/1000) ) > day*86400 :
                if 'SUCCESS' in snapshotIndex(i["index"])['snapshot']["state"]:
                    deleteIndex(i['index'])
                    print(i['index'] + "已在ES中快照并删除！")
                    snapshoted_deleted_nginx_indices.append(i['index'])
                else:
                    print("Nginx日志索引：" + i['index'] + "快照失败")
                    sentry_sdk.capture_exception(Exception("Nginx日志索引：" + i['index'] + "快照失败"))
                    continue
        if snapshoted_deleted_nginx_indices:
            Notification_Context = "[索引快照清理任务]\n成功将以下" + str(day) + "天之前的应用Nginx索引进行了快照\n" + "\n".join(str(i) for i in snapshoted_deleted_nginx_indices)
            dingdingNotification(dingding_webhook_token, Notification_Context, day)
        else:
            Notification_Context = "[索引快照清理任务]\n没有超过" + str(day) + "天的应用Nginx日志索引需要被快照删除！"
            dingdingNotification(dingding_webhook_token, Notification_Context, day)

    elif type == 'mysqlslowlog' :
        snapshoted_deleted_mysqlslowlog_indices = []
        for i in getMysqlSlowQueryLogIndex():
            cts = time.time()
            if ((cts - int(i["cd"]) / 1000)) > day * 86400:
                if 'SUCCESS' in snapshotIndex(i["index"])['snapshot']["state"]:
                    deleteIndex(i['index'])
                    print(i['index'] + "已在ES中快照并删除！")
                    snapshoted_deleted_mysqlslowlog_indices.append(i['index'])
                else:
                    print("MySQL慢查询日志索引：" + i['index'] + "快照失败")
                    sentry_sdk.capture_exception(Exception("MySQL慢查询日志索引：" + i['index'] + "快照失败"))
                    continue
        if snapshoted_deleted_mysqlslowlog_indices :
            Notification_Context = "[索引快照清理任务]\n成功将以下" + str(day) + "天之前的MySQL慢查询日志索引进行了快照\n" + "\n".join(str(i) for i in snapshoted_deleted_mysqlslowlog_indices)
            dingdingNotification(dingding_webhook_token, Notification_Context, day)
        else:
            Notification_Context = "[索引快照清理任务]\n没有超过" + str(day) + "天的MySQL慢查询日志索引需要被快照删除！"
            dingdingNotification(dingding_webhook_token, Notification_Context, day)

def main():
    print("====================="+time.strftime("%Y-%m-%d %H:%M:%S", time.localtime())+"开始清理es中的索引=====================")
    print("................开始清理"+app_index_retain_day+"天前应用相关索引................")
    # 快照删除指定期限之前的应用索引
    snapshotAndDeleteAppIndex('app',app_index_retain_day)
    print("................开始清理"+nginx_index_retain_day+"天前nginx相关索引................")
    # 快照删除指定期限之前Nginx索引
    snapshotAndDeleteAppIndex('nginx',nginx_index_retain_day)
    
    exit(0)

if __name__ == "__main__" :
    main()
```

## 2、requirements.txt

```bash
elasticsearch==7.0.0
pyyaml
requests
sentry_sdk
```

## 3、操作步骤

- Python版本：3

- 默认清理策略
    - 快照删除指定日期前的应用日记索引
    - 快照删除指定日期前的应用Nginx日记索引 （索引名包含Nginx关键字的）
    
- 安装依赖

    ```bash
    pip3 install -r requierements.txt
    ```

- 执行脚本

    ```bash
    PYTHONIOENCODING=utf-8 python3 es-index-snapshots-clean.py 
    ```

- Crontab定时执行脚本：每天凌晨1点执行

    ```bash
    0 0 1 * * ? python3 es-index-snapshots-clean.py 
    ```


# 四、官方的curator索引管理工具

https://www.elastic.co/guide/en/elasticsearch/client/curator/5.8/index.html

# 五、ES自带的ILM(index lifecycle management)功能

https://www.elastic.co/guide/en/elasticsearch/reference/7.17/index-lifecycle-management.html
