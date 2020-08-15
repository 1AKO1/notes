# 1 初识数据库

## 1.1 连接数据库

``` mysql
mysql -u root -p123456

update mysql.user set authentication_string=password('123456') where user='root' ahnd Host='localhost'; -- 修改密码

flush privileges; -- 刷新权限

-- ---------------------------------
-- 所有的语句都用;结尾
show databases; -- 展示所有的数据库

user shcool -- 切换数据库 use 数据库名
Database changed

show tables; -- 查看数据库中所有的表

describe student; -- 显示数据库中表的所有信息

crate database westos; -- 创建一个数据库

exit; -- 退出连接

-- 单行注释(SQL 本来的注释)

/*
sql的多行注释
*/
```





# 2 操作数据库

操作数据库 > 操作数据库 > 操作数据库中表的数据

SQL语句不分大小写



## 2.1 操作数据库

- 创建数据库

    ```mysql
    create database [if not exists] westos;
    ```

- 删除数据库

    ```mysql
    drop database [if exists] westos;
    ```

- 使用数据库

    ```mysql
    use school; -- tab 上面 ` ` , 如果表明或者字段名是一个特殊字符，就要带 ``
    ```

- 查看数据库

    ```mysql
    show school;  -- 查看所有的数据库
    ```

    

## 2.2 数据库的列类型

>   数值

- tinyint             非常小的数据  1个字节
- smallint            较小的数据    2个字节
- mediumint  中等大小的数据  3个字节
- **int                      标准的整数     4个字节**
- bigint                较大的数据       8个字节
- float                  浮点数               4个字节
- double             浮点数                8个字节（精度问题！）
- decimal            字符串形式浮点数   金融计算的时候，一般使用这个， 字符串可以存很长很长的

>   字符串

- char   字符串固定大小  0~255
- **varchar 可变字符串   0~65535  常用 String**
- tinytext          微型文本           2^8 - 1
- **text                字符串                2 ^ 16 - 1 保存大文本**

>   日期

java.util.Data



-   date  	YYYY-MM-DD        日期格式
-   time      HH：mm：ss         时间格式
-   **datatime   YYYY-MM-DD  HH：mm：ss  最常用的时间格式**
-   **timestamp   时间戳， 1970.1.1到现在的毫秒数     常用**
-   year  年份



>   null

-   没有值，未知
-   ==注意，不要使用null运算，结果也是null==





## 2.3 数据库的字段属性（重点）

-   Unsigned
    -   无符号的整数
    -   声明该列不能为负数
-   zerofill
    -   0填充的
    -   不足的位数，使用0来填充， int（3）， 5 ---- 005
-   自增
    -   通常理解为自增，自动在上一条记录的基础上+1（默认）
    -   通常用来设计唯一的主键 index， 类型必须是整数类型
    -   可以自定义设计主键自增的起始值和步长
-   非空 NULL not null
    -   如果为not null， 如果不给他赋值，就会报错
    -   null， 如果不给他赋值，就会填充为null
-   默认





拓展：

```mysql
/* 每一个表，都必须存在以下五个字段！ 未来做项目用， 表示一个记录的存在意义 

id 主键
`version`	乐观锁
is_delete	伪删除
gmt_create	创建时间
gmt_update	修改时间
*/
```



## 2.4 创建数据库表

```mysql
/*
目标：创建一个school数据库
创建学生表(列,字段) 使用SQL 创建
学号 int 密码 varchar(20) 姓名 varchar(2) 出生日期(datatime), 家庭住址， email

注意：
使用英文(), 表的名称和字段 使用``括起来
字符串使用' ' 括起来
AUTO_INCREMENT自增
所有的语句后面加, 
PRIMARY KEY 主键， 一般一个标志有一个唯一主键， 放在最下面加
*/
CREATE TABLE IF NOT EXISTS `student`(
	`id` INT(4) NOT NULL AUTO_INCREMENT COMMENT '学号',
	`name` VARCHAR(30) NOT NULL DEFAULT '匿名' COMMENT '姓名',
	`pwd` VARCHAR(20) NOT NULL DEFAULT '123456' COMMENT '密码',
	`gender` VARCHAR(2) NOT NULL DEFAULT '女' COMMENT '性别',
	`birthday` DATETIME DEFAULT NULL COMMENT '出生日期',
	`address` VARCHAR(100) DEFAULT NULL COMMENT '家庭住址',
	`email` VARCHAR(50) DEFAULT NULL COMMENT '邮箱',
	PRIMARY KEY(`id`)
)ENGINE=INNODB DEFAULT CHARSET=utf8;
```

**格式**

```mysql
CREATE TABLE [IF NOT EXISTS] `表名` (
	`字段名` 列类型 [属性] [索引] [注释],
    `字段名` 列类型 [属性] [索引] [注释],
    ······
    `字段名` 列类型 [属性] [索引] [注释]
)[表类型][字符集设置][注释];
```

**常用命令**

```mysql
show create database school -- 查看创建数据库的语句
show create table student -- 查看student数据表的定义语句
desc student -- 显示表的结构
```



## 2.5 数据表的类型

```mysql
/*关于数据库引擎*/

--   INNODB 默认使用
--   MYISAM 早些年使用
```

|            | MYISAM          | INNODB         |
| ---------- | --------------- | -------------- |
| 事务支持   | 不支持          | 支持           |
| 数据行锁定 | 不支持 （表锁） | 支持           |
| 外键约束   | 不支持          | 支持           |
| 全文索引   | 支持            | 不支持         |
| 表空间大小 | 较小            | 较大，约为两倍 |

常规使用操作

-   MYISAM 节约空间，速度快
-   INNODB 安全性高，事务的处理，多表多用户的处理





>   在物理空间存在的位置

所有的数据库文件都存在data目录下，本质还是文件存储！

MYSQL引擎在物理文件上的区别

-   InnoDB在数据库表中只有一个 *.frm 文件， 以及上级目录下的 ibdata1文件
-   MYISAM对应文件
    -   *.frm 	表结构的定义文件
    -   *.MYD    数据文件(data)
    -   *.MYI      索引文件(index)



