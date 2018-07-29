# plsql中的游标

游标是一个私有的SQL内存工作区，游标的作用是用于临时在内存中存储数据库中提取的数据块，否则频繁的读取磁盘势必会效率低下，Oracle有两种游标，分别为显示游标和隐式游标

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

隐式游标：当plsql在处理 DML 操作的时候都会隐式的产生一个游标

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

```sql
declare
  --定义强类型的游标变量
  type ref_person_cur1 is ref cursor return cux_cursor_test % rowtype;
  l_ref_person_cur1 ref_person_cur1;
  
  --弱类型的游标变量
  type ref_person_cur2 is ref cursor;
  l_ref_person_cur2 ref_person_cur2;
  
  --定义变量接受游标返回结果
  l_row_rec cux_cursor_test % rowtype;
  type person_rec_type is record(person_id number, person_name varchar2(140), age number);
  type person_tbl_type is table of person_rec_type index by binary_integer;
  l_person_tbl person_tbl_type;
  l_dync_sql varchar(100) := null;
  
begin
  dbms_output.put_line('强类型游标变量');
  open l_ref_person_cur1 for select * from cux_cursor_test;
  loop
    --获取强类型游标变量的结果必须用指定的返回类型接受
    fetch l_ref_person_cur1 into l_row_rec;
    exit when l_ref_person_cur1 % notfound;
    dbms_output.put_line('Person name : ' || l_row_rec.person_name);
  end loop;
  close l_ref_person_cur1;
  
  
  dbms_output.put_line('弱类型游标变量');
  l_dync_sql := 'select * from cux_cursor_test where 1 = 1 and person_name = :1';
  --弱类型游标变量绑定动态SQL
  open l_ref_person_cur2 for l_dync_sql using 'gb';
  --一次接收多行结果集
  fetch l_ref_person_cur2 bulk collect into l_person_tbl;
  for i in 1..l_person_tbl.count loop
    dbms_output.put_line('person name : ' || l_person_tbl(i).person_name);
  end loop;
  close l_ref_person_cur2;
  
end;
```

自定义游标变量：

1. 定义 REF CURSOR 类型 : TYPE ref_type_name IS REF CURSOR[RETURN return type] (分为强类型和弱类型
2. 定义游标变量 ：variable ref_type_name
3. 定义变量接收返回结果
4. 打开游标接收数据，关闭游标