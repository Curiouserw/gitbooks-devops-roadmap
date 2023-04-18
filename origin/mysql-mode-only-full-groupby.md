# MySQL 5.7 GroupBy的语义检查

> this is incompatible witn select_mode=only_full_group_mode

ONLY_FULL_GROUP_BY是MySQL提供的一个sql_mode，通过这个sql_mode来提供SQL语句GROUP BY合法性的检查，在MySQL的sql_mode是非ONLY_FULL_GROUP_BY语义时。一条select语句，MySQL允许target list中输出的表达式是除聚集函数或group by column以外的表达式，这个表达式的值可能在经过group by操作后变成undefined，例如：

```sql
mysql> create database test charset utf8mb4;
mysql> use test;
mysql> create table tt(id int,count int);
mysql> insert into tt values(1,1),(1,2),(2,3),(2,4);
mysql> select * from tt group by id;
+------+-------+
| id   | count |
+------+-------+
|    1 |     1 |
|    2 |     3 |
+------+-------+
2 rows in set (0.00 sec)
```

**而对于语义限制都比较严谨的多家数据库，如SQLServer、Oracle、PostgreSql都不支持select target list中出现语义不明确的列，这样的语句在这些数据库中是会被报错的，所以从MySQL 5.7版本开始修正了这个语义，就是我们所说的ONLY_FULL_GROUP_BY语义，例如查看MySQL 5.7默认的sql_mode如下：**

```sql
mysql> select @@global.sql_mode;
ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```

**去掉ONLY_FULL_GROUP_BY模式，如下操作：** 

```
mysql> set global sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION';
#或者
mysql> set @@sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION';
上面是改变了全局sql_mode，对于新建的数据库有效。对于已存在的数据库，则需要在对应的数据下执行：
mysql> set sql_mode ='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION';

#再或者
在my.cnf 里面设置
sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION'
在sql_mode 中去掉only_full_group_by 
```

**我们把刚才的查询再次执行：**

```sql
mysql> select id,count from tt group by id;
ERROR 1055 (42000): Expression #2 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'test.tt.count' which is 
not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
```

**刚才通过的查询语句被server拒绝掉了！**

**所以ONLY_FULL_GROUP_BY的语义就是确定select target list中的所有列的值都是明确语义，简单的说来，在ONLY_FULL_GROUP_BY模式下，target list中的值要么是来自于聚集函数的结果，要么是来自于group by list中的表达式的值。但是由于表达式的表现形式非常丰富，对于程序来说，很难精确的确定一些表达式的输出结果是明确的，比如：**

```sql
mysql> select count from tt group by id+count,id;
ERROR 1055 (42000): Expression #1 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'test.tt.count' which is 
not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
```

**在上面的查询语句中，其实count的值也是能被唯一确定的，但是由于程序无法分析出这种复杂的关系，所以这条查询也被拒绝掉了。**

**我们来看下哪些语句是在mysql的ONLY_FULL_GROUP_BY模式下是被支持的。** 

```sql
mysql> select id+1 from tt group by id+1;
+------+
| id+1 |
+------+
|    2 |
|    3 |
+------+
2 rows in set (0.00 sec)
```

**这条语句target list中的id+1和group by中的id+1是严格匹配的，所以mysql认为target list中的id+1是语义明确的，因此该语句可以通过。**

**但下面这条就无法通过了。**

```sql
mysql> select id+1 from tt group by 1+id;
ERROR 1055 (42000): Expression #1 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'test.tt.id' which is 
not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
```

**因此，如果查询语句中的target list, having condition 或者order by list里引用了的表达式不是聚集函数，但是和group by list中的表达式严格匹配，该语句也是合法的（id+1和id+1是严格匹配的，id+1和id+2在mysql认为是不严格匹配的， id+1和1+id也是不严格匹配的）。**

```sql
mysql> select id,max(count) from tt group by count;
ERROR 1055 (42000): Expression #1 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'test.tt.id' which is 
not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by

```

**这条query被server拒绝掉了，因为target list中的id没有出现在聚集函数中，并且也没有出现在group by list中。**

```sql
mysql> select id+1 as a from tt group by a order by id+1;
+------+
| a    |
+------+
|    2 |
|    3 |
+------+
2 rows in set (0.00 sec)
```

**mysql允许target list中对于非聚集函数的alias column被group by、having condition以及order by语句引用(version 5.7中允许having condition引用alias column，version 5.6不支持having condition引用alias column)，从上面两条语句可以看出，group by和order by中引用了alias column，并且其等价于基础列语义。**

```
mysql> select id+count from tt group by id,count;
+----------+
| id+count |
+----------+
|        2 |
|        3 |
|        5 |
|        6 |
+----------+
4 rows in set (0.00 sec)
```

**从上面的语句可以看出，mysql的ONLY_FULL_GROUP_BY模式支持对basic column进行组合但是不支持对于复杂表达式进行组合，这个受限于表达式分析程度。**

**总结一下：**

> **MySQL对于ONLY_FULL_GROUP_BY语义的判断规则是，如果group by list中的表达式是basic column，那么target list中允许出现表达式是group by list中basic column或者alias column的组合结果，如果group by list中的表达式是复杂表达式(非basic column或者alias column)，那么要求target list中的表达式必须能够严格和group by list中的表达式进行匹配，否者这条查询会被认为不合法。**

```sql
# 查看当前SQL MOD
SHOW VARIABLES LIKE 'sql_mode';
# 设置临时会话级别的SQL MOD。会话结束后会自动恢复
set session sql_mode='STRICT_TRANS_TABLES,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION';
```