>   设置数据库表的字符编码

```sql
CHARSET=utf8
```

不设置的话，会是mysql默认的字符集编码 ~ Latin1 ， 不支持中文



也可以在my.ini中配置默认编码

```sql
character-set-server=utf8
```





## 2.6 修改删除表

>   修改

```sql
-- 修改表名
-- ALTER TABLE 旧表名 RENAME AS 新表名;
ALTER TABLE teachertest RENAME AS teacher;
-- 添加字段
-- ALTER TABLE 表名 ADD 字段名 列属性;
ALTER TABLE teacher ADD age INT(11);

-- 修改约束
-- ALTER TABLE 表名 MODIFY 字段名 列属性[];
ALTER TABLE teacher MODIFY age VARCHAR(11);
-- 修改字段名（也可以修改约束）
-- ALTER TABLE 表名 CHANGE 字段名 列属性[];
ALTER TABLE teacher CHANGE age age1 INT(1);

-- 删除表字段
-- ALTER TABLE 表名 DROP 字段名;
ALTER TABLE teacher DROP age;


```

>   删除

```sql
-- 删除表
DROP TABLE IF EXISTS teacher
```

==所有的创建和删除尽量加上判断，以免报错==



**注意**

-   `` 字段名 包裹
-   注释 -- #
-   sql关键字大小写不敏感，建议小写
-   所有的符号全用英文





# 3. MySQL数据管理

## 3.1 外键

>   方式一、在创建表的时候，增加约束（麻烦，比较复杂）

```sql
CREATE TABLE IF NOT EXISTS `grade`(
	`gradeid` INT(10) NOT NULL AUTO_INCREMENT COMMENT '学号',
	`gradename` VARCHAR(50) NOT NULL COMMENT '年纪名称',
	PRIMARY KEY(`gradeid`)
)ENGINE=INNODB DEFAULT CHARSET=utf8;


CREATE TABLE IF NOT EXISTS `student`(
	`id` INT(4) NOT NULL AUTO_INCREMENT COMMENT '学号',
	`name` VARCHAR(30) NOT NULL DEFAULT '匿名' COMMENT '姓名',
	`pwd` VARCHAR(20) NOT NULL DEFAULT '123456' COMMENT '密码',
	`gender` VARCHAR(2) NOT NULL DEFAULT '女' COMMENT '性别',
	`birthday` DATETIME DEFAULT NULL COMMENT '出生日期',
	`address` VARCHAR(100) DEFAULT NULL COMMENT '家庭住址',
	`email` VARCHAR(50) DEFAULT NULL COMMENT '邮箱',
	`gradeid` INT(10) NOT NULL COMMENT '学生年级',
	PRIMARY KEY(`id`),
	KEY `FK_gradeid` (`gradeid`),
	CONSTRAINT `FK_gradeid` FOREIGN KEY (`gradeid`) REFERENCES `grade` (`gradeid`)
)ENGINE=INNODB DEFAULT CHARSET=utf8;
```



删除有外键关系的表的时候，必须先删除引用别人的表（从表），再删除被引用的表（主表）



>   方式二：创建表之后添加表的外键约束

```sql
ALTER TABLE `student`
ADD CONSTRAINT `FK_gradeid` FOREIGN KEY (`gradeid`) REFERENCES `grade` (`gradeid`);
-- ADD CONSTRAINT '约束名' FOREIGN KEY (`作为外键的列`) REFERENCES `目标表名` (`目标字段名`);
```



以上的操作都是物理外键，数据库级别的外键，我们不建议使用!(避免数据库过多造成困扰)



==最佳实践==：

-   数据库就是单纯的表，只用来村数据，只有行（数据）、列（字段）
-   我们想使用多张表的数据，想使用外键（程序实践）

## 3.2 DML语言

**数据库意义**：数据存储、数据管理

DML语言：数据操作语言    **DataManipulationLanguage**

-   insert
-   update
-   delete

## 3.3 添加

>   insert

```sql
-- 插入语句
-- insert into 表名([字段名1,字段名2,字段名3])
-- values('值1','值2','值3');

INSERT INTO grade(`gradename`) VALUES ('大一');

-- 插入多个字段
INSERT INTO grade(`gradename`)
VALUES('大二'),('大三');
```

可以不写字段名，那么后面的值就要一一对应

## 3.4 修改

>   update

```sql
-- 修改学员名字，带了简介
update `student` set `name`='奥里给' where id = 1

-- 在不指定条件的情况下， 会改动所有的表
update `student` set `name` = '奥里给';

-- 语法
-- update 表名 set colnum_name = value, [colnum_name = value,...] where [条件]

```

```sql
-- 指示的范围是闭区间
between ... and ... 
```

-   value的值可以是具体的值，也可以是变量
-   多个属性设置的时候，用逗号隔开
-   colnum_name 尽量带上``
-   不指定筛选条件会修改所有的列

## 3.5 删除

>   delete 命令

```sql
-- delete from 表名 [where 条件] 
-- 删除数据
delete from `student`;

-- 删除指定数据
delete from `student` where id = 1;
```



>   truncate命令

作用： 完全清空一个数据库表，表的结构和索引约束不会变

```sql
-- 清空表
truncate `student`;
```

>   delete 和 truncate 区别

相同点：都能删除数据，不清除表结构

不同：

-   delete删除表后不会清除自增
-   truncate 删除表后 自增会归零、不会影响事务



`delete`删除， 重启数据库后的现象

-   InnoDB 自增列会从1开始 （存在内存中，断电即失）
-   MyISAM 继续从上一个子增量开始（存在文件中的，不会丢失）



# 4. DQL查询数据（重点）

## 4.1 DQL

(Data Query LANGUAGE : 数据查询语言)

-   所以查询都用 select
-   简单查询，复杂查询都可以



## 4.2 查询指定字段

