# MySQL 常见操作

# 1、相关网站

官方文档：https://dev.mysql.com/doc/refman/5.7/en/

下载地址：https://dev.mysql.com/downloads/

# 2、常用服务端配置

```bash
[mysqld]
datadir=/data/mysql/data
# 为MySQL客户程序与服务器之间的本地通信指定一个套接字文件(Linux下默认是/var/lib/mysql/mysql.sock文件)
socket=/data/mysql/data/mysql.sock
symbolic-links=0
log-error=/data/mysql/logs/mysqld.log
pid-file=/data/mysql/data/mysqld.pid
lower_case_table_names=0
federated
init_connect='SET NAMES utf8'
character_set_server=utf8
transaction-isolation=READ COMMITTED
# key_buffer是用于索引块的缓冲区大小，增加它可得到更好处理的索引(对所有读和多重写)。 索引块是缓冲的并且被所有的线程共享，key_buffer的大小视内存大小而定。
key_buffer=384M      
# 为所有线程打开表的数量。增加该值能增加mysqld要求的文件描述符的数量。可以避免频繁的打开数据表产生的开销                              
table_cache=512
# 每个需要进行排序的线程分配该大小的一个缓冲区。增加这值加速ORDER BY或GROUP BY操作。注意：该参数对应的分配内存是每连接独占！如果有100个连接，那么实际分配的总共排序缓冲区大小为100×6=600MB
sort_buffer_size=2M        
# 读查询操作所能使用的缓冲区大小。和sort_buffer_size一样，该参数对应的分配内存也是每连接独享。                               
read_buffer_size=2M 
# 指定MySQL查询结果缓冲区的大小
query_cache_size=32M
# 在使用行指针排序之后，随机读用的。
read_rnd_buffer_size=8M 
# MyISAM表发生变化时重新排序所需的缓冲
myisam_sort_buffer_size=64M 
# 最大并发线程数，取值为服务器逻辑CPU数量×2，如果CPU支持H.T超线程，再×2
thread_concurrency=8 
#缓存可重用的线程数
thread_cache=8
# 避免MySQL的外部锁定，减少出错几率增强稳定性。 
skip-locking
# back_log参数的值指出在MySQL暂时停止响应新请求之前的短时间内多少个请求可以被存在堆栈中。如果系统在一个短时间内有很多连接，则需要增大该参数的值，该参数值指定到来的TCP/IP连接的侦听队列的大小。试图设定back_log高于你的操作系统的限制将是无效的。默认值为50。对于Linux系统推荐设置为小于512的整数。
back_log=384 
# MySQL服务器同时处理的数据库连接的最大数量(默认设置是100)。超过限制后会报 Too many connections 错误
max_connections=n 
# 用来存放索引区块的RMA值(默认设置是8M)，增加它可得到更好处理的索引(对所有读和多重写)    
key_buffer_size=n
#每个进行一个顺序扫描的线程为其扫描的每张表分配这个大小的一个缓冲区。如果你做很多顺序扫描，你可能想要增加该值。默认数值是131072(128K)    
record_buffer=131072
# 服务器在关闭它之前在一个连接上等待行动的秒数。   
wait_timeout=3
# 服务器在关闭它前在一个交互连接上等待行动的秒数。一个交互的客户被定义为对 mysql_real_connect()使用 CLIENT_INTERACTIVE 选项的客户。默认数值是28800，可以把它改为3600。 
interactive_timeout=3600
# 禁止MySQL对外部连接进行DNS解析，使用这一选项可以消除MySQL进行DNS解析的时间。但需要注意，如果开启该选项，则所有远程主机连接授权都要使用IP地址方式，否则MySQL将无法正常处理连接请求！    
skip-name-resolve            
skip-innodb
# 关闭不需要的表类型
skip-bdb 
# 开启Binlog
log-bin=/data/mysql/data/mysql-bin.log
expire-logs-days=14
max-binlog-size=500M
server-id=1
binlog_format=ROW
binlog_row_image=FULL
# 开启慢查询日志
slow_query_log=1
slow_query_log_file=/data/mysql/logs/slowquery.log
long_query_time=2

[mysqldump]
# 服务器和客户端之间最大能发送的可能信息包 
max_allowed_packet=16M

[mysql]
socket=/data/mysql/data/mysql.sock
default-character-set=utf8

[client]
default-character-set=utf8
socket=/data/mysql/data/mysql.sock
```

