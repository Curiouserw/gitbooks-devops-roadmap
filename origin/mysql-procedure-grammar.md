# MySQL存储过程

# 一、**存储过程简介**

  SQL语句需要先编译然后执行，而存储过程（Stored Procedure）是一组为了完成特定功能的SQL语句集，经编译后存储在数据库中，用户通过指定存储过程的名字并给定参数（如果该存储过程带有参数）来调用执行它。

  存储过程是数据库的一个重要的功能，MySQL 5.0以前并不支持存储过程，这使得MySQL在应用上大打折扣。好在MySQL 5.0开始支持存储过程，这样即可以大大提高数据库的处理速度，同时也可以提高数据库编程的灵活性。

  存储过程是可编程的函数，在数据库中创建并保存，可以由SQL语句和控制结构组成。当想要在不同的应用程序或平台上执行相同的函数，或者封装特定功能时，存储过程是非常有用的。数据库中的存储过程可以看做是对编程中面向对象方法的模拟，它允许控制数据的访问方式。

存储过程的优点：

- **增强SQL语言的功能和灵活性**：存储过程可以用控制语句编写，有很强的灵活性，可以完成复杂的判断和较复杂的运算。
- **标准组件式编程**：存储过程被创建后，可以在程序中被多次调用，而不必重新编写该存储过程的SQL语句。而且数据库专业人员可以随时对存储过程进行修改，对应用程序源代码毫无影响。
- **较快的执行速度**：如果某一操作包含大量的Transaction-SQL代码或分别被多次执行，那么存储过程要比批处理的执行速度快很多。因为存储过程是预编译的。在首次运行一个存储过程时查询，优化器对其进行分析优化，并且给出最终被存储在系统表中的执行计划。而批处理的Transaction-SQL语句在每次运行时都要进行编译和优化，速度相对要慢一些。
- **减少网络流量**：针对同一个数据库对象的操作（如查询、修改），如果这一操作所涉及的Transaction-SQL语句被组织进存储过程，那么当在客户计算机上调用该存储过程时，网络中传送的只是该调用语句，从而大大减少网络流量并降低了网络负载。
- **作为一种安全机制来充分利用**：通过对执行某一存储过程的权限进行限制，能够实现对相应的数据的访问权限的限制，避免了非授权用户对数据的访问，保证了数据的安全。

  

1. 存储过程的定义信息保存在数据字典表information_schema.routines中
2. mysql中存储过程和函数的语法非常接近所以就放在一起，主要区别就是函数必须有返回值（return），并且函数的参数只有IN类型而存储过程有IN、OUT、INOUT这三种类型。

   

# 二、存储过程语法规则

```
CREATE [DEFINER = { user | CURRENT_USER }]  PROCEDURE 过程名([[IN|OUT|INOUT] 参数名 数据类型[,[IN|OUT|INOUT] 参数名 数据类型…]]) 
    [特性 ...]  过程体:(有效的SQL语句,记得末尾加;)

DEFINER：用于指明存储过程是由哪个用户定义的，默认存储过程的定义者是存储过程，跟存储过程的使用权限无关
        （如果在创建存储过程时指定为root@%，将有可能导致root@localhost在使用存储过程时出现权限问题。-- root@localhost是默认值）

形参：
    IN 输入参数:表示该参数的值必须在调用存储过程时指定，在存储过程中修改该参数的值不能被返回，为默认值 
    OUT 输出参数:该值可在存储过程内部被改变，并可返回 
    INOUT 输入输出参数:调用时指定，并且可被改变和返回
特性:
    COMMENT 'string'  | LANGUAGE SQL  | [NOT] DETERMINISTIC  | { CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA }  | SQL SECURITY { DEFINER | INVOKER }
    -- COMMENT：给存储过程添加注释信息
    -- LANGUAGE：用来说明语句部分是SQL语句，未来可能会支持其它类型的语句。
    -- SQL SECURITY { DEFINER | INVOKER }指明谁有权限来执行： DEFINER表示只有定义者自己才能够执行；INVOKER表示调用者可以执行。
    -- [NOT] DETERMINISTIC：
        --如果程序或线程总是对同样的输入参数产生同样的结果，则被认为它是“确定的”，否则就是“非确定”的。如果既没有给定DETERMINISTIC也没有给定NOT DETERMINISTIC，默认的就是NOT DETERMINISTIC（非确定的）
    -- NO SQL：表示子程序不包含SQL语句。
    -- READS SQL DATA：表示子程序包含读数据的语句，但不包含写数据的语句。
    -- MODIFIES SQL DATA：表示子程序包含写数据的语句。
```



