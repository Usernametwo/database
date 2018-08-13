# 分析函数

```sql
select e.last_name,
       e.salary,
       d.department_id,
       avg(e.salary) over(partition by d.department_name) dept_avg_sal,
       max(e.salary) over(partition by d.department_name) dept_max_sal,
       min(e.salary) over(partition by d.department_name) dept_min_sal
from employees e, departments d
where e.department_id = d.department_id
order by d.department_id

select d.department_name,
       e.last_name,
       e.salary,
       --如果第三名有两个相同的sal，那么下一名是第五名
       rank() over(partition by d.department_name order by e.salary desc) dept_salary_rank1,
       --如果第三名有两个相同的sal，那么下一名是第四名
       dense_rank() over(partition by d.department_name order by e.salary desc) dept_salary_rank2,
       --在本partition中是第几个，和排名无关，从1开始
       row_number() over(partition by d.department_name order by e.salary desc) dept_salary_rank3
from employees e, departments d
where e.department_id = d.department_id
order by department_name
```

分析函数

```sql
--Error:ora-01466
create table dep_cpy
as select * from departments


delete from dep_cpy
where department_name = 'Finance'

commit

select * from dep_cpy where department_name = 'Finance'

select * from dep_cpy as of timestamp sysdate-5/(24*60)
where department_name = 'Finance'
```

闪回