# Exercise

```sql
-- 1 查询雇佣时间在1997年之后的员工信息。
select employee_id, first_name, last_name, email, phone_number, to_char(hire_date, 'dd-Month-yy'), job_id
from employees e
where to_char(hire_date,'yyyy') > 1997

--2 查询有提成的员工信息（last name,  job, salary, and commission），并按工资降序排列。
select last_name, job_id, salary, to_char(commission_pct)
from employees
where commission_pct is not null and commission_pct != 0
order by salary desc

--3 Show the employees that have no commission with a 10% raise in their salary (round off the salaries).
select 'The salary of '||last_name||' after a 10% raise is '||round(salary*1.1)
from employees
where commission_pct is null or commission_pct = 0

--4 Show the last names of all employees together with the number of years and the number ofcompleted months that they have been employed.
select last_name, trunc(months_between(sysdate, hire_date)/12) as years, trunc(mod(months_between(sysdate, hire_date), 12)) as months
from employees

--5 Show those employees that have a name starting with J, K, L, or M.
select last_name
from employees
where last_name like 'J%'
or    last_name like 'K%'
or    last_name like 'L%'
or    last_name like 'M%'

--6 Show all employees, and indicate with “Yes” or “No” whether they receive a commission
select last_name, salary, case
                            when commission_pct is null then 'No'
                            when commission_pct = 0 then 'No'
                            else 'Yes'
                          end "COM"
from employees

--7 Show the department names, locations, names, job titles, and salaries of employees who workin location 1800.
select d.department_name, d.location_id, e.last_name, e.job_id, e.salary
from employees e, departments d
where e.department_id = d.department_id (+)
and   d.location_id = 1800

--8 How many employees have a name that ends with an n? Create two possible solutions
select count(*)
from employees e
where e.last_name like '%n'

select count(*)
from employees e
-- can not use instr because of 'Alnan'
where substr(e.last_name, -1) = 'n'

--9 Show the names and locations for all departments, and the number of employees working in each department. Make sure that departments without employees are included as well.
select d.department_id, d.department_name, d.location_id, count(e.employee_id)
from employees e, departments d
where d.department_id = e.department_id (+)
group by d.department_id, d.department_name, d.location_id

--10 Which jobs are found in departments 10 and 20?
select distinct e.job_id
from employees e, departments d
where d.department_id = e.department_id (+)
and   d.department_id between 10 and 20

--11 Which jobs are found in the Administration and Executive departments, and how many employees do these jobs? Show the job with the highest frequency first.
select e.job_id, count(e.job_id) as frequency
from employees e, departments d
where d.department_id = e.department_id (+)
and   (d.department_name = 'Administration' or d.department_name = 'Executive')
group by e.job_id
order by frequency desc

--12 Show all employees who were hired in the first half of the month (before the 16th of the month).
select last_name, to_char(hire_date, 'dd-month-yy') as hire_date
from employees
where to_char(hire_date, 'dd') < 16

--13 Show the names, salaries, and the number of dollars (in thousands) that all employees earn.
select last_name, to_char(salary,'999999'), trunc(salary/1000) as thousands
from employees

--14 Show all employees who have managers with a salary higher than $15,000. Show the following data: employee name, manager name, manager salary, and salary grade of the manager.
select e.last_name, manager.last_name as manager, to_number(manager.salary, '9999999') as salary, 'E' as GRA
from employees e, employees manager
where e.manager_id = manager.employee_id (+)
and   manager.salary > 15000

--15 Show the department number, name, number of employees, and average salary of all departments, together with the names, salaries, and jobs of the employees working in each department.
select d.department_id, d.department_name,
count(e.employee_id) over (partition by d.department_id) as employees,
avg(e.salary) over (partition by d.department_id) as avg_sal,
last_name, to_number(e.salary, '9999999') as salary, e.job_id
from employees e, departments d
where e.department_id = d.department_id (+)

--16 Show the department number and the lowest salary of the department with the highest average salary.
select e.department_id, min(e.salary)
from employees e, (select avg(salary) as sal
                     from employees
                     group by department_id) avg_sal
group by e.department_id
having avg(e.salary) = max(avg_sal.sal)

--17 Show the department numbers, names, and locations of the departments where no sales representatives work.
select d.department_id, d.department_name, d.manager_id, d.location_id
from employees e, departments d
where d.department_id = e.department_id (+)
and   ('Sales Representative' not in (select j.job_title
                                      from jobs j, employees e
                                      where j.job_id = e.job_id (+)
                                      and e.department_id = d.department_id))
group by d.department_id, d.department_name, d.manager_id, d.location_id
order by d.department_id

select distinct d.department_id, d.department_name, d.manager_id, d.location_id
from departments d, employees e
where d.department_id = e.department_id (+)
and   (e.employee_id not in (select id from sales_reps) or e.employee_id is null)

--18 Show the department number, department name, and the number of employees working in each department that:
--a  Includes fewer than 3 employees
select d.department_id, d.department_name, count(*)
from departments d,employees e
where d.department_id  = e.department_id (+)
group by d.department_id, d.department_name
having count(*) < 3
--b  Has the highest number of employees
select d.department_id, d.department_name, count(*)
from departments d,employees e
where d.department_id  = e.department_id (+)
group by d.department_id, d.department_name
having count(*) = (select max(count(*))
                   from employees e, departments d
                   where d.department_id = e.department_id (+)
                   group by d.department_id
                  )
--c  Has the lowest  number of employees
select d.department_id, d.department_name, count(*)
from departments d,employees e
where d.department_id  = e.department_id (+)
group by d.department_id, d.department_name
having count(*) = (select min(count(*))
                   from employees e, departments d
                   where d.department_id = e.department_id (+)
                   group by d.department_id
                  )

--19 Show the employee number, last name, salary, department number, and the average salary in their department for all employees.
select e.employee_id, e.last_name, d.department_id, avg(e.salary) over(partition by e.department_id)
from employees e, departments d
where e.department_id = d.department_id(+)
order by e.employee_id

--20 Show all employees who were hired on the day of the week on which the highest number of employees were hired.
with max_day as
(select to_char(e.hire_date, 'day') as day
from employees e
group by to_char(e.hire_date, 'day')
having count(e.employee_id) = (select max(count(e.employee_id))
                                    from employees e
                                    group by to_char(hire_date, 'day')))
select last_name, to_char(e.hire_date, 'day') as day
from employees e, max_day
where to_char(e.hire_date, 'day') = max_day.day

--21 Create an anniversary overview based on the hire date of the employees. Sort the anniversaries in ascending order.
select last_name, to_char(hire_date, 'month dd') as birth_day
from employees
order by to_number(to_char(hire_date - trunc(hire_date, 'yyyy')))

--22 Find the job that was filled in the first half of 1990 and the same job that was filled during the same period in 1991.
select job_id
from employees
where hire_date < to_date('1990-6-01','yyyy-mm-dd')
INTERSECT
select job_id
from employees
where hire_date < to_date('1991-6-01','yyyy-mm-dd')

--23 Write a compound query to produce a list of employees showing raise percentages, employee IDs, and old salary and new salary increase. Employees in departments 10, 50, and 110 are given a 5% raise, employees in department 60 are given a 10% raise, employees in departments 20 and 80 are given a  15% raise, and employees in department 90 are not given
select case department_id
            when 10 then '5% raise'
            when 50 then '5% raise'
            when 110 then '5% raise'
            when 60 then '10% raise'
            when 20 then '15% raise'
            when 80 then '15% raise'
            when 90 then '0% raise'
      end "RAISE",
      employee_id,
      salary,
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

--24 Alter the session to set the NLS_DATE_FORMAT to  DD-MON-YYYY HH24:MI:SS.
alter session set NLS_DATE_FORMAT = 'DD-MON-YYYY HH24:MI:SS'

--25 Write queries to display the time zone offsets (TZ_OFFSET) for the following time zones.
--Australia/Sydney
select tz_offset('Australia/Sydney')
from dual
--Chile/Easter Island
select tz_offset('Chile/EasterIsland')
from dual
--Alter the session to set the TIME_ZONE parameter value to the time zone offset of Australia/Sydney
alter session
set time_zone = 'Australia/Sydney'
--. Display the SYSDATE, CURRENT_DATE, CURRENT_TIMESTAMP, and LOCALTIMESTAMP for this session
select sysdate, current_date, current_timestamp, localtimestamp
from dual
--Alter the session to set the TIME_ZONE parameter value to the time zone offset of Chile/Easter Island.
alter session
set time_zone = 'Chile/EasterIsland'
--Display the SYSDATE, CURRENT_DATE, CURRENT_TIMESTAMP, and LOCALTIMESTAMP for this session
select sysdate, current_date, current_timestamp, localtimestamp
from dual
--Alter the session to set the NLS_DATE_FORMAT to  DD-MON-YYYY.
alter session set NLS_DATE_FORMAT = 'DD-MON-YYYY'

--26 Write a query to display the last names, month of the date of join, and hire date of those employees who have joined in the month of January, irrespective of the year of join.
select last_name, to_number(to_char(hire_date, 'mm'), '99') as month, to_char(hire_date, 'dd-month-yyyy')
from employees
where to_char(hire_date, 'mm') = 1

--27 Write a query to display the following for those departments whose department ID is greater than 80:
--  The total salary for every job within a department
--  The total salary  
--  The total salary for those cities in which the departments are located
--  The total salary for every job, irrespective of the department
--  The total salary for every department irrespective of the city
--  The total salary of the cities in which the departments are located
--  Total salary for the departments, irrespective of  job titles and cities
select l.city, d.department_name as dname, e.job_id, sum(e.salary)
from departments d, locations l, employees e
where d.department_id > 80
and   d.location_id = l.location_id (+)
and   e.department_id = d.department_id (+)
group by cube(d.department_name, l.city, e.job_id)

--28 Write a query to display the following groupings:
--  Department ID, Job ID
--  Job ID, Manager ID
--The query should calculate the maximum and minimum salaries for each of these groups
select e.department_id, e.job_id as job, e.manager_id, max(e.salary), min(e.salary)
from employees e, departments d
where e.department_id = d.department_id (+)
group by grouping sets((e.department_id, e.job_id), (e.job_id, e.manager_id))

--29 Write a query to display the top three earners in the EMPLOYEES table. Display their last names and salaries.
select last_name, salary
from employees
where rownum <= 3
order by salary desc

--30 Write a query to display the employee ID and last names of the employees who work in the state of California.
--Hint: Use scalar subqueries.
select e.employee_id, e.last_name
from employees e
where exists (select *
              from departments d
              where d.department_id = e.department_id
              and   exists (
                  select * from locations l
                  where l.location_id = d.location_id
                  and   l.city = 'California'))

select e.employee_id, e.last_name
from employees e, departments d, locations l
where e.department_id  = d.department_id (+)
and   d.location_id = l.location_id (+)
and   l.city = 'California'

--31 Write a query to delete the oldest JOB_HISTORY row of  an employee by looking up the JOB_HISTORY table for the MIN(START_DATE) for the employee. Delete the records of only those employees who have changed at least two jobs. If your query executes correctly, you will get the feedback:
--Hint: Use a correlated DELETE command.
delete
from job_history j
where j.start_date = (
              select min(start_date)
              from job_history
              where employee_id = j.employee_id
              having count(*) >= 2)

--32 Roll back the transaction
rollback

--33 Write a query to display the job IDs of those jobs whose maximum salary is above half the maximum salary in the whole company. Use the WITH clause to write this query. Name the query MAX_SAL_CALC.
with
MAX_SAL_CALC as (
  select max(salary) as max_sal
  from employees
)
select j.job_title, j.max_salary
from MAX_SAL_CALC cm, jobs j
where j.max_salary * 2 > cm.max_sal

--34 Write a SQL statement to display employee number, last name, start date, and salary, showing:
--De Haan’s direct reports
select manager.employee_id, manager.last_name, manager.hire_date, manager.salary
from employees e, employees manager
where e.last_name = 'De Haan'
and   e.manager_id = manager.employee_id
--The organization tree under De Haan (employee number 102)
select employee_id, last_name, hire_date, salary
from employees
start with employee_id = 102
connect by prior employee_id = manager_id

--35 Write a hierarchical query to display the employee number, manager number, and employee last name for all employees who are two levels below employee De Haan (employee number 102). Also display the level of the employee.
select employee_id, manager_id, level, last_name
from employees
where level = 3
start with employee_id = 102
connect by prior employee_id = manager_id

--36 Produce a hierarchical report to display the employee number, manager number, the LEVEL pseudocolumn, and employee last name. For every row in the EMPLOYEES table, you should print a tree structure showing the employee, the employee’s manager, then the manager’s manager, and so on. Use indentations for the NAME column
select employee_id, manager_id, level, lpad(last_name, level*2-2 + length(last_name), '_') as last_name
from employees
start with last_name = 'KingA'
connect by prior employee_id = manager_id

--37 Write a query to do the following:
-- Retrieve the details of the employee ID, hire date, salary, and manager ID of those employees whose employee ID is more than or equal to 200 from the EMPLOYEES table.
-- If the salary is less than $5,000, insert the details of employee ID and salary into the SPECIAL_SAL table.
-- Insert the details of employee ID, hire date, and salary into the SAL_HISTORY table.
-- Insert the details of employee ID, manager ID, and salary into the MGR_HISTORY table
insert first
when salary < 5000 then
  into special_sal values(employee_id, hire_date, salary, manager_id)
else
  into sal_history values (employee_id, hire_date, salary)
  into mgr_history values (employee_id, manager_id, salary)
select employee_id, hire_date, salary, manager_id
from employees
where employee_id >= 200

--38 Query the SPECIAL_SAL, SAL_HISTORY and the MGR_HISTORY tables to view the inserted records.
select *
from special_sal
select *
from sal_history
select *
from mgr_history

--39 Create the LOCATIONS_NAMED_INDEX table based on the following table instance chart. Name the index for the PRIMARY KEY column as LOCATIONS_PK_IDX.
create table LOCATIONS_NAMED_INDEX(
       Deptno number(4) primary key, Dname varchar2(30)
)
select index_name
from user_indexes
where table_name = 'LOCATIONS_NAMED_INDEX'
ALTER INDEX () RENAME TO LOCATIONS_PK_IDX

--40 Query the USER_INDEXES table to display the INDEX_NAME for the LOCATIONS_NAMED_INDEX table
select index_name
from user_indexes
where table_name = 'LOCATIONS_NAMED_INDEX'

--41 Write a SQL script file to drop all objects (tables, views, indexes, sequences, synonyms, and  so on) that you own.
--Drop specified objects
select 'DROP TABLE '||TABLE_NAME||';'as txt from user_tables
UNION select 'DROP INDEX '||INDEX_NAME||';'as txt FROM  user_indexes;
--Drop all objects
select 'DROP '|| uo.OBJECT_TYPE||' '||uo.OBJECT_NAME||';'as txt from user_objects uo
```