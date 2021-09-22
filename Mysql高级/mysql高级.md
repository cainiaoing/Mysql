## 索引重要字段的含义

```markdown
# id
	select查询的序列号,包含一组数字，表示查询中执行select子句或操作表的顺序
		id相同，执行顺序由上至下
		id不同，如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行
		id相同不同，同时存在
			id如果相同，可以认为是一组，从上往下顺序执行；
			在所有组中，id值越大，优先级越高，越先执行
	id号每个号码，表示一趟独立的查询。一个sql 的查询趟数越少越好。

# type
	type显示的是访问类型，是较为重要的一个指标，结果值从最好到最坏依次是： 
 	system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > 			index_subquery > range > index > ALL 
 
	system>const>eq_ref>ref>range>index>ALL
 
 	一般来说，得保证查询至少达到range级别，最好能达到ref
# key_len
	表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度
	key_len字段能够帮你检查是否充分的利用上了索引
	
# row
	rows列显示MySQL认为它执行查询时必须检查的行数。
	越少越好
	
# Extra
	包含不适合在其他列中显示但十分重要的额外信息
```





## 1、批量生成数据

```markdown
表有 dept(id,deptName,address,ceo)
	emp(id,empno,name,age,deptId)
```



```mysql
# 一、建表
 CREATE TABLE `dept` (
 `id` INT(11) NOT NULL AUTO_INCREMENT,
 `deptName` VARCHAR(30) DEFAULT NULL,
 `address` VARCHAR(40) DEFAULT NULL,
 ceo INT NULL ,
 PRIMARY KEY (`id`)
) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
 

CREATE TABLE `emp` (
 `id` INT(11) NOT NULL AUTO_INCREMENT,
 `empno` INT NOT NULL ,
 `name` VARCHAR(20) DEFAULT NULL,
 `age` INT(3) DEFAULT NULL,
 `deptId` INT(11) DEFAULT NULL,
 PRIMARY KEY (`id`)
 #CONSTRAINT `fk_dept_id` FOREIGN KEY (`deptId`) REFERENCES `t_dept` (`id`)
) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

# 二、修改mysql二进值日志参数log_bin，做主从复制用的

-- 1、查看差数状态语句（看该日志能不能写函数，函数必须能写）
	show variables like 'log_bin_trust_function_creators';
-- 2、日志参数打开语句
	set global log_bin_trust_function_creators=1; （global全局和局部的区别）
/*
创建函数，假如报错：This function has none of DETERMINISTIC......
# 由于开启过慢查询日志，因为我们开启了 bin-log, 我们就必须为我们的function指定一个参数。
 
show variables like 'log_bin_trust_function_creators';
 
set global log_bin_trust_function_creators=1;
 
# 这样添加了参数以后，如果mysqld重启，上述参数又会消失，永久方法：
 
windows下my.ini[mysqld]加上log_bin_trust_function_creators=1 
 
linux下    /etc/my.cnf下my.cnf[mysqld]加上log_bin_trust_function_creators=1

*/

# 三、创建函数，保证每条数据都不同
1、随机产生字符串
DELIMITER $$  -- delimiter 字符 设置结束符号
CREATE FUNCTION rand_string(n INT) RETURNS VARCHAR(255) -- 创建函数，返回n个函数值
BEGIN    -- 函数头
	DECLARE chars_str VARCHAR(100) DEFAULT 'abcdefghijklmnopqrstuvwxyzABCDEFJHIJKLMNOPQRSTUVWXYZ';
 	-- declare 定义变量chars_str default的初始值为“...”这是原字符，产生的字符要从这里截取
 	DECLARE return_str VARCHAR(255) DEFAULT ''; -- 定义返回值的变量
 	DECLARE i INT DEFAULT 0; -- 定义变量i从0开始要执行n次
 	WHILE i < n DO  
 	SET return_str =CONCAT(return_str,SUBSTRING(chars_str,FLOOR(1+RAND()*52),1)); -- 26+26=52 
 	SET i = i + 1;
 	END WHILE;
 	RETURN return_str; -- 得到一个长度为n的字符串，保证最后得到的字符串都不一样
END $$  -- 函数尾
 
 
#假如要删除
#drop function rand_string;

2、随机产生部门编号（单个）
#用于随机产生多少到多少的编号
DELIMITER $$
CREATE FUNCTION  rand_num (from_num INT ,to_num INT) RETURNS INT(11)
BEGIN   
	-- from_num=起始编号，to_num结束编号
	DECLARE i INT DEFAULT 0;  
	SET i = FLOOR(from_num +RAND()*(to_num -from_num+1))   ;
	RETURN i;  
 END$$ 
 
#假如要删除
#drop function rand_num;

# 四、创建存储过程
1、创建往emp表中插入数据的存储过程

DELIMITER $$
CREATE PROCEDURE  insert_emp(  START INT ,  max_num INT ) -- 创建存储过程，默认是in类型的参数
BEGIN  
DECLARE i INT DEFAULT 0;   
	#set autocommit =0 把autocommit设置成0  这样就避免了mysql一行一行的提交
 	SET autocommit = 0;    
 	REPEAT  -- repeat(循环)
 	SET i = i + 1;  
 	INSERT INTO emp (empno, NAME ,age ,deptid ) VALUES ((START+i) ,rand_string(6)   , rand_num(30,50),rand_num(1,10000));  
 	-- 每次emp得到的是随机字符串的长度为6，数字从age-30，50，和从id-1-1000的数据
 	UNTIL i = max_num  
 	END REPEAT;  -- 结束循环
 	COMMIT;  -- 提交一次即可
 END$$ 
 
#删除
# DELIMITER ;
# drop PROCEDURE insert_emp;
 
2、创建往dept表中插入数据的存储过程
#执行存储过程，往dept表添加随机数据
DELIMITER $$
CREATE PROCEDURE `insert_dept`(  max_num INT )
BEGIN  
DECLARE i INT DEFAULT 0;   
 	SET autocommit = 0;    
 	REPEAT  
 	SET i = i + 1;  
 INSERT INTO dept ( deptname,address,ceo ) VALUES (rand_string(8),rand_string(10),rand_num(1,500000));  
 	UNTIL i = max_num  
 	END REPEAT;  
 	COMMIT;  
 END$$
 

# 五、调用存储过程
 #执行存储过程，往dept表添加1万条数据
DELIMITER ; -- delimiter ; 则结束符号为“;”
CALL insert_dept(10000); 

#执行存储过程，往emp表添加50万条数据
DELIMITER ;
CALL insert_emp(100000,500000); 

# 六、批量删除某个表上的所有索引 ,批量删除索引工具，不需要会，了解就好
1、存储过程：（不需要会，了解就好）
DELIMITER $$
CREATE  PROCEDURE `proc_drop_index`(dbname VARCHAR(200),tablename VARCHAR(200))
BEGIN
       DECLARE done INT DEFAULT 0;
       DECLARE ct INT DEFAULT 0;
       DECLARE _index VARCHAR(200) DEFAULT '';
       -- 查询索引，找出删除的索引
       DECLARE _cur CURSOR FOR  SELECT   index_name   FROM information_schema.STATISTICS   WHERE table_schema=dbname AND table_name=tablename AND seq_in_index=1 AND    index_name <>'PRIMARY'  ;
      
       DECLARE  CONTINUE HANDLER FOR NOT FOUND set done=2 ;   
        -- 用游标找只想要删除的索引指针
        OPEN _cur; -- 打开游标
        FETCH   _cur INTO _index; -- 取出游标的数据
        WHILE  _index<>'' DO   -- 但凡该数据不能与没有长度的字符串，就因该删除
        		 -- 索引转换成mysql语句
               SET @str = CONCAT("drop index ",_index," on ",tablename ); 
               PREPARE sql_str FROM @str ; -- 预编译
               EXECUTE  sql_str; -- 执行，执行之后，该mysql语句就删除了
               DEALLOCATE PREPARE sql_str;  -- deallocate prepare
               SET _index=''; 
               FETCH   _cur INTO _index; 
        END WHILE;
   CLOSE _cur;
   END$$
2、执行存储过程
CALL proc_drop_index("dept","emp");  

```

