# homework day4

```sql
-- 显示工资涨幅，员工编号，原来的工资和增加的工资： 部门号10、50、110的有5%的涨幅，部门号为60的涨10%的工资部门号为20和80涨幅为15%，部门为90的不涨工资
select case department_id
            when 10 then '5% raise'
            when 50 then '5% raise'
            when 110 then '5% raise'
            when 60 then '10% raise'
            when 20 then '15% raise'
            when 80 then '15% raise'
            when 90 then '0% raise'
      end "RAISE",
      salary,
      employee_id,
      case department_id
            when 10 then salary*0.05
            when 50 then salary*0.05
            when 110 then salary*0.05
            when 60 then salary*0.1
            when 20 then salary*0.15
            when 80 then salary*0.15
            when 90 then 0
      end "NEW_SALARY"
from employees

-- 找出哪个工作的最大工资大于整个公司内最大的工资的一半，使用with子句，显示出工作名，最大工资
with 
company_max_sal as (
select max(salary) as max_sal
from employees
),

job_max_sal as (
select max(salary) as max_sal, job_id
from employees
group by job_id
)

select j.job_title, jm.max_sal
from company_max_sal cm, job_max_sal jm, jobs j
where jm.max_sal * 2 > cm.max_sal and
jm.job_id = j.job_id (+)

-- 使用EXISTS/NOT EXISTS 找出部门员工数不大于3的部门信息，显示部门编号，部门名称
select d.department_id, d.department_name
from departments d
where exists ((
      select count(*)
      from employees e
      where e.department_id = d.department_id
      having count(*) <= 3)
)

-- 显示员工编号、上司、等级、姓，要求姓前面加上下划线，等级越低横线越长
select employee_id, manager_id, level, lpad(last_name, length(last_name) + level*2 -2, '_')
from employees
start with last_name = 'KingA'
connect by prior employee_id = manager_id

-- 将employees表复制到EMP_CPY,在EMP_CPY上添加SEX列，根据实际情况，将sex列内容补充完整;复习group rollup函数
create table emp_cpy as
select * from employees

alter table emp_cpy
add (sex varchar2(12))

update emp_cpy
set sex = (
    select case mod(employee_id, 2)
                when 0 then 'man'
                when 1 then 'woman'
           end "SEX"
    from employees
    where employees.employee_id = emp_cpy.employee_id
```