# 3、初始化数据目录

```bash
mkdir -p /data/mysql/{logs,data} &&\
chown -R mysql:mysql /data/mysql &&\
mysqld --initialize --user=mysql
```

# 4、修改root用户本地连接密码

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY '***';
flush privileges; 
```

或者

```sql
set password for 'root'@'localhost'=password('***');
flush privileges; 
```

# 5、添加远程登录用户

默认只允许root帐户在本地登录，如果要在其它机器上连接mysql，必须修改root允许远程连接，或者添加一个允许远程连接的帐户，为了安全起见，添加一个新的帐户：

```sql
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '***' WITH GRANT OPTION;
flush privileges; 
```

# 6、修改用户密码

## ①用set password命令 

```sql
SET PASSWORD FOR 'root'@'localhost' = PASSWORD('***');
flush privileges; 
```

## ②用mysqladmin  

```bash
mysqladmin -uroot -p*** password 1234abcd  

格式：mysqladmin -u用户名 -p旧密码 password 新密码 
```

## ③update更新user表  

```bash
11234abcd') where user = 'root';
flush privileges;

# mysql 5.7
use mysql 
update user set authentication_string = PASSWORD('1234abcd') where user = 'root';
flush privileges;
```

# 7、跳过密码验证重置root密码

- 修改my.cnf，追加**skip-grant-tables**，重启mysql服务，然后就可以不输入密码直接登录

# 8、表复制

## 复制表结构及数据到新表

```
CREATE TABLE 新表 SELECT * FROM 旧表 
```

## 只复制表结构到新表

```
CREATE TABLE 新表 SELECT * FROM 旧表 WHERE 1=2
 
