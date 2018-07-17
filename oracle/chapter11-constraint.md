# 数据库对象-约束

Oracle数据库使用约束来阻止对数据库表中数据的不合法增删改动作

常用的约束有如下几种:

1. NOT NULL
2. UNIQUE : 唯一性约束
3. PRIMARY KEY
4. FOREIGN KEY
5. CHECK : 自定义约束

```sql
select * from all_constraints

create table A(
    employee_id number(6),
    first_name varchar(20)
    constraint first_name_nn
    not null,
    job_id varchar(10) not null,
    constraint A_emp_pk primary key (employee_id)
)

alter table A add constraint first_name_not_null unique (first_name)

create table employee(
    employee_id number(6),
    last_name varchar2(25),
    salary number(2)
    constraint emp_salary_min
    check (salary > 0),
    department_id number(4),
    constraint emp_dept_fk foreign key (department_id)
    references departments(department_id)
)
```

外键的约束类型：

1. REFERENCES : 表示列中的值必须在父表中存在
2. ON DELETE CASCADE : 当父表记录删除的时候自动删除子表中的相应记录
3. ON DELETE SET NULL : 当父表删除时自动把子表中的相应记录设为NULL

```sql
alter table employees
drop constraint emp_salary_min

alter table departments
drop primary key cascade
```

删除约束

```sql
alter table employees
disable constraint emp_salary_min

alter employees
enable constraint emp_salary_min
```

失效/生效约束

```sql
select constraint_name, constraint_type, search_condition
from user_constraints
where table_name = 'EMPLOYEES'
```

查看表的约束