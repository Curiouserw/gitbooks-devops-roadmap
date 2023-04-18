# MySQL性能测试

# 一、性能指标

- TPS ：Transactions Per Second ，即数据库每秒执行的事务数，以 commit 成功次数为准。
- QPS ：Queries Per Second ，即数据库每秒执行的 SQL 数（含 insert、select、update、delete 等）。
- RT ：Response Time ，响应时间。包括平均响应时间、最小响应时间、最大响应时间、每个响应时间的查询占比。比较需要重点关注的是，前 95-99% 的最大响应时间。因为它决定了大多数情况下的短板。
- Concurrency Threads ：并发量，每秒可处理的查询请求的数量。

总结来说，实际就是 2 个维度：

- 吞吐量
- 延迟

# 二、测试工具

- ## mysqlslap

  mysqlslap可以模拟服务器的负载，并输出计时信息。在MySQL 4.1或者更新的版本中都可以使用。测试时可以执行并发连接数，并指定SQL 语句（可以在命令行上执行，也可以把SQL 语句写入到参数文件中）。如果没有指定SQL 语句，mysqlslap 会自动生成查询schema 的SELECT 语句。

- ## **sysbench**

  sysbench是一款多线程系统压测工具。它可以根据影响数据库服务器性能的各种因素来评估系统的性能。例如，可以用来测试文件I/O、操作系统调度器、内存分配和传输速度、POSIX 线程，以及数据库服务器等。sysbench 支持Lua 脚本语言，Lua 对于各种测试场景的设置可以非常灵活。sysbench 是我们非常喜欢的一种全能测试工具，支持MySQL、操作系统和硬件的硬件测试。

- ## **MySQL Benchmark Suite （sql-bench）**

  在MySQL 的发行包中也提供了一款自己的基准测试套件，可以用于在不同数据库服务器上进行比较测试。它是单线程的，主要用于测试服务器执行查询的速度。结果会显示哪种类型的操作在服务器上执行得更快。

  　　这个测试套件的主要好处是包含了大量预定义的测试，容易使用，所以可以很轻松地用于比较不同存储引擎或者不同配置的性能测试。其也可以用于高层次测试，比较两个服务器的总体性能。当然也可以只执行预定义测试的子集（例如只测试UPDATE 的性能）。这些测试大部分是CPU 密集型的，但也有些短时间的测试需要大量的磁盘I/O 操作。

  　　这个套件的最大缺点主要有：它是单用户模式的，测试的数据集很小且用户无法使用指定的数据，并且同一个测试多次运行的结果可能会相差很大。因为是单线程且串行执行的，所以无法测试多CPU 的能力，只能用于比较单CPU 服务器的性能差别。使用这个套件测试数据库服务器还需要Perl 和BDB 的支持，相关文档请参考.

- ## **Super Smack**

  Super Smack是一款用于MySQL 和PostgreSQL的基准测试工具，可以提供压力测试和负载生成。这是一个复杂而强大的工具，可以模拟多用户访问，可以加载测试数据到数据库，并支持使用随机数据填充测试表。测试定义在"smack"文件中，smack 文件使用一种简单的语法定义测试的客户端、表、查询等测试要素。

- ## 等等

# 三、sysbench测试实例

参考：[sysbench](benchmark-tools-sysbench.md) 中的测试示例

# 四、mysqlslap测试实例

**mysqlslap是一个mysql官方提供的压力测试工具。以下是比较重要的参数：**

- –defaults-file，配置文件存放位置
- –concurrency，并发数
- –engines，引擎
- –iterations，迭代的实验次数
- –socket，socket文件位置

**自动测试：**

- –auto-generate-sql，自动产生测试SQL
- –auto-generate-sql-load-type，测试SQL的类型。类型有mixed，update，write，key，read。
- –number-of-queries，执行的SQL总数量
- –number-int-cols，表内int列的数量
- –number-char-cols，表内char列的数量
- –create-schema，指定数据库名称
- –query，指定SQL语句，可以定位到某个包含SQL的文件

1、测试同时不同的存储引擎的性能进行对比

```bash
mysqlslap -uroot -p123456 \
-a --concurrency=50,100 --number-of-queries 1000 --iterations=5 --engine=myisam,innodb 
```

2、执行一次测试，分别50和100个并发，执行1000次总查询

```bash
mysqlslap -uroot -p123456 \
-a --concurrency=50,100 --number-of-queries 1000 6
```

3、50和100个并发分别得到一次测试结果(Benchmark)，并发数越多，执行完所有查询的时间越长。为了准确起见，可以多迭代测试几次:

```bash
mysqlslap -uroot -p123456 \
-a --concurrency=50,100 --number-of-queries 1000 --iterations=5 
```
