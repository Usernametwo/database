# 数据库对象-表

数据类型：

1. VARCHAR2(size) : 可变长字符串
2. CHAR(size) : 定长字符串
3. NUMBER : 可变长数值
4. DATE
5. LONG : 可变长大字符串,最大2G
6. CLOB : 可变长大字符串,最大4G
7. RAW 和 LONG RAW : 二进制数据
8. BLOB : 大二进制数据
9. BFILE : 存储于外部文件的二进制数据，最大可到4G
10. ROWID : 64进制18位长度的数据（地址）
11. TIMESTAMP : 精确到分秒级的日期
12. INTERVAL YEAR TO MONTH
13. INTERVAL DAY TO SECOND

```sql
select * from all_tables

create table time_example
(order_date1 timestamp,
order_date2 timestamp with time zone,
order_date3 timestamp with local time zone)

insert into time_example values
(
'15-NOV-00 09:34:28',
'15-NOV-00 09:34:28 am -8:00',
'15-NOV-00 09:34:28 am'
)

--复制表
create table time_example_copy as select * from time_example

--仅复制表结构
create table time_example_copy as select * from time_example where 1=2
```

创建表 : CREATE TABLE

```sql
create table A(a number, b number)

alter table A
add (c number, d number)

alter table A
modify (c varchar(20), d varchar(20))

alter table A
drop (c)

drop table A

rename time_example to time_example_rename

--清空内容，保留表结构，慎用，没有回滚机会，delete有回滚机会
truncate table time_example
```

更改表 ： ALTER ， DROP

表被删除后，任何依赖于这张表的视图，package等数据库对象都自动变为无效
