# 常用存储过程

## 1、修改Database中所有表的所有字段的编码格式（mysql）

```sql
delimiter $$
CREATE PROCEDURE Test ()
begin
    DECLARE cnt VARCHAR(100); -- 声明变量用来记录查询出的表名
    DECLARE i int; -- 循环条件，同时可以用来标记表第几张表
    set i = 0;  
    -- 循环开始
    while i < 77 do    -- 这里是77是因为我的数据库中表的数量是77，想不写死可以通过再定义一个变量，动态赋值
        select table_name into @cnt from information_schema.`TABLES` where TABLE_SCHEMA = 'redmine' limit i,1;    -- TABLE_SCHEMA是DataBase名
        set @sql = concat("alter table ", @cnt, " convert to character set utf8mb4");  -- 拼接，注意语句中的空格
        prepare stmt from @sql; -- 预处理
            execute stmt;   -- 执行
        deallocate prepare stmt; -- 释放
        set i = i + 1;  
    end while;  -- 循环结束，注意分号
end$$
delimiter ;
call Test ;
```

参考：https://blog.csdn.net/LUNG108/article/details/78285054

