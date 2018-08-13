# 单行函数

SQL的函数类型分为单行函数和多行函数

```sql
select employee_id, last_name, department_id
from employees
where lower(last_name) = 'higgins'

select initcap('fei zi yang')
from dual
```

大小写转换函数 :

1. LOWER()
2. UPPER()
3. INITCAP()

```sql
select employee_id, concat(first_name, last_name) name,
job_id, length(last_name),
instr(last_name, 'a') "Contains a?"
from employees
where substr(job_id, 4) = 'REP'

select length('中国') from dual
select lengthb('中国') from dual

select substr('上海汉德', 2, 2) from dual
select substrb('上海汉德', 2, 2) from dual
select substrb('上海汉德', 3, 2) from dual
```

字符串操作函数 :

1. CONCAT():字符串连接
2. SUBSTR():字符串截取，按照字符个数来取值
3. LENGTH():字符个数
4. INSTR('HelloWorld', 'W'):6
5. LPAD(24000, 10, '*'):*****24000
6. RPAD(24000, 10, '*'):24000*****
7. TRIM(' HelloWorld'):HelloWorld

    TRIM('Hello World'):Hello World
8. LENGTHB():字节数
9. SUBSTRB():按照字节来取值
10. REGEXP_LIKE
11. REGEXP_INSTR
12. ...

```sql
select round(45.923, 2), round(45.923, 0), round(45.923, -1)
from dual

select trunc(45.923, 2), trunc(45.923), trunc(45.923, -2)
from dual

select last_name, salary, mod(salary, 5000)
from employees
where job_id = 'SA_REP'
```

数字操作函数:

1. ROUND
2. TRUNC
3. MOD

```sql
select ROUND(to_date('16-9-1995','dd-mm-yyyy'), 'yy')
from dual
```

日期操作函数:

1. MONTHS_BETWEEN(to_date('01-9-1995','dd-mm-yyyy'), to_date('11-1-1994','dd-mm-yyyy')):19.6774193548387
2. ADD_MONTHS(to_date('11-1-1994', 'dd-mm-yyyy'), 6):1994-07-11
3. NEXT_DAY(to_date('01-9-1995','dd-mm-yyyy'), 1):1995-09-03
4. LAST_DAY(to_date('01-9-1995','dd-mm-yyyy')):1995-09-30
5. ROUND(to_date('16-9-1995','dd-mm-yyyy'), 'mm'):1995-10-01
6. TRUNC(to_date('16-9-1995','dd-mm-yyyy'), 'mm'):1995-09-01

```sql
select last_name, (SYSDATE - hire_date) / 7 as weeks,
sysdate + 1 as tomorrow, hire_Date + 8/24
from employees
where department_id = 90
```

日期的运算操作

```sql
select last_name, to_char(hire_date, 'fmDD "of" Month YYYY') as HIREDATE
from employees

alter session set NLS_CURRENCY = '￥'

select to_char(salary, 'L99,999.00')
from employees

select to_number('$45,345', '$9,999,999,999.99')
from dual

select to_date('2018-09-10', 'yyyy-mm-dd')
from dual
```

Oracle数据类型的隐式转换规则：

1. 对于赋值操作可以：
    - varchar2 or char -> number
    - varchar2 or char -> date
    - number -> varcahr2
    - date -> varchar

2. 对于表达式比较操作仅可以
    - varchar2 or char -> number
    - varchar2 or char -> date

Oralce数据类型的显式转换:

1. number -> character : TO_CHAR
2. date -> character : TO_CHAR
3. character -> number : TO_NUMBER
4. character -> date : TO_DATE

```sql
select last_name, nvl(to_char(manager_id), 'No Manager')
from employees
where manager_id is null

select last_name, salary, nvl(commission_pct, 0),
(salary*12) + (salary*12*nvl(commission_pct, 0)) an_sal
from employees
```

其他常用函数:

1. NVL(expr1, expr2):if expr1 is null return expr2 else return expr1
2. NVL2(expr1, expr2, expr3):if expr1 is null return expr3 else return expr2
3. NULLIF(expr1, expr2):if expr1 = expr2 return null else return expr1
4. COALESCE(expr1, expr2, ..., exprn):for expr in exprs : if expr is not null return expr

```sql
select last_name, job_id, salary,
       case job_id
         when 'IT_PROG' then 1.10*salary
         when 'WY_CLERK' then 1.15*salary
         else salary
       end "revised_salary"
from employees

select last_name, job_id, salary,
       decode(job_id, 'IT_PROG', 1.10*salary,
                      'ST_CLERK', 1.15*salary,
                      salary) "revised_salary"
from employees
```

条件表达式