```sql
-- 查询所有的学生
-- select 字段 from 表
select * from student;

-- 查询指定字段
select `StudentNo`, `StudentName` from student;

-- 别名， 给结果起一个名字 AS
select `StudentNo` as 学号, `StudentName` as 学生姓名 from student as s;

-- 函数 concat(a, b)
select concat('姓名：', StudentName) as 新名字 from 表;
```

语法： select 字段, …… from 表

>   有的时候 列名可能不是那么的见名知意， 用 as改名



>    去重 

```sql
-- 查询有哪些同学参加了考试，成绩
select * from result;
select `StudentNo` from result;
select distinct `StudentNo` from result;
```

>   数据库的列（表达式）

```sql
select version() -- 查询系统版本 (函数)
select 100 * 3 - 1 as 计算结果 -- 用来计算 (计算表达式)
select @@auto_increment_increment -- 查询自增步长 （变量）

-- 学员成绩 + 1 查看
select `StudentNO`, `StudentResult` + 1 as '提分后' from result;
```

**数据库中的表达式： 文本值、列、Null、函数、计算表达式、系统变量**



## 4.3 where 条件子句

作用：检索数据中符合条件的值

>   逻辑运算符

| 运算符     | 语法                 | 描述                       |
| ---------- | -------------------- | -------------------------- |
| and &&     | a and b   a && b     | 逻辑与，两个都为真，才为真 |
| or    \|\| | a or b      a \|\| b | 逻辑或，一个为真，就为真   |
| not ！     | not a       ！a      | 逻辑非，真为假，假为真     |



>   模糊查询： 比较运算符
>
>   | 运算符      | 语法                     | 描述                              |
>   | ----------- | ------------------------ | --------------------------------- |
>   | IS NULL     | a is null                | 如果操作符为null，则结果为真      |
>   | IS NOT NULL | a is not null            | 如果操作符为not null， 则结果为真 |
>   | BETWEEN     | a between b and c        | 若a在b和c之间，则为真             |
>   | LIKE        | a like b                 | SQL匹配，如果a匹配b，则结果为真   |
>   | IN          | a in （b1, b2, b3, …..） | 若a在后面b的集合中，则结果为真    |

```sql
-- ===================== 模糊查询 ========================

-- 查询姓刘的同学
-- like结合 %(代表0到任意个字符) _(一个字符)
SELECT `StudentNo`, `StudentName` FROM `student`
WHERE StudentName LIKE '刘%';

SELECT `StudentNo`, `StudentName` FROM `student`
WHERE StudentName LIKE '刘_';

SELECT `StudentNo`, `StudentName` FROM `student`
WHERE StudentName LIKE '%刘_';

-- ===== in ======
SELECT `StudentNo`, `StudentName` FROM `student`
WHERE StudentNo IN (1001, 1002, 1003);

-- 查询生日不为空的学生
SELECT `StudentNo`, `StudentName` FROM `student`
WHERE `borndate` IS NOT NULL;
-- 查询生日为空的学生
SELECT `StudentNo`, `StudentName` FROM `student`
WHERE `borndate` IS  NULL;
```



## 4.4 连表查询

>   join 对比

```sql
-- ================ 连表查询 =========================
SELECT s.StudentNo, StudentName, SubjectNo, StudentResult
FROM student AS s
INNER JOIN result AS r
ON s.`studentno` = r.`studentno`;

SELECT s.StudentNo, StudentName, SubjectNo, StudentResult
FROM student AS s
RIGHT JOIN result AS r
ON s.`studentno` = r.`studentno`;

SELECT s.StudentNo, StudentName, SubjectNo, StudentResult
FROM student AS s
LEFT JOIN result AS r
ON s.`studentno` = r.`studentno`;
```

| 操作       | 描述                                       |
| ---------- | ------------------------------------------ |
| inner join | 如果表中至少有一个匹配，则返回行           |
| left join  | 即使右表中没有匹配，也会从左表中返回所有值 |
| right join | 会从右表中返回所有制，即使左表中没有匹配   |

-   假设存在一种多张表查询，慢慢来，先查询两张表 然后再慢慢添加



>   自连接

自己的表和自己的表连接，核心：**一张表拆为两张一样的表即可**

父类

| categoryid | categoryName |
| ---------- | ------------ |
| 2          | 信息技术     |
| 3          | 软件开发     |
| 5          | 美术设计     |
|            |              |

子类

| pid  | categoryid | categoryName |
| ---- | ---------- | ------------ |
| 3    | 4          | 数据库       |
| 2    | 8          | 办公信息     |
| 3    | 6          | web开发      |
| 5    | 7          | ps技术       |

操作：查询父类对应子类的关系

| 父类     | 子类     |
| -------- | -------- |
| 信息技术 | 办公信息 |
| 软件开发 | 数据库   |
| 软件开发 | web开发  |
| 美术设计 | ps技术   |

```sql
-- 把一张表拆成两张一样的表
SELECT a.`categoryName` AS '父栏目', b.`categoryName` AS '子栏目'
FROM `category` AS a, `category` AS b
WHERE a.`categoryid` = b.`pid`;
```



```sql
-- 查询参加了 数据库结构-1 考试的同学信息： 学号、学生姓名、科目名、分数
SELECT s.`studentno`, `studentname`, `subjectname`, `studentresult` FROM `student` AS s
INNER JOIN `result` AS r
ON s.`studentno` = r.`studentno`
INNER JOIN `subject` AS sub
ON r.`subjectno` = sub.`subjectno`
WHERE `subjectname` = '数据库结构-1';
```





## 4.5 分页和排序

>   排序

```sql
-- 排序：升序ASC, 降序 DESC
-- ORDER BY 通过哪个字段排序，怎么排
SELECT s.`studentno`, `studentname`, `subjectname`, `studentresult` 
FROM `student` AS s
INNER JOIN `result` AS r
ON s.`studentno` = r.`studentno`
INNER JOIN `subject` AS sub
ON r.`subjectno` = sub.`subjectno`
WHERE `subjectname` = '数据库结构-1'
ORDER BY studentresult ASC;
```



