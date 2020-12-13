---
title: 01-sql刷题牛客
date: 2020-08-25 23:22:16
tags: [sql,数据分析,刷题]
categories: [数据分析,sql]
---

今天是刷题记录的第一天，加油啦，小孟冲冲冲！！！

1. 

```sql
查找最晚入职员工的所有信息，为了减轻入门难度，目前所有的数据里员工入职的日期都不是同一天(sqlite里面的注释为--,mysql为comment)
CREATE TABLE employees (

`emp_no int(11) NOT NULL, -- '员工编号'`

`birth_date date NOT NULL,`

`first_name varchar(14) NOT NULL,`

`last_name varchar(16) NOT NULL,`

`gender char(1) NOT NULL,`

`hire_date date NOT NULLPRIMARY KEY (emp_no));
```

自己的写法，考虑的不是那么全面，只能查出一条数据

```mysql
SELECT * FROM employees
ORDER BY hire_date DESC
LIMIt 1;
```

但是假如最后一天有多人入职，这种写法就hold不住了。

```mysql
SELECT * FROM employees
WHERE hire_date=(SELECT MAX(hire_date) FROM employees);
```

2.

```sql
查找入职员工时间排名倒数第三的员工所有信息，为了减轻入门难度，目前所有的数据里员工入职的日期都不是同一天
CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
```

这题主要的考点是查找倒数第三，这种关于顺序的问题可以用 LIMIT(A,B)来处理，表示从第A行取第B个数据，LIMIT(2,1)意思就是从第2个数据取第1个数据，也就是第三名。

```sql
SELECT * FROM employees
WHERE emp_no IN
(SELECT emp_no FROM employees ORDER BY hire_date DESC LIMIT 2,1);
```

3.

```sql
查找各个部门当前(dept_manager.to_date='9999-01-01')领导当前(salaries.to_date='9999-01-01')薪水详情以及其对应部门编号dept_no(注:请以salaries表为主表进行查询，输出结果以salaries.emp_no升序排序，并且请注意输出结果里面dept_no列是最后一列)

CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL, -- '员工编号',
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));

