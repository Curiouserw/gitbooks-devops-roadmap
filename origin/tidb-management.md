# TiDB管理

# 一、系统表

`mysql` 库里存储的是 TiDB 系统表

## 1、权限系统表

- **user**：用户账户，全局权限

  - `user` 表的权限是全局的，并且不管默认数据库是哪一个。比如 `user` 里面有 `DELETE` 权限，任何一行，任何的表，任何的数据库。
  - `User+Host` 可能会匹配 `user` 表里面多行，为了处理这种情况，`user` 表的行是排序过的，客户端连接时会依次去匹配，并使用首次匹配到的那一行做权限验证。排序是按 `Host` 在前，`User` 在后。

- **db**：数据库级别的权限

  `db`表里面，User 为空是匹配匿名用户，User 里面不能有通配符。Host 和 Db 列里面可以有 `%` 和 `_`，可以模式匹配。

- **tables_priv**：表级别的权限

  `tables_priv` 和 `columns_priv` 中使用 `%` 是类似的，但是在`Db`, `Table_name`, `Column_name` 这些列不能包含 `%`。加载进来时排序也是类似的。

- **columns_priv**：列级别的权限，当前暂不支持

**权限生效时机**

- TiDB 启动时，将一些权限检查的表加载到内存，之后使用缓存的数据来验证权限。系统会周期性的将授权表从数据库同步到缓存，生效则是由同步的周期决定，目前这个值设定的是 5 分钟。
- 如果授权表已被直接修改，则不会通知 TiDB 节点更新缓存，如果需要立即生效，可以手动执行`FLUSH PRIVILEGES;`

## 2、服务端帮助信息系统表

- `help_topic` 目前为空

## 3、统计信息相关系统表

- `stats_buckets` 统计信息的桶
- `stats_histograms` 统计信息的直方图
- `stats_meta` 表的元信息，比如总行数和修改数
- `analyze_jobs` 正在执行的统计信息收集任务以及过去 7 天内的历史任务记录

## 4、GC Worker 相关系统表

- `gc_delete_range`

##  5、其它系统表

- `GLOBAL_VARIABLES` 全局系统变量表
- `tidb` 用于 TiDB 在 bootstrap 的时候记录相关版本信息

# 二、用户管理

## 1、创建普通用户

创建用户

```bash
CREATE USER `username`@`192.168.%.%` IDENTIFIED BY '****';
GRANT Select ON `username`.* TO `username`@`192.168.%.%`;
```

删除用户

```bash
DROP USER 'username'@'192.168.%.%'
```

## 2、创建Dashboard访问用户

Dashboard访问用户所需权限

