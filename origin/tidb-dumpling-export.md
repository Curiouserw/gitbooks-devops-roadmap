## Dumpling全量备份或导出

# 一、Dumpling 

## 1、简介

`Dumpling` 是使用 go 开发的数据备份工具，项目地址可以参考 [`Dumpling`](https://github.com/pingcap/dumpling)。

`Dumpling` 的更多具体用法可查看 [Dumpling 主要选项表](https://docs.pingcap.com/zh/tidb/stable/dumpling-overview#dumpling-主要选项表)。

`Dumpling` 包含在` tidb-toolkit` 安装包中，下载链接：https://download.pingcap.org/tidb-toolkit-v4.0.5-linux-amd64.tar.gz

为了快速地备份恢复数据（特别是数据量巨大的库），可以参考以下建议：

- 导出来的数据文件应当尽可能的小，可以通过设置选项 `-F` 来控制导出来的文件大小。
- 如果后续使用 TiDB Lightning 对备份文件进行恢复，建议把 `dumpling` -F 选项的值设置为 `256m`。
- 如果导出的表中有些表的行数非常多，可以通过设置选项 `-r` 来开启表内并发。

## 2、对比Mydumper

- 支持导出多种数据形式，包括 SQL/CSV

- 支持全新的 [table-filter](https://github.com/pingcap/tidb-tools/blob/master/pkg/table-filter/README.md)，筛选数据更加方便

- 针对 TiDB 进行了更多优化：
  - 支持配置 TiDB 单条 SQL 内存限制

  - 针对 TiDB v4.0.0 以上版本支持自动调整 TiDB GC 时间
  - 使用 TiDB 的隐藏列 `_tidb_rowid` 优化了单表内数据的并发导出性能
  - 对于 TiDB 可以设置 [tidb_snapshot](https://docs.pingcap.com/zh/tidb/stable/read-historical-data#操作流程) 的值指定备份数据的时间点，从而保证备份的一致性，而不是通过 `FLUSH TABLES WITH READ LOCK` 来保证备份一致性。

# 二、从TiDB/MySQL 导出数据

## 1、源库导出账号所需权限

- SELECT
- RELOAD
- LOCK TABLES
- REPLICATION CLIENT

## 2、安装及主要参数

### ①安装

- **二进制**

  ```bash
  version=v4.0.5 && \
  curl -# https://download.pingcap.org/tidb-toolkit-$version-linux-amd64.tar.gz | tar -zxC /opt && \
  ln -s /opt/tidb-toolkit-$version-linux-amd64 /opt/tidb-toolkit-$version && \
  echo "export PATH=/opt/tidb-toolkit-$version/bin:$PATH" >> /etc/profile && \
  source /etc/profile && \ 
  dumpling -V
  ```

- **源码构建**

  ```bash
  # MacOS源码构建（要求：golang>=1.6,二进制输出路径：bin/dumpling）
  git clone https://github.com/pingcap/dumpling.git && \
  cd dumpling && \
  git checkout v4.0.5 && \
  make build && \
  chmox +X ./bin/dumpling && \
  mv ./bin/dumpling /usr/local/bin/ && \
  dumpling --version
  ```

- **Docker**

  ```bash
  docker pull pingcap/dumpling:v5.1.0
  docker run -it -v 本地存储导出SQL文件的目录 pingcap/dumpling:v5.1.0 sh
  # dumpling命令执行路径在根目录下,具体的导出任务可以在容器中执行
  ```

- **MacOS**

  ```bash
  brew install dumpling
  dumpling --version
  ```

### ②命令参数

| 主要选项                   | 用途                                                         | 默认值                                  |
| :------------------------- | ------------------------------------------------------------ | --------------------------------------- |
| -V 或 --version            | 输出 Dumpling 版本并直接退出                                 |                                         |
| -B 或 --database           | 导出指定数据库                                               |                                         |
| -T 或 --tables-list        | 导出指定数据表                                               |                                         |
| -f 或 --filter             | 导出能匹配模式的表，语法可参考 [table-filter](https://docs.pingcap.com/zh/tidb/stable/table-filter) | `*.*`（导出所有库表）                   |
| --case-sensitive           | table-filter 是否大小写敏感                                  | false，大小写不敏感                     |
| -h 或 --host               | 连接的数据库主机的地址                                       | "127.0.0.1"                             |
| -t 或 --threads            | 备份并发线程数                                               | 4                                       |
| -r 或 --rows               | 将 table 划分成 row 行数据，一般针对大表操作并发生成多个文件。 |                                         |
| -L 或 --logfile            | 日志输出地址，为空时会输出到控制台                           | ""                                      |
| --loglevel                 | 日志级别 {debug,info,warn,error,dpanic,panic,fatal}          | "info"                                  |
| --logfmt                   | 日志输出格式 {text,json}                                     | "text"                                  |
| -d 或 --no-data            | 不导出数据，适用于只导出 schema 场景                         |                                         |
| --no-header                | 导出 csv 格式的 table 数据，不生成 header                    |                                         |
| -W 或 --no-views           | 不导出 view                                                  | true                                    |
| -m 或 --no-schemas         | 不导出 schema，只导出数据                                    |                                         |
| -s 或--statement-size      | 控制 `INSERT` SQL 语句的大小，单位 bytes                     |                                         |
| -F 或 --filesize           | 将 table 数据划分出来的文件大小，需指明单位（如 `128B`, `64KiB`, `32MiB`, `1.5GiB`） |                                         |
| --filetype                 | 导出文件类型（csv/sql）                                      | "sql"                                   |
| -o 或 --output             | 导出文件路径                                                 | "./export-${time}"                      |
| -S 或 --sql                | 根据指定的 sql 导出数据，该选项不支持并发导出                |                                         |
| --consistency              | flush: dump 前用 FTWRL<br/>snapshot: 通过 TSO 来指定 dump 某个快照时间点的 TiDB 数据 <br/>lock: 对需要 dump 的所有表执行 `lock tables read` 命令 <br/>none: 不加锁 dump，无法保证一致性 <br/>auto: MySQL 默认用 flush, TiDB 默认用 snapshot | "auto"                                  |
| --snapshot                 | snapshot tso，只在 consistency=snapshot 下生效               |                                         |
| --where                    | 对备份的数据表通过 where 条件指定范围                        |                                         |
| -p 或 --password           | 连接的数据库主机的密码                                       |                                         |
| -P 或 --port               | 连接的数据库主机的端口                                       | 4000                                    |
| -u 或 --user               | 连接的数据库主机的用户名                                     | "root"                                  |
| --dump-empty-database      | 导出空数据库的建库语句                                       | true                                    |
| --ca                       | 用于 TLS 连接的 certificate authority 文件的地址             |                                         |
| --cert                     | 用于 TLS 连接的 client certificate 文件的地址                |                                         |
| --key                      | 用于 TLS 连接的 client private key 文件的地址                |                                         |
| --csv-delimiter            | csv 文件中字符类型变量的定界符                               | '"'                                     |
| --csv-separator            | csv 文件中各值的分隔符                                       | ','                                     |
| --csv-null-value           | csv 文件空值的表示                                           | "\N"                                    |
| --escape-backslash         | 使用反斜杠 (`\`) 来转义导出文件中的特殊字符                  | true                                    |
| --output-filename-template | 以[golang template](https://golang.org/pkg/text/template) 格式表示的数据文件名格式 支持 DB Table、Index 三个参数 分别表示数据文件的库名、表名、分块 ID | \{\{.DB}}.\{\{.Table\}\}.\{\{.Index\}\} |
| --status-addr              | Dumpling 的服务地址，包含了 Prometheus 拉取 metrics 信息及 pprof 调试的地址 | ":8281"                                 |
| --tidb-mem-quota-query     | 单条 dumpling 命令导出 SQL 语句的内存限制，单位为 byte，默认为 32 GB | 34359738368                             |

## 3、导出数据文件格式

### ①导出到 sql 文件

Dumpling 默认导出数据格式为 sql 文件。也可以通过设置 `--filetype sql` 导出数据到 sql 文件：

```bash
dumpling -u root -P 4000 -h 127.0.0.1 --filetype sql --threads 32 -o /data/dumpling-export -F 256
```

上述命令中，`-h`、`-P`、`-u` 分别是地址，端口，用户。如果需要密码验证，可以用 `-p $YOUR_SECRET_PASSWORD` 传给 Dumpling。

### ②导出到 csv 文件

假如导出数据的格式是 CSV（使用 `--filetype csv` 即可导出 CSV 文件），还可以使用 `--sql <SQL>` 导出指定 SQL 选择出来的记录，例如，导出 `test.sbtest1` 中所有 `id < 100` 的记录：

```bash
dumpling -u root -P 4000 -h 127.0.0.1 -o /data/dumpling-export/test \
  --filetype csv \
  --sql 'select * from `test`.`sbtest1` where id < 100'
```

> **注意：**
>
> 1. `--sql` 选项暂时仅仅可用于导出 csv 的场景。
> 2. 这里需要在要导出的所有表上执行 `select * from <table-name> where id < 100` 语句。如果部分表没有指定的字段，那么导出会失败。

## 4、筛选导出的数据

### ①使用 `--where` 选项筛选数据

默认情况下，除了系统数据库中的表之外，Dumpling 会导出整个数据库的表。你可以使用 `--where <SQL where expression>` 来选定要导出的记录。

```bash
dumpling -u root -P 4000 -h 127.0.0.1 -o /data/dumpling-export/test \
  --where "id < 100"
```

上述命令将会导出各个表的 id < 100 的数据。

### ②使用 `--filter` 选项筛选数据

Dumpling 可以通过 `--filter` 指定 table-filter 来筛选特定的库表。table-filter 的语法与 .gitignore 相似，详细语法参考[表库过滤](https://docs.pingcap.com/zh/tidb/stable/table-filter)。

```bash
dumpling -u root -P 4000 -h 127.0.0.1 -o /data/dumpling-export/test \
  --filter "employees.*" \
  --filter "*.WorkOrder"
```

上述命令将会导出 `employees` 数据库的所有表，以及所有数据库中的 `WorkOrder` 表。

### ③使用 `-B` 或 `-T` 选项筛选数据

Dumpling 也可以通过 `-B` 或 `-T` 选项导出特定的数据库/数据表。

> **注意：**
>
> 1. `--filter` 选项与 `-T` 选项不可同时使用。
> 2. `-T` 选项只能接受完整的 `库名.表名` 形式，不支持只指定表名。例：Dumpling 无法识别 `-T WorkOrder`。

例如通过指定：

- `-B employees` ：导出 `employees` 数据库
- `-T employees.WorkOrder` ：导出 `employees.WorkOrder` 数据表

## 5、并发提高 Dumpling导出效率选项

默认情况下，导出的文件会存储到 `./export-<current local time>` 目录下。常用选项如下：

- `-o` 用于选择存储导出文件的目录。
- `-F` 选项用于指定单个文件的最大大小，默认单位为 `MiB`。可以接受类似 `5GiB` 或 `8KB` 的输入。
- `-r` 选项用于指定单个文件的最大记录数（或者说，数据库中的行数），开启后 Dumpling 会开启表内并发，提高导出大表的速度。

利用以上选项可以让 Dumpling 的并行度更高。

## 6、调整 Dumpling 的数据一致性选项

> **注意：**
> 在大多数场景下，用户不需要调整 Dumpling 的默认数据一致性选项。

Dumpling 通过 `--consistency <consistency level>` 标志控制导出数据“一致性保证”的方式。对于 TiDB 来说，默认情况下，会通过获取某个时间戳的快照来保证一致性（即 `--consistency snapshot`）。在使用 snapshot 来保证一致性的时候，可以使用 `--snapshot` 选项指定要备份的时间戳。还可以使用以下的一致性级别：

- `flush`：使用 [`FLUSH TABLES WITH READ LOCK`](https://dev.mysql.com/doc/refman/8.0/en/flush.html#flush-tables-with-read-lock) 来保证一致性。
- `snapshot`：获取指定时间戳的一致性快照并导出。
- `lock`：为待导出的所有表上读锁。
- `none`：不做任何一致性保证。
- `auto`：对 MySQL 使用 `flush`，对 TiDB 使用 `snapshot`。

## 7、导出的SQL文件

TiDB Dumping导出的SQL文件命名格式都有：

- `metadata`：此文件包含导出的起始时间以及 master binary log 的位置。

  ```bash
  Started dump at: 2020-11-10 10:40:19
  SHOW MASTER STATUS:
          Log: tidb-binlog
          Pos: 420747102018863124
  
  Finished dump at: 2020-11-10 10:40:20
  ```

- `{database}-schema-create.sql`：创建database的 SQL 文件on

- `{database}.{table}-schema.sql`：创建 table 的 SQL 文件

- `{database}.{table}.{0001}.{sql|csv`}：数据源文件

- `*-schema-view.sql`、`*-schema-trigger.sql`、`*-schema-post.sql`：其他导出文件

后续如果想使用TiDB Lighting将这些SQL文件导入到TiDB**另外一个的DB**中的话，可批量将SQL文件名的database部分改掉

```bash
# 例如源库DB为Test，想把数据导入到目标库Test-2中
old_database_name=test
new_database_name=Test-2
for i in $(ls *.sql | grep -v schema-create );do 
		mv $i $new_database_name.${i#*.};
done
mv ${old_database_name}-schema-create.sql ${new_database_name}-schema-create.sql
echo "" > ${new_database_name}-schema-create.sql
```

## 8、导出 TiDB 的历史数据快照

Dumpling 可以通过 `--snapshot` 指定导出某个 [tidb_snapshot](https://docs.pingcap.com/zh/tidb/stable/read-historical-data#操作流程) 时的数据。

`--snapshot` 选项可设为 TSO（`SHOW MASTER STATUS` 输出的 `Position` 字段）或有效的 `datetime` 时间，例如：

```bash
dumpling --snapshot 417773951312461825
dumpling --snapshot "2020-07-02 17:12:45"
```

即可导出 TSO 为 `417773951312461825` 或 `2020-07-02 17:12:45` 时的 TiDB 历史数据快照。

## 9、导出大规模数据时的 TiDB GC 设置

从 v4.0.0 版本开始，Dumpling 可以自动延长 GC 时间（Dumpling 需要访问 TiDB 集群的 PD 地址），而 v4.0.0 之前的版本，需要手动调整 GC 时间，否则 `dumpling` 备份时可能出现以下报错：

```bash
Could not read data from testSchema.testTable: GC life time is shorter than transaction duration, transaction starts at 2019-08-05 21:10:01.451 +0800 CST, GC safe point is 2019-08-05 21:14:53.801 +0800 CST
```

**手动调整 GC 时间的步骤：**

1. 执行 `dumpling` 命令前，查询 TiDB 集群的 [GC](https://docs.pingcap.com/zh/tidb/stable/garbage-collection-overview) 值并在 MySQL 客户端执行下列语句将其调整为合适的值：

   ```bash
   SELECT * FROM mysql.tidb WHERE VARIABLE_NAME = 'tikv_gc_life_time';
   +-----------------------+--------------------+
   | VARIABLE_NAME         | VARIABLE_VALUE     |                                                       
   +-----------------------+--------------------+
   | tikv_gc_life_time     | 10m0s              |                                                        
   +-----------------------+--------------------+
   
   update mysql.tidb set VARIABLE_VALUE = '720h' where VARIABLE_NAME = 'tikv_gc_life_time';
   ```
   
2. 执行 `dumpling` 命令后，将 TiDB 集群的 GC 值恢复到第 1 步中的初始值：

   ```bash
   update mysql.tidb set VARIABLE_VALUE = '10m' where VARIABLE_NAME = 'tikv_gc_life_time';
   ```

# 三、常用操作

## 1、zsh函数备份脚本

- 可备份 TiDB 、MySQL
- 可备份某些整库
- 可备份某些库的某些表

```bash
check_commands_exists() {
    need_commands=($*)
    no_commands=""
    for command in "${need_commands[@]}" ; do
        if ! command -v $command &> /dev/null; then
            no_commands+=" $command"
        fi
    done

    if [[ $no_commands ]]; then
        echo -e "\033[31m$no_commands 命令不存在，正在下载安装！\033[0m"
        os_type=$(uname -s)
        case "$os_type" in
            Linux*)
                os_distribution=$(lsb_release -si)
                if [ "$os_distribution" = "Ubuntu" ] || [ "$os_distribution" = "Debian" ]; then
                    apt install -y $no_commands >/dev/null 2>&1
                elif [ "$os_distribution" = "CentOS" ]; then
                    yum install -y $no_commands >/dev/null 2>&1
                elif [ "$os_distribution" = "Fedora" ]; then
                    dnf install -y $no_commands >/dev/null 2>&1
                fi
                if [[ $? == 0 ]] ;then
                    echo -e "\033[31m$no_commands 命令已安装！\033[0m"
                fi
                ;;
            Darwin*)
                brew install $no_commands >/dev/null 2>&1
                if [[ $? == 0 ]] ;then
                    echo -e "\033[31m$no_commands 命令已安装！\033[0m"
                fi
                ;;
            CYGWIN*|MINGW32*|MSYS*|MINGW*)
                echo -e "\033[31mWindows操作系统，请手动安装！\033[0m"
                ;;
            *)
                echo -e "\033[31m未知的操作系统\033[0m"
                return
                ;;
        esac
    fi
}

dumpling-mysql-tidb-data-to-file() {
  check_commands_exists pidof mysql_config_editor
  echo "数据库连接地址: "
  echo -e "  \e[;33m1. 手动输入数据库连接信息\033[0m"
  echo -e "  \e[;33m2. 生产环境TiDB  - 192.168.1.12:4000\033[0m"
  echo -e "  \e[;33m3. 测试环境TiDB  - 192.168.1.13:4000\033[0m"
  echo -e "  \e[;33m4. 测试环境MySQL - 192.168.1.14:3306\033[0m"
  echo -e "  \e[;33m5. 退出\033[0m"

  read "?请设置数据库连接地址，默认[1]: " DB_HOST_OPT
  until [[ -z "$DB_HOST_OPT" || "$DB_HOST_OPT" =~ ^[12345]$ ]]; do
    echo -e "  \033[41;37m输入$DB_HOST_OPT为无效的选项！可选项：1,2,3,4,5\033[0m"
    read "?请重新设置数据库连接地址，默认[1]: " DB_HOST_OPT
  done
  [[ -z "$DB_HOST_OPT" ]] && DB_HOST_OPT=1
  case "$DB_HOST_OPT" in
  1)
    read "?请输入数据库连接地址: " DB_HOST
    until [[ ! -z "$DB_HOST" && $DB_HOST =~ ^192.168.[1-9]{1,3}\.*$ ]]; do
      if [ ! $DB_HOST ] ;then
        echo -e "  \033[41;37m数据库连接地址为必填项，不能为空！\033[0m"
      else
        echo  -e "  \033[41;37m$DB_HOST为无效的内网IP地址！\033[0m"
      fi
      read "?请重新输入数据库连接地址: " DB_HOST
    done
    read "?请输入数据库连接端口: " DB_HOST_PORT
    until [[ ! -z "$DB_HOST_PORT" && $DB_HOST_PORT =~ ^(3306|4000)$ ]]; do
      if [ ! $DB_HOST_PORT ] ;then
        echo -e "  \033[41;37m数据库连接端口为必填项，不能为空！\033[0m"
      else
        echo -e "  \033[41;37m$DB_HOST_PORT为无效的数据库端口！\033[0m"
      fi
      read "?请重新输入数据库连接端口: " DB_HOST_PORT
    done
    read "?请输入数据库连接用户名: " DB_USERNAME
    until [[ ! -z "$DB_USERNAME" ]]; do
      echo -e "  \033[41;37m数据库连接用户为必填项，不能为空！\033[0m"
      read "?请重新输入数据库连接用户名: " DB_USERNAME
    done
    echo -e "请输入数据库用户密码: "
    mysql_config_editor remove --login-path=tmp
    mysql_config_editor set --login-path=tmp --user=$DB_USERNAME --host=$DB_HOST --port=$DB_HOST_PORT --password
    DB_ALIAS=tmp
    ;;
  2)
    DB_ALIAS=prod-tidb
    output=$(mysql_config_editor print --login-path=$DB_ALIAS)
    DB_HOST=$(echo "$output" | awk -F'"' '/host/ {print $2}')
    DB_HOST_PORT=$(echo "$output" | awk -F'= ' '/port/ {print $2}')
    DB_USERNAME=$(echo "$output" | awk -F'"' '/user/ {print $2}')
    ;;
  3)
    DB_ALIAS=test-tidb
    output=$(mysql_config_editor print --login-path=$DB_ALIAS)
    DB_HOST=$(echo "$output" | awk -F'"' '/host/ {print $2}')
    DB_HOST_PORT=$(echo "$output" | awk -F'= ' '/port/ {print $2}')
    DB_USERNAME=$(echo "$output" | awk -F'"' '/user/ {print $2}')
    ;;
  4) 
    DB_ALIAS=test-mysql
    output=$(mysql_config_editor print --login-path=$DB_ALIAS)
    DB_HOST=$(echo "$output" | awk -F'"' '/host/ {print $2}')
    DB_HOST_PORT=$(echo "$output" | awk -F'= ' '/port/ {print $2}')
    DB_USERNAME=$(echo "$output" | awk -F'"' '/user/ {print $2}')
    ;;
  5)
    return
    ;;  
  esac

  read -s "?请输入数据库 $DB_HOST:$DB_HOST_PORT 连接用户 $DB_USERNAME 的密码: " DB_PASSWORD
  until [[ ! -z "$DB_PASSWORD" ]]; do
    if [ ! $DB_PASSWORD ] ;then
      echo -e "  \033[41;37m数据库连接密码为必填项。不能为空！\033[0m"
    else
      echo -e "  \033[41;37m$DB_PASSWORD为无效的输入！\033[0m"
    fi
    read -s "?请输入数据库 $DB_HOST:$DB_HOST_PORT 连接用户 $DB_USERNAME 的密码: " DB_PASSWORD
  done
  mysql_config_editor remove --login-path=$DB_ALIAS
  echo
  echo -e "请再次输入"
  mysql_config_editor set --login-path=$DB_ALIAS --host=$DB_HOST --port=$DB_HOST_PORT --user=$DB_USERNAME --password

  show_select_database(){
    all_databases=$(mysql --login-path=$DB_ALIAS -e 'show databases;'  |grep -i -Ev '"+-----------------------+"|Database|Warning|METRICS_SCHEMA|INFORMATION_SCHEMA|mysql|PERFORMANCE_SCHEMA' 2> /dev/null)
    count=1
    for db in ${(f)all_databases}; do
      printf "\e[;36m%-2s. %-24s\033[0m" "$count" "$db"
      if [ $((count % 5)) -eq 0 ]; then
        echo ""
      fi
      ((count++))
    done
    selected_databases_all=""
    selected_databases_set=()
    selected_databases_summary=""
    echo
    while true; do
      read "?请输入要备份的数据库序号(以逗号分隔): " selected_indices
      IFS=',' read -A -r indices <<< "$selected_indices"
      invalid_indices=()
      for index in "${indices[@]}"; do
        if ((index >= 1 && index <= count)); then
          selected_database=$(echo "$all_databases" | sed -n "${index}p")
          if [[ "$all_databases" =~ $selected_database ]]; then
            if ! [[ "${selected_databases_set[@]}" =~ "$selected_database" ]]; then
              selected_databases_set+=("$selected_database")
              selected_databases_summary+="$selected_database,"
            fi
          else
            invalid_indices+=("$index")
          fi
        else
          invalid_indices+=("$index")
        fi
      done
      if [ ${#invalid_indices[@]} -eq 0 ]; then
        break
      else
        echo "  \033[41;37m${invalid_indices[@]}为无效的数据库序号\033[0m"
      fi
    done
    BACKUP_DBS=${selected_databases_summary%,}
  }

  show_select_database_tables(){
    IFS=',' read -A -r SELECTED_DBS <<< "$1"
    selected_tables_summary=""
    for SELECTED_DB in ${SELECTED_DBS[@]} ;do 
      db_tables=$(mysql --login-path=$DB_ALIAS --database $SELECTED_DB -e 'show tables;'  |grep -i -Ev '"+-----------------------+"|Tables_in|Warning' 2> /dev/null)
      count=1
      for table in ${(f)db_tables}; do
        printf "\e[;36m%-3s. %-32s\033[0m" "$count" "$table"
        if [ $((count % 5)) -eq 0 ]; then
          echo ""
        fi
        ((count++))
      done
      echostr+="\n  $SELECTED_DB库的: \n    "
      echotablesstr=""
      selected_tables_all=""
      selected_tables_set=()
      echo
      while true; do
        read "?请输入 $SELECTED_DB 库中要备份表的序号(多张表以逗号分隔): " selected_indices
        IFS=',' read -A -r indices <<< "$selected_indices"
        invalid_indices=()
        for index in "${indices[@]}"; do
          if ((index >= 1 && index <= count)); then
            selected_table=$(echo "$db_tables" | sed -n "${index}p")
            if [[ "$db_tables" =~ $selected_table ]]; then
              if ! [[ "${selected_tables_set[@]}" =~ "$selected_table" ]]; then
                selected_tables_set+=("$SELECTED_DB.$selected_table")
                selected_tables_summary+="$SELECTED_DB.$selected_table,"
                echotablesstr+="$selected_table,"
              fi
            else
              invalid_indices+=("$index")
            fi
          else
            invalid_indices+=("$index")
          fi
        done
        if [ ${#invalid_indices[@]} -eq 0 ]; then
          break
        else
          echo "  \033[41;37m${invalid_indices[@]}为无效的数据库表序号\033[0m"
        fi
      done
      echostr+="${echotablesstr%,} 表"
    done
    BACKUP_DB_TABLES="${selected_tables_summary%,}"
  }


  # 检查数据库地址与端口的连通性
  if ! nc -znv $DB_HOST $DB_HOST_PORT >/dev/null 2>&1; then
    echo  -e "  \033[41;37m数据库连接地址: $DB_HOST的$DB_HOST_PORT端口连接测试失败，请检查后重新设置！\033[0m"
    return
  fi

  # 检查数据库用户是否可登录
  connect_res=$(timeout 3 mysql --login-path=$DB_ALIAS -e "select 1;" 2>&1) 
  if [[ $? != 0 ]] ; then
    echo  -e "  \033[41;37m数据库用户$DB_USERNAME无法连接$DB_HOST:$DB_HOST_PORT，请检查用户名密码以及用户客户端IP地址的正确性！\033[0m"
    echo  -e "    \033[41;37m报错信息：$connect_res ！\033[0m"
    return
  fi
  echo "备份类型: "
  echo -e "  \e[;33m1. 备份某些库\033[0m"
  echo -e "  \e[;33m2. 备份某些库中的表\033[0m"
  read "?请选择要备份的类型，默认[2]: " BACKUP_TYPE_OPT
  until [[ -z "$BACKUP_TYPE_OPT" || "$BACKUP_TYPE_OPT" =~ '^[12]$' ]]; do
    echo "$BACKUP_TYPE_OPT: 无效的选项."
    read -p "请重新选择要备份的类型，默认[2]: " BACKUP_TYPE_OPT
  done
  [[ -z "$BACKUP_TYPE_OPT" ]] && BACKUP_TYPE_OPT=2

  # 检查数据库用户是否有备份权限
  timeout 3 mysql --login-path=$DB_ALIAS -e "show grants for $DB_USERNAME@'%';" >/dev/null 2>&1 |grep -E 'RELOAD|ALL PRIVILEGES|REPLICATION CLIENT|SELECT' > /dev/null
  if [[ $? != 0 ]] ; then
    timeout 3 mysql --login-path=$DB_ALIAS -e "show grants for $DB_USERNAME@'192.168.1.3';" >/dev/null 2>&1 |grep -E 'RELOAD|ALL PRIVILEGES|REPLICATION CLIENT|SELECT' > /dev/null
    if [[ $? != 0 ]] ; then
      echo  -e "  \033[41;37m数据库用户$DB_USERNAME在%或192.168.1.3地址上对实例$DB_HOST:$DB_HOST_PORT没有备份权限，请使用高权限用户授予其RELOAD,REPLICATION CLIENT权限！\033[0m\n  \033[41;37m授权SQL: GRANT SELECT,RELOAD,REPLICATION CLIENT for '$DB_USERNAME'@'%' ; GRANT SELECT,RELOAD,REPLICATION CLIENT for '$DB_USERNAME'@'192.168.1.3' ;\033[0m"
      return
    fi
  fi
  local BACKUP_DBS=""
  local BACKUP_DB_TABLES=""
  case "$BACKUP_TYPE_OPT" in
    1)
      show_select_database $DB_HOST $DB_HOST_PORT $DB_USERNAME $DB_PASSWORD
      tput setaf 3
      read "?是否可以开始备份$DB_HOST:$DB_HOST_PORT上的 $BACKUP_DBS 库[Yy/Nn]? " READY_RUN
      until [[ ! -z "$READY_RUN" && ${READY_RUN} =~ ^[yYnN]*$ ]]; do
        read "?是否可以开始备份$DB_HOST:$DB_HOST_PORT上的 $BACKUP_DBS 库[Yy/Nn]? " READY_RUN
      done
      tput sgr0
      if [[ $READY_RUN =~ ^[yY]*$ ]] ;then
        DATE=$(date +"%Y%m%d%M")
        BACKUPFILE_PATH='/Users/test/Desktop'
        mkdir LE_PATH/$DATE/{sql,logs}
        nohup dumpling \
          -c gzip \
          -h ${DB_HOST} \
          -P ${DB_HOST_PORT} \
          -u ${DB_USERNAME} \
          -p ${DB_PASSWORD} \
          --filetype sql \
          --threads 8 \
          -B $BACKUP_DBS \
          -o $BACKUPFILE_PATH/$DATE/sql \
          -F 256MiB \
          --logfile $BACKUPFILE_PATH/$DATE/logs/export-task.log >$BACKUPFILE_PATH/$DATE/logs/dumpling-nohupout.log 2>&1 &
        sleep 2
        gtail -f --pid=$(pidof dumpling) $BACKUPFILE_PATH/$DATE/logs/export-task.log $BACKUPFILE_PATH/$DATE/logs/dumpling-nohupout.log 2>/dev/null
        if grep -q "dump data successfully" $BACKUPFILE_PATH/$DATE/logs/export-task.log ;then
          echo -e "\033[41;37m$DB_HOST上的 $BACKUP_DBS 库已备份在 $BACKUPFILE_PATH/$DATE\033[0m"
          open $BACKUPFILE_PATH/$DATE
        else
          echo -e "\033[41;37m数据库Database备份失败！错误信息如下：\033[0m"
          grep "failed" $BACKUPFILE_PATH/$DATE/logs/dumpling-nohupout.log
        fi
      else
        return
      fi
      ;;
    2)
        local echostr=""
        show_select_database
        show_select_database_tables $BACKUP_DBS
        tput setaf 6
        echo -e ${echostr#*\n}
        tput sgr0
        read "?是否可以开始备份$DB_HOST:$DB_HOST_PORT以上的的库表 [Yy/Nn]? " READY_RUN
        until [[ ! -z "$READY_RUN" && ${READY_RUN} =~ ^[yYnN]*$ ]]; do
          read "?是否可以开始备份$DB_HOST:$DB_HOST_PORT以上的的库表 [Yy/Nn]? " READY_RUN
        done
        if [[ $READY_RUN =~ ^[yY]*$ ]] ;then
          echo 
          tput setaf 3
          DATE=$(date +"%Y%m%d%M")
          BACKUPFILE_PATH='/Users/test/Desktop'
          mkdir -p $BACKUPFILE_PATH/$DATE/{sql,logs}
          nohup dumpling \
            -c gzip \
            -h ${DB_HOST} \
            -P ${DB_HOST_PORT} \
            -u ${DB_USERNAME} \
            -p ${DB_PASSWORD} \
            --filetype sql \
            --threads 8 \
            -T $BACKUP_DB_TABLES \
            -o $BACKUPFILE_PATH/$DATE/sql \
            -F 256MiB \
            --logfile $BACKUPFILE_PATH/$DATE/logs/export-task.log >$BACKUPFILE_PATH/$DATE/logs/dumpling-nohupout.log 2>&1 &
          sleep 2
          gtail -f --pid=$(pidof dumpling) $BACKUPFILE_PATH/$DATE/logs/export-task.log $BACKUPFILE_PATH/$DATE/logs/dumpling-nohupout.log 2>/dev/null
          if grep -q "dump data successfully" $BACKUPFILE_PATH/$DATE/logs/export-task.log ;then
            echo -e "\033[41;37m数据库表已备份成功，文件在$BACKUPFILE_PATH/$DATE\033[0m"
            open $BACKUPFILE_PATH/$DATE
          else
            echo -e "\033[41;37m数据库表备份失败！错误信息如下：\033[0m"
            grep "failed" $BACKUPFILE_PATH/$DATE/logs/dumpling-nohupout.log
          fi
          tput sgr0
        fi
      ;;
  esac
}
```

# 四、实战注意

如果后续使用TiDB lightning恢复备份数据到新集群的话，
- Dumpling备份时不要进行压缩。TiDB lightning无法从压缩备份文件中直接导入到 TIDB。（TiDB lightning v5.1.0是不行，后续版本不知道会不会支持）

  ```bash
  for i in `find /data/dumpling-data/2022042507/sql -name "*sql.gz"` ;do gunzip $i ;done
  ```

- TiDB lightning 开起监控 Web，可以方便查看导入进度

- TiDB lightning 一定要开起断点续传功能，建议使用“file”模式
