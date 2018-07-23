# plsql variable

plsql 每一段程序都是由Block组成的。plsql的块包括三种：匿名块(declare)，存储过程(procedure)，函数(function)

```sql
declare
    v_hiredate employees.hire_date%type;
    v_deptno number(2) not null := 10;
    v_location varchar2(13) := 'Atlanta';
    c_comm constant number := 1400;
```

plsql的变量类型：

1. 系统内置的常规简单变量类型：比如数据库表的字段类型
2. 用户自定义变量
3. 引用类型
4. 大类型对象（LOB）：保存了一个指向大对象的地址

如果变量没有初始化默认为null
sqlplus变量:

```sql
variable g_salary number
begin
    select salary
    into :g_salary
    from employees
    where employee_id = 178;
end
/
print g_salary
```

可绑定变量是一种在宿主环境中定义的变量(plsql里面无法使用)

```sql
declare
    v_sal number(9,2) := &p_annual_salary;
begin
    v_sal   := v_sal/12;
    dbms_output.put_line('The monthly salary is ' || to_char(v_sal));
end;
```

&:用来表示输入

DBMS_OUTPUT.PUT_LINE():输出函数,oracle封装的存储过程

plsql注释:单行(--),多行(/**/)

SQL函数在PLSQL的过程语句中的使用：

1. 大多数可以使用，比如单行的数值与字符串函数，数据类型转换函数，日期函数，时间函数，求最大最小值
2. 有些函数不能使用，比如：Decode函数，分组函数(avg,min.max,count...),但是这些函数可以在sql(比如select)里面用

```sql
begin
  <<outer>>
  declare
    birthdate date;
  begin
    declare
      birthdate date;
    begin
      outer.birthdate := to_date('03-07-1976', 'dd-mm-yyyy');
      dbms_output.put_line(birthdate);
      dbms_output.put_line(outer.birthdate);
    end;
  end;
end;
```

块限定词(label)