**存储过程体包含了在过程调用时必须执行的语句，例如：dml、ddl语句，if-then-else和while-do语句、声明变量的declare语句等,使用DELIMITER $$ 命令将语句的结束符号从分号 ; 临时改为两个$$，使得过程体中使用的分号被直接传递到服务器，而不会被客户端（如mysql）解释**

```
-- 简单过程体
    begin
        dml、ddl语句，if-then-else和while-do语句、声明变量的declare语句等;
    end$$    

-- 加标签的过程体（）
    [begin_label:] BEGIN
　    　dml、ddl语句，if-then-else和while-do语句、声明变量的declare语句等;
    END [end_label]

--嵌套的过程体 （ ）
    BEGIN
　    　BEGIN
　    　　　BEGIN
　　　    　　　statements; 
　　    　　END
　    　END
    END
```

## 变量作用域

内部的变量在其作用域范围内享有更高的优先权，当执行到end。变量时，内部变量消失，此时已经在其作用域外，变量不再可见了，应为在存储 
过程外再也不能找到这个申明的变量，但是你可以通过out参数或者将其值指派 
给会话变量来保存其值。

```
mysql > DELIMITER //  
mysql > CREATE PROCEDURE proc3()  
     -> begin 
     -> declare x1 varchar(5) default 'outer';  
     -> begin 
     -> declare x1 varchar(5) default 'inner';  
     -> select x1;  
     -> end;  
     -> select x1;  
    -> end;  
    -> //  
mysql > DELIMITER ;  
```

## 条件语句

### if-then -else语句

```
 mysql > DELIMITER //  
 mysql > CREATE PROCEDURE proc2(IN parameter int)  
      -> begin 
      -> declare var int;  
      -> set var=parameter+1;  
      -> if var=0 then 
      -> insert into t values(17);  
      -> end if;  
      -> if parameter=0 then 
     -> update t set s1=s1+1;  
     -> else 
     -> update t set s1=s1+2;  
     -> end if;  
     -> end;  
     -> //  
mysql > DELIMITER ;  
```

### case语句：

```
mysql > DELIMITER //  
mysql > CREATE PROCEDURE proc3 (in parameter int)  
     -> begin 
     -> declare var int;  
     -> set var=parameter+1;  
     -> case var  
     -> when 0 then   
     -> insert into t values(17);  
     -> when 1 then   
    -> insert into t values(18);  
    -> else   
    -> insert into t values(19);  
    -> end case;  
    -> end;  
    -> //  
mysql > DELIMITER ; 
case
        when var=0 then
               insert into t values(30);
        when var>0 then
        when var<0 then
        else
end case
```

## 循环语句

### while ···· end while

```
mysql > DELIMITER //  
mysql > CREATE PROCEDURE proc4()  
     -> begin 
     -> declare var int;  
     -> set var=0;  
     -> while var<6 do  
     -> insert into t values(var);  
     -> set var=var+1;  
     -> end while;  
    -> end;  
    -> //  
mysql > DELIMITER ; 
		while条件 do
			--循环体
		endwhile
```

### repeat···· end repeat

它在执行操作后检查结果，而while则是执行前进行检查。

```
mysql > DELIMITER //  
mysql > CREATE PROCEDURE proc5 ()  
     -> begin   
     -> declare v int;  
     -> set v=0;  
     -> repeat  
     -> insert into t values(v);  
     -> set v=v+1;  
     -> until v>=5  
    -> end repeat;  
    -> end;  
    -> //  
mysql > DELIMITER ;  
		repeat
		--循环体
		until循环条件     
		endrepeat;
```

### loop ·····endloop

loop循环不需要初始条件，这点和while 循环相似，同时和repeat循环一样不需要结束条件, leave语句的意义是离开循环。

```
mysql > DELIMITER //  
mysql > CREATE PROCEDURE proc6 ()  
     -> begin 
     -> declare v int;  
     -> set v=0;  
     -> LOOP_LABLE:loop  
     -> insert into t values(v);  
     -> set v=v+1;  
     -> if v >=5 then 
    -> leave LOOP_LABLE;  
    -> end if;  
    -> end loop;  
    -> end;  
    -> //  
mysql > DELIMITER ;  
```

## LABLES 标号

标号可以用在begin repeat while 或者loop 语句前，语句标号只能在合法的语句前面使用。可以跳出循环，使运行指令达到复合语句的最后一步。

## ITERATE 迭代

通过引用复合语句的标号,来从新开始复合语句