## 2、如何删除索引

```mysql
1、查询索引名
2、取出索引名
3、怎么把自负串转换成sql
4、删除索引



1 查出该表有哪些索引，索引名-->集合（能删除的索引）

SHOW INDEX FROM t_emp -- 这样能找出索引信息，但不能取用
元数据：meta DATA  描述数据的数据

-- 用以下语句
SELECT index_name  FROM information_schema.STATISTICS WHERE table_name='emp' AND table_schema='mydb'
 AND index_name <>'PRIMARY' AND seq_in_index = 1

-- 从information_schema.statistics总找表名table_name为‘查询的索引表明’and 表所存在的库名table_schema='库名' and index_name <>'parimary' and seq_in_index=1; 主键索引不能删，并且seq序列等于一的可以删除

2 如何循环集合（取出索引名）-- 不需要会，了解就好
 CURSOR 游标  -- cursor 游标
 FETCH xxx INTO xxx -- fetch xxx into xxx


3 如何让mysql执行一个字符串
PREPARE 预编译 XXX  -- prepare 预编译 xxx
EXECUTE         -- execute 执行，删除了mysql语句，也就是索引被删除了


CALL proc_drop_index ('mydb','emp'); // 执行批量删除的索引名
CALL proc_drop_index ('mydb','dept'); // 执行批量删除的索引名

```

