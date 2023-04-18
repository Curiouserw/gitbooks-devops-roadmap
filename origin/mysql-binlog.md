# MySQL Binlog

# 一、简介

MySQL Binlog（MySQL Binary Log，MySQL的二进制日志文件）,它记录了所有的 `DDL` 和 `DML` 语句（除了数据查询语句select、show等），**以事件形式记录**，还包含语句所执行的消耗的时间。MySQL的二进制日志是事务安全型的，并以二进制的形式保存在磁盘中；

**作用**：

- 查看数据库的变更历史
- 数据库增量备份和恢复
- MySQL的复制（主从数据库的复制）

# 二、Binglog

## 1、Binglog日志格式

### ①STATEMENT格式：基于SQL语句的复制

- 每一条会修改数据的sql都会记录在binlog中。
- 在 `MySQL 5.7.7` 之前，默认的格式是 `STATEMENT`

- **优点**：不需要记录每一行的变化，减少了binlog日志量，节约了IO，提高性能。

- **缺点**：由于记录的只是执行语句，为了这些语句能在slave上正确运行，因此还必须记录每条语句在执行的时候的一些相关信息，以保证所有语句能在slave得到和在master端执行时候相同 的结果。另外mysql 的复制,像一些特定函数功能，slave可与master上要保持一致会有很多相关问题。

- **注意**：相比row能节约多少性能与日志量，这个取决于应用的SQL情况，正常同一条记录修改或者插入row格式所产生的日志量还小于Statement产生的日志量，但是考虑到如果带条件的update操作，以及整表删除，alter表等操作，ROW格式会产生大量日志，因此在考虑是否使用ROW格式日志时应该跟据应用的实际情况，其所产生的日志量会增加多少，以及带来的IO性能问题。

### ②ROW格式：基于行的复制

- 5.1.5版本的MySQL才开始支持row level的复制,它不记录sql语句上下文相关信息，仅保存哪条记录被修改。
- 在 `MySQL 5.7.7` 及更高版本中，默认值是 `ROW`
- **优点**： binlog中可以不记录执行的sql语句的上下文相关的信息，仅需要记录那一条记录被修改成什么了。所以rowlevel的日志内容会非常清楚的记录下每一行数据修改的细节。而且不会出现某些特定情况下的存储过程，或function，以及trigger的调用和触发无法被正确复制的问题.
- **缺点**:所有的执行的语句当记录到日志中的时候，都将以每行记录的修改来记录，这样可能会产生大量的日志内容。
- **注意**:新版本的MySQL中对row level模式也被做了优化，并不是所有的修改都会以row level来记录，像遇到表结构变更的时候就会以statement模式来记录，如果sql语句确实就是update或者delete等修改数据的语句，那么还是会记录所有行的变更。

### ③MIXED格式：混合模式复制

- 从5.1.8版本开始，MySQL提供了Mixed格式，实际上就是Statement与Row的结合。

- 在Mixed模式下，一般的语句修改使用statment格式保存binlog，如一些函数，statement无法完成主从复制的操作，则采用row格式保存binlog，MySQL会根据执行的每一条具体的sql语句来区分对待记录的日志形式，也就是在Statement和Row之间选择一种。

## 2、Binlog日志文件

- 二进制日志索引文件（文件名后缀为.index）用于记录所有有效的的二进制文件
- 二进制日志文件（文件名后缀为.00000*）记录数据库所有的DDL和DML语句事件



binlog是一个二进制文件集合，每个binlog文件以一个4字节的魔数开头，接着是一组Events:

- 魔数：0xfe62696e对应的是0xfebin；
- Event：每个Event包含header和data两个部分；header提供了Event的创建时间，哪个服务器等信息，data部分提供的是针对该Event的具体信息，如具体数据的修改；
- 第一个Event用于描述binlog文件的格式版本，这个格式就是event写入binlog文件的格式；
- 其余的Event按照第一个Event的格式版本写入；
- 最后一个Event用于说明下一个binlog文件；
- binlog的索引文件是一个文本文件，其中内容为当前的binlog文件列表

## 3、Binlog事件类型

binlog 事件的结构主要有3个版本：

