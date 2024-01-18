# MySQL 数据安全

# 一、简介

数据库中存储着各种各样的数据，但当涉及到用户敏感信息，比如手机号、身份证号码等，应该以密文形式存储在数据库中，mysql数据库有自带的加密函数

- **ENCODE / DECODE**

  传入两个值，一个是要加密的记录，一个是加密和解密的key.**加密之后的二进制字符长度和原始长度是一样的**，以blob类型存储

  > BLOB 类型的字段用于存储二进制数据 MySQL 中，BLOB 是个类型系列，包括：TinyBlob、Blob、MediumBlob、LongBlob，这几个类型之间的唯一区别是在存储文件的最大大小上不同。 MySQL 的四种 BLOB 类型 类型 大小 (单位：字节) TinyBlob 最大 255 Blob 最大 65K Medium。

- **AES_ENCRYPT / AES_DECRYPT**

  这种加密算法使用AES(高级加密标准，Advanced Encryption Standard)，使用key_str加密，key_str的长度可以达到256位，加密的结果是一个二进制的字符串，以blob类型存储

- **DES_ENCRYPT/DES_DECRYPT**

  这种加密方法使用了3DES（三重加密数据算法，听着就知道加密等级比较gap），加密时可以选择使用key_num还是key_str

# 二、数据加密

## 1. 测试加解密过程

```sql
加密
	# 使用字符串“密码”SHA512值的十六进制值作为密码，以AES方式加密字符串“testpasswd”
  select hex(AES_ENCRYPT('testpasswd', HEX(SHA2('密码',512))));

解密
	# 获取密码的Hex值
	SELECT HEX(SHA2('密码',512))
	# 解密加密过后的字段值
	SELECT AES_DECRYPT(UNHEX('加密后的字段值'),'密码的Hex值');
```

## 2. 创建副本表，批量修改数据

```sql
set @saltpasswd=HEX(SHA2('密码',512))

# 创建副本表
CREATE  TABLE IF NOT EXISTS 副本表 (LIKE 源表); 

# 修改副本表字段长度。字段长度太小，数值会被截断。长度最小50
alter table 副本表 modify column 长度小的字段 varchar(50);

# 复制源表数据并加密指定字段到副本表中
INSERT INTO 副本表 (
	`字段名1`,
	`字段名2`,
  `....`,
  `副本表的所有字段`
) SELECT
	`字段名1`,
	`字段名2`,
  hex(AES_ENCRYPT(要加密的字段1,@saltpasswd)),
  `....`,
  hex(AES_ENCRYPT(要加密的字段2,@saltpasswd)),
  `源表的所有字段`
FROM
	源表;
```

## 3. 查询加密的数据

```sql
set @saltpasswd=HEX(SHA2('密码',512));

SELECT
	*,
	AES_DECRYPT( unhex( 加密的字段 ), @saltpasswd ) AS decrypt_context
FROM
	副本表;
```

## 4. 测试

```bash
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(50) NOT NULL AUTO_INCREMENT,
  `phone` varchar(50) DEFAULT NULL,
  `sfz` varchar(50) DEFAULT NULL,
  `birthday` varchar(50) DEFAULT NULL
  PRIMARY KEY (`id`)
) ENGINE=InnoDB 

insert into user(name,phone,sfz,birthday) values('test1','13111110112','310000000000','20001118');
insert into user(name,phone,sfz,birthday) values('test1','13111110113','310000000001','20000118');

set @saltpasswd=HEX(SHA2('密码',512))

insert into user(phone) values(hex(AES_ENCRYPT('13111110111',@saltpasswd)));
insert into user(phone) values(hex(AES_ENCRYPT('13111110112',@saltpasswd)));
```

# 三、备份文件加密

参考：[Xtrabackup备份](xtrabackup-backup.md)第五章第一小节

# 四、用户权限安全

| 场景         | 用户名命名规则  | IP限制                     | 权限                                          | 模式          | 备注                                       |
| ------------ | --------------- | -------------------------- | --------------------------------------------- | ------------- | ------------------------------------------ |
| 应用使用     | app_*           | 应用所在服务器 IP 地址     | select, insert, delete, update                | DML、DQL      | 应用账号不能创建表，更改表结构             |
| 管理工具使用 | yearning_*      | 管理工具所在服务器 IP 地址 | select, insert, delete, create, alter, update | DDL、DML、DQL | 创建表，更改表结构只能在审计管理工具上执行 |
| 个人直连使用 | 姓名简拼_*      | 个人电脑出口 IP 地址       | select                                        | DQL           |                                            |
| 报表程序使用 | report_*        | 报表程序所在服务器 IP 地址 | select                                        | DQL           |                                            |
| 运维工具使用 | tool_工具名简拼 | 运维工具所在服务器 IP 地址 | 不同用途不同权限                              |               | 禁止给 Drop                                |

- 密码位数至少10位以上，必须包含英文大小写、数字、特殊字符(_#@*-)等。
- 所有用户授予权限到具体库，禁止授权*****所有库

# 参考

- http://www.gimoo.net/t/1712/5a4080a95a1fd.html
- https://dev.mysql.com/doc/refman/5.6/en/encryption-functions.html#function_aes-decrypt