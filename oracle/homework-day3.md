# homework day3

```sql
-- 显示所有在欧洲区域工作的员工，显示他们的部门，姓名，岗位，薪资，国籍，结果按照国籍升序，薪资降序排列
select  d.department_name,
        e.last_name,
        e.job_id,
        e.salary,
        c.country_name
from    employees e,
        departments d,
        locations l,
        countries c
where   e.department_id = d.department_id (+) and
        d.location_id = l.location_id (+) and
        l.country_id = c.country_id (+) and
        c.region_id = 1
order by c.country_name, e.salary desc

-- 用尽可能多的方式显示姓名以小写’s’结尾的人员总数；至少两种方法

select count(*)
from employees
where last_name like '%s'

select count(*)
from employees
where substr(last_name, -1) = 's'

select count(*)
from employees
where regexp_like(last_name, '+[s]$')

-- 查询所有薪资大于‘IT_PROG’部门任何一人的薪资员工信息，显示姓名、薪资、岗位；用2种方法实现
select  employee_id,
        last_name,
        job_id, salary
from    employees
where   salary > all (
            select salary
            from employees
            where job_id = 'IT_PROG'
        )

select  employee_id,
        last_name,
        job_id,
        salary
from    employees
where   salary >  (
            select max(salary)
            from employees
            where job_id = 'IT_PROG'
        )

-- 创建一张表与employees表相同的表，表名：employees_工号;将’Marketing’，’IT’ 两个部门下员工导入该表。提供脚本
create table employee_copy as
select * from employees
where job_id like '%IT%' or job_id like '% Marketing’%'

-- 查询HR下所有的约束
Select *
From dba_constraints
Where owner = 'HR'
Select *
From all_constraints
Where owner = 'HR'
Select *
From user_constraints
where owner = 'HR'
```