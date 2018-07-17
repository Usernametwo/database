# 数据库对象-视图

```sql
create view empvu80
as select employee_id ,last_name, salary
from employees
where department_id = 80

create view dept_sum_vu
(name, minsal, maxsal, avgsal)
as select d.department_name, min(e.salary),
max(e.salary), avg(e.salary)
from employees e, departments d
where e.department_id = d.department_id
group by d.department_name
```

创建视图

```sql
drop view empvu80
```

删除视图

```sql
select employee_id, salary
from (
select employee_id,salary
from employees
order by salary)
where ROWNUM <= 10;
```

TOP-N查询