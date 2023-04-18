通过邮件发送统计TiDB慢SQL的Python脚本

```sql
CREATE USER `export-slowsql-user`@`192.168.1.%` IDENTIFIED BY '****';
GRANT PROCESS ON INFORMATION_SCHEMA.* TO 'export-slowsql-user'@'192.168.1.%'
```



```python
#!/usr/bin/python3
# -*- coding: UTF-8 -*-
import datetime
import smtplib
import os
from email import encoders
from email.header import Header
from email.mime.base import MIMEBase
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
import mysql.connector
import xlwt

# 获取大约多少秒的慢SQL
slowtime = 2
# 获取最近多少天之前到现在的慢SQL
slow_from_day = 7
# TiDB数据库地址
DB_HOST=""
# TiDB数据库端口
DB_PORT=""
# 用于查询慢SQL的用户
DB_USER=""
DB_PASSWORD=""
# 记录慢SQL变所在的Database
DB_DATABASE="INFORMATION_SCHEMA"
# SMTP服务器的地址
SMTP_HOST=""
# SMTP用于发信的用户名
SMTP_USER=""
# SMTP用于发信的密码
SMTP_USER_PASSWORD=""
# 用于记录慢SQL的EXCEL文件名
RECORDS_EXCEL_FILENAME="tidb-slowlogs.xls"
nt = datetime.datetime.now()
lastsomeday = (nt - datetime.timedelta(days=slow_from_day)).date()


def sendEmail(attachfilename):
    sender = SMTP_USER
    # 接收者邮件地址
    receivers = ['要邮件通知的邮箱地址', '要邮件通知的邮箱地址']

    message = MIMEMultipart()
    message['From'] = Header(SMTP_USER, 'utf-8')  # 发送者
    message['To'] = Header('; '.join(str(e) for e in receivers) + '; ', 'utf-8')
    # 邮件主题内容
    message['Subject'] = Header(nt.strftime('%Y%m%d') + "TiDB最近" +
                                str(slow_from_day) + "天(" +
                                lastsomeday.strftime('%Y%m%d') + "~" + nt.strftime('%Y%m%d') +
                                ")的慢SQL", 'utf-8')
    # 邮件正文内容
    message.attach(MIMEText("TiDB最近" + str(slow_from_day) + "天("
                            + lastsomeday.strftime('%Y%m%d') + "~" +
                            nt.strftime('%Y%m%d') +
                            ")，查询时间超过" + str(slowtime) +
                            "秒的慢SQL。详情见附件Excel文件(排除了部分用户和主机客户端相关的SQL)", 'plain', 'utf-8'))
    # 附件内容
    attachment = MIMEBase('application', "octet-stream")
    attachment.set_payload(open(attachfilename, "rb").read())
    encoders.encode_base64(attachment)
    attachment.add_header('Content-Disposition',
                          'attachment; filename="' + lastsomeday.strftime('%Y%m%d') + "-" + nt.strftime(
                              '%m%d') + '-'+RECORDS_EXCEL_FILENAME+'"')
    message.attach(attachment)

    try:
        smtpObj = smtplib.SMTP(SMTP_HOST)
        smtpObj.login(SMTP_USER, SMTP_USER_PASSWORD)
        smtpObj.sendmail(sender, receivers, message.as_string())
        print("邮件发送成功")
    except smtplib.SMTPException:
        print("Error: 无法发送邮件")


def dbConnet(host, port, user, passwd, db):
    db = mysql.connector.connect(
        host=host,
        port=port,
        user=user,
        password=passwd,
        database=db
    )
    cursor = db.cursor(buffered=True, dictionary=True)
    return cursor, db


def createExcel(data, filename):
    # 创建excel工作表
    workbook = xlwt.Workbook(encoding='utf-8')
    worksheet = workbook.add_sheet('sheet1')
    # 设置表头
    worksheet.write(0, 0, label='UID')
    worksheet.write(0, 1, label='Count')
    worksheet.write(0, 2, label='数据库')
    worksheet.write(0, 3, label='执行耗时')
    worksheet.write(0, 4, label='扫描总Key个数')
    worksheet.write(0, 5, label='执行时间')
    worksheet.write(0, 6, label='执行用户')
    worksheet.write(0, 7, label='主机')
    worksheet.write(0, 8, label='返回结果行数')
    worksheet.write(0, 9, label='内存')
    worksheet.write(0, 10, label='Cop_time')
    worksheet.write(0, 11, label='Write_sql_response_total')
    worksheet.write(0, 12, label='Total_keys')
    worksheet.write(0, 13, label='Process_keys')
    worksheet.write(0, 14, label='SQL语句')
    ordered_list = ["UID",
                    "Count",
                    "数据库",
                    "执行耗时",
                    "扫描总Key个数",
                    "执行时间",
                    "执行用户",
                    "主机",
                    "返回结果行数",
                    "内存",
                    "Cop_time",
                    "Write_sql_response_total",
                    "Total_keys",
                    "Process_keys",
                    "SQL语句",
                    ]
    row = 1
    for player in data:
        for _key, _value in player.items():
            col = ordered_list.index(_key)
            worksheet.write(row, col, _value)
        row += 1
    workbook.save(filename)


def execute_sql(sql):
    mycursor, mydb = dbConnet(DB_HOST, DB_PORT, DB_USER, DB_PASSWORD, DB_DATABASE)
    mycursor.execute(sql)
    sqlresults = mycursor.fetchall()
    mydb.commit()
    mycursor.close()
    mydb.close()
    if sqlresults:
        filename = 'prod-tidb-slowlogs.xls'
        createExcel(sqlresults, filename)
        sendEmail(filename)
        if os.path.exists(RECORDS_EXCEL_FILENAME):
            os.remove(RECORDS_EXCEL_FILENAME)
            print("已删除临时文件")
        else:
            print("临时文件不存在!")
    else:
        print(lastsomeday.strftime('%Y%m%d') + "至" + nt.strftime('%Y%m%d') + "没有相关的慢SQL")


def main():
    print("====" + nt.strftime('%Y%m%d %H:%M:%s') + "====")
    query = ("SELECT "
             "Digest AS 'UID',"
             "count( 1 ) AS 'Count',"
             "DB AS '数据库',"
             "CONCAT( CAST(( Query_time ) AS CHAR ( 4 )), 's' ) AS '执行耗时',"
             "Total_keys AS '扫描总Key个数',"
             "CONCAT('\\'',Time) AS '执行时间', "
             "User AS '执行用户',"
             "Host AS '主机',"
             "Result_rows AS '返回结果行数',"
             "CONCAT( CAST(( Mem_max / 1024 / 1024 ) AS CHAR ( 4 )), 'MB' ) AS '内存',"
             "CONCAT( CAST(( Cop_time ) AS CHAR ( 4 )), 's' ) AS 'Cop_time',"
             "CONCAT( CAST(( Write_sql_response_total ) AS CHAR ( 4 )), 's' ) AS 'Write_sql_response_total',"
             "Total_keys,"
             "Process_keys,"
             "Query AS 'SQL语句'"
             "FROM	INFORMATION_SCHEMA.CLUSTER_SLOW_QUERY "
             "WHERE "
             "Query_time >= '%s' "
             "AND Time >= DATE_SUB( '%s' , INTERVAL '%s' DAY )  "
             "AND db not in  ('','要排除的DB')"
             "AND host not in  ('要排除的客户端IP地址','要排除的客户端IP地址') "
             "AND user NOT IN ( '要排除的客户端用户名' , '要排除的客户端用户名', '要排除的客户端用户名')"
             "GROUP BY Digest "
             "ORDER BY Query_time DESC , Result_rows DESC, Count DESC "
             "LIMIT 1000;" % (slowtime, nt, slow_from_day)
             )
    execute_sql(query)


if __name__ == "__main__":
    main()
```