# .Net Core

- [.Net Core](#net-core)
  - [架構筆記](#架構筆記)
  - [Program \& Startup](#program--startup)
    - [Authentication](#authentication)
    - [Route](#route)
    - [Dependency Injection](#dependency-injection)
      - [註冊服務](#註冊服務)
      - [取得服務](#取得服務)
    - [Use app](#use-app)
    - [Middlewares](#middlewares)
      - [Middleware註冊](#middleware註冊)
      - [Filter註冊](#filter註冊)
        - [Authorize Filter](#authorize-filter)
  - [Environment](#environment)
    - [Environment Variable](#environment-variable)
      - [Set](#set)
      - [Get](#get)
  - [Frontend](#frontend)
    - [Tag Helper](#tag-helper)
  - [Deployment](#deployment)
    - [Linux](#linux)
    - [問題排解](#問題排解)
  - [傳輸協定](#傳輸協定)
    - [gRPC](#grpc)
  - [Roslyn](#roslyn)
    - [Source Generator](#source-generator)


## 架構筆記

1. 如果不使用DI相依性注入，有在Constructer New出來使用的class，在結束使用後記憶體將暫時不會釋出，尤其連線相關的variable，因此最好手動dispose
ps. Controller呼叫new service，new service時也會new DB connection，如果request完service並沒有dispose connection就會出現問題 (unit of work會出現、但不使用unit of work則應該每次呼叫DAL時都會using connection -> using完會自動dispose) </br>
2. Unit of work為集中connection的方法，將各自獨立的repository共用SqlConnection，並使用SqlTransaction將各次的SQL指令作為一個單位，如果該單位沒有成功Save change，則所有的改動都可以還原到最開始的狀態 (rollback) </br>
3. 程式架構 View(前端) -> Controller(資料轉換、方法呼叫) -> Service Layer(邏輯處理、呼叫資料源) -> Data Access Layer(控制資料庫、API等連線)，各層間不可以反向引用(但可反向依賴interface)，達成封裝性的要求，以避免耦合過多，如要跨層使用方法可用static方法寫成Utility共用 (如Enum、Util方法等) </br>
4. 如物件導向有類似功能但又不盡相同的class，可將共有的變數、方法提升為base class，再利用繼承擴充base class，擴充方法如果都差不多則用Abstract, Interface去規定名稱、型別等，以避免類似的class中存在大量重複的code、類似方法規格不統一等

## Program & Startup

### Authentication

  - 詳見`Authentication.md`檔案

### Route

* 運作環境有route prefix時，可在Map Route前呼叫UsePathBase (務必要放在所有route設定前)，這樣有無prefix都可以正常呼叫
* 注意login path也需要加上prefix才能在登入時導到正確的路徑

```C#
// .Net 6 Program.cs
app.UsePathBase("/v2"); // 這邊設定prefix

// ...
app.UseEndpoints(endpoints =>
{
    endpoints.MapControllerRoute(
        name: "default",
        pattern: "{controller=Home}/{action=Index}/{id?}");
    // 這邊要注意將原本的index page轉為有prefix的index page，避免cookie無法辨別
    endpoints.MapGet("/", context =>
    {
        context.Response.Redirect($"{RouteResource.BackendRoutePrefix}/Home/Index");
        return Task.CompletedTask;
    });
    //...
});
// Cookie驗證的部分，要注意login path也要包含basepath (或是於自行施作CookieAuthenticationEvents時要把prefix也包含進去Url)
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.LoginPath = "/v2" + "/Home/Login";
    });
```

### Dependency Injection

#### 註冊服務

* AddSingleton
* AddScoped
* AddTransient

#### 取得服務

* Host (IHost, IWebHost...) instance

    ```C#
    app.ApplicationServices.GetService<IMyService1>();  // Return null if no service exisits
    app.ApplicationServices.GetRequiredService<IMyService2>();  // Throw excetion if no service exists
    ```

    ※ 此方法僅適用singleton service，建議scoped service使用`services.CreateScope`，再使用`scope.ServiceProvider.GetRequiredService`來取得</br></br>

* Registered service

    ```C#
    public MyService2 : IMyService2
    {
        private readonly IMyService1 _myService1;
        public MyService2(IMyService1 myService1)
        {
            // 建議使用此方法，而不是下方
            _myService1 = myService1;
        }

        public MyService2(IServiceProvider provider)
        {
            // 此方法雖然簡潔，但是在Build host時無法檢查instance是否為null
            // 而且會迫使所有Service在每個Request都New Instance (不管有沒有用到)，記憶體堪憂!
            _myService1 = provider.GetRequiredService<IService1>();
        }
    }
    ```

### Use app

### Middlewares

#### Middleware註冊

- Middleware是針對HTTP請求的中介層

```C#
// Program.cs (.Net 6)

builder.Services.AddTransient<MyMiddleware>();
//...
var app = builder.Build();
//...
app.UseMiddleware<MyMiddleware>();
```

```C#
// MyMiddleware.cs

public class MyMiddleware : IMiddleware
{
    public async Task InvokeAsync(HttpContext context, RequestDelegate next)
    {
         // Wait for next middleware return (The response body will be written)
        await next(context);

        // ...
    }
```

#### Filter註冊

- Action Filter屬於MVC獨有中介層，與Middleware不同
- 可利用Exception, Action & Result Filter捕捉資訊

```C#
// Startup.cs

services.AddControllersWithViews(options =>
{
    options.Filters.Add<ValidateModelFilter>();  // Call ModelState.IsValid for controller
})
```

```C#
// ValidateModelFilter.cs

public class ValidateModelFilter : IActionFilter
{
    public void OnActionExecuted(ActionExecutedContext context)
    {
        if (context.ModelState.IsValid)
        {
            return;
        }

        var validateErrors = context.ModelState.Select(x => x.Value.Errors).Where(y => y.Count > 0);
        string errorMsg = String.Join("; ", validateErrors.Select(x => x.FirstOrDefault().ErrorMessage ?? x.FirstOrDefault().Exception.Message));
        context.Result = new ObjectResult(errorMsg)
        {
            StatusCode = 400
        };
    }

    public void OnActionExecuting(ActionExecutingContext context)
    {
        // Do nothing
    }
}
```

##### Authorize Filter

* 設定步驟

  1. 需要在program.cs當中設定middleware `UseAuthorization`
     * 注意!!!!! 一定要放在UseAuthentication後面! 先驗證再授權，否則永遠進不到Authentication middleare!)

        ```C#
        // Program.cs
        app.UseAuthentication();
        app.UseAuthorization(); // <--一定要放在UseAuthentication後面! 先驗證user合法性、再處理授權
        ```

  2. Controller/Action當中加入Attribute `[Authorize]`

        ```C#
        [Authorize] // 擇一放置即可
        public class HomeController : Controller
        {
            [Authorize] // 擇一放置即可
            public IActionResult Index()
            {
                return View();
            }
        }
        ```

  3. `[AllowAnonymous]`會覆蓋過`[Authorize]`：比如controller設定`[Authorize]`，則特定action要跳過授權檢查的話加入`[AllowAnonymous]`即可

        ```C#
        [Authorize] // 全部action套用
        public class HomeController : Controller
        {
            [AllowAnonymous] // 特定action不需要"授權前"的驗證
            public IActionResult Index()
            {
                return View();
            }
        }
        ```


* 客製化實作IAuthorizationFilter

    ```C#
    [AttributeUsage(AttributeTargets.All)]
    public class MyAuthorizeAttribute : Attribute, IAuthorizationFilter
    {
        public void OnAuthorization(AuthorizationFilterContext context)
        {
            var userService = context.HttpContext.RequestServices.GetRequiredService<IUserService>();
            var userId = context.HttpContext.User.FindFirstValue(ClaimTypes.NameIdentifier);
            var controllerName = context.HttpContext.Request.RouteValues["controller"]?.ToString();

            if (!userService.HasPermission(userId, controllerName))
            {
                context.Result = new ForbidResult();
            }
        }
    }
    ```

  * Action指定需要授權的Role

  ```C#

  ```

## Environment

### Environment Variable

#### Set

* Unit test </br>
  ![1.png](images/.Net%20Core/1.png) </br>
* Debug </br>
  ![2.png](images/.Net%20Core/2.png) </br>
  ![3.png](images/.Net%20Core/3.png) </br>
  ![4.png](images/.Net%20Core/4.png) </br>
* IIS </br>
  ![6.png](images/.Net%20Core/6.png) </br>
* Azure Web App </br>
  ![7.png](images/.Net%20Core/7.png) </br>
* Process (PowerShell) </br>
  ![5.png](images/.Net%20Core/5.png) </br>

#### Get

## Frontend

### Tag Helper

- 可設定類似HTML Tag的自定義Element，產生HTML & JS Text
- 設定方式
  - 建立Class, Implement TagHelper & Attribute HtmlTargetElement
- 優點:
  - 強型別物件，方便追蹤
  - 有後端驗證
  - 前後端整合
- 缺點:
  - 部分設定不全，前端工具支援度受限(需要理解整個前端套件的設定)
  - 動態功能支援性受限(Razor只渲染一次)
  - 有時候設定上或可讀性較不直覺
  - 前後端職責混和
- 與前端架構使用: 視情況與習慣使用
- 範例

    ```C#
    // backend
    using Microsoft.AspNetCore.Razor.TagHelpers;

    [HtmlTargetElement("email")]
    public class EmailTagHelper : TagHelper
    {
        public string Address { get; set; }
        public string Display { get; set; }

        public override void Process(TagHelperContext context, TagHelperOutput output)
        {
            output.TagName = "a"; // 更改tag為<a>
            output.Attributes.SetAttribute("href", $"mailto:{Address}");
            output.Content.SetContent(Display);
        }
    }
    ```

    ```HTML
    <!--.cshtml-->
    @addTagHelper *, YourNamespace

    <!DOCTYPE html>
    <html>
    <head>
        <meta name="viewport" content="width=device-width" />
        <title>Email Tag Helper Example</title>
    </head>
    <body>
        <h1>Email Tag Helper Example</h1>

        <email address="john@example.com" display="Email John"></email>
    </body>
    </html>
    ```

  - 輸出結果範例

    ```HTML
    <a href="mailto:john@example.com">Email John</a>
    ```

## Deployment

### Linux

### 問題排解

* 站台運作後出現HTTP Error 500.30 - ANCM In-Process Start Failure
  * appsettings.json有格式錯誤，移除錯誤即可
  * Visual studio有bug，更新即可

## 傳輸協定

### gRPC

1. 建立 gRPC 服務和客戶端
    1. 步驟一：建立 gRPC 服務
    開啟 Visual Studio 2022，選擇 新建專案。
    搜尋 gRPC，選擇 ASP.NET Core gRPC 服務，然後點擊 下一步。
    在 配置新專案 對話框中，輸入專案名稱（例如 GrpcGreeter），選擇 .NET 8.0 (Long Term Support)，然 後點擊 建立。
    在 Additional information 對話框中，選擇 Create。
    2. 步驟二：編輯 proto 文件
    在專案中，找到 Protos/greet.proto 文件。
    編輯 greet.proto 文件以定義 gRPC 服務和消息。

    ```C#
    syntax = "proto3";

    option csharp_namespace = "GrpcGreeter";

    package greet;

    // The greeting service definition.
    service Greeter {
    // Sends a greeting
    rpc SayHello (HelloRequest) returns (HelloReply);
    }

    // The request message containing the user's name.
    message HelloRequest {
    string name = 1;
    }

    // The response message containing the greetings
    message HelloReply {
    string message = 1;
    }

    ```

   3. 步驟三：實現 gRPC 服務
   在 Services 文件夾中，創建 GreeterService.cs 文件。
   實現 GreeterService 類：
    (Build後會自動生成Model檔案-HelloRequest)

    ```C#
    using Grpc.Core;
    using GrpcGreeter;

    public class GreeterService : Greeter.GreeterBase
    {
        private readonly ILogger<GreeterService> _logger;
        public GreeterService(ILogger<GreeterService> logger)
        {
            _logger = logger;
        }

        public override Task<HelloReply> SayHello(HelloRequest request, ServerCallContext context)
        {
            return Task.FromResult(new HelloReply
            {
                Message = "Hello " + request.Name
            });
        }
    }
    ```

2. 建立 gRPC 客戶端
   1. 步驟一：建立新的 .NET Console 應用程式
   在 Visual Studio 中，建立新的 .NET Console 應用程式。
   添加 Grpc.Net.Client 和 Google.Protobuf NuGet 套件。
   2. 步驟二：編寫客戶端代碼
   在 Program.cs 文件中，編寫以下代碼：

   ```C#
   using Grpc.Net.Client;
   using GrpcGreeter;
   using System.Threading.Tasks;

   class Program
   {
       static async Task Main(string[] args)
       {
           // The port number must match the port of the gRPC server.
           using var channel = GrpcChannel.ForAddress("https://localhost:5001");
           var client = new Greeter.GreeterClient(channel);
           var reply = await client.SayHelloAsync(new HelloRequest { Name = "World" });
           Console.WriteLine("Greeting: " + reply.Message);
       }
   }
   ```

## Roslyn

### Source Generator

- 由.Net 6後引進的Build自動化流程
  - 用途範例: 根據專案內容自動生成檔案
  - 參考[官方文件](https://github.com/dotnet/roslyn/blob/main/docs/features/incremental-generators.md)
- 用法
  1. 新增Generator project
  2. 更改cshtml，設定output type為Analyzer
    - 目的: 告訴MSBuild該專案為Compiler analyzer，在編譯C#時自動加載來進行Generator

    ```XML
    <ProjectReference Include="..\MyAnalyzerProject\MyAnalyzer.csproj" OutputItemType="Analyzer" />
    ```

  3. 實作邏輯
     - 繼承IncreamentalGenerator，實作
       - IncrementalGeneratorInitializationContext.SyntaxProvider.CreateSyntaxProvider可生成一個providor，負責取得語法樹(SyntaxTree)的相關內容做抽取過濾
       - IncrementalGeneratorInitializationContext.RegisterImplementationSourceOutput可將provider進行操作(比如自動生成檔案)
       - 生成範例:

           ```C#
           using System.Collections.Generic;
           using System.Linq;
           using Microsoft.CodeAnalysis;
           using Microsoft.CodeAnalysis.CSharp.Syntax;
           using Microsoft.CodeAnalysis.Text;

           [Generator]
           public class MyClassNameGenerator : IIncrementalGenerator
           {
               public void Initialize(IncrementalGeneratorInitializationContext context)
               {
                   // 取得所有類別宣告
                   var classDeclarations = context.SyntaxProvider
                       .CreateSyntaxProvider(
                           (node, _) => node is ClassDeclarationSyntax,
                           (ctx, _) => (ClassDeclarationSyntax)ctx.Node
                       )
                       .Where(cls => cls.Identifier.Text == "TargetClass"); // 指定類名

                   // 註冊代碼生成
                   context.RegisterSourceOutput(classDeclarations, (ctx, cls) =>
                   {
                       var namespaceName = GetNamespace(cls);
                       var className = cls.Identifier.Text;

                       var sourceCode = $@"// <auto-generated/>
           namespace {namespaceName}
           {{
               public static class GeneratedInfo
               {{
                   public const string FullName = ""{namespaceName}.{className}"";
               }}
           }}";

                       ctx.AddSource($"{className}.g.cs", SourceText.From(sourceCode, System.Text.Encoding.UTF8));
                   });
               }

               private static string GetNamespace(ClassDeclarationSyntax cls)
               {
                   var namespaceDecl = cls.Ancestors().OfType<NamespaceDeclarationSyntax>().FirstOrDefault();
                   return namespaceDecl != null ? namespaceDecl.Name.ToString() : "UnknownNamespace";
               }
           }
           ```

- 何謂語法樹?
  - 如果你使用 Roslyn，以下是 JSON 表示法的簡單範例：

        ```JSON
        {
        "Root": {
            "Kind": "CompilationUnit",
            "Members": [
            {
                "Kind": "NamespaceDeclaration",
                "Name": "ExampleNamespace",
                "Members": [
                {
                    "Kind": "ClassDeclaration",
                    "Name": "MyClass",
                    "Members": [
                    {
                        "Kind": "MethodDeclaration",
                        "Name": "MyMethod",
                        "ReturnType": "void",
                        "Parameters": [
                        {
                            "Kind": "Parameter",
                            "Name": "x",
                            "Type": "int"
                        }
                        ]
                    }
                    ]
                }
                ]
            }
            ]
        }
        }
        ```

        這個 JSON 代表了一個 C# 程式：

        ```C#
        namespace ExampleNamespace
        {
            class MyClass
            {
                void MyMethod(int x) { }
            }
        }
        ```

- 如何debug source generator?
  - 指定generator project為startup project
  - 開啟"被掃描要建立檔案"的project properties → Debug → Launch Profiles → 新增Executable & Project以外的profile，並刪除其他profiles
  - 也可以直接改launchSettings，如下:

    ```JSON
    {
        "profiles": {
        "SourceGeneratorDebug": {
            "commandName": "Project",
            "environmentVariables": {
            "DOTNET_GENERATOR_DEBUG": "1"
            }
        }
        }
    }
    ```