```
mysql > DELIMITER //  
mysql > CREATE PROCEDURE proc10 ()  
     -> begin 
     -> declare v int;  
     -> set v=0;  
     -> LOOP_LABLE:loop  
    -> if v=3 then   
    -> set v=v+1;  
       -> ITERATE LOOP_LABLE;  
       -> end if;  
    -> insert into t values(v);  
    -> set v=v+1;  
       -> if v>=5 then 
       -> leave LOOP_LABLE;  
    -> end if;  
    -> end loop;  
    -> end;  
    -> //  
mysql > DELIMITER ; 
```

# 三、存储过程的基本函数

## 字符串类

```
CHARSET(str) //返回字串字符集
CONCAT (string2 [,... ]) //连接字串
INSTR (string ,substring ) //返回substring首次在string中出现的位置,不存在返回0
LCASE (string2 ) //转换成小写
LEFT (string2 ,length ) //从string2中的左边起取length个字符
LENGTH (string ) //string长度
LOAD_FILE (file_name ) //从文件读取内容
LOCATE (substring , string [,start_position ] ) 同INSTR,但可指定开始位置
LPAD (string2 ,length ,pad ) //重复用pad加在string开头,直到字串长度为length
LTRIM (string2 ) //去除前端空格
REPEAT (string2 ,count ) //重复count次
REPLACE (str ,search_str ,replace_str ) //在str中用replace_str替换search_str
RPAD (string2 ,length ,pad) //在str后用pad补充,直到长度为length
RTRIM (string2 ) //去除后端空格
STRCMP (string1 ,string2 ) //逐字符比较两字串大小,
SUBSTRING (str , position [,length ]) //从str的position开始,取length个字符,
注：mysql中处理字符串时，默认第一个字符下标为1，即参数position必须大于等于1 
1.  mysql> select substring('abcd',0,2);  
2.  +-----------------------+  
3.  | substring('abcd',0,2) |  
4.  +-----------------------+  
5.  |                       |  
6.  +-----------------------+  
7.  1 row in set (0.00 sec)  
8.   
9.  mysql> select substring('abcd',1,2);  
10.+-----------------------+  
11.| substring('abcd',1,2) |  
12.+-----------------------+  
13.|     ab                |  
14.+-----------------------+  
15.1 row in set (0.02 sec)  
TRIM([[BOTH|LEADING|TRAILING][padding] FROM]string2) //去除指定位置的指定字符
UCASE (string2 ) //转换成大写
RIGHT(string2,length) //取string2最后length个字符
SPACE(count) //生成count个空格
```

## 数学类

```
ABS (number2 ) //绝对值
BIN (decimal_number ) //十进制转二进制
CEILING (number2 ) //向上取整
CONV(number2,from_base,to_base) //进制转换
FLOOR (number2 ) //向下取整
FORMAT (number,decimal_places ) //保留小数位数
HEX (DecimalNumber ) //转十六进制
注：HEX()中可传入字符串，则返回其ASC-11码，如HEX('DEF')返回4142143
也可以传入十进制整数，返回其十六进制编码，如HEX(25)返回19
LEAST (number , number2 [,..]) //求最小值
MOD (numerator ,denominator ) //求余
POWER (number ,power ) //求指数
RAND([seed]) //随机数
ROUND (number [,decimals ]) //四舍五入,decimals为小数位数]
注：返回类型并非均为整数，如：
(1)默认变为整形值
1.  mysql> select round(1.23);  
2.  +-------------+  
3.  | round(1.23) |  
4.  +-------------+  
5.  |           1 |  
6.  +-------------+  
7.  1 row in set (0.00 sec)  
8.   
9.  mysql> select round(1.56);  
10.+-------------+  
11.| round(1.56) |  
12.+-------------+  
13.|           2 |  
14.+-------------+  
15.1 row in set (0.00 sec) 


(2)可以设定小数位数，返回浮点型数据
1.  mysql> select round(1.567,2);  
2.  +----------------+  
3.  | round(1.567,2) |  
4.  +----------------+  
5.  |           1.57 |  
6.  +----------------+  
7.  1 row in set (0.00 sec) 
SIGN (number2 ) //
```

## 日期时间类

