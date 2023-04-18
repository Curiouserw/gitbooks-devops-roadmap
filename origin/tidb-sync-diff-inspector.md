# TiDB数据校验工具：sync-diff-inspector

# 一、简介

## 1、简介

sync-diff-inspector 是一个用于校验 MySQL／TiDB 中两份数据是否一致的工具。该工具提供了修复数据的功能（适用于修复少量不一致的数据）。

## 2、**主要功能**

- 对比表结构和数据
- 如果数据不一致，则生成用于修复数据的 SQL 语句
- 支持不同库名或表名的数据校验
- 支持分库分表场景下的数据校验
- 支持 TiDB 主从集群的数据校验
- 支持从 TiDB DM 拉取配置的数据校验

## <font color=red>3、使用限制</font>

- 对于 MySQL 和 TiDB 之间的数据同步不支持在线校验，需要保证上下游校验的表中没有数据写入，或者保证某个范围内的数据不再变更，通过配置 range 来校验这个范围内的数据

- FLOAT、DOUBLE 等浮点数类型在 TiDB 和 MySQL 中的实现方式不同，在计算 checksum 时会分别取 6 位和 15 位有效数字。如果不使用该特性，需要设置 ignore-columns 忽略这些列的检查。

- 支持对不包含主键或者唯一索引的表进行校验，但是如果数据不一致，生成的用于修复的 SQL 可能无法正确修复数据。

## 4、所需的数据库权限

**sync-diff-inspector** 需要获取表结构信息、查询数据，需要的数据库权限如下：

- 上游数据库
  - **SELECT**（查数据进行对比）
  - **SHOW_DATABASES**（查看库名）
  - **RELOAD**（查看表结构）
- 下游数据库
  - **SELECT**（查数据进行对比）
  - **SHOW_DATABASES**（查看库名）
  - **RELOAD**（查看表结构）

## 5、数据修复

校验过程中遇到不同的行，会生成修复数据的 SQL 语句。一个 chunk 如果出现数据不一致，就会生成一个以 `chunk.Index` 命名的 SQL 文件。文件位于 `${output}/fix-on-${instance}` 文件夹下。其中 `${instance}` 为 `config.toml` 中 `task.target-instance` 的值。

一个 SQL 文件会包含该 chunk 的所属表以及表示的范围信息。对每个修复 SQL 语句，有三种情况：

- 下游数据库缺失行，则是 REPLACE 语句
- 下游数据库冗余行，则是 DELETE 语句
- 下游数据库行部分数据不一致，则是 REPLACE 语句，但会在 SQL 文件中通过注释的方法标明不同的列

## <font color=red>6、注意事项</font>

- 如果上下游数据表结构不一致，则无法进行数据校验

  ```bash
  [2023/01/09 09:06:19.089 +00:00] [ERROR] [utils.go:368] ["column properties not compatible"] ["upstream table"=btest1] ["column name"=pad] ["column type"=254] ["downstream table"=btest1] ["column name"=pad] ["column type"=254] [stack="github.com/pingcap/tidb-tools/sync_diff_inspector/utils.CompareStruct\n\t/home/jenkins/agent/workspace/d_tidb_tools_multi_branch_master/go/src/github.com/pingcap/tidb-tools/sync_diff_inspector/utils/utils.go:368\nmain.(*Diff).compareStruct\n\t/home/jenkins/agent/workspace/d_tidb_tools_multi_branch_master/go/src/github.com/pingcap/tidb-tools/sync_diff_inspector/diff.go:318\nmain.(*Diff).StructEqual\n\t/home/jenkins/agent/workspace/d_tidb_tools_multi_branch_master/go/src/github.com/pingcap/tidb-tools/sync_diff_inspector/diff.go:302\nmain.checkSyncState\n\t/home/jenkins/agent/workspace/d_tidb_tools_multi_branch_master/go/src/github.com/pingcap/tidb-tools/sync_diff_inspector/main.go:125\nmain.main\n\t/home/jenkins/agent/workspace/d_tidb_tools_multi_branch_master/go/src/github.com/pingcap/tidb-tools/sync_diff_inspector/main.go:104\nruntime.main\n\t/usr/local/go/src/runtime/proc.go:250"]
  ```

