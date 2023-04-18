# MysqlDump常用操作

# 1、备份所有数据库

```
$> mysqldump -uroot -proot --all-databases > /data/mysql-backup/all.sql
```

# 2、备份db1、db2两个数据库的所有数据

```
$> mysqldump -uroot -proot --databases db1 db2 > /data/mysql-backup/user.sql
```

# 3、备份db1中的a1、a2表

```
$> mysqldump -uroot -proot --databases db1 --tables a1 a2  > /data/mysql-backup/db1.sql
```

PS:导出指定表只能针对一个数据库进行导出，且导出的内容中和导出数据库也不一样，导出指定表的导出文本中没有创建数据库的判断语句，只有删除表-创建表-导入数据

# 4、筛选导出表

```bash
mysqldump -u user -proot --databases db1 db2 --ignore-table=db1.tbl --ignore-table=db2.tb2  > /data/mysql-backup/db.sql
```

# 5、只导出表结构不导出数据，--no-data 

```
$> mysqldump -uroot -proot --no-data --databases db1 >/tmp/db1.sql
```

# 6、跨服务区导出导入数据库

**PS：远程数据库必须存在**

```bash
# 将h1服务器中的db1数据库的所有数据导入到h2中的db2数据库中
mysqldump --host=192.168.1.8 -uroot -proot -C --databases test | mysql --host=192.168.1.9 -uroot -proot test 
# PS：远程数据库必须存在
```

# 7、定时备份任务

```bash
# 每天早上1点定时备份MySQL数据
0 1 * * * mysqldump -u user -proot --databases db1 db2 --ignore-table=db1.tb1 --ignore-table=db2.tbl1  > /data/mysql-backup/db-$(date +"\%Y\%m\%d\%H\%M").sql
```

# 8、MysqlDump的参数说明

