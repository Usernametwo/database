# plsql 动态SQL

SQL语言分为五大类：

1. DDL : Create, Alter, Drop等
2. DQL : Select
3. DML : Insert, Update, Delete
4. DTL : Commit, Rollback事务提交与回滚语句
5. DCL : Grant, Revoke授予权限与回收权限语句

在plsql中，DML, DQL, DTL是可以直接使用静态SQL的，但是 DDL 和 DCL 就不能在plsql中直接使用，而需要动态SQL

实现动态SQL有两种方式，一种是本地动态SQL(EXECUTE IMMEDIATE),另一种是使用 DBMS_SQL 内部程序包来实现(很少使用，没有前者灵活)

```sql
DECLARE
    l_dync_sql VARCHAR2(100);
BEGIN
    l_dync_sql := 'create table cux_dync_test(id number, creation_date date)';
    EXECUTE IMMEDIATE l_dync_sql;
END;

DECLARE
    l_dync_sql    VARCHAR2(100);
    l_person_name VARCHAR2(140);
    l_age         NUMBER;
BEGIN
    l_dync_sql := 'select person_name, age from cux_cursor_test where person_id = :1';
    EXECUTE IMMEDIATE l_dync_sql
        INTO l_person_name, l_age
        USING 101;
    dbms_output.put_line('Person name: ' || l_person_name);
    dbms_output.put_line('Age: ' || l_age);
END;
```

本地动态SQL, USING 传参，如果动态 SQL 有单行的结果直接用 into 接收

```sql
declare
  type ref_person_cursor is ref cursor;
  l_person_ref ref_person_cursor;
  l_dync_sql varchar2(100);
  l_person_id number;
  l_person_name varchar2(20);
  l_age number;
begin
  l_dync_sql := 'select person_id, person_name, age from cux_cursor_test';
  open l_person_ref for l_dync_sql;
  loop
    fetch l_person_ref into l_person_id, l_person_name, l_age;
    exit when l_person_ref % notfound;
    dbms_output.put_line('Person id:' || l_person_id);
    dbms_output.put_line('Person name:' || l_person_name);
    dbms_output.put_line('age:' || l_age);
  end loop;
  close l_person_ref;
end;
```

使用游标来接收动态SQL的多行输出

```sql
declare
l_dync_sql varchar2(100);
l_one_number number := 10;
l_two_number number := 20;
l_sum number;
l_minus number;
begin
  l_dync_sql := 'begin get_sum(:1, :2, :3); end;';
  execute immediate l_dync_sql
  using l_one_number, l_two_number, out l_sum;
  dbms_output.put_line('Procedure out sum is : ' || l_sum);
  
  l_dync_sql := 'begin :1 := get_minus(:2, :3); end;';
  execute immediate l_dync_sql
  using out l_minus, l_one_number, l_two_number;
  dbms_output.put_line('Function out minus is : ' || l_minus);
end;
```

动态SQL执行 procedure 和 function

```sql
DECLARE
    l_dync_sql   VARCHAR2(100);
    l_one_number NUMBER := 10;
    l_two_number NUMBER := 20;
    l_sum        NUMBER;
    l_minus      NUMBER;
    l_cursor     NUMBER;
    l_row        NUMBER;
BEGIN
    --打开cursor处理sql语句
    l_cursor   := dbms_sql.open_cursor;
    l_dync_sql := 'Begin get_sum(:1, :2, :3); end;';
    dbms_sql.parse(l_cursor, l_dync_sql, dbms_sql.native);
    --按照顺序绑定变量，不分in或者out
    dbms_sql.bind_variable(l_cursor, ':1', l_one_number);
    dbms_sql.bind_variable(l_cursor, ':2', l_two_number);
    dbms_sql.bind_variable(l_cursor, ':3', l_sum);
    l_row := dbms_sql.execute(l_cursor);
    --获取绑定的变量值
    dbms_sql.variable_value(l_cursor, ':3', l_sum);
    IF dbms_sql.is_open(l_cursor) THEN
        dbms_sql.close_cursor(l_cursor);
    END IF;
    dbms_output.put_line('Procedure out sum is : ' || l_sum);

    l_cursor   := dbms_sql.open_cursor;
    l_dync_sql := 'begin :1 := get_minus(:2, :3); end;';
    dbms_sql.parse(l_cursor, l_dync_sql, dbms_sql.native);
    dbms_sql.bind_variable(l_cursor, ':1', l_minus);
    dbms_sql.bind_variable(l_cursor, ':2', l_one_number);
    dbms_sql.bind_variable(l_cursor, ':3', l_two_number);
    l_row := dbms_sql.execute(l_cursor);
    dbms_sql.variable_value(l_cursor, ':1', l_minus);
    dbms_output.put_line('Function out minus is : ' || l_minus);
    IF dbms_sql.is_open(l_cursor) THEN
        dbms_sql.close_cursor(l_cursor);
    END IF;
EXCEPTION
    WHEN OTHERS THEN
        dbms_output.put_line(SQLCODE || '.' || SQLERRM);
        RAISE;
END;
```

