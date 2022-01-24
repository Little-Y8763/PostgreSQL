# PostgreSQL

* [Install](#Install)
* [pgAdmin](#pgAdmin)
* [Note](#Note)
* [測試](#測試)

<h1 id="Install"> Install </h1>

>選則作業系統類型(以 windows 為例)

![1](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012101.PNG)

>點`Download the installer`

![2](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012102.PNG)

>選擇版本

![3](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012103.PNG)

>確認下載成功，執行安裝檔

![4](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012104.PNG)

>Next

![5](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012105.PNG)

>設定 Admin 密碼

![6](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012106.PNG)

>輸入可以讓 server 監聽的 port

![7](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012107.PNG)

>Finish

![8](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012108.PNG)

<h1 id="pgAdmin"> pgAdmin </h1>

>輸入 Admin 密碼

![9](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012109.PNG)

>將左邊展開，點擊 Databases 以下的隨便一個東西，再點擊紅框裡的圖示就可以開啟 Query Editor，或是也可以從上方 Tools 裡選擇 Query Tool 一樣可以開啟 Query Editor

![10](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012110-2.PNG)
![11](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012110-3.PNG)

>開啟 Query Editor 後，就可以開始下 Query (以 CREATE TABLE 為例)，下好後點擊`三角形圖示`或`F5`就可以執行 

![12](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012112.PNG)

>Query Editor 下方的 Messages 可以看到一些資訊，像 Query 執行的時間或是錯誤訊息

![13](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012113.PNG)

>剛 CREATE TABLE 完，展開左邊的 Tables 的話會沒有東西，Refresh 過後再點 Tables 裡面才會有東西

![14](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012114.PNG)

![15](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012115.PNG)

>Query History 可以看到之前下過的 Query，點選其中一個 Query 可以複製或是直接複製到 Query Editor

![16](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012123.PNG)

![17](https://github.com/Little-Y8763/PostgreSQL/blob/main/Doc/picture/2022012124.PNG)

<h1 id="Note"> Note </h1>

## Query Note

>Create Table 以日期做分區
```sql
CREATE TABLE TEST ("deviceId" text, "dateTime" timestamp(0) without time zone, "A360000" real, "A360002" real, "A360004" real, "A360006" real, "A360008" real, "A360010" real, "A360012" real, "A360014" real, "A360016" real, "A360018" real, "A360020" real, "A360022" real, "A360024" real, "A360026" real, "A360028" real) PARTITION BY RANGE ("dateTime");
```

>Create Partitions
```sql
CREATE TABLE test_1 PARTITION OF test FOR VALUES FROM ('2022-01-01') TO ('2029-01-01');
```

>查看有幾個 Partition 和全部 Partition 佔的容量
```sql
SELECT count(*) AS child_amount, pg_size_pretty(sum(pg_relation_size(inhrelid::regclass))) AS child_size
FROM pg_inherits 
WHERE inhparent='test'::regclass;
```

>建立Index
```sql
CREATE INDEX INDEX_DEVICE ON TEST ("deviceId","dateTime");
```

>查看 Table 有哪些 Index
```sql
SELECT * FROM PG_INDEXES WHERE TABLENAME = 'test';
SELECT * FROM PG_STATIO_ALL_INDEXES WHERE RELNAME = 'test';
```

>刪除 Index
```sql
DROP INDEX IF EXISTS index_name
```

>Table Size(一般的 Table 才看的到，有建 Partitions 的就要用上面的)
```sql
SELECT PG_SIZE_PRETTY(PG_RELATION_SIZE('test'));
```

## Data Type Note

* character varying(n) = varchar(n)，可變長度，但有限制
* character(n) = char(n)，固定長度，空白填充
* text = varchar，可變且無限長度
* timestamp(p) with time zone = timestamp(p)，p可以填0~6，是秒保留的小數位數
* data type 有 [] 就是可以存 array

## 參考資料

* https://docs.postgresql.tw/reference/sql-commands/copy
* https://medium.com/d-d-mag/postgresql-%E7%95%B6%E4%B8%AD%E7%9A%84-index-e7e1e8d9340c
* https://docs.postgresql.tw/the-sql-language/ddl/table-partitioning
* https://www.enterprisedb.com/postgres-tutorials/how-use-table-partitioning-scale-postgresql
* https://stackoverflow.com/questions/18973289/pg-relation-size-command-is-not-providing-size

<h1 id="測試"> 測試 </h1>

## 測試環境

* OS Name: Microsoft Windows Server 2016 Standard Evaluation
* OS Version: 10.0.14393 Build 14393
* CPU: Intel Xeon E3-1240 v6
* RAM: DDR4 16G(Kingston 9965684-005.A00G 8G + Apacer 76.C105G.D100B 8G)
* STORAGE: ST1000DM010-2EP102

## 寫入測試資料
```c#
var device = new List<string>() { "ttt1", "ttt2", "ttt3", "ttt4", "ttt5", "ttt6", "ttt7", "ttt8", "ttt9", "ttt10" };
Stopwatch stopWatch = new Stopwatch();
Console.WriteLine("Press Enter");
Console.ReadLine();
stopWatch.Start();

var rand = new Random();
var connString = "Host=localhost;Port=5432;Username=postgres;Password=CSIEcsie2964;Database=postgres";
var count = 0;

using (var conn = new NpgsqlConnection(connString))
{
    conn.Open();

    using (var writer = conn.BeginBinaryImport("COPY test (\"deviceId\",\"dateTime\",\"A360000\",\"A360002\",\"A360004\",\"A360006\",\"A360008\",\"A360010\",\"A360012\",\"A360014\",\"A360016\",\"A360018\",\"A360020\",\"A360022\",\"A360024\",\"A360026\",\"A360028\") FROM STDIN (FORMAT BINARY)"))
    {
        for (int j = 1; j < device.Count + 1; j++)
        {
            for (int i = 1; i < 2000001; i++)
            {
                count++;
                writer.StartRow();

                writer.Write(device[j - 1]);
                writer.Write(DateTime.Now.AddMinutes(count));
                for (int k = 0; k < 15; k++)
                {
                    writer.Write((float)rand.NextDouble());
                }
            }
        }
        writer.Complete();
    }
}
stopWatch.Stop();
var time = stopWatch.Elapsed;
string timeFormat = string.Format("{0:D2}h:{1:D2}m:{2:D2}s:{3:D3}ms", time.Hours, time.Minutes, time.Seconds, time.Milliseconds);
Console.WriteLine(timeFormat);
```

## 測試結果
>寫入測試資料時間

10台設備，每台設備200萬筆資料，總共2000萬筆資料 2m4s

>Query 執行時間

* 查詢 2000萬筆資料 1m26s

* where 兩個條件 deviceId,time(小時)的最後1筆資料(資料總共38年)

 1. 沒 index 沒 partition 2.3~2.9s，用 deviceId 和 dateTime 建index 112ms~244ms
 2. 沒 index 6個 partition(以7年分割) 658ms~910ms，用 deviceId 和 dateTime 建index 115ms~289ms
 3. 沒 index 457個 partition(以月分割) 128ms~280ms，用 deviceId 和 dateTime 建index 114ms~225ms
 4. 沒 index 1371個 partition(以15天分割) 114ms~389ms，用 deviceId 和 dateTime 建index 112ms~302ms