```
--all-databases  , -A  导出全部数据库。
mysqldump  -uroot -p --all-databases
--all-tablespaces  , -Y 导出全部表空间。
mysqldump  -uroot -p --all-databases --all-tablespaces
--no-tablespaces  , -y 不导出任何表空间信息。
mysqldump  -uroot -p --all-databases --no-tablespaces
--add-drop-database 每个数据库创建之前添加drop数据库语句。
mysqldump  -uroot -p --all-databases --add-drop-database
--add-drop-table 每个数据表创建之前添加drop数据表语句。(默认为打开状态，使用--skip-add-drop-table取消选项)
mysqldump  -uroot -p --all-databases  (默认添加drop语句)
mysqldump  -uroot -p --all-databases –skip-add-drop-table  (取消drop语句)
--add-locks
在每个表导出之前增加LOCK TABLES并且之后UNLOCK  TABLE。(默认为打开状态，使用--skip-add-locks取消选项)
mysqldump  -uroot -p --all-databases  (默认添加LOCK语句)
mysqldump  -uroot -p --all-databases –skip-add-locks   (取消LOCK语句)
--allow-keywords 允许创建是关键词的列名字。这由表名前缀于每个列名做到。
mysqldump  -uroot -p --all-databases --allow-keywords
--apply-slave-statements 在'CHANGE MASTER'前添加'STOP SLAVE'，并且在导出的最后添加'START SLAVE'。
mysqldump  -uroot -p --all-databases --apply-slave-statements
--character-sets-dir  字符集文件的目录
mysqldump  -uroot -p --all-databases  --character-sets-dir=/usr/local/mysql/share/mysql/charsets
--comments
附加注释信息。默认为打开，可以用--skip-comments取消
mysqldump  -uroot -p --all-databases  (默认记录注释)
mysqldump  -uroot -p --all-databases --skip-comments   (取消注释)
--compatible
导出的数据将和其它数据库或旧版本的MySQL 相兼容。值可以为ansi、mysql323、mysql40、postgresql、oracle、mssql、db2、maxdb、no_key_options、no_tables_options、no_field_options等，
要使用几个值，用逗号将它们隔开。它并不保证能完全兼容，而是尽量兼容。
mysqldump  -uroot -p --all-databases --compatible=ansi
--compact
导出更少的输出信息(用于调试)。去掉注释和头尾等结构。可以使用选项：--skip-add-drop-table  --skip-add-locks --skip-comments --skip-disable-keys
mysqldump  -uroot -p --all-databases --compact
--complete-insert,  -c
使用完整的insert语句(包含列名称)。这么做能提高插入效率，但是可能会受到max_allowed_packet参数的影响而导致插入失败。
mysqldump  -uroot -p --all-databases --complete-insert
--compress, -C
在客户端和服务器之间启用压缩传递所有信息
mysqldump  -uroot -p --all-databases --compress
--create-options,  -a
在CREATE TABLE语句中包括所有MySQL特性选项。(默认为打开状态)
mysqldump  -uroot -p --all-databases
--databases,  -B
导出几个数据库。参数后面所有名字参量都被看作数据库名。
mysqldump  -uroot -p --databases test mysql
--debug
输出debug信息，用于调试。默认值为：d:t,/tmp/mysqldump.trace


mysqldump  -uroot -p --all-databases --debug
mysqldump  -uroot -p --all-databases --debug=” d:t,/tmp/debug.trace”
--debug-check
检查内存和打开文件使用说明并退出。
mysqldump  -uroot -p --all-databases --debug-check
--debug-info
输出调试信息并退出
mysqldump  -uroot -p --all-databases --debug-info
--default-character-set
设置默认字符集，默认值为utf8
mysqldump  -uroot -p --all-databases --default-character-set=utf8
--delayed-insert

采用延时插入方式（INSERT DELAYED）导出数据
mysqldump  -uroot -p --all-databases --delayed-insert
--delete-master-logs
master备份后删除日志. 这个参数将自动激活--master-data。
mysqldump  -uroot -p --all-databases --delete-master-logs
--disable-keys
对于每个表，用/*!40000 ALTER TABLE tbl_name DISABLE KEYS */;和/*!40000 ALTER TABLE tbl_name ENABLE KEYS */;语句引用INSERT语句。这样可以更快地导入dump出来的文件，因为它是在插入所有行后创建索引的。该选项只适合MyISAM表，默认为打开状态。
mysqldump  -uroot -p --all-databases 
--dump-slave
该选项将主的binlog位置和文件名追加到导出数据的文件中(show slave status)。设置为1时，将会以CHANGE MASTER命令输出到数据文件；设置为2时，会在change前加上注释。该选项将会打开--lock-all-tables，除非--single-transaction被指定。该选项会自动关闭--lock-tables选项。默认值为0。
mysqldump  -uroot -p --all-databases --dump-slave=1
mysqldump  -uroot -p --all-databases --dump-slave=2

--master-data
该选项将当前服务器的binlog的位置和文件名追加到输出文件中(show master status)。如果为1，将会输出CHANGE MASTER 命令；如果为2，输出的CHANGE  MASTER命令前添加注释信息。该选项将打开--lock-all-tables 选项，除非--single-transaction也被指定（在这种情况下，全局读锁在开始导出时获得很短的时间；其他内容参考下面的--single-transaction选项）。该选项自动关闭--lock-tables选项。
mysqldump  -uroot -p --host=localhost --all-databases --master-data=1;
mysqldump  -uroot -p --host=localhost --all-databases --master-data=2;
--events, -E
导出事件。
mysqldump  -uroot -p --all-databases --events
--extended-insert,  -e
使用具有多个VALUES列的INSERT语法。这样使导出文件更小，并加速导入时的速度。默认为打开状态，使用--skip-extended-insert取消选项。
mysqldump  -uroot -p --all-databases
mysqldump  -uroot -p --all-databases--skip-extended-insert   (取消选项)
--fields-terminated-by
导出文件中忽略给定字段。与--tab选项一起使用，不能用于--databases和--all-databases选项
mysqldump  -uroot -p test test --tab=”/home/mysql” --fields-terminated-by=”#”
--fields-enclosed-by
输出文件中的各个字段用给定字符包裹。与--tab选项一起使用，不能用于--databases和--all-databases选项
mysqldump  -uroot -p test test --tab=”/home/mysql” --fields-enclosed-by=”#”
--fields-optionally-enclosed-by
输出文件中的各个字段用给定字符选择性包裹。与--tab选项一起使用，不能用于--databases和--all-databases选项
mysqldump  -uroot -p test test --tab=”/home/mysql”  --fields-enclosed-by=”#” --fields-optionally-enclosed-by  =”#”
--fields-escaped-by
输出文件中的各个字段忽略给定字符。与--tab选项一起使用，不能用于--databases和--all-databases选项
mysqldump  -uroot -p mysql user --tab=”/home/mysql” --fields-escaped-by=”#”
--flush-logs
开始导出之前刷新日志。
请注意：假如一次导出多个数据库(使用选项--databases或者--all-databases)，将会逐个数据库刷新日志。除使用--lock-all-tables或者--master-data外。在这种情况下，日志将会被刷新一次，相应的所以表同时被锁定。因此，如果打算同时导出和刷新日志应该使用--lock-all-tables 或者--master-data 和--flush-logs。
mysqldump  -uroot -p --all-databases --flush-logs
--flush-privileges
在导出mysql数据库之后，发出一条FLUSH  PRIVILEGES 语句。为了正确恢复，该选项应该用于导出mysql数据库和依赖mysql数据库数据的任何时候。
mysqldump  -uroot -p --all-databases --flush-privileges
--force 在导出过程中忽略出现的SQL错误。
mysqldump  -uroot -p --all-databases --force
--help 显示帮助信息并退出。
mysqldump  --help
--hex-blob 使用十六进制格式导出二进制字符串字段。如果有二进制数据就必须使用该选项。影响到的字段类型有BINARY、VARBINARY、BLOB。
mysqldump  -uroot -p --all-databases --hex-blob
--host, -h 需要导出的主机信息
mysqldump  -uroot -p --host=localhost --all-databases
--ignore-table 不导出指定表。指定忽略多个表时，需要重复多次，每次一个表。每个表必须同时指定数据库和表名。例如：--ignore-table=database.table1 --ignore-table=database.table2 ……
mysqldump  -uroot -p --host=localhost --all-databases --ignore-table=mysql.user
--include-master-host-port 在--dump-slave产生的'CHANGE  MASTER TO..'语句中增加'MASTER_HOST=<host>，MASTER_PORT=<port>'  
mysqldump  -uroot -p --host=localhost --all-databases --include-master-host-port
--insert-ignore 在插入行时使用INSERT IGNORE语句.
mysqldump  -uroot -p --host=localhost --all-databases --insert-ignore
--lines-terminated-by
输出文件的每行用给定字符串划分。与--tab选项一起使用，不能用于--databases和--all-databases选项。
mysqldump  -uroot -p --host=localhost test test --tab=”/tmp/mysql”  --lines-terminated-by=”##”
--lock-all-tables,  -x 提交请求锁定所有数据库中的所有表，以保证数据的一致性。这是一个全局读锁，并且自动关闭--single-transaction 和--lock-tables 选项。
mysqldump  -uroot -p --host=localhost --all-databases --lock-all-tables
--lock-tables,  -l 开始导出前，锁定所有表。用READ  LOCAL锁定表以允许MyISAM表并行插入。对于支持事务的表例如InnoDB和BDB，--single-transaction是一个更好的选择，因为它根本不需要锁定表。
请注意当导出多个数据库时，--lock-tables分别为每个数据库锁定表。因此，该选项不能保证导出文件中的表在数据库之间的逻辑一致性。不同数据库表的导出状态可以完全不同。
mysqldump  -uroot -p --host=localhost --all-databases --lock-tables
--log-error 附加警告和错误信息到给定文件
mysqldump  -uroot -p --host=localhost --all-databases  --log-error=/tmp/mysqldump_error_log.err
--max_allowed_packet 服务器发送和接受的最大包长度。
mysqldump  -uroot -p --host=localhost --all-databases --max_allowed_packet=10240
--net_buffer_length TCP/IP和socket连接的缓存大小。
mysqldump  -uroot -p --host=localhost --all-databases --net_buffer_length=1024
--no-autocommit 使用autocommit/commit 语句包裹表。
mysqldump  -uroot -p --host=localhost --all-databases --no-autocommit
--no-create-db,  -n 只导出数据，而不添加CREATE DATABASE 语句。
mysqldump  -uroot -p --host=localhost --all-databases --no-create-db
--no-create-info,  -t 只导出数据，而不添加CREATE TABLE 语句。
mysqldump  -uroot -p --host=localhost --all-databases --no-create-info
--no-data, -d 不导出任何数据，只导出数据库表结构。
mysqldump  -uroot -p --host=localhost --all-databases --no-data
--no-set-names,  -N
等同于--skip-set-charset
mysqldump  -uroot -p --host=localhost --all-databases --no-set-names
--opt
等同于--add-drop-table,  --add-locks, --create-options, --quick, --extended-insert, --lock-tables,  --set-charset, --disable-keys 该选项默认开启,  可以用--skip-opt禁用.
mysqldump  -uroot -p --host=localhost --all-databases --opt
--order-by-primary
如果存在主键，或者第一个唯一键，对每个表的记录进行排序。在导出MyISAM表到InnoDB表时有效，但会使得导出工作花费很长时间。 
mysqldump  -uroot -p --host=localhost --all-databases --order-by-primary
--password, -p 连接数据库密码
--pipe(windows系统可用) 使用命名管道连接mysql
mysqldump  -uroot -p --host=localhost --all-databases --pipe
--port, -P 连接数据库端口号
--protocol 使用的连接协议，包括：tcp, socket, pipe, memory.
mysqldump  -uroot -p --host=localhost --all-databases --protocol=tcp
--quick, -q
不缓冲查询，直接导出到标准输出。默认为打开状态，使用--skip-quick取消该选项。
mysqldump  -uroot -p --host=localhost --all-databases 
mysqldump  -uroot -p --host=localhost --all-databases --skip-quick
--quote-names,-Q
使用（`）引起表和列名。默认为打开状态，使用--skip-quote-names取消该选项。
mysqldump  -uroot -p --host=localhost --all-databases
mysqldump  -uroot -p --host=localhost --all-databases --skip-quote-names
--replace 使用REPLACE INTO 取代INSERT INTO.
mysqldump  -uroot -p --host=localhost --all-databases --replace
--result-file,  -r 直接输出到指定文件中。该选项应该用在使用回车换行对（\\r\\n）换行的系统上（例如：DOS，Windows）。该选项确保只有一行被使用。
mysqldump  -uroot -p --host=localhost --all-databases --result-file=/tmp/mysqldump_result_file.txt
--routines, -R 导出存储过程以及自定义函数。
mysqldump  -uroot -p --host=localhost --all-databases --routines
--set-charset 添加'SET NAMES  default_character_set'到输出文件。默认为打开状态，使用--skip-set-charset关闭选项。
mysqldump  -uroot -p --host=localhost --all-databases 
mysqldump  -uroot -p --host=localhost --all-databases --skip-set-charset
--single-transaction
该选项在导出数据之前提交一个BEGIN SQL语句，BEGIN 不会阻塞任何应用程序且能保证导出时数据库的一致性状态。它只适用于多版本存储引擎，仅InnoDB。本选项和--lock-tables 选项是互斥的，因为LOCK  TABLES 会使任何挂起的事务隐含提交。要想导出大表的话，应结合使用--quick 选项。
mysqldump  -uroot -p --host=localhost --all-databases --single-transaction
--dump-date 将导出时间添加到输出文件中。默认为打开状态，使用--skip-dump-date关闭选项。
mysqldump  -uroot -p --host=localhost --all-databases
mysqldump  -uroot -p --host=localhost --all-databases --skip-dump-date
--skip-opt 禁用–opt选项.
mysqldump  -uroot -p --host=localhost --all-databases --skip-opt
--socket,-S 指定连接mysql的socket文件位置，默认路径/tmp/mysql.sock
mysqldump  -uroot -p --host=localhost --all-databases --socket=/tmp/mysqld.sock
--tab,-T 为每个表在给定路径创建tab分割的文本文件。注意：仅仅用于mysqldump和mysqld服务器运行在相同机器上。注意使用--tab不能指定--databases参数
mysqldump  -uroot -p --host=localhost test test --tab="/home/mysql"
--tables
覆盖--databases (-B)参数，指定需要导出的表名，在后面的版本会使用table取代tables。
mysqldump  -uroot -p --host=localhost --databases test --tables test
--triggers 导出触发器。该选项默认启用，用--skip-triggers禁用它。
mysqldump  -uroot -p --host=localhost --all-databases --triggers
--tz-utc 在导出顶部设置时区TIME_ZONE='+00:00' ，以保证在不同时区导出的TIMESTAMP 数据或者数据被移动其他时区时的正确性。
mysqldump  -uroot -p --host=localhost --all-databases --tz-utc
--user, -u 指定连接的用户名。
--verbose, --v 输出多种平台信息。
--version, -V 输出mysqldump版本信息并退出
--where, -w
只转储给定的WHERE条件选择的记录。请注意如果条件包含命令解释符专用空格或字符，一定要将条件引用起来。
mysqldump  -uroot -p --host=localhost --all-databases --where=” user=’root’”
--xml, -X
导出XML格式.
mysqldump  -uroot -p --host=localhost --all-databases --xml
--plugin_dir 客户端插件的目录，用于兼容不同的插件版本。
mysqldump  -uroot -p --host=localhost --all-databases --plugin_dir=”/usr/local/lib/plugin”
--default_auth 客户端插件默认使用权限。
mysqldump  -uroot -p --host=localhost --all-databases --default-auth=”/usr/local/lib/plugin/<PLUGIN>”
```

#  