| Binlog 版本 | MySQL版本             |                                                              |
| :---------- | :-------------------- | ------------------------------------------------------------ |
| 1           | MySQL 3.23 - < 4.0.0  | 支持 "statement based replication events"                    |
| 2           | MySQL 4.0.0 - 4.0.1   |                                                              |
| 3           | MySQL 4.0.2 - < 5.0.0 | added the relay logs and changed the meaning of the log position |
| 4           | MySQL 5.0.0+          | added the [FORMAT_DESCRIPTION_EVENT](https://dev.mysql.com/doc/internals/en/format-description-event.html) and made the protocol extensible |

- a [FORMAT_DESCRIPTION_EVENT](https://dev.mysql.com/doc/internals/en/format-description-event.html) version = 4
- a [START_EVENT_V3](https://dev.mysql.com/doc/internals/en/start-event-v3.html)
  - if `event-size` == 13 + 56: version = 1
  - if `event-size` == 19 + 56: version = 3
  - otherwise: invalid

v4版本的binlog事件类型：

| Hex| 事件类型                 | 说明                                                         |
| :----------------------- | :----------------------------------------------------------- | :----------------------- |
| 0x00 | UNKNOWN_EVENT            | 此事件从不会被触发，也不会被写入binlog中；发生在当读取binlog时，不能被识别其他任何事件，那被视为UNKNOWN_EVENT |
| 0x01 | START_EVENT_V3           | 每个binlog文件开始的时候写入的事件，此事件被用在MySQL3.23 – 4.1，MYSQL5.0以后已经被 FORMAT_DESCRIPTION_EVENT 取代 |
| 0x02 | QUERY_EVENT              | 执行更新语句时会生成此事件，包括：create，insert，update，delete； |
| 0x03 | STOP_EVENT               | 当mysqld停止时生成此事件                                     |
| 0x04 | ROTATE_EVENT             | 当mysqld切换到新的binlog文件生成此事件，切换到新的binlog文件可以通过执行flush logs命令或者binlog文件大于 `max_binlog_size` 参数配置的大小； |
| 0x05 | INTVAR_EVENT             | 当sql语句中使用了AUTO_INCREMENT的字段或者LAST_INSERT_ID()函数；此事件没有被用在binlog_format为ROW模式的情况下 |
| 0x06 | LOAD_EVENT               | 执行LOAD DATA INFILE 语句时产生此事件，在MySQL 3.23版本中使用 |
| 0x07 | SLAVE_EVENT              | 未使用                                                       |
| 0x08 | CREATE_FILE_EVENT        | 执行LOAD DATA INFILE 语句时产生此事件，在MySQL4.0和4.1版本中使用 |
| 0x09 | APPEND_BLOCK_EVENT       | 执行LOAD DATA INFILE 语句时产生此事件，在MySQL4.0版本中使用  |
| 0x0a | EXEC_LOAD_EVENT          | 执行LOAD DATA INFILE 语句时产生此事件，在MySQL4.0和4.1版本中使用 |
| 0x0b | DELETE_FILE_EVENT        | 执行LOAD DATA INFILE 语句时产生此事件，在MySQL4.0版本中使用  |
| 0x0c | NEW_LOAD_EVENT           | 执行LOAD DATA INFILE 语句时产生此事件，在MySQL4.0和4.1版本中使用 |
| 0x0d | RAND_EVENT               | 执行包含RAND()函数的语句产生此事件，此事件没有被用在binlog_format为ROW模式的情况下 |
| 0x0e | USER_VAR_EVENT           | 执行包含了用户变量的语句产生此事件，此事件没有被用在binlog_format为ROW模式的情况下 |
| 0x0f | FORMAT_DESCRIPTION_EVENT | 描述事件，被写在每个binlog文件的开始位置，用在MySQL5.0以后的版本中，代替了START_EVENT_V3 |
| 0x10 | XID_EVENT                | 支持XA的存储引擎才有，本地测试的数据库存储引擎是innodb，所有上面出现了XID_EVENT；innodb事务提交产生了QUERY_EVENT的BEGIN声明，QUERY_EVENT以及COMMIT声明，如果是myIsam存储引擎也会有BEGIN和COMMIT声明，只是COMMIT类型不是XID_EVENT |
| 0x11 | BEGIN_LOAD_QUERY_EVENT   | 执行LOAD DATA INFILE 语句时产生此事件，在MySQL5.0版本中使用  |
| 0x12 | EXECUTE_LOAD_QUERY_EVENT | 执行LOAD DATA INFILE 语句时产生此事件，在MySQL5.0版本中使用  |
| 0x13 | TABLE_MAP_EVENT          | 用在binlog_format为ROW模式下，将表的定义映射到一个数字，在行操作事件之前记录（包括：WRITE_ROWS_EVENT，UPDATE_ROWS_EVENT，DELETE_ROWS_EVENT） |
|  | PRE_GA_WRITE_ROWS_EVENT  | 已过期，被 WRITE_ROWS_EVENT 代替                             |
|  | PRE_GA_UPDATE_ROWS_EVENT | 已过期，被 UPDATE_ROWS_EVENT 代替                            |
|  | PRE_GA_DELETE_ROWS_EVENT | 已过期，被 DELETE_ROWS_EVENT 代替                            |
| 0x14 | WRITE_ROWS_EVENT         | 用在binlog_format为ROW模式下，对应 insert 操作               |
| 0x15 | UPDATE_ROWS_EVENT        | 用在binlog_format为ROW模式下，对应 update 操作               |
| 0x16 | DELETE_ROWS_EVENT        | 用在binlog_format为ROW模式下，对应 delete 操作               |
| 0x1a | INCIDENT_EVENT           | 主服务器发生了不正常的事件，通知从服务器并告知可能会导致数据处于不一致的状态 |
| 0x1b | HEARTBEAT_LOG_EVENT      | 主服务器告诉从服务器，主服务器还活着，不写入到日志文件中     |

## 4、Binlog事件结构

一个事件对象分为事件头和事件体，事件的结构如下：

```
+=====================================+
| event  | timestamp         0 : 4    |
| header +----------------------------+
|        | type_code         4 : 1    |
|        +----------------------------+
|        | server_id         5 : 4    |
|        +----------------------------+
|        | event_length      9 : 4    |
|        +----------------------------+
|        | next_position    13 : 4    |
|        +----------------------------+
|        | flags            17 : 2    |
|        +----------------------------+
|        | extra_headers    19 : x-19 |
+=====================================+
| event  | fixed part        x : y    |
| data   +----------------------------+
|        | variable part              |
+=====================================+
```

如果事件头的长度是 `x` 字节，那么事件体的长度为 `(event_length - x)` 字节；设事件体中 `fixed part` 的长度为 `y` 字节，那么 `variable part` 的长度为 `(event_length - (x + y))` 字节

# 三、配置Binlog

## 1、配置

在`/etc/my.cnf`的`[mysqld]`部分添加以下配置，然后重启MySQL

```bash
[mysqld]
log-bin=/data/mysql/logs/binlogs/mysql-bin.log
expire-logs-days=14
max-binlog-size=500M
server-id=1
binlog_format=ROW
binlog_row_image=FULL
relay_log_info_repository=TABLE
```

## 2、验证

```bash
show variables like '%binlog%'
# 检查MySQL是否已经开启binlog
show variables like 'log_bin'

# 查看binlog文件列表及大小
show binary logs

# 查看binlog内容
show binlog events

# 查看当前最新一个binlog日志文件的状态信息，显示正在写入的二进制文件，及当前position
show master status;

#查看所有binlog日志列表
show master logs;
```

## 3、注意

当遇到以下3种情况时，MySQL会重新生成一个新的日志文件，文件序号递增：

- MySQL服务器停止或重启时
- 使用 `flush logs` 命令；
- 当 binlog 文件大小超过 `max_binlog_size` 变量的值时；

> `max_binlog_size` 的最小值是4096字节，最大值和默认值是 1GB (1073741824字节)。事务被写入到binlog的一个块中，所以它不会在几个二进制日志之间被拆分。因此，如果你有很大的事务，为了保证事务的完整性，不可能做切换日志的动作，只能将该事务的日志都记录到当前日志文件中，直到事务结束，你可能会看到binlog文件大于 max_binlog_size 的情况。

# 四、使用mysqlbinlog命令查看Binlog

因为binlog日志文件是二进制文件`，没法用vi等打开，这时就`需要mysql的自带的mysqlbinlog工具进行解码

```
mysqlbinlog [参数] binlog文件路径

参数：
  -?, --help          Display this help and exit.
  --base64-output=name
                      Determine when the output statements should be
                      base64-encoded BINLOG statements: 'never' disables it and
                      works only for binlogs without row-based events;
                      'decode-rows' decodes row events into commented
                      pseudo-SQL statements if the --verbose option is also
                      given; 'auto' prints base64 only when necessary (i.e.,
                      for row-based events and format description events).  If
                      no --base64-output[=name] option is given at all, the
                      default is 'auto'.
  --bind-address=name IP address to bind to.
  --character-sets-dir=name
                      Directory for character set files.
  -d, --database=name List entries for just this database (local log only).
  --debug-check       Check memory and open file usage at exit .
  --debug-info        Print some debug info at exit.
  --default-auth=name Default authentication client-side plugin to use.
  -D, --disable-log-bin
                      Disable binary log. This is useful, if you enabled
                      --to-last-log and are sending the output to the same
                      MySQL server. This way you could avoid an endless loop.
                      You would also like to use it when restoring after a
                      crash to avoid duplication of the statements you already
                      have. NOTE: you will need a SUPER privilege to use this
                      option.
  -F, --force-if-open Force if binlog was not closed properly.
                      (Defaults to on; use --skip-force-if-open to disable.)
  -f, --force-read    Force reading unknown binlog events.
  -H, --hexdump       Augment output with hexadecimal and ASCII event dump.
  -h, --host=name     Get the binlog from server.
  -l, --local-load=name
                      Prepare local temporary files for LOAD DATA INFILE in the
                      specified directory.
  -o, --offset=#      Skip the first N entries.
  -p, --password[=name]
                      Password to connect to remote server.
  --plugin-dir=name   Directory for client-side plugins.
  -P, --port=#        Port number to use for connection or 0 for default to, in
                      order of preference, my.cnf, $MYSQL_TCP_PORT,
                      /etc/services, built-in default (3306).
  --protocol=name     The protocol to use for connection (tcp, socket, pipe,
                      memory).
  -R, --read-from-remote-server
                      Read binary logs from a MySQL server. This is an alias
                      for read-from-remote-master=BINLOG-DUMP-NON-GTIDS.
  --read-from-remote-master=name
                      Read binary logs from a MySQL server through the
                      COM_BINLOG_DUMP or COM_BINLOG_DUMP_GTID commands by
                      setting the option to either BINLOG-DUMP-NON-GTIDS or
                      BINLOG-DUMP-GTIDS, respectively. If
                      --read-from-remote-master=BINLOG-DUMP-GTIDS is combined
                      with --exclude-gtids, transactions can be filtered out on
                      the master avoiding unnecessary network traffic.
  --raw               Requires -R. Output raw binlog data instead of SQL
                      statements, output is to log files.
  -r, --result-file=name
                      Direct output to a given file. With --raw this is a
                      prefix for the file names.
  --secure-auth       Refuse client connecting to server if it uses old
                      (pre-4.1.1) protocol.
                      (Defaults to on; use --skip-secure-auth to disable.)
  --server-id=#       Extract only binlog entries created by the server having
                      the given id.
  --server-id-bits=#  Set number of significant bits in server-id
  --set-charset=name  Add 'SET NAMES character_set' to the output.
  -s, --short-form    Just show regular queries: no extra info and no row-based
                      events. This is for testing only, and should not be used
                      in production systems. If you want to suppress
                      base64-output, consider using --base64-output=never
                      instead.
  -S, --socket=name   The socket file to use for connection.
  --start-datetime=name 
                      Start reading the binlog at first event having a datetime
                      equal or posterior to the argument; the argument must be
                      a date and time in the local time zone, in any format
                      accepted by the MySQL server for DATETIME and TIMESTAMP
                      types, for example: 2004-12-25 11:25:56 (you should
                      probably use quotes for your shell to set it properly).
  -j, --start-position=#
                      Start reading the binlog at position N. Applies to the
                      first binlog passed on the command line.
  --stop-datetime=name
                      Stop reading the binlog at first event having a datetime
                      equal or posterior to the argument; the argument must be
                      a date and time in the local time zone, in any format
                      accepted by the MySQL server for DATETIME and TIMESTAMP
                      types, for example: 2004-12-25 11:25:56 (you should
                      probably use quotes for your shell to set it properly).
  --stop-never        Wait for more data from the server instead of stopping at
                      the end of the last log. Implicitly sets --to-last-log
                      but instead of stopping at the end of the last log it
                      continues to wait till the server disconnects.
  --stop-never-slave-server-id=#
                      The slave server_id used for --read-from-remote-server
                      --stop-never.
  --stop-position=#   Stop reading the binlog at position N. Applies to the
                      last binlog passed on the command line.
  -t, --to-last-log   Requires -R. Will not stop at the end of the requested
                      binlog but rather continue printing until the end of the
                      last binlog of the MySQL server. If you send the output
                      to the same MySQL server, that may lead to an endless
                      loop.

  -u, --user=name  连接远程服务器的用户名。
  -v, --verbose       Reconstruct pseudo-SQL statements out of row events. -v
                      -v adds comments on column data types.
  -V, --version       打印mysqlbinlog的版本信息并退出
  --open-files-limit=#
                      Used to reserve file descriptors for use by this program.
  -c, --verify-binlog-checksum
                      Verify checksum binlog events.
  --binlog-row-event-max-size=#
                      The maximum size of a row-based binary log event in
                      bytes. Rows will be grouped into events smaller than this
                      size if possible. 改值必须是256的倍数。
                      指定基于行的binlog的大小，
  --skip-gtids        Do not print Global Transaction Identifier information
                      (SET GTID_NEXT=... etc).
  --include-gtids=name
                      Print events whose Global Transaction Identifiers were
                      provided.
  --exclude-gtids=name
                      Print all events but those whose Global Transaction
                      Identifiers were provided.

Variables (--variable-name=value)
and boolean options {FALSE|TRUE}  Value (after reading options)
--------------------------------- ----------------------------------------
base64-output                     (No default value)
bind-address                      (No default value)
character-sets-dir                (No default value)
database                          (No default value)
debug-check                       FALSE
debug-info                        FALSE
default-auth                      (No default value)
disable-log-bin                   FALSE
force-if-open                     TRUE
force-read                        FALSE
hexdump                           FALSE
host                              (No default value)
local-load                        (No default value)
offset                            0
plugin-dir                        (No default value)
port                              0
read-from-remote-server           FALSE
read-from-remote-master           (No default value)
raw                               FALSE
result-file                       (No default value)
secure-auth                       TRUE
server-id                         0
server-id-bits                    32
set-charset                       (No default value)
short-form                        FALSE
socket                            /data/mysql/data/mysqld.sock
start-datetime                    (No default value)
start-position                    4
stop-datetime                     (No default value)
stop-never                        FALSE
stop-never-slave-server-id        -1
stop-position                     18446744073709551615
to-last-log                       FALSE
user                              (No default value)
open-files-limit                  64
verify-binlog-checksum            FALSE
binlog-row-event-max-size         4294967040
skip-gtids                        FALSE
include-gtids                     (No default value)
exclude-gtids                     (No default value)

```

# 五、使用SQL语句查看Binlog

```sql
show binlog events [IN 'log_name'] [FROM pos] [LIMIT [offset,] row_count];
-- IN 'log_name' ：指定要查询的binlog文件名(不指定就是第一个binlog文件)
-- FROM pos ：指定从哪个pos起始点开始查起(不指定就是从整个文件首个pos点开始算)
-- LIMIT [offset,] ：偏移量(不指定就是0)
-- row_count ：查询总条数(不指定就是所有行)
```



# 参考：

1. https://blog.csdn.net/ouyang111222/article/details/50300851