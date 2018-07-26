# 数据库触发器

```sql
CREATE OR REPLACE TRIGGER secure_emp
    BEFORE INSERT ON employees
BEGIN
    IF (to_char(SYSDATE, 'DY') IN ('星期六', '星期日')) OR (to_char(SYSDATE, 'HH24:MI') NOT BETWEEN '08:00' AND '18:00') THEN
        raise_application_error(-20500, 'You may insert into EMPLOYEES table only during business hours.');
    END IF;
END;

--多事件触发器
CREATE OR REPLACE TRIGGER secure_emp
    BEFORE INSERT OR UPDATE ON employees
BEGIN
    IF (to_char(SYSDATE, 'dy') IN ('星期六', '星期日')) OR (to_char(SYSDATE, 'HH24') NOT BETWEEN '08' AND '18') THEN
        IF deleting THEN
            raise_application_error(-20502, 'you may delete from employees table only during business hours.');
        ELSIF inserting THEN
            dbms_output.put_line('you are inserting into employees in saturday or sunday');
        ELSIF updating('salary') THEN
            dbms_output.put_line('you are updating employees in saturday or sunday');
        ELSE
            dbms_output.put_line('you are updating');
        END IF;
    END IF;
END;

--Row级触发器
CREATE OR REPLACE TRIGGER restrict_salary
    BEFORE INSERT OR UPDATE OF salary ON employees
    FOR EACH ROW
BEGIN
    IF NOT (:new.job_id IN ('AD_PRES', 'AD_VP')) AND :new.salary > 15000 THEN
        raise_application_error(-20202, 'Employee');
    END IF;
END;

--row触发器里面的 new 和 old
CREATE OR REPLACE TRIGGER audit_emp_values
    AFTER DELETE OR INSERT OR UPDATE ON employees
    FOR EACH ROW
BEGIN
    INSERT INTO audit_emp_table
        (user_name
        ,TIMESTAMP
        ,id
        ,old_last_name
        ,new_last_name
        ,old_title
        ,new_title
        ,old_salary
        ,new_salary)
    VALUES
        (USER
        ,SYSDATE
        ,:old.employee_id
        ,:old.last_name
        ,:new.last_name
        ,:old.job_id
        ,:new.job_id
        ,:old.salary
        ,:new.salary);
END;

--when
CREATE OR REPLACE TRIGGER derive_commission_pct
    BEFORE INSERT OR UPDATE OF salary ON employees
    FOR EACH ROW
    WHEN (new.job_id = 'SA_REP')
BEGIN
    IF inserting THEN
    --如果是after无法对new赋值
        :new.commission_pct := 0;
    ELSIF :old.commission_pct IS NULL THEN
        :new.commission_pct := 0;
    ELSE
        :new.commission_pct := :old.commission_pct + 0.05;
    END IF;
END;

--生效失效
ALTER TRIGGER trigger_name DISABLE|ENABLE
ALTER TRIGGER table_name DISABLE|ENABLE ALL TRIGGER

--重新编译
ALTER TRIGGER trigger_name COMPILE

--删除
DROP TRIGGER trigger_name

--trigger 在 constraint 之前触发
UPDATE employees
set department_id = 999
where employee_id = 170

CREATE OR REPLACE TRIGGER constr_emp_trig
AFTER UPDATE ON employees
 FOR EACH ROW
BEGIN
 INSERT INTO departments
 VALUES (999, 'dept999', 140, 2400);
END;

-- trigger of database
CREATE OR REPLACE TRIGGER logon_trig
    AFTER logon ON SCHEMA
BEGIN
    INSERT INTO log_trig_table
        (user_id
        ,log_date
        ,action)
    VALUES
        (USER
        ,SYSDATE
        ,'Logging on');
END;

CREATE OR REPLACE TRIGGER logoff_trig
    BEFORE logoff ON SCHEMA
BEGIN
    INSERT INTO log_trig_table
        (user_id
        ,log_date
        ,action)
    VALUES
        (USER
        ,SYSDATE
        ,'Logging off');
END;

```

创建Trigger:

1. 时机：before，after，instead of
2. 事件：insert， update， delete
3. 对象：table，view
4. 类型：row，statement
5. 条件：满足特定 when 条件才能执行
6. 内容：plsql块

instead of：用 trigger 的内容替换事件本身的动作
Row级：SQL语句影响到的每一行都会引发trigger
statement级：一句SQL语句引发一次，不管他影响多少行（包括0行）

```sql
CREATE OR REPLACE TRIGGER check_salary
    BEFORE INSERT OR UPDATE OF salary, job_id ON employees
    FOR EACH ROW
    WHEN (new.job_id <> 'AD_PRES')
DECLARE
    v_minsalary employees.salary%TYPE;
    v_maxsalary employees.salary%TYPE;
BEGIN
    SELECT MIN(salary)
          ,MAX(salary)
    INTO   v_minsalary
          ,v_maxsalary
    FROM   employees
    WHERE  job_id = :new.job_id;
    IF :new.salary < v_minsalary OR :new.salary > v_maxsalary THEN
        raise_application_error(-20505, 'Out of range');
    END IF;
END;

UPDATE employees
SET    salary = 3400
WHERE  last_name = 'Stiles';

UPDATE employees
SET salary = 3400
WHERE last_name = 'Stiles';
```

冲突表：当某张表上的针对DML动作的Trigger需要访问到表自身的数据时，会发生冲突

尽量减少使用trigger