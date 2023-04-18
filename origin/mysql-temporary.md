# MySQL临时表

# 一、临时表的种类

## 全局临时表

这种临时表从数据库实例启动后开始生效，在数据库实例销毁后失效。在MySQL里面这种临时表对应的是内存表，即memory引擎。

## 会话级别临时表

这种临时表在用户登录系统成功后生效，在用户退出时失效。在MySQL里的临时表指的就是以create temporary table这样的关键词创建的表。

## 事务级别临时表

这种临时表在事务开始时生效，事务提交或者回滚后失效。 在MySQL里面没有这种临时表，必须利用会话级别的临时表间接实现。

## 检索级别临时表

这种临时表在SQL语句执行之间产生，执行完毕后失效。 在MySQL里面这种临时表不是很固定，跟随MySQL默认存储引擎来变化。
比如默认存储引擎是MyISAM，临时表的引擎就是MyISAM，并且文件生成形式以及数据运作形式和MyISAM一样，只是数据保存在内存里；如果默认引擎是INNODB，那么临时表的引擎就是INNODB，此时它的所有信息都保存在共享表空间ibdata里面。

# 二、MySQL 5.7的临时表空间优化

1. MySQL 5.7 把临时表的数据以及回滚信息（仅限于未压缩表）从共享表空间里面剥离出来，形成自己单独的表空间，参数为innodb_temp_data_file_path。

2. 在MySQL 5.7 中把临时表的相关检索信息保存在系统信息表中：information_schema.innodb_temp_table_info. 而MySQL 5.7之前的版本想要查看临时表的系统信息是没有太好的办法。

**注意：**

虽然INNODB临时表有自己的表空间，但是目前还不能自己定义临时表空间文件的保存路径，只能是继承innodb_data_home_dir。此时如果想要拿其他的磁盘，比如内存盘来充当临时表空间的保存地址，只能用老办法，做软链。

# 三、临时表使用建议

- 设置 innodb_temp_data_file_path 选项，设定文件最大上限，超过上限时，需要生成临时表的SQL无法被执行（一般这种SQL效率也比较低，可借此机会进行优化）。
- 检查 INFORMATION_SCHEMA.INNODB_TEMP_TABLE_INFO，找到最大的临时表对应的线程，kill之即可释放，但 ibtmp1 文件则不能释放（除非重启）。
- 择机重启实例，释放ibtmp1文件，和ibdata1不同，ibtmp1重启时会被重新初始化而 ibdata1 则不可以。
- 定期检查运行时长超过N秒（比如N=300）的SQL，考虑干掉，避免垃圾SQL长时间运行影响业务。

# 四、临时表的创建

```bash
mysql> create temporary table temp1(sid int,sname varchar(10)); 
Query OK, 0 rows affected (0.00 sec) 

mysql> insert into temp1 values(1,'aaa'); 
Query OK, 1 row affected (0.00 sec) 

mysql> select * from temp1; 
+------+-------+ 
| sid | sname  | 
+------+-------+ 
| 1   | aaa    |
+------+-------+ 
1 row in set (0.00 sec)
mysql> show tables;
+-----------------+
| Tables_in_test1 |
+-----------------+
| app01            | 
| app02            | 
| app03            |
+-----------------+ 
3 rows in set (0.00 sec) 
另起一个会话：
mysql> use test1; 
Reading table information for completion of table and column names You can turn off this feature to get a quicker startup with -A Database changed 
mysql> show tables; 
+-----------------+
| Tables_in_test1 |
+-----------------+ 
| app01         |
| app02         | 
| app03         | 
+-----------------+ 
3 rows in set (0.00 sec) 
mysql> select * from temp1; ERROR 1146 (42S02): Table 'test1.temp1' doesn't exist 
退出本次会话：
mysql> use test1; 
Reading table information for completion of table and column names You can turn off this feature to get a quicker startup with -A Database changed 
mysql> select * from temp1;
ERROR 1146 (42S02): Table 'test1.temp1' doesn't exist
```