>   分页

```sql
-- 为什么要分页
-- 缓解数据库压力，给人的体验更好，瀑布流

-- 分页，每页只显示五条数据
-- 语法： limit 起始行数， 页面大小

-- 第一页 limit 0,5
-- 第二页 limit 5, 5
-- 第三页 limit 10, 5
-- 第N页 limit （N-1）* pagesize, pagesize

```



## 4.6 子查询

本质：==在where语句中嵌套一个子查询语句==



## 4.7 分组和过滤

group by 和 having



# 5 MySQL函数

可以去官网看

## 5.1 常用函数

```sql
-- 数学运算
SELECT ABS(-8);      -- 绝对值
SELECT CEILING(9.4); -- 向上取整
SELECT FLOOR(9.4);   -- 向下取整
SELECT RAND();       -- 返回0~1的随机数
SELECT SIGN(0);	     -- 判断一个数的符号 0 - 0 负数 -1 正数 1

-- 字符串函数
SELECT CHAR_LENGTH('哦啦啦哦咳咳'); -- 字符串长度
SELECT CONCAT('我','是','buba');    -- 拼接字符串
SELECT INSERT('我爱编程', 1, 2, '超级热爱') 
-- 查询， 从某个位置开始替换某个长度，第一个是index 第二个是length
SELECT LOWER('AOLIGEI');    -- 转小写
SELECT UPPER('aoligei');	    -- 转大写
SELECT INSTR('kuangshen', 'h');  -- 返回第一次出现的字串的索引

SELECT REPLACE('狂神说坚持就能成功', '坚持', '努力'); 
-- 就是替换指定的字符串
SELECT SUBSTR('狂神说坚持就能成功', 4, 6); 
-- 返回指定字符串（源字符串， 截取位置， 截取长度）

SELECT REVERSE('清晨我上马');

-- 查询姓 周的同学， 名字 
SELECT REPLACE(studentname, '周', '邹') FROM student
WHERE studentname LIKE '周%';

-- 时间和日期的函数(重要)
SELECT CURRENT_DATE() -- 获取当前日期; 
SELECT CURDATE();     -- 同上
SELECT NOW();         -- 获取当前时间 多了时分秒
SELECT LOCALTIME();   -- 本地时间
SELECT SYSDATE();     -- 系统时间
SELECT YEAR(NOW());  
SELECT MONTH(NOW()); 
SELECT WEEK(NOW());   
SELECT DAY(NOW());   
SELECT HOUR(NOW());  
SELECT MINUTE(NOW());  
SELECT SECOND(NOW());   

-- 系统
SELECT SYSTEM_USER();
SELECT USER();
SELECT VERSION();
```





## 5.2 聚合函数（常用）

| 函数名称 | 描述   |
| -------- | ------ |
| COUNT()  | 计数   |
| SUM()    | 求和   |
| AVG()    | 平均值 |
| MAX()    | 最大值 |
| MIN()    | 最小值 |
| …        | …      |

```sql
SELECT COUNT(student) FROM student ; -- 会忽略所有的null值
SELECT COUNT(*) FROM student;    --  不会忽略null ， 本质是计算行数
SELECT COUNT(1) FROM studnet;    --  不会忽略null ， 本质是计算行数

/*
InnoDB引擎下
1.count(*)和count(1)的效率是一样的！两者没有性能差异！
如果表存在主键，他们都是根据主键去count的，速度都较快；如果不存在主键，则速度都较慢！

2.count(1) /count(*)会统计表中的所有的记录数，包含字段为null 的记录；
count(列名) 会统计该字段在表中出现的次数，不统计字段为null 的记录。
且在效率方面count(非主键列)的效率往往低于count(*)/count(1) ！count(主键列)效率差不多！
*/
```



## 5.3 数据库级别的MD5加密(拓展)

md5 不可逆

MD5 破解网站原理， 背后有一个字典，MD5 加密后的值

```sql
--  ======测试MD5 加密 =======

CREATE TABLE `testmd5`(
	`id` INT(4) NOT NULL,
	`name` VARCHAR(20) NOT NULL,
	`pwd` VARCHAR(50) NOT NULL,
	PRIMARY KEY(`id`)
)ENGINE=INNODB DEFAULT CHARSET=utf8;

INSERT INTO testmd5 VALUES
(1, 'zhangsan', '123456'),
(2, 'lisi', '123456'),
(3, 'wangwu', '123456'),
(4, 'zhaoliu', '123456');

-- 加密
UPDATE testmd5 SET pwd=MD5(pwd);

-- 插入的时候加密
INSERT INTO testmd5 VALUES (5, 'xiaoming ', MD5('123456'));

-- 如何校验 ： 将用户传递进来的密码，进行md5加密，然后对比加密后的值
SELECT * FROM testmd5 WHERE `name` = 'xiaoming' AND pwd = MD5('123456');
```





# 6 事务

## 6.1 什么是事务

事务是针对修改数据库中某一项数据的一系列操作的集合，不可分割。

要么全部做完成功，做不完或者不做都是失败



>   事务原则：ACID原则  原子性，一致性，隔离性，持久性 （脏读， 幻读）

-   原子性：某个操作可能会有好几个步骤，比如转钱。这几个步骤要么一起成功，要么一起失败，不能只发生其中一个动作
-   一致性：
    -   操作前后数据是正确的，符合程序员预期的
-   持久性
    -   事务没有提交，恢复到原状
    -   事务提交了，持久化到数据库
    -   事务提交具有不可逆性
-   隔离性
    -   排除其他事务对本事务的影响



>   事务的隔离级别

-   脏读

    指一个事务读取了另一个事务没有提交的数据

-   不可重复读

    在一个事务内读取表中的一行数据，多次读取结果不同

-   幻读（虚）

    指在一个事务内读取到了别的事务插入的数据，导致前后读取不一致。



>   执行事务

