# SQLCipher



# 一、简介

SQLCipher是一个在SQLite基础之上进行扩展的开源数据库，它主要是在SQLite的基础之上增加了数据加密功能。SQLCipher采用的是256位AES对称加密算法

Github：https://github.com/sqlcipher/sqlcipher



# 二、安装

## MacOS

```bash
brew install sqlcipher
```

# 三、操作

## 1、创建加密数据库

```sqlite
PRAGMA key = 'thisiskey';
create table test (id integer, name text);
```

## 2、打开加密数据库

```sqlite
PRAGMA key = 'thisiskey';
.schema
```

## 3、修改数据库密码

```sqlite
PRAGMA KEY = 'thisiskey';
PRAGMA REKEY = 'newkey';
```

## 4、加密已有的数据库

```sqlite
ATTACH DATABASE 'encrypted.db' AS encrypted KEY 'thisiskey';
SELECT sqlcipher_export('encrypted');
DETACH DATABASE encrypted;
```

# 参考：

- https://www.modb.pro/db/65057
- https://blog.csdn.net/u010333084/article/details/104984553