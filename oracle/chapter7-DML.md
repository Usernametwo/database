# DML语句

```sql
insert into departments(department_id, department_name)
values (31, 'ORA technology center')

insert into departments
values (31, 'ORA technology center', NULL, NULL)

insert into sales_reps(id, name, salary, commission_pct)
select employee_id, last_name, salary, commission_pct
from employees
where job_id like '%REP%'

insert into
(select employee_id, last_name,
email, hire_date, job_id, salary, department_id
from employees)
values (9999, 'Taylor', 'DTAYLOR', to_date('07-06-1999', 'dd-mm-yyyy'),
'ST_CLERK', 5000, 50)

insert into
(select employee_id, last_name,
email, hire_date, job_id, salary, department_id
from employees
where department_id = 50 with check option)
values (99998, 'Taylor', 'JMSITH', to_date('07-06-1999', 'dd-mm-yyyy'),
'ST_CLERK', 5000, 50)
```

INSERT语句：

1. 写出表名和列名：在列中，对于不允许为NULL的列，必须写出来，对于允许为NULL的列，可以不写出来；在value中，对于列中未写出来的列，默认赋NULL值
2. 仅写出表名：在value中必须对应写出每个列的值，即使允许为NULL的列，也要在value中写NULL
3. 从另一个表中copy一行：在这种方式下无需使用values关键字
4. 使用子查询作为插入目标 ：with check option可以检查要插入的内容是否符合目标子查询的where条件

```sql
update employees set department_id = 70
where employee_id = 113

update employees set job_id =
(select job_id
from employees
where employee_id = 205),
salary =
(select salary
from employees
where employee_id = 205)
where employee_id = 114
```

UPDATE语句：

1. 更新符合条件的行中某些列为具体的值
2. 使用子查询的结果作为更新后的值

当存在约束的时候，某些更新可能会失败

```sql
delete from departments
where department_name = 'Finance'

delete from copy_emp
truncate table copy_emp
```

DELETE语句：

1. 删除某些符合条件的记录
2. 删除一张表中的所有记录：使用TRUNCATE TABLE之后无法回滚

当存在某些约束的时候删除操作可能会失败

```sql
merge into copy_emp c
using employees e
on(c.employee_id = e.employee_id)
when matched then
    update set
        c.first_name = e.first_name
        c.last_name = e.last_name
        ...
        c.department_id = e.department_id
when not matched then
    insert values(e.employee_id, e.first_name, e.last_name ...)
```

MERGE语句 : 比较整合语句