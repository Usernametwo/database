# SELECT语句

```sql
select last_name, salary, salary + 300
from employees
```

SQL语句中的数学表达式：对于`数值`和`日期类型`字段，可以进行“加减乘除”

```sql
select null + 5, null * 5
from dual
```

关于NULL的概念：一个数值与NULL进行四则运算其结果还是NULL

```sql
select last_name as name, commission_pct comm, salary*12 "annual salary"
from employees
```

给列起别名：注意双引号

```sql
select last_name || job_id as "employees"
from employees

select last_name || ' is a ' || job_id as "employee detail"
from employees
```

字符串连接操作符:`||`

```sql
select distinct department_id
from employees
```

`DISTINCT`：去重