- sync-diff-inspector 在校验数据时会消耗一定的服务器资源，需要避免在业务高峰期间校验。

- 在数据对比前，需要注意表中的 collation 设置。如果表的主键或唯一键为 varchar 类型，且上下游数据库中 collation 设置不同，可能会因为排序问题导致最终校验结果不正确，需要在 sync-diff-inspector 的配置文件中增加 collation 设置。

- sync-diff-inspector 会优先使用 TiDB 的统计信息来划分 chunk，需要尽量保证统计信息精确，可以在**业务空闲期**手动执行 `analyze table {table_name}`。

- table-rule 的规则需要特殊注意，例如设置了 `schema-pattern="test1"`，`table-pattern = "t_1"`，`target-schema="test2"`，`target-table = "t_2"`，会对比 source 中的表 `test1`.`t_1` 和 target 中的表 `test2`.`t_2`。sync-diff-inspector 默认开启 sharding，如果 source 中还有表 `test2`.`t_2`，则会把 source 端的表 `test1`.`t_1` 和表 `test2`.`t_2` 作为 sharding 与 target 中的表 `test2`.`t_2` 进行一致性校验。

- 生成的 SQL 文件仅作为修复数据的参考，需要确认后再执行这些 SQL 修复数据。

# 二、安装

## 1、二进制

sync-diff-inspector 的安装包位于 TiDB 离线工具包中

## 2、Docker

```bash
docker pull pingcap/tidb-enterprise-tools:nightly
```

# 三、配置文件

## 1、配置文件详解

配置总共分为五个部分：

- **Global config:** 通用配置，包括校验的线程数量、是否输出修复 SQL 、是否比对数据等。
- **Datasource config**: 配置上下游数据库实例。
- **Routes**: 上游多表名通过正则匹配下游单表名的规则。（可选）
- **Task config**: 配置校验哪些表，如果有的表在上下游有一定的映射关系或者有一些特殊要求，则需要对指定的表进行配置。
- **Table config**: 对具体表的特殊配置，例如指定范围、忽略的列等等。（可选）

