# MySQL的用户及权限管理

# 一、用户管理

## 创建用户

### 方式1：CREATE USER命令

必须要拥有CREATE USER权限。

```
CREATE USER user[IDENTIFIED BY [PASSWORD] 'password'],
[user[IDENTIFIED BY [PASSWORD] 'password']]...

CREATE USER 'name'@'%' IDENTIFIED BY 'pwd';
```

### 方式2：INSERT方式

必须拥有mysql.user表的INSERT权限。另外，ssl_cipher、x509_issuer、x509_subject等必须要设置值

```
INSERT INTO mysql.user(Host,User,Password,ssl_cipher,x509_issuer,x509_subject) VALUES('%','name',PASSWORD('pwd'),'','','')
```

## 删除用户

### 方式1：DROP USER命令

需要拥有DROP USER权限。

```
DROP USER user[,user]…

user是需要删除的用户，由用户名(User)和主机名(Host)构成。
DROP USER name@'1.1.1.1'
```

### 方式2：DELETE方式

```sql
DELETE FROM mydb.user WHERE Host = '% AND User = 'admin';
```

## 重命名用户

```bash
rename user 'jack'@'%' to 'jim'@'%'; 
```

# 二、权限管理

## 1、用户权限验证过程

- 第一阶段：服务器首先会检查你是否允许连接。因为创建用户的时候会加上主机限制，可以限制成本地、某个IP、某个IP段、以及任何地方等，只允许你从配置的指定地方登陆。
- 第二阶段：如果你能连接，Mysql会检查你发出的每个请求，看你是否有足够的权限实施它。比如你要更新某个表、或者查询某个表，Mysql会查看你对哪个表或者某个列是否有权限。

## 2、用户授权原则

  1、只授予能满足需要的最小权限，防止用户干坏事。比如用户只是需要查询，那就只给select权限就可以了，不要给用户赋予update、insert或者delete权限。

  2、创建用户的时候限制用户的登录主机，一般是限制成指定IP或者内网IP段。

  3、初始化数据库的时候删除没有密码的用户。安装完数据库的时候会自动创建一些用户，这些用户默认没有密码。

  4、为每个用户设置满足密码复杂度的密码。

  5、定期清理不需要的用户。回收权限或者删除用户。

## 3、GRANT命令授权语法

```bash
GRANT
    priv_type [(column_list)]
      [, priv_type [(column_list)]] ...
    ON [object_type] priv_level
    TO user [auth_option] [, user [auth_option]] ...
    [REQUIRE {NONE | tls_option [[AND] tls_option] ...}]
    [WITH {GRANT OPTION | resource_option} ...]
    
    
object_type: {
    TABLE
  | FUNCTION
  | PROCEDURE
}

priv_level: {
    *
  | *.*
  | db_name.*
  | db_name.tbl_name
  | tbl_name
  | db_name.routine_name
}

user由用户名(User)和主机名(Host)构成,中间用@隔开，最好加上单引号

auth_option: {
    IDENTIFIED BY 'auth_string'
  | IDENTIFIED WITH auth_plugin
  | IDENTIFIED WITH auth_plugin BY 'auth_string'
  | IDENTIFIED WITH auth_plugin AS 'auth_string'
  | IDENTIFIED BY PASSWORD 'auth_string'
}

tls_option: {
    SSL
  | X509
  | CIPHER 'cipher'
  | ISSUER 'issuer'
  | SUBJECT 'subject'
}

resource_option: {
  | MAX_QUERIES_PER_HOUR count
  | MAX_UPDATES_PER_HOUR count
  | MAX_CONNECTIONS_PER_HOUR count
  | MAX_USER_CONNECTIONS count
}    
```

### Mysql权限层级

- **user表**：全局层级

  - 存储用户记录的表。关键字段有Host、User、Password。

  - 创建对所有表有SELECT操作权限的用户

    ```bash
    GRANT SELECT ON *.* TO name@'1.1.1.1' IDENTIFIED BY 'pwd';
    ```

- **db表**：数据库层级

  - 存储该用户对一个数据库所有的操作权限。关键字段有Host、User、Db。

  - 授予所有权限

    ```bash
    GRANT ALL ON mydb.* TO name@'1.1.1.1' IDENTIFIED BY 'pwd';
    ```

