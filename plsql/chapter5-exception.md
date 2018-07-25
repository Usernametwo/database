# plsql中的异常处理

plsql中的异常一般有两种：

1. Oracle内部错误抛出的异常
2. 程序员现实抛出的异常

有些常见异常 Oracle 已经定义好了,使用时无需事先声明：

1. NO_DATA_FOUND
2. TOO_MANY_ROWS
3. INVALID_CURSOR
4. ZERO_DIVIDE
5. DUP_VAL_ON_INDEX

```sql
begin
...
exception
  when no_data_found then
    statement1;
  when too_many_rows then
    statement2;
  when others then
    rollback;
    v_error_code := SQLCODE;
    v_error_message := SQLERRM;
    insert into errors
    value(v_error_code, v_error_message);
end;
```

Oracle 提供了两个内置函数 SQLCODE 和 SQLERRM 分别用来返回Oracle 错误号和错误描述

```sql
--DEFINE p_deptno = 10;  sqlplus
DECLARE
    p_deptno NUMBER := 10;
    e_emps_remaining EXCEPTION;
    PRAGMA EXCEPTION_INIT(e_emps_remaining, -2292);
BEGIN
    DELETE FROM departments
    WHERE  department_id = &p_deptno;
    COMMIT;
EXCEPTION
    WHEN e_emps_remaining THEN
        dbms_output.put_line('Cannot remove dept ' || to_char(&p_deptno) || '. Employees exist. ');
END;
```

处理非预定义的Oracle错误：此类错误属于Oracle错误，有编号，但无错误名称定义，使用时需要先声明，并
进行错误初始化

```sql
DECLARE
    e_invalid_department EXCEPTION;
BEGIN
    UPDATE departments
    SET    department_name = &p_department_desc
    WHERE  department_id = &p_department_number;
    IF SQL%NOTFOUND THEN
        RAISE e_invalid_department;
    END IF;
    COMMIT;
EXCEPTION
    WHEN e_invalid_department THEN
        dbms_output.put_line('No such department id.');
END;
```

处理用户自定义的错误: 这种错误一般是程序员根据具体的业务逻辑定义的应用类错误，需要先声明后使用

```sql
    BEGIN
...
DELETE FROM employees
WHERE  manager_id = v_mgr;
IF SQL%NOTFOUND THEN
    raise_application_error(-20202, 'This is not a valid manager');
END IF;

EXCEPTION
    WHEN no_data_found THEN
        raise_application_error(-20201, 'Manager is not a valid employee.');
```

RAISE_APPLICATION_ERROR() 函数 : 无需预定义错误

如果在循环中出现异常，接下来的循环都不会执行，如果还想继续执行下去需要写一个匿名块或者procedure 或者函数来捕获这个异常，之后将控制权交给调用者则可以继续循环。