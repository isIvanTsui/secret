# 创建表

![](C:\Users\PC\Desktop\note\img\relation.png)

drop table if exists student;
drop table if exists teacher;
drop table if exists course;
drop table if exists sc;
CREATE TABLE student 
  ( 
     id    INT, 
     `name` nvarchar(32), 
     age  INT, 
     sex  nvarchar(8) 
  );
CREATE TABLE course 
  ( 
     id    INT, 
     `name` nvarchar(32), 
     t_id    INT 
  ); 
CREATE TABLE sc
  ( 
     s_id    INT, 
     c_id    INT, 
     score INT 
  );
CREATE TABLE teacher 
  ( 
     id    INT, 
     `name` nvarchar(16) 
  );

# 准备数据

insert into Student values
(1,'刘一',18,'男'),
(2,'钱二',19,'女'),
(3,'张三',17,'男'),
(4,'李四',18,'女'),
(5,'王五',17,'男'),
(6,'赵六',19,'女');

 insert into Teacher values
 (1,'叶平'),
 (2,'贺高'),
 (3,'杨艳'),
 (4,'周磊');

 insert into Course values
 (1,'语文',1),
 (2,'数学',2),
 (3,'英语',3),
 (4,'物理',4);

insert into SC values
(1,1,56),
(1,2,78),
(1,3,67),
(1,4,58),
(2,2,81),
(2,1,79),
(2,3,92),
(2,4,68),
(3,1,91),
(3,2,47),
(3,3,88),
(3,4,56),
(4,2,88),
(4,3,90),
(4,4,93),
(5,1,46),
(5,3,78),
(5,4,53),
(6,1,35),
(6,2,68),
(6,4,71);

# 开始练习

问题： 
1、查询“001”课程比“002”课程成绩高的所有学生的学号； 
**select a.s_id from (select sc.s_id, score from sc where sc.c_id = 1) a, (select sc.s_id, score from sc where sc.c_id = 2) b where a.score > b.score and a.s_id = b.s_id;** 
2、查询平均成绩大于60分的同学的学号和平均成绩； 
   **select sc.s_id, avg(sc.score) from sc group by sc.s_id having avg(sc.score) > 60;**
3、查询所有同学的学号、姓名、选课数、总成绩； 
  **select student.id, name, count(sc.c_id), sum(sc.score) from student inner join sc on student.id = sc.s_id group by student.id;**
4、查询姓“李”的老师的个数； 
  **select count(*) from teacher where `name` like '李%';** 
5、查询没学过“叶平”老师课的同学的学号、姓名； 
    **select * from student where id in(**
**select s_id from sc where c_id <> (select id from course where t_id = (select id from teacher where name = '叶平'))**
**);**
6、查询学过“001”并且也学过编号“002”课程的同学的学号、姓名； 
  select Student.S#,Student.Sname from Student,SC where Student.S#=SC.S# and SC.C#='001'and exists( Select * from SC as SC_2 where SC_2.S#=SC.S# and SC_2.C#='002'); 
7、查询学过“叶平”老师所教的所有课的同学的学号、姓名； 
  select S#,Sname 
  from Student 
  where S# in (select S# from SC ,Course ,Teacher where SC.C#=Course.C# and Teacher.T#=Course.T# and Teacher.Tname='叶平' group by S# having count(SC.C#)=(select count(C#) from Course,Teacher  where Teacher.T#=Course.T# and Tname='叶平')); 
8、查询课程编号“002”的成绩比课程编号“001”课程低的所有同学的学号、姓名； 
  Select S#,Sname from (select Student.S#,Student.Sname,score ,(select score from SC SC_2 where SC_2.S#=Student.S# and SC_2.C#='002') score2 
  from Student,SC where Student.S#=SC.S# and C#='001') S_2 where score2 <score; 
9、查询所有课程成绩小于60分的同学的学号、姓名； 
  select S#,Sname 
  from Student 
  where S# not in (select S.S# from Student AS S,SC where S.S#=SC.S# and score>60); 
10、查询没有学全所有课的同学的学号、姓名； 
    select Student.S#,Student.Sname 
    from Student,SC 
    where Student.S#=SC.S# group by  Student.S#,Student.Sname having count(C#) <(select count(C#) from Course); 
11、查询至少有一门课与学号为“1001”的同学所学相同的同学的学号和姓名； 
    select distinct S#,Sname from Student,SC where Student.S#=SC.S# and SC.C# in (select C# from SC where S#='1001'); 
12、查询至少学过学号为“001”同学所有一门课的其他同学学号和姓名； 
    select distinct SC.S#,Sname 
    from Student,SC 
    where Student.S#=SC.S# and C# in (select C# from SC where S#='001'); 