```sql
-- mysql 默认开启事务自动提交
SET autocommit = 0  -- 关闭
SET autocommit = 1  -- 开启（默认）

-- 事务开启
START TRANSACTION


-- 提交：持久化（成功）
COMMIT

-- 回滚：回到原来的样子（失败！）
ROLLBACK

-- 事务结束
SET autocommit = 1 -- 开启自动提交

-- 保存点
SAVEPOINT 保存点名 -- 设置一个事务的保存点
ROLLBACK TO SAVEPOINT 保存点名 -- 回滚到保存点名
RELEASE SAVEPOINT 保存点名 -- 撤销保存点
```



>   模拟场景

```sql
CREATE DATABASE shop CHARACTER SET utf8 COLLATE utf8_general_ci;

USE shop;

CREATE TABLE `account`(
	`id` INT(3) NOT NULL AUTO_INCREMENT,
	`name` VARCHAR(30) NOT NULL,
	`money` DECIMAL(9, 2) NOT NULL,
	PRIMARY KEY(`id`)
)ENGINE=INNODB DEFAULT CHARSET=utf8;

INSERT INTO account(`name`, `money`) VALUES ('A', 2000.00),('B', 10000.00);

-- 模拟转账
SET autocommit = 0; -- 关闭自动提交
START TRANSACTION; -- 开启一个事务
UPDATE account SET money = money - 500 WHERE `name` = 'A';  -- A减500
UPDATE account SET money = money + 500 WHERE `name` = 'B';  -- B加500

COMMIT; -- 提交事务， 就被持久化了

ROLLBACK; -- 回滚

SET autocommit = 1; -- 恢复默认值
```





# 7 索引

>   MySQL对索引的定义为:**索引(index)是帮助MySQL高效获取数据的数据结构**
>
>   提取句子主干,就可以得到索引的本质:索引时数据结构



## 7.1 索引的分类

-   主键索引(primary key)
    -   唯一标识,主键不可重复,只能有一个列作为主键
-   唯一索引(UNIQUE KEY)
    -   避免重复列的出现,唯一索引可以重复,多个列都可以标识为 唯一索引
-   常规索引(key/index)
    -   默认的,index. key关键字设置
-   全文索引(fulltext)
    -   在特定数据库引擎下才有, MyISAM
    -   快速定位数据

```sql
-- 索引的使用
-- 1. 在创建表的时候给字段增加索引
-- 2. 创建完毕后，增加索引

-- 显示所有的索引信息
SHOW INDEX FROM student;

-- 增加一个全文索引 (索引名) 列名
ALTER TABLE `student` ADD FULLTEXT INDEX `studentname`(`studentname`);

-- explain 分析sql执行的情况
EXPLAIN SELECT * FROM student; -- 非全文索引
EXPLAIN SELECT * FROM student WHERE MATCH(studentname) AGAINST('刘');
```





## 7.2 测试索引

