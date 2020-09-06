---
title: 01-sql刷题牛客
date: 2020-08-25 23:22:16
tags: [sql,数据分析,刷题]
categories: [数据分析,sql]
---

今天是刷题记录的第一天，加油啦，小孟冲冲冲！！！

 `查找最晚入职员工的所有信息，为了减轻入门难度，目前所有的数据里员工入职的日期都不是同一天(sqlite里面的注释为--,mysql为comment)`
`CREATE TABLE employees (`

`emp_no int(11) NOT NULL, -- '员工编号'`

`birth_date date NOT NULL,`

`first_name varchar(14) NOT NULL,`

`last_name varchar(16) NOT NULL,`

`gender char(1) NOT NULL,`

`hire_date date NOT NULLPRIMARY KEY (emp_no));`,
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