```
ADDTIME (date2 ,time_interval )//将time_interval加到date2
CONVERT_TZ (datetime2 ,fromTZ ,toTZ ) //转换时区
CURRENT_DATE ( ) //当前日期
CURRENT_TIME ( ) //当前时间
CURRENT_TIMESTAMP ( ) //当前时间戳
DATE (datetime ) //返回datetime的日期部分
DATE_ADD (date2 , INTERVAL d_value d_type ) //在date2中加上日期或时间
DATE_FORMAT (datetime ,FormatCodes ) //使用formatcodes格式显示datetime
DATE_SUB (date2 , INTERVAL d_value d_type ) //在date2上减去一个时间
DATEDIFF (date1 ,date2 ) //两个日期差
DAY (date ) //返回日期的天
DAYNAME (date ) //英文星期
DAYOFWEEK (date ) //星期(1-7) ,1为星期天
DAYOFYEAR (date ) //一年中的第几天
EXTRACT (interval_name FROM date ) //从date中提取日期的指定部分
MAKEDATE (year ,day ) //给出年及年中的第几天,生成日期串
MAKETIME (hour ,minute ,second ) //生成时间串
MONTHNAME (date ) //英文月份名
NOW ( ) //当前时间
SEC_TO_TIME (seconds ) //秒数转成时间
STR_TO_DATE (string ,format ) //字串转成时间,以format格式显示
TIMEDIFF (datetime1 ,datetime2 ) //两个时间差
TIME_TO_SEC (time ) //时间转秒数]
WEEK (date_time [,start_of_week ]) //第几周
YEAR (datetime ) //年份
DAYOFMONTH(datetime) //月的第几天
HOUR(datetime) //小时
LAST_DAY(date) //date的月的最后日期
MICROSECOND(datetime) //微秒
MONTH(datetime) //月
MINUTE(datetime) //分返回符号,正负或0
SQRT(number2) //开平方
```

## MySql分页存储过程

MySql测试版本：5.0.41-community-nt

```
DROP PROCEDURE IF EXISTS pr_pager;
CREATE PROCEDURE pr_pager(

    IN    p_table_name        VARCHAR(1024),        
    IN    p_fields            VARCHAR(1024),        
    IN    p_page_size            INT,                
    IN    p_page_now            INT,                
    IN    p_order_string        VARCHAR(128),        
    IN    p_where_string        VARCHAR(1024),        
     OUT    p_out_rows            INT                    

)
    NOT DETERMINISTIC
     SQL SECURITY DEFINER
     COMMENT '分页存储过程'

BEGIN


    DECLARE m_begin_row INT DEFAULT 0;
    DECLARE m_limit_string CHAR(64);


    SET m_begin_row = (p_page_now - 1) * p_page_size;
    SET m_limit_string = CONCAT(' LIMIT ', m_begin_row, ', ', p_page_size);

    SET @COUNT_STRING = CONCAT('SELECT COUNT(*) INTO @ROWS_TOTAL FROM ', p_table_name, ' ', p_where_string);
    SET @MAIN_STRING = CONCAT('SELECT ', p_fields, ' FROM ', p_table_name, ' ', p_where_string, ' ', p_order_string,m_limit_string);


    PREPARE count_stmt FROM @COUNT_STRING;
    EXECUTE count_stmt;
    DEALLOCATE PREPARE count_stmt;
    SET p_out_rows = @ROWS_TOTAL;

    PREPARE main_stmt FROM @MAIN_STRING;
    EXECUTE main_stmt;
    DEALLOCATE PREPARE main_stmt;

END;
```

## **调用**

```
mysql> call pr_pager("t","var",3,3,"","",@result);
mysql> call pr_pager("t","var",3,2,"","",@result);
```

# 四、存储过程操作

## 1、查看存储过程

```
#查询存储过程
SELECT name FROM mysql.proc WHERE db='数据库名' and `type` = 'PROCEDURE' \G;
SELECT routine_name FROM information_schema.routines WHERE routine_schema='数据库名';

SHOW PROCEDURE STATUS WHERE db='数据库名';

#查看存储过程详细信息
SHOW CREATE PROCEDURE 数据库.存储过程名;
```

## 2、调用存储过程

```
call 数据库.存储过程名;
```

## 3、删除存储过程

```
DROP PROCEDURE [IF EXISTS] db_name.sp_name;

#如果存储过程或存储函数不存在时，仍然进行删除，可以使用IF EXISTS子句，它可以防止发生错误，产生一个用SHOW WARNINGS查看的警告。
```

## 4、修改存储过程

```
-- 只能改变存储过程的特征，不能修改过程的参数以及过程体。如果想做这样的修改，必须先使用DROP PROCEDURE 删除过程，然后使用and CREATE PROCEDURE重建过程。
ALTER {PROCEDURE | FUNCTION} sp_name [特征 ...]
```