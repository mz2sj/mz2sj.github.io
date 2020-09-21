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

`使用group by子句时，select子句中只能有聚合键、聚合函数、常数。emp_no并不符合这个要求。`

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

