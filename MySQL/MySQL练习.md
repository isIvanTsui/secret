### 表和数据

> 学生表

```sql
drop table if exists student;
create table student(sid varchar(10),sname varchar(10),sage datetime,ssex varchar(10));
insert into Student values('01' , '赵雷' , '1990-01-01' , '男');
insert into Student values('02' , '钱电' , '1990-12-21' , '男');
insert into Student values('03' , '孙风' , '1990-12-20' , '男');
insert into Student values('04' , '李云' , '1990-12-06' , '男');
insert into Student values('05' , '周梅' , '1991-12-01' , '女');
insert into Student values('06' , '吴兰' , '1992-01-01' , '女');
insert into Student values('07' , '郑竹' , '1989-01-01' , '女');
insert into Student values('09' , '张三' , '2017-12-20' , '女');
insert into Student values('10' , '李四' , '2017-12-25' , '女');
insert into Student values('11' , '李四' , '2012-06-06' , '女');
insert into Student values('12' , '赵六' , '2013-06-13' , '女');
insert into Student values('13' , '孙七' , '2014-06-01' , '女');
```

> 课程表

```sql
drop table if exists course;
create table course(cid varchar(10),cname nvarchar(10),tid varchar(10));
insert into course values('01' , '语文' , '02');
insert into course values('02' , '数学' , '01');
insert into course values('03' , '英语' , '03');
```

> 教师表

```sql
drop table if exists teacher;
create table teacher(tid varchar(10),tname varchar(10));
insert into teacher values('01' , '张三');
insert into teacher values('02' , '李四');
insert into teacher values('03' , '王五');
```

> 成绩表

```sql
drop table if exists sc;
create table sc(sid varchar(10),cid varchar(10),score decimal(18,1));
insert into sc values('01' , '01' , 80);
insert into sc values('01' , '02' , 90);
insert into sc values('01' , '03' , 99);
insert into sc values('02' , '01' , 70);
insert into SC values('02' , '02' , 60);
insert into SC values('02' , '03' , 80);
insert into SC values('03' , '01' , 80);
insert into SC values('03' , '02' , 80);
insert into SC values('03' , '03' , 80);
insert into SC values('04' , '01' , 50);
insert into SC values('04' , '02' , 30);
insert into SC values('04' , '03' , 20);
insert into SC values('05' , '01' , 76);
insert into SC values('05' , '02' , 87);
insert into SC values('06' , '01' , 31);
insert into SC values('06' , '03' , 34);
insert into SC values('07' , '02' , 89);
insert into SC values('07' , '03' , 98);
```

### 开始查询

> 查询数学(cid=" 01 ")成绩比语文(cid=" 02 ")成绩高的学生的信息及课程分数

```sql
SELECT
	s.*,
	a.score AS '数学成绩',
	b.score AS '语文成绩' 
FROM
	( SELECT * FROM sc WHERE sc.cid = '01' ) a
	JOIN ( SELECT * FROM sc WHERE sc.cid = '02' ) b ON a.sid = b.sid
	JOIN student s ON a.sid = s.sid 
HAVING
	a.score > b.score;
```

> 查询存在数学成绩，但可能不存在语文成绩的学生的成绩

```sql
SELECT
	a.sid, a.cid, a.score as '数学成绩', b.cid, b.score as '语文成绩' 
FROM
	( SELECT * FROM sc WHERE sc.cid = '01' ) a
	LEFT JOIN ( SELECT * FROM sc WHERE sc.cid = '02' ) b ON a.sid = b.sid;
```

> 查询不存在数学成绩，但存在语文成绩的学生的成绩情况

```sql
SELECT
	* 
FROM
	( SELECT * FROM sc WHERE sc.cid = '01' ) a
	RIGHT JOIN ( SELECT * FROM sc WHERE sc.cid = '02' ) b ON a.sid = b.sid 
HAVING
	a.score IS NULL;
```

> 查询平均成绩大于等于 60 分的同学的学生编号和学生姓名和平均成绩

```sql
SELECT
	s.sid,
	s.sname,
	avg( sc.score ) 
FROM
	student s
	JOIN sc ON s.sid = sc.sid 
GROUP BY
	s.sid 
HAVING
	avg( sc.score ) >= 60;
```

> 查询在 SC 表存在成绩的学生信息

```sql
SELECT
	* 
FROM
	student s 
WHERE
	EXISTS ( SELECT sc.sid FROM sc WHERE sc.sid = s.sid );
```

> 查询所有同学的学生编号、学生姓名、选课总数、所有课程的成绩总和

```sql
select s.*,count(sc.cid),sum(sc.score) from sc right join student s on sc.sid = s.sid group by sc.sid;
```

> 查有成绩的学生信息

```sql
SELECT
	* 
FROM
	student s 
WHERE
	EXISTS ( SELECT DISTINCT sc.sid FROM sc WHERE sc.sid = s.sid );
```

> 查询'张三'老师所教的学生的学生信息