DBMS_SQL 步骤:

1. dbms_sql.parse(v_cursor, dynamic_sql, dbms_sql.native) : 如果 dynamic_sql 写错了，这一句会报错
2. dbms_sql.bind_variable(v_cursor, ':1', v_param1) : 绑定变量
3. l_row := dbms_sql.execute(v_cursor) : 执行动态sql
4. dbms_sql.variable_value(v_cursor, ':1', v_result) : 取出结果
5. dbms_sql.close_cursor(v_cursor) : 关闭游标

```sql
DECLARE
    l_dync_sql    VARCHAR2(100);
    l_person_id   NUMBER;
    l_person_name VARCHAR2(20);
    l_age         NUMBER;
    l_cursor      NUMBER;
    l_row         NUMBER;
BEGIN
    --打开cursor处理sql语句
    l_cursor   := dbms_sql.open_cursor;
    l_dync_sql := 'select person_id, person_name, age from cux_cursor_test';
    dbms_sql.parse(l_cursor, l_dync_sql, dbms_sql.native);

    --按照顺序绑定
    /*
        dbms_sql.bind_variable(l_cursor, ':1', l_one_number);
        dbms_sql.bind_variable(l_cursor, ':2', l_two_number);
        dbms_sql.bind_variable(l_cursor, ':3', l_sum);
    */

    --绑定列值
    dbms_sql.define_column(l_cursor, 1, l_person_id);
    dbms_sql.define_column(l_cursor, 2, l_person_name, 140);
    dbms_sql.define_column(l_cursor, 3, l_age);
    l_row := dbms_sql.execute(l_cursor);

    LOOP
        IF dbms_sql.fetch_rows(l_cursor) > 0 THEN
            --获取列值
            dbms_sql.column_value(l_cursor, 1, l_person_id);
            dbms_sql.column_value(l_cursor, 2, l_person_name);
            dbms_sql.column_value(l_cursor, 3, l_age);
            dbms_output.put_line('Person id : ' || l_person_id);
            dbms_output.put_line('Person name : ' || l_person_name);
            dbms_output.put_line('Person age : ' || l_age);
        ELSE
            EXIT;
        END IF;
    END LOOP;

    IF dbms_sql.is_open(l_cursor) THEN
        dbms_sql.close_cursor(l_cursor);
    END IF;

EXCEPTION
    WHEN OTHERS THEN
        IF dbms_sql.is_open(l_cursor) THEN
            dbms_sql.close_cursor(l_cursor);
        END IF;
        dbms_output.put_line(SQLCODE || '.' || SQLERRM);
END;
```

获取多行结果集

```sql
declare
  l_person_id_tb1 dbms_sql.number_table;
  l_dync_sql varchar2(100);
  l_cursor number;
  l_row number;
  l_person_id number;
  l_person_name varchar2(140);
  l_age number;
begin
  l_cursor := dbms_sql.open_cursor;
  l_person_id_tb1(1) := 101;
  l_person_id_tb1(2) := 102;
  l_dync_sql := 'Update cux_cursor_test set age = 30 where person_id = :1';
  dbms_sql.parse(l_cursor, l_dync_sql, dbms_sql.native);
  --传递数组中从 1 到 2 的值
  dbms_sql.bind_array(l_cursor, ':1', l_person_id_tb1, 1, 2);
  l_row := dbms_sql.execute(l_cursor);
  dbms_sql.close_cursor(l_cursor);
end;
```

绑定数组变量，但是绑定数组变量的操作都是用在DML语句（update，insert，delete）中，不能用在DQL中