# 多表关联查询

数据库的范式:

1. 第一范式：数据库表的每一列都是不可分割的原子数据项(无重复的域)。例如一个实体中的A列可以分为B列和C列，那么就不符合第一范式
2. 第二范式：在第一范式的基础上，消除非主属性对主码的部分函数依赖（将这个属性和主码的这一部分分离出来形成一个新的实体，新实体与原实体是一对多的关系），即实体的属性完全依赖于主键。例如一个实体主码为(A, B),这个实体的C列仅依赖于A，那么应该把这个实体的C列和A列提取出来变成单独的一个实体，与原实体是一对多的关系
3. 第三范式：在第二范式的基础上，消除传递依赖（任何非主属性不依赖于其他非主属性）。例如一个实体主码为A，这个实体的列C依赖于B，那么应该把列B和C提取出来变成单独的一个实体
4. BC范式：在第三范式的基础上，消除非主属性对主码子集的依赖。例如一个实体的主码为(A,B,C)，这个实体的D列仅依赖于(A,B)，那么应该将(A,B,D)列提取出来组成一个新的实体，与原实体是一对多的关系

为了避免冗余信息，表设计结构遵循BC范式

```sql
select e.employee_id, d.department_id, d.location_id
from employees e, departments d

select e.employee_id, d.department_id, d.location_id
from employees e cross join departments d
```

笛卡尔积

交叉连接

```sql
select e.employee_id, e.last_name, e.department_id, d.department_id, d.location_id
from employees e, departments d
where e.department_id = d.department_id

select e.employee_id, e.last_name, e.department_id, d.department_id, d.location_id
from employees e inner join departments d
on e.department_id = d.department_id

select e.employee_id, e.last_name, e.department_id, d.department_id, d.location_id
from employees e join departments d
on e.department_id = d.department_id

select e.employee_id, e.last_name, department_id, d.location_id
from employees e natural join departments d

select *
from employees  join departments
using (manager_id)
```

等值连接:使用=

自然连接:自然连接会让系统自动查找两张表中的所有列名相同的字段，并试图建立“等值连接”;如果想指定列连接，需要用到 using 子句

```sql
select e.last_name, e.salary, j.job_title
from employees e, jobs j
where e.salary between j.min_salary and j.max_salary

select e.last_name, e.salary, j.job_title
from employees e inner join jobs j
on e.salary between j.min_salary and j.max_salary
```

不等值连接:使用不等连接符，如：>,!=,between .. and ..., ...

INNER JOIN(JOIN):内连接包含等值连接(包含自然连接)和不等值连接

```sql
SELECT e.last_name, e.department_id, d.department_name
FROM employees e, departments d
WHERE e.department_id = d.department_id(+)

SELECT e.last_name, e.department_id, d.department_name
FROM employees e left outer join departments d
on e.department_id = d.department_id

SELECT e.last_name, e.department_id, d.department_name
FROM employees e right outer join departments d
on e.department_id = d.department_id
```

左右外连接：左外连接以左边的表为主表，右外连接以右边的表为主表。主表存在而从表不存在的值会用null替代

```sql
SELECT e.last_name, e.department_id, d.department_name
FROM employees e
FULL OUTER JOIN departments d
ON (e.department_id = d.department_id)
```

FULL OUTER JOIN:全外连接，对于一张表存在另一张表不存在的记录会将不存在的字段用null替代

OUTER JOIN:外连接，包括等值连接(LEFT OUTER JOIN, RIGHT OUTER JOIN, FULL OUTER JOIN)和不等值连接(此处没有例子，可以自己尝试下)

```sql
SELECT worker.last_name || ' works for ' || manager.last_name
FROM employees worker, employees manager
WHERE worker.manager_id = manager.employee_id
```

自连接