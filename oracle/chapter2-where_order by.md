# 条件限制和排序

***

## 条件限制

```sql
select employee_id, last_name, job_id, department_id
from employees
where department_id = 90
```

条件限制关键词：WHERE

```sql
select last_name, salary
from employees
where salary between 2500 and 3500

select employee_id, last_name, salary, manager_id
from employees
where manager_id in (100, 101, 201)

select last_name
from employees
where last_name like '_o%'

select last_name, manager_id
from employees
where manager_id is null
```

比较操作符:

1. =
2. \>
3. \>=
4. <
5. <=
6. <> : 不等于
7. BETWEEN ... AND ...
8. IN(set)
9. LIKE : 模糊匹配
    - `%`代表0个或多个字符
    - `_`代表一个字符

    ```sql
    select * from t_char where a like '%\%%' escape '\'

    select * from t_char where a like '%K%%' escape 'K'
    ```

    如果要匹配通配符本身则使用`ESCAPE`关键字
10. IS NULL : 判断是否为空不能使用=NULL

```sql
select employee_id, last_name, job_id, salary
from employees
where salary >= 10000
and job_id like '%MAN%'

select employee_id, last_name, job_id, salary
from employees
where salary >= 10000
or job_id like '%MAN%'

select last_name, job_id
from employees
where job_id not in ('IT_PROG', 'ST_CLERK', 'SA_REP')
```

逻辑操作符:

1. AND
2. OR
3. NOT

***

## 排序

```sql
select last_name, job_id, department_id, hire_date
from employees
order by hire_date

select last_name, job_id, department_id, hire_date
from employees
order by hire_date desc

select employee_id, last_name, salary*12 annsal
from employees
order by annsal

select last_name, department_id, salary
from employees
order by department_id desc, salary desc
```

ORDER BY子句排序：

1. ASC : 升序(default)
2. DESC : 降序