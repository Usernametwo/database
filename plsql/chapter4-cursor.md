# plsql中的游标

游标是一个私有的SQL工作区域，Oracle有两种游标，分别为显示游标和隐式游标

```sql
-- Command Window
variable rows_deleted varchar2(30);
declare
    v_employee_id employees.employee_id % type := 176;
begin
    delete from employees
    where employee_id = v_employee_id;
    :rows_deleted := (SQL % rowcount || ' rows deleted.');
end;

declare
    rows_deleted varchar2(30);
    v_employee_id employees.employee_id % type := 176;
begin
    delete from employees
    where employee_id = v_employee_id;
    rows_deleted := (SQL % rowcount || ' rows deleted.');
    dbms_output.put_line(rows_deleted);
end;
```

隐式游标：

1. SQL % ROWCOUNT : 受最近的SQL语句影响的行数
2. SQL % FOUND: 受最近的SQL语句是否未影响了一行以上的数据
3. SQL % NOTFOUND
4. SQL % ISOPEN : 对于隐式游标而言一直返回false

```sql
DECLARE
    v_empno employees.employee_id%TYPE;
    v_ename employees.last_name%TYPE;
    CURSOR emp_cursor IS
        SELECT employee_id, last_name
        FROM   employees;
BEGIN
    OPEN emp_cursor;
    LOOP
        FETCH emp_cursor
            INTO v_empno, v_ename;
        EXIT WHEN emp_cursor % ROWCOUNT > 10 OR emp_cursor % NOTFOUND;
        dbms_output.put_line(to_char(v_empno) || ' ' || v_ename);
    END LOOP;
    CLOSE emp_cursor;
END;

begin
    for emp_record in (select last_name, department_id from employees) loop
        if emp_record.department_id = 80 then
            dbms_output.put_line(emp_record.last_name);
        end if;
    end loop;
end;

declare
    cursor emp_cursor is
        select last_name, department_id
        from employees;
begin
    for emp_record in emp_cursor loop
        if emp_record.department_id = 80 then
            dbms_output.put_line(emp_record.last_name);
        end if;
    end loop;
end;

declare
    v_empno employees.employee_id%TYPE;
    v_ename employees.last_name%TYPE;
    cursor emp_cursor(p_deptno number, p_job varchar2) is
        select employee_id, last_name
        from employees
        where department_id = p_deptno
        and   job_id = p_job;
begin
    open emp_cursor(80, 'SA_REP');
    loop
        fetch emp_cursor into v_empno, v_ename;
        EXIT when emp_cursor % NOTFOUND;
        dbms_output.put_line(v_ename);
    end loop;
    close emp_cursor;
end;
```

显式游标：对于返回多行结果的SQL语句的返回结果，可使用显示游标独立的处理其中的每一行数据

```sql
DECLARE
    CURSOR emp_cursor IS
        SELECT employee_id
              ,last_name
              ,department_name
        FROM   employees
              ,departments
        WHERE  employees.department_id = departments.department_id
        AND    employees.department_id = 80
        FOR    UPDATE OF salary NOWAIT;
BEGIN
    FOR emp_data IN emp_cursor LOOP
        UPDATE employees e
        SET    e.salary = 29999
        WHERE  e.employee_id = emp_data.employee_id;
    END LOOP;
END;
```

FOR UPDATE [COLUMN] NOWAIT : 当我们打开游标执行更新或者删除的时候，FOR UPDATE [COLUMN] NOWAIT 可以用来锁定记录，当另一个人试图更新本记录的时候会报错，否则会一直等待

```sql
DECLARE
    CURSOR sal_cursor IS
        SELECT e.department_id
              ,employee_id
              ,last_name
              ,salary
        FROM   employees   e
              ,departments d
        WHERE  d.department_id = e.department_id
        AND    d.department_id = 60
        FOR    UPDATE OF salary NOWAIT;
BEGIN
    FOR emp_record IN sal_cursor LOOP
        IF emp_record.salary < 5000 THEN
            UPDATE employees
            SET    salary = emp_record.salary * 1.10
            WHERE  CURRENT OF sal_cursor;
        END IF;
    END LOOP;
END;
```

WHERE  CURRENT OF sal_cursor : 指向游标的当前记录

```sql
DECLARE
    TYPE row_list IS TABLE OF employees % ROWTYPE;
    row_s row_list;
    CURSOR emp_cursor IS
        SELECT *
        FROM   employees;
BEGIN
    OPEN emp_cursor;
    LOOP
        FETCH emp_cursor BULK COLLECT
            INTO row_s LIMIT 10;
        FOR i IN 1 .. row_s.count LOOP
            dbms_output.put_line(row_s(i).employee_id || ' name : ' || row_s(i).last_name);
        END LOOP;
        EXIT WHEN emp_cursor%NOTFOUND;
    END LOOP;
    CLOSE emp_cursor;
END;
```

批量读取：bulk collect to ** limit number : 可以适当提高效率