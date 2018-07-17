# 事务控制

ORACLE数据库通过事务来控制数据的一致性，当用户进程或者系统崩溃时，事务可提供用户更多的灵活性和控制，以确保数据的一致性
***

## 隐式事务提交和回滚

当下列事件发生时，会隐式的执行Commit动作

1. 数据定义语句被执行的时候，比如新建一张create table ... 等DDL语句
2. 数据控制语句被执行的时候，比如赋权: grant ...
3. 正常退出 plsql

当发生如下事件时，会隐式的执行Rollback动作：

1. 非正常退出plsql时

在Commit或者Rollback前后数据的状态:

1. 在数据已经被更改，但没有Commit前，被更改记录处于被锁定状态，其他用户无法进行更改
2. 在数据已经被更改，但没有Commit前，只有当前Session的用户可以看到这种变更，其他Session的用户无法看到数据变化
3. 在数据已经被更改，并且被Commit后，被更改记录自动解锁，其他用户可以进行更改
4. 在数据已经被更改，并且被Commit后，其他Session用户再次访问这些数据看到的是改变之后的数据
5. RollBack前后的数据状态同上

```sql
-- DDL,commit auto
create table tbl (name varchar2(10));
--DML
insert into tbl values('gb');
rollback;
select * from tbl;

insert into tbl values('gb');
commit;
rollback;
select * from tbl;
```

COMMIT : 将数据的变化永久保存
ROLLBACK : 将变化之前的数据还原回去

一旦COMMIT就不能ROLLBACK回去了

SQL语言分为五大类：

1. DDL : Create, Alter, Drop等，隐式Commit
2. DQL : Select不存在提交问题
3. DML : Insert, Update, Delete需要显示Commit
4. DTL : Commit, Rollback事务提交与回滚语句
5. DCL : Grant, Revoke授予权限与回收权限语句

***

## 读一致性

Oracle的读一致性概念指：

1. 在任何时刻，确保数据的一致性视图
2. 一个用户对数据的更改并且commit之后不会影响另一个用户对数据的更改
3. 读一致性确保在同一时刻内：

    - 读数据的人不需要等待写数据的人
    - 写数据的人不需要等待读数据的人

当由用户修改数据的时候，Oracle会先把那部分原始数据备份到回滚段，在Commit之前，其他Session用户读到的这部分数据是回滚段上面的，在提交之后，回滚段被释放