## 3、单表索引优化

```mysql
# 查看没有建立索引之前执行查询语句会造成什么，no_cache，就表示不会给缓冲带来负担
explain select sql_no_cache * from emp where emp.age = 30; -- 这样直接用select查的话,全表扫描，效率不高，建立索引
create index idx_age on emp(age);-- 效率高了

explain select sql_no_cache * from emp where emp.age = 30 and deptid = 4;
create index idx_age_depid on emp(age,deptid);

explain select sql_no_cache * from emp where emp.age = 30 and depti = 4 and emp.name = 'abcd';
create index idx_age_depid_name on emp(age,deptid,name);

explain select sql_no_cache * from emp where deptid = 4 and emp.age = 30 -- 顺序可乱
create index idx_age_depid on emp(age,deptid);

# 2、索引的数据结构
explain select sql_no_cache * from emp where emp.age = 30 and emp.name = 'abcd';
create index idx_age_depid_name on emp(age,deptid,name); -- deptid没了，命中边少

explain select sql_no_cache * from emp where  depti = 4 and emp.name = 'abcd';
create index idx_age_depid_name on emp(age,deptid,name); -- 首字段没了，一个也命中不了


# 3、这两条sql哪种写法更好
EXPLAIN  SELECT SQL_NO_CACHE * FROM emp WHERE   emp.name  LIKE 'abc%' 
 
EXPLAIN   SELECT SQL_NO_CACHE * FROM emp WHERE   LEFT(emp.name,3)  = 'abc'

   -- 建立索引
create index idx_age on emp(name);-- 效率高了,第一个变快了，第二个没变化，因为有left函数的存在，使索引失效了

# 4、如果系统经常出现的sql如下：
 
 EXPLAIN SELECT  SQL_NO_CACHE * FROM emp WHERE emp.age=30 AND emp.deptId>20 AND emp.name = 'abc' ; 
 
create index idx_age_name_deptid on emp(age,deptId,name) -- 建立索引
 -- 注意点，范围查询的字段，后面的索引会失效，所以在建立索引的时候，把范围查询的字段放在最后
create index idx_age_name_deptid on emp(age,name,deptid) -- 正确的索引方式

# 5、<> ，!= ,in not null 表示否定的，消极的字眼，索引都会失效

EXPLAIN SELECT SQL_NO_CACHE * FROM emp WHERE   emp.name <>  'abc' 
CREATE INDEX idx_name ON emp(NAME)

# 6、is null 索引不会失效，但是is not null会失效
EXPLAIN SELECT * FROM emp WHERE age IS NULL
  
EXPLAIN SELECT * FROM emp WHERE age IS NOT NULL
CREATE INDEX idx_name ON emp(age)

# 7、like以通配符开头('%abc...')mysql索引失效会变成全表扫描的操作

# 8、数据类型不匹配，索引失效
```

