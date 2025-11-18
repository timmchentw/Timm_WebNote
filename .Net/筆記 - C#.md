# C# 筆記

- [C# 筆記](#c-筆記)
  - [**《字串處理》**](#字串處理)
    - [檔案字串](#檔案字串)
    - [Regex (Regular Expression字串格式檢查)](#regex-regular-expression字串格式檢查)
      - [語法](#語法)
      - [Email檢查](#email檢查)
  - [變數型態](#變數型態)
    - [基本型態](#基本型態)
    - [資料結構](#資料結構)
    - [資料Interface](#資料interface)
  - [修飾詞](#修飾詞)
  - [Property與Field](#property與field)
  - [建構子 Contructor](#建構子-contructor)
  - [靜態修飾 Static](#靜態修飾-static)
  - [繼承性 Inheritance](#繼承性-inheritance)
  - [修飾詞](#修飾詞-1)
  - [運算子](#運算子)
  - [判斷式與迴圈](#判斷式與迴圈)
    - [**IF**](#if)
    - [**Null Operator**](#null-operator)
    - [**SWITCH**](#switch)
    - [**FOR**](#for)
    - [**FOREACH**](#foreach)
    - [**TRY \& CATCH**](#try--catch)
    - [IS](#is)
    - [HTML處理](#html處理)
      - [HtmlAgilityPack](#htmlagilitypack)
    - [檔案讀寫](#檔案讀寫)
  - [注意事項](#注意事項)
  - [Cache快取](#cache快取)
  - [錯誤處理](#錯誤處理)
  - [Recursion (遞迴)](#recursion-遞迴)
  - [Delegate](#delegate)
  - [Expression](#expression)
  - [Yield](#yield)
  - [Class](#class)
    - [Abstract Class \& Interface](#abstract-class--interface)
    - [Partial Class](#partial-class)
  - [Lock](#lock)
  - [網址與路徑](#網址與路徑)
    - [Path](#path)
    - [Directory \& File](#directory--file)
  - [Stream](#stream)
  - [Expression-bodied](#expression-bodied)
    - [Expression-bodied Methods](#expression-bodied-methods)
      - [Expression-bodied members](#expression-bodied-members)
  - [Asynchronize](#asynchronize)
    - [async/await](#asyncawait)
  - [Reflection](#reflection)
    - [typeof](#typeof)
    - [Property](#property)
    - [Change type](#change-type)
    - [Attributes](#attributes)
    - [CreateInstance](#createinstance)
  - [Generic](#generic)
    - [where](#where)
  - [Assembly](#assembly)
  - [Url](#url)
  - [Comment](#comment)
    - [Documentation](#documentation)
  - [Extension](#extension)
  - [Reference:](#reference)

* [用Dict去做掃描比LINQ Where速度快上10倍以上](https://blog.darkthread.net/blog/linq-search-performance-issue)

## **《字串處理》**

| 功能 | 語法 |
|-----|-----|
| 黏合字串與變數  |  `$"ABC....{Var or "string"}...."` |
| 黏合字串與變數2 | `$"...{0}...{1}...{2}...", var1, var2, var3` |
| 黏合字串與變數3 | `"String" + Var` |
| 字串中的""    |  `@"....\"......\"..."` |
| 字串中的{}    |  `@"....{{......}}..."` |
| 字串中的IF    |  `$"....{(Var==1 ? "string1": "string2")}"` |
| 字串長度      |  `{Var.Length} -> "3"` |
| 取代          |  `.replace(OldChar. newChar)` |
| 去除頭尾      |  `.trim()`    |
| 去除尾字      |  `.trimEnd(myChar)` |
| 去除前字      |  `.trimStart(myChar)` |
| 去除特定區間  |  `.remove(ref_pos, char_num)` |
| 分割         |  `.split(myChars)` |
| 檢查         |  `.contains("something")` |
| 轉換大小寫    |  `.ToUpper`  `.ToLower`  `.TitleCase`(字首大寫) |
| 轉為數字      |  `int.Parse("String")` |
| 檢查是否為空  |  `string.isNullorEmpty("String")` |

### 檔案字串

| 功能 | 語法 |
|-----|-----|
| 取得檔名(不包含附檔名)  |  `Path.GetFileNameWithoutExtension(filePath)` |
| 取得副檔名             | `Path.GetExtension(filePath)` |
| 資料根目錄             | `Path.GetPathRoot(filePath)` |
| 取得路徑               | `Path.GetFullPath(filePath)` |


### Regex (Regular Expression字串格式檢查)

#### 語法

| 功能 | 語法 |
|-----|-----|
| 允許範圍 | `[a-z]` `[0-9]` |
| 字元集合 |  `(...)` |
| |
| 首字為a    |    `^a` |
| 尾字為a    |    `a$` |
| 前面有a可出現1次以上 | `a+` |
| 前面有a可出現1次以下 | `a?` |
| 前面有a可出現或不出現 | `a*` |
| 旁邊出現A     |  `A\b` |
| 旁邊不出現A    | `A\B` |
| 任何字元(\n外) | `.` |
| 任何字元(含\n) | `[\s\S]` |
| 任何英文字元   | `\w`  |
| 任何非英文字元 | `\W` |
| 任何空白字元   | `\s` |
| 任何非空白字元 | `\S` |
| 任何數字字元   | `\d` (包含"一二...") |
| 任何非數字字元 | `\D` |
| |
| ab以外所有   |  `[^ab]` |
| abc擇一即可  | `[abc]` |
| abcd中排除bc  | `[[a-d]-[bc]]` |
| a或b或c       | `(a\|b\|c)` |
| |
| a重複n次    |  `a{n}` |
| a重複n到m次  | `a{n,m}` |
| a至少重複1次 | `a{n,}` |
| |
| 所有大寫字母 |   `(?-i:\p{Lu})` |
| 所有非大寫字母 | `(?-i:\P{Lu})` |
| 檢查中文字母 |   `\p{IsCJKUnifiedIdeographs}` |

| **特殊符號** | 語法 |
|-----|-----|
| (     |     `\(` |
| /     |    `//` |
| .     |     `\.` |
| +     |     `\+` |
| ?     |     `\?` |
| *     |     `\*` |
| 後退鍵 |     `[\b]` |
| 換行   |     `\r\n` |
| TAB    |    `\t` |

| 表達 | 語法 |
|-----|-----|
| 表示式 |    `var regex = /expression/` |
| 進行檢查 |  `regex.test("string")` |

#### Email檢查

`^ [a-zA-Z0-9._%+-]+@ [a-zA-Z0-9.-]+ (. [a-zA-Z] {2,}|. [0-9] {1,})$`

* `^` 這個符號表示要匹配字符串的開始位置，也就是說，只有當字符串的第一個字符符合後面的條件時，才能繼續匹配。
* `[a-zA-Z0-9._%+-]+ `這個部分表示要匹配一個或多個的字母、數字、下劃線、點號、百分號、加號或減號。這些字符都可以作為email地址的用戶名的部分。 **(`%` `+` 兩者可能部分系統不支援)**
  * `[ ]` 表示一個字符集合，裡面的字符可以任意選擇一個。
  * `+ `表示至少要有一個，可以有多個。
* `@ `這個符號表示要匹配@字符，這是email地址的必要部分，用來分隔用戶名和域名。
* `[a-zA-Z0-9.-]+ `這個部分表示要匹配一個或多個的字母、數字、點號或減號。這些字符都可以作為email地址的域名的部分。注意，這裡沒有下劃線、百分號、加號等字符，因為它們不是有效的域名字符。
  * `. `這個符號表示要匹配一個點號，這是email地址的必要部分，用來分隔域名和頂級域名（TLD）。
* `[a-zA-Z] {2,}|. [0-9] {1,} `這個部分表示要匹配兩種可能的情況之一：一種是兩個或更多的字母，另一種是一個或更多的數字。這些都可以作為email地址的頂級域名（TLD）的部分。
  * `| `表示或者的意思，表示要選擇左邊或者右邊的條件之一。
  * `{ } `表示重複次數，裡面的數字表示最少和最多的次數。如果只有一個數字，表示最少和最多都是那個數字。如果有兩個數字，用逗號隔開，表示最少和最多分別是那兩個數字。如果只有一個數字，後面有逗號，表示最少是那個數字，最多沒有限制。
* `$ `這個符號表示要匹配字符串的結束位置，也就是說，只有當字符串的最後一個字符符合前面的條件時，才能完成匹配。

## 變數型態

### 基本型態
* 數字邏輯: `int` `float` `double` `decimal`(指定位數) `bool`(True & False)
* 其他: `string` `char` `object` `interface` `delegate (action, func)`

### 資料結構
| 型態 | 名稱 | 特性 | 範例|
|-----|-----|-----|-----|
| Array | 陣列 | 須宣告型別+固定資料長度 | `int[] newArray = new int[3]{777,666,123};` |
| List | 列表 | 須宣告型別+可插入(.Insert)移除(.Remove) | `List<string> newList = new List<string>();`  <br>  `newList.Add("newString");` |
| Span | 跨度 | 高効、連續Memory(Stack)、直接指向而非複製、適用長文字 | `{ 1, 2, 3 }.AsSpan()` |
| Datatable | 資料表 | 複合資料型態+分別插入Col與Row | `DataTable dt = new DataTable();`  <br>  `dt.Columns.Add("cName", typeof("string"));`  <br>   `Datarow row = dt.NewRow();`  <br>  `row["cName"] = ...`  <br>  `dt.Rows.Add(row)`|
| Stack | 堆疊 |  |  |
| Queue | 佇列 |  |  |
| Dictionary | 字典(雜湊表) | 快速Lookup |  |
| Hashset | 雜湊集合 | 可自動忽略重複的Add to list元素(同Dictionary的Key) | `HashSet<string> keys = new(["aaa", "bbb"], StringComparer.OrdinalIgnoreCase);  // Comparer先加入在後面的Key compare就可以內建使用` </br> `bool isKeyExist = keys.Contains("aaa");` |
| ConcurrentDictionary | 跨執行緒字典 | 可避免thread衝突 |  |
| Tuple | 多參數 |||
| struct | class參數 |  | MyClass.MyProp |
| Enum | 列舉 |  |  |
| NameValueCollection |  |  |  |
| Delegate | 委派 | 最為彈性(不須指定型別) |  |
| Action | 輸入委派 | 定義輸入型別的委派 |  |
| Func | 輸入輸出委派 | 定義輸入輸出型別的委派 |  |

### 資料Interface

| 型態 | 名稱 | 特性 | 範例|
|-----|-----|-----|-----|
| IEnumerable |  | 最簡便的多值物件規範(最不嚴謹，注意set value可能不可用) |  |
| ICollection |  | 擴充IEnumerable，如Add(), Remove(), Contains() |  |
| IList |  | 擴充ICollection，如Insert(), RemoveAt(), this[] (注重順序、index效能等) |  |

## 修飾詞

* 存取授權

| 存取類別 | 說明 |
| ----- | ----- |
| public | 可公開存取 |
| private | 同class可存取 (不可被引用或呼叫)| 
| protected | 同class 或 原class衍伸之class可存取 | 
| internal | 同project可存取 |  

* 返回型別: `void` 物件無輸出 (不需return)、`Task` (搭配async不須output)
* `static` 靜態修飾 (可直接呼叫不需new、可防止改動、同class可供全部共用、缺點為input參數無法共享、注意記憶體使用量與全域共用修改的問題)
* `async` 非同步方法，務必在方法內包含至少一個await (async method可呼叫sync method，反之無法，因此建議code都使用async方法為呼叫源頭)
* `abstract`抽象方法(不可實作)、`virtual`(可複寫方法)、`override`(已覆寫方法)

## Property與Field

* Field: 為Class變數，通常為Private (小心有順序呼叫問題)

  ```C#
  private string _something;
  ```

* Property: 封裝Field或其他數據，有get/set accessor，簡化操作Field的方法

  ```C#
  public string Something { get; set; }

  // 也可以設定Accessibility
  public string SomethingRestrictSet { get; private set; }

  // 與Field互動，達到內外操作分離
  public string SomethingPublic
  {
    get { return _something; }
    set
    {
      if (_something == null)
        _something = value;
    }
  }

  // 限制只能由constructor來做set
  public string SomethingOnlyCtorCanSet { get; init; }

  // Expression-bodied Members (簡化getter/setter)
  public string SomethingSimplier
  {
    get => _something;
    set => _something = value;
  }

  // Indexers
  // 用法如: var collection = new SomeCustomCollection<string>();
  //         collection[0] = "Hello world";
  public class SomeCustomCollection<T>
  {
    private T[] _array = new T[100];
    public T this[int i]
    {
      get { return _array[i]; }
      set { _array[i] = value; }
    }
  }

  // Static (適合一些Singleton情境，因記憶體只存一處，注意Async問題，慎用!!!)
  public static string SomethingStatic { get; set; }

  // Abstract
  public abstract string SomethingNeedToImplement { get; set; }
  ```

## 建構子 Contructor

1. 無output，可input (有即強迫使用者輸入input)
2. 與class同名，且無型別
3. 可直接執行與賦值
4. input可overloaded (創很多個constructor，每個input都不一樣)
5. 可直接access readonly property

```C#
public class MyClass
{
  private string _input { get; }

  public MyClass(string input)
  {
      _input = input;
  }
}
```

## 靜態修飾 Static

**(1) 可直接呼叫**

```C#
class Math
{
  public /**/static/**/ int max(int a, int b)
  {
    int result;
    if (a < b) result = b;
    else result = a;
    return result;
  }
}
```

使用時不必new新物件(Math m = new Math)  <br>

```C#
int a = 10;
int b = 20;
int c = Math.max(a,b)
```

**(2) 同class可供全部共用**  <br>

於class Student中 (如[建構子](#建構子)所示)  <br>

```C#
// 加入靜態宣告
public static int PassGrade = 2;
```

```C#
// 當使用此class時，更改靜態宣告值
Student.PassGrade = 1; 
```

```C#
// 更改結果可供全class使用
s1.isPass() -> True
```

p.s.

```C#
在Class Student中新增:
pulic bool isPass()
{
  if (Grade >= PassGrade) return true;
  else return false
}
```


## 繼承性 Inheritance

```C#
class B : A         //B繼承A之屬性(內建變數、Function等)
{...}
```

* 

## 修飾詞

| Base基底修飾 | 說明 |
| -- | -- |
| `abstract` | 未具現化，需實作才可使用 (not instantiated) |
| `virtual` | 可直接具現化，並可以被修改覆蓋 |

| Derived衍伸修飾 | 說明 |
| -- | -- |
| `override` | 覆蓋原有base class的功能 (`BaseClass bc = new DeriveClass()`也同樣適用) |
| `new` | 不覆蓋原有base class的功能，但可以做擴展 |

`sealed` 防止被繼承衍伸 <br>

[微軟範例](https://docs.microsoft.com/zh-tw/dotnet/csharp/programming-guide/classes-and-structs/knowing-when-to-use-override-and-new-keywords) <br>

## 運算子

| 運算子 | 意義 |
| --- | --- |
| `++` `--` | ±1 | 
| `&&` `\|\|` | AND / OR  |
| `==` `!=` | 等於 / 不等於  | 
| `+=` `-=` | 原變數再加/減多少 |
| `/` `%` | 除取商數/餘數 |
| 其他運算子 | 意義 |
| `..` | Range operator，為指定序列範圍，如`0..5`為指定0到5 |
| `^` | Index from end operator，為序列結尾起算，如`^3..^1`為指定倒數3到1 |

## 判斷式與迴圈

### **IF** 

`if (!var)` → `if(int==0)` or `if(string==null)`  <br>
!代表相反之涵義

### **Null Operator**

* `? :` 簡化條件式為單行

```C#
// 宣告a整數值，當b>0時則a=b，否則為a=c
int a = (b > 0) ? b : c;
// 原始如下:
int a;
if (b > 0)
  a = b; 
else
  a = c;
```

* `?.` 允許取得Null Property (Null Condition Operator)

```C#
// 允許取得Null instance的Property (會返回null)
string val = myClass?.Value;
// 原始如下:
string val = null;
if (myClass != null)
  val = myClass.Value;
```

* `??` 簡化檢查null值 (null-coalescing)

```C#
// 宣告a整數值，當b為非null時a=b；當b為null時a=c；當c為null時a=d...
int a = b ?? c ?? d;
// 原始如下:
int a = b;
if (b == null)
  a = c;
if (c == null)
  a = d;
```

* `??=` 簡化Null Set Value (null-coalescing assignment)

```C#
string val = null;
string val2 = val ??= "default";
// 原始如下:
string val = null;
if (val == null)
  val = "default";
string val2 = val;
```

### **SWITCH**

簡化版IF

```C#
switch (var)
{
  case 1: ...
    break;
  case 2: ...
    throw ...;
  default : ...
    return ...;
}
```

### **FOR**

```C#
for (int i = 0; i < 10; i++)
{
  i ;
}
```

### **FOREACH** 

將LIST逐層處理

```C#
foreach (var item in a_LIST)
{
  item ;
}
```

### **TRY & CATCH**

嘗試執行與失敗接住

```C#
try
{
    //try something here
}
catch (Exception e)
{
    //go to here if error occured
    console.log(e)   // Explain the error or logging
}
catch (Exception e) when (e.Message.Contains("500")))
{
    // Conditional catch
}
finally 
{
    //any condition will go to here. Such as dispose
}
```

### IS

用於判斷型別 (可在後面直接命名轉型物件)

```C#
if (exception is ArgumentException argEx)
{
  Console.WriteLine(argEx.Message);
  // ...
  // 等於
  // ArgumentException argEx = exception as ArgumentException 
}
```

### HTML處理

#### HtmlAgilityPack

* HtmlDocument

```C#
HtmlDocument doc = new HtmlDocument();
doc.Load('File Path String')
// doc.LoadHtml('Html String')
```

* HtmlNode
  
```C#
HtmlNode node1 = doc.CreateTextNode('Html Node String');
doc.DocumentNode.AppendChild(node1);  // 對整個HTML插入子節點 (PreoendChild / AppendChild)

HtmlNode node2 = doc.SelectSingleNode('XPATH');
node2.ParentNode.InsertBefore(node1);  // 選定節點並插入子節點 (InsertBefore / InsertAfter)
```

* 取出Html (String)
`doc.DocumentNode.InnerHtml`

### 檔案讀寫

* StreamReader / StreamWriter  `讀寫文字檔`

```C#
StreamReader sr = new StreamReader(filepath, Encoding.UTF8);   // Read HTML file
string myString = sr.ReadToEnd();
sr.Close();
sr.Dispose();
// 寫入則為：sw.Write(myString)，其他宣告與指令相同
```

* 

## 注意事項

* 使用WebClient載入https連結時，如.net framework版本較舊、憑證版本較新，須先預設較新版本的SSL/TLS憑證

```C#
System.Net.ServicePointManager.SecurityProtocol = System.Net.SecurityProtocolType.Tls12;  // 定義TLS1.2版本

WebClient wc = new WebClient();
MemoryStream ms = new MemoryStream(wc.DownloadData(URL));
HtmlDocument doc = new HtmlDocument();
doc.Load(ms, Encoding.UTF8);
```

## Cache快取

* MemoryCache (存在全域，不隨Http Request而變)
* HttpContext Cache (隨著Http Request而變，可為各個user個別存取)
* ObjectCache 
* Distributed Cache

```C#
List<string> currentCache = HttpContext.Current.Cache["cacheName"] as List<string>
List<string> newCache = new List<string>();

if (!Enumerable.SequenceEqual(currentCache, newCache)) // 自訂時機判斷更新Cache (此例子比較兩個List值是否相同)
{
    // Insert -> 覆寫 Override current cache ;  Add -> 不覆寫 Add cache only when currentCache is null
    HttpContext.Current.Cache.Insert("cacheName",
                                     newCache,
                                     null,
                                     DateTime.Now.AddDays(1),                       // absoluteExpiration -> 指定時間刪除 (通常設定DateTime.Now往後推多久)
                                     System.Web.Caching.Cache.NoSlidingExpiration,  // slidingExpiration ->  最後一次取用後多久刪除 (有存取就會重新計算)
                                     System.Web.Caching.CacheItemPriority.High,     // 設定快取優先度
                                     new System.Web.Caching.CacheItemRemovedCallback(RemovedCallback)); // 刪除快取後的callback行為 (通知程式快取已刪除用)
}
```

```C#
public static void RemovedCallback(string k, object v, System.Web.Caching.CacheItemRemovedReason r)
{
    itemRemoved = true;
    reason = r;
}
```

## 錯誤處理

* [.Net MVC 六種處理exception的方式(包含controller接錯)](http://programming.signage-cloud.org/dotnet_mvc_exception_handle/)
* [請直接Throw而非Throw ex防止資訊丟失](https://www.dotblogs.com.tw/wasichris/2015/06/07/151505)
* [包裝程式中共用的 try...catch(簡化try catch)](https://dotblogs.azurewebsites.net/rainmaker/2014/11/19/147361)
* [建立客製化exception必要步驟](https://rainmakerho.github.io/2018/08/21/2018032/)
* [StackOverFlow正確建立可序列化Exception](https://stackoverflow.com/questions/94488/what-is-the-correct-way-to-make-a-custom-net-exception-serializable)

* 簡化Thorw Exception

```C#
ArgumentNullException.ThrowIfNull(stringToBeCheck, nameof(stringToBeCheck))
```

## Recursion (遞迴)

```C#
public static void RecursionMethod(string input)
{
    RecursionMethod(input);
    //...
}
```

## Delegate

```C#
public static void TryToDo(Func<string, bool> delegateMethod)
{
    string something = "Test";
    try
    {
        bool result = delegateMethod(something);
    }
    catch
    {
        // Log
    }
}
```

## Expression

非Runtime的表示式 (適合用於欄位Mapping、延遲執行等)，常見例子為LINQ

```C#
Expression<Func<string, int>> example = (stringInput => Convert.ToInt32(stringInput));

Expression<Func<MyClass, object>> example2 = (x => x.Prop1);
```

## Yield

分批作業

```C#
public IEnumerable<string> Sample() 
{
    for (i = 0; i < 10; i++)
    {
        yield return i;
    }
}
```

## Class

### Abstract Class & Interface

* Abstract: 允許實作邏輯，並可標記abstract方法，強制Implement class override抽象方法 (適合用於共享部分Protected function)
* Interface: 強制Implement class實作指定Function/Property，並不可含任何Implementation

### Partial Class

* Partial class允許不同的property存在不同檔案
* 適合用於大型開發、並且各自維護部分欄位的狀況
* 不允許相同的property / function存在各partial class裡面

```C#
// File1.cs
public partial class MyClass
{
    public void Method1() 
    {
        Console.WriteLine("Method1");
    }
}

// File2.cs
public partial class MyClass
{
    public void Method2() 
    {
        Console.WriteLine("Method2");
    }
}

// Usage
MyClass myClass = new MyClass();
myClass.Method1(); // Output: Method1
myClass.Method2(); // Output: Method2
```

## Lock

* 鎖定某個物件，使得其他執行緒在lock語法結束前不能做更改
* 如要Lock現有的Class，則可直接建立一個無意義的Object property，避免當下執行緒卡死

  ```C#
  public class ClassToBeLocked
  {
      private static object _lockObj { get; set; }
      
      public void DoSomething()
      {
        lock (_lockObj)
        {
          //...
        }
      }
  }
  ```

## 網址與路徑

### Path

  ```C#
    // 黏合URL Path
    Path.Combine("directory/root/", "/relativePath/file");
    Path.GetFileName("fullpath");
    Path.GetExtension("fullpath");
  ```

### Directory & File

  ```C#
    File.Exists("FilePath");
    File.Delete("FilePath");
    var fileInfo = new FileInfo("FilePath")

    Directory.CreateDirectory("DirectoryPath")
    Directory.Exists("DirectoryPath");
    Directory.Delete("DirectoryPath", recursive: true);
    var directoryInfo = new DirectoryInfo("DirectoryPath");

  ```

## Stream

```C#
var fileStream = new MemoryStream();
await file.CopyToAsync(fileStream);
fileStream.Position = 0;    // 重置讀取位置，方便下一次Stream被讀取時從頭開始 (否則會有Exception)
```

## Expression-bodied

### Expression-bodied Methods

  ```C#
  MyFunction(int a, int b) => a + b;
  ```

#### Expression-bodied members

## Asynchronize

### async/await

* `GetAwaiter`
* `Task.CompletedTask`
* `XXXAsync().ConfigureAwait(false)` <br>
    Why? 參考[About ConfigureAwait](https://medium.com/ricos-note/about-configureawait-5f173cd5f4f)

## Reflection

### typeof

* typeof()
* .GetType() 
* Nullable.GetUnderlyingType(type)

### Property

* typeof(T).GetProperties()


### Change type

* 泛型取得PropertyInfo，再偵測Property型別，轉換成目標型別，設定到該Property上

  ```C#
  TClass result = new TClass();
  object stringData = "123";
  PropertyInfo prop = typeof(TClass).GetProperties[0];
  Type propType = prop.PropertyType.Name.Contains("Nullable") ?
              Nullable.GetUnderlyingType(prop.PropertyType)! : prop.PropertyType;  // Allow to convert to nullable type
  prop.SetValue(result, Convert.ChangeType(stringData, propType));
  ```


### Attributes

* 自製Attribute，參考[官方文件說明](https://learn.microsoft.com/en-us/dotnet/standard/attributes/retrieving-information-stored-in-attributes)

  ```C#
  [System.AttributeUsage(System.AttributeTargets.Class | System.AttributeTargets.Struct)]  
  public class AuthorAttribute : System.Attribute  
  {  
      private string name;  
      public double version;  
    
      public AuthorAttribute(string name)  
      {  
          this.name = name;  
          version = 1.0;  
      }  
  }  
  ```

* InternalVisiableTo (可以允許特定Project/DLL可以存取此Class的Internal相目)

  ```C#
  [assembly: InternalsVisibleTo("MyTestProjectAssembly")]
  namespace MyProductCodeProject
  {
      internal class MyClass
      {
          //...
      }
  }
  ```

### CreateInstance

```C#
Type type = Type.GetType(name, true);
object instance = Activator.CreateInstance(type);
PropertyInfo prop = type.GetProperty(property);
prop.SetValue(instance, value, null);
```

## Generic

### where

規定Generic型別(可多個)

```C#
public static void GenericMethod<T>()
  where T : class 
  where T : enum
{

}
```

## Assembly

* 取得Assembly file
  
  ```C#
  Assembly.GetExecutingAssembly().Location;
  ```

* 取得Assembly 資料夾路徑
  
  ```C#
  Path.GetDirectoryName(Assembly.GetExecutingAssembly().Location);
  ```
  

## Url

* 將Url轉成正規格式：`Uri.EscapeUriString("...")`
* 將整串文字轉成Url Encode格式 (用於放置於Parameter內) `HttpUtility.UrlEncode("...")`

## Comment

* `//`
* `/* ... */`

### Documentation

* 對Class參數進行說明的功能 ```///```
* [C# Documentation: A Start to Finish Guide](https://blog.submain.com/c-documentation-start-finish-guide/)

## Extension

* Static class + static function + this parameter可直接將該型別新增extension方法

```C#
public static class Extensions
{
  public static string GetEnumDescription(this Enum enum)
  {
      FieldInfo? fieldInfo = enum.GetType().GetField(value.ToString());

      var fieldAttribute = fieldInfo?.GetCustomAttributes(typeof(DescriptionAttribute), false);
      if (fieldAttribute != null)
      {
          DescriptionAttribute[] attributes = fieldAttribute as DescriptionAttribute[];

          if (attributes != null && attributes.Any())
          {
              return attributes.First().Description;
          }
      }

      return enum.ToString();
  }
}
```


## Reference:

* [小山的教學平台](https://www.youtube.com/channel/UCmumrs_hb9s6eoVI29gLBgA) (建構子、靜態修飾、繼承性)
* []()
