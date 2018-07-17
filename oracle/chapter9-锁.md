# 锁的概念

Oracle中的锁的主要作用是：防止并发事务对相同的资源进行更改的时候，相互破坏
对于某一用户对数据进行更改而未提交，Oracle会隐式的加锁。

当然也可以显式加锁，比如 : select ... from tableA where ... For UPDATE NoWait

```sql
--未验证
create table test_table
(pk1 number,
field1 varchar(200));

alter table test_table add constraint test_table_pk primary key(pk1);

insert into test_table values(1, 'AAA');

select a.*, C.type, C.LMODE
from v$locked_object a, all_objects b, v$lock c
where a.OBJECT_ID = b.OBJECT_ID
and a.SESSION_ID = c.SID
and b.OBJECT_NAME = 'test_table';

insert into test_table values(1, 'BBB');

select decode(request, 0, 'Holder:', 'Waiter:') || sid sess,
id1,
id2,
Imode,
request,
TYPE
from gv$lock
where (id1, id2, TYPE) in
(select id1, id2, TYPE
from gv$lock
where request > 0
and TYPE != 'HW')
order by id1, request;
```

由于程序问题导致Insert 相同主键会导致死锁的可能

解锁