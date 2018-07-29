# plsql中的存储过程和函数

## 存储过程

```sql
CREATE OR REPLACE PROCEDURE raise_salary(p_id IN employees.employee_id % TYPE) IS
BEGIN
    UPDATE employees
    SET    salary = salary * 1.10
    WHERE  employee_id = p_id;
END raise_salary;

CREATE OR REPLACE PROCEDURE query_emp(p_id     IN employees.employee_id % TYPE
                                     ,p_name   OUT employees.last_name % TYPE
                                     ,p_salary OUT employees.salary % TYPE
                                     ,p_comm   OUT employees.commission_pct % TYPE) IS
BEGIN
    SELECT last_name
          ,salary
          ,commission_pct
    INTO   p_name
          ,p_salary
          ,p_comm
    FROM   employees
    WHERE  employee_id = p_id;
END query_emp;

CREATE OR REPLACE PROCEDURE format_phone(p_phone_no IN OUT VARCHAR) IS
BEGIN
    p_phone_no := '(' || sunstr(p_phone_no, 1, 3) || ')' || sunstr(p_phone_no, 4, 3) || '-' || substr(p_phone_no, 7);
END format_phone
```

创建存储过程

存储过程的参数模式：

1. IN ：传进来的可以是变量，常量，或者表达式，可以赋予默认值
2. OUT ：传进来的必须是个变量
3. IN OUT ： 传进来的必须是个变量

```sql
--program window -> procedure
CREATE OR REPLACE PROCEDURE add_dept(p_name IN departments.department_name % TYPE DEFAULT 'unknow'
                                    ,p_loc  IN departments.location_id % TYPE DEFAULT 1700) IS
BEGIN
    INSERT INTO departments
        (department_id
        ,department_name
        ,location_id)
    VALUES
        (departments_seq.nextval
        ,p_name
        ,p_loc);
END add_dept;

-- SQL window
BEGIN
    add_dept;
    add_dept('TRAINING', 2500);
    add_dept(p_loc => 2400, p_name => 'EDUCATION');
    add_dept(p_loc => 1200);
END;
```

参数传递方式 ( 按顺序传递 或者 使用=>符号传递 )

如果被调用procedure中有异常且未被处理，则会返回给调用者的Exception处理部分

```sql
deop procedure procedure_name
```

删除存储过程：再运行一遍procedure就好了，但是系统procedure慎用

***

## 函数

```sql
-- program window -> function
CREATE OR REPLACE FUNCTION get_sal(p_id IN employees.employee_id % TYPE) RETURN NUMBER IS
    v_salary employees.salary % TYPE := 0;
BEGIN
    SELECT salary
    INTO   v_salary
    FROM   employees
    WHERE  employee_id = p_id;
    RETURN v_salary;
END get_sal;
```

plsql函数,调用方式和procedure差不多

用户自定义函数的限制：

1. 只能用IN模式的参数（不能有OUT, IN OUT模式的参数
2. 只能接收和返回SQL类型的参数，不能接收和返回PLSQL中特有的参数（比如记录，内存表）
3. 在SQL中使用的函数，其函数体内部不能有DML语句
4. 在UPDATE/DELETE语句中调用的函数，其函数体内部不能有针对用一张表的查询语句
5. 在SQL中调用的函数，其函数体内部不能有事务结束语句（Commit， Rollback）
6. 对于无参的函数或者procedure，无需写括号（不管是调用还是声明亦或者是实现）

```sql
drop function function_name
```

删除储存函数

***

## procedure function 权限

函数和过程对数据访问的权限：定义者权限和调用者权限

```sql
--SQL window  user:student1
create table A(id number);
insert into A values (3);

--procedure window  user:student1
CREATE OR REPLACE PROCEDURE query_a(id OUT NUMBER) IS
BEGIN
    SELECT *
    INTO   id
    FROM   a
    WHERE  id = 3;
END query_a;

--SQL window   user:student1
DECLARE
    a_id NUMBER;
BEGIN
    query_a(id => a_id);
    dbms_output.put_line(a_id);
END;

--grant execute privileges to ora1  user:student1
grant execute
on    query_a
to ora1

--SQL window user:ora1
DECLARE
    a_id NUMBER;
BEGIN
    student1.query_a(id => a_id);
    dbms_output.put_line(a_id);
END;
```

定义者权限：在student1里面创建的 table 和 procedure，这个 procedure 访问这个 table ，之后将 procedure 的执行权限赋给 ora1 用户，  ora1 用户虽然没有 table 的 select 权限但是还是可以通过执行 procedure 来访问这个表，这是因为在访问 table 的时候是用的定义者权限。如果没有显示声明则 procedure 过程中都是使用的定义者权限

```sql
-- procedure window user:student1
CREATE OR REPLACE PROCEDURE query_a(id OUT NUMBER) AUTHID CURRENT_USER IS
BEGIN
    SELECT *
    INTO   id
    FROM   student1.a
    WHERE  id = 3;
END query_a;

--sql window user:student1
grant execute
on    query_a
to ora1

--sql window user:ora1
--error ORA-00942
DECLARE
    a_id NUMBER;
BEGIN
    student1.query_a(id => a_id);
    dbms_output.put_line(a_id);
END;

--sql window user:student1
grant select
on    a
to ora1

--sql window user:ora1
DECLARE
    a_id NUMBER;
BEGIN
    student1.query_a(id => a_id);
    dbms_output.put_line(a_id);
END;
```

调用者权限：AUTHID CURRENT_USER

```sql
-- procedure 值传递
CREATE OR REPLACE PROCEDURE copy_parameter(p_number IN OUT NUMBER) IS
    test_exception EXCEPTION;
BEGIN
    p_number := 10000;
    RAISE test_exception;
END copy_parameter;
--procedure 引用传递
CREATE OR REPLACE PROCEDURE nocopy_parameter(p_number IN OUT NOCOPY NUMBER) IS
    test_exception EXCEPTION;
BEGIN
    p_number := 9999;
    RAISE test_exception;
END nocopy_parameter;

--sql window
declare
  l_number number := 20000;
begin
  copy_parameter(p_number => l_number);
  exception
    when others then
      dbms_output.put_line('copy_parameter: ' || l_number);
      begin
        l_number := 20000;
        nocopy_parameter(p_number => l_number);
        exception
          when others then
            dbms_output.put_line('nocopy_parameter: ' || l_number);
      end;
end;
```

值传递和引用传递(nocopy)