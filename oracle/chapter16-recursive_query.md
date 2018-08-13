# 递归查询

```sql
select last_name||' reports to'||
prior last_name "Walk Top Down"
from employees
start with last_name = 'KingA'
connect by prior employee_id = manager_id

select employee_id, last_name, job_id, manager_id
from employees
start with employee_id = 101
connect by prior manager_id = employee_id

select lpad(last_name, length(last_name) + (level*2) - 2, '_')
       as org_chart
from employees
start with last_name='KingA'
connect by prior employee_id = manager_id
```

递归查询