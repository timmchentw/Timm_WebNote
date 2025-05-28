# Authentication & Authorization

* Authentication為驗證User登入、身分是否合；Authorization則不同，主要作為Role assign(授權)

- [Authentication \& Authorization](#authentication--authorization)
  - [Middleware Setup](#middleware-setup)
    - [Shared cookie (Single sign-on)](#shared-cookie-single-sign-on)
    - [Redirect to login](#redirect-to-login)
    - [Identity](#identity)
  - [Policy Based Authorization](#policy-based-authorization)
    - [RBAC](#rbac)
    - [ABAC](#abac)
    - [Add Policy](#add-policy)
    - [Authorize Use Case](#authorize-use-case)
    - [AuthorizationHandler](#authorizationhandler)


## Middleware Setup

* 在Startup的部分，設定AddAuthentication middleware

```C#
// .Net 6 Program.cs
var builder = WebApplication.CreateBuilder(args);
// ...
builder.Services.AddScoped<MyCookieAuthenticationEvents>(); // 註冊客製化Auth事件(登入轉址、登出等)
// ...
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.ExpireTimeSpan = TimeSpan.FromHours(12);
        options.SlidingExpiration = false;
        options.EventsType = typeof(MyCookieAuthenticationEvents);
        options.AccessDeniedPath = "/Home/AccessDeny";
        options.LoginPath = $"/Home/Login";
    });
// ...
var app = builder.Build();
app.UseAuthentication();
```

* 自定義CookieAuthenticationEvents middleware (記得在上方EventsType註冊)
* 可客製如取得當下的url、domain轉換等等

```C#
public class MyCookieAuthenticationEvents : CookieAuthenticationEvents
{
    public MyCookieAuthenticationEvents()
    {
    }

    public override async Task ValidatePrincipal(CookieValidatePrincipalContext context)
    {
        var userPrincipal = context.Principal;

        if (userPrincipal == null ||
            !string.IsNullOrEmpty(userPrincipal.Claims.Where(x => x.Type == ClaimTypes.NameIdentifier).FirstOrDefault()?.Value))
        {
            // 設定登出條件 (看登入時claim如何設定)
            context.RejectPrincipal();
            await context.HttpContext.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);
            return;
        }
    }

    public override Task RedirectToLogin(RedirectContext<CookieAuthenticationOptions> context)
    {
        // 可設定domain轉換
        string host = context.Request.Host.Value;
        if (context.Request.Host.Host.Equals("mywebsitesample.azurewebsites.net", StringComparison.OrdinalIgnoreCase))
        {
            host = "my.domain.com";
        }

        // 取得現在的url，並在轉址時附上為query string
        var currentUrl = $"{context.Request.Scheme}://{host}{context.Request.PathBase}{context.Request.Path}{context.Request.QueryString}";
        string loginUrl = $"https://{host}{context.Options.LoginPath}";

        var redirectUrlWithReturnUrl = QueryHelpers.AddQueryString(loginUrl, "returnUrl", currentUrl);
        context.Response.Redirect(redirectUrlWithReturnUrl);

        return Task.CompletedTask;
    }
}
```

### Shared cookie (Single sign-on)

* 注意要設定cookie name & path (不同path的cookie不互通)
* 也要設定data protection (用於在不同Websites間解密cookie內容)
* 指定SameSite的cookie存取嚴格程度(strict, lax, none...)
* 範例參考[官方文件](https://learn.microsoft.com/zh-cn/aspnet/core/security/cookie-sharing?view=aspnetcore-8.0)、[cookie嚴格程度說明](https://learn.microsoft.com/en-us/aspnet/core/security/samesite?view=aspnetcore-8.0)、[Data Protection設定](https://learn.microsoft.com/en-us/aspnet/core/security/data-protection/configuration/overview?view=aspnetcore-8.0)

```C#
// .Net 6 Program.cs
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        // 這塊跟上方一樣
        options.ExpireTimeSpan = TimeSpan.FromHours(24);
        options.SlidingExpiration = false;
        options.EventsType = typeof(MyCookieAuthenticationEvents);
        options.AccessDeniedPath = "/Home/AccessDeny";
        options.LoginPath = "/Home/Login";

        // 這塊設定共享cookie
        options.Cookie.SameSite = SameSiteMode.Strict;  // 可設定嚴格程度
        options.Cookie.Name = ".AspNet.SharedCookie";
        options.Cookie.Path = "/";  // 避免有不同basePath時，cookie可能會存在不同路徑 (如/v1, /v2會被視為不同存在)
        options.DataProtectionProvider = DataProtectionProvider.Create(
            new DirectoryInfo(
                (RuntimeInformation.IsOSPlatform(OSPlatform.Windows)) ?
                    @"C:\inetpub\wwwroot\MyWebsite\Keys\"
                    : "/var/www/MyWebsite/Keys/"));  // key加密檔案存在server端，host端必須要access到同一個檔案才行(即各個websites要放在同一個file system裡面)
    });
```

* 跨Server需要使用到KeyVault、或是DB，可參考[官方文件說明](https://learn.microsoft.com/en-us/aspnet/core/security/data-protection/configuration/overview?view=aspnetcore-8.0#protectkeyswithazurekeyvault)，以下為dbcontext的用法 (MyDbContext必須實作`IDataProtectionKeyContext`)

```C#
// .Net 6 Program.cs
using Microsoft.AspNetCore.DataProtection;
// ...
builder.Services.AddDataProtection()
    .SetApplicationName("AllWebsiteShouldUseSameName")
    .PersistKeysToDbContext<MyDbContext>();
// Cookie那邊不需特定設定，記得上面區塊的'options.DataProtectionProvider'就不用特別設定了
```

* 想瞭解.Net Core底層如何判定cookie是否合法，可參考以下部分進行debug (中斷點可下在上方自行覆寫的MyCookieAuthenticationEvents當中)
  * HandleAuthenticateAsync
  * ReadCookieTicket
  * HandleChallengeAsync
  * 注意：如果無論如何驗證都過了還是無法通過authorize，可檢查一下middleware設置的`UseAuthentication`必須放在`UseAuthorization`前面
  ![image](images/.Net%20Core/9.png) </br>
  * F12→Application查看Cookie資料
    ![image](images/.Net%20Core/10.png) </br>

### Redirect to login

* login get提供return url (由前端傳到post)

```C#
public class HomeController
{
    private AccountService _accountService;
    private HomeController _logger;

    public HomeController(IHttpContextAccessor httpContextAccessor, AccountService accountService, ILogger<HomeController> logger)
    {
        _accountService = accountService;
        _logger = logger;
    }

    // login page記得允許匿名存取
    [AllowAnonymous]
    public IActionResult Login(string? returnUrl)
    {
        // 由前一個頁面提供Return URL (參考前面MyCookieAuthenticationEvents.RedirectToLogin)
        ViewBag["ReturnUrl"] = returnUrl;
        return View();
    }

    // login page記得允許匿名存取
    [HttpPost]
    [AllowAnonymous]
    [ValidateAntiForgeryToken]
    public async Task<IActionResult> Login(LoginViewModel model)
    {
        if (ModelState.IsValid)
        {
            // 自行驗證user合法性(如DB, API資料驗證)
            var loginTicket = await _accountService.Login(new LoginDto()
            {
                Email = model.Email,
                Password = model.Password,
                ClientIp = HttpContext.Connection.RemoteIpAddress?.ToString() ?? ""
            });

            if (loginTicket.Success && loginTicket.Data != null)
            {
                var account = loginTicket.Data;

                // Identity日後可從HttpContext當中取得登入時的資訊
                var claims = new List<Claim>()
                {
                    new Claim(ClaimTypes.NameIdentifier, account.Id.ToString()),
                    new Claim(ClaimTypes.Name, string.IsNullOrEmpty(account.Name) ? String.Empty : account.Name),
                    new Claim(ClaimTypes.Role, JsonConvert.SerializeObject(user.Roles)),
                };

                var claimsIdentity = new ClaimsIdentity(claims, CookieAuthenticationDefaults.AuthenticationScheme);

                // 設定cookie sign in
                await HttpContext.SignInAsync(CookieAuthenticationDefaults.AuthenticationScheme, new ClaimsPrincipal(claimsIdentity),
                    new AuthenticationProperties()
                    {
                        ExpiresUtc = DateTime.UtcNow.AddHours(12),
                        IsPersistent = true,
                    });

                _logger.LogInformation("User logged in. Name: {Name}, Time: {Time} .", account.Name, DateTime.Now.ToString());

                // Return url前安全檢查
                if (IsValidReturnUrl(model.ReturnUrl))
                {
                    return Redirect(model.ReturnUrl);
                }
                else
                {
                    return RedirectToAction("Index");
                }
            }
            else
            {
                ViewBag["ResponseCode"] = loginTicket.Code;
                ViewBag["ResponseMessage"] = loginTicket.Message;
            }
        }
        return View();
    }

    [AllowAnonymous]
    public async Task<IActionResult> LogOut(string? returnUrl)
    {
        await HttpContext.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);

        // Return url前安全檢查
        if (IsValidReturnUrl(returnUrl))
        {
            return Redirect(returnUrl);
        }
        else
        {
            return RedirectToAction("Index");
        }
    }

    private bool IsValidReturnUrl(string returnUrl)
    {
        bool isValid = false;

        if (!string.IsNullOrEmpty(returnUrl))
        {
            var requestHost = Accessor.HttpContext?.Request?.Headers["X-Forwarded-Host"].FirstOrDefault()   // DNS設定的公開domain
                ?? Accessor.HttpContext?.Request?.Host.Host;

            // 比較Return url Domain與目前站台是否相同
            if (Uri.TryCreate(returnUrl, UriKind.Absolute, out Uri? uriResult))
            {
                isValid = string.Equals(uriResult.Host, requestHost, StringComparison.OrdinalIgnoreCase);
            }
            else
            {
                _logger.LogInformation("Parse returnUrl failed. Current domain: {CurrentDomain}; Return url: {ReturnUrl}", requestHost, returnUrl);
            }
        }
        return isValid;
    }
}
```

### Identity

* 官方[非Identity架構方法指南](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/cookie?view=aspnetcore-8.0)
* Claim & Identity說明見[ASP.NET Core Authentication系列（一）理解Claim, ClaimsIdentity, ClaimsPrincipal](https://www.cnblogs.com/liang24/p/13910368.html)
  * HttpContextAccessor可取得當時存的Identity

    ```C#
    public class MyService
    {
        protected int? UserId
        {
            get
            {
                return Convert.ToInt32(_httpContext.HttpContext?.User.FindFirstValue(ClaimTypes.NameIdentifier));
            }
        }

        protected IHttpContextAccessor _httpContext { get; }
        public BaseService(IHttpContextAccessor httpContextAccessor)
        {
            _httpContext = httpContextAccessor;
        }
    }
    ```


* 有設定才能在Controller上設置[Authorize]來驗證User

## Policy Based Authorization

- 更加靈活的授權機制，可自定義授權Policy，利用`[Authorize]`或`IAuthorizationService`來進行Policy檢查
- [MSDN Document](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies?view=aspnetcore-9.0)


### RBAC

- Role-Based Access Control
- 依據User Role進行權限授權
- 優點: 簡單、明確、好管理
- 簡單Policy範例:

```C#
//program.cs
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AdminOnly", policy =>
        policy.RequireRole("Admin"));
});
```

### ABAC

- Attribute-Based Access Control
- 依據User, Resource or Environment進行權限授權
  - User: 如職位、部門、等級
  - Resource: 如產品類型、價格
  - Environment: 如存取時間、位置
- 優點: 更好客製化、更複雜的權限控制
- 簡單Policy範例:

```C#
//program.cs
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AtLeast21", policy =>
        policy.RequireAssertion(context =>
            context.User.HasClaim(c => c.Type == "Age" && int.Parse(c.Value) >= 21)));
    // Requirement & Handler 應也算ABAC
});
```

### Add Policy

- 設定範例

```C#
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// 要先有Authentication才能用Authorization
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.Authority = "https://your-auth-server.com";
        options.Audience = "your-api";
    });

// 這邊設定各種Policy
builder.Services.AddAuthorization(options =>
{
    // 簡單Role
    options.AddPolicy("AdminOnly", policy =>
        policy.RequireRole("Admin"));
    // 簡單Claim
    options.AddPolicy("RequireEmployeeId", policy =>
        policy.RequireClaim("Permission", "CanViewPage", "CanViewAnything"));
    // 複合條件
    options.AddPolicy("ManagerWithID", policy =>
    {
        policy.RequireRole("Manager");
        policy.RequireClaim("Permission", "CanViewPage", "CanViewAnything");
    });
    // 更複雜操作
    options.AddPolicy("LimitedOrFull", policy =>
        policy.RequireAssertion(context =>
            context.User.HasClaim(c => c.Type == "Limited" || c.Type == "Full")));
});

var app = builder.Build();

// 記得要Use (先Authentication再Authorization)
app.UseAuthentication();
app.UseAuthorization();
```

### Authorize Use Case

- 用法範例
  - Attribute: 適合無參數授權(如Role檢查)

    ```C#
    // controller
    [Authorize(Policy = "AdminOnly")]
    [ApiController]
    [Route("api/[controller]")]
    public class AdminController : ControllerBase
    {
        [HttpGet]
        public IActionResult GetAdminData()
        {
            return Ok("這是管理員專屬的資料");
        }
    }
    ```

  - IAuthorizationService: 適合含參數檢查(如單產品權限檢查)
    - 一般需要透過Implement `AuthorizationHandler` 取出參數處理邏輯，可見下方 [AuthorizationHandler](#authorizationhandler)

    ```C#
    public class DocumentController(IAuthorizationService authorizationService) : Controller
    {
        public async Task<IActionResult> EditDocument(int documentId)
        {
            AuthorizationResult authorizationResult = await authorizationService.AuthorizeAsync(User, documentId, "EditDocumentPolicy");

            if (authorizationResult.Succeeded)
            {
                return Ok("授權成功");
            }
            else
            {
                return Forbid();
            }
        }
    }
    ```


### AuthorizationHandler

- AuthorizationHandler: 決定授權是否核可的邏輯，並依賴Requirement進行評估
  - Claim(Identity)如何設定請參考上方[Redirect To Login](#redirect-to-login)
- AuthorizationRequirement:
  1. 只負責定義"用於權限判斷的需求參數"(比如編輯產品必須要有的role)，不負責授權驗證
  2. 與Policy做綁定
     - 單一Requirement綁定多個Handler的話，則逐一觸發
     - 多個Handler只要都沒有觸發`context.Fail`，就算部分handler沒有觸發`context.Succeed`也不會驗證失敗

```C#
using Microsoft.AspNetCore.Authorization;
using System.Security.Claims;
using System.Threading.Tasks;

// 只定義授權需求的參數
public class EditDocumentRequirement : IAuthorizationRequirement
{
    public string AllowedRole { get; }

    public EditDocumentRequirement(string allowedRole)
    {
        AllowedRole = allowedRole;
    }
}

// 授權評估的邏輯
public class EditDocumentHandler(IDocumentRepository documentRepository) : AuthorizationHandler<EditDocumentRequirement>
{
    protected override async Task HandleRequirementAsync(
        AuthorizationHandlerContext context, EditDocumentRequirement requirement)
    {
        // 相關資訊可從Claim、Database等等地方取得比對
        var userIdClaim = context.User.FindFirst(c => c.Type == "UserId")?.Value;
        var userRoleClaim = context.User.FindFirst(c => c.Type == ClaimTypes.Role)?.Value;
        var documentIdClaim = context.Resource as int?; // 從呼叫端取得

        if (documentIdClaim != null && userIdClaim != null)
        {
            // 檢查資料庫，確認使用者是否為文件擁有者
            var document = await documentRepository.GetDocumentByIdAsync(documentIdClaim.Value);
            if (document != null && document.OwnerId == userIdClaim)
            {
                context.Succeed(requirement);
                return;
            }
        }

        // 如果使用者擁有允許的角色，則授權成功
        if (userRoleClaim == requirement.AllowedRole) // 這邊取用requirement參數比對
        {
            context.Succeed(requirement);
        }

        // 單一Handler的話，沒有context.Succeed則不會通過驗證
    }
}
```

- 註冊: AddPolicy進行註冊，其中Policy Name會作為使用端依據

```C#
// program.cs
services.AddAuthorization(options =>
{
    // Requirement綁定Policy
    options.AddPolicy("EditDocumentPolicy", policy =>
        policy.Requirements.Add(new EditDocumentRequirement("EditorRole")));
});

// 記得要註冊Handler
builder.Services.AddSingleton<IAuthorizationHandler, EditDocumentHandler>();
builder.Services.AddScoped<IDocumentRepository, DocumentRepository>();

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();
```

- 使用端: 使用IAuthorizationService/Authorize attribute進行

```C#
// controller
public class DocumentController(IAuthorizationService authorizationService) : Controller
{
    public async Task<IActionResult> EditDocument(int documentId)
    {
        AuthorizationResult authorizationResult = await authorizationService.AuthorizeAsync(User, documentId, "EditPolicy");

        if (authorizationResult.Succeeded)
        {
            return Ok("授權成功");
        }
        else
        {
            return Forbid();
        }
    }
}
```
