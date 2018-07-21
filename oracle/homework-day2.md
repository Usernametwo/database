# homework day2

```sql
-- 显示部门编号、部门名字、该部门的员工数、每个部门的平均工资，部门负责人信息，包括姓名、薪水、职业；平均工资保留2位小数，千分位分隔符显示；结果按部门升序
select d.department_id,
       d.department_name,
       count(e.employee_id) as employees,
       to_char(avg(e.salary),'999,999,999.00') as avg_sal,
       manager.last_name as last_name,
       manager.salary as salary,
       manager.job_id as job_id
from   employees e,
       departments d,
       employees manager
where  d.department_id = e.department_id(+) and
       d.manager_id = manager.employee_id (+)
group by d.department_id,
         d.department_name,
         manager.first_name,
         manager.last_name,
         manager.salary,
         manager.job_id
order by d.department_id

-- 显示员工数最多的部门信息，显示部门ID、名称、部门员工数，部门的主管经理姓名
select d.department_id,
       d.department_name,
       count(*),
       manager.first_name||' '||manager.last_name as manager_name
from   departments d,
       employees manager,
       employees e
where  d.department_id = e.department_id (+) and
       d.manager_id = manager.employee_id (+) and
       d.department_id = (
                       select d.department_id
                       from employees e,
                            departments d
                       where d.department_id =  e.department_id (+)
                       group by d.department_id
                       having count(e.employee_id) = (
                                                   select max(count(e.employee_id))
                                                   from departments d,
                                                        employees e
                                                   where d.department_id = e.department_id (+)
                                                   group by d.department_id
                                                   )
                       )
group by d.department_id,
         d.department_name,
         manager.first_name,
         manager.last_name

select d.department_id,
       d.department_name,
       department_max_id.count_num as "COUNT(*)",
       manager.first_name||' '||manager.last_name as manager_name
from   departments d,
       employees manager,
       (
         select d.department_id,
                count(e.employee_id) as count_num
         from   employees e,
                departments d
         where  d.department_id =  e.department_id (+)
         group by d.department_id
         having count(e.employee_id) = (
             select max(count(e.employee_id)) as max_count
             from   departments d, employees e
             where  d.department_id = e.department_id (+)
             group by d.department_id
           )
        ) department_max_id
where d.department_id = department_max_id.department_id and
      d.manager_id = manager.employee_id (+)

-- 此种方法效率不高
with   emp_count as (
    select distinct d.department_id,
        d.department_name,
        count(*) over(partition by d.department_id) as num,
        manager.last_name
    from departments d, employees e, employees manager
    where d.department_id = e.department_id(+) and
        d.manager_id = manager.employee_id(+)
    order by num desc
)
select department_id, department_name, num, last_name
from emp_count
where rownum <= 1

-- 显示工号、姓名、薪水、部门编号、薪资，薪资与部门平均工资的差异情况；按照部门ID排序
select e.employee_id,
       e.last_name,
       e.department_id,
       e.salary,
       (e.salary - department_avg.avg_salary) as salary_gap
from   employees e,
       (select department_id, avg(salary) as avg_salary
        from employees
        group by department_id) department_avg
where  e.department_id = department_avg.department_id (+)
order by e.department_id

-- 分析函数
select e.employee_id,
       e.last_name,
       e.department_id,
       e.salary,
       e.salary - avg(e.salary) over (partition by d.department_id)  as salary_gap
from   employees e, departments d
where  e.department_id = d.department_id (+)
order by e.department_id

-- 周几录取的人数最少，显示人名和日期
select e.employee_id,
       e.first_name,
       e.last_name,
       to_char(hire_date, 'DAY') as day
from   employees e
where  to_char(e.hire_date, 'DAY') = (
          select to_char(hire_date, 'DAY')
          from   employees
          group by to_char(hire_date, 'DAY')
          having count(employee_id) = (
            select min(count(employee_id))
            from   employees
            group by to_char(hire_date, 'DAY')
          )
       )

-- 查询所有hr用户下的索引(在system用户下)
Select *
From dba_indexes
Where owner = 'HR'
Select *
From all_indexes
Where owner = 'HR'
Select *
From user_indexes
Where table_owner = 'HR'

```