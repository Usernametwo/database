# 子查询

```sql
select last_name
from employees
where salary >
(select salary
from employees
where last_name = 'Abel')

select employee_id, last_name
from employees
where salary =
(select min(salary)
from employees)

select employee_id ,last_name
from employees
where salary < any
(select salary
from employees
where job_id = 'IT_PROG')
and job_id <> 'IT_PROG'
```

单行比较必须对应单行子查询（返回单一结果的查询），比如=,>等比较操作符
多行比较必须对应多行子查询（返回一个数据集的查询）,比如 IN, > ANY, > ALL等