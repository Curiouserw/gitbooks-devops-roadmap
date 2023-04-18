# Python 语法总结

# 一、通过SMTP发送邮件

## 1. 发送带附件的邮件

```python
#!/usr/bin/python3
# -*- coding: UTF-8 -*-
import smtplib
import datetime
from email import encoders
from email.header import Header
from email.mime.base import MIMEBase
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText

smtp_server = 'smtp.163.com'
smtp_user = ''
smtp_user_pasw = ''
email_sender = smtp_user
# 接收者邮件地址
email_receivers = ['test1@163.com', 'test2@163.com']


def sendEmail(attachfilename):
    message = MIMEMultipart()
    message['From'] = Header(email_sender, 'utf-8')
    message['To'] = Header('; '.join(str(e) for e in email_receivers) + '; ', 'utf-8')
    # 邮件主题内容
    message['Subject'] = Header(datetime.datetime.now().strftime('%Y%m%d%H') + "Python测试SMTP发送邮件", 'utf-8')
    # 邮件正文内容
    message.attach(MIMEText("Python测试SMTP发送邮件，详情见附件Excel文件", 'plain', 'utf-8'))
    # 附件内容
    attachment = MIMEBase('application', "octet-stream")
    attachment.set_payload(open(attachfilename, "rb").read())
    encoders.encode_base64(attachment)
    attachment.add_header('Content-Disposition', 'attachment; filename="附件文件名"')
    message.attach(attachment)

    try:
        smtpObj = smtplib.SMTP(smtp_server)
        smtpObj.login(smtp_user, smtp_user_pasw)
        smtpObj.sendmail(email_sender, email_receivers, message.as_string())
        print("邮件发送成功")
    except smtplib.SMTPException:
        print("Error: 无法发送邮件")
def main():
    sendEmail("/tmp/test-email-attachfile.txt")


if __name__ == "__main__":
    main()
```

## 2. 发送邮件

```python
#!/usr/bin/python3
# -*- coding: UTF-8 -*-
import smtplib
import datetime
from email.header import Header
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText

smtp_server = 'smtp.163.com'
smtp_user = ''
smtp_user_pasw = ''
email_sender = smtp_user
email_receivers = ['test1@163.com','test2@163.com']


def sendEmail(email_context):
    message = MIMEMultipart()
    message['From'] = Header(email_sender, 'utf-8')
    message['To'] = Header(''.join(str(e) for e in email_receivers) + '; ', 'utf-8')
    message['Subject'] = Header(datetime.datetime.now().strftime('%Y%m%d%H') + "Python测试SMTP发送邮件", 'utf-8')
    message = MIMEText(email_context, 'plain', 'utf-8')
    try:
        smtpObj = smtplib.SMTP(smtp_server)
        smtpObj.login(smtp_user, smtp_user_pasw)
        smtpObj.sendmail(email_sender, email_receivers, message.as_string())
        print("邮件发送成功")
    except smtplib.SMTPException:
        print("Error: 无法发送邮件")


def main():
    sendEmail("test,test")


if __name__ == "__main__":
    main()
```

参考：

- https://www.runoob.com/python/python-email.html
- https://www.runoon.com/python3/python3-smtp-sendmail.html

# 二、写入EXCEL

参考：https://blog.csdn.net/u013250071/article/details/81911434

```python
#!/usr/bin/python3
# -*- coding: UTF-8 -*-
import xlwt
def createExcel(data,filename):
    # 创建excel工作表
    workbook = xlwt.Workbook(encoding='utf-8')
    # 给表格新建一个名为 sheer1 的工作簿
    worksheet = workbook.add_sheet('sheet1')

    # 创建一个样式对象，初始化样式
    style = xlwt.XFStyle()
    al = xlwt.Alignment()
    # 其中horz代表水平对齐方式，vert代表垂直对齐方式
    # VERT_TOP          = 0x00    上端对齐
    # VERT_CENTER    = 0x01    居中对齐（垂直方向上）
    # VERT_BOTTOM  = 0x02    低端对齐
    # HORZ_LEFT        = 0x01    左端对齐
    # HORZ_CENTER   = 0x02    居中对齐（水平方向上）
    # HORZ_RIGHT     = 0x03    右端对齐
    # style.alignment = al
    
    # 设置表头
    worksheet.write(0, 0, label='id')
    worksheet.write(0, 1, label='用户名')
    worksheet.write(0, 2, label='性别')
    worksheet.write(0, 3, label='年龄')
    worksheet.write(0, 4, label='学历')
    ordered_list = ["id", "用户名", "性别", "年龄", "学历"]
    row = 1
    for player in data:
        for _key, _value in player.items():
            col = ordered_list.index(_key)
            worksheet.write(row, col, _value, style)
        row += 1
    workbook.save(filename)

def main():
    testdata=[
        {'id': '1', '用户名': "test1", '性别': "男", '年龄': 18,'学历': "大学本科"},
        {'id': '2', '用户名': "test2", '性别': "男", '年龄': 25,'学历': "大学本科"}
    ]
    createExcel(testdata,"test.xls")

if __name__ == "__main__":
    main()
```

# 三、列表操作

## 1、遍历字典列表

