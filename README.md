# PostgreSQL

# Install

![1](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012101.PNG)

![2](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012102.PNG)

![3](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012103.PNG)

![4](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012104.PNG)

![5](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012105.PNG)

![6](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012106.PNG)

![7](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012107.PNG)

![8](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012108.PNG)

# pgAdmin

![9](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012109.PNG)

![10](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012110.PNG)

![11](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012111.PNG)

![12](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012112.PNG)

![13](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012113.PNG)

![14](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012114.PNG)

![15](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012115.PNG)

![16](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012116.PNG)

![17](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012117.PNG)

![18](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012118.PNG)

![19](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012119.PNG)

![20](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012120.PNG)

![21](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012121.PNG)

![22](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012122.PNG)

# aaa

```sql
CREATE TABLE TEST ("deviceId" text, "dateTime" timestamp(0) without time zone, "A360000" real, "A360002" real, "A360004" real, "A360006" real, "A360008" real, "A360010" real, "A360012" real, "A360014" real, "A360016" real, "A360018" real, "A360020" real, "A360022" real, "A360024" real, "A360026" real, "A360028" real) PARTITION BY RANGE ("dateTime");
```

```sql
CREATE TABLE test_1 PARTITION OF test FOR VALUES FROM ('2022-01-01') TO ('2029-01-01');
```

```sql
SELECT count(*) AS child_amount, pg_size_pretty(sum(pg_relation_size(inhrelid::regclass))) AS child_size
FROM pg_inherits 
WHERE inhparent='test'::regclass;
```

```sql
SELECT * FROM PG_INDEXES WHERE TABLENAME = 'test';
SELECT * FROM PG_STATIO_ALL_INDEXES WHERE RELNAME = 'test';
```

```sql
DROP INDEX IF EXISTS index_name
```

```sql
CREATE INDEX INDEX_DEVICE ON TEST ("deviceId","dateTime");
```

```sql
SELECT PG_SIZE_PRETTY(PG_RELATION_SIZE('test'));
```

```sql
TRUNCATE TABLE test
```

character varying(n) = varchar(n),可變長度，但有限制
character(n) = char(n),固定長度，空白填充
text = varchar,可變且無限長度

timestamp(p) with time zone = timestamp(p),p可以填0~6,是秒保留的小數位數
有 [] 就是 array

https://docs.postgresql.tw/the-sql-language/ddl/table-partitioning
https://www.postgresql.org/docs/9.4/ddl-partitioning.html
https://www.enterprisedb.com/postgres-tutorials/how-use-table-partitioning-scale-postgresql
https://medium.com/d-d-mag/postgresql-%E7%95%B6%E4%B8%AD%E7%9A%84-index-e7e1e8d9340c

CPU: intel Xeon E3-1240 v6
RAM: 16G
STORAGE: HDD

insert
200萬x10台 2m4s

select
2000萬 1m26s
ttt1 200萬 14.5s,最後1筆 1.88s,deviceId和dateTime建index 113ms~269ms 
某欄位平均 5.2s

where 兩個條件 deviceId,time(小時)最後1筆:
(資料總共38年)
1. 沒 index 沒 partition 2.3~2.9s, deviceId和dateTime建index 112ms~244ms
2. 沒 index 6個 partition(7年分割) 658ms~910ms, deviceId和dateTime建index 109ms~289ms
3. 沒 index 457個 partition(月分割) 128ms~280ms, deviceId和dateTime建index 114ms~225ms
4. 沒 index 1371個 partition(15天分割) 114ms~389ms, deviceId和dateTime建index 112ms~302ms