13、把“SC”表中“叶平”老师教的课的成绩都更改为此课程的平均成绩； 
    update SC set score=(select avg(SC_2.score) 
    from SC SC_2 
    where SC_2.C#=SC.C# ) from Course,Teacher where Course.C#=SC.C# and Course.T#=Teacher.T# and Teacher.Tname='叶平'); 
14、查询和“1002”号的同学学习的课程完全相同的其他同学学号和姓名； 
    select S# from SC where C# in (select C# from SC where S#='1002') 
    group by S# having count(*)=(select count(*) from SC where S#='1002'); 
15、删除学习“叶平”老师课的SC表记录； 
    Delect SC 
    from course ,Teacher  
    where Course.C#=SC.C# and Course.T#= Teacher.T# and Tname='叶平'; 
16、向SC表中插入一些记录，这些记录要求符合以下条件：没有上过编号“003”课程的同学学号、2、 
    号课的平均成绩； 
    Insert SC select S#,'002',(Select avg(score) 
    from SC where C#='002') from Student where S# not in (Select S# from SC where C#='002'); 
17、按平均成绩从高到低显示所有学生的“数据库”、“企业管理”、“英语”三门的课程成绩，按如下形式显示： 学生ID,,数据库,企业管理,英语,有效课程数,有效平均分 
    SELECT S# as 学生ID 
        ,(SELECT score FROM SC WHERE SC.S#=t.S# AND C#='004') AS 数据库 
        ,(SELECT score FROM SC WHERE SC.S#=t.S# AND C#='001') AS 企业管理 
        ,(SELECT score FROM SC WHERE SC.S#=t.S# AND C#='006') AS 英语 
        ,COUNT(*) AS 有效课程数, AVG(t.score) AS 平均成绩 
    FROM SC AS t 
    GROUP BY S# 
    ORDER BY avg(t.score)  
18、查询各科成绩最高和最低的分：以如下形式显示：课程ID，最高分，最低分 
    SELECT L.C# As 课程ID,L.score AS 最高分,R.score AS 最低分 
    FROM SC L ,SC AS R 
    WHERE L.C# = R.C# and 
        L.score = (SELECT MAX(IL.score) 
                      FROM SC AS IL,Student AS IM 
                      WHERE L.C# = IL.C# and IM.S#=IL.S# 
                      GROUP BY IL.C#) 
        AND 
        R.Score = (SELECT MIN(IR.score) 
                      FROM SC AS IR 
                      WHERE R.C# = IR.C# 
                  GROUP BY IR.C# 
                    ); 自己写的:select c# ,max(score)as 最高分 ,min(score) as 最低分 from dbo.sc  group by c#
19、按各科平均成绩从低到高和及格率的百分数从高到低顺序 
    SELECT t.C# AS 课程号,max(course.Cname)AS 课程名,isnull(AVG(score),0) AS 平均成绩 
        ,100 * SUM(CASE WHEN  isnull(score,0)>=60 THEN 1 ELSE 0 END)/COUNT(*) AS 及格百分数 
    FROM SC T,Course 
    where t.C#=course.C# 
    GROUP BY t.C# 
    ORDER BY 100 * SUM(CASE WHEN  isnull(score,0)>=60 THEN 1 ELSE 0 END)/COUNT(*) DESC 
20、查询如下课程平均成绩和及格率的百分数(用"1行"显示): 企业管理（001），马克思（002），OO&UML （003），数据库（004） 
    SELECT SUM(CASE WHEN C# ='001' THEN score ELSE 0 END)/SUM(CASE C# WHEN '001' THEN 1 ELSE 0 END) AS 企业管理平均分 
        ,100 * SUM(CASE WHEN C# = '001' AND score >= 60 THEN 1 ELSE 0 END)/SUM(CASE WHEN C# = '001' THEN 1 ELSE 0 END) AS 企业管理及格百分数 
        ,SUM(CASE WHEN C# = '002' THEN score ELSE 0 END)/SUM(CASE C# WHEN '002' THEN 1 ELSE 0 END) AS 马克思平均分 
        ,100 * SUM(CASE WHEN C# = '002' AND score >= 60 THEN 1 ELSE 0 END)/SUM(CASE WHEN C# = '002' THEN 1 ELSE 0 END) AS 马克思及格百分数 
        ,SUM(CASE WHEN C# = '003' THEN score ELSE 0 END)/SUM(CASE C# WHEN '003' THEN 1 ELSE 0 END) AS UML平均分 
        ,100 * SUM(CASE WHEN C# = '003' AND score >= 60 THEN 1 ELSE 0 END)/SUM(CASE WHEN C# = '003' THEN 1 ELSE 0 END) AS UML及格百分数 
        ,SUM(CASE WHEN C# = '004' THEN score ELSE 0 END)/SUM(CASE C# WHEN '004' THEN 1 ELSE 0 END) AS 数据库平均分 
        ,100 * SUM(CASE WHEN C# = '004' AND score >= 60 THEN 1 ELSE 0 END)/SUM(CASE WHEN C# = '004' THEN 1 ELSE 0 END) AS 数据库及格百分数 
  FROM SC