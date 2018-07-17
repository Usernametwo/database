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

MAX和MIN可用于任何类型，其他的只能用于`数值类型`

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

```sql
select department_id, manager_id, count(*), sum(salary) as sum_salary
from employees
group by department_id, manager_id
order by department_id, manager_id

select department_id, manager_id, count(*), sum(salary) as sum_salary
from employees
group by rollup(department_id, manager_id)
order by department_id, manager_id

select department_id, manager_id, employee_id, count(*), sum(salary) as sum_salary
from employees
group by employee_id, rollup(department_id, manager_id)
order by department_id, manager_id
```

ROLLUP : 除了正常的分组结果外，rollup还会多返回两个分组,group by rollup(A, B)的结果实际上是group by A,B + group by A + 没有group by 的结果; group by rollup(A, B), C 的结果实际上是group by A,B,C + group by A,C + group by C 的结果

```sql
select department_id, manager_id, count(*), sum(salary) as sum_salary
from employees
group by department_id, manager_id
order by department_id, manager_id

select department_id, manager_id, count(*), sum(salary) as sum_salary
from employees
group by cube(department_id, manager_id)
order by department_id, manager_id

select department_id, manager_id, employee_id, count(*), sum(salary) as sum_salary
from employees
group by employee_id, cube(department_id, manager_id)
order by department_id, manager_id
```

CUBE : 除了正常的分组结果外，cube还会多返回三个分组,group by cube(A, B)的结果实际上是group by A,B + group by A + group by B + 没有group by 的结果; group by rollup(A, B), C 的结果实际上是group by A,B,C + group by A,C + group by B,C + group by C 的结果

```sql
select department_id, manager_id, count(*), sum(salary) as sum_salary,
grouping(department_id) as F1,
grouping(manager_id) as F2
from employees
group by rollup(department_id, manager_id)
order by department_id, manager_id

select department_id, manager_id, count(*), sum(salary) as sum_salary,
grouping(department_id) as F1,
grouping(manager_id) as F2
from employees
group by cube(department_id, manager_id)
order by department_id, manager_id

select department_id, manager_id, count(*), sum(salary) as sum_salary,
grouping(department_id) as F1,
grouping(manager_id) as F2
from employees
group by cube(department_id, manager_id)
-- can not use F1 or F2
having grouping(department_id) = 1 or grouping(manager_id) = 1
order by department_id, manager_id
```

Grouping : 如果是 grouping by A 出来的结果, 那么 grouping(B) 就是 1，其他同理(可以用来区分rollup和cube多出来的行)

```sql
select department_id, manager_id, count(*), sum(salary) as sum_salary,
grouping_id(department_id, manager_id) as grouping_id
from employees
group by cube(department_id, manager_id)
order by department_id, manager_id
```

GROUPING_ID : grouping_id(A, B) 的值为 F_A + F_B，如果是 group by A 出来的结果就是1，如果是没有 group by 出来的结果集结果为 2

```

```