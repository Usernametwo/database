# 数据库对象-序列，索引，同义词

```sql
create sequence dept_deptid_seq
increment by 1
start with 1

insert into departments(department_id,
department_name, location_id)
values (dept_deptid_seq.NEXTVAL,
'Support', 2500)

select dept_deptid_seq.CURRVAL
from dual

drop sequence dept_deptid_seq
```

序列

```sql
create index emp_last_name_index
on employees(last_name)

select *
from departments
where upper(department_name) = 'SALES'
```

普通索引 ：对查询没有什么帮助，需要建立函数索引

```sql
create index upper_dept_name_index
on departments(upper(department_name))

select *
from departments
where upper(department_name) = 'SALES'

begin
  for i in 1000 .. 9999 loop
    insert into departments values(i, 'Test_department_name', 200, 1700);
  end loop;
end;

create index dept_name_idx on departments(department_name);
drop index dept_name_idx

create index upper_dept_name_idx
on departments(upper(department_name))
```

函数索引

```sql
select * from employees@orcl

create synonym C for employees@orcl

select * from C

drop synonym C
```

SYNONYM : 同义词