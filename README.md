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

### 7.游标

```
--定义游标
DECLARE m_cursor CURSOR SCROLL FOR
SELECT id,state FROM @room --语句
OPEN m_cursor              --打开游标
DECLARE @id INT,@state INT  --游标数据存放的位置
FETCH NEXT FROM m_cursor INTO @id,@state  --填充数据
WHILE @@FETCH_STATUS=0 --提取成功
BEGIN
  IF @state=0
  BEGIN
    UPDATE dbo.room SET now_state=0 WHERE room_id=@id
  END
  FETCH NEXT FROM m_cursor INTO @id,@state
END
CLOSE m_cursor      --关闭游标
DEALLOCATE m_cursor --删除游标
```

### 8.定义表变量

```
--定义表变量
DECLARE @room TABLE(id int,state int)

--插入数据
INSERT INTO @room( id, state ) 
SELECT room_id,CASE WHEN @end_time<start_time THEN 1 
					   WHEN @start_time>end_time THEN 1
					   ELSE 0 END variable FROM dbo.meeting 
WHERE CONVERT(VARCHAR(10),start_time,23)=CONVERT(VARCHAR(10),@start_time,23)
```

### 9.xml path

```
SELECT a.meeting_id,a.title,
(SELECT username+',' FROM @info WHERE meeting_id=a.meeting_id FOR XML PATH('')) AS userlist FROM @info a
GROUP BY a.meeting_id,a.title
```

### 10.循环处理

```
WHILE(CHARINDEX(',',@uid)<>0)
  BEGIN
      SET @person=SUBSTRING(@uid,1,CHARINDEX(',',@uid)-1)
      INSERT INTO @selectUser
              ( username, uid )
      SELECT username,uid FROM dbo.[user] WHERE userid=@person
      SET @uid=STUFF(@uid,1,CHARINDEX(',',@uid),'')
  END
```

### 11.随机大数

```
create PROC [dbo].[GetRandStr]
(
    @count INT,
    @no VARCHAR(1000) OUTPUT
)
AS
BEGIN
    DECLARE @randomstr VARCHAR(1000);
    DECLARE @charpool VARCHAR(1000);
    DECLARE @i INT;
    DECLARE @counter INTEGER;
    SET @charpool = '12345678910AaBbCcDdEeFfGgHhIiJjKkLlMmNnOoPpQqRrSsTtUuVvWwXxYyZz';
    SET @i = 1;
    SET @randomstr = '';
    WHILE @i <= @count
    BEGIN
        here:
        SET @counter = CAST(RAND() * 100 / 1.6 AS INTEGER);
        IF @counter < 1
            GOTO here;
        SET @randomstr = @randomstr + SUBSTRING(@charpool, @counter, 1);
        SET @i = @i + 1;
    END;
    SET @no = LEFT(@randomstr, ISNULL(@count, 180));
    RETURN;
END;
```
或
```
set @id=RIGHT(1000000+ISNULL(RIGHT(MAX(id),6),0)+1,7)
```
