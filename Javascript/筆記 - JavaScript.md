[JavaScript](#JavaScript) </br>
[JQuery](#JQuery) </br>
[ReactJS](#ReactJS) </br>
[AngularJS](#AngularJS) </br>
[Vue](#Vue) </br>

# JavaScript
- 載入與初始化 (載入外部套件可使用網址)
```html
<script type="text/javascript" src="~/Scripts/JS套件.js"></script>
```

- => (簡化function) </br>
**單行簡化*- (無x則為空括號)
```javascript
var func = x => x+1
// 原型為
var func = function(x) return(x+1)
```
**多行簡化**
```javascript
var func = x => {
  var calc = x+1; 
  return calc
  }
// 原型為
var func = function(x) {
  var calc = x+1; 
  return calc
}
```

- HTML DOM控制

| document | 說明 |
| --- | --- |
| `.getElementByName()` | 取得DOM |
| `.getElementsById()` | 取得DOMs (Array) |
| `.getElementById().value` | 取得值 |
| `.CreateElement()` | 創造DOM |
| `.evaluate(xpath, document, null, XPathResult.FIRST_ORDERED_NODE_TYPE, null).singleNodeValue` | 以Xpath搜尋第一個DOM節點 |
| `.addEventListener("input", function() {});`| 當input改變時執行 |
| －－－網址控制－－－ | |
| `.location.href` | 完整URL |
| `.location.host` | 域名+端口號 |
| `.location.hostname` | 域名 |
| `.location.hash` | #後部分 |
| `.location.search` | ?後部分 |
| `.location.reload()` | 刷新/開新網頁 |
| `.location.replace(URL)` | 開新網頁 |

- 下載功能
```javascript
var a = document.createElement('a');    // Create hyperlink for download
var url = "下載網址在這裡";
a.href = url;
a.download = url;
a.click();
```

## 資料處理
### Object
|語法|說明|
|--|--|
| `Object.assign({}, ori, new)`     | 不覆蓋情況下繼承(與覆蓋同key值)到原本的object，產生新的object |
| `Object.values(obj)`              | 獲得obj內值組成的Array  |
| `Object.keys(obj)`                | 獲得obj每個key值組成的Array  |
| `Object.keys(obj).length === 0 && obj.constructor === Object)`| 確認obj為空的Object |
### Array
|語法|說明|
|--|--|
| `Array.from(obj)`                 | 轉換obj類陣列資料並產生新的Array |
| `myArray.map((item, [index])=>{ })` | 迭代myArray每列並回傳結果為Array，包含各列元素(item)與編號(index) |
| `myArray.concat(myArray2)`        | 列合併兩個Array |
| `myArray.find(x => x === value)`  | 查詢Array中元素 |
| `myArray.slice([begin], [end])` | 取得begin到end之間的Array |
| `myArray.splice(begin, [deleteNum], [newArrayItem])` | (會改變原本myArray)取代begin後的array items，替換為新的，並回傳"被切掉"的部分 |
| `myArray.split(seperator, [num])` | 以符號去切割字串 |

## ES6
- let   可變變數，且只存在該循環中(local)，不會更改原變數記憶體 <br>
- const 不可變變數，不會更改原變數記憶體


# JQuery
- 第一次使用須於html引用
```html
<script src="~/Scripts/jquery-3.3.1.min.js"></script>
```
- 網頁載入完成時直接執行
```javascript
$(document).ready(function(){
  // TYPE SCRIPT HERE
});
```
或簡寫為：
```javascript
$(function(){ 
  // TYPE SCRIPT HERE 
});
```
- $(this)
$('...').on(...)的事件中，<br>
可使用$(this)指到目前的DOM

### 基本功能

| $("#id") | 說明 |
| --- | --- |
| `.change()` | 值改變時觸發 |
| `.val()` `.val("")` | 取得值/改變值 |
| `.click()` | 直接點擊/被點擊時 |
| `.select()` | 直接點擊/被點擊時 |
| `.empty()` | 清空子節點 |
| `.remove()` | 刪除該節點 |
| `.before()` `.after()` | 插入HTML項目到該節點前/後 |
| `.prepend()` `.append()` | 插入HTML子項目該節點最前/後 |
| `.contents` | 填入內容 |
| `.find` | 尋找 |
| `.show` | 顯示 |
| `.hide` | 隱藏 |
| `.children('...')` | 子元素DOM |
| `.parent('...')` | 父元素DOM |
| `.first('...')` | 第一個DOM |
| `.siblings('...')` | 平行DOM遍歷 |
| `.prev('...')` | 向前平行DOM |
| `.next('...')` | 向後平行DOM |
| `.each(function() { ... })` | 每個DOM都... |
| `.eq(n)` | 第幾個DOM(index) |
| `.closest('...')` | 找最近的DOM |
| `.addClass('...')` | 新增class |
| `.hasClass('...')` | 判斷DOM是否含有class name |
| `.removeClass('...')` | 刪除class |
| `.trigger()`| 觸發事件 |
| `.prop('prop_name', new_value)`| 更改property內容 |
| `.attr('attribute_name', new_value)`| 更改attribute內容 |
| `.removeAttr('attribute_name')`| 移除attribute內容 |
| `.valid()` | 驗證form |

### IIFE (Immediately Invoked Function Expression)

- 頁面載入完成

```javascript
(function( window, undefined ) {
  // ...
})( window );
```

### AJAX

- .ajax
- [.getJSON](https://www.runoob.com/jquery/ajax-getjson.html)


觸發動作後，須在括號內加入`function() {...}`

### 套件與功能

#### 表格
- [DataTables](https://datatables.net/) 動態表格
- [Dynatable](https://github.com/alfajango/jquery-dynatable) 動態表格
- [igGrid](https://www.igniteui.com/help/iggrid-overview) 功能齊全的Infragistics系列-動態表格

#### 通知
- [Dialog](https://jqueryui.com/dialog/) 彈出視窗
- [Bootstrap Toasts](https://getbootstrap.com/docs/5.0/components/toasts/) 彈出小窗格

#### 選單
- [Collapse (bootstrap)](https://www.runoob.com/bootstrap/bootstrap-collapse-plugin.html) 開闔式按鈕
- [zTree](http://www.treejs.cn/v3/main.php#_zTreeInfo) 樹狀圖選單


#### Input
- 使用`<input type="file" id="file">`在前端取得檔案

  ```javascript
  $('#file').change( function(e) {

#### 其他
- [Dialog](https://jqueryui.com/dialog/) 彈出視窗(JQuery)

  ```html
  <script>
  $( function() {
    $( "#dialog" ).dialog();
  } );
  </script>
 
  <div id="dialog" title="Basic dialog">
    <p>This is the default dialog which is useful for displaying information. The dialog window can be moved, resized and closed with the &apos;x&apos; icon.</p>
  </div>
  ```

- [PNotify](https://sciactive.com/pnotify/) 彈出通知
  
  ```js
  new PNotify({ text: "Something wrong!", type: 'error', styling: 'bootstrap3' });
  ```

- [toastr](https://github.com/CodeSeven/toastr)

  ```js
  toastr.success("success","");
  toastr.info("info","");
  toastr.error("error","");
  toastr.warning("warning","");
  ```

- [DataTables](https://datatables.net/) 動態表格 (JQuery)
  
  ```html
  <script>
  $('#myTable').dataTable({
      destroy: true,
      serverSide: true,
      searching: false,
      ordering: true,
      scrollX:true,
      displayStart: 0,
      ajax: {
          url: '...',
          type:"POST",
          data : { query: "keyword" },
          datatype: 'json'
      },
      rowCallback: function( row, data, index ) {
          // 根據資料做客製
          $('td',row).css("background-color", "white");
      },
      order: [[ 1, 'asc' ]],
      columns: [
          {
              orderable: false,
              render: (data, type, row)=>{
                  return '<a class="btn btn-round btn-primary" type="button" href="/myAction/Edit?Id='+row.id+'">Edit</a>';
              }
          },
          { data: 'myviewmodel.nesteddata.id', name: 'Id' }
          { data: 'myviewmodel.nesteddata.name', name: 'Name' }
      ]
  });
  </script>

  <table id="myTable" class="table table-striped table-bordered" width="100%" style="white-space: nowrap;">
      <thead>
          <tr>
              <th></th>
              <th>Id</th>
              <th>Name</th>
          </tr>
      </thead>
  </table>
  ```

- [Dropzone](https://www.dropzonejs.com/) 拖拉式上傳
- [Collapse (bootstrap)](https://www.runoob.com/bootstrap/bootstrap-collapse-plugin.html) 開闔式按鈕
- [zTree](http://www.treejs.cn/v3/main.php#_zTreeInfo) 樹狀圖選單
- [tokenInput](https://loopj.com/jquery-tokeninput/) 搜尋建議標籤 
- [redactor](https://imperavi.com/redactor/) HTML編輯器

### 功能

- 使用`<input type="file" id="file">`在前端取得檔案

  ```javascript
  $('#file').change(function(e) {
    var uploadedFiles = e.target.files
    var filename1 = uploadedFiles[0].name
  });
  ```

- [tokenInput](https://loopj.com/jquery-tokeninput/) 搜尋建議標籤 
- [Dropzone](https://www.dropzonejs.com/) 拖拉式上傳
- [igCombo](https://www.igniteui.com/help/igcombo-overview) 功能齊全的Infragistics系列-搜尋與下拉
- [Dual-List-Selector](https://www.patternfly.org/components/dual-list-selector/) 左右清單選擇器

#### 跨頁溝通

- sessionStorage
  - Session Storage可達到跨頁間的資訊互傳
  - 分頁間不互通
  - 只能儲存簡單型別(建議為JSON)

  ```javascript
  var dataJson = sessionStorage.getItem('target.data.key');
  var data = JSON.parse(dataJson);
  sessionStorage.setItem('target.data.key', JSON.stringify(data));
  ```

- BroadcastChannel
  - 用於跨頁間的Event Trigger

  ```javascript
  var broadcastChannel = new BroadcastChannel('sample.channel');
  broadcastChannel.onmessage = function (event) {
      debugger;
      if (event.data.eventName === "method1") {
          someFunction();
      }
  };

  broadcastChannel.postMessage("method1");
  ```

#### 擴充功能
- [redactor](https://imperavi.com/redactor/) HTML編輯器


# [ReactJS](https://github.com/timmchentw/Web-Note-Text/blob/master/%E7%AD%86%E8%A8%98%20-%20JS%20React.md)
# AngularJS
# Vue


# Storage

- LocalStorage: 用於儲存長期客戶端常使用的資料
- SessionStorage: 用於儲存"各Tab"客戶端常使用的資料 (關閉Tab即會清除)