```bash
######################### Global config #########################

# 检查数据的线程数量，上下游数据库的连接数会略大于该值
check-thread-count = 4
# 如果开启，若表存在不一致，则输出用于修复的 SQL 语句。
export-fix-sql = true
# 只对比表结构而不对比数据
check-struct-only = false

######################### Datasource config #########################
[data-sources]
[data-sources.mysql1] # mysql1 是该数据库实例唯一标识的自定义 id，用于下面 task.source-instances/task.target-instance 中
    host = "127.0.0.1"
    port = 3306
    user = "root"
    password = "" # 设置连接上游数据库的密码，可为明文或 Base64 编码。
    #（可选）使用映射规则来匹配上游多个分表，其中 rule1 和 rule2 在下面 Routes 配置栏中定义
    route-rules = ["rule1", "rule2"]
[data-sources.tidb0]
    host = "127.0.0.1"
    port = 4000
    user = "root"
    password = "" # 设置连接下游数据库的密码，可为明文或 Base64 编码。
    #（可选）使用 TLS 连接 TiDB
    # security.ca-path = ".../ca.crt"
    # security.cert-path = ".../cert.crt"
    # security.key-path = ".../key.crt"
    
    #（可选）使用 TiDB 的 snapshot 功能，如果开启的话会使用历史数据进行对比
    # snapshot = "386902609362944000"
    # 当 snapshot 设置为 "auto" 时，使用 TiCDC 在上下游的同步时间点，具体参考 <https://github.com/pingcap/tidb-tools/issues/663>
    # snapshot = "auto"
    
########################### Routes ###########################
# 如果需要对比大量的不同库名或者表名的表的数据，或者用于校验上游多个分表与下游总表的数据，可以通过 table-rule 来设置映射关系
# 可以只配置 schema 或者 table 的映射关系，也可以都配置
[routes]
[routes.rule1] # rule1 是该配置的唯一标识的自定义 id，用于上面 data-sources.route-rules 中
schema-pattern = "test_*"      # 匹配数据源的库名，支持通配符 "*" 和 "?"
table-pattern = "t_*"          # 匹配数据源的表名，支持通配符 "*" 和 "?"
target-schema = "test"         # 目标库名
target-table = "t" # 目标表名

[routes.rule2]
schema-pattern = "test2_*"      # 匹配数据源的库名，支持通配符 "*" 和 "?"
table-pattern = "t2_*"          # 匹配数据源的表名，支持通配符 "*" 和 "?"
target-schema = "test2"         # 目标库名
target-table = "t2" # 目标表名

######################### Task config #########################
# 配置需要对比的*目标数据库*中的表
[task]
    # output-dir 会保存如下信息
    # 1 sql: 检查出错误后生成的修复 SQL 文件，并且一个 chunk 对应一个文件
    # 2 log: sync-diff.log 保存日志信息
    # 3 summary: summary.txt 保存总结
    # 4 checkpoint: a dir 保存断点续传信息
    output-dir = "./output"
    # 上游数据库，内容是 data-sources 声明的唯一标识 id
    source-instances = ["mysql1"]
    # 下游数据库，内容是 data-sources 声明的唯一标识 id
    target-instance = "tidb0"
    # 需要比对的下游数据库的表，每个表需要包含数据库名和表名，两者由 `.` 隔开
    # 使用 ? 来匹配任意一个字符；使用 * 来匹配任意；详细匹配规则参考 golang regexp pkg: https://github.com/google/re2/wiki/Syntax
    target-check-tables = ["schema*.table*", "!c.*", "test2.t2"]
    #（可选）对部分表的额外配置，其中 config1 在下面 Table config 配置栏中定义
    target-configs = ["config1"]

######################### Table config #########################
# 对部分表进行特殊的配置，配置的表必须包含在 task.target-check-tables 中
[table-configs.config1] # config1 是该配置的唯一标识自定义 id，用于上面 task.target-configs 中
# 目标表名称，可以使用正则来匹配多个表，但不允许存在一个表同时被多个特殊配置匹配。
target-tables = ["schema*.test*", "test2.t2"]
#（可选）指定检查的数据的范围，需要符合 sql 中 where 条件的语法
range = "age > 10 AND age < 20"
#（可选）指定用于划分 chunk 的列，如果不配置该项，sync-diff-inspector 会选取一些合适的列（主键／唯一键／索引）
index-fields = ["col1","col2"]
#（可选）忽略某些列的检查，例如 sync-diff-inspector 目前还不支持的一些类型（json，bit，blob 等），
# 或者是浮点类型数据在 TiDB 和 MySQL 中的表现可能存在差异，可以使用 ignore-columns 忽略检查这些列
ignore-columns = ["",""]
#（可选）指定划分该表的 chunk 的大小，若不指定可以删去或者将其配置为 0。
chunk-size = 0
#（可选）指定该表的 collation，若不指定可以删去或者将其配置为空字符串。
collation = ""
```

## 2、基于DM同步场景下数据校验的配置