```sql
CREATE TABLE `app_user` (
`id` BIGINT(20) UNSIGNED NOT NULL AUTO_INCREMENT,
`name` VARCHAR(50) DEFAULT '',
`email` VARCHAR(50) NOT NULL,
`phone` VARCHAR(20) DEFAULT '',
`gender` TINYINT(4) UNSIGNED DEFAULT '0',
`password` VARCHAR(100) NOT NULL DEFAULT '',
`age` TINYINT(4) DEFAULT NULL,
`create_time` DATETIME DEFAULT CURRENT_TIMESTAMP,
`update_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
PRIMARY KEY (`id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8

-- 插入100万数据.
DELIMITER $$
-- 写函数之前必须要写，标志
CREATE FUNCTION mock_data1 ()
RETURNS INT
BEGIN
DECLARE num INT DEFAULT 1000000;
DECLARE i INT DEFAULT 0;
WHILE i<num DO
INSERT INTO `app_user`(`name`,`email`,`phone`,`gender`)VALUES(CONCAT('用户',i),'19224305@qq.com','123456789',FLOOR(RAND()*2));
SET i=i+1;
END WHILE;
RETURN i;
END;

SELECT mock_data1() -- 执行此函数 生成一百万条数据
 
SELECT * FROM app_user WHERE `name` = '用户9999';   -- 0.4s
EXPLAIN SELECT * FROM app_user WHERE `name` = '用户9999';

-- id _ 表名 _ 字段名
-- create index 索引名 on 表(字段)
CREATE INDEX id_app_user_name ON app_user(`name`);

SELECT * FROM app_user WHERE `name` = '用户9999';  -- 0.002s
```

==索引在数据量小的时候，用处不大，但是数据量大的时候，区别十分明显==



## 7.3 索引原则

-   索引不是越多越好
-   不要对经常变动的数据加索引
-   小数据量的表不需要加索引
-   索引一般加在常用来查询的字段上！



>   索引的数据结构

Hash类型的索引





# 8 权限管理和备份

## 8.1 用户管理

>   SQL 命令操作

用户表： mysql.user

本质：对这张表进行增删改查



```sql
-- 创建用户
-- CREATE USER 用户名 IDENTIFIED BY '密码';
CREATE USER kuangshen IDENTIFIED BY '123456';

-- 修改密码（修改当前用户密码）

SET PASSWORD = PASSWORD('123456');

-- 修改密码（修改指定用户密码）

SET PASSWORD FOR kuangshen = PASSWORD('123456');

-- 重命名
-- RENAME USER 原名 TO 新名;
RENAME USER kuangshen TO kuangshen2;

-- 用户授权  ALL PRIVILEGES on 库.表
-- ALL PRIVILEGES 除了给别人授权，别的啥都能干
GRANT ALL PRIVILEGES ON *.* TO kuangshen;

-- 查询权限
SHOW GRANTS FOR kuangshen; -- 查看指定用户的权限
SHOW GRANTS FOR root@localhost;

-- root的权限就是所有权限+tgrant权限 GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' WITH GRANT OPTION

-- 撤销权限 revoke 哪些权限， 在哪个数据库撤销，给谁撤销
REVOKE ALL PRIVILEGES ON *.* FROM kuangshen;

-- 删除用户
DROP USER kuangshen;
```



## 8.2 MySQL备份

为什么要备份

-   保证重要的数据不丢失
-   数据转移

MySQL数据库备份方式

-   直接拷贝物理文件

-   在SQLyog可视化工具中手动导出

    -   在想要导出的表

-   使用命令行导出 `mysqldump`

    ```bash
    # mysqldump -h主机 -u用户名 -p密码 数据库 表名> 物理磁盘位置/文件名
    mysqldump -hlocalhost -uroot -p123456 school student > D:/a.sql
    
    # mysqldump -h主机 -u用户名 -p密码 数据库 表名1 表名2 表名3> 物理磁盘位置/文件名
    mysqldump -hlocalhost -uroot -p123456 school student app_user> D:/b.sql
    # mysqldump -h主机 -u用户名 -p密码 数据库> 物理磁盘位置/文件名
    mysqldump -hlocalhost -uroot -p123456 school > D:/c.sql
    
    
    # 导入
    # 登录的情况下， 切换到指定的数据库
    # source 备份文件
    source d:/a.sql
    
    ```



# 9 规范数据库设计

##  9.1 为什么需要设计

==当数据库比较复杂的时候，我们就需要设计==

**糟糕的数据库设计：**

-   数据冗余，浪费空间
-   数据库插入和删除都很麻烦、异常

**良好的数据库设计：**

-   节省内存空间
-   保证数据库的完整性

**软件开发中，关于数据库的设计**

-   分析需求：分析业务和需要处理的数据库的需求
-   概要设计：设计关系图E-R图



设计数据库的步骤：（个人博客）

-   收集信息，分析需求
    -   用户表（用户登陆注销，用户的个人信息，写博客，创建分类）
    -   分类表（文章分类，谁创建的）
    -   文章表（文章的信息）
    -   评论表
    -   友链表（友链信息）
    -   自定义表（系统信息，某个关键的字，或者一些主题） key：value 就两列
    -   说说表（发表心情 id…）
-   标识实体（把需求落实到每个字段）

-   标识实体之间的关系
    -   写博客：user -》 blog
    -   创建分类：user -》 category
    -   关注: user -> user
    -   友链： links
    -   评论： user —> user -> blog





## 9.2 三大范式

为什么需要数据规范化

-   信息重复
-   更新异常
-   插入异常
    -   无法正常显示信息
-   删除异常
    -   丢失有效的信息



>   三大范式

**第一范式（1NF）**

原子性：保证每一列不可再分

**第二范式（2NF）**

前提：满足第一范式

每张表只描述一件事情，

**第三范式（3NF）**

前提：满足第一范式和第二范式

数据表中的每一列数据都和主键直接相关，而不能间接相关



（规范数据库的设计）

**规范性 和 性能的问题**

关联查询的表不得超过三张表

-   考虑商业化的需求和目标。（成本，用户体验）数据库的性能更加重要
-   在规范性能的问题的时候，需要适当的考虑一下规范性
-   故意给某些表增加一些冗余的字段。（从多表查询变为单表查询）
-   故意增加一些计算列（从大数据量降低为小数据量的查询：索引）





# 10 JDBC（重点）

## 10.1 数据库驱动

我们的程序会通过数据库驱动，和数据库打交道



## 10.2 JDBC

SUN攻速为了简化开发人员的（对数据库的统一）操作，提供了一个（Java操作数据库）规范，俗称JDBC。这些规范的具体实现由具体的厂商去做

对于开发人员来说，我们只需要掌握JDBC接口的操作即可



java.sql

javax.sql

还需要导入一个数据库驱动包 mysql-connector-java-5.1.47.jar



## 10.3 第一个JDBC程序

> 创建测试数据库

```sql
CREATE DATABASE jdbcStudy CHARACTER SET utf8 COLLATE utf8_general_ci;

USE jdbcStudy;

CREATE TABLE `users`(
id INT PRIMARY KEY,
NAME VARCHAR(40),
PASSWORD VARCHAR(40),
email VARCHAR(60),
birthday DATE
);

INSERT INTO `users`(id,NAME,PASSWORD,email,birthday)
VALUES(1,'zhansan','123456','zs@sina.com','1980-12-04'),
(2,'lisi','123456','lisi@sina.com','1981-12-04'),
(3,'wangwu','123456','wangwu@sina.com','1979-12-04')
```



1. 创建一个项目
2. 导入数据库驱动
3. 编写测试代码

```java

import java.sql.*;
//我的第一个jdbc程序
public class jdbcFirst {
    public static void main(String[] args) throws ClassNotFoundException, SQLException {
        // 1. 加载驱动
        Class.forName("com.mysql.jdbc.Driver"); // 加载驱动
        // 2.用户信息和url
        String url = "jdbc:mysql://localhost:3306/jdbcstudy?useUnicode=true&characterEncoding=utf8";
        String username = "root";
        String password = "123456";
        // 3. 连接成功，返回数据库对象
        Connection connection = DriverManager.getConnection(url, username, password);
        // 4. 执行sql的对象 statement执行sql的对象
        Statement statement = connection.createStatement();
        // 5. 执行SQL的对象去执行SQL
        String sql = "select * from users";

        ResultSet resultSet = statement.executeQuery(sql); // 结果集中封装了全部的查询结果

        while (resultSet.next()){
            System.out.println("id = "+ resultSet.getObject("id"));
            System.out.println("name = "+ resultSet.getObject("NAME"));
            System.out.println("pwd = "+ resultSet.getObject("PASSWORD"));
            System.out.println("email = "+ resultSet.getObject("email"));
            System.out.println("birthday = "+ resultSet.getObject("birthday"));
            System.out.println("=====================================================");
        }

        // 6.释放连接
        resultSet.close();
        statement.close();
        connection.close();
    }
}
```

步骤总结：

1. 加载驱动
2. 连接数据库 DriverManager
3. 获得执行sql的对象 statement
4. 获得返回的结果集 resultset
5. 释放连接



> DriverManager

```java
// 1. 加载驱动
// DriverManager.registerDriver(new com.mysql.jdbc.Driver());
// 里面就是静态代码块，直接通过反射调用就可以直接注册，上面的方式相当于注册了两次。
Class.forName("com.mysql.jdbc.Driver"); // 加载驱动

Connection connection = DriverManager.getConnection(url, username, password);
// connection 代表数据库
// 数据库级别的事情都可以做,比如自动提交 事务提交 回滚
connection.setAutoCommit();
connection.commit();
connection.rollback();
```



> URL

```java
String url = "jdbc:mysql://localhost:3306/jdbcstudy?useUnicode=true&characterEncoding=utf8";

// mysql -- 3306
// 协议://主机地址:端口号/数据库名?参数1&参数2&参数3
// oracle -- 1521
// jdbc:roacle:thin:@localhost:1521:sid
```



> statement 执行SQL的对象  prepareStatement 执行SQL 的对象

```java
statement.executeQuery(); // 查询操作返回 ResultSet
statement.execute();      // 执行任何操作 会有判断，效率可能低一点
statement.executeUpdate();// 更新、插入删除都用这个，返回受影响行数
```



> ResultSet 查询的结果集:封装了所有的查询结果

获得指定的数据类型

```java
resultSet.getObject();// 不知道列类型的时候使用
//知道列类型就用指定类型
resultSet.getString();
resultSet.getInt();
resultSet.getFloat();
resultSet.getDate();
```

遍历, 指针

```java
resultSet.beforeFirst(); // 移动到最前面
resultSet.afterLast();  // 移动到最后面
resultSet.next(); // 移动到下一行
resultSet.previous(); //移动到前一行
resultSet.absolute(row);// 移动到指定行
```



> 释放资源

```java
// 6.释放连接
resultSet.close();
statement.close();
connection.close();
```



## 10.4 statement对象

==jdbc中的statement对象用于向数据库发送SQL语句，想完成对数据库的增删改查，只需要通过这个对象向数据库发送增删改查语句即可。==



Statement对象的executeUpdate方法，用于向数据库发送增删改的语句，executeUpdate执行完后，将会返回一个整数（即增删改查语句导致了数据库几行数据发生了变化）。



Statement对象的executeQuery方法，用于想数据库发送查询语句，executeQuery方法返回代表查询结果的ResultSet对象。



> CRUD操作 - create

使用executeUpdate（String sql）方法完成数据添加操作，实例操作

```java
Statement st = conn.createStatement();
String sql = "update user set name='' where name=''";
int num = st.executeUpdate(sql);
if(num > 0){
    System.out.println("修改成功！")
}
```



> CRUD 操作 -read

使用executeQuery（String sql）方法完成数据查询操作，实例操作：

```java
Statement st = conn.createStatement();
String sql = "select * from user where id = 1";
ResultSet rs = st.executeQuery(sql);
if(rs.next()){
    //根据获取列的数据类型，分别调用rs的相应方法映射到java对象中
}
```



> 代码实现

1. 提取工具类

```java
import java.io.IOException;
import java.io.InputStream;
import java.sql.*;
import java.util.Properties;

public class jdbcUtils {
    private static String driver = null;
    private static String url = null;
    private static String username = null;
    private static String password = null;

    static {
        try{
            InputStream in = jdbcUtils.class.getClassLoader().getResourceAsStream("db.properties");
            Properties properties = new Properties();
            properties.load(in);
            driver = properties.getProperty("driver");
            url = properties.getProperty("url");
            username = properties.getProperty("username");
            password = properties.getProperty("password");

            // 1.驱动只需要加载一次，所以放到静态
            Class.forName(driver);
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }

    //获取连接
    public static Connection getConnection() throws SQLException {
        return DriverManager.getConnection(url, username, password);
    }

    // 释放连接资源
    public static void relese(Connection conn, Statement st, ResultSet rs){
        if(rs!=null){
            try {
                rs.close();
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        }
        if(st!=null){
            try {
                st.close();
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        }
        if (conn!=null){
            try {
                conn.close();
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        }
    }
}

```

2. 编写增删改的方法， `executeUpdate`

```java
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import lesson2.utils.jdbcUtils;

public class Testinsert {
    public static void main(String[] args) {
        Connection conn = null;
        Statement st = null;
        ResultSet rs = null;

        try {
            conn = jdbcUtils.getConnection();// 获得数据库的连接
            st = conn.createStatement();
            String sql = "insert into users(id, `Name`, `PASSWORD`, `email`, `birthday`)\n"+
                    "values(4, 'kuangshen', '123456', '1418407007@qq.com', '2020-01-01')";

            int i = st.executeUpdate(sql);
            if(i > 0){
                System.out.println("插入成功！");
            }
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }finally {
            jdbcUtils.relese(conn, st, null);
        }
    }
}
```

```java
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class testdelete {
    public static void main(String[] args) {
        Connection conn = null;
        Statement st = null;
        ResultSet rs = null;

        try {
            conn = jdbcUtils.getConnection();// 获得数据库的连接
            st = conn.createStatement();
            String sql = "delete from users where id = 4";

            int i = st.executeUpdate(sql);
            if(i > 0){
                System.out.println("删除成功！");
            }
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }finally {
            jdbcUtils.relese(conn, st, null);
        }
    }
}
```

```java
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class testupdate {
    public static void main(String[] args) {
        Connection conn = null;
        Statement st = null;
        ResultSet rs = null;

        try {
            conn = jdbcUtils.getConnection();// 获得数据库的连接
            st = conn.createStatement();
            String sql = "update users set `NAME` = 'kuangshen' where id = 1";

            int i = st.executeUpdate(sql);
            if(i > 0){
                System.out.println("修改成功！");
            }
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }finally {
            jdbcUtils.relese(conn, st, null);
        }
    }
}
```



3. 查询 `executeQuery`

```java
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class testselect {

    public static void main(String[] args) {
        Connection conn = null;
        Statement st = null;
        ResultSet rs = null;
        try {
            conn = jdbcUtils.getConnection();
            st = conn.createStatement();
            String sql = "select * from users where id = 1";
            rs = st.executeQuery(sql);
            if (rs.next()){
                System.out.println(rs.getString("NAME"));
            }
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        } finally {
            jdbcUtils.relese(conn, st, rs);
        }
    }
}
```



> SQL注入的问题

sql存在漏洞，会存在攻击导致数据泄露。 ==SQL会被拼接or==

```sql
select * from users where `name` = '' or '1=1' and `password` = '' or '1=1'; 
-- 恒为真
```



## 10.5 PreparedStatement对象

PreparedStatement对象可以防止SQL注入，效率更高

1. 新增

    ```java
    import lesson2.utils.jdbcUtils;
    import java.sql.Connection;
    import java.sql.PreparedStatement;
    import java.sql.SQLException;
    import java.util.Date;
    
    public class testinsert {
        public static void main(String[] args) {
            Connection conn = null;
            PreparedStatement st = null;
            try {
                conn = jdbcUtils.getConnection();
    
                //区别
                // 使用 ？ 占位符 代替参数
                String sql = "insert into users(id, `name`, `password`, `email`, `birthday`) value (?, ?, ?, ?, ?)";
                st = conn.prepareStatement(sql); // 预编译SQL， 先写SQL 不执行
    
                st.setInt(1, 4);
                st.setString(2, "qinjiang");
                st.setString(3, "12321123");
                st.setString(4, "1418407007@qq.com");
    
                /*
                 * 注意点   sql.Date 数据库    java.sql.Date()
                 *         util.Date java    new Date().getTime() 获得时间戳
                 * */
                st.setDate(5, new java.sql.Date(new Date().getTime()));
    
                // 执行
    
                int i = st.executeUpdate();
                if (i > 0){
                    System.out.println("插入成功！");
                }
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }finally {
                jdbcUtils.relese(conn, st, null);
            }
        }
    }
    ```

2. 删除

    ```java
    import lesson2.utils.jdbcUtils;
    import java.sql.Connection;
    import java.sql.PreparedStatement;
    import java.sql.SQLException;
    import java.util.Date;
    
    public class testDelete {
        public static void main(String[] args) {
            Connection conn = null;
            PreparedStatement st = null;
            try {
                conn = jdbcUtils.getConnection();
    
                //区别
                // 使用 ？ 占位符 代替参数
                String sql = "delete from users where id=?";
                st = conn.prepareStatement(sql); // 预编译SQL， 先写SQL 不执行
    
                st.setInt(1, 4);
    
                // 执行
    
                int i = st.executeUpdate();
                if (i > 0){
                    System.out.println("删除成功！");
                }
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }finally {
                jdbcUtils.relese(conn, st, null);
            }
        }
    }
    ```

3. 更新

    ```java
    import lesson2.utils.jdbcUtils;
    import java.sql.Connection;
    import java.sql.PreparedStatement;
    import java.sql.SQLException;
    import java.util.Date;
    
    public class testUpdate {
        public static void main(String[] args) {
            Connection conn = null;
            PreparedStatement st = null;
            try {
                conn = jdbcUtils.getConnection();
    
                //区别
                // 使用 ？ 占位符 代替参数
                String sql = "update users set `name` = ? where id = ?";
                st = conn.prepareStatement(sql); // 预编译SQL， 先写SQL 不执行
    
                st.setString(1, "狂神");
                st.setInt(2, 1);
    
    
                // 执行
    
                int i = st.executeUpdate();
                if (i > 0){
                    System.out.println("插入成功！");
                }
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }finally {
                jdbcUtils.relese(conn, st, null);
            }
        }
    }
    ```

4. 查询

    ```java
    import lesson2.utils.jdbcUtils;
    import java.sql.Connection;
    import java.sql.PreparedStatement;
    import java.sql.ResultSet;
    import java.sql.SQLException;
    
    
    public class testSelect {
        public static void main(String[] args) {
            Connection conn = null;
            PreparedStatement st = null;
            ResultSet rs = null;
            try {
                conn = jdbcUtils.getConnection();
                String sql = "select * from users where id = ?";
    
                // PreparedStatement 防止SQL注入的本质，把传递进来的参数当作字符
                // 假设其中存在转移字符，比如‘ 就会被直接被转义
                st = conn.prepareStatement(sql);
                st.setInt(1, 1);
    
                rs = st.executeQuery();
    
                if (rs.next()){
                    System.out.println(rs.getString("name"));
                    System.out.println(rs.getString("password"));
                    System.out.println("===============================");
                }
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }finally {
                jdbcUtils.relese(conn, st, rs);
            }
        }
    }
    ```

5. 防止SQL注入



## 10.8 事务

> ACID原则

原子性： 要么全部成功，要么全部失败

一致性：代码执行前后，结果符合程序员预期，数据正确

隔离性：多个事务之间互不干扰

持久性：一旦提交就不可逆了，会持久化到数据库



隔离性问题：

- 脏读： ？
- 不可重复读：？
- 虚读：？



> 代码实现

1. 开启事务 `conn.setAutoCommit(false);`
2. 一组业务执行完毕，提交事务
3. 可以catch语句中显式定义回滚语句，但是默认失败就会进行回滚





## 10.9 数据库连接池

数据库连接 -- 执行完毕 -- 释放

连接 -- 释放 十分浪费资源

**池化技术： 预先准备一些资源， 过来就连接接预先准备好的**

常用连接数是多少，就将最少连接数设置为多少

最小连接数： 10

最大连接数：100 业务最高承载上限

排队等待  --》 等待超时





编写连接池， 实现一个接口 DataSource



> 开源数据源实现

DBCP

C3P0

Druid: 阿里巴巴



使用了这些数据库连接池之后，我们在项目开发中就不需要编写连接数据库的代码了



> DBCP

需要jar包 commons-dbcp-1.4.jar  commons-pool-1.6.jar