索引结构（通路不可断）

![image-20210922173403079](mysql%E9%AB%98%E7%BA%A7.assets/image-20210922173403079.png)

## 4、关联、子查询优化

```markdown
class(id, card)
book(bookid,card)
```



```mysql
一、建表
CREATE TABLE IF NOT EXISTS `class` (
`id` INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
`card` INT(10) UNSIGNED NOT NULL,
PRIMARY KEY (`id`)
);
CREATE TABLE IF NOT EXISTS `book` (
`bookid` INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
`card` INT(10) UNSIGNED NOT NULL,
PRIMARY KEY (`bookid`)
);
 
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
 
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));

二、案例
# 下面开始explain分析
EXPLAIN SELECT * FROM class LEFT JOIN book ON class.card = book.card;
#结论：type 有All
 
# 添加索引优化
ALTER TABLE `book` ADD INDEX Y ( `card`); -- 被驱动表建立索引，有意义
alter table `class` add index X(`card`); -- 驱动表建立索引，没意义
 drop index Y on book;
 
换成inner join
   -- inner join 系统会自己选则被驱动表，把有索引的当成被驱动表，没索引的当成驱动表
   # 注意：当有两个表的数据，一个表数据大，一个表数据小，把小表当成驱动表，大表当成被驱动表，当然用inner join的话，mysql会自动地把小表当成被驱动表
   
delete from class where id<5;
 
# 第2次explain
EXPLAIN SELECT * FROM class LEFT JOIN book ON class.card = book.card;
#可以看到第二行的 type 变为了 ref,rows 也变成了优化比较明显。
#这是由左连接特性决定的。LEFT JOIN 条件用于确定如何从右表搜索行,左边一定都有,
#所以右边是我们的关键点,一定需要建立索引。
 
# 删除旧索引 + 新建 + 第3次explain
DROP INDEX Y ON book;
ALTER TABLE class ADD INDEX X (card);
EXPLAIN SELECT * FROM class LEFT JOIN book ON class.card = book.card;
 
三、建议
子查询尽量不要放在被驱动表，有可能使用不到索引

能关联的尽量直接关联，不用子查询

尽量不要使用not in  或者 not exists
```

![image-20210922181107690](mysql%E9%AB%98%E7%BA%A7.assets/image-20210922181107690.png)

## **5、8个经典的子查询优化索引**