即:让WHERE条件不成立.
```

**方法二:(低版本的mysql不支持，mysql4.0.25 不支持，mysql5已经支持了)**

```
CREATE TABLE 新表 LIKE 旧表 
```

## 复制旧表的数据到新表(假设两个表结构一样)

```
INSERT INTO 新表 SELECT * FROM 旧表 
```

## 复制旧表的数据到新表

(假设两个表结构不一样)

```
INSERT INTO 新表(字段1,字段2,…….) SELECT 字段1,字段2,…… FROM 旧表
```

参考: https://www.cnblogs.com/lxboy2009/p/7234535.html

# 9、show status命令

`show status like '%下面变量%';`

```bash
Aborted_clients 							由于客户没有正确关闭连接已经死掉，已经放弃的连接数量。 
Aborted_connects 							尝试已经失败的MySQL服务器的连接的次数。 
Connections 									试图连接MySQL服务器的次数。 
Created_tmp_tables 						当执行语句时，已经被创造了的隐含临时表的数量。 
Delayed_insert_threads 				正在使用的延迟插入处理器线程的数量。 
Delayed_writes								用INSERT DELAYED写入的行数。 
Delayed_errors 								用INSERT DELAYED写入的发生某些错误(可能重复键值)的行数。 
Flush_commands 								执行FLUSH命令的次数。 
Handler_delete 								请求从一张表中删除行的次数。 
Handler_read_first 						请求读入表中第一行的次数。 
Handler_read_key 							请求数字基于键读行。 
Handler_read_next 						请求读入基于一个键的一行的次数。 
Handler_read_rnd 							请求读入基于一个固定位置的一行的次数。 
Handler_update 								请求更新表中一行的次数。 
Handler_write 								请求向表中插入一行的次数。 
Key_blocks_used 							用于关键字缓存的块的数量。 
Key_read_requests 						请求从缓存读入一个键值的次数。 
Key_reads 										从磁盘物理读入一个键值的次数。 
Key_write_requests 						请求将一个关键字块写入缓存次数。 
Key_writes 										将一个键值块物理写入磁盘的次数。 
Max_used_connections 					同时使用的连接的最大数目。 
Not_flushed_key_blocks 				在键缓存中已经改变但是还没被清空到磁盘上的键块。 
Not_flushed_delayed_rows 			在INSERT DELAY队列中等待写入的行的数量。 
Open_tables 									打开表的数量。 
Open_files 										打开文件的数量。 
Open_streams 									打开流的数量(主要用于日志记载） 
Opened_tables 								已经打开的表的数量。 
Questions 										发往服务器的查询的数量。 
Slow_queries 									要花超过long_query_time时间的查询数量。 
Threads_connected 						当前打开的连接的数量。 
Threads_running 							不在睡眠的线程数量。 
Uptime 												服务器工作了多少秒。
```

# 10、查看状态或监控信息

```sql
# 如果是root帐号，你能看到所有用户的当前连接。如果是其它普通帐号，只能看到自己占用的连接。 
# show processlist;只列出前100条，
show processlist;
# 如果想全列出请使用
show full processlist;

# 最大使用连接数
show status like 'Max_used_connections'
# 当前打开的连接数
show status like 'Threads_connected'
# 未从缓冲池读取的次数
show status like 'Innodb_buffer_pool_reads'
# 从缓冲池读取的次数
show status like 'Innodb_buffer_pool_read_requests'
# 缓冲池的总页数
show status like 'Innodb_buffer_pool_pages_total'
# 缓冲池空闲的页数
show status like 'Innodb_buffer_pool_pages_free'
# 缓存命中率计算
（1-Innodb_buffer_pool_reads/Innodb_buffer_pool_read_requests）*100%
# 缓存池使用率为
((Innodb_buffer_pool_pages_total-Innodb_buffer_pool_pages_free）/Innodb_buffer_pool_pages_total）*100%

# 锁等待个数
show status like 'Innodb_row_lock_waits'
# 平均每次锁等待时间
show status like 'Innodb_row_lock_time_avg'
# 查看是否存在表锁(有数据代表存在锁表，空为无表锁)
show open TABLES where in_use > 0;

# insert 数量
show status like 'Com_insert'
# delete 数量
show status like 'Com_delete'
# update 数量
show status like 'Com_update'
# select 数量
show status like 'Com_select'
# 发送吞吐量
show status like 'Bytes_sent'
# 接收吞吐量
show status like 'Bytes_received'

# 总吞吐量：Bytes_sent+Bytes_received

show variables like 'long_query_time'
```

# 11、查看支持的引擎

```bash
show engines；
```

# 12、清空表中数据

```bash
# truncate语句直接清空表中数据
truncate 表名；
# drop语句清空表中数据
delete from 表名;
```

# 13、导入导出数据到CSV

## 导出

```bash
select * from Test.User into 
	outfile 'UserData.csv'
	fields terminated by '@'           # 字段间以@分割
	optionally enclosed by "" 
	escaped by "Curiouser"				
  lines terminated by '\n';         # 数据行之间以\n分割
```

如果在导出期间出现以下错误

> The MySQL server is running with the --secure-file-priv option so it cannot execute this statement
>
> 可以用sql>show variables like '%secure%'查看secure-file-priv当前的值。导出的数据文件必须是这个值的指定路径才可以。默认有可能是NULL就代表禁止导出。可在配置文件中设置此变量的值,然后重启服务。
>
> [mysqld]
>
> secure-file-priv=/home/Curiouser/Desktop

## 导入

```
sql>load data infile '/var/lib/mysql-files/UserData.csv' 
			into table User
			fields terminated by '@'
			optionally enclosed by ""
			escaped by ""
			lines terminated by '\n';
```

当CSV文件中的每一行记录的列数小于数据库表时，使用下列语句：

```
sql>load data infile '/var/lib/mysql-files/UserData.csv' 
 			into table User
 			fields terminated by '@'
 			lines terminated by '\n'
 			(Id,Name,Addr,Phone);
```

# 14、MySQL字符集的设置

## 设置MySQL默认字符集

- 基于session会话

  ```bash
  set character_set_client=utf8mb4;
  # 主要用来设置客户端使用的字符集。
  set character_set_database=utf8mb4;
  # 主要用来设置默认创建数据库的编码格式，如果在创建数据库时没有设置编码格式，就按照这个格式设置。
  set character_set_server=utf8mb4;
  # 服务器安装时指定的默认编码格式，这个变量建议由系统自己管理，不要人为定义
  set character_set_connection=utf8mb4;
  # 主要用来设置连接数据库时的字符集，如果程序中没有指明连接数据库使用的字符集类型则按照这个字符集设置。
  set character_set_results=utf8mb4;
  # 数据库给客户端返回时使用的编码格式，如果没有指明，使用服务器默认的编码格式。
  set character_set_system=utf8mb4;
  # 数据库系统使用的编码格式，这个值一直是utf8，不需要设置，它是为存储系统元数据的编码格式。
  set collation_connection=utf8mb4;
  set collation_server=utf8mb4;
  set collation_database=utf8mb4;
  ```

- 基于全局global

  ```bash
  # 设置全局的数据库字符编码
  set global character_set_database=utf8mb4;
  set global character_ser_server=utf8mb4;
  ```

- 永久性改变,在配置文件中修改数据库的字符编码(需重启服务)

  ```bash
  [mysqld]
  character-set-server=utf8mb4 
  [client]
  default-character-set=utf8mb4 
  [mysql]
  default-character-set=utf8mb4
  ```

## 字符集的查看

- 查看MySQL支持的字符集

  ```sql
  show charset;
  
  +----------+---------------------------------+---------------------+--------+
  | Charset  | Description                     | Default collation   | Maxlen |
  +----------+---------------------------------+---------------------+--------+
  | big5     | Big5 Traditional Chinese        | big5_chinese_ci     |      2 |
  | dec8     | DEC West European               | dec8_swedish_ci     |      1 |
  | cp850    | DOS West European               | cp850_general_ci    |      1 |
  | hp8      | HP West European                | hp8_english_ci      |      1 |
  | koi8r    | KOI8-R Relcom Russian           | koi8r_general_ci    |      1 |
  | latin1   | cp1252 West European            | latin1_swedish_ci   |      1 |
  | latin2   | ISO 8859-2 Central European     | latin2_general_ci   |      1 |
  | swe7     | 7bit Swedish                    | swe7_swedish_ci     |      1 |
  | ascii    | US ASCII                        | ascii_general_ci    |      1 |
  | ujis     | EUC-JP Japanese                 | ujis_japanese_ci    |      3 |
  | sjis     | Shift-JIS Japanese              | sjis_japanese_ci    |      2 |
  | hebrew   | ISO 8859-8 Hebrew               | hebrew_general_ci   |      1 |
  | tis620   | TIS620 Thai                     | tis620_thai_ci      |      1 |
  | euckr    | EUC-KR Korean                   | euckr_korean_ci     |      2 |
  | koi8u    | KOI8-U Ukrainian                | koi8u_general_ci    |      1 |
  | gb2312   | GB2312 Simplified Chinese       | gb2312_chinese_ci   |      2 |
  | greek    | ISO 8859-7 Greek                | greek_general_ci    |      1 |
  | cp1250   | Windows Central European        | cp1250_general_ci   |      1 |
  | gbk      | GBK Simplified Chinese          | gbk_chinese_ci      |      2 |
  | latin5   | ISO 8859-9 Turkish              | latin5_turkish_ci   |      1 |
  | armscii8 | ARMSCII-8 Armenian              | armscii8_general_ci |      1 |
  | utf8     | UTF-8 Unicode                   | utf8_general_ci     |      3 |
  | ucs2     | UCS-2 Unicode                   | ucs2_general_ci     |      2 |
  | cp866    | DOS Russian                     | cp866_general_ci    |      1 |
  | keybcs2  | DOS Kamenicky Czech-Slovak      | keybcs2_general_ci  |      1 |
  | macce    | Mac Central European            | macce_general_ci    |      1 |
  | macroman | Mac West European               | macroman_general_ci |      1 |
  | cp852    | DOS Central European            | cp852_general_ci    |      1 |
  | latin7   | ISO 8859-13 Baltic              | latin7_general_ci   |      1 |
  | utf8mb4  | UTF-8 Unicode                   | utf8mb4_general_ci  |      4 |
  | cp1251   | Windows Cyrillic                | cp1251_general_ci   |      1 |
  | utf16    | UTF-16 Unicode                  | utf16_general_ci    |      4 |
  | utf16le  | UTF-16LE Unicode                | utf16le_general_ci  |      4 |
  | cp1256   | Windows Arabic                  | cp1256_general_ci   |      1 |
  | cp1257   | Windows Baltic                  | cp1257_general_ci   |      1 |
  | utf32    | UTF-32 Unicode                  | utf32_general_ci    |      4 |
  | binary   | Binary pseudo charset           | binary              |      1 |
  | geostd8  | GEOSTD8 Georgian                | geostd8_general_ci  |      1 |
  | cp932    | SJIS for Windows Japanese       | cp932_japanese_ci   |      2 |
  | eucjpms  | UJIS for Windows Japanese       | eucjpms_japanese_ci |      3 |
  | gb18030  | China National Standard GB18030 | gb18030_chinese_ci  |      4 |
  +----------+---------------------------------+---------------------+--------+
  ```

- 查看MySQL已配置的默认字符设置

  ```sql
  show variables like '%character%';
  
  +--------------------------+----------------------------+
  | Variable_name            | Value                      |
  +--------------------------+----------------------------+
  | character_set_client     | utf8mb4                    |
  | character_set_connection | utf8mb4                    |
  | character_set_database   | latin1                     |
  | character_set_filesystem | binary                     |
  | character_set_results    | utf8mb4                    |
  | character_set_server     | latin1                     |
  | character_set_system     | utf8                       |
  | character_sets_dir       | /usr/share/mysql/charsets/ |
  +--------------------------+----------------------------+
  ```

- 查看`库`已配置的字符设置

  ```sql
  show create database test1;
  +----------+----------------------------------------------------------------------+
  | Database | Create Database                                                      |
  +----------+----------------------------------------------------------------------+
  | test1    | CREATE DATABASE `test1` /*!40100 DEFAULT CHARACTER SET utf8mb4 */    |
  +----------+----------------------------------------------------------------------+
  ```

- 查看`表`已配置的字符设置

  ```sql
  show table status from 库名 like '表名';
  ```

- 查看`表中字段`已配置的字符设置

  ```sql
  show full columns from 表名;
  ```

- 查看`库中所有表`的字符集设置

  ```sql
  select TABLE_NAME,TABLE_COLLATION from information_schema.`TABLES`;
  ```

- 查看`所有库所有表中的字段`的字符集设置

  ```bash
  select TABLE_SCHEMA ,TABLE_NAME,COLUMN_NAME,COLLATION_NAME from information_schema.`COLUMNS`
  ```

## 修改字符集

- 修改库的字符集

  ```sql
  alter database 库名 default character set 字符集;
  ```
  
- 修改表的字符集

  ```sql
  alter table 表名 convert to character set 字符集;
  ```

- 修改字段的字符集

  ```sql
  alter table 表名 modify 字段名 字段属性 character set 字符集；
  ```
# 15、Mysql大小写敏感设置

**MySQL在Windows下都不区分大小写。 Linux下MySQL安装完后是默认区分表名的大小写，但不区分列名的大小写。数据库名、表名、列名、别名大小写规则是这样的**： 

1. 数据库名与表名是严格区分大小写

2. 表的别名是严格区分大小写
3. 列名与列的别名在所有的情况下均是忽略大小写
4. 变量名也是严格区分大小写

**修改大小写敏感设置**

1. 在`/etc/my.cnf `中的`[mysqld`]后添加添加`lower_case_table_names=1（0:区分大小写，1:不区分大小写)`，重启MYSQL服务

   - **为0时**
     表示区分大小写，使用CREATE TABLE或CREATE DATABASE语句指定的大小写字母在硬盘上保存表名和数据库名。名称比较对大小写敏感。在大小写不敏感的操作系统如windows或Mac OS x上我们不能将该参数设为0，如果在大小写不敏感的文件系统上将--lowercase-table-names强制设为0，并且使用不同的大小写访问MyISAM表名，可能会导致索引破坏。
   - **为1时**
     表示将名字转化为小写后存储，名称比较对大小写不敏感。MySQL将所有表名转换为小写在存储和查找表上。该行为也适合数据库名和表的别名。该值为Windows的默认值。
   - **为2时**
     表名和数据库名在硬盘上使用CREATE TABLE或CREATE DATABASE语句指定的大小写字母进行保存，但MySQL将它们转换为小写在查找表上。名称比较对大小写不敏感，即按照大小写来保存，按照小写来比较。注释：只在对大小写不敏感的文件系统上使用! innodb表名用小写保存。如果你使用innodb表，为了避免避免大小写敏感问题，可以把lower_case_table_names=1把lower_case_table_names从0改变为1在你把lower_case_table_names设置为1时，在restart你的mysqld之前，请把数据库名和表名更改为小写

2. 如果想在查询时区分字段值的大小写，则字段值需要设置BINARY属性

   - 创建时设置

     ```sql
     CREATE TABLE T( 
     	A VARCHAR(10) BINARY 
     ); 
     ```

   - 使用alter修改

     ```sql
     ALTER TABLE `tablename` MODIFY COLUMN `cloname` VARCHAR(45) BINARY; 
     ```

     

# 16、查看数据库或表容量大小

- ### 查看所有数据库容量大小

  ```sql
  select 
  	table_schema as '数据库',
  	sum(table_rows) as '记录数',
  	sum(truncate(data_length/1024/1024, 2)) as '数据容量(MB)',
  	sum(truncate(index_length/1024/1024, 2)) as '索引容量(MB)'
  from information_schema.tables
  group by table_schema
  order by sum(data_length) desc, sum(index_length) desc;
  ```

- ### 查看所有数据库各表容量大小

  ```bash
  select 
  	table_schema as '数据库',
  	table_name as '表名',
  	table_rows as '记录数',
  	truncate(data_length/1024/1024, 2) as '数据容量(MB)',
  	truncate(index_length/1024/1024, 2) as '索引容量(MB)'
  from information_schema.tables
  order by data_length desc, index_length desc;
  ```

- ### 查看指定数据库容量大小

  ```bash
  select 
  	table_schema as '数据库',
  	sum(table_rows) as '记录数',
  	sum(truncate(data_length/1024/1024, 2)) as '数据容量(MB)',
  	sum(truncate(index_length/1024/1024, 2)) as '索引容量(MB)'
  from information_schema.tables
  where table_schema='mysql';
  ```

- ### 查看指定数据库各表容量大小

  ```bash
  select 
  	table_schema as '数据库',
  	table_name as '表名',
  	table_rows as '记录数',
  	truncate(data_length/1024/1024, 2) as '数据容量(MB)',
  	truncate(index_length/1024/1024, 2) as '索引容量(MB)'
  from information_schema.tables
  where table_schema='mysql'
  order by data_length desc, index_length desc;
  ```


# 17、开起慢查询日志

参考：[采集MySQL慢查询日志到Elasticsearch中附录的第二小节](logstash-采集MySQL慢查询日志到Elasticsearch.md)

MySQL可以将执行超过指定时间的DQL、DML、DDL等语句记录下来。默认慢查询日志记录是关闭的

- **log_slow_queries** ：表示是否开启慢查询日志，5.6以前的版本使用此参数指定是否开启慢查询日志，5.6以后的版本使用`slow_query_log`取代此参数，如果你使用的mysql版本刚好是5.5，那么你可以看到这两个参数同时存在，此时我们不用同时设置它们，设置这两个参数中的任何一个，另一个也会自动保持一致。

- **log_output** : 表示当慢查询日志开启以后，以哪种方式存放，log_output可以设置为4种值，"`FILE`"、"`TABLE`"、"`FILE,TABLE`"、"`NONE`"。此值为"`FILE`"表示慢查询日志存放于指定的文件中，此值为"`TABLE`"表示慢查询日志存放于mysql库的`slow_log`表中，此值为"`FILE,TABLE`"表示将慢查询日志同时存放于指定的文件与`slow_log`表中，一般不会进行这样的设置，因为这样会徒增很多IO压力，如果开启，建议设置为"`table`",此值为"`NONE`"时表示不记录查询日志，即使`slow_query_log`设置为`ON`，如果`log_output`设置为`NONE`，也不会记录慢查询日志，其实，`log_output`不止用于控制慢查询日志的输出，查询日志的输出也是由此参数进行控制，也就是说，`log_output`设置为`file`，就表示查询日志和慢查询日志都存放到对应的文件中，设置为`table`，查询日志和慢查询日志就都存放在对应的数据库表中。

- **slow_query_log** ：表示是否开启慢查询日志，此参数与`log_slow_queries`的作用没有区别，5.6以后的版本使用此参数替代`log_slow_queries`。

- **slow_query_log_file** ：当使用文件存储慢查询日志时(log_output设置为"`FILE`"或者"`FILE,TABLE`"时)，指定慢查询日志存储于哪个日志文件中，默认的慢查询日志文件名为"`主机名-slow.log`"，慢查询日志的位置为`datadir`参数所对应的目录位置，一般情况下为 `/var/lib/mysql`

- **long_query_time** ：表示"多长时间的查询"被认定为"慢查询"，此值得默认值为10秒，表示超过10秒的查询被认定为慢查询。

- **log_queries_not_using_indexes** ：表示如果运行的sql语句没有使用到索引，是否也被当做慢查询语句记录到慢查询日志中，`OFF`表示不记录，`ON`表示记录。

- **log_throttle_queries_not_using_indexes** ：5.6.5版本新引入的参数，当`log_queries_not_using_inde`设置为`ON`时，没有使用索引的查询语句也会被当做慢查询语句记录到慢查询日志中，使用`log_throttle_queries_not_using_indexes`可以限制这种语句每分钟记录到慢查询日志中的次数，因为在生产环境中，有可能有很多没有使用索引的语句，此类语句频繁的被记录到慢查询日志中，可能会导致慢查询日志快速不断的增长，管理员可以通过此参数进行控制。
- **min_examined_row_limit** ：扫描记录少于改值的SQL不记录到慢查询日志，结合去记录没有使用索引的SQL语句的例子，有可能存在某一个表，数据量维持在几行左右，且没有建立索引。这种表即使不建立索引，查询也很快，扫描记录很小，如果确定有这种表，则可以通过此参数设置，将这个SQL不记录到慢查询日志。
- **log_slow_admin_statements**：记录超时的管理操作SQL到慢查询日志，比如`ALTER/ANALYZE TABLE`
-  **log_slow_slave_statements**：在从服务器上开启慢查询日志
- **log_timestamps(5.7+)**： 写入时区信息。可根据需求记录UTC时间或者服务器本地系统时间

查询慢日志是否开起等其他参数

```bash
# 查询慢日志是否开启
show variables like 'slow_query%';
# 查询多少秒的查询视为慢查询
show variables like 'long_query_time%';
# 查询慢查询日志输出到哪儿。
show variables like 'log_output%';
```

MySQL 8

```bash
set global log_output=‘FILE’; – 开启慢日志,纪录到 mysql.slow_log 表
set global long_query_time=2; – 设置超过0.1秒的查询为慢查询
set global slow_query_log=‘ON’;-- 打开慢日志记录
```

# 18、慢查询日志统计分析工具mysqldumpslow

通过`mysqldumpslow`命令我们可以更加方便的从不同的维度对慢日志进行排序、查找、统计。但是`mysqldumpslow`只能作用于慢查询日志文件

```bash
-s：排序规则参数
	c: 执行计数
  l: 锁定时间
  r: 返回记录
  t: 执行时间
  al:平均锁定时间
  ar:平均返回记录数
  at:平均执行时间
-t 是top n的意思，返回多少条数据。
-g 可以跟上正则匹配模式，大小写不敏感。

# 得到返回记录最多的20个sql
mysqldumpslow -s r -t 20 sqlslow.log
# 得到平均访问次数最多的20条sql
mysqldumpslow -s ar -t 20 sqlslow.log
# 得到平均访问次数最多,并且里面含有ttt字符的20条sql
mysqldumpslow -s ar -t 20 -g "ttt" sqldlow.log

# 如果出现 -bash: mysqldumpslow: command not found 错误，请执行"ln -s /usr/local/mysql/bin/mysqldumpslow /usr/bin"
# 如果出现如下错误，Died at /usr/bin/mysqldumpslow line 161, <> chunk 405659.说明你要分析的sql日志太大了，请拆分后再分析
```

# 19、锁表锁库操作

如果对mysql进行操作时，网络中断或SQL异常 , 可能会导致表或者库卡死 , 锁死，无法进行后续操作 （如果用 navicat 等工具连接操作, 操作都会在转圈圈，无法完成操作）。此时要找出造成锁库锁表的SQL语句的进程，将其杀死，中断其操作执行，即可解决锁库锁表。

①查看锁死SQL的进程ID

```bash
# 进程ID在trx_mysql_thread_id那一列
select id, db, user, host, command, time, state, info
from information_schema.processlist
where command != 'Sleep'
order by time desc 
```

或者

```bash
SELECT
	concat( 'kill ', id, ';' ) 
FROM
	information_schema.PROCESSLIST 
WHERE
	Command = 'Sleep' 
	AND Time > 10 
	AND db = 'test' 
ORDER BY
	Time DESC;
```

```bash
# sql语句在info列，进程ID在ID列
show processlist; 
```

②杀死相关进程

```bash
kill -9 进程号
```



# 20、统计DB所有表的实际条数

如果从`information_schema.tables` 表统计获取各表的信息话，对于非事务性表，`table_rows` 这个值是精确的，对于事务性引擎，这个值通常是估算的。例如 MyISAM，存储精确的数目。对于其它存储引擎，比如 InnoDB ，本值是一个大约的数，与实际值相差可达 40 到 50% 。在这些情况下，使用 `SELECT COUNT(*)` 来获得准确的数目。对于在 `information_schema` 数据库中的表， Rows 值为 NULL 。

```bash
SELECT CONCAT( 'SELECT "', TABLE_NAME, '", COUNT(*) FROM ', TABLE_SCHEMA, '.', TABLE_NAME, ' UNION ALL' )  EXEC_SQL
FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA = 'DB名字';
```

上述SQL会输出用于统计指定DB中所有表行数的SQL语句，复制以后，删除最后一行末尾的`UNION ALL`，然后执行即可获取所有表的实际条数

**参考**：https://commandnotfound.cn/sql/7/345/MySQL-%E6%9F%A5%E8%AF%A2%E6%95%B0%E6%8D%AE%E5%BA%93%E4%B8%8B%E5%90%84%E4%B8%AA%E8%A1%A8%E7%9A%84%E8%A1%8C%E6%95%B0%E4%BF%A1%E6%81%AF



# 21、Ubuntu20 APT安装MySQL5.7

```bash
wget https://dev.mysql.com/get/mysql-apt-config_0.8.12-1_all.deb
dpkg -i mysql-apt-config_0.8.12-1_all.deb
# ---会弹出CUI对话页面,安装后续提示确定即可---> Ubuntu Bionic，Ok--->这届
```

```bash
bash -c 'cat > /etc/apt/sources.list.d/mysql.list <<OF
deb http://repo.mysql.com/apt/ubuntu/ bionic mysql-apt-config
deb http://repo.mysql.com/apt/ubuntu/ bionic mysql-5.7
deb http://repo.mysql.com/apt/ubuntu/ bionic mysql-tools
deb http://repo.mysql.com/apt/ubuntu/ bionic mysql-tools-preview
deb-src http://repo.mysql.com/apt/ubuntu/ bionic mysql-5.7
EOF' && \
apt update && \
apt-cache policy mysql-server && \
apt install -y mysql-client=5.7* mysql-community-server=5.7* mysql-server=5.7* 

```

# 22、设置MySQL8默认证方式

MySQL 8.0 调整了账号认证方式，把 `caching_sha2_password` 插件认证方式作为默认首选，这就导致很多需要使用密码登录的客户端，远程登录 MySQL 时报错：

```sql
MySQL said: Authentication plugin 'caching_sha2_password'
cannot be loaded: dlopen(
/usr/local/lib/plugin/caching_sha2_password.so, 2): not found
```



- MySQL添加以下配置到mysqld.conf中并重启

  ```bash
  default_authentication_plugin=mysql_native_password
  ```

- 对于已有用户的认证方式变更

  ```bash
  ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '**密码**';
  ```

