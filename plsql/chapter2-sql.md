# plsql sql语句

```sql
-- set serveroutput on;  sqlplus
declare
    v_hire_date employees.hire_date % type;
    v_salary employees.salary % type;
begin
    select hire_date, salary
    into v_hire_date, v_salary
    from employees
    where employee_id = 100;
    dbms_output.put_line(v_hire_date);
    dbms_output.put_line(v_salary);
end;

-- set serveroutput on;  sqlplus
declare
    v_sum_sal number(10, 2);
    v_deptno number not null := 60;
begin
    select sum(salary)
    into v_sum_sal
    from employees
    where department_id = v_deptno;
    dbms_output.put_line('the sum salary is '||to_char(v_sum_sal));
end;
```

SELECT INTO 语句

```sql
declare
  l_person_name varchar2(140);
  l_age number;
  
  type person_rec_type is record(
  person_name varchar(140),
  age number);
  type person_rec_tbl is table of person_rec_type index by binary_integer;
  person_tbl person_rec_tbl;
begin
  insert into cux_cursor_test values(103, 'juzhihua', 25)
  returning person_name, age into l_person_name, l_age;
  dbms_output.put_line('person_name : ' || l_person_name);
  
  delete cux_cursor_test
  returning person_name, age bulk collect into person_tbl;
  for i in 1..person_tbl.count loop
    dbms_output.put_line('person_name : ' || person_tbl(i).person_name);
  end loop;
end;
```

RETURNING INTO 语句 ：可以返回 DML 语句中影响到的行中的所有列的值

```sql
declare
  v_sal_increse employees.salary%type := 800;
begin
  update employees
  set salary = salary + v_sal_increse
  where job_id = 'ST_CLERK';
end;

declare
    v_empno employees.employee_id%type := 100;
begin
    merge into copy_emp c
        using employees e
        on(e.employee_id = v_empno)
        when matched then
            update set
                c.first_name = e.first_name,
                c.last_name = e.last_name,
                c.email = e.email,
                ...
        when not matched then
            insert values(e.employee_id, e.first_name, e.last_name, ...);
end;
```

insert, update, delete, merge 语句，均可以在plsql中执行

```sql
begin
    -- no
    if null then
      dbms_output.put_line('yes');
    else
      dbms_output.put_line('no');
    end if;
    -- no
    if null and true then
      dbms_output.put_line('yes');
    else
      dbms_output.put_line('no');
    end if;
    -- no
    if null and false then
      dbms_output.put_line('yes');
    else
      dbms_output.put_line('no');
    end if;
end;
```

关于 null 的判断:

1. null and true (true and null) : null
2. null and false (false and null) : false
3. null or true (true or null) : true
4. null or false (false or null) : null
5. not null : null
6. null and null : null
7. null or null : null

```sql
if condition then
    statements;
elsif confition then
    statements;
...
end if;

case selector
    when expr1 then result1
    when expr2 then result2
    ...
    else result3;
end case;
```

plsql的判断语句

```sql
declare
    v_country_id locations.country_id % type := 'CA';
    v_location_id locations.location_id % type;
    v_city locations.city % type := 'Montreal';
begin
    select max(location_id)
    into v_location_id
    from locations
    where country_id = v_country_id;
    for i in 1..3 loop
        insert into locations(location_id, city, country_id)
        values((v_location_id + i), v_city, v_country_id);
    end loop;
end;

loop
    statement1;
    ...
    exit [when condition];
end loop;

while condition loop
    statement1;
    statement2;
    ...
end loop;

for counter in [reverse] lower_bound..upper_bound loop
    statement1;
    statement2;
    ...
end loop;
```

循环

```sql
begin
    <<outer_loop>>
    loop
        v_counter := v_counter + 1;
        exit when v_counter > 10;
        <<inner_loop>>
        loop
            ...
            -- leave both loops
            exit outer_loop when total_done = 'Yes';
            -- leave inner loop
            exit when inner_done = 'Yes';
            ...
        end loop inner_loop;
    end loop outer_loop;
end;
```

嵌套循环和label
