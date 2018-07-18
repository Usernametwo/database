# 用户权限控制

```sql
create user HOPS identified by HOPS
alter user HOPS default tablespace HOPS_DATA quota unlimited on HOPS_DATA
alter user HOPS temporary tablespace temp
grant create session
      , create procedure
      , create sequence
      , create trigger
      , create view
      , create synonym
      , alter session
to HOPS
grant resource to HOPS
```

给用户赋予系统权限

```sql
create role manager
grant create table, create view to manager
grant manager to HOPS
```

通过角色来简化管理

```sql
grant update(department_name, location_id)
on departments
to HOPS,manager

-- with grant option 可以让用户进一步将该权限赋给别人
grant select, insert
on departments
to HOPS
with grant option

--to public 让所有人获得相关权限
grant select
on orcl.departments
to public
```

给用户赋予对象权限

```sql
revoke select
on orcl.departments
from public
```

REVOKE：回收权限

```sql
select * from role_sys_privs
```

通过数据字典视图查询系统中的赋权情况

1. ROLE_SYS_PRIVS：角色对应的系统权限
2. ROLE_TAB_PRIVS：角色对应的表权限
3. USER_ROLE_PRIVS：用户的角色分配表
4. USER_TAB_PRIVS_MADE
5. USER_TAB_PRIVS_RECD
6. USER_COL_PRIVS_MADE
7. USER_COL_PRIVS_RECO
8. USER_SYS_PRIVS：用户的系统权限

```sql
create public database link hq.acme.com
using 'sales'

select * from emp@HQ.ACME.COM
```

DB-LINK访问另一个数据库中的表