CREATE TABLE `dept_manager` (
`dept_no` char(4) NOT NULL, -- '部门编号'
`emp_no` int(11) NOT NULL, -- '员工编号'
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
```

常规的连接查询，注意输出就可以了

```sql
SELECT s.emp_no,s.salary,s.from_date,s.to_date,dm.dept_no
FROM salaries s INNER JOIN dept_manager dm
ON s.emp_no=dm.emp_no
WHERE dm.to_date='9999-01-01' AND s.to_date='9999-01-01'
ORDER BY s.emp_no ASC;
```

4.

```sql
查找所有已经分配部门的员工的last_name和first_name以及dept_no(请注意输出描述里各个列的前后顺序)
CREATE TABLE `dept_emp` (
`emp_no` int(11) NOT NULL,
`dept_no` char(4) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
```

两个表只有一个共同列，直接使用内连接。此外，因为dept_no 已经指定ＮＯＴ　ＮＵＬＬ了所以题目所有的员工必然都分配了部门。

```sql
SELECT e.last_name,e.first_name,de.dept_no
FROM employees e INNER JOIN dept_emp de
ON e.emp_no=de.emp_no
WHERE NOT (de.dept_no ISNULL);
```

５.

```sql
查找所有员工的last_name和first_name以及对应部门编号dept_no，也包括暂时没有分配具体部门的员工(请注意输出描述里各个列的前后顺序)
CREATE TABLE `dept_emp` (
`emp_no` int(11) NOT NULL,
`dept_no` char(4) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
```

这题说了包括暂时没有分配具体部门的员工,所有只要是员工就得查出来,所以对employees使用左连接啦,使用内连接的话,有的员工可能没有分配部门,那么部门表就不会有他,那再内连接就找不到他的数据了.

```sql
SELECT e.last_name,e.first_name,de.dept_no
FROM employees e LEFT JOIN dept_emp de
ON e.emp_no=de.emp_no;
```

6.

```sql
查找所有员工入职时候的薪水情况，给出emp_no以及salary， 并按照emp_no进行逆序(请注意，一个员工可能有多次涨薪的情况)
CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```

几天不写手感又生疏了呢！重点在于刚入职，入职时间就是hire_date呀

```sql
SELECT e.emp_no,s.salary FROM
employees e INNER JOIN salaries s
ON e.emp_no=s.emp_no
AND s.from_date=e.hire_date
ORDER BY e.emp_no DESC;
```

也可以用聚合函数，HAVING用来进行过滤

```sql
SELECT emp_no,salary FROM salaries
GROUP BY emp_no 
HAVING from_date=MIN(from_date)
ORDER BY emp_no DESC;
```

7.

```sql
查找薪水变动超过15次的员工号emp_no以及其对应的变动次数t
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```

这题考察的还是聚合函数，注意题目要求变动次数t

```sql
SELECT emp_no,COUNT(*) AS t FROM salaries
GROUP BY emp_no
HAVING t>15;
```

8.

```sql
找出所有员工当前(to_date='9999-01-01')具体的薪水salary情况，对于相同的薪水只显示一次,并按照逆序显示
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```

单纯的查salary表里有多少个salary，相同薪水只显示一次，用DISTINCT可以解决

```sql
SELECT DISTINCT(salary)
FROM salaries
WHERE to_date='9999-01-01'
ORDER BY salary DESC;
```

看讨论区说DISTINCT消耗资源较大，用GROUP BY 解决数据重复问题,注意GROUP BY 要放在WHERE 子句后面

```sql
SELECT salary
FROM salaries
WHERE to_date='9999-01-01'
GROUP BY salary
ORDER BY salary DESC;
```

9.

```sql
获取所有部门当前(dept_manager.to_date='9999-01-01')manager的当前(salaries.to_date='9999-01-01')薪水情况，给出dept_no, emp_no以及salary(请注意，同一个人可能有多条薪水情况记录)
CREATE TABLE `dept_manager` (
`dept_no` char(4) NOT NULL,
`emp_no` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```

注意时间，其他没什么

```sql
SELECT dm.dept_no,s.emp_no,s.salary
FROM dept_manager dm INNER JOIN salaries s 
ON dm.emp_no=s.emp_no
WHERE dm.to_date='9999-01-01' AND s.to_date='9999-01-01';
```

10.

```sql
获取所有非manager的员工emp_no
CREATE TABLE `dept_manager` (
`dept_no` char(4) NOT NULL,
`emp_no` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
```

这个题的特点是解法多，先来简单的

```sql
SELECT e.emp_no FROM employees e
WHERE e.emp_no NOT IN 
(SELECT dm.emp_no FROM dept_manager dm);
```

也可以利用连接查询获取dept_no为空的员工

```sql
SELECT e.emp_no
FROM employees e LEFT JOIN dept_manager dm
ON e.emp_no=dm.emp_no
WHERE dm.dept_no  IS NULL;
```

还有集合运算的解法

EXCEPT 集合差运算|UNION 集合并运算|INTERSECT 集合交运算

```sql
SELECT emp_no FROM employees
EXCEPT
SELECT emp_no FROM dept_manager;
```

11.

```sql
获取所有员工当前的(dept_manager.to_date='9999-01-01')manager，如果员工是manager的话不显示(也就是如果当前的manager是自己的话结果不显示)。输出结果第一列给出当前员工的emp_no,第二列给出其manager对应的emp_no。
CREATE TABLE `dept_emp` (
`emp_no` int(11) NOT NULL, -- '所有的员工编号'
`dept_no` char(4) NOT NULL, -- '部门编号'
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
CREATE TABLE `dept_manager` (
`dept_no` char(4) NOT NULL, -- '部门编号'
`emp_no` int(11) NOT NULL, -- '经理编号'
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
```

关于时间限制的坑，还有一点就是该关联的键是dept_no,是根据dept_no来找相同部门的,再有一点就是学到了`<>`不等于。`NOT IN`的写法也是可以的，但是要是查询的结果，比较两个单独变量相等用不等于呀。

```sql
SELECT de.emp_no AS emp_no,dm.emp_no AS manager_no
FROM dept_emp de INNER JOIN dept_manager dm 
ON de.dept_no=dm.dept_no
WHERE de.emp_no <> dm.emp_no AND dm.to_date='9999-01-01'
```

12.

```sql
获取所有部门中当前(dept_emp.to_date = '9999-01-01')员工当前(salaries.to_date='9999-01-01')薪水最高的相关信息，给出dept_no, emp_no以及其对应的salary，按照部门升序排列。
CREATE TABLE `dept_emp` (
`emp_no` int(11) NOT NULL,
`dept_no` char(4) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```

用group_by反而错了，看下别人的解释

`使用group by子句时，select子句中只能有聚合键、聚合函数、常数，就是除了常数和聚合其他键的函数外，不能有其他键，有其他键一定要使用聚合。emp_no并不符合这个要求。`

13.

```sql
从titles表获取按照title进行分组，每组个数大于等于2，给出title以及对应的数目t。
CREATE TABLE IF NOT EXISTS "titles" (
`emp_no` int(11) NOT NULL,
`title` varchar(50) NOT NULL,
`from_date` date NOT NULL,
`to_date` date DEFAULT NULL);
```

简单的GROUP BY~

```sql
SELECT title,COUNT(*) AS t
FROM titles
GROUP BY title;
```

14.

```从titles表获取按照title进行分组，每组个数大于等于2，给出title以及对应的数目t。
注意对于重复的emp_no进行忽略(即emp_no重复的title不计算，title对应的数目t不增加)。
CREATE TABLE IF NOT EXISTS `titles` (
`emp_no` int(11) NOT NULL,
`title` varchar(50) NOT NULL,
`from_date` date NOT NULL,
`to_date` date DEFAULT NULL);
```

重复的emp_no可以用DISTINCT区分，记住一个点，去重用DISTINCT

```sql
SELECT title,COUNT(DISTINCT emp_no) AS t
FROM titles
GROUP BY title
HAVING t>=2;
```

15.

```sql
查找employees表所有emp_no为奇数，且last_name不为Mary(注意大小写)的员工信息，并按照hire_date逆序排列(题目不能使用mod函数)
CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
```

被这个不能使用mod函数给唬住了，咱还可以使用`%`嘛

```sql
SELECT * FROM employees
WHERE last_name!='Mary' AND emp_no%2==1
ORDER BY hire_date DESC;
```

16.

```sql
统计出当前(titles.to_date='9999-01-01')各个title类型对应的员工当前(salaries.to_date='9999-01-01')薪水对应的平均工资。结果给出title以及平均工资avg。
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
CREATE TABLE IF NOT EXISTS "titles" (
`emp_no` int(11) NOT NULL,
`title` varchar(50) NOT NULL,
`from_date` date NOT NULL,
`to_date` date DEFAULT NULL);
```

GROUPY BY 要放在WHERE之后，WHERE子句中不能使用GROUPY BY

```sql
SELECT t.title,AVG(s.salary) AS avg
FROM salaries s INNER JOIN titles t
ON s.emp_no=t.emp_no
WHERE t.to_date='9999-01-01' AND s.to_date='9999-01-01'
GROUP BY t.title;
```

17.

```sql
获取当前（to_date='9999-01-01'）薪水第二多的员工的emp_no以及其对应的薪水salary
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```

先上一个错误做法

```sql
SELECT emp_no,salary
FROM salaries
WHERE to_date='9999-01-01'
ORDER by salary DESC
LIMIT 1,1
```

假如最大工资又两个是相同的，那么这样拿到的数据就不是第二大的了。以后碰到这种牵扯导顺序的题，要考虑能否把数据查全查准，用GROUP BY 查到值排名顺序的数据

```sql
SELECT emp_no,salary
FROM salaries
WHERE salary =(SELECT salary FROM salaries GROUP BY salary ORDER BY salary DESC LIMIT 1,1)
AND to_date='9999-01-01';
```

18.

```sql
查找当前薪水(to_date='9999-01-01')排名第二多的员工编号emp_no、薪水salary、last_name以及first_name，你可以不使用order by完成吗
CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```

排序题不让用ORDER BY,求第二大salary,小于最大值中的最大值就是第二大啦.

```sql
SELECT e.emp_no,MAX(s.salary),e.last_name,e.first_name
FROM employees e INNER JOIN salaries s ON e.emp_no=s.emp_no
WHERE s.salary<(SELECT MAX(salary) FROM salaries)
AND s.to_date='9999-01-01';
```

当然这种只能求解第二高，下面这个有点难，表内条件自连接

```sql
select e.emp_no,s.salary,e.last_name,e.first_name
from
employees e
join 
salaries s on e.emp_no=s.emp_no 
and  s.to_date='9999-01-01'
and s.salary = 
(
     select s1.salary
     from 
     salaries s1
     join
     salaries s2 on s1.salary<=s2.salary 
     and s1.to_date='9999-01-01' and s2.to_date='9999-01-01'
     group by s1.salary
     having count(distinct s2.salary)=2
 )
```

19.

```sql
查找所有员工的last_name和first_name以及对应的dept_name，也包括暂时没有分配部门的员工
CREATE TABLE `departments` (
`dept_no` char(4) NOT NULL,
`dept_name` varchar(40) NOT NULL,
PRIMARY KEY (`dept_no`));
CREATE TABLE `dept_emp` (
`emp_no` int(11) NOT NULL,
`dept_no` char(4) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
```

考察外联结

```sql
SELECT e.last_name,e.first_name,d.dept_name
FROM (employees e LEFT JOIN dept_emp de ON e.emp_no=de.emp_no)
LEFT JOIN departments d ON d.dept_no=de.dept_no;
```

也可以这样写

```sql
SELECT e.last_name,e.first_name,d.dept_name
FROM (employees e LEFT JOIN dept_emp de ON e.emp_no=de.emp_no) AS t
LEFT JOIN departments d ON d.dept_no=t.dept_no;
```

20.

```sql
查找员工编号emp_no为10001其自入职以来的薪水salary涨幅(总共涨了多少)growth(可能有多次涨薪，没有降薪)
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```

是计算涨幅呀，最简单的

```sql
SELECT MAX(salary)-MIN(salary) AS growth FROM salaries
WHERE emp_no=10001;
```

保险起见还是按照日期来选初始工资和近期工资

```sql
SELECT
(SELECT salary FROM salaries WHERE emp_no=10001 ORDER BY from_date DESC LIMIT 0,1)
-
(SELECT salary FROM salaries WHERE emp_no=10001 ORDER BY from_date ASC LIMIT 0,1)
AS growth;
```

21.

```sql
查找所有员工自入职以来的薪水涨幅情况，给出员工编号emp_no以及其对应的薪水涨幅growth，并按照growth进行升序
（注:可能有employees表和salaries表里存在记录的员工，有对应的员工编号和涨薪记录，但是已经离职了，离职的员工salaries表的最新的to_date!='9999-01-01'，这样的数据不显示在查找结果里面）
CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL, --  '入职时间'
PRIMARY KEY (`emp_no`));
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL, --  '一条薪水记录开始时间'
`to_date` date NOT NULL, --  '一条薪水记录结束时间'
PRIMARY KEY (`emp_no`,`from_date`));
```

这题的重点是找到现在的工资对应的时间和初到公司对应的时间，然后迎刃而解。注意hire_date是员工被雇佣的时间。

```sql
SELECT e.emp_no,a.salary-b.salary AS growth
FROM employees e INNER JOIN salaries a
ON e.emp_no=a.emp_no 
INNER JOIN salaries b
ON e.hire_date=b.from_date
WHERE a.to_date='9999-01-01'
ORDER BY growth ASC;
```

22.

```sql
统计各个部门的工资记录数，给出部门编码dept_no、部门名称dept_name以及部门在salaries表里面有多少条记录sum
CREATE TABLE `departments` (
`dept_no` char(4) NOT NULL,
`dept_name` varchar(40) NOT NULL,
PRIMARY KEY (`dept_no`));
CREATE TABLE `dept_emp` (
`emp_no` int(11) NOT NULL,
`dept_no` char(4) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```

统计各个部门，统计各个部门，明显是按某类计算肯定要GROUP BY 啊，你这脑子

```sql
SELECT d.dept_no,d.dept_name,COUNT(t.salary) AS sum
FROM (salaries s INNER JOIN dept_emp de
ON s.emp_no=de.emp_no) AS t
INNER JOIN departments d
ON t.dept_no=d.dept_no
GROUP BY d.dept_no;
```

23.❌

```sql
对所有员工的当前(to_date='9999-01-01')薪水按照salary进行按照1-N的排名，相同salary并列且按照emp_no升序排列
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```

这题的难点是如何在不同rank()函数的情况下进行排名，排序其实可以转化为当前值小于其他值的个数，又因为salary有并列行为，所以进行计数时要使用distinct，至于有的人提到使用groupby这点还没太看懂。

```sql
SELECT s1.emp_no,s1.salary ,
(SELECT COUNT(DISTINCT s2.salary) FROM salaries s2 WHERE s2.salary>=s1.salary
AND s2.to_date='9999-01-01') AS rank
FROM salaries s1
WHERE s1.to_date='9999-01-01'
ORDER BY rank,s1.emp_no ASC;
```

起始sql中有关排序的函数也可以解决上面的问题

```sql
select emp_no,salary, dense_rank() over (order by salary desc) as rank
from salaries
where to_date='9999-01-01'
order by rank asc,emp_no asc;
```

下面介绍几个函数的区别:

比如对分数进行排名：90、85、85、70

`ROW_NUMBER:1,2,3,4`

`RANK:1,2,2,4`

`DENSE_RANK:1、2、2、3`

`NTILE`函数是将有序分区中的行分发到指定数目的组中，各个组有编号，编号从1开始，就像我们说的’分区’一样 ，分为几个区，一个区会有多少个。

24.

```sql
获取所有非manager员工当前的薪水情况，给出dept_no、emp_no以及salary ，当前表示to_date='9999-01-01'
CREATE TABLE `dept_emp` (
`emp_no` int(11) NOT NULL,
`dept_no` char(4) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
CREATE TABLE `dept_manager` (
`dept_no` char(4) NOT NULL,
`emp_no` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```

这个没有很难，但是前面salary少写了一个字母就耽误了很长时间。这里不能`INNNER JOIN dept_manager`，否则就只生下来manager，写代码还是要有逻辑，不能瞎写。

```sql
SELECT de.dept_no,s.emp_no,s.salary
FROM salaries s INNER JOIN dept_emp de ON s.emp_no=de.emp_no  AND s.to_date='9999-01-01'
WHERE de.emp_no NOT IN (SELECT emp_no FROM dept_manager) AND de.to_date='9999-01-01';
```

25.❌

```sql
获取员工其当前的薪水比其manager当前薪水还高的相关信息，当前表示to_date='9999-01-01',
结果第一列给出员工的emp_no，
第二列给出其manager的manager_no，
第三列给出该员工当前的薪水emp_salary,
第四列给该员工对应的manager当前的薪水manager_salary
CREATE TABLE `dept_emp` (
`emp_no` int(11) NOT NULL,
`dept_no` char(4) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
CREATE TABLE `dept_manager` (
`dept_no` char(4) NOT NULL,
`emp_no` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```

看到这种题目要学会分解，dept_emp和salaries能组成员工薪水表，dept_manager和salaies能组成经理薪水表，两者都共有一个dept_no，那么只要两者dept_no相同，再比较两表的薪水大小就ok了。

```sql
SELECT t1.emp_no,t2.emp_no AS manager_no,t1.salary AS emp_salary,t2.salary AS manager_salary FROM
(SELECT s.emp_no,s.salary,de.dept_no FROM salaries s 
 INNER JOIN dept_emp de ON s.emp_no=de.emp_no AND s.to_date='9999-01-01' AND de.to_date='9999-01-01')
 t1 INNER JOIN 
 (SELECT s.emp_no,s.salary,dm.dept_no FROM salaries s
 INNER JOIN dept_manager dm ON s.emp_no=dm.emp_no AND s.to_date='9999-01-01' AND dm.to_date='9999-01-01')
 t2 ON t1.dept_no=t2.dept_no
WHERE t1.salary>t2.salary;
```

这里还有一个点就是JOIN后的ON的表达式不仅局限于等于还可以使用大于等于，和where语句起到的作用相似。或者用下面这种写法，将复杂查询分为两个简单查询，再组合。salary表可以出现两次，用where字句起到类似join的作用。

```sql
SELECT s1.emp_no,s2.emp_no AS manager_no,s1.salary AS emp_salary,s2.salary AS manager_salary
FROM dept_emp de,dept_manager dm,salaries s1,salaries s2
WHERE de.emp_no=s1.emp_no
AND dm.emp_no=s2.emp_no
AND de.dept_no=dm.dept_no
AND s1.salary>s2.salary
AND de.to_date='9999-01-01'
AND dm.to_date='9999-01-01'
AND s1.to_date='9999-01-01'
AND s2.to_date='9999-01-01'
```

26.

```sql
汇总各个部门当前员工的title类型的分配数目，即结果给出部门编号dept_no、dept_name、其部门下所有的当前(dept_emp.to_date = '9999-01-01')员工的当前(titles.to_date = '9999-01-01')title以及该类型title对应的数目count，结果按照dept_no升序排序
(注：因为员工可能有离职，所有dept_emp里面to_date不为'9999-01-01'就已经离职了，不计入统计，而且员工可能有晋升，所以如果titles.to_date 不为 '9999-01-01'，那么这个可能是员工之前的职位信息，也不计入统计)
CREATE TABLE `departments` (
`dept_no` char(4) NOT NULL,
`dept_name` varchar(40) NOT NULL,
PRIMARY KEY (`dept_no`));
CREATE TABLE `dept_emp` (
`emp_no` int(11) NOT NULL,
`dept_no` char(4) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
CREATE TABLE IF NOT EXISTS `titles` (
`emp_no` int(11) NOT NULL,
`title` varchar(50) NOT NULL,
`from_date` date NOT NULL,
`to_date` date DEFAULT NULL);
输入描述:
```

自己竟然写出来了

```sql
SELECT de.dept_no,d.dept_name,t.title,COUNT(de.emp_no) as count
FROM dept_emp de INNER JOIN titles t ON de.emp_no=t.emp_no INNER JOIN departments d ON de.dept_no=d.dept_no
WHERE de.to_date='9999-01-01' AND t.to_date='9999-01-01'
GROUP BY de.dept_no,t.title;
```

这道题也不难，无脑连接后加上条件再groupby就ok了，倒是groupby的用法自己又快忘光了呢！

27. ❌

```sql
给出每个员工每年薪水涨幅超过5000的员工编号emp_no、薪水变更开始日期from_date以及薪水涨幅值salary_growth，并按照salary_growth逆序排列。
提示：在sqlite中获取datetime时间对应的年份函数为strftime('%Y', to_date)
(数据保证每个员工的每条薪水记录to_date-from_date=1年，而且同一员工的下一条薪水记录from_data=上一条薪水记录的to_data)

CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
如：插入
INSERT INTO salaries VALUES(10001,52117,'1986-06-26','1987-06-26');
INSERT INTO salaries VALUES(10001,62102,'1987-06-26','1988-06-25');
INSERT INTO salaries VALUES(10002,72527,'1996-08-03','1997-08-03');
INSERT INTO salaries VALUES(10002,72527,'1997-08-03','1998-08-03');
INSERT INTO salaries VALUES(10002,72527,'1998-08-03','1999-08-03');
INSERT INTO salaries VALUES(10003,43616,'1996-12-02','1997-12-02');
INSERT INTO salaries VALUES(10003,43466,'1997-12-02','1998-12-02');
```

先放上自己的写法，对于同一个人相邻两条记录的比较我们可以直接让同样的表按某种条件内连接就ok

```sql
SELECT 
s1.emp_no,s2.from_date,(s2.salary-s1.salary) AS salary_growth
FROM salaries s1  INNER JOIN salaries s2
ON s1.emp_no=s2.emp_no AND s1.to_date=s2.from_date
WHERE salary_growth>5000
ORDER BY salary_growth DESC;
```

这道题评论也给我看晕了，主要是存在多次涨薪的问题，比如说在一年内多次涨薪的问题。给我整晕了。

```sql
SELECT s2.emp_no, s2.from_date, (s2.salary - s1.salary) AS salary_growth
FROM salaries AS s1, salaries AS s2
WHERE s1.emp_no = s2.emp_no 
AND salary_growth > 5000
AND (strftime("%Y",s2.to_date) - strftime("%Y",s1.to_date) = 1 
     OR strftime("%Y",s2.from_date) - strftime("%Y",s1.from_date) = 1 )
ORDER BY salary_growth DESC
```

写上去就算看一看strftime的用法吧。
28.❌

```sql
CREATE TABLE IF NOT EXISTS film (
film_id smallint(5)  NOT NULL DEFAULT '0',
title varchar(255) NOT NULL,
description text,
PRIMARY KEY (film_id));
CREATE TABLE category  (
category_id  tinyint(3)  NOT NULL ,
name  varchar(25) NOT NULL, `last_update` timestamp,
PRIMARY KEY ( category_id ));
CREATE TABLE film_category  (
film_id  smallint(5)  NOT NULL,
category_id  tinyint(3)  NOT NULL, `last_update` timestamp);
查找描述信息(film.description)中包含robot的电影对应的分类名称(category.name)以及电影数目(count(film.film_id))，而且还需要该分类包含电影总数量(count(film_category.category_id))>=5部
```

第二个条件有点无语，不是太理解,正确答案的理解是这两个条件是并列关系。

```sql
select name,count(name)
from film,film_category,category
where film.description like '%robot%' and film.film_id= film_category.film_id and film_category.category_id= category.category_id
and category.category_id in (select category_id from film_category group by category_id having count(film_id)>=5)
```

自己的想法是先满足第一个条件后，再分组，不过通不过测试

```sql
SELECT c.name,COUNT(f.film_id) AS film_num
FROM film f INNER JOIN film_category fc ON f.film_id=fc.film_id INNER JOIN category c ON c.category_id=fc.category_id
WHERE f.description like '%robot%'
GROUP BY c.category_id
HAVING film_num>=5;
```

29.

```sql
CREATE TABLE IF NOT EXISTS film (
film_id smallint(5)  NOT NULL DEFAULT '0',
title varchar(255) NOT NULL,
description text,
PRIMARY KEY (film_id));

CREATE TABLE category  (
category_id  tinyint(3)  NOT NULL ,
name  varchar(25) NOT NULL, `last_update` timestamp,
PRIMARY KEY ( category_id ));

CREATE TABLE film_category  (
film_id  smallint(5)  NOT NULL,
category_id  tinyint(3)  NOT NULL, `last_update` timestamp);
使用join查询方式找出没有分类的电影id以及名称
```

看清题目，判断是否为NULL，用IS

```sql
SELECT f.film_id,f.title
FROM film f LEFT JOIN film_category fc ON f.film_id=fc.film_id 
WHERE fc.category_id IS NULL;
```

30.

```sql
CREATE TABLE IF NOT EXISTS film (
film_id smallint(5)  NOT NULL DEFAULT '0',
title varchar(255) NOT NULL,
description text,
PRIMARY KEY (film_id));

CREATE TABLE category  (
category_id  tinyint(3)  NOT NULL ,
name  varchar(25) NOT NULL, `last_update` timestamp,
PRIMARY KEY ( category_id ));

CREATE TABLE film_category  (
film_id  smallint(5)  NOT NULL,
category_id  tinyint(3)  NOT NULL, `last_update` timestamp);
你能使用子查询的方式找出属于Action分类的所有电影对应的title,description吗
```

注意action的条件

```sql
SELECT f.title,f.description 
FROM film f INNER JOIN film_category fc ON f.film_id=fc.film_id INNER JOIN category c ON fc.category_id=c.category_id
WHERE c.name='Action';
```

题目还提到要用子查询的方式

```sql
SELECT title,description FROM film where film_id IN
(SELECT film_id from film_category WHERE category_id in (SELECT category_id from category WHERE name='Action'))
```



31.

```sql
将employees表的所有员工的last_name和first_name拼接起来作为Name，中间以一个空格区分
(注：sqllite,字符串拼接为 || 符号，不支持concat函数，mysql支持concat函数)
CREATE TABLE `employees` ( `emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
```

```sql
SELECT last_name || ' ' || first_name
FROM employees;
```

再看看CONCAT用法

```sql
CONCAT方法：
select CONCAT(CONCAT(last_name," "),first_name) as name  from employees
或者
select CONCAT(last_name," "，first_name) as name  from employees
```

32.

创建一个actor表，包含如下列信息

| 列表        | 类型        | 是否为NULL | 含义   |
| :---------- | :---------- | :--------- | :----- |
| actor_id    | smallint(5) | not null   | 主键id |
| first_name  | varchar(45) | not null   | 名字   |
| last_name   | varchar(45) | not null   | 姓氏   |
| last_update | date        | not null   | 日期   |

```sql
create table if not exists actor(
    actor_id smallint(5)  not null ,
    first_name varchar(45)  not null,
    last_name varchar(45)  not null,
    last_update timestamp  not null default (datetime('now','localtime')),
    primary key(actor_id)
)
```

33.

题目已经先执行了如下语句:

```sql
drop table if exists actor;
CREATE TABLE actor (
   actor_id  smallint(5)  NOT NULL PRIMARY KEY,
   first_name  varchar(45) NOT NULL,
   last_name  varchar(45) NOT NULL,
   last_update  DATETIME NOT NULL)
```

请你对于表actor批量插入如下数据(不能有2条insert语句哦!)

| actor_id | first_name | last_name | last_update         |
| :------- | :--------- | :-------- | :------------------ |
| 1        | PENELOPE   | GUINESS   | 2006-02-15 12:34:33 |
| 2        | NICK       | WAHLBERG  | 2006-02-15 12:34:33 |

```sql
INSERT INTO actor VALUES
(1,'PENELOPE','GUINESS','2006-02-15 12:34:33'),
(2,'NICK','WAHLBERG','2006-02-15 12:34:33')
```

34.

题目已经先执行了如下语句:

```sql
drop table if exists actor;
CREATE TABLE actor (
   actor_id  smallint(5)  NOT NULL PRIMARY KEY,
   first_name  varchar(45) NOT NULL,
   last_name  varchar(45) NOT NULL,
   last_update  DATETIME NOT NULL);
insert into actor values ('3', 'WD', 'GUINESS', '2006-02-15 12:34:33');
```

对于表actor插入如下数据,如果数据已经存在，请忽略(不支持使用replace操作)

| actor_id | first_name | last_name | last_update           |
| :------- | :--------- | :-------- | :-------------------- |
| '3'      | 'ED'       | 'CHASE'   | '2006-02-15 12:34:33' |

35.

题目已经先执行了如下语句:

```sq
drop table if exists actor;
CREATE TABLE actor (
   actor_id  smallint(5)  NOT NULL PRIMARY KEY,
   first_name  varchar(45) NOT NULL,
   last_name  varchar(45) NOT NULL,
   last_update  DATETIME NOT NULL);
insert into actor values ('3', 'WD', 'GUINESS', '2006-02-15 12:34:33');
```

对于表actor插入如下数据,如果数据已经存在，请忽略(不支持使用replace操作)

| actor_id | first_name | last_name | last_update           |
| :------- | :--------- | :-------- | :-------------------- |
| '3'      | 'ED'       | 'CHASE'   | '2006-02-15 12:34:33' |

sqlite

```sql
insert or ignore into actor 
values(3,'ED','CHASE','2006-02-15 12:34:33');
```

mysql

```sql
insert IGNORE into actor 
values(3,'ED','CHASE','2006-02-15 12:34:33');
```

36.

对于如下表actor，其对应的数据为:

| actor_id | first_name | last_name | last_update         |
| :------- | :--------- | :-------- | :------------------ |
| 1        | PENELOPE   | GUINESS   | 2006-02-15 12:34:33 |
| 2        | NICK       | WAHLBERG  | 2006-02-15 12:34:33 |

请你创建一个actor_name表，并且将actor表中的所有first_name以及last_name导入该表.

actor_name表结构如下：

| 列表       | 类型        | 是否为NULL | 含义 |
| :--------- | :---------- | :--------- | :--- |
| first_name | varchar(45) | not null   | 名字 |
| last_name  | varchar(45) | not null   | 姓氏 |

一种写法是先建表，然后插入

```sql
CREATE TABLE actor_name(
first_name varchar(45) not null,
last_name varchar(45) not null);
INSERT INTO actor_name SELECT first_name,last_name FROM actor;
```

另一种是用查询到的数据建表

sqlite

```sql
CREATE TABLE actor_name AS
SELECT first_name,last_name FROM actor
```

mysql

```sql
CREATE TABLE actor_name
SELECT first_name,last_name FROM actor;
```

37.

针对如下表actor结构创建索引：

(注:在 SQLite 中,除了重命名表和在已有的表中添加列,ALTER TABLE 命令不支持其他操作，

mysql支持ALTER TABLE创建索引)

```sql
CREATE TABLE actor  (
   actor_id  smallint(5)  NOT NULL PRIMARY KEY,
   first_name  varchar(45) NOT NULL,
   last_name  varchar(45) NOT NULL,
   last_update  datetime NOT NULL);
```

对first_name创建唯一索引uniq_idx_firstname，对last_name创建普通索引idx_lastname

```sql
CREATE UNIQUE INDEX uniq_idx_firstname ON actor(first_name);
CREATE INDEX idx_lastname ON actor(last_name);
```

mysql

```sql
ALTER TABLE actor ADD UNIQUE INDEX  uniq_idx_firstname(first_name);
ALTER TABLE actor ADD INDEX idx_lastname(last_name);
```

38.

针对actor表创建视图actor_name_view，只包含first_name以及last_name两列，并对这两列重新命名，first_name为first_name_v，last_name修改为last_name_v：

```sql
CREATE TABLE  actor  (
   actor_id  smallint(5)  NOT NULL PRIMARY KEY,
   first_name  varchar(45) NOT NULL,
   last_name  varchar(45) NOT NULL,
   last_update datetime NOT NULL);
```

```sql
CREATE VIEW actor_name_view AS
SELECT first_name first_name_v,last_name last_name_v
FROM actor;
```

```sql
CREATE VIEW actor_name_view (fist_name_v, last_name_v) AS
SELECT first_name, last_name FROM actor 
```

39.

```sql
针对salaries表emp_no字段创建索引idx_emp_no，查询emp_no为10005, 使用强制索引。
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
create index idx_emp_no on salaries(emp_no);
```

sqlite

```sql
SELECT * FROM salaries INDEXED BY idx_emp_no WHERE emp_no=10005;
```

mysql

```sql
SELECT * FROM salaries FROCE INDEX BY idx_emp_no WHERE emp_no=10005;
```

40.

存在actor表，包含如下列信息：

```sql
CREATE TABLE  actor  (
   actor_id  smallint(5)  NOT NULL PRIMARY KEY,
   first_name  varchar(45) NOT NULL,
   last_name  varchar(45) NOT NULL,
   last_update  datetime NOT NULL);
```

现在在last_update后面新增加一列名字为create_date, 类型为datetime, NOT NULL，默认值为'2020-10-01 00:00:00'

```sql
ALTER TABLE actor
ADD COLUMN create_date datetime not null default  '2020-10-01 00:00:00' AFTER last_update; 
```

41.

```sql
构造一个触发器audit_log，在向employees_test表中插入一条数据的时候，触发插入相关的数据到audit中。
CREATE TABLE employees_test(
ID INT PRIMARY KEY NOT NULL,
NAME TEXT NOT NULL,
AGE INT NOT NULL,
ADDRESS CHAR(50),
SALARY REAL
)

CREATE TABLE audit(
EMP_no INT NOT NULL,
NAME TEXT NOT NULL
);
```

begin和end之间的语句要用分号结束

```sql
CREATE TRIGGER audit_log AFTER INSERT ON employees_test
begin 
INSERT INTO audit(EMP_no,NAME) VALUES
(new.ID,new.NAME);
end;
```

42.

```sql
删除emp_no重复的记录，只保留最小的id对应的记录。
CREATE TABLE IF NOT EXISTS titles_test (
id int(11) not null primary key,
emp_no int(11) NOT NULL,
title varchar(50) NOT NULL,
from_date date NOT NULL,
to_date date DEFAULT NULL);

insert into titles_test values ('1', '10001', 'Senior Engineer', '1986-06-26', '9999-01-01'),
('2', '10002', 'Staff', '1996-08-03', '9999-01-01'),
('3', '10003', 'Senior Engineer', '1995-12-03', '9999-01-01'),
('4', '10004', 'Senior Engineer', '1995-12-03', '9999-01-01'),
('5', '10001', 'Senior Engineer', '1986-06-26', '9999-01-01'),
('6', '10002', 'Staff', '1996-08-03', '9999-01-01'),
('7', '10003', 'Senior Engineer', '1995-12-03', '9999-01-01');
```

对groupby又产生了大大的疑惑，不是说select后面只能接groupby的相关字段吗

```sql
DELETE FROM titles_test
WHERE id NOT IN  (
        SELECT MIN(id) FROM titles_test GROUP BY emp_no)
```

两两比较找出最大值或者最小值

```sql
delete from titles_test where id in(
    select a.id from titles_test a,titles_test b where a.emp_no=b.emp_no and a.id>b.id)
```

43.

```sql
将所有to_date为9999-01-01的全部更新为NULL,且 from_date更新为2001-01-01。
CREATE TABLE IF NOT EXISTS titles_test (
id int(11) not null primary key,
emp_no int(11) NOT NULL,
title varchar(50) NOT NULL,
from_date date NOT NULL,
to_date date DEFAULT NULL);

insert into titles_test values ('1', '10001', 'Senior Engineer', '1986-06-26', '9999-01-01'),
('2', '10002', 'Staff', '1996-08-03', '9999-01-01'),
('3', '10003', 'Senior Engineer', '1995-12-03', '9999-01-01'),
('4', '10004', 'Senior Engineer', '1995-12-03', '9999-01-01'),
('5', '10001', 'Senior Engineer', '1986-06-26', '9999-01-01'),
('6', '10002', 'Staff', '1996-08-03', '9999-01-01'),
('7', '10003', 'Senior Engineer', '1995-12-03', '9999-01-01');

更新后的值:
titles_test 表的值：
```

| id   | emp_no | title           | from_date  | to_date |
| ---- | ------ | --------------- | ---------- | ------- |
| 1    | 10001  | Senior Engineer | 2001-01-01 | NULL    |
| 2    | 10002  | Staff           | 2001-01-01 | NULL    |
| 3    | 10003  | Senior Engineer | 2001-01-01 | NULL    |
| 4    | 10004  | Senior Engineer | 2001-01-01 | NULL    |
| 5    | 10001  | Senior Engineer | 2001-01-01 | NULL    |
| 6    | 10002  | Staff           | 2001-01-01 | NULL    |
| 7    | 10003  | Senior Engineer | 2001-01-01 | NULL    |

UPDATE语句更新多列用逗号隔开哦~不是用AND

```sql
UPDATE titles_test
SET to_date=NULL,from_date='2001-01-01' 
WHERE to_date='9999-01-01';
```

44.

将id=5以及emp_no=10001的行数据替换成id=5以及emp_no=10005,其他数据保持不变，**使用replace实现，直接使用update会报错了**。

```sql
CREATE TABLE titles_test (
   id int(11) not null primary key,
   emp_no  int(11) NOT NULL,
   title  varchar(50) NOT NULL,
   from_date  date NOT NULL,
   to_date  date DEFAULT NULL);

insert into titles_test values
('1', '10001', 'Senior Engineer', '1986-06-26', '9999-01-01'),
('2', '10002', 'Staff', '1996-08-03', '9999-01-01'),
('3', '10003', 'Senior Engineer', '1995-12-03', '9999-01-01'),
('4', '10004', 'Senior Engineer', '1995-12-03', '9999-01-01'),
('5', '10001', 'Senior Engineer', '1986-06-26', '9999-01-01'),
('6', '10002', 'Staff', '1996-08-03', '9999-01-01'),
('7', '10003', 'Senior Engineer', '1995-12-03', '9999-01-01');
```

mark一下replace的用法

```sql
UPDATE titles_test SET emp_no=replace(emp_no,10001,10005) 
WHERE id=5;
```

全字段更新替换,会插入一条新记录。并且要将所有字段的值写出，否则将置为空。

```sql
REPLACE INTO titles_test VALUES (5, 10005, 'Senior Engineer', '1986-06-26', '9999-01-01')
```

45.

将titles_test表名修改为titles_2017。

```sql
CREATE TABLE IF NOT EXISTS titles_test (
id int(11) not null primary key,
emp_no int(11) NOT NULL,
title varchar(50) NOT NULL,
from_date date NOT NULL,
to_date date DEFAULT NULL);

insert into titles_test values ('1', '10001', 'Senior Engineer', '1986-06-26', '9999-01-01'),
('2', '10002', 'Staff', '1996-08-03', '9999-01-01'),
('3', '10003', 'Senior Engineer', '1995-12-03', '9999-01-01'),
('4', '10004', 'Senior Engineer', '1995-12-03', '9999-01-01'),
('5', '10001', 'Senior Engineer', '1986-06-26', '9999-01-01'),
('6', '10002', 'Staff', '1996-08-03', '9999-01-01'),
('7', '10003', 'Senior Engineer', '1995-12-03', '9999-01-01');
```

sqlite

```sql
ALTER TABLE titles_test RENAME TO titles_2017;
```

mysql

```sql
alter table titles_test rename titles_2017
```

46.

在audit表上创建外键约束，其emp_no对应employees_test表的主键id。

(以下2个表已经创建了)

```sql
CREATE TABLE employees_test(
ID INT PRIMARY KEY NOT NULL,
NAME TEXT NOT NULL,
AGE INT NOT NULL,
ADDRESS CHAR(50),
SALARY REAL
);

CREATE TABLE audit(
EMP_no INT NOT NULL,
create_date datetime NOT NULL
);
```

先看mysql的用法

```sql
ALTER TABLE audit
ADD FOREIGN KEY (emp_no)
REFERENCES employees_test (id);
```

再看sqlite,注意外键格式

```sql
DROP TABLE audit;
CREATE TABLE audit(
emp_no INT NOT NULL,
create_date datetime NOT NULL,
FOREIGN KEY(emp_no) REFERENCES employees_test(id))
```

47.

```sql
请你写出更新语句，将所有获取奖金的员工当前的(salaries.to_date='9999-01-01')薪水增加10%。(emp_bonus里面的emp_no都是当前获奖的所有员工)
create table emp_bonus(
emp_no int not null,
btype smallint not null);
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL, PRIMARY KEY (`emp_no`,`from_date`));
如：
INSERT INTO emp_bonus VALUES (10001,1);
INSERT INTO salaries VALUES(10001,85097,'2001-06-22','2002-06-22');
INSERT INTO salaries VALUES(10001,88958,'2002-06-22','9999-01-01');
```

```sql
UPDATE salaries
SET salary=salary*1.1
WHERE emp_no IN
(SELECT emp_no FROM emp_bonus) AND to_date='9999-01-01';
```

48.

```sql
将employees表中的所有员工的last_name和first_name通过(')连接起来。(sqlite不支持concat，请用||实现，mysql支持concat)
CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
输出格式:
```

```sql
SELECT last_name || "'" || first_name FROM employees;
```

```sql
SELECT CONCAT(last_name, '''', first_name) as name FROM employees;
```

49.

查找字符串'10,A,B' 中逗号','出现的次数cnt。

技巧题哈，学习了LENGTH 和REPLACE的用法

```sql
SELECT LENGTH('10,A,B' )-LENGTH(REPLACE('10,A,B',',','')) AS cnt
```

50.

```sql
获取Employees中的first_name，查询按照first_name最后两个字母，按照升序进行排列
CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
```

```sql
SELECT first_name FROM 
(SELECT first_name,SUBSTR(first_name,-2) AS first_name_2 FROM employees)
ORDER BY first_name_2 ASC;
```

```sql
SELECT first_name FROM employees 
ORDER BY substr(first_name,-2,2) asc;
```

```sql
SELECT first_name from employees 
order by right(first_name, 2);
```

51.

```sql
按照dept_no进行汇总，属于同一个部门的emp_no按照逗号进行连接，结果给出dept_no以及连接出的结果employees
CREATE TABLE `dept_emp` (
`emp_no` int(11) NOT NULL,
`dept_no` char(4) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
输出格式:
```

记录一下GROUP_CONCAT的用法

sqlite

```sql
SELECT dept_no,GROUP_CONCAT(emp_no) AS employees
FROM dept_emp
GROUP BY dept_no;
```

mysql

```sql
SELECT dept_no, group_concat(DISTINCT emp_no ORDER BY emp_no ASC SEPARATOR ',') AS employees FROM dept_emp GROUP BY dept_no;
```

52.

```SQL
查找排除最大、最小salary之后的当前(to_date = '9999-01-01' )员工的平均工资avg_salary。
CREATE TABLE `salaries` ( `emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```

关于时间的限制

```sql
SELECT AVG(salary) AS avg_salary FROM salaries 
WHERE to_date = '9999-01-01' 
AND salary NOT IN (SELECT MAX(salary) FROM salaries WHERE to_date = '9999-01-01')
AND salary NOT IN (SELECT MIN(salary) FROM salaries WHERE to_date = '9999-01-01')
```

```sql
SELECT
    AVG(salary)
FROM
    salaries
WHERE
    salary<>(SELECT MAX(salary) FROM salaries WHERE to_date='9999-01-01') 
AND
    salary<>(SELECT MIN(salary) FROM salaries WHERE to_date='9999-01-01') 
AND
    to_date='9999-01-01'
```

53.

```sql
分页查询employees表，每5行一页，返回第2页的数据
CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
```

LIMIT 后的数字代表返回几条记录，OFFSET 后的数字代表从第几条记录开始返回

```sql
SELECT * FROM employees
LIMIT 5 OFFSET 5;
```

在 LIMIT X,Y 中，Y代表返回几条记录，X代表从第几条记录开始返回（第一条记录序号为0），切勿记反。

54.

```sql
获取所有员工的emp_no、部门编号dept_no以及对应的bonus类型btype和received，没有分配奖金的员工不显示对应的bonus类型btype和received
CREATE TABLE `dept_emp` (
`emp_no` int(11) NOT NULL,
`dept_no` char(4) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));

CREATE TABLE `emp_bonus`(
emp_no int(11) NOT NULL,
received datetime NOT NULL,
btype smallint(5) NOT NULL);

CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
```

```sql
SELECT e.emp_no,de.dept_no,eb.btype,eb.received
FROM employees e INNER JOIN dept_emp de ON e.emp_no=de.emp_no LEFT JOIN emp_bonus eb ON e.emp_no=eb.emp_no
```

55.

```sql
使用含有关键字exists查找未分配具体部门的员工的所有信息。
CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
CREATE TABLE `dept_emp` (
`emp_no` int(11) NOT NULL,
`dept_no` char(4) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
```

这个EXISTS当作条件判断得理解理解

IN是先执行子查询，得到一个结果集，将结果集代入外层谓词条件执行主查询，子查询只需要执行一次
EXISTS是先从主查询中取得一条数据，再代入到子查询中，执行一次子查询，判断子查询是否能返回结果，主查询有多少条数据，子查询就要执行多少次,EXISTS中要添加判断条件？？？

```sql
SELECT * FROM employees
WHERE  NOT EXISTS 
(SELECT emp_no FROM dept_emp WHERE employees.emp_no=dept_emp.emp_no)
```

56.

```sql
获取有奖金的员工相关信息。
CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
CREATE TABLE `dept_emp` (
`emp_no` int(11) NOT NULL,
`dept_no` char(4) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
create table emp_bonus(
emp_no int not null,
received datetime not null,
btype smallint not null);
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL, PRIMARY KEY (`emp_no`,`from_date`));
给出emp_no、first_name、last_name、奖金类型btype、对应的当前薪水情况salary以及奖金金额bonus。 bonus类型btype为1其奖金为薪水salary的10%，btype为2其奖金为薪水的20%，其他类型均为薪水的30%。 当前薪水表示to_date='9999-01-01'
```

```sql
SELECT e.emp_no,e.first_name,e.last_name,eb.btype,s.salary,s.salary*eb.btype*0.1 as bonus
FROM employees e 
INNER JOIN emp_bonus eb ON e.emp_no=eb.emp_no 
INNER JOIN salaries s ON e.emp_no=s.emp_no 
WHERE s.to_date='9999-01-01'
```

看看CASE WHEN THEN END的用法

```sql
SELECT e.emp_no,e.first_name,e.last_name,eb.btype,s.salary,
(CASE eb.btype
WHEN 1 THEN s.salary*0.1
WHEN 2 THEN s.salary*0.2
WHEN 3 THEN s.salary*0.3
END) AS bonus
FROM employees e 
INNER JOIN emp_bonus eb ON e.emp_no=eb.emp_no 
INNER JOIN salaries s ON e.emp_no=s.emp_no 
WHERE s.to_date='9999-01-01'
```

57.

```sql
按照salary的累计和running_total，其中running_total为前N个当前( to_date = '9999-01-01')员工的salary累计和，其他以此类推。 具体结果如下Demo展示。。
CREATE TABLE `salaries` ( `emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
输出格式:
```

让我们来学习一种新的求和方法，两表相同按某列拍好序后，比较大小即可求和。

```sql
SELECT emp_no,salary,
(select sum(s1.salary) FROM salaries s1 WHERE s1.emp_no<=s2.emp_no AND s1.to_date='9999-01-01') AS running_total
FROM salaries s2 WHERE to_date='9999-01-01' ORDER BY s2.emp_no
```

```sql
SELECT emp_no,salary,
SUM(salary) OVER(ORDER BY emp_no) AS running_total
FROM salaries
WHERE to_date= '9999-01-01';
```

58.❌

```sql
对于employees表中，输出first_name排名(按first_name升序排序)为奇数的first_name
CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
```

对于这个COUNT的用法还是存疑(聚合或是满足某种条件之后？？？)，可以用于计数，两两比较之间的计数，用于排序

```sql
SELECT first_name FROM employees e1
WHERE 
(SELECT COUNT(*) FROM employees e2 WHERE e1.first_name>=e2.first_name)%2=1;
```

用row_number() over 进行排序

```sql
select tt.first_name
from
(select first_name,
row_number() over (order by first_name) as row_idx
from employees) as tt
where tt.row_idx%2=1;
```

```sql
SELECT
    e.first_name
FROM employees e JOIN
(
    SELECT 
        first_name
        , ROW_NUMBER() OVER(ORDER BY first_name ASC) AS  r_num
    FROM employees
) AS t 
ON e.first_name = t.first_name
WHERE t.r_num % 2 = 1;
```

59.

在牛客刷题的小伙伴们都有着牛客积分，积分(grade)表简化可以如下:

![](https://uploadfiles.nowcoder.com/images/20200813/557336_1597308877503_0244CF2104BC29F1AAC07582264ADC57)

```sql
SELECT number FROM grade GROUP BY number HAVING COUNT(id)>=3
```

这个思路也很牛逼:要三次以上的积分，那么肯定要查找3个id不同但是积分相同的情况

```sql
SELECT  DISTINCT g1.number AS times
FROM
    grade g1,
    grade g2,
    grade g3
WHERE
    g1.id != g2.id
    AND g2.id != g3.id 
    AND g1.id !=g3.id
    AND g1.number = g2.number
    AND g2.number = g3.number
```

60.❌

在牛客刷题有一个通过题目个数的(passing_number)表，id是主键，简化如下:

![](https://uploadfiles.nowcoder.com/images/20200813/557336_1597308916086_6B7E692D15E9D27D855F76C56E00D52A)

第1行表示id为1的用户通过了4个题目;

.....

第6行表示id为6的用户通过了4个题目;



请你根据上表，输出通过的题目的排名，通过题目个数相同的，排名相同，此时按照id升序排列，数据如下:

![](https://uploadfiles.nowcoder.com/images/20201105/557336_1604558013211_15EB7DEE744C7810B57ED3EF3D65D7FE)

```sql
SELECT id,number,DENSE_RANK() OVER (ORDER BY number DESC) AS t_rank
FROM passing_number ORDER BY t_rank,id ASC;
```

这种做法也得理解理解

```sql
select a.id,a.number,
(select count(distinct b.number) from passing_number b where b.number>=a.number )
 from passing_number a order by a.number desc, a.id asc;
```

61.

有一个person表，主键是id，如下:

![](https://uploadfiles.nowcoder.com/images/20200813/557336_1597308809115_59865103D43294F21EB0E6AA59EE2533)

有一个任务(task)表如下，主键也是id，如下:

![](https://uploadfiles.nowcoder.com/images/20200813/557336_1597308818265_982AA463886BC0847F36791DBF155842)

请你找到每个人的任务情况，并且输出出来，没有任务的也要输出，而且输出结果按照person的id升序排序，输出情况如下:

![](https://uploadfiles.nowcoder.com/images/20200813/557336_1597308835496_51484617FEE1F3A5CD46EAA4F7D900CA)

```sql
SELECT person.id,person.name,task.content
FROM person LEFT JOIN task ON person.id=task.person_id
ORDER BY person.id ASC;
```

62.

现在有一个需求，让你统计正常用户发送给正常用户邮件失败的概率:
有一个邮件(email)表，id为主键， type是枚举类型，枚举成员为(completed，no_completed)，completed代表邮件发送是成功的，no_completed代表邮件是发送失败的。简况如下:

![](https://uploadfiles.nowcoder.com/images/20200817/557336_1597652115615_43081841018939871F6352EF230E6D8E)

第1行表示为id为2的用户在2020-01-11成功发送了一封邮件给了id为3的用户;
...
第3行表示为id为1的用户在2020-01-11**没有成功**发送一封邮件给了id为4的用户;
...
第6行表示为id为4的用户在2020-01-12成功发送了一封邮件给了id为1的用户;


下面是一个用户(user)表，id为主键，is_blacklist为0代表为正常用户，is_blacklist为1代表为黑名单用户，简况如下:

![](https://uploadfiles.nowcoder.com/images/20200817/557336_1597652932880_7440E658C1F32DF28A6F4360EAB2D9BB)

结果表示:

2020-01-11失败的概率为0.500，因为email的第1条数据，发送的用户id为2是黑名单用户，所以不计入统计，正常用户发正常用户总共2次，但是失败了1次，所以概率是0.500;

2020-01-12没有失败的情况，所以概率为0.000.
(注意: sqlite 1/2得到的不是0.5，得到的是0，只有1*1.0/2才会得到0.5，sqlite四舍五入的函数为round)

注意先乘1.0，不然会得到0

```sql
SELECT e.date,ROUND(
    SUM(CASE e.type WHEN 'completed' THEN 0 WHEN 'no_completed' THEN 1 END)*1.0/COUNT(e.type),3) AS p
FROM email e INNER JOIN user u1 ON e.send_id=u1.id INNER JOIN user u2 ON e.receive_id=u2.id 
WHERE u1.is_blacklist=0 AND u2.is_blacklist=0
GROUP BY e.date ORDER BY e.date;
```

63.

牛客每天有很多人登录，请你统计一下牛客每个用户最近登录是哪一天。

有一个登录(login)记录表，简况如下:

![](https://uploadfiles.nowcoder.com/images/20200817/557336_1597652540640_64D4B3DE58C69D6263B209D2B1AB6393)

第1行表示id为2的用户在2020-10-12使用了客户端id为1的设备登录了牛客网
。。。
第4行表示id为3的用户在2020-10-13使用了客户端id为2的设备登录了牛客网


请你写出一个sql语句查询每个用户最近一天登录的日子，并且按照user_id升序排序，上面的例子查询结果如下:

![](https://uploadfiles.nowcoder.com/images/20201026/557336_1603700593855_F4BBCB1D5E81A508C2A06167DCF2F5ED)

老师说过select 的字段要包含在group或者要用聚合函数

```sql
SELECT user_id,MAX(date) d
FROM login
GROUP BY user_id
ORDER BY user_id ASC;
```

64.

牛客每天有很多人登录，请你统计一下牛客每个用户最近登录是哪一天，用的是什么设备.

有一个登录(login)记录表，简况如下:

![](https://uploadfiles.nowcoder.com/images/20200901/557336_1598928123396_B0B18451FDC78934AF1D2CF7E366835C)

第1行表示id为2的用户在2020-10-12使用了客户端id为1的设备登录了牛客网
。。。
第4行表示id为3的用户在2020-10-13使用了客户端id为2的设备登录了牛客网

还有一个用户(user)表，简况如下:

![](https://uploadfiles.nowcoder.com/images/20200817/557336_1597652611050_C098FF7CF52D3DC2ECE5019F4E7A5E88)

![](https://uploadfiles.nowcoder.com/images/20200817/557336_1597652618264_F2C14AA3F53E74C2FE5A266283E56241)

请你写出一个sql语句查询每个用户最近一天登录的日子，用户的名字，以及用户用的设备的名字，并且查询结果按照user的name升序排序，上面的例子查询结果如下

![](https://uploadfiles.nowcoder.com/images/20200817/557336_1597652643237_528AC16903A00B04890D9E308B2B5FDE)

老生常谈，group by后select后只能跟聚合和group by的字段

```sql
SELECT u.name,c.name,l.date
FROM login l INNER JOIN user u ON l.user_id=u.id 
INNER JOIN client c ON l.client_id=c.id
INNER JOIN (SELECT user_id,MAX(date) max_date FROM login GROUP BY user_id) AS T ON u.id=T.user_id
WHERE l.date=T.max_date
ORDER BY u.name ASC
```

这个答案更简明

```sql
select u.name,c.name,l1.date
from login l1,user u,client c
where l1.date=(select max(l2.date) from login l2 where l1.user_id=l2.user_id)
and l1.user_id=u.id
and l1.client_id=c.id
order by u.name
```

总结一下，遇到取最大最小值的问题或者是计数，可以对主键进行连接，然后增加判断条件，排序就用大于小于等于，最大最小就用max min

65.

牛客每天有很多人登录，请你统计一下牛客新登录用户的次日成功的留存率，
有一个登录(login)记录表，简况如下:

![](https://uploadfiles.nowcoder.com/images/20201026/557336_1603701796116_54BBC7EED94F0CAAFEC8DB042BBBCD01)

第1行表示id为2的用户在2020-10-12使用了客户端id为1的设备第一次新登录了牛客网
。。。

第4行表示id为3的用户在2020-10-12使用了客户端id为2的设备登录了牛客网

。。。

最后1行表示id为1的用户在2020-10-14使用了客户端id为2的设备登录了牛客网

请你写出一个sql语句查询新登录用户次日成功的留存率，即第1天登陆之后，第2天再次登陆的概率,保存小数点后面3位(3位之后的四舍五入)，上面的例子查询结果如下:

(sqlite里查找某一天的后一天的用法是:date(yyyy-mm-dd, '+1 day')，四舍五入的函数为round，sqlite 1/2得到的不是0.5，得到的是0，只有1*1.0/2才会得到0.5

mysql里查找某一天的后一天的用法是:DATE_ADD(yyyy-mm-dd,INTERVAL 1 DAY)，四舍五入的函数为round)

```sql

```

66.

牛客每天有很多人登录，请你统计一下牛客每个日期登录新用户个数，
有一个登录(login)记录表，简况如下:

![](https://uploadfiles.nowcoder.com/images/20200820/557336_1597903671918_2C3C4BF94A59FB9AEEA6FEA89DEE17C9)

第1行表示id为2的用户在2020-10-12使用了客户端id为1的设备登录了牛客网，因为是第1次登录，所以是新用户
。。。
第4行表示id为2的用户在2020-10-13使用了客户端id为2的设备登录了牛客网，因为是第2次登录，所以是老用户
。。
最后1行表示id为4的用户在2020-10-15使用了客户端id为1的设备登录了牛客网，因为是第2次登录，所以是老用户请你写出一个sql语句查询每个日期登录新用户个数，并且查询结果按照日期升序排序，上面的例子查询结果如下:

![](https://uploadfiles.nowcoder.com/images/20200820/557336_1597903683115_47DE8F52D6F847E788BB9EF6279481C4)

```sql

```

67.

牛客每天有很多人登录，请你统计一下牛客每个日期新用户的次日留存率。
有一个登录(login)记录表，简况如下:

![](https://uploadfiles.nowcoder.com/images/20200820/557336_1597903752757_A02F3DF1419BC2D3D4EE9B2B4557053B)

第1行表示id为2的用户在2020-10-12使用了客户端id为1的设备登录了牛客网，因为是第1次登录，所以是新用户
。。。
第4行表示id为2的用户在2020-10-13使用了客户端id为2的设备登录了牛客网，因为是第2次登录，所以是老用户
。。
最后1行表示id为4的用户在2020-10-15使用了客户端id为1的设备登录了牛客网，因为是第2次登录，所以是老用户

请你写出一个sql语句查询每个日期新用户的次日留存率，结果保留小数点后面3位数(3位之后的四舍五入)，并且查询结果按照日期升序排序，上面的例子查询结果如下:

![](https://uploadfiles.nowcoder.com/images/20200820/557336_1597903761838_F734DB69B9941F0DF86776922B0CF347)

```sq

```

68.❌

牛客每天有很多人登录，请你统计一下牛客每个用户查询刷题信息，包括: 用户的名字，以及截止到某天，累计总共通过了多少题。 不存在没有登录却刷题的情况，但是存在登录了没刷题的情况，不会存在刷题表里面，有提交代码没有通过的情况，但是会记录在刷题表里，只不过通过数目是0。
有一个登录(login)记录表，简况如下:

![](https://uploadfiles.nowcoder.com/images/20200820/557336_1597903934415_A5E822A1E15CEBC7E16183ECD9D7CC7A)

第1行表示id为2的用户在2020-10-12使用了客户端id为1的设备登录了牛客网
。。。
第5行表示id为3的用户在2020-10-13使用了客户端id为2的设备登录了牛客网
有一个刷题（passing_number)表，简况如下:

![](https://uploadfiles.nowcoder.com/images/20200820/557336_1597903944909_9C60ECB78BE56145269BDFA74F1074D3)

第1行表示id为2的用户在2020-10-12通过了4个题目。
。。。
第3行表示id为1的用户在2020-10-13提交了代码但是没有通过任何题目。
第4行表示id为4的用户在2020-10-13通过了2个题目
还有一个用户(user)表，简况如下:

![](https://uploadfiles.nowcoder.com/images/20200820/557336_1597903952301_4763AF7B377AF42CB5A87C53B965DC45)

请你写出一个sql语句查询刷题信息，包括: 用户的名字，以及截止到某天，累计总共通过了多少题，并且查询结果先按照日期升序排序，再按照姓名升序排序，有登录却没有刷题的哪一天的数据不需要输出，上面的例子查询结果如下:

![](https://uploadfiles.nowcoder.com/images/20201015/557336_1602736313104_6BE4C7264FB9F7CED1DD6A6D44652D6C)

题目的含义是各个用户在不同时间累积答题求和，采用sum函数进行开窗处理，将user_id进行分区，再通过时间升序排序，进而实现了在每个user_id分区中以升序日期排序的通过题数的逐个递加

```sql
SELECT name,date,
SUM(number) over(partition by user_id order by date)
FROM passing_number p
LEFT JOIN user u
ON p.user_id=u.id
ORDER BY date,name
```

还有一种是同一张表内连接的方法，这个得花时间理解，对于这种分组累加很有必要

```sql
SELECT u.name,pn1.date,SUM(pn2.number)
FROM user u INNER JOIN passing_number pn1 ON u.id=pn1.user_id
INNER JOIN passing_number pn2 ON pn1.user_id=pn2.user_id
WHERE pn1.date>=pn2.date
GROUP BY pn1.id,pn1.date
ORDER BY pn1.date ASC,u.name ASC;
```

69.

牛客每次考试完，都会有一个成绩表(grade)，如下:

![](https://uploadfiles.nowcoder.com/images/20200919/557336_1600513881865_4F8855D8505FBEAA5F2561794CD2C0B0)

第1行表示用户id为1的用户选择了C++岗位并且考了11001分

。。。

第8行表示用户id为8的用户选择了前端岗位并且考了9999分

 请你写一个sql语句查询各个岗位分数的平均数，并且按照分数降序排序，结果保留小数点后面3位(3位之后四舍五入):

![](https://uploadfiles.nowcoder.com/images/20200919/557336_1600513888833_CE4B33A183934FE8410A2864D09E32E7)

```sql
SELECT job,ROUND(AVG(score),3) AS avg
FROM grade
GROUP BY job
ORDER BY avg DESC;
```

70.

牛客每次考试完，都会有一个成绩表(grade)，如下:

![](https://uploadfiles.nowcoder.com/images/20200919/557336_1600514055044_2CCD09B7C5B20F369E0D5D52AAE75E69)

第1行表示用户id为1的用户选择了C++岗位并且考了11001分

。。。

第8行表示用户id为8的用户选择了前端岗位并且考了9999分

请你写一个sql语句查询用户分数大于其所在工作(job)分数的平均分的所有grade的属性，并且以id的升序排序，如下:

![](https://uploadfiles.nowcoder.com/images/20200919/557336_1600514070904_62340E72F33FAB48C78342F76BA93044)

感觉自己写的很烂

```sql
SELECT g.id,g.job,g.score 
FROM grade g,(SELECT job,AVG(score) avg_score FROM grade GROUP BY job) AS t
WHERE g.job=t.job AND g.score>t.avg_score 
ORDER BY g.id ASC;
```

这个开窗函数看起来就很简洁

```sql
SELECT id,job,score FROM (SELECT *,AVG(score) OVER (PARTITION BY job) avg_score FROM grade)
WHERE score>avg_score
ORDER BY id;
```

71.

牛客每次举办企业笔试的时候，企业一般都会有不同的语言岗位，比如C++工程师，JAVA工程师，Python工程师，每个用户笔试完有不同的分数，现在有一个分数(grade)表简化如下:

![](https://uploadfiles.nowcoder.com/images/20200817/557336_1597652427039_07E19104EBD63DA9EDB41F5DD4489F99)

第1行表示用户id为1的选择了language_id为1岗位的最后考试完的分数为12000，
....
第7行表示用户id为7的选择了language_id为2岗位的最后考试完的分数为11000，

不同的语言岗位(language)表简化如下:

![](https://uploadfiles.nowcoder.com/images/20200817/557336_1597652437695_C8AFE1A16BE6BBA7929B73D79DE0E398)

请你找出每个岗位分数排名前2的用户，得到的结果先按照language的name升序排序，再按照积分降序排序，最后按照grade的id升序排序，得到结果如下:

![](https://uploadfiles.nowcoder.com/images/20200817/557336_1597652444918_D40D8F8B37A92E9E7A15A5231F7D3822)

```sql
SELECT id,name,score FROM
(SELECT g.id AS id,l.name AS name,g.score AS score,DENSE_RANK() OVER (PARTITION BY g.language_id ORDER BY score DESC) AS rank_score
FROM grade g INNER JOIN language l ON g.language_id=l.id 
ORDER BY l.name ASC,score DESC,id ASC)
WHERE rank_score<=2
```

72.

牛客每次考试完，都会有一个成绩表(grade)，如下:

![](https://uploadfiles.nowcoder.com/images/20200919/557336_1600514637504_1968643213375655C970A80233B90BFA)

第1行表示用户id为1的用户选择了C++岗位并且考了11001分

。。。

第8行表示用户id为8的用户选择了前端岗位并且考了9999分

请你写一个sql语句查询各个岗位分数升序排列之后的中位数位置的范围，并且按job升序排序，结果如下:

![](https://uploadfiles.nowcoder.com/images/20200919/557336_1600514646549_3FA97323294AEEA1C641FFAD64EF069B)

解释:

第1行表示C++岗位的中位数位置范围为[2,2]，也就是2。因为C++岗位总共3个人，是奇数，所以中位数位置为2是正确的(即位置为2的10000是中位数)

第2行表示Java岗位的中位数位置范围为[1,2]。因为Java岗位总共2个人，是偶数，所以要知道中位数，需要知道2个位置的数字，而因为只有2个人，所以中位数位置为[1,2]是正确的(即需要知道位置为1的12000与位置为2的13000才能计算出中位数为12500)

第3行表示前端岗位的中位数位置范围为[2,2]，也就是2。因为前端岗位总共3个人，是奇数，所以中位数位置为2是正确的(即位置为2的11000是中位数)

(注意: sqlite 1/2得到的不是0.5，得到的是0，只有1*1.0/2才会得到0.5，sqlite四舍五入的函数为round，sqlite不支持floor函数，支持cast(x as integer) 函数，不支持if函数，支持case when ...then ...else ..end函数)

```sql

```

73.

牛客每次考试完，都会有一个成绩表(grade)，如下:

![](https://uploadfiles.nowcoder.com/images/20200919/557336_1600514714857_0A390C0623561C804BC63A48121FD094)

第1行表示用户id为1的用户选择了C++岗位并且考了11001分

。。。

第8行表示用户id为8的用户选择了前端岗位并且考了9999分

请你写一个sql语句查询各个岗位分数的中位数位置上的所有grade信息，并且按id升序排序，结果如下:

![](https://uploadfiles.nowcoder.com/images/20200919/557336_1600514720752_917884B00BBAAAF5E9969D36073440C3)



```sql

```

