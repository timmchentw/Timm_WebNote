# ReactJS
DOM控管與簡化、改進效率之架構 <br><br>
[安裝React於MVC5](https://github.com/timmchentw/ReactDemo/blob/master/%E5%AE%89%E8%A3%9D%E8%88%87%E4%BD%BF%E7%94%A8React.md)

## JSX
* 使用Render渲染單一標籤
* 可自定義Attrobites屬性與賦值
* {}內可使用JS語法
* 使用?:判斷式
* 使用樣式變數，再賦予style
* 註解為 { /* ... */ }
* import 功能 from '套件'
* class
* export

### Class & Component
```javascript
class A extends React.Component{
  constructor(){
    super();        // 繼承父物件屬性
    this.state={    // 物件狀態
      "number":"1"
    }
  }
  render() {        // 渲染html
    return (
      <div>
        Number is {this.state.number}
      </div>
    )
  }
}
```
### Render 
```javascript
render() { return () }
```

### Components






Reference: </br>
[激戰ReactJS 30天系列 (洪啟瑞-30天熱度)](https://ithelp.ithome.com.tw/articles/10193503) </br>
[ Setting up a React Environment for ASP.NET MVC (Sung M. Kim)](https://dev.to/dance2die/setting-up-a-react-environment-for-aspnet-mvc-44la) </br>
[激戰ReactJS 【Day03】所需插件的介紹與安裝 (洪啟瑞-30天熱度)](https://ithelp.ithome.com.tw/articles/10193004) </br>
[教學課程：在 Visual Studio 中建立 Node.js 和 React 應用程式](https://docs.microsoft.com/zh-tw/visualstudio/javascript/tutorial-nodejs-with-react-and-jsx?view=vs-2019) </br></br></br>




### 其他資源 </br>
使用NPM指令安裝套件於特定目錄
```
npm install --prefix ./install/here <package>
```
</br></br>

[使用Visual Studio Nuget套件安裝於MVC架構之教學](https://reactjs.net/tutorials/aspnet4.html)
1. NuGet安裝以下三套件：`React.Web.Mvc4` `JavaScriptEngineSwitcher.V8` `JavaScriptEngineSwitcher.V8.Native.win-x64`
2. 註冊JS引擎，於App_Start/ReactConfig.cs添加：
```C#
using JavaScriptEngineSwitcher.Core;
using JavaScriptEngineSwitcher.V8;

    public static class ReactConfig
    {
        public static void Configure()
        {
            // 添加以下兩行
            JsEngineSwitcher.Current.DefaultEngineName = V8JsEngine.EngineName;
            JsEngineSwitcher.Current.EngineFactories.AddV8();
        }
    }
```
3. 欲使用React之View，添加引用：
```html
<script crossorigin src="https://cdnjs.cloudflare.com/ajax/libs/react/16.8.0/umd/react.development.js"></script>
<script crossorigin src="https://cdnjs.cloudflare.com/ajax/libs/react-dom/16.8.0/umd/react-dom.development.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/remarkable/1.7.1/remarkable.min.js"></script>
<script src="@Url.Content("~/Scripts/Tutorial.jsx")"></script>
```
4. 於Scripts資料夾中創建.jsx檔案，即為React物件 </br>

