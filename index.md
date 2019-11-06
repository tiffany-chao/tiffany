---
title: 'Database note'
disqus: hackmd
---

Basic Database Concept
===


## Table of Contents

[TOC]

## Data Type

Numerical/Approximate

| Data Type | Description | Byte |
| -------- | -------- | -------- |
| BIGINT     | interger, no decimal     | 8     |
| INT | -2147483647 to 2147483647  | 4 |
| SMALLINT| -32768 to 32767| 2|
| TINYINT     | 0-255(+)    | 1     |

| Data Type | Description | Byte |
| -------- | -------- | -------- |
| REAL     | approximate     | 7     |
| FLOAT |approximate  | 16 |
| DECIMAL|exact numerical value| p|
| NUMERIC     | exact numerical value   | p     |

Boolean and String
| Data Type | Description |
| -------- | -------- | -------- |
| Boolean     | store T or F    | 
| Binary |fixed length binary string  | 
| Character |fixed length character string  |
| VARBINARY     | variable length binary string   | 
| VARCHAR|variable length character string| 

#### Reference: http://www-db.deis.unibo.it/courses/TW/DOCS/w3schools/sql/sql_datatypes_general.asp.html

## Basic composition

1. Table: rows and columns
2. Page: contains tables, 8kb

2. Extend: 8 pages to make the data more consistent

## Index
To make the query with higher efficiency
### Cluster Index

* B-tree structure
* rapid row retrieval
* A cluster index is a form of tables which consist of column and rows.
* Cluster index exists on the physical level
* It sorts the data at physical level
* ordered data rows
* It works for the complete table
* There is a whole table in form of sorted data 
* A table can contain only one cluster index

```sql=
-- serach which index is primary
Use TEST_DB_file
execute sp_helpindex [Table_repl_1] 
```
### Non Cluster Index
* A non cluster index is in the form of a report about the tables.
* They are not created on the physical level but at the logical level
* It does not sort the data at physical level
* no need to order data rows
* A table has 255 non clustered indexes
* A table has many non clustered indexes.
* It work on the order of data

