# SQL server指令
|[**行為**<br>(CRUD)](#行為)|[**來源**](#來源)|[**操作**](#操作) |[**特殊動作**](#特殊動作)|[**技巧**](#技巧)|
|:--:|:--:|:--:|:--:|:--:|
|[SELECT](#SELECT) <br> [INSERT](#INSERT) <br> [UPDATE](#UPDATE) <br> [DELETE](#DELETE) <br> |[FROM](#FROM) <br> [JOIN](#JOIN) <br> [PIVOT](#PIVOT) <br> |[WHERE](#WHERE) <br> [GROUP](#GROUP) <br> [HAVING](#HAVING) <br> [ORDER](#ORDER) <br> [UNION](#UNION) <br> |[CREATE 新增表格](#CREATE-TABLE) <br> [ALTER 修改表格](#ALTER-TABLE) <br> [DROP 刪除表格](#DROP-TABLE) <br> [TRUNCATE 重置表格](#TRUNCATE-TABLE) <br> | [WITH 子查詢(多層簡化)](#WITH) <br> [OVER 分組處理](#OVER) <br> [DATETIME 時間操作](#DATETIME-時間操作) <br> [Identity 自動流水號](#Identity-自動流水號) <br> [Primary KEY 主索引鍵](#Primary-Key-設定主索引鍵) <br> [取得欄位重複row數量](#取得欄位重複row數量) <br> [取得DB資訊](#取得DB資訊) <br> [DECLARE 臨時變數](#宣告臨時變數) <br>  [其他](#其他) <br> |


## 行為 (CRUD)
### SELECT
```SQL
SELECT *                          /* 全選 */
SELECT [Col1], [Col2], ...        /* 多選 */
SELECT DISTINCT [Col1]                    /* Col1中只取不重複項 */
SELECT DISTINCT [Col1], [Col2]            /* Col1不重複取，Col2不變 */
SELECT CASE WHEN () THEN () ELSE () END   /* IF條件式判斷 */
SELECT CASE a.Country WHEN ('TW') THEN ('Taiwan') WHEN ('JP') THEN ('Japan') ... END   /* IF條件式判斷(取代值) */
SELECT REPLACE( [Col1], '原字串', '新字串' ) AS 'MyCol1'  /* 取代字串並定義新Col */
/* 取代字串可為：CHAR(9)->Tab、CHAR(10)->換行、CHAR(9)->ENTER */
SELECT ISNULL(a.Col2, b.Col2) AS 'MyCol2' FROM (...) b   /* 可在AS時也用From去多抓資料 */
SELECT * INTO [NewTable] FROM [OldTable]    /* 創新表並匯入資料 */
``` 

### INSERT
```SQL
INSERT INTO [DataBase]
             ( [Col1], [Col2], ... )
       VALUES( '001', '002', ... )
       
INSERT INTO [Destination]
SELECT colA, colB, colC
FROM   [SourceDbTable]
```

### UPDATE 
```SQL
/* Update的目標一定要存在，不可null */
UPDATE [DataBase]
SET [Col1]='001', [Col2]='002', ...
WHERE ...
```

### DELETE 
```SQL
DELETE FROM [DataBase] 
WHERE ...
```
### MERGE

## 來源
### FROM
```SQL
/* 命名此來源為a (AS可略) */
FROM [DataBase] AS a

/* nolock防止SQL表鎖住(WITH可略) */
-- 僅限SELECT使用，可提升讀取速度、防止其他寫入行為失敗，但可能讀取到transaction即時更新的未完整資料
FROM [DataBase] AS a WITH (nolock)
```
### JOIN
```SQL
INNER JOIN [DataBase2] AS b       /* 被JOIN的表為b */
        ON a.[Col1] = b.[Col2]    /* 當a表Col1等於b表Col2時，取聯集 */
```
* JOIN類別：
```SQL
INNER JOIN                /* 兩表皆不可JOIN有null之列 */
```
```SQL
LEFT JOIN                 /* a表不變，b表可允許JOIN有null之列 */

LEFT JOIN                 /* a表去除含b表之列 */
    WHERE b.[Col2] IS NULL
```
```SQL
RIGHT JOIN                /* b表不變，a表可允許JOIN有null之列 */

RIGHT JOIN                /* b表去除含a表之列 */
     WHERE a.[Col1] IS NULL   
```
```SQL
FULL OUTER JOIN           /* 兩表皆不變，皆可允許JOIN有null之列 */

FULL OUTER JOIN           /* 兩表只取互不包含之列 */
          WHERE a.[Col1] IS NULL     /* a或b皆可 */
```
![image](https://dsin.files.wordpress.com/2013/03/sqljoins_cheatsheet.png) <br>
[Reference](https://dsin.wordpress.com/2013/03/16/sql-join-cheat-sheet/)

### PIVOT
轉置資料(PIVOT/UNPIVOT)
```SQL
SELECT ID, AVG(Score) AS Average_Score   
FROM DBTable
GROUP BY ID
```
|ID|Average_Score|
|--|--|
|1|60|
|3|100|
```SQL
SELECT 'Average_Score' AS ID, [1], [2], [3]
FROM  
(SELECT ID, Score   
    FROM DBTable) AS SourceTable  
PIVOT   --/UNPIVOT
(
AVG(Score)          // 目標值
FOR ID              // GROUP BY對象
IN ([0], [1], [2])  // 轉置後指定資料，值可直接抓
) AS PivotTable;  
```
|ID|1|2|3|
|--|--|--|--|
|Average_Score|60|NULL|100|



## 操作
### WHERE
```SQL
/* 括號內可優先處理邏輯 */
WHERE (Col1 = 123 OR Col1 like '%123_')   /* like可搜尋字串contains、%代表任何可有可無字元、_代表指定字元數量 */
      AND Col1 is not null                /* is null */
      
WHERE Col1 in ('con1', 'con2')            /* 過濾多條件 */
WHERE Col1 i (SELECT...)                  /* 也可以用SELECT後的rows直接套進去多條件 */
WHERE Col2 BETWEEN 0 AND 100              /* between兩值之間 */
WHERE Col3 like N'%某非英文條件%'           /* 尋找相似字詞，N代表unicode(可存取非英文字元)、%代表任意值 */
WHERE LEFT(Col3, 3) <> '123'              /* 字串左3個"不等於"123 */ 
WHERE DateCol BETWEEN '2020-07-10 00:00:00' AND '2020-07-11 00:00:00' /* 在某個時間區間內 */
```
目標函式：`LEFT(*, n)` `RIGHT(*, n)` `ISNULL(*, '')`(取代null為'')    <br>
判斷符： `<>` `like '%XXX_'` `is not null` `BETWEEN * AND *`  <br>

### GROUP
```SQL
/* 以Goods為組別，進行同組別計算 */
SELECT [Goods], SUM([Count])
FROM [DB]
GROUP BY Goods

-- 如果要細分更多層組別，可GROUP BY多個類別
SELECT [Goods], [Season], SUM([Count])
FROM [DB]
GROUP BY Goods, Seaso
-- 此例即可列出各Goods在各Season的總和
```

### HAVING
```SQL
SELECT [Goods], SUM([Count]) as CountSum
FROM [DB]
GROUP BY Goods
HAVING CountSum > 100      /* 當函數結果需要篩選時，使用HAVING並置底 */
```

### ORDER
```SQL
ORDER BY Col1 DESC        /* ASC為升冪 */
```

### UNION
```SQL
/* 合併同格式表(rbind)，並合併重複資料 */
(SELECT...)
UNION
(SELECT...)

/* 合併同格式表(rbind)，不管有無重複資料 */
(SELECT...)
UNION ALL
(SELECT...)
```

<br>

## 特殊動作

### CREATE TABLE
**新增表格**
```SQL
CREATE TABLE NewTableName
(ColName1 char(5),
 ColName2 varchar(10), ...)
 
 --從現有表格複製
SELECT *
INTO New_Table_Name
FROM Old_Table_Name

--創表格並可設定新欄位
CREATE TABLE NEW_TBL AS
SELECT Col1, Col2, Col3, 'Newcol' as Col4
FROM OLD_TBL;
```
 
### ALTER TABLE
**修改表格**
#### * 新增欄(Col) <br>
```SQL
ALTER [DB]
ADD 'NewColName' nvarchar(max)  /* 最後一欄為欄位格式 */
```

| 資料類型(n=Int\|max) | 意義 | 占用資源 | 
| --- | --- | --- | 
|`char(n)` | 指定n字元  | byte| 
|`nchar(n)` | 指定n字元+萬國編碼| byte\*2| 
|`varchar(n)` | 可變動n字元內| +2byte| 
|`nvarchar(n)` | 可變動n字元內+萬國編碼| byte\*2+2| 

`int` `smallint` `date` `datetime` `bit` ``<br> <br>

#### * 刪除欄(Col)
```SQL
ALTER [DB]
DROP 'OldColName'
```

### DROP TABLE
**刪除表格**
```SQL
DROP TABLE OldTableName
```

### TRUNCATE TABLE
**重置表格** <br>
相比delete效能較佳，能刪除該表所有資料，並重置識別編號，但會保留資料表結構及其欄位、條件約束、索引等
```SQL
TRUNCATE TABLE OldTableName
```

 <br> <br>
## 技巧
### WITH
**子查詢 (多層簡化-可優先查詢)**
```SQL
/* 需置於整個SQL最前頭處 */
WITH  MyTable AS (SELECT...),
      MyTable2 AS (SELECT...)
/* 供以下使用 */
SELECT a.[Col1] b.[Col2] c.[Col3]
FROM [DataBase] a
LEFT JOIN MyTable b ON ...
LEFT JOIN MyTable2 c ON ...
```
相當於多層語法
```SQL
SELECT a.[Col1] b.[Col2] c.[Col3]
FROM [DataBase] a
LEFT JOIN (SELECT...) b ON ...
LEFT JOIN (SELECT...) c ON ...
```

### OVER
**分組處理 (可用於pick組別中最新者、組別加總或計數等)**
* 分組新增序號
```SQL
SELECT [Goods], [Count], [Date],

       ROW_NUMBER() OVER (PARTITION BY [Goods]      /* [Goods]中分割成各群組；ROW_NUMBER() 為新增序號 */
                              ORDER BY [Date] DESC) /* 各群組中再依據[Date]進行排序 */
                      AS ROW_NUM
```

結果如下： 

| Goods | Count | Date | ROW_NUM |
| --- | --- | --- | --- |
| 1 | 666 | 20190101 | 1 |
| 1 | 300 | 20180101 | 2 |
| 1 | 123 | 20170101 | 3 |
| 2 | 1 | 20190101 | 1 |
| 2 | 0 | 20180101 | 2 |

(最後加入`HAVING ROW_NUM=1`即可得各分組最新資料) <br>

* 分組加總
```SQL
SELECT [Goods],

       SUM(Count) OVER (PARTITION BY [Goods])      /* [Goods]中分割成各群組，並進行[Count]的SUM加總 */
                      AS SUM_Count                 /* SUM COUNT AVG  MAX MIN 等函數皆可以使用 */
```

結果如下： 

| Goods | SUM_Count |
| --- | --- |
| 1 |1089 |
| 2 | 1 |

### DATETIME 時間操作
#### 相關函式
取得日期參數 <br>
`YEAR()` `MONTH()` `DAY()` <br>
指定日期格式 <br>
`'2020-01-01 00:00:00'`

#### 切換最小單位
##### (EX: YYYY/MM/DD 12:34 → YYYY/MM/DD 00:00)
適用於計算單日/單月/單年的COUNT、SUM等
```SQL
dateadd(day,0,datediff(day,0, myDate))  
-- datediff計算julian date到myDate之間的"天數"
-- dateadd將這些"天數"從julian date加回來 (即取到00:00)
```
  
### Identity 自動流水號欄
`資料表右鍵` → `設計` → `點選目標欄` → `識別規格` → `(為識別) 是` → `種子、增量設為1` → `儲存`  <br>
此欄之後新增皆會+1，成為流水號 <br>
 
### Primary Key 設定主索引鍵
`資料表右鍵` → `設計` → `點選目標欄` → `(右鍵) 設定主索引鍵`  <br>
此欄會成為主索引(primary key)，為非null且不重複 <br>
 
### 取得欄位重複row數量
```SQL
SELECT Name, Grade, COUNT(*)
FROM TABLE
GROUP BY Name, Grade 
HAVING COUNT(*) > 2
```
可找出重複的ROW的資訊 <br>

### 取得DB資訊
```SQL
-- 取得TABLE列表
  SELECT *
  FROM INFORMATION_SCHEMA.TABLES
  
-- 取得Column資訊
  SELECT *
  FROM INFORMATION_SCHEMA.COLUMNS
  WHERE TABLE_NAME = 'M'
```
### 宣告臨時變數
DECLARE
```SQL
DECLARE @VAR_NAME, ...

-- 之後語法就可以直接使用@VAR_NAME
```
### 轉置合併多欄為單格(FOR XML PATH)
|Gender|Student_Name|
|--|--|
|Male|Timm|
|Female|Vivian|
|Femail|Ariel|

|Gender|Student_Count|Student_Name_List|
|--|--|--|
|Male|1|Timm|
|Female|2|Vivian,Ariel|
```SQL
WITH Student_Gender_Name AS (
	SELECT Gender, Student_Name
	FROM Student_Gender
)
SELECT t.Gender, COUNT(*) AS Student_Count,
	STUFF(  (SELECT DISTINCT ',' + b.Student_Name
	         FROM Student_Gender_Name b
           WHERE a.Gender = b.Gender
           FOR XML PATH (''))  , 1, 1, '') AS Student_Name_List
FROM (
	SELECT *
	FROM Student_Gender_Name a
) as t
group by t.Gender
```
※ STUFF(targetColumn, startCharNum, charLength, replaceChar)為中間插入/取代的用法，此處用於移除第一個字元','

## 其他
### 函數：
`AVG` `COUNT` `MAX` `MIN` `SUM` <br>
`ISNULL(ColumnName, 'nullToNewString')` `LEFT(ColumnName, n)取得字串左邊n個字` `RIGHT(ColumnName, n)取得字串右邊n個字` <br>
`CHARINDEX('_', Column)取得指定字串位置` `SUBSTRING(ColumnName, start, n)切取start字串後n個字` `LEN(ColumnName)取得字串長度`

### 運算：
`ROUND(ColumnName, n)` 四捨五入n位數 <br>
`CAST(ColumnName AS NewFormat)` 切換變數型態為其他型態 (ex: int → float)