```bash
#!/usr/bin/python3
# -*- coding: UTF-8 -*-
def main():
    testdata=[
        {'id': '1', '用户名': "test1", '性别': "男", '年龄': 18,'学历': "大学本科"},
        {'id': '2', '用户名': "test2", '性别': "男", '年龄': 25,'学历': "大学本科"}
    ]
    row = 1
    for i in testdata:
        for _key,_value in i.items():
            print(_key,"=",_value)
        row += 1
        
if __name__ == "__main__":
    main()
```

## 2、列表去重添加

```python
#!/usr/bin/python3
# -*- coding: UTF-8 -*-
data_example={"1": "aa", "2": "bb", "3": "cc"}
target = []
u_name = "2"
if u_name in data_example:
   target.append(json.loads(data_example).get(u_name))
print(target)
# 输出：['bb']
```



## 3. 删除字典列表元素

### ①For循环删除

```python
test_list = [{"id" : 1, "data" : "1111"},
             {"id" : 2, "data" : "2222"},
             {"id" : 3, "data" : "3333"}]
for i in range(len(test_list)):
    if test_list[i]['id'] == 2:
        del test_list[i]
        break
# 输出：   [{‘id’: 1, ‘data’: ‘1111’}, {‘id’: 3, ‘data’: ‘2222’}]
```

### ②lambda删除

```python
test_list = [{"id" : 1, "data" : "1111"},
             {"id" : 2, "data" : "2222"},
             {"id" : 3, "data" : "3333"}]
res = list(filter(lambda i: i['id'] != 2, test_list))
print (str(res))
# 输出：   [{‘id’: 1, ‘data’: ‘1111’}, {‘id’: 3, ‘data’: ‘2222’}]
```

参考：https://www.geeksforgeeks.org/python-removing-dictionary-from-list-of-dictionaries/

## 4. 列表的遍历与拼接

```python
#!/usr/bin/python3
# -*- coding: UTF-8 -*-

def main():
    test_list = ['hello','world','I\'m','Python']
    print(' '.join(str(e) for e in test_list) + '!')
if __name__ == "__main__":
    main()

# 输出
hello world I'm Python!
```

# 四、时间处理

ISO 8601

```python
#!/usr/bin/python3
# -*- coding: UTF-8 -*-
import datetime
 
created_at="2022-08-16T06:43:49.869Z"
updated_at="2022-08-16T06:48:09.429Z"

print( datetime.datetime.strptime(updated_at, "%Y-%m-%dT%H:%M:%S.%fZ")  - datetime.datetime.strptime(created_at,"%Y-%m-%dT%H:%M:%S.%fZ") )
```

```python
from datetime import datetime, timedelta

# 时间格式为ISO 8601
str_time = "2023-01-13T01:30:43.559Z"
f_time = (datetime.strptime(str_time, "%Y-%m-%dT%H:%M:%S.%fZ") + timedelta(hours=8)).strftime("%Y-%m-%d %H:%M:%S")   
print(f_time)

# 输出：2023-01-13 09:30:43
```

# 五、发送钉钉通知

```python
#!/usr/bin/python3
# encoding: utf-8
import hmac, json, base64, hashlib, urllib, mureq, time

dingding_secret = "钉钉自定义机器人Webhook的Token"
dingding_token = "钉钉自定义机器人消息加签的Secret"

ddmsg = {
    "msgtype": "markdown",
    "markdown": {
        "title": "",
        "text": "",
    },
    "at": {
        "atMobiles": [],
    }
}


def dingdingnotify(msg):
    timestamp = (round(time.time() * 1000))
    secret_enc = dingding_secret.encode('utf-8')
    string_to_sign = '{}\n{}'.format(timestamp, dingding_secret)
    string_to_sign_enc = string_to_sign.encode('utf-8')
    hmac_code = hmac.new(secret_enc, string_to_sign_enc, digestmod=hashlib.sha256).digest()
    sign = urllib.parse.quote(base64.b64encode(hmac_code))
    url = "https://oapi.dingtalk.com/robot/send?access_token=" + dingding_token + "&timestamp=" + str(
        timestamp) + "&sign=" + str(sign)
    headers = {"Content-Type": "application/json", "Charset": "UTF-8"}
    return mureq.post(url, str.encode(json.dumps(msg)), headers=headers)


if __name__ == '__main__':
    ddmsg['markdown']['title'] = "主题"
    ddmsg['markdown']['text'] = "MarkDown格式的正文，在正文里添加@人的手机号，且只有在群内的成员才可被@，非群内成员手机号会被脱敏"
    ddmsg['at']['atMobiles'] = ["要@的人的手机号1", "要@的人的手机号2", "要@的人的手机号3"]

    notify_respone = dingdingnotify(ddmsg)
```

参考：https://open.dingtalk.com/document/robots/custom-robot-access#title-zob-eyu-qse

# 六、命令参数

```python
#!/usr/bin/python3
# encoding: utf-8
import sys,argparse

def todo(arg_id, arg_name):
    print(arg_id, arg_name)


def main():
    if args.id and args.name:
        todo(args.id,args.name)

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument('--id',
                        dest='id',
                        nargs='?',
                        default=None,
                        type=int,
                        required=True,
                        help="请输入ID"
                        )
    parser.add_argument('--name',
                        dest='name',
                        nargs='?',
                        default=None,
                        type=str,
                        required=True,
                        help="请输用户名"
                        )
    args = parser.parse_args()
    sys.exit(main)
```

参考：

- https://docs.python.org/zh-cn/3/howto/argparse.html
- https://www.jianshu.com/p/5e28e0590b71
