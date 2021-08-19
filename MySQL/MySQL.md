**SQL语句分类**
**1)Data Definition Language (DDL数据定义语言) 如：建库，建表**
**2)Data Manipulation Language(DML数据操纵语言)，如：对表中的记录操作增删改**
**3)Data Query Language(DQL 数据查询语言)，如：对表中的查询操作**
**4)Data Control Language(DCL 数据控制语言)，如：对用户权限的设置**

# DataBase

### 创建数据库

![create_database](https://raw.githubusercontent.com/isIvanTsui/img/master/create_database.png)

### 查看所有的数据库

show databases;

-- 查看某个数据库的定义信息
show create database db3;

将db3数据库的字符集改成utf8
alter database db3 character set utf8;

删除数据库的语法
DROP DATABASE 数据库名;

-- 查看正在使用的数据库
select database();



# Table

### 创建表

![create_table](https://raw.githubusercontent.com/isIvanTsui/img/master/create_table.png)

查看某个数据库中的所有表
SHOW TABLES;

**查看表结构**
**DESC 表名;**

查看创建表的SQL语句

SHOW CREATE TABLE 表名;

快速创建一个表结构相同的表
CREATE TABLE 新表名 LIKE 旧表名;

判断表是否存在，如果存在则删除表
DROP TABLE IF EXISTS 表名;

**添加表列ADD**
**ALTER TABLE 表名 ADD 列名 类型;**

**修改列类型MODIFY**
**ALTER TABLE 表名 MODIFY列名 新的类型;**

**修改列名 CHANGE**
**ALTER TABLE 表名 CHANGE 旧列名 新列名 类型;**

**删除列 DROP**
**ALTER TABLE 表名 DROP 列名;**

修改表名
RENAME TABLE 表名 TO 新表名;

修改字符集character set
ALTER TABLE 表名 character set 字符集;

### 向表中插入数据

![insert_tablepng](https://raw.githubusercontent.com/isIvanTsui/img/master/insert_tablepng.png)

将表名2中的所有的列复制到表名1中
INSERT INTO 表名1 SELECT * FROM 表名2;
只复制部分列
INSERT INTO 表名1(列1, 列2) SELECT 列1, 列2 FROM student;

### 更新表中数据

![update_table](https://raw.githubusercontent.com/isIvanTsui/img/master/update_table.png)

### 删除表记录

![delete_table](https://raw.githubusercontent.com/isIvanTsui/img/master/delete_table.png)



### 表的查询

**查询指定列并且结果不出现重复数据**
**SELECT DISTINCT 字段名 FROM 表名;**

某列数据和固定值运算
SELECT 列名1 + 固定值 FROM 表名;

某列数据和其他列数据参与运算
SELECT 列名1 + 列名2 FROM 表名;





![operation_characterpng](https://raw.githubusercontent.com/isIvanTsui/img/master/operation_characterpng.png)

![logical_operator](https://raw.githubusercontent.com/isIvanTsui/img/master/logical_operator.png)

![keywords_in](https://raw.githubusercontent.com/isIvanTsui/img/master/keywords_in.png)

![tik1](https://raw.githubusercontent.com/isIvanTsui/img/master/tik1.png)



### 表的约束

![constraint](https://raw.githubusercontent.com/isIvanTsui/img/master/constraint.png)

![create_primarykey](https://raw.githubusercontent.com/isIvanTsui/img/master/create_primarykey.png)



自增

创建表时指定起始值
CREATE TABLE 表名(
列名 int primary key AUTO_INCREMENT
) AUTO_INCREMENT=起始值;

创建好以后修改起始值
ALTER TABLE 表名 AUTO_INCREMENT=起始值;
alter table st4 auto_increment = 2000;



![foreign_key](https://raw.githubusercontent.com/isIvanTsui/img/master/foreign_key.png)

create table employee(
id int primary key auto_increment,
name varchar(20),
age int,
dep_id int, -- 外键对应主表的主键
-- 创建外键约束
**constraint emp_depid_fk foreign key (dep_id) references department(id)**
)

**-- 删除employee表的emp_depid_fk外键**
**alter table employee drop foreign key emp_depid_fk;**

**-- 在employee表情存在的情况下添加外键**
**alter table employee add constraint emp_depid_fk**
**foreign key (dep_id) references department(id);**

# 存储引擎

![MySQL_Memory_Engine](https://raw.githubusercontent.com/isIvanTsui/img/master/MySQL_Memory_Engine.jpeg)



# 面试问题

**MySQL架构**

![MySQL_architecture](https://raw.githubusercontent.com/isIvanTsui/img/master/MySQL_architecture.png)

![sql_invoke](https://raw.githubusercontent.com/isIvanTsui/img/master/sql_invoke.png)

MySQL 整个查询执行过程，总的来说分为 6 个步骤 :

SQL执行步骤：请求、缓存、SQL解析、优化SQL查询、调用引擎执行，返回结果
    1、连接：客户端向 MySQL 服务器发送一条查询请求，与connectors交互：连接池认证相关处理。
    2、缓存：服务器首先检查查询缓存，如果命中缓存，则立刻返回存储在缓存中的结果，否则进入下一阶段
    3、解析：服务器进行SQL解析(词法语法)、预处理。
    4、优化：再由优化器生成对应的执行计划。
    5、执行：MySQL 根据执行计划，调用存储引擎的 API来执行查询。
    6、结果：将结果返回给客户端，同时缓存查询结果。

### 1、MySQL语句的执行顺序

1、from 子句组装来自不同数据源的数据；
2、where 子句基于指定的条件对记录行进行筛选；
3、group by 子句将数据划分为多个分组；
4、使用聚集函数进行计算；
5、使用 having 子句筛选分组；
6、计算所有的表达式；
7、select 的字段；
8、使用 order by 对结果集进行排序。

### 2、MySQL聚合函数

| [AVG([distinct\] expr)](https://www.cnblogs.com/geaozhang/p/6745147.html#sum-avg) | 求平均值     |
| ------------------------------------------------------------ | ------------ |
| [COUNT({*\|[distinct\] } expr)](https://www.cnblogs.com/geaozhang/p/6745147.html#count) | 统计行的数量 |
| [MAX([distinct\] expr)](https://www.cnblogs.com/geaozhang/p/6745147.html#max-min) | 求最大值     |
| [MIN([distinct\] expr)](https://www.cnblogs.com/geaozhang/p/6745147.html#max-min) | 求最小值     |
| [SUM([distinct\] expr)](https://www.cnblogs.com/geaozhang/p/6745147.html#sum-avg) | 求累加和     |

### 3、左连接、右连接、内连接

```sql
-- 左外链接 left
SELECT * FROM students st LEFT JOIN score sc ON st.sid=sc.stu_id;
-- 右外链接 right
SELECT * FROM students st RIGHT JOIN score sc ON st.sid=sc.stu_id;
-- 显示内连接所有数据:
SELECT * FROM students st INNER JOIN score sc ON st.sid=sc.stu_id;
```

左连接（左外连接）：以左表作为基准进行查询，左表数据会全部显示出来，右表如果和左表匹配的数据则显示相应字段的数据，如果不匹配则显示为 null。
右连接（右外连接）：以右表作为基准进行查询，右表数据会全部显示出来，左表如果和右表匹配的数据则显示相应字段的数据，如果不匹配则显示为 null。
全连接：先以左表进行左外连接，再以右表进行右外连接

### 4、数据库的三范式

第一范式(1NF)：每个列都不可以再拆分。
第二范式(2NF)：在第一范式的基础上，非主键列完全依赖于主键，而不能是依赖于主键的一部分。
第三范式(3NF)：在第二范式的基础上，非主键列只依赖于主键，不依赖于其他非主键。

**1** **第一范式(确保每列保持原子性)**

第一范式是最基本的范式。如果数据库表中的所有字段值都是不可分解的原子值，就说明该数据库表满足了第一范式。

第一范式的合理遵循需要根据系统的实际需求来定。比如某些数据库系统中需要用到“地址”这个属性，本来直接将“地址”属性设计成一个数据库表的字段就行。但是如果系统经常会访问“地址”属性中的“城市”部分，那么就非要将“地址”这个属性重新拆分为省份、城市、详细地址等多个部分进行存储，这样在对地址中某一部分操作的时候将非常方便。这样设计才算满足了数据库的第一范式，如下表所示。

![img](https://pic002.cnblogs.com/images/2012/270324/2012040114023352.png)

上表所示的用户信息遵循了第一范式的要求，这样在对用户使用城市进行分类的时候就非常方便，也提高了数据库的性能。

​         

**2** **．第二范式(确保表中的每列都和主键相关)**

第二范式在第一范式的基础之上更进一层。第二范式需要确保数据库表中的每一列都和主键相关，而不能只与主键的某一部分相关（主要针对联合主键而言）。也就是说在一个数据库表中，一个表中只能保存一种数据，不可以把多种数据保存在同一张数据库表中。

比如要设计一个订单信息表，因为订单中可能会有多种商品，所以要将订单编号和商品编号作为数据库表的联合主键，如下表所示。

 **订单信息表**

![img](https://pic002.cnblogs.com/images/2012/270324/2012040114063976.png)

这样就产生一个问题：这个表中是以订单编号和商品编号作为联合主键。这样在该表中商品名称、单位、商品价格等信息不与该表的主键相关，而仅仅是与商品编号相关。所以在这里违反了第二范式的设计原则。

而如果把这个订单信息表进行拆分，把商品信息分离到另一个表中，把订单项目表也分离到另一个表中，就非常完美了。如下所示。

![img](https://pic002.cnblogs.com/images/2012/270324/2012040114082156.png)

这样设计，在很大程度上减小了数据库的冗余。如果要获取订单的商品信息，使用商品编号到商品信息表中查询即可。

​         

**3** **．第三范式(确保每列都和主键列直接相关,而不是间接相关)**

第三范式需要确保数据表中的每一列数据都和主键直接相关，而不能间接相关。

**第三范式需要确保数据表中的每一列数据都和主键直接相关，而不能间接相关。**

举例说明：

![students](https://raw.githubusercontent.com/isIvanTsui/img/master/students.png)

上表中，所有属性都完全依赖于学号，所以满足第二范式，但是“班主任性别”和“班主任年龄”直接依赖的是“班主任姓名”，

而不是主键“学号”，所以需做如下调整：

![students2](https://raw.githubusercontent.com/isIvanTsui/img/master/students2.png) ![9383439](https://raw.githubusercontent.com/isIvanTsui/img/master/9383439.png)

这样以来，就满足了第三范式的要求。

### 5、数据库的存储引擎

**InnoDB存储引擎**

InnoDB 是事务型数据库的首选引擎，支持事务安全表（ACID），支持行锁定和外键。MySQL5.5.5之后，InnoDB 作为默认的存储引擎，InnoDB 主要特性有：

支持事务
灾难恢复性好
为处理巨大数据量的最大性能设计
实现了缓冲管理，不仅能缓冲索引也能缓冲数据，并且会自动创建散列索引以加快数据的获取
支持外键完整性约束。存储表中的数据时，每张表的存储都按逐渐顺序存放，如果没有显示在表定义时指定主键，InnoDB会为每一行生成一个6B的ROWID,并以此作为主键。
被用在众多需要高性能的大型数据库站点上
**MyISAM存储引擎**

MyISAM 基于 ISAM 的存储引擎，并对其进行扩展。它是在Web、数据存储和其他应用环境下最常使用的存储引擎之一。MyISAM 拥有较高的插入、查询速度，但不支持事务。在 MySQL5.5.5 之前的版本中，MyISAM 是默认的存储引擎。MyISAM 主要特性有：

不支持事务
使用表级锁，并发性差
主机宕机后，MyISAM表易损坏，灾难恢复性不佳
可以配合锁，实现操作系统下的复制备份、迁移
只缓存索引，数据的缓存是利用操作系统缓冲区来实现的。可能引发过多的系统调用且效率不佳
数据紧凑存储，因此可获得更小的索引和更快的全表扫描性能
可以把数据文件和索引文件放在不同目录
使用 MyISAM 引擎创建数据库，将产生3个文件。文件的名字以表的名字开始，扩展名指出文件类型：frm 文件存储表定义，数据文件的扩展名为 .MYD（MYData），索引文件的扩展名是 .MYI（MYIndex）。

**MEMORY存储引擎**

MEMORY 存储引擎将表中的数据存储在内存中，为查询和引用其他表数据提供快速访问。MEMORY 主要特性有：

使用表级锁，虽然内存访问快，但如果频繁的读写，表级锁会成为瓶颈
只支持固定大小的行。Varchar类型的字段会存储为固定长度的Char类型，浪费空间
不支持TEXT、BLOB字段。当有些查询需要使用到临时表（使用的也是MEMORY存储引擎）时，如果表中有TEXT、BLOB字段，那么会转换为基于磁盘的MyISAM表，严重降低性能
由于内存资源成本昂贵，一般不建议设置过大的内存表，如果内存表满了，可通过清除数据或调整内存表参数来避免报错
服务器重启后数据会丢失，复制维护时需要小心

### 6、MyISAM和InnoDB的区别

1. <span style="color:red">myisam是默认表类型不是事物安全的；innodb支持事物。</span>
2. <span style="color:red">myisam不支持外键；Innodb支持外键。</span>
3. <span style="color:red">myisam支持表级锁（不支持高并发，以读为主）；innodb支持行锁（共享锁，排它锁，意向锁），粒度更小，但是在执行不能确定扫描范围的sql语句时，innodb同样会锁全表。</span>
4. <span style="color:red">执行大量select，myisam是最好的选择；执行大量的update和insert最好用innodb。</span>
5. <span style="color:red">myisam在磁盘上存储上有三个文件.frm(存储表定义)  .myd（存储表数据）  .myi（存储表索引）；innodb磁盘上存储的是表空间数据文件和日志文件，innodb表大小只受限于操作系统大小。</span>
6. <span style="color:red">myisam使用非聚集索引，索引和数据分开，只缓存索引；innodb使用聚集索引，索引和数据存在一个文件。</span>
7. <span style="color:red">myisam保存表具体行数；innodb不保存。</span>
8. <span style="color:red">delete from table时，innodb不会重新简历表，而会一行一行的删除。</span>

**如何选择：**

    1. **是否要支持事务，如果要请选择innodb，如果不需要可以考虑MyISAM；**

2. **如果表中绝大多数都只是读查询，可以考虑MyISAM，如果既有读也有写，请使用InnoDB。**

3. **系统奔溃后，MyISAM恢复起来更困难，能否接受；**

4. **MySQL5.5版本开始Innodb已经成为Mysql的默认引擎(之前是MyISAM)，说明其优势是有目共睹的，如果你不知道用什么，那就用InnoDB，至少不会差。**



### 7、MySQL中常见索引

MySQL中常见索引有：

- 普通索引
- 唯一索引
- 主键索引
- 组合索引

**一、普通索引(index)**

普通所以只有一个功能，就是加快查找速度。操作如下

1、先创建一个表

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
create table tab1(
    nid int not null auto_increment primary key,
    name varchar(32) not null,
    email varchar(64) not null,
    extra text,
    index ix_name (name)
)
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

2、创建索引

```
create index 索引名称 on 表名(列名)
```

3、删除索引

```
drop 索引名称 on 表名;
```

4、查看索引

```
show index from 表名;
```

5、注意事项(对于创建索引时如果是BLOB 和 TEXT 类型，必须指定length。)

```
create index index_name on tab1(extra(32));
```

**二、唯一索引(unique)**

唯一性索引unique index和一般索引normal index最大的差异就是在索引列上增加了一层唯一约束。添加唯一性索引的数据列可以为空，但是只要存在数据值，就必须是唯一的。

1、创建表+唯一索引

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
create table tab2(
    nid int not null auto_increment primary key,
    name varchar(32) not null,
    email varchar(64) not null,
    extra text,
    unique ix_name (name)  -- 重点在这里
)
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

2、创建索引

```
create unique index 索引名 on 表名(列名)
```

3、删除索引

```
drop unique index 索引名 on 表名
```

三、主键索引

在数据库关系图中为表定义一个主键将自动创建主键索引，主键索引是唯一索引的特殊类型。主键索引要求主键中的每个值是唯一的。当在查询中使用主键索引时，它还允许快速访问数据。数据不能为空

1、创建表+主键索引

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
create table in1(
    nid int not null auto_increment,
    name varchar(32) not null,
    email varchar(64) not null,
    extra text,
    primary key(nid),
    index zhang (name)
)
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

2、创建主键

```
alter table 表名 add primary key(列名);
```

3、删除主键

```
alter table 表名 drop primary key;
alter table 表名  modify  列名 int, drop primary key;
```

四、组合索引

组合索引，就是组合查询的意思嘛嘻嘻，将两列或者多列组合成一个索引进行查询

其应用场景为：频繁的同时使用n列来进行查询，如：where name = '张岩林' and email = 666。

1、创建表

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
create table in3(
    nid int not null auto_increment primary key,
    name varchar(32) not null,
    email varchar(64) not null,
    extra text
)
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

2、创建组合索引

```
create index ix_name_email on in3(name,email);
```

如上创建组合索引之后，查询有的会使用索引，有的不会：

- name and email  -- 使用索引
- name         -- 使用索引
- email         -- 不使用索引

其他注意事项

- `避免使用``select` `*`
- \```count``(1)或``count``(列) 代替 ``count``(*)`
- `创建表时尽量时 ``char` `代替 ``varchar`
- `表的字段顺序固定长度的字段优先`
- `组合索引代替多个单列索引（经常使用多个条件查询时）`
- `尽量使用短索引`
- `使用连接（``JOIN``）来代替子查询(Sub-Queries)`
- `连表时注意条件类型需一致`
- `索引散列值（重复少）不适合建索引，例：性别不适合`

### 8、count（*）、count（1）和count（列名）的区别

**1** **、执行效果上：** 

l count(*)包括了所有的列，相当于行数，在统计结果的时候，不会忽略列值为NULL  

l count(1)包括了忽略所有列，用1代表代码行，在统计结果的时候，不会忽略列值为NULL  

l count(列名)只包括列名那一列，在统计结果的时候，会忽略列值为空（这里的空不是只空字符串或者0，而是表示null）的计数，即某个字段值为NULL时，不统计。

 

**2** **、执行效率上：** 

l 列名为主键，count(列名)会比count(1)快  

l 列名不为主键，count(1)会比count(列名)快  

l 如果表多个列并且没有主键，则 count（1） 的执行效率优于 count（*）  

l 如果有主键，则 select count（主键）的执行效率是最优的  

l 如果表只有一个字段，则 select count（*）最优。

### 9、什么是存储过程？有哪些优缺点？

**什么是存储过程**：存储过程可以说是一个记录集吧，它是由一些T-SQL语句组成的代码块，这些T-SQL语句代码像一个方法一样实现一些功能（对单表或多表的增删改查），然后再给这个代码块取一个名字，在用到这个功能的时候调用他就行了。

存储过程的优点：

- **能够将代码封装起来**
- **保存在数据库之中**
- **让编程语言进行调用**
- **存储过程是一个预编译的代码块，执行效率比较高**
- **一个存储过程替代大量T_SQL语句 ，可以降低网络通信量，提高通信速率**

存储过程的缺点：

- **每个数据库的存储过程语法几乎都不一样，十分难以维护（不通用）**
- **业务逻辑放在数据库上，难以迭代**



### 10、drop、delete与truncate分别在什么场景之下使用？

我们来对比一下他们的区别：

drop table

- 1)属于DDL
- 2)不可回滚
- 3)不可带where
- 4)表内容和结构删除
- 5)删除速度快

truncate table

- 1)属于DDL
- 2)不可回滚
- 3)不可带where
- 4)表内容删除
- 5)删除速度快

delete from

- 1)属于DML
- 2)可回滚
- 3)可带where
- 4)表结构在，表内容要看where执行的情况
- 5)删除速度慢,需要逐行删除
- **不再需要一张表的时候，用drop**
- **想删除部分数据行时候，用delete，并且带上where子句**
- **保留表而删除所有数据的时候用truncate**

### 11、什么是事务？

事务简单来说：**一个Session中所进行所有的操作，要么同时成功，要么同时失败**

**ACID — 数据库事务正确执行的四个基本要素**

- 包含：原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）、持久性（Durability）。

**一个支持事务（Transaction）中的数据库系统，必需要具有这四种特性，否则在事务过程（Transaction processing）当中无法保证数据的正确性，交易过程极可能达不到交易。**

举个例子:**A向B转账，转账这个流程中如果出现问题，事务可以让数据恢复成原来一样【A账户的钱没变，B账户的钱也没变】。**

### 12、数据库的乐观锁和悲观锁是什么？

**悲观锁：**当要对数据库中的一条数据进行修改的时候，为了避免同时被其他人修改，最好的办法就是直接对该数据进行加锁以防止并发。这种借助数据库锁机制，在修改数据之前先锁定，再修改的方式被称之为悲观并发控制【Pessimistic Concurrency Control，缩写“PCC”，又名“悲观锁”】。

**乐观锁：**乐观锁是相对悲观锁而言的，乐观锁假设数据一般情况不会造成冲突，所以在数据进行提交更新的时候，才会正式对数据的冲突与否进行检测，如果冲突，则返回给用户异常信息，让用户决定如何去做。乐观锁适用于读多写少的场景，这样可以提高程序的吞吐量。

- 悲观锁：假定会发生并发冲突，屏蔽一切可能违反数据完整性的操作
  - **在查询完数据的时候就把事务锁起来，直到提交事务**
  - 实现方式：使用数据库中的锁机制
- 乐观锁：假设不会发生并发冲突，只在提交操作时检查是否违反数据完整性。
  - **在修改数据的时候把事务锁起来，通过version的方式来进行锁定**
  - 实现方式：使用version版本或者时间戳

悲观锁：

![img](https://segmentfault.com/img/remote/1460000013517922?w=686&h=367)

乐观锁：

![img](https://segmentfault.com/img/remote/1460000013517923?w=818&h=567)

### 13、超键、候选键、主键、外键分别是什么？

- 超键：**在关系中能唯一标识元组的属性集称为关系模式的超键**。一个属性可以为作为一个超键，多个属性组合在一起也可以作为一个超键。**超键包含候选键和主键**。
- **候选键(候选码)：是最小超键，即没有冗余元素的超键**。
- **主键(主码)：数据库表中对储存数据对象予以唯一和完整标识的数据列或属性的组合，也叫做选中的候选键**。一个数据列只能有一个主键，且主键的取值不能缺失，即不能为空值（Null）。
- **外键：在一个表中存在的另一个表的主键称此表的外键**。

> 比如一个小范围的所有人，没有重名的，考虑以下属性
>
> 身份证 姓名 性别 年龄
>
> 身份证唯一，所以是一个超键
> 姓名唯一，所以是一个超键
> （姓名，性别）唯一，所以是一个超键
> （姓名，性别，年龄）唯一，所以是一个超键
> --这里可以看出，超键的组合是唯一的，但可能不是最小唯一的
>
> --主键是选中的一个候选键

**候选码和主码：**

例子：邮寄地址（城市名，街道名，邮政编码，单位名，收件人）

- **它有两个候选键:{城市名，街道名} 和 {街道名，邮政编码}**
- **如果我选取{城市名，街道名}作为唯一标识实体的属性，那么{城市名，街道名} 就是主码(主键)**

### 14、SQL 约束有哪几种

- NOT NULL: 用于控制字段的内容一定不能为空（NULL）。
- UNIQUE: 控件字段内容不能重复，一个表允许有多个 Unique 约束。
- PRIMARY KEY: 也是用于控件字段内容不能重复，但它在一个表只允许出现一个。
- FOREIGN KEY: 用于预防破坏表之间连接的动作，也能防止非法数据插入外键列，因为它必须是它指向的那个表中的值之一。
- CHECK: 用于控制字段的值范围。

### 15、MyIASM和Innodb两种引擎所使用的索引的数据结构是什么

答案:都是B+树!

MyIASM引擎，B+树的数据结构中存储的内容实际上是实际数据的地址值。也就是说它的索引和实际数据是分开的，**只不过使用索引指向了实际数据。这种索引的模式被称为非聚集索引。**

Innodb引擎的索引的数据结构也是B+树，**只不过数据结构中存储的都是实际的数据，这种索引有被称为聚集索引**。

**myisam:**

![img](C:\Users\Administrator\Desktop\secret\img\myisam)

**innodb:**

![img](C:\Users\Administrator\Desktop\secret\img\innodb)

### 16、varchar和char的区别

**Char是一种固定长度的类型，varchar是一种可变长度的类型**

### 17、mysql有关权限的表都有哪几个

MySQL服务器通过权限表来控制用户对数据库的访问，权限表存放在mysql数据库里，由mysql_install_db脚本初始化。这些权限表分别user，db，table_priv，columns_priv和host。下面分别介绍一下这些表的结构和内容：

- user权限表：记录允许连接到服务器的用户帐号信息，里面的权限是全局级的。
- db权限表：记录各个帐号在各个数据库上的操作权限。
- table_priv权限表：记录数据表级的操作权限。
- columns_priv权限表：记录数据列级的操作权限。
- host权限表：配合db权限表对给定主机上数据库级操作权限作更细致的控制。这个权限表不受GRANT和REVOKE语句的影响。

### 18、视图和表的区别

视图是已经编译好的sql语句，是基于sql语句的结果集的可视化的表，而表不是
视图是窗口，表示内容
视图没有实际的物理记录，而表有
视图的建立和删除只影响视图本身，不影响对应的表

**两者的联系：**视图是基于基本表上建立的表，它的结构和内容都来自基本表，它依据基本表存在而存在。一个视图可以对应一个基本表，也可以对应多个基本表。

### 19、存储过程

- 存储过程就是具有名字的一段代码，用来完成一个特定的功能。

**创建“pro_add”的存储过程来求两数和:**

```sql
CREATE PROCEDURE pr_add( a INT, b INT ) BEGIN
	DECLARE c INT;
	IF a IS NULL THEN SET a = 0;
	END IF;
	IF b IS NULL THEN SET b = 0;
	END IF;
	SET c = a + b;
SELECT c as sum;
```

使用时：

```sql
call pr_add(1, 2);
```

### 20、触发器

**触发器**是一种特殊类型的存储过程，它不同于存储过程，主要是通过事件触发而被执行的，即不是主动调用而执行的；而存储过程则需要主动调用其名字执行

触发器：trigger，是指事先为某张表绑定一段代码，当表中的某些内容发生改变（增、删、改）的时候，系统会自动触发代码并执行。

**创建触发器：**

```sql
delimiter 自定义结束符号
create trigger 触发器名字 触发时间 触发事件 on 表 for each row
begin
    -- 触发器内容主体，每行用分号结尾
end
自定义的结束符合

delimiter ;
```

`on 表 for each`：触发对象，触发器绑定的实质是表中的所有行，因此当每一行发生指定改变时，触发器就会发生

- **触发时间**
  当 SQL 指令发生时，会令行中数据发生变化，而每张表中对应的行有两种状态：数据操作前和操作后

  - before：表中数据发生改变前的状态
  - after：表中数据发生改变后的状态
  - PS：如果 before 触发器失败或者语句本身失败，将不执行 after 触发器(如果有的话)

- ##### 触发事件

  触发器是针对数据发送改变才会被触发，对应的操作只有

  - INSERT
  - DELETE
  - UPDATE

**example：**

如果**订单表**发生数据插入，对应的商品库存应该减少。因此这里对订单表创建触发器

```sql
delimiter ##
-- 创建触发器
create trigger after_insert_order after insert on orders for each row
begin
    -- 更新商品表的库存，这里只指定了更新第一件商品的库存
    update goods set goods_num = goods_num - 1 where id = 1;
end
##

delimiter ;
```

