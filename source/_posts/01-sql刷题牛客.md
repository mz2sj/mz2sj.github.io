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

