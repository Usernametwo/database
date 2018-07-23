# plsql中的复杂自定义数据类型

plsql中常用的自定义类型有：记录类型和plsql内存表类型

```sql
type emp_record_type is record
(last_name varchar2(25),
job_id varchar2(10),
salary number(8, 2));
emp_record emp_record_type
```

记录类型

```sql
declare
    emp_rec employees % rowtype;
begin
    select * into emp_rec
    from employees
    where employee_id = &employee_number;
    dbms_output.put_line(emp_rec.last_name);
end;
```

% rowtype 表示某张表的记录类型
% type 表示某张表某一列的记录类型

```sql
begin
  type ename_table_type is table of employee.last_name % type index by binary_integer;
  ename_table ename_table_type;
end;

declare
  type ename_table_type is table of employees.last_name % type index by binary_integer;
  type hiredate_table_type is table of date index by binary_integer;
  ename_table    ename_table_type;
  hiredate_table hiredate_table_type;
begin
  ename_table(1) := 'CAMERON';
  hiredate_table(8) := sysdate + 7;
  if ename_table.exists(1) then
    dbms_output.put_line(hiredate_table(8));
  end if;
end;


declare
    type emp_table_type is table of employees % rowtype index by binary_integer;
    my_emp_table emp_table_type;
    v_count number(3) := 104;
begin
    for i in 100..v_count
    loop
        select * into my_emp_table(i) from employees
        where employee_id = 1;
    end loop;
    for i in my_emp_table.first..my_emp_table.last
    loop
        dbms_output.put_line(my_emp_table(i).last_name);
    end loop;
end;

declare
    type emp_table_type is table of employees % rowtype index by binary_integer;
    my_emp_table emp_table_type;
    v_count number(3) := 104;
begin
    for i in 100..v_count
    loop
        select * into my_emp_table(i) from employees
        where employee_id = 100;
    end loop;
    for i in my_emp_table.first..my_emp_table.last
    loop
        dbms_output.put_line(my_emp_table(i).last_name);
    end loop;
end;
```

plsql内存表(数组)