当使用 [TiDB DM](https://docs.pingcap.com/zh/tidb/stable/dm-overview) 等同步工具时，需要校验 DM 同步后数据的一致性。可以从 `DM-master` 拉取指定 `task-name` 的配置，进行数据校验。

```toml
######################### Global config #########################
# 检查数据的线程数量，上下游数据库的连接数会略大于该值
check-thread-count = 4
# 如果开启，若表存在不一致，则输出用于修复的 SQL 语句
export-fix-sql = true
# 只对比表结构而不对比数据
check-struct-only = false
# dm-master 的地址, 格式为 "http://127.0.0.1:8261"
dm-addr = "http://127.0.0.1:8261"
# 指定 DM 的 `task-name`
dm-task = "test"

######################### Task config #########################
[task]
    output-dir = "./output"
    # 需要比对的下游数据库的表，每个表需要包含数据库名和表名，两者由 `.` 隔开
    target-check-tables = ["hb_test.*"]
```

该配置在 dm-task="test" 中，会对该任务下 hb_test 库的所有表进行检验，自动从 DM 配置中获取上游对下游库名的正则匹配，以校验 DM 同步后数据的一致性。

# 四、任务运行

```bash
mkdir output 
docker run -it --rm \
  -v ${PWD}:/test \
  -v ${PWD}/output:/output \
  pingcap/tidb-enterprise-tools:nightly /sync_diff_inspector -C /test/sync_diff_inspector-config.toml
```

该命令最终会在 config.toml 中的 output-dir 输出目录输出本次比对的检查报告 summary.txt 和日志 sync_diff.log。在输出目录下还会生成由 config.toml 文件内容哈希值命名的文件夹，该文件夹下包括断点续传 checkpoint 结点信息以及数据存在不一致时生成的 SQL 修复数据。

```bash
output/
  |-- summary.txt                      # 保存校验结果的总结
  |-- sync_diff.log                    # 保存 sync-diff-inspector 执行过程中输出的日志信息
  |-- checkpoint                       # 保存断点续传信息
  |   |-- bbfec8cc8d1f58a5800e63aa73e5 # config hash 占位文件，标识该输出目录（output/）对应的配置文件
  │   └-- sync_diff_checkpoints.pb     # 断点续传信息
  |-- fix-on-target                    # 保存用于修复不一致的 SQL 文件
  |   |-- xxx.sql
  |   └-- xxx.sql
```

- **日志**：sync-diff-inspector 的日志存放在 `${output}/sync_diff.log` 中，其中 `${output}` 是 `config.toml` 文件中 `output-dir` 的值。

- **校验进度**

  sync-diff-inspector 会在运行时定期（间隔 10s）输出校验进度到 checkpoint 中(位于 `${output}/checkpoint/sync_diff_checkpoints.pb` 中，其中 `${output}` 是 `config.toml` 文件中 `output-dir` 的值。

- **校验结果**

  当校验结束时，sync-diff-inspector 会输出一份校验报告，位于 `${output}/summary.txt` 中，其中 `${output}` 是 `config.toml` 文件中 `output-dir` 的值。

  ```bash
  +----------------------+--------------------+------------------------+----------+-----------+
  |        TABLE         | STRUCTURE EQUALITY | DATA DIFF ROWS         | UPCOUNT  | DOWNCOUNT |
  |        数据库.表名     | 表结构是否相同       | 数据差异行(增加/删除行数)  |上游数据总数|下游数据总数 |
  +----------------------+--------------------+------------------------+----------+-----------+
  | `test`.`btest1`      | true               |    +4/-3               |     999  |       998 |
  +----------------------+--------------------+------------------------+----------+-----------+
  
  Time Cost: 156.398881ms
  Average Speed: 0.832264MB/s
  ```

# 五、测试

- 目标表两条数据字段值有变更
- 目标表缺失一条数据
- 目标表多一条数据

```sql
-- table: test.btest1
-- range in sequence: Full
/*
  DIFF COLUMNS ╏                                                            `C`                                                             
╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╋╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍
  source data  ╏ '2-15688153734-6-1-9-5-6-5-51724173961-3'  
╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╋╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍
  target data  ╏ '1'                                                                                                                        
╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╋╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍
*/
REPLACE INTO `test`.`btest1`(`id`,`k`,`c`,`pad`) VALUES (1,367,'2-15688153734-6-1-9-5-6-5-51724173961-3','1111');
REPLACE INTO `test`.`btest1`(`id`,`k`,`c`,`pad`) VALUES (4,88,'222222','333333');
/*
  DIFF COLUMNS ╏                                                            `C`                                                             
╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╋╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍
  source data  ╏ '3-02480546617-1-1-5-9-7-6-6-2'  
╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╋╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍
  target data  ╏ '2'                                                                                                                        
╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╋╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍
*/
REPLACE INTO `test`.`btest1`(`id`,`k`,`c`,`pad`) VALUES (5,53,'3-02480546617-1-1-5-9-7-6-6-2','222');
DELETE FROM `test`.`btest1` WHERE `id` = 22 AND `k` = 350 AND `c` = '666666' AND `pad` = '777777' LIMIT 1;
```



# 参考

- https://docs.pingcap.com/zh/tidb/stable/sync-diff-inspector-overview