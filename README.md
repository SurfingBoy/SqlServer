### 1.创建触发器（不能插入数据）

```
create trigger tr_insert on bank
for 
insert
as
delete *from bank where cid=(select cid from inserted)
```
### 2.删除谁就让谁的账户加上10元

```
create trigger tr_delete on bank
instead of
delete
as
update bank balance=balance+10 where cid=(select cid from deleted)
```

### 3.子查询

```
with name as (select *from bank where banlance>10000)
select *from name
```

### 4.删除重复行

```
with m as 
(select *,ROW_NUMBER() OVER (PARTITION BY depotid,time ORDER BY depotid) rn FROM test)

delete from m where rn=2
```

### 5.统计登录次数

```
select u.UserId 用户编号,u.Name 姓名,
count(case when L.Time is not null then 1 else null end) 登录次数,
count(distinct convert(char(10),L.Time,120)) 登录天数，
count(distinct case when convert(char(7),L.Time,120)='2017-01' then 1 else null end ) 一月份登陆次数 
from [User] u 
left join [Log] L on u.UserId=L.UserId
where convert(char(4),L.Time,120)=2011
group by u.UserId,u.Name
having count(case when L.Time is not null then 1 else null end)>=0
```

### 6.创建链接服务器

```
EXEC sp_addlinkedserver 
@server='SHYL',--被访问的服务器别名 
@srvproduct='', 
@provider='SQLOLEDB', 
@datasrc="w" --要访问的服务器地址

EXEC sp_addlinkedsrvlogin 
'SHYL', --被访问的服务器别名 
'false', 
NULL, 
'sa', --帐号 
'' --密码 
```