### Common issue
* Fragmentation
* detect: 
1. sys.dm_db_index_phsical_stats
![](https://i.imgur.com/Nl7maYR.png)
2. use SSMS: tables-index-properties-framentation
* repair:
1. SSMS: tables-index-reorganize
2. T-sql:
![](https://i.imgur.com/YXRXLw1.png)

* Missing index
* Find:![](https://i.imgur.com/eru1kYk.png)

* Underutilized index
* Existing column store index
### Columnstore
#### Reference:
https://stackoverflow.com/questions/2373906/difference-between-cluster-and-non-cluster-index-in-sql

### Note: 


索引設計注意事項
索引設計基本上需對應資料存取 TSQL 才能到位，這裡提出一些索引設計須注意相關事項讓大家參考：
 
 1. 先建立非叢集索引後建立叢集索引
 前面索引類型內容有提到，當一個資料表沒有叢集索引時，先建立非叢集索引的分葉層頁面指標將指向真實資料位置，如叢集索引已經存在時將指向叢集索引，先建立非叢集索引可以避免當你重建叢集索引時，連帶影響非叢集索引一起重建 (須注意索引兩次重建問題)，這不僅拉長重建索引時間，也增加交易紀錄檔大小且浪費硬碟空間。
 
 2. 使用索引壓縮
 使用索引壓縮可以提高緩衝區命中率 (緩衝區可存放更多詞條) 並減少 I/O 和硬碟空間，建立索引時可以善加利用，但由於壓縮操作需要使用較多 CPU 資源，所以須注意 CPU 資源競爭情況。
 #### Reference:
 https://blogs.technet.microsoft.com/technet_taiwan/2015/01/22/tsql-3/
 
 ## Constraints
 used to specify the data type of table
 ### Primary key
 It is recommended to set a primary key for each table. It is a unique value and non-null. Like ID that no one is the same. We can use this to select.
 ### Unique key
 It is unique in column but can be null.
 ### Foreign key
 Ensure the data must match the other table.
 ### Check key
 Provide T or F condition to check some condition. eg. birth>1995
 ### Index
 ### Not full
 
 ## View
 Virtual table created by query
 ### Pros
 1. safety
 2. independent
 
 ### Cons 
 1. If you have a lot of views in your stack, and you use them frequently (views that have joins to other views), database changes can be a nightmare.
 ## Derived Table
 * 用select出來的結果當成是一個derived table,是動態的,不用額外宣告變數的temp table
 
 ## Stored procedure
 * like function
 * EXEC
 * Functions follow the computer-sciency definition in that they MUST return a value and cannot alter the data they receive as parameters (the arguments). Functions are not allowed to change anything, must have at least one parameter, and they must return a value. Stored procs do not have to have a parameter, can change database objects, and do not have to return a value.
 
 ![](https://i.imgur.com/QhS7YaP.png)
 
 
 
 
 ## Alarm and Trigger
 trigger:
 ![](https://i.imgur.com/EJtH95q.png)
 
 ## CTE(common table expression)
 * With CTE_name(column_1,column2)
 As (CTE_query)
 * Temporable result sets
 ![](https://i.imgur.com/5JCRZqe.png)
 
 ## Cursor
 * work slowly
 * read row by row
 * use cursor when return multiple records
 * just like for loop
 * may combine with store procedure so that we can read easily
 ```gherkin=
 Delcare filmcursor CURSOR
 FOR SELECT filmid from tbfilm
 
 Open filmcursor
 --do sth useful
 Fetch next from filmcursor
 --into @id
 --feed the value into variable
 while @@fetch_status=0
 --is succesful get cursor
 Fetch next from filmcursor
 Close filmcursor
 Deallocate filmcursor
 
 ```
 
 ## Transaction
 Usually start with being and ends with "commit".(Notice: If forget to use commit, may cause block.) After make the transaction, the sql will be implemented sequentially.
 
 * Four properties(acid)
 1. atomicity:全做或全沒做
 * 單獨性．比如一個 Transaction 裡有一個 Update command，一個 Delete command．如果 Update command 成功了而 Delete command 失敗了，則這個 Transaction 便是失敗的，所以 Update command 必須回復修改過的資料
 * 例外:primary key violation/lock expiration timeout
 3. consistency: db要遵守rules
 * 一致性代表的是在 Transaction 執行前後，資料皆處於合乎所有規則的狀態．例如，某個欄位具有 foreign key 的關係，其資料的內容不是能隨意產生的，必乎符合 foreign key 的關係，所以 transaction 在執行後，這樣的關係必需持續下去
 5. isolation: sql provide two modes-lock(share lock)/row versioning
 * row versioning:default for Azure SQL Database,其他使用者可以讀到舊的資料
 * 這指的是不同的 Transaction 之間執行時不能被彼此干擾．假設有兩個 Transaction 在同一時間對相同的一百筆資料進行更新動作，而其中一個 Transaction 在更新到一半時產生錯誤，於是進行資料回復．然而，另一個 Transaction 是完成的．可想而知最後的資料狀態一定是無法預測，因為不清楚那些資料是失敗的 Transaction 做資料回復的，還是成功的 Transaction 所更新完成的
 6. durability:always write data change in log on disk before written to database on disk 
 * 資料的耐力．資料庫引擎必須要保證一旦 Transaction 完成後，不論發生任何事情，如系統當機或電力中斷等，運作後的資料都必須存在於儲存媒介中，也就是在硬碟裡．
 ### Transaction failure
 1. system failure/crash
 2. transaction or system error
 eg.1/0 or give overrage parameter
 3. concurrency control enforcement
 4. local error or exception
 5. disk failure
 6. physical problem
 eg. earthquakes (hard to recover)
 
 * no too long transaction is better
 ## Security
 In order to keep the database safe, limit the users to access the database is necessary.
 
 * securable:需要被保護的物件,eg. server,DB...
 * principal:帳號，安全身分辨識
 * permission:權限,主體可以對securable執行的動作
 * server(login)->db(user)
 * Authentication驗證(證實身分有效)
 * 授權login連到某database
 * We will have two way to enter the instance. Windows or SQL server.
 
 * Logins and role
 We can create the user to map the login for database. Also, we can assign the role for them, eg. admin, guest...
 * schema:命名空間
 * partially contained database部分自主資料庫
 * 授權使用者存取:grant/revoke/deny
 
 * Audit
 We can use this function to audit the command or log in server.
 * Define server audit
 * audit target
 * audit actions and groups
 * create audit specification
 * Trigger
 * DML:INSERT,UPDATE,DELETE
 * DDL:建立,卸除,變更指定
 * 加密
 ## Backup 
 * full:資料檔(.mdf)和交易紀錄檔(.ldf)
 * differential:指備份資料檔,從上次完整備份之後差異的部分
 * 交易紀錄檔備份:指備份交易紀錄檔,LSN紀錄序號是連貫的
 * 檔案/檔案群組:主要資料檔,次要資料檔,交易紀錄檔
 
 ## Recovery
 * simple:不備份交易紀錄,在checkpoint發生時會截斷已完成或回復的交易紀錄
 * full:記錄所有交易直到備份完之後
 * bulk-logged:大部分交易紀錄保存,除了bulk行為(eg.建立索引,大量載入資料)
 
 * recovery:當還原程序執行完之後，資料庫回復正常狀態，可供存取(使用於最後一次備份)
 * norecovery:還原程序執行後進入離線模式，資料庫無法使用
 * standby:還原程序執行後進入資料庫唯讀模式
 * 實際操作: http://vito-note.blogspot.com/2013/07/blog-post_4016.html
 
 
 ## Normalization
 ref: http://cc.cust.edu.tw/~ccchen/doc/db_04.pdf
 使得欄位之間資料彼此不重複,減少記憶體的使用,提升效能
 
 * 1NF
 * 每一個欄位都只有一個值
 * 沒有兩筆以上的資料完全相通
 * 資料表中有primary key,其他欄位和primary key相依
 ![](https://i.imgur.com/DAYLdNI.png)
 
 * 2NF
 * 分割出部分功能相依(eg.姓名相依學號,課程名稱只相依課程代號),另創表格填入
 * 
 ==標記==
 
 
 
 SQL function
 ===
 * DDL(Data definition language)-定義和管理資料庫的所有物件的語言
 * create
 * alter
 * drop
 * 隱式提交,用sql命令間接完成提交
 * DML(Data manipulation language)-處理資料操作
 * insert
 * delete
 * update
 * select
 * 顯示提交,在最後執行commit/rollback
 * DCL(Data control language)-授予或回收訪問資料庫的某種特權
 * grant
 * rollback
 * commit
 * TCL-for transaction
 
 * Join
 ![](https://i.imgur.com/smSmBBO.png)
 
 
 * Group function
 ![](https://i.imgur.com/xCS6ZHE.png)
 
 
 
 
 
 Error and Case study
 ===
 
 
 
 ## High Availability(HA)
 1. Replication
 將主要要資料庫的特定資料透過訂閱發行複製到另一個資料庫上
 3. Log shipping
 以整個資料庫為複寫對象,利用排程於固定時間,將主資料庫複寫到目標資料庫中
 5. Database mirroring
 log shipping+auto failover可進行切換,也是對整個資料庫進行複寫
 7. Failover clustering
 ## Failover
 
 Usually work under a backup situation, will standby in case the primary server can't work. As long as that happended, it will transfer the data from the primary to standby machine. (將資料轉移到其他地方)
 
 ### Properties
 1. share storage
 
 ### Common collocation
 1. AG(avalibility group) failover
 2. Alawyas on: monitor the server
 3. Clusterd failover: share IP, storage, DB
 
 ## LeaseTimeout
 
 It controls the connection mechanism between server and resource. It is a "handshake" between DLL resource and SQL server. Usually when over the lease, it is because of the system wide event or application on system cause massive paging to occure and slow the system.
 
 ---
 
 ```sql=
 sp_server_diagnostics
 only one parameter(HealthCheckTimeout)-repeat time(>5s) 
 ```
 ---
 ## Always On Avalibility Group
 
 
 Database mirroring technique
 
 1. is avalible for multiple database.
 2. build on top of windows failover cluster
 3. dba can make them read-only
 4. primary and secondary db
 5. share IP, independent DB & Storage
 
 Discussion
 1. https://dba.stackexchange.com/questions/219604/always-on-failover-clustering-vs-always-on-availability-groups
 ## Blocking
 
 Lock: when someone is using the table, alter or update, others cannot use until he finished
 
 Blocking:不只一個user同時在使用同一個東西
 * 如何設定:https://dotblogs.com.tw/stanley14/2017/11/21/find_sql_block2
 
 DeadLock: Assume there are two table, both are waiting for the other to commit the transaction and no one can move forward
 * https://dotblogs.com.tw/stanley14/2017/01/28/191015
 
 
 SQL interview quesitons
 ===
 * what is difference bewteen delete and truncate?
 ![](https://i.imgur.com/M2PpOUz.png)
 
 * what is DBMS? What are its different type?
 ![](https://i.imgur.com/ZesR6vl.png)
 
 * Data integrity
 
 ![](https://i.imgur.com/N2lHbik.png)
 
 * null value != zero
 
 ![](https://i.imgur.com/MXQFNyZ.png)
 
 * select 第三高的薪水
 
 ![](https://i.imgur.com/eoT2MIw.png)
 
 * Relationship type
 ![](https://i.imgur.com/EJQS3tI.png)
 * self-referencing: 單一一個table的兩個欄位可以互相有relation
 
 # Oracle
 ## Architecture
 
 ![](https://i.imgur.com/cAGPkMl.png)
 
 # Reference
 ### Books
 * https://pdfs.semanticscholar.org/cf15/b7b2a1d9316813e56abc9370145098385c5a.pdf
 ### FCI, WSFC, AG
 1. https://ifun01.com/49BNFKZ.html
 2. https://mssqltaiwan.wordpress.com/2018/01/09/fci-and-always-on/

