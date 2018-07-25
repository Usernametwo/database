# 内置plsql工具包

```sql
-- procedure
CREATE OR REPLACE PROCEDURE delete_all_rows(p_tab_name IN VARCHAR2
                                           ,p_rows_del OUT NUMBER) IS
    cursor_name INTEGER;
BEGIN
    cursor_name := dbms_sql.open_cursor;
    dbms_sql.parse(cursor_name, 'delete from ' || p_tab_name, dbms_sql.native);
    p_rows_del := dbms_sql.execute(cursor_name);
    dbms_sql.close_cursor(cursor_name);
END delete_all_rows;

CREATE OR REPLACE PROCEDURE delete_all_rows(p_tab_name IN VARCHAR2
                                           ,p_rows_del OUT NUMBER) IS
BEGIN
    EXECUTE IMMEDIATE 'delete from ' || p_tab_name;
    p_rows_del := SQL % ROWCOUNT;
END delete_all_rows;

-- SQL window
declare
  rows_ number;
begin
  delete_all_rows(p_tab_name => 'B', p_rows_del => rows_);
  dbms_output.put_line(rows_);
end;
```

dynamic SQL : Oracle 内置工具包 DBMS_SQL 来执行，也可以使用 EXECUTE IMMEDIATE 语句来执行

```sql
--在程序中执行编译命令
dbms_ddl.alter_compile('PROCEDURE', 'STUDENT1', 'delete_all_rows');
--在程序中执行数据收集命令，用来优化
dbms_ddl.analyze_object('TABLE', 'STUDENT1', 'JOBS', 'COMPUTE');
```

在程序中执行DDL : DBMS_DDL 包

```sql
DECLARE
    jobno NUMBER;
BEGIN
    dbms_job.submit(job       => jobno
                   ,what      => 'dbms_ddl.analyze_object(''TABLE'', ''STUDENT1'', ''JOBS'', ''COMPUTE'');'
                   ,next_date => trunc(SYSDATE + 1)
                   ,interval  => 'trunc(SYSDATE + 1)');
    dbms_output.put_line('job no:' || jobno);
END;

BEGIN
    dbms_job.change(46, NULL, trunc(SYSDATE + 1) + 6 / 24, 'SYSDATE+4/24');
END;

SELECT *
FROM   dba_jobs
```

Oracle定义JOB来定期执行某个程序

```sql
--procedure
CREATE OR REPLACE PROCEDURE sal_status(p_filedir  IN VARCHAR2
                                      ,p_filename IN VARCHAR2) IS
    v_filehandle utl_file.file_type;
    CURSOR emp_info IS
        SELECT last_name
              ,salary
              ,department_id
        FROM   employees
        ORDER  BY department_id;
    v_newdeptno employees.department_id % TYPE;
    v_olddeptno employees.department_id % TYPE := 0;
BEGIN
    v_filehandle := utl_file.fopen(p_filedir, p_filename, 'w');
    utl_file.putf(v_filehandle, 'SALARY REPORT: GENERATED ON %s\n', SYSDATE);
    utl_file.new_line(v_filehandle);
    FOR v_emp_rec IN emp_info LOOP
        v_newdeptno := v_emp_rec.department_id;
        IF v_newdeptno <> v_olddeptno THEN
            utl_file.putf(v_filehandle, 'DEPARTMENT: %s\n', v_emp_rec.department_id);
        END IF;
        utl_file.putf(v_filehandle, ' EMPLOYEE: %s EARNS: %s\n', v_emp_rec.last_name, v_emp_rec.salary);
        v_olddeptno := v_newdeptno;
    END LOOP;
    utl_file.put_line(v_filehandle, '*** END OF REPORT ***');
    utl_file.fclose(v_filehandle);
EXCEPTION
    WHEN utl_file.invalid_filehandle THEN
        raise_application_error(-20001, 'invalid file.');
    WHEN utl_file.write_error THEN
        raise_application_error(-20002, 'unable to write to file.');
END sal_status;

--UTL_FILE
--在服务器上面创建文件夹
--使用system用户将已经存在的路径创建为Oracle的对象  目录名称  绝对路径
create directory hand_file_test as 'D:\HandFileTest';
--赋予所有用户读写该目录的权限
grant read,write on directory hand_file_test to public;

BEGIN
    sal_status(p_filedir => 'HAND_FILE_TEST', p_filename => 'HandFileTest1.txt');
END;
```

UTL_FILE : 读写外部文件