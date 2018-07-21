# homework day1

```sql
-- 根据数据字典，查询当前用户下有哪些表
Select *
From dba_tables
Where owner = 'STUDENT1'
Select *
From all_tables
Where owner = 'STUDENT1'
Select *
From user_tables

-- 查询所有员工编号，员工姓名、邮件、雇佣日期、部门、部门详细地址信息
select e.employee_id,
       e.first_name||' '||e.last_name as full_name,
       e.email,
       e.hire_date,
       d.department_name,
       manager.first_name||' '||manager.last_name manager_name,
       l.street_address,
       l.city,
       l.state_province
from   employees e, departments d, employees manager, locations l
where  e.department_id = d.department_id(+)
and    e.manager_id = manager.employee_id(+)
and    d.location_id = l.location_id(+)

-- 查询1997年入职员工员工编号，员工姓名，雇佣日期，工龄(保留2位小数)、按照First_Name升序排列
select e.employee_id,
       e.first_name,
       e.last_name,
       --to_char(months_between(sysdate, hire_date)/12, '99999.99') as work_age
       to_char((sysdate-hire_date)/365, '999,999,999.99') as work_age
from   employees e
where  to_number(to_char(hire_date, 'yyyy')) = 1997
order by first_name

-- 显示姓、薪水，佣金（commission） ，然后按薪水降序排列
select e.first_name,
       e.last_name,
       e.salary,
       --nvl
       e.commission_pct
from   employees e
where  e.commission_pct is not null
order by e.salary desc

-- 显示姓名、薪水，佣金（commission），佣金为空的，统一加上0.05;其余的加上0.03，按照薪资降序、变更后佣金升序排列
select e.first_name,
       e.last_name,
       e.salary,
       e.commission_pct,
       nvl(e.commission_pct,0) + nvl2(e.commission_pct, 0.03, 0.05) as new_commission_pct
from employees e
order by e.salary desc, new_commission_pct * salary

-- 显示名字以J、K、L、M开头的雇员
select *
from   employees
--substr(last_name, 1, 1)
--regexp_like(last_name, '^[JKLM]+')
where  last_name like 'J%'
or     last_name like 'K%'
or     last_name like 'L%'
or     last_name like 'M%'

-- 显示员工姓名，薪水，调整后薪水(按部门调整：IT提升30%,Salse 提升50%、其余部门提升20%)；按照调整后薪水升序排列
select e.first_name,
       e.last_name,
       e.salary,
       --decode
       case d.department_name
            when 'IT' then e.salary*1.3
            when 'Salse' then e.salary*1.5
            else e.salary*1.2
       end new_salary
from   employees e, departments d
where  e.department_id = d.department_id(+)
order  by new_salary

-- 显示在每月中旬雇佣的员工,显示姓名，雇佣日期
select first_name,
       last_name,
       to_char(hire_date,'dd/fmmm/yyyy') as hire_date
from   employees
where  to_number(to_char(hire_date, 'dd')) between 11 and 20

```