- 当所连接的 TiDB 服务器未启用[安全增强模式 (SEM)](https://docs.pingcap.com/zh/tidb/dev/system-variables#tidb_enable_enhanced_security) 时，要访问 TiDB Dashboard，SQL 用户应当拥有以下**所有**权限：

  - PROCESS
  - SHOW DATABASES
  - CONFIG
  - DASHBOARD_CLIENT
  - 无IP限制，创建时使用%不限制用户登录IP

  ```sql
  CREATE USER 'dashboardAdmin'@'%' IDENTIFIED BY '<YOUR_PASSWORD>';
  GRANT PROCESS, CONFIG ON *.* TO 'dashboardAdmin'@'%';
  GRANT SHOW DATABASES ON *.* TO 'dashboardAdmin'@'%';
  GRANT DASHBOARD_CLIENT ON *.* TO 'dashboardAdmin'@'%';
  ```

- 当所连接的 TiDB 服务器启用了[安全增强模式 (SEM)](https://docs.pingcap.com/zh/tidb/dev/system-variables#tidb_enable_enhanced_security) 时，要访问 TiDB Dashboard，SQL 用户应当拥有以下**所有**权限：

  - PROCESS
  - SHOW DATABASES
  - CONFIG
  - DASHBOARD_CLIENT
  - RESTRICTED_TABLES_ADMIN
  - RESTRICTED_STATUS_ADMIN
  - RESTRICTED_VARIABLES_ADMIN
  - 无IP限制，创建时使用%不限制用户登录IP

  ```bash
  CREATE USER 'dashboardAdmin'@'%' IDENTIFIED BY '<YOUR_PASSWORD>';
  GRANT PROCESS, CONFIG ON *.* TO 'dashboardAdmin'@'%';
  GRANT SHOW DATABASES ON *.* TO 'dashboardAdmin'@'%';
  GRANT DASHBOARD_CLIENT ON *.* TO 'dashboardAdmin'@'%';
  GRANT RESTRICTED_STATUS_ADMIN ON *.* TO 'dashboardAdmin'@'%';
  GRANT RESTRICTED_TABLES_ADMIN ON *.* TO 'dashboardAdmin'@'%';
  GRANT RESTRICTED_VARIABLES_ADMIN ON *.* TO 'dashboardAdmin'@'%';
  ```

- 若希望 SQL 用户在登录 TiDB Dashboard 后允许修改界面上的各项配置，SQL 用户还应当拥有以下权限：

  - SYSTEM_VARIABLES_ADMIN
  
  ```sql
  -- 如果要使自定义的 SQL 用户能修改 TiDB Dashboard 界面上的各项配置，可以增加以下权限
  GRANT SYSTEM_VARIABLES_ADMIN ON *.* TO 'dashboardAdmin'@'%';
  ```

# 三、慢SQL

TiDB 会将执行时间超过 [slow-threshold](https://docs.pingcap.com/zh/tidb/v5.1/tidb-configuration-file#slow-threshold)（默认值为 300 毫秒）的语句输出到 [slow-query-file](https://docs.pingcap.com/zh/tidb/v5.1/tidb-configuration-file#slow-query-file)（默认值："tidb-slow.log"）日志文件中，用于帮助用户定位慢查询语句，分析和解决 SQL 执行的性能问题。

TiDB 默认启用慢查询日志，可以修改配置 [`enable-slow-log`](https://docs.pingcap.com/zh/tidb/v5.1/tidb-configuration-file#enable-slow-log) 来启用或禁用它。

## Slow Query 基础信息

- `Time`：表示日志打印时间。
- `Query_time`：表示执行这个语句花费的时间。
- `Parse_time`：表示这个语句在语法解析阶段花费的时间。
- `Compile_time`：表示这个语句在查询优化阶段花费的时间。
- `Query`：表示 SQL 语句。慢日志里面不会打印 `Query`，但映射到内存表后，对应的字段叫 `Query`。
- `Digest`：表示 SQL 语句的指纹。
- `Txn_start_ts`：表示事务的开始时间戳，也是事务的唯一 ID，可以用这个值在 TiDB 日志中查找事务相关的其他日志。
- `Is_internal`：表示是否为 TiDB 内部的 SQL 语句。`true` 表示 TiDB 系统内部执行的 SQL 语句，`false` 表示用户执行的 SQL 语句。
- `Index_ids`：表示语句涉及到的索引的 ID。
- `Succ`：表示语句是否执行成功。
- `Backoff_time`：表示语句遇到需要重试的错误时在重试前等待的时间，常见的需要重试的错误有以下几种：遇到了 lock、Region 分裂、`tikv server is busy`。
- `Plan`：表示语句的执行计划，用 `select tidb_decode_plan('xxx...')` SQL 语句可以解析出具体的执行计划。
- `Prepared`：表示这个语句是否是 `Prepare` 或 `Execute` 的请求。
- `Plan_from_cache`：表示这个语句是否命中了执行计划缓存。
- `Rewrite_time`：表示这个语句在查询改写阶段花费的时间。
- `Preproc_subqueries`：表示这个语句中被提前执行的子查询个数，如 `where id in (select if from t)` 这个子查询就可能被提前执行。
- `Preproc_subqueries_time`：表示这个语句中被提前执行的子查询耗时。
- `Exec_retry_count`：表示这个语句执行的重试次数。一般出现在悲观事务中，上锁失败时重试执行该语句。
- `Exec_retry_time`：表示这个语句的重试执行时间。例如某个查询一共执行了三次（前两次失败），则 `Exec_retry_time` 表示前两次的执行时间之和，`Query_time` 减去 `Exec_retry_time` 则为最后一次执行时间。

## 和事务执行相关的字段

- `Prewrite_time`：表示事务两阶段提交中第一阶段（prewrite 阶段）的耗时。
- `Commit_time`：表示事务两阶段提交中第二阶段（commit 阶段）的耗时。
- `Get_commit_ts_time`：表示事务两阶段提交中第二阶段（commit 阶段）获取 commit 时间戳的耗时。
- `Local_latch_wait_time`：表示事务两阶段提交中第二阶段（commit 阶段）发起前在 TiDB 侧等锁的耗时。
- `Write_keys`：表示该事务向 TiKV 的 Write CF 写入 Key 的数量。
- `Write_size`：表示事务提交时写 key 或 value 的总大小。
- `Prewrite_region`：表示事务两阶段提交中第一阶段（prewrite 阶段）涉及的 TiKV Region 数量。每个 Region 会触发一次远程过程调用。

## 和内存使用相关的字段

- `Mem_max`：表示执行期间 TiDB 使用的最大内存空间，单位为 byte。

## 和硬盘使用相关的字段

- `Disk_max`: 表示执行期间 TiDB 使用的最大硬盘空间，单位为 byte。

## 和 SQL 执行的用户相关的字段

- `User`：表示执行语句的用户名。
- `Conn_ID`：表示用户的链接 ID，可以用类似 `con:3` 的关键字在 TiDB 日志中查找该链接相关的其他日志。
- `DB`：表示执行语句时使用的 database。

## 和 TiKV Coprocessor Task 相关的字段

- `Request_count`：表示这个语句发送的 Coprocessor 请求的数量。
- `Total_keys`：表示 Coprocessor 扫过的 key 的数量。
- `Process_time`：执行 SQL 在 TiKV 的处理时间之和，因为数据会并行的发到 TiKV 执行，这个值可能会超过 `Query_time`。
- `Wait_time`：表示这个语句在 TiKV 的等待时间之和，因为 TiKV 的 Coprocessor 线程数是有限的，当所有的 Coprocessor 线程都在工作的时候，请求会排队；当队列中有某些请求耗时很长的时候，后面的请求的等待时间都会增加。
- `Process_keys`：表示 Coprocessor 处理的 key 的数量。相比 total_keys，processed_keys 不包含 MVCC 的旧版本。如果 processed_keys 和 total_keys 相差很大，说明旧版本比较多。
- `Cop_proc_avg`：cop-task 的平均执行时间。
- `Cop_proc_p90`：cop-task 的 P90 分位执行时间。
- `Cop_proc_max`：cop-task 的最大执行时间。
- `Cop_proc_addr`：执行时间最长的 cop-task 所在地址。
- `Cop_wait_avg`：cop-task 的平均等待时间。
- `Cop_wait_p90`：cop-task 的 P90 分位等待时间。
- `Cop_wait_max`：cop-task 的最大等待时间。
- `Cop_wait_addr`：等待时间最长的 cop-task 所在地址。
- `Cop_backoff_{backoff-type}_total_times`：因某种错误造成的 backoff 总次数。
- `Cop_backoff_{backoff-type}_total_time`：因某种错误造成的 backoff 总时间。
- `Cop_backoff_{backoff-type}_max_time`：因某种错误造成的最大 backoff 时间。
- `Cop_backoff_{backoff-type}_max_addr`：因某种错误造成的最大 backoff 时间的 cop-task 地址。
- `Cop_backoff_{backoff-type}_avg_time`：因某种错误造成的平均 backoff 时间。
- `Cop_backoff_{backoff-type}_p90_time`：因某种错误造成的 P90 分位 backoff 时间。

## 相关系统变量

- [tidb_slow_log_threshold](https://docs.pingcap.com/zh/tidb/v5.1/system-variables#tidb_slow_log_threshold)：设置慢日志的阈值，执行时间超过阈值的 SQL 语句将被记录到慢日志中。默认值是 300 ms。
- [tidb_query_log_max_len](https://docs.pingcap.com/zh/tidb/v5.1/system-variables#tidb_query_log_max_len)：设置慢日志记录 SQL 语句的最大长度。默认值是 4096 byte。
- [tidb_redact_log](https://docs.pingcap.com/zh/tidb/v5.1/system-variables#tidb_redact_log)：设置慢日志记录 SQL 时是否将用户数据脱敏用 `?` 代替。默认值是 0 ，即关闭该功能。
- [tidb_enable_collect_execution_info](https://docs.pingcap.com/zh/tidb/v5.1/system-variables#tidb_enable_collect_execution_info)：设置是否记录执行计划中各个算子的物理执行信息，默认值是 `1`。该功能对性能的影响约为 3%。

## 统计慢SQL

```sql
SELECT
	DB AS '数据库',
	`Query` AS 'SQL语句',
	Query_time AS '执行耗时Query_time',
	Time AS '执行时间',
	`User` AS '执行用户',
	`Host` AS '主机',
	Total_keys AS '扫描总Key个数Total_keys',
	Index_names AS '涉及索引名'
FROM
	INFORMATION_SCHEMA.SLOW_QUERY 
WHERE
	DB = '数据库名' 
ORDER BY
	Query_time DESC 
	LIMIT 100;
```

# 四、TIDB集群安全相关

## 1、日志脱敏

### ①TiDB 组件日志脱敏

### ②TiKV 组件日志脱敏

### ③PD 组件日志脱敏

### ④TiFlash 组件日志脱敏

https://docs.pingcap.com/zh/tidb/stable/log-redaction

## 2、关闭发送遥测数据

### ①TiDB禁用遥测

- **动态禁用**

  ```sql
  # 修改系统全局变量tidb_enable_telemetry动态禁用 TiDB 遥测功能：
  SET GLOBAL tidb_enable_telemetry = 0;
  ```

  配置文件的禁用优先级高于全局变量。若通过配置文件禁用了遥测功能，则全局变量的配置将不起作用，遥测功能总是处于关闭状态。

- **集群级别禁用**

  修改集群配置文件

  ```bash
  server_configs:
    tidb:
      enable-telemetry: false
  ```

  重启集群TiDB组件

### ②TiDB Dashboard禁用遥测

修改集群配置文件

```bash
server_configs:
  pd:
    dashboard.enable-telemetry: false
```

重启集群PD组件

### ③TiUP禁用遥测

```bash
tiup telemetry disable
# 禁用之后~/.tiup/telemetry/meta.yaml文件中的status字段会显示Disable
```

### ④TiSpark禁用遥测

在 Spark 配置文件设置 `spark.tispark.telemetry.enable = false` 来禁用 TiSpark 的遥测功能。

https://docs.pingcap.com/zh/tidb/stable/telemetry

# 五、下载的性能测试数据展示

在TIDB Dashboard中做的性能测试，数组只保存几天。可下载下来进行长期保存。如果要再次以Web形式展示，使用以下命令

```bash
go tool pprof --http=0.0.0.0:1234 cpu_xxx.proto
```

# 六、Kill进程

语法：**`KILL TIDB? ( 'CONNECTION' | 'QUERY' )? CONNECTION_ID`**

```sql
SELECT ID, USER, INSTANCE, INFO FROM INFORMATION_SCHEMA.CLUSTER_PROCESSLIST;

+---------------------+------+-----------------+-----------------------------------------------------------------------------+
| ID                  | USER | INSTANCE        | INFO                                                                        |
+---------------------+------+-----------------+-----------------------------------------------------------------------------+
| 8306449708033769879 | root | 127.0.0.1:10082 | select sleep(30), 'foo'                                                     |
| 5857102839209263511 | root | 127.0.0.1:10080 | select sleep(50)                                                            |
| 5857102839209263513 | root | 127.0.0.1:10080 | SELECT ID, USER, INSTANCE, INFO FROM INFORMATION_SCHEMA.CLUSTER_PROCESSLIST |
+---------------------+------+-----------------+-----------------------------------------------------------------------------+

KILL 5857102839209263513;
```

**MySQL 兼容性**

- MySQL 的 `KILL` 语句仅能终止当前连接的 MySQL 实例上的连接，TiDB 的 `KILL` 语句能终止整个集群中任意一个 TiDB 实例上的连接。
- 暂时不支持使用 MySQL 命令行 ctrl+c 终止查询或连接。

**行为变更说明**

TiDB 从 v6.1.0 起新增 Global Kill 功能（由 [`enable-global-kill`](https://docs.pingcap.com/zh/tidb/stable/tidb-configuration-file#enable-global-kill-从-v610-版本开始引入) 配置项控制，默认启用）。启用 Global Kill 功能时，`KILL` 语句和 `KILL TIDB` 语句均能跨节点终止查询或连接，且无需担心错误地终止其他查询或连接。当你使用客户端连接到任何一个 TiDB 节点执行 `KILL` 语句或 `KILL TIDB` 语句时，该语句会被转发给对应的 TiDB 节点。当客户端和 TiDB 中间有代理时，`KILL` 及 `KILL TIDB` 语句也会被转发给对应的 TiDB 节点执行。

对于 TiDB v6.1.0 之前的版本，或未启用 Global Kill 功能时：

- `KILL` 语句与 MySQL 不兼容，负载均衡器后面通常放有多个 TiDB 服务器，这种不兼容有助于防止在错误的 TiDB 服务器上终止连接。你需要显式地增加 `TIDB` 后缀，通过执行 `KILL TIDB` 语句来终止当前连接的 TiDB 实例上的其他连接。
- **强烈不建议**在配置文件里设置 [`compatible-kill-query = true`](https://docs.pingcap.com/zh/tidb/stable/tidb-configuration-file#compatible-kill-query)，**除非**你确定客户端将始终连接到同一个 TiDB 节点。这是因为当你在默认的 MySQL 客户端按下 ctrl+c 时，客户端会开启一个新连接，并在这个新连接中执行 `KILL` 语句。此时，如果客户端和 TiDB 中间有代理，新连接可能会被路由到其他的 TiDB 节点，从而错误地终止其他会话。
- `KILL TIDB` 语句是 TiDB 的扩展语法，其功能与 MySQL 命令 `KILL [CONNECTION|QUERY]` 和 MySQL 命令行 ctrl+c 相同。在同一个 TiDB 节点上，你可以安全地使用 `KILL TIDB` 语句。

https://docs.pingcap.com/zh/tidb/stable/sql-statement-kill#kill