```sql
SELECT
	s.*,
	t.tname 
FROM
	teacher t
	JOIN course c ON c.tid = t.tid
	JOIN sc ON c.cid = sc.cid
	JOIN student s ON s.sid = sc.sid 
HAVING
	t.tname = '张三';
```

> 查询没有学全所有课程的同学的信息

```sql
SELECT
	* 
FROM
	student s 
WHERE
	EXISTS (
	SELECT
		sc.sid,
		count( sc.score ) 
	FROM
		sc 
	GROUP BY
		sc.sid 
	HAVING
		count( sc.score ) >= ( SELECT count(*) FROM course ) 
	AND s.sid = sc.sid 
	);
```

> 查询至少有一门课与学号为" 01 "的同学所学相同的同学的信息

```sql
with temp as (select sc.cid from sc where sc.sid = '01')
select distinct s.* from sc join student s on sc.sid = s.sid and exists(select temp.cid from temp where temp.cid = sc.cid);
-- 或者
select * from student 
where student.sid in (
    select sc.sid from sc 
    where sc.cid in(
        select sc.cid from sc 
        where sc.sid = '01'
    )
);
```

<span style="color:red">**查询各科成绩前三名的记录**</span> 

```sql
select * from (select *,rank() over(partition by sc.cid order by sc.score desc) as graderank from sc) A
 where A.graderank <= 3;
```

<span style="color:red">**查有成绩的学生信息**</span>

```sql
SELECT
	s.sid,
	s.sname,
	count(*) AS 选课总数,
	sum( sc.score ) AS 总成绩,
	sum( CASE WHEN sc.cid = 01 THEN score ELSE NULL END ) AS score_01,
	sum( CASE WHEN sc.cid = 02 THEN score ELSE NULL END ) AS score_02,
	sum( CASE WHEN sc.cid = 03 THEN score ELSE NULL END ) AS score_03 
FROM
	sc
	JOIN student s ON sc.sid = s.sid 
GROUP BY
	s.sid;
```

> 查询没有学全所有课程的同学的信息

```sql
SELECT
	sc.*,
sum(case when sc.cid = 01 then score else null end)	as score_01,
sum(case when sc.cid = 02 then score else null end) as score_02,
sum(case when sc.cid = 03 then score else null end) as score_03
FROM
	student s
	JOIN sc ON s.sid = sc.sid 
GROUP BY
	s.sid 
HAVING
	count( sc.cid ) < ( SELECT count(*) FROM course );
```

> 查询和" 01 "号的同学学习的课程完全相同的其他同学的信息

```sql
select * from student s 
where s.sid in(
		select sc.sid from sc 
		where sc.cid in (select sc.cid from sc where sc.sid = '01') and sc.sid <> '01' 
		group by sc.sid 
		having count(sc.cid) >= (select count(*) from sc where sc.sid = '01')
)
```

<span style="color:red">**查询每个同学的平均成绩和有多少科不及格**</span>

```sql
SELECT
	sc.sid,
	avg( sc.score ) AS 平均成绩,
	sum( CASE WHEN sc.score < 60 THEN 1 ELSE 0 END ) AS 不及格科目数 
FROM
	sc 
GROUP BY
	sc.sid;
```

> 查询各科成绩最高分、最低分和平均分，以如下形式显示：课程 ID，课程 name，最高分，最低分，平均分，及格率，中等率，优良率，优秀率(及格为>=60，中等为：70-80，优良为：80-90，优秀为：>=90）。
> 要求输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序排列

```sql
SELECT
	c.cid AS 课程号,
	c.cname AS 课程名称,
	count(*) AS 选修人数,
	max( score ) AS 最高分,
	min( score ) AS 最低分,
	avg( score ) AS 平均分,
	sum( CASE WHEN score >= 60 THEN 1 ELSE 0 END )/ count(*) AS 及格率,
	sum( CASE WHEN score >= 70 AND score < 80 THEN 1 ELSE 0 END )/ count(*) AS 中等率,
	sum( CASE WHEN score >= 80 AND score < 90 THEN 1 ELSE 0 END )/ count(*) AS 优良率,
	sum( CASE WHEN score >= 90 THEN 1 ELSE 0 END )/ count(*) AS 优秀率 
FROM
	sc,
	course c 
WHERE
	c.cid = sc.cid 
GROUP BY
	c.cid 
ORDER BY
	count(*) DESC,
	c.cid ASC
```

> 查询每个同学的各科名次

```sql
select s.sid, s.sname, a.rank_score_01, b.rank_score_02, c.rank_score_03
from student s 
left join (select sc.sid, rank() over(partition by sc.cid order by sc.score) as rank_score_01 from sc where sc.cid = '01') a on s.sid = a.sid 
left join (select sc.sid, rank() over(partition by sc.cid order by sc.score) as rank_score_02 from sc where sc.cid = '02') b on s.sid = b.sid 
left join (select sc.sid, rank() over(partition by sc.cid order by sc.score) as rank_score_03 from sc where sc.cid = '03') c on c.sid = s.sid
```

17
