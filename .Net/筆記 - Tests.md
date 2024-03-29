# 自動化測試

## MSTest (V2)

* 當測試Case Input沒有可顯示的型別做辨別，則Test Explorer不會跳出Cases

### Lifecycle

* ClassInitialize (Static)
* TestInitialize (Non-Static)
* TestMethod (Non-Static)
* TestCleanup (Non-Static)
* ClassCleanup (Static)

### Test methods

* TestMethod
* DataTestMethod
* DataRow
* DynamicData

※ 以下範例為.Net 6環境，Host & DI語法稍有不同

```C#
[TestClass]
public class MyTests
{
    private static IHost _host;
    private static IServiceScope _serviceScope;
    private static IService _service;

    // 在所有Test之前 (必須為Static)
    [ClassInitialize]
    public static void Initialize(TestContext testContext)
    {
        // Set Environment
        Environment.SetEnvironmentVariable("ASPNETCORE_ENVIRONMENT", testContext.DeploymentDirectory.ToLower().Contains("debug") ? Environments.Development : Environments.Staging);

        // Dependency Injection
        var builder = WebApplication.CreateBuilder();
        builder.Services.AddSingleton(builder.Configuration.GetSection("MyConfig").Get<MyConfig>());    // Configuration in appsettings.json
        builder.Services.AddScoped<IService, MyService>();
        _host = builder.Build();
    }

    // 在每次Test之前
    [TestInitialize]
    public void TestInit()
    {
        _serviceScope = _host.Services.CreateScope();   // Create scope for each test
        _service = _serviceScope.ServiceProvider.GetRequiredService<IService>();    // Get service of testing
    }

    // 簡單Cases (僅限Const參數)
    [DataTestMethod]
    [DataRow(true)]
    [DataRow(false)]
    public void MyMethod1Test(bool isChecked)
    {
        var result = _testClass.MyMethod1();
        Assert.IsTrue(result);
    }

    // 複雜Cases (非Const參數)
    private static IEnumerable<object[]> MyMethod2TestCases
    {
        get
        {
            yield return new object[] { "2021", "output1" }; 
            yield return new object[] { "2022", "output2" }; 
        }
    }
    [DataTestMethod]
    [DynamicData(nameof(MyTest1Cases), DynamicDataSourceType.Property)]
    public void MyMethod2Test(string input, string expect)
    {
        var result = _testClass.MyMethod2(input);
        Assert.AreEqual(result, expect);
    }

    // 在每次Test之後
    [TestCleanup]
    public void Cleanup()
    {
        _serviceScope.Dispose();    // Clear service scope of current test
    }

    // 在所有Test結束之後 (必須為Static)
    [ClassCleanup]
    public static void Clean()
    {
        _host.Dispose();   // Clear app host
    }
}
```

### DI註冊 (參考[黑大文章](https://blog.darkthread.net/blog/aspnetcore-efcore-unitest/))

```C#
[TestClass]
public class MyTests
{
    private IWebHost _webHost;
    private T GetService<T>()
    {
        var scope = _webHost.Services.CreateScope();
        return scope.ServiceProvider.GetRequiredService<T>();
    }

    [ClassInitialize]
    public static void Init(TestContext testContext)
    {
        _webHost = WebHost.CreateDefaultBuilder()
            .UseStartup<Startup>()
            .Build();
    }

    [TestInitialize]
    public void Initialize()
    {
        
    }
}
```

### Mock

#### Methods

* Setup
  * CallBack
* It.IsAny
* Verify


```C#
services.AddScoped<IMyService>((services) => 
{
    var mockService = new Mock<IMyService>();
    moqService.Setup(x => x.GetBoolResult().Returns(true));  // 同步方法
    moqService.Setup(x => x.GetBoolResultAsync().Returns(Task.FromResult(true));  // 非同步方法
    moqService.Setup(x => x.GetStringResult(It.IsAny<string>()).Returns("..."));  // 有Input的Function
    return moqService.Object;
});
```

#### Partial Mock

可將Instance進行Mock，使部分Function回傳指定結果

* Mock Implement Class
* As 目標Interface
* CallBase必須為True
* 目標方法套用Virtual修飾

```C#
// Startup.cs
services.AddScoped<IBasicService, BasicService>();
services.AddScoped<IMyService>((services) => 
{
    var realService = new Mock<MyService>(services.GetRequiredService<IBasicService>());    // 遵照Constructor
    var moqService = realService.As<IMyService>();  // 轉成目標Interface
    moqService.Setup(x => x.CanGetData().Returns(true));    // Mock其中一個Function
    moqService.CallBase = true; // 開啟才能Call base class
    return moqService.Object;
});

// MyService.cs
public class MyService : IMyService
{
    private IBasicService _basicService
    public MyService(IBasicService basicService)
    {
        _basicService = basicService;
    }
    public virtual Task<bool> CanGetData()
    {
        // 必須使用virtual才能讓Mock Override
        return false;
    }
}

```

#### Mock Logger

* Mock目標Class的ILogger
* 使用Verify驗證Function被呼叫的次數、內容等

```C#
 [TestClass]
public class Tests
{
    [TestMethod]
    public async Task MockLoggerTest()
    {
        Mock<ILogger<MyService>> mockLogger = new Mock<ILogger<MyService>>();
        var service = new MyService(mockLogger.Object);
        
        service.Run();

        mockLogger.Verify(x => x.Log(
                LogLevel.Error,
                It.IsAny<EventId>(),
                //It.Is<It.IsAnyType>((o, t) => string.Equals("Index page say hello", o.ToString(), StringComparison.InvariantCultureIgnoreCase)),  // 可比較輸入內容
                It.Is<It.IsAnyType>((o, t) => true),    // 所有內容都接受
                It.IsAny<Exception>(),
                It.IsAny<Func<It.IsAnyType, Exception?, string>>()),
            Times.Never);   //可設定檢查被觸發幾次
    }
}

// MyService.cs
public class MyService
{
    private ILogger<MyService> _logger;

    public MyService(ILogger<MyService> logger)
    {
        _logger = logger;
    }

    public void Run()
    {
        _logger.LogError("Some error message");
    }
}
```

#### Mock DbContext (Entity Framework)

```C#

```