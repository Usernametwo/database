# plsql中的大对象操作

LOB用来存储非结构化的大数据，比如大文本，图片，电影等

Oracle数据库里面的LOB有四种类型:

1. CLOB：字符大对象，存储在数据库内部
2. NCLOB：多字节字符大对象，存储在数据库内部
3. BLOB：二进制大对象：存储在数据库内部
4. BFILE：二进制文件，存储在数据库外部

```sql
-- procedure
CREATE OR REPLACE PROCEDURE load_emp_bfile(p_file_loc IN VARCHAR2) IS
    v_file     BFILE;
    v_filename VARCHAR2(16);
    CURSOR emp_cursor IS
        SELECT last_name
        FROM   employees
        WHERE  department_id = 60
        FOR    UPDATE;
BEGIN
    FOR emp_record IN emp_cursor LOOP
        v_filename := emp_record.last_name || '.bmp';
        v_file     := bfilename(p_file_loc, v_filename);
        dbms_lob.fileopen(v_file);
        UPDATE employees
        SET    emp_video = v_file
        WHERE  CURRENT OF emp_cursor;
        dbms_output.put_line('LOADED FILE: ' || v_filename || ' SIZE: ' || dbms_lob.getlength(v_file));
        dbms_lob.fileclose(v_file);
    END LOOP;
END load_emp_bfile;


-- 更改表结构
alter table employees  add (emp_video bfile);

--使用system用户将已经存在的路径创建为Oracle的对象  目录名称  绝对路径
create directory hand_file_test as 'D:\HandFileTest';
--赋予所有用户读写该目录的权限
grant read,write on directory hand_file_test to public;

begin
      load_emp_bfile('HAND_FILE_TEST');
end;


declare
b_file bfile;
begin
  b_file := bfilename('DIR_NAME', 'Lorentz.bmp');
  dbms_lob.fileopen(b_file);
  insert into B values(5, b_file);
  dbms_lob.fileclose(b_file);
end;
```

使用 bfile 的一般步骤：

1. 在操作系统上创建目录和文件
2. 在数据库表中添加bfile字段
3. 在Oracle数据库中创建 directory 对象
4. 授予权限给数据库用户
5. bfilename函数 ： bfile(directory, filename) return bfile
6. 使用 dbms_lob.fileopen(bfile) 读取bfile
7. 将 bfile 插入到表中
8. 使用 dbms_lob.fileclose(bfile) 关闭文件

```sql
-- PROCEDURE
CREATE OR REPLACE PROCEDURE emp_file_exists(p_file_loc IN VARCHAR2) IS
    v_file        BFILE;
    v_filename    VARCHAR2(16);
    v_file_exists BOOLEAN;
    CURSOR emp_cursor IS
        SELECT *
        FROM   employees
        WHERE  department_id IN (60, 70);
BEGIN
    FOR emp_record IN emp_cursor LOOP
        v_filename    := emp_record.last_name || '.bmp';
        v_file        := bfilename(p_file_loc, v_filename);
        v_file_exists := (dbms_lob.fileexists(v_file) = 1);
        IF v_file_exists THEN
            dbms_lob.fileopen(v_file);
            dbms_output.put_line(v_filename || ' exists');
            dbms_lob.fileclose(v_file);
        ELSE
            dbms_output.put_line(v_filename || ' does not exists');
        END IF;
    END LOOP;
END;

--SQL window
begin
  emp_file_exists('HAND_FILE_TEST');
end;

declare
b_file bfile;
begin
  b_file := bfilename('DIR_NAME', 'Lorentz.bmp');
  dbms_output.put_line(dbms_lob.fileexists(b_file));
  dbms_output.put_line(dbms_lob.fileexists(bfilename('DIR_NAME', '1.bmp')));
end;
```

使用 DBMS_LOB.FILEEXISTS(bfile) 测试文件是否存在，存在返回1，不存在返回0

```sql
alter table employees add(resume clob, picture blob)

delete employees where employee_id = 405;
INSERT INTO employees
    (employee_id
    ,first_name
    ,last_name
    ,email
    ,hire_date
    ,job_id
    ,salary
    ,resume
    ,picture)
VALUES
    (405
    ,'Marvin'
    ,'Ellis'
    ,'MELLIS'
    ,SYSDATE
    ,'AD_ASST'
    ,4000
    ,empty_clob()
    ,NULL);

UPDATE employees
SET    resume  = 'Date of Birth: 8 February 1951'
      ,picture = empty_blob()
WHERE  employee_id = 405

UPDATE employees
SET    resume = 'Date of Birth: 1 June 1956'
WHERE  employee_id = 170;
```

EMPTY_CLOB(), EMPTY_BLOB() 跟 NULL 是不同的概念，IS NULL 对这两种情况返回FALSE

```sql
DECLARE
    lobloc CLOB; -- serves as the LOB locator
    text   VARCHAR2(32767) := 'Resigned: 5 August 2000';
    amount NUMBER; -- amount to be written
    offset INTEGER; -- where to start writing
BEGIN
    SELECT resume
    INTO   lobloc
    FROM   employees
    WHERE  employee_id = 405
    FOR    UPDATE;
    offset := dbms_lob.getlength(lobloc) + 2;
    amount := length(text);
    dbms_lob.write(lobloc, amount, offset, text);
    text := ' Resigned: 30 September 2000';
    SELECT resume
    INTO   lobloc
    FROM   employees
    WHERE  employee_id = 170
    FOR    UPDATE;
    amount := length(text);
    dbms_lob.writeappend(lobloc, amount, text);
    COMMIT;
END;
```

数据库表中LOB列的增删改

```sql
SELECT dbms_lob.substr(resume, 5, 18)
      ,dbms_lob.instr(resume, ' = ')
      ,dbms_lob.instr(resume, 'June')
FROM   employees
WHERE  employee_id IN (170, 405);
```

LOB常用函数