```mysql
    CALL proc_drop_index('mydb','emp');
    CALL proc_drop_index('mydb','dept');
    -- 以下的索引，mysql会选择最优的
CREATE INDEX idx_age_empno_name ON emp(age,empno,NAME);
create index idx_age_eno on emp(age,empno); 
CREATE INDEX idx_age_name ON emp(age,NAME);
#1、列出自己的掌门比自己年龄小的人员

    SELECT a.`name`,a.`age`,c.`name` ceoname,c.`age` ceoage FROM 
    t_emp a 
    LEFT JOIN t_dept b ON a.`deptId`= b.`id` 
    LEFT JOIN t_emp c ON b.`CEO`= c.`id`
    WHERE c.`age`<a.`age`


    
    #优化  
    EXPLAIN SELECT SQL_NO_CACHE a.`name`,a.`age`,c.`name` ceoname,c.`age` ceoage FROM 
    emp a 
    LEFT JOIN dept b ON a.`deptId`= b.`id` 
    LEFT JOIN emp c ON b.`CEO`= c.`id`
    WHERE c.`age`<a.`age`
    
    CREATE INDEX idx_age ON emp(age)
    

#2、列出所有年龄低于自己门派平均年龄的人员

SELECT c.`name`,c.`age`,aa.age FROM t_emp c INNER JOIN
(
    SELECT a.`deptId`,AVG(a.`age`)age FROM t_emp a
    WHERE a.`deptId` IS NOT NULL
    GROUP BY a.`deptId`
 )aa ON c.`deptId`=aa.deptid 
 WHERE c.`age`< aa.age

#优化 

EXPLAIN SELECT SQL_NO_CACHE c.`name`,c.`age`,aa.age FROM emp c INNER JOIN
(
    SELECT a.`deptId`,AVG(a.`age`)age FROM emp a
    WHERE a.`deptId` IS NOT NULL
    GROUP BY a.`deptId`
 )aa ON c.`deptId`=aa.deptid 
 WHERE c.`age`< aa.age
 
 CREATE INDEX idx_deptid ON emp(deptid)
 
  CREATE INDEX idx_deptid_age ON emp(deptid,age)




#3、列出至少有2个年龄大于40岁的成员的门派

 SELECT b.`deptName`,COUNT(*) FROM t_emp a 
 INNER JOIN t_dept b ON b.`id` = a.`deptId`
 WHERE a.age >40
 GROUP BY b.`deptName`,b.`id` 
 HAVING COUNT(*)>=2
 
 #优化 straight_join不管表中是不是一primary key的关键字，只要用到它，它的前面使驱动表，后面使被驱动表
 EXPLAIN SELECT SQL_NO_CACHE b.`deptName`,COUNT(*) FROM  
dept b STRAIGHT_JOIN emp a  ON b.`id` = a.`deptId`
 WHERE a.age >40
 GROUP BY b.`deptName`,b.`id` 
 HAVING COUNT(*)>=2
 
 CREATE INDEX  idx_deptid_age ON emp(deptid,age)
 CREATE INDEX  idx_deptname ON dept(deptname)

  
 STRAIGHT_JOIN 强制确定驱动表和被驱动表 1、概念非常明确 2、对数据量的比例非常明确（明确那个表是小表那个表是大表）
 如果不用它，那么含primary key的表，只能作为被驱动表，尽管他是小表

#4、至少有2位非掌门人成员的门派
SELECT * FROM t_emp a WHERE a.id NOT IN
{-- 子查询，索引失效， 要把子查询尽可能的弄成关联查询，避免索引失效
 SELECT b.`ceo` FROM t_dept b WHERE b.`ceo`IS NOT NULL
}  

NOT IN -->LEFT JOIN xxx ON xx WHERE xx IS NULL


SELECT c.deptname,  c.id,COUNT(*) FROM t_emp a 
INNER JOIN t_dept c ON a.`deptId` =c.`id`
LEFT JOIN t_dept b ON a.`id`=b.`ceo`
WHERE b.`id` IS NULL
GROUP BY c.`id` ,c.deptname
HAVING COUNT(*)>=2

#优化 

EXPLAIN SELECT SQL_NO_CACHE c.deptname,  c.id,COUNT(*) 
FROM  dept c STRAIGHT_JOIN emp a 
  ON a.`deptId` =c.`id`
LEFT JOIN dept b ON a.`id`=b.`ceo`
WHERE b.`id` IS NULL
GROUP BY c.deptname,c.`id` 
HAVING COUNT(*)>=2

CREATE INDEX idx_ceo_deptnam ON dept(ceo,deptname)
CREATE INDEX idx_deptnam ON dept(deptname)
CREATE INDEX idx_deptid ON emp(deptid)

SELECT b.`id`,b.`deptName` ,COUNT(*) FROM t_emp a INNER JOIN  t_dept b ON a.`deptId`= b.`id`
GROUP BY b.`deptName`,b.`id`

SELECT b.`id`,b.`deptName`, COUNT(*) FROM emp a INNER JOIN  dept b ON a.`deptId`= b.`id`
GROUP BY b.`deptName`,b.`id`

UPDATE t_dept SET deptname='明教' WHERE id=5

#5、列出全部人员，并增加一列备注“是否为掌门”，如果是掌门人显示是，不是掌门人显示否
CASE WHEN
IF
 
SELECT  a.`name`, CASE WHEN b.`id` IS NULL THEN '否' ELSE '是' END '是否为掌门'
FROM  t_emp a 
LEFT JOIN t_dept b ON a.`id`=b.`ceo`  
 


#6、列出全部门派，并增加一列备注“老鸟or菜鸟”，若门派的平均值年龄>50显示“老鸟”，否则显示“菜鸟”

SELECT b.`deptName`,
IF (AVG(a.age)>50,'老鸟','菜鸟')'老鸟or菜鸟'
 FROM t_emp a
INNER JOIN t_dept b ON a.`deptId`= b.`id`
 GROUP BY b.`id` ,b.`deptName`

#7、显示每个门派年龄最大的人

SELECT NAME,age FROM t_emp a
INNER JOIN
(
SELECT deptid,MAX(age) maxage
FROM t_emp
WHERE deptid IS NOT NULL
GROUP BY deptid
) aa ON a.`age`= aa.maxage AND a.`deptId`=aa.deptid

#优化 
EXPLAIN SELECT SQL_NO_CACHE NAME,age FROM emp a
INNER JOIN
(
SELECT deptid,MAX(age) maxage
FROM emp
WHERE deptid IS NOT NULL
GROUP BY deptid
) aa ON a.`age`= aa.maxage AND a.`deptId`=aa.deptid


CREATE INDEX idx_deptid_age ON emp(deptid,age)


#错例
SELECT b.`deptName`,a.`name`,MAX(a.`age`)FROM t_dept b
   LEFT JOIN t_emp a ON b.`id`=a.`deptId`
   WHERE a.name IS NOT NULL
   GROUP BY b.`deptName`


UPDATE t_emp SET age=100 WHERE id =2



#8、显示每个门派年龄第二大的人
SET @rank=0;
SET @last_deptid=0;
SELECT a.deptid,a.name,a.age
 FROM(    
    SELECT t.*,
     IF(@last_deptid=deptid,@rank:=@rank+1,@rank:=1) AS rk, -- 意思是把if条件里面的rand的值赋给rk
     @last_deptid:=deptid AS last_deptid -- 同理 把last_deptid的值给到last_deptid
    FROM t_emp t
    ORDER BY deptid,age DESC
    
 )a WHERE a.rk=2;

#分组排序
SET @rank=0;
SET @last_deptid=0;
SELECT * FROM
(
 SELECT t.*,
     IF(@last_deptid=deptid,@rank:=@rank+1,@rank:=1) AS rk,
     @last_deptid:=deptid AS last_deptid
    FROM t_emp t
    ORDER BY deptid,age DESC
) a WHERE a.rk <=1


#oracle rank() over()

UPDATE t_emp SET age=100 WHERE id =1

SET @rank=0;
SET @last_deptid=0;
SET @last_age=0;

 SELECT t.*,
     IF(@last_deptid=deptid,
     IF(@last_age = age,@rank,@rank:=@rank+1)
     ,@rank:=1) AS rk,
     @last_deptid:=deptid AS last_deptid,
     @last_age :=age AS last_age
    FROM t_emp t
    ORDER BY deptid,age DESC
```

