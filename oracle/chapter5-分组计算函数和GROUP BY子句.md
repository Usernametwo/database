# **分组计算单元和GROUP BY子句**

## **分组计算**

1. 求和：SUM
2. 平均值：AVG
3. 最大值：MAX
4. 最小值：MIN
5. 求方差：VARIANCE
6. 求标准差：STDEEV
7. 计数：COUNT

```sql
select AVG(salary), MAX(salary), MIN(salary), SUM(salary)
from employees
where job_id like '%REP%'

select MIN(hire_date), MAX(hire_date)
from employees
```

MAX和MIN可用于任何类型，其他的只能用于数值类型

```sql
select count(*) from
( select department_id
from employees
group by department_id )

select count(department_id) from
( select department_id
from employees
group by department_id )
```

count(*)统计所有的行数，包括null的行

count(expr)统计表达式不为空的行数

count(distinct expr)统计表达式不为空且不重复的行数

```sql
--A--
select avg(commission_pct)
from employees;

select(select sum(commission_pct) from employees)/(select count(commission_pct) from employees)
from dual


--B--
select avg(nvl(commission_pct, 0))
from employees

select(select sum(commission_pct) from employees)/(select count(*) from employees)
from dual
```

avg和sum均不统计空行
***

## **GROUP BY子句**

```sql
select avg(salary)
from employees
group by department_id

select department_id dept_id, job_id, sum(salary)
from employees
group by department_id, job_id
```

使用group by子句进行分组

select查询语句中同时选择分组计算函数表达式和其他独立字段时，其他独立字段必须出现在group by子句中

```sql
select department_id, count(last_name)
from employees
where salary > 5000
group by department_id
having avg(salary) > 8000
```

不能在where条件中使用分组计算函数表达式，必须使用having子句

```sql
select max(avg(salary))
from employees
group by department_id
```

分组函数的嵌套使用