- **tables_priv表**：表层级
  记录了对一个表的单独授权记录.关键字段有Host、User、Db、Table_name、Table_priv、Column_priv。

  - 当授权all在某张表的时候，Table_priv会有如下授权记录：

    - Select,Insert,Update,Delete,Create,Drop,References,Index,Alter,Create View,Show view,Trigger。

  - 单独授权表的某一列，会记录在此表的Column_priv里  
    - GRANT UPDATE(age) ON mydb.user TO name@'1.1.1.1';
    - GRANT SELECT(birthday) ON mydb.user TO name@'1.1.1.1';    
    - 此时会在另一张表columns_priv表中留下单独授权记录
  
- **columns_priv表**：列层级
  记录对表的某一列的授权记录。关键字段Host、User、Db、Table_name、Column_name。
- **procs_priv表**：子程序层级
  可以对存储过程和存储函数进行权限设置。关键字段Host、User、proc_priv

### MySQL 5.7的权限列表

官方文档：https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html

| Privilege                                                    | Grant Table Column           | Context                               |
| ------------------------------------------------------------ | ---------------------------- | ------------------------------------- |
| [`ALL [PRIVILEGES\]`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_all) | Synonym for “all privileges” | Server administration                 |
| [`ALTER`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_alter) | `Alter_priv`                 | Tables                                |
| [`ALTER ROUTINE`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_alter-routine) | `Alter_routine_priv`         | Stored routines                       |
| [`CREATE`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_create) | `Create_priv`                | Databases, tables, or indexes         |
| [`CREATE ROUTINE`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_create-routine) | `Create_routine_priv`        | Stored routines                       |
| [`CREATE TABLESPACE`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_create-tablespace) | `Create_tablespace_priv`     | Server administration                 |
| [`CREATE TEMPORARY TABLES`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_create-temporary-tables) | `Create_tmp_table_priv`      | Tables                                |
| [`CREATE USER`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_create-user) | `Create_user_priv`           | Server administration                 |
| [`CREATE VIEW`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_create-view) | `Create_view_priv`           | Views                                 |
| [`DELETE`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_delete) | `Delete_priv`                | Tables                                |
| [`DROP`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_drop) | `Drop_priv`                  | Databases, tables, or views           |
| [`EVENT`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_event) | `Event_priv`                 | Databases                             |
| [`EXECUTE`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_execute) | `Execute_priv`               | Stored routines                       |
| [`FILE`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_file) | `File_priv`                  | File access on server host            |
| [`GRANT OPTION`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_grant-option) | `Grant_priv`                 | Databases, tables, or stored routines |
| [`INDEX`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_index) | `Index_priv`                 | Tables                                |
| [`INSERT`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_insert) | `Insert_priv`                | Tables or columns                     |
| [`LOCK TABLES`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_lock-tables) | `Lock_tables_priv`           | Databases                             |
| [`PROCESS`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_process) | `Process_priv`               | Server administration                 |
| [`PROXY`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_proxy) | See `proxies_priv` table     | Server administration                 |
| [`REFERENCES`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_references) | `References_priv`            | Databases or tables                   |
| [`RELOAD`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_reload) | `Reload_priv`                | Server administration                 |
| [`REPLICATION CLIENT`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_replication-client) | `Repl_client_priv`           | Server administration                 |
| [`REPLICATION SLAVE`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_replication-slave) | `Repl_slave_priv`            | Server administration                 |
| [`SELECT`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_select) | `Select_priv`                | Tables or columns                     |
| [`SHOW DATABASES`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_show-databases) | `Show_db_priv`               | Server administration                 |
| [`SHOW VIEW`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_show-view) | `Show_view_priv`             | Views                                 |
| [`SHUTDOWN`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_shutdown) | `Shutdown_priv`              | Server administration                 |
| [`SUPER`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_super) | `Super_priv`                 | Server administration                 |
| [`TRIGGER`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_trigger) | `Trigger_priv`               | Tables                                |
| [`UPDATE`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_update) | `Update_priv`                | Tables or columns                     |
| [`USAGE`](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_usage) | Synonym for “no privileges”  | Server administration                 |

## 4、查看用户权限

查看当前用户的权限

```sql
show grants; 
```

查看某个用户的所有权限

```sql
show grants for 'jack'; 

show grants for admin'@'localhost';
```

## 5、REVOKE命令回收权限语法

```bash
REVOKE
    priv_type [(column_list)]
      [, priv_type [(column_list)]] ...
    ON [object_type] priv_level
    FROM user [, user] ...

REVOKE ALL [PRIVILEGES], GRANT OPTION
    FROM user [, user] ...

REVOKE PROXY ON user
    FROM user [, user] ..
```