## 6、查询优化

```mysql
 create index idx_age_deptid_name on emp (age,deptid,name) -- 创建索引
 
以下  是否能使用到索引，能否去掉using filesort
 
 
1、explain  select SQL_NO_CACHE * from emp order by age,deptid; 
 
 # 没有过滤条件，必排序，不会用到索引排序
 
2、 explain  select SQL_NO_CACHE * from emp order by age,deptid limit 10; 

ORDER BY子句，尽量使用Index方式排序,避免使用FileSort方式排序 （limit也是顾虑条件）

#无过滤 不索引（不索引排序）
 
3、  explain  select * from emp where age=45 order by deptid; -- age=45属于索引过滤条件
 
4、explain  select * from emp where age=45 order by   deptid,name; 
 
5、explain  select * from emp where age=45 order by  deptid,empno;
 
6、explain  select * from emp where age=45 order by  name,deptid;
 
7、 explain select * from emp where deptid=45 order by age;
 
#顺序错，必排序（非索引排序）
 
8、  explain select * from emp where age=45 order by  deptid desc, name desc ;
 
9、 explain select * from emp where age=45 order by  deptid asc, name desc ;
 #方向反 必排序（要么同时升序，要么同时降序，否则不会用到索引排序）


三、索引的选则
执行案例前先清除emp上的索引，只留主键
#查询 年龄为30岁的，且员工编号小于101000的用户，按用户名称排序
  
SELECT SQL_NO_CACHE * FROM emp WHERE age =30 AND empno <101000 ORDER BY NAME ;
 CREATE INDEX idx_age_empno_name ON emp(age,empno,NAME);
create index idx_age_eno on emp(age,empno); 
CREATE INDEX idx_age_name ON emp(age,NAME);

#结论：很显然,type 是 ALL,即最坏的情况。Extra 里还出现了 Using filesort,也是最坏的情况。优化是必须的。
 
#开始优化：
思路：  尽量让where的过滤条件和排序使用上索引
但是一共两个字段(deptno,empno)上有过滤条件，一个字段(ename)有索引 
1、我们建一个三个字段的组合索引可否？

 CREATE INDEX idx_age_empno_name ON emp(age,empno,NAME);
我们发现using filesort 依然存在，所以name 并没有用到索引。
原因是因为empno是一个范围过滤，所以索引后面的字段不会再使用索引了。
 
所以我们建一个3值索引是没有意义的 
那么我们先删掉这个索引，DROP INDEX idx_age_empno_name ON emp
 
为了去掉filesort我们可以把索引建成
CREATE INDEX idx_age_name ON emp(age,NAME);

# 也就是说empno 和name这个两个字段我只能二选其一。
 这样我们优化掉了 using filesort。
 执行一下sql


速度果然提高了4倍。
 
 .......
 
但是 
如果我们选择那个范围过滤，而放弃排序上的索引呢
建立 
DROP INDEX idx_age_name ON emp
create index idx_age_eno on emp(age,empno); 


 果然出现了filesort，而且type还是range光看字面其实并不美好。
我们来执行以下sql

 
结果竟然有 filesort的 sql 运行速度，超过了已经优化掉 filesort的 sql ，而且快了好多倍。何故？
 
原因是所有的排序都是在条件过滤之后才执行的，所以如果条件过滤了大部分数据的话，几百几千条数据进行排序其实并不是很消耗性能，即使索引优化了排序但实际提升性能很有限。  相对的 empno<101000 这个条件如果没有用到索引的话，要对几万条的数据进行扫描，这是非常消耗性能的，所以索引放在这个字段上性价比最高，是最优选择。
 
结论： 当范围条件和group by 或者 order by  的字段出现二选一时 ，优先观察条件字段的过滤数量，如果过滤的数据足够多，而需要排序的数据并不多时，优先把索引放在范围字段上。反之，亦然。
group by 就算没用上过滤条件，也能进行索引。
```



