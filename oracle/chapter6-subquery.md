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

```sql
select a.last_name, a.salary,
       a.department_id, b.salavg
from employees a, (select
                  department_id, avg(salary) salavg
                  from employees
                  group by department_id) b
where a.department_id = b.department_id
and   a.salary > b.salavg
```

非相关子查询当做一张表来用

```sql
select last_name, salary, department_id
from employees outer
where salary > (select avg(salary)
                from employees
                where department_id = outer.department_id)

select e.employee_id, last_name, e.job_id
from employees e
where 2 <= (select count(*)
            from job_history
            where employee_id = e.employee_id)
```

相关子查询：子查询中参考了外部主查询的表

```sql
select employee_id, last_name, job_id, department_id
from employees outer
where exists(select 'X'
             from employees
             where manager_id = outer.employee_id)

select employee_id, last_name, job_id, department_id
from employees
where employee_id in (select manager_id
                      from employees
                      where manager_id is not null)

select department_id, department_name
from departments d
where not exists (select 'X'
                  from employees
                  where department_id = d.department_id)

select department_id, department_name
from departments
where department_id not in (select department_id
                            from employees
                            --necessary
                            where department_id is not null)
```

EXISTS 和 IN : not in 语句后面的set里面不要有null

```sql
create table employee_copy
as select * from employees

alter table employee_copy
add(department_name varchar2(24))

update employee_copy e
set department_name = (
                       select department_name
                       from departments d
                       where e.department_id = d.department_id
                       )
```

在UPDATE中使用相关子查询

```sql
delete table job_history JH
       where employee_id = (
            select employee_id
            from employees E
            where JH.employee_id = E.employee_id
            and start_date = (
                        select min(start_date)
                        from job_history JH
                        where JH.employee_id = E.employee_id
                    )
            and 5 > (
                    select count(*)
                    from job_history JH
                    where JH.employee_id = E.employee_id
                    group by employee_id
                    having count(*) >= 4
                )
            )
```

在DELETE中使用相关子查询

```sql
with
dept_costs as (
           select d.department_name, sum(e.salary) as dept_total
           from employees e, departments d
           where e.department_id = d.department_id
           group by d.department_name
),
avg_cost as (
         select sum(dept_total) / count(*) as dept_avg
         from dept_costs
)
select * from dept_costs
where dept_total > (select dept_avg
                    from avg_cost)
order by department_name
```

使用 WITH 子句：可以适当提高性能