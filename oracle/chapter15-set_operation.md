# 集合操作

```sql
select employee_id, job_id
from employees
union
select employee_id, job_id
from job_history
```

UNION：去除重复记录

```sql
select employee_id, job_id
from employees
union all
select employee_id, job_id
from job_history
order by employee_id
```

UNION ALL：保留重复记录

```sql
select employee_id, job_id
from employees
intersect
select employee_id, job_id
from job_history
order by employee_id
```

INTERSECT：取交集

```sql
select employee_id, job_id
from employees
minus
select employee_id, job_id
from job_history
order by employee_id
```

MINUS：取差集