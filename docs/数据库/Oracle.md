# Oracle

```sql
SELECT b.sid oracleID,  
       b.username 登录Oracle用户名,  
       b.serial#,  
       spid 操作系统ID,  
       paddr,  
       sql_text 正在执行的SQL,  
       b.machine 计算机名  
FROM v$process a, v$session b, v$sqlarea c  
WHERE a.addr = b.paddr  
   AND b.sql_hash_value = c.hash_value 
```


<a name="7jWo3"></a>
### 查询 oracle 操作记录
```sql
select t.FIRST_LOAD_TIME, t.SQL_TEXT,t.SQL_FULLTEXT  from v$sqlarea t  
where t.FIRST_LOAD_TIME like '2020-05-28%'  
  and (sql_fulltext like '%update%' or sql_fulltext like '%UPDATE%' )  
  and t.MODULE='JDBC Thin Client'
  order by t.FIRST_LOAD_TIME desc;
-- 查询当前的 事务
select * from v$sql,v$transaction where LAST_ACTIVE_TIME=START_DATE;
```
网上不少人都是这么推荐，这个就是查询 5 月 28 号的操作记录，但是根据我的测试结合官方文档，这个 FIRST_LOAD_TIME 并不是表示运行时间，而是 sql 首次加载的时间，如果刚刚运行完，建议通过 LAST_ACTIVE_TIME 去查询更加准确。<br />
<br />

