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



