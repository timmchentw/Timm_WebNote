# Domain Driven Development (DDD)

領域驅動設計

- [Domain Driven Development (DDD)](#domain-driven-development-ddd)
  - [架構套件 - ABP Framework](#架構套件---abp-framework)
    - [註冊](#註冊)
    - [Settings](#settings)
    - [DDD架構實作](#ddd架構實作)
      - [Entity: 最底層的實體物件](#entity-最底層的實體物件)
      - [AggregateRoot: 集合Properties \& Entities的集合體](#aggregateroot-集合properties--entities的集合體)
      - [Audit Properties: 提供不少Interface/Class作為擴充properties使用](#audit-properties-提供不少interfaceclass作為擴充properties使用)
      - [Value Object: 描述Domain但不含邏輯的值物件](#value-object-描述domain但不含邏輯的值物件)
      - [Repositories: Domain與資料層的中介，負責資料處理、操作等](#repositories-domain與資料層的中介負責資料處理操作等)
      - [Specification: 基於Domain物件的特例](#specification-基於domain物件的特例)
      - [Domain Service: 商業邏輯存在的地方](#domain-service-商業邏輯存在的地方)
      - [Application Service](#application-service)
    - [Pattern與架構套裝](#pattern與架構套裝)
      - [Event](#event)
  - [重點心得](#重點心得)
    - [Aggregate Root / Entity](#aggregate-root--entity)
    - [Domain Service](#domain-service)
    - [Application Service](#application-service-1)


## <a name='-ABPFramework'></a>架構套件 - ABP Framework

- 文件: https://docs.abp.io/en/abp/latest
- 架構與安裝: https://docs.abp.io/en/abp/latest/Startup-Templates/Application

### <a name=''></a>註冊

- 在Program / Startup檔案中註冊Application

    ```C#
    // Startup.cs
    public class Startup
    {
        public void ConfigureServices(IServiceCollection services)
        {
            _ = services.AddApplication<MyAbpModule>();
        }

        public void Configure(IApplicationBuilder app)
        {
            app.InitializeApplication();
        }
    }
    ```

    ```C#
    [DependsOn(
        typeof(AbpAspNetCoreMvcModule),
        typeof(AbpAutofacModule),
        typeof(AbpAutoMapperModule),
        typeof(AbpAuditLoggingEntityFrameworkCoreModule),
        typeof(AbpSettingManagementEntityFrameworkCoreModule),
        typeof(MyCustomModule)
        )]
    public class MyAbpModule : AbpModule
    {
        public override void OnApplicationInitialization(ApplicationInitializationContext context)
        {
            base.OnApplicationInitialization(context);

            IApplicationBuilder app = context.GetApplicationBuilder();
            IWebHostEnvironment env = context.GetEnvironment();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllerRoute(name: AreaName.Default,
                                            pattern: "{controller=Home}/{action=Index}");

                void MapAreaControllerRoute(string area)
                {
                    endpoints.MapAreaControllerRoute(name: area,
                                                    areaName: area,
                                                    pattern: area + "/{controller=Home}/{action=Index}");
                }
            });
        }

        public override void ConfigureServices(ServiceConfigurationContext context)
        {
            SetupAbpAntiForgeryOptions();
            SetupAbpDbContextOptions();
            SetupBasicConfigurations(context.Services);
            SetupAppConfiguration(context.Services);
        }

        private void SetupAbpAuditLogging(IServiceCollection services)
        {
            AbpAuditLoggingDbProperties.DbSchema = "log";
            AbpAuditLoggingDbProperties.DbTablePrefix = string.Empty;

            Configure<AbpAuditingOptions>(options =>
            {
                options.ApplicationName = "MyWeb";
                options.AlwaysLogOnException = false;
                options.Contributors.Add(new MyWebLogContributor());
            });

            AbpAspNetCoreAuditingOptions option = new();
            services.GetConfiguration()
                .GetSection("AuditLogging")
                .Bind(option);

            Configure<AbpAspNetCoreAuditingOptions>(o =>
            {
                foreach (string url in option.IgnoredUrls)
                {
                    _ = o.IgnoredUrls.AddIfNotContains(url);
                }
            });
        }

        private void SetupAbpAntiForgeryOptions()
        {
            Configure<AbpAntiForgeryOptions>(options => options.AutoValidate = false);
        }

        private void SetupAbpDbContextOptions()
        {
            Configure<AbpDbContextOptions>(options => options.UseSqlServer());
        }

        private static void SetupBasicConfigurations(IServiceCollection services)
        {
            services.AddControllersWithViews()
                    .AddNewtonsoftJson();

            services.AddRouting()
                    .AddMvc()
                    .AddControllersAsServices();

            services.AddHttpContextAccessor();
        }

        private static void SetupAppConfiguration(IServiceCollection services)
        {
            services.AddSingleton<IAppConfiguration, AppConfiguration>();
        }
    }

    // 可由上方主要AbpModule Reference子Module
    public class MyCustomModule : AbpModule
    {
        public override void ConfigureServices(ServiceConfigurationContext context)
        {
            context.Services.ConfigureOptions(typeof(MyCustomConfigureOptions));

            SetupCustomServices(context.Services);
        }

        private static void SetupCustomServices(IServiceCollection services)
        {
            services.AddSingleton<IMyService2, MyService2>();
        }
    }

    // Option
    internal class MyCustomConfigureOptions : IPostConfigureOptions<StaticFileOptions>
    {
        private readonly IWebHostEnvironment _environment;

        public ReportConfigureOptions(IWebHostEnvironment environment)
        {
            _environment = environment;
        }

        public void PostConfigure(string name, StaticFileOptions options)
        {
            //settings for options here...
        }
    }
    ```

### <a name='Settings'></a>Settings

- 用於自定義key去做get/set所使用的實作功能

    ```C#
    // reference class
    private readonly ISettingManager _settingManager;

    public MyService(ISettingManager settingManager)
    {
        _settingManager = settingManager;
    }

    // usage
    string layoutType = await _settingManager.GetOrNullForCurrentUserAsync("App.UI.LayoutType");
    await _settingManager.SetForCurrentUserAsync("App.UI.LayoutType", "LeftMenu");

    ```

### <a name='DDD'></a>DDD架構實作

#### Entity: 最底層的實體物件

- 有單一Id Property則繼承這個abstract class，並且給予Id型別 (Guid, int...)
- `EntityEquals` 提供比較方法
- 提供Cache方法，DI時使用`context.Services.AddEntityCache<MyClass, Guid>();`，也可在同時使用設定、model mapping等等

    ```C#
    public class Book : Entity<Guid>
    {
        public string Name { get; set; }

        public float Price { get; set; }
    }
    ```

  - Composite key則自行override function "GetKeys"提供

    ```C#
    public class OrderLine : Entity
    {
        public virtual Guid OrderId { get; protected set; }

        public virtual Guid ProductId { get; protected set; }

        public virtual int Count { get; protected set; }

        protected OrderLine()
        {

        }

        internal OrderLine(Guid orderId, Guid productId, int count)
        {
            OrderId = orderId;
            ProductId = productId;
            Count = count;
        }

        internal void ChangeCount(int newCount)
        {
            Count = newCount;
        }
        
        public override object[] GetKeys()
        {
            return new Object[] {OrderId, ProductId};
        }
    }
    ```

#### AggregateRoot: 集合Properties & Entities的集合體
  
- 可包含操作邏輯 (如CRUD)，注意只針對其自身的型別與實體的操作
  - 比如新增/刪除Owned entity instance

```C#
public class Order : AggregateRoot<Guid>
{
    public virtual string ReferenceNo { get; protected set; }

    public virtual int TotalItemCount { get; protected set; }

    public virtual DateTime CreationTime { get; protected set; }

    public virtual List<OrderLine> OrderLines { get; protected set; }

    protected Order()
    {

    }

    public Order(Guid id, string referenceNo)
    {
        Check.NotNull(referenceNo, nameof(referenceNo));
        
        Id = id;
        ReferenceNo = referenceNo;
        
        OrderLines = new List<OrderLine>();
    }

    public void AddProduct(Guid productId, int count)
    {
        if (count <= 0)
        {
            throw new ArgumentException(
                "You can not add zero or negative count of products!",
                nameof(count)
            );
        }

        var existingLine = OrderLines.FirstOrDefault(ol => ol.ProductId == productId);

        if (existingLine == null)
        {
            OrderLines.Add(new OrderLine(this.Id, productId, count));
        }
        else
        {
            existingLine.ChangeCount(existingLine.Count + count);
        }

        TotalItemCount += count;
    }
}
```

#### Audit Properties: 提供不少Interface/Class作為擴充properties使用

- 比如: IHasCreationTime, IMayHaveCreator, ICreationAuditedObject

#### Value Object: 描述Domain但不含邏輯的值物件

- `ValueEquals` 提供比較方法
- 無特定ID結構，較多用於比較值

    ```C#
    public class Address : ValueObject
    {
        public Guid CityId { get; private set; }

        public string Street { get; private set; }

        public int Number { get; private set; }

        private Address()
        {
            
        }
        
        public Address(
            Guid cityId,
            string street,
            int number)
        {
            CityId = cityId;
            Street = street;
            Number = number;
        }

        protected override IEnumerable<object> GetAtomicValues()
        {
            yield return Street;
            yield return CityId;
            yield return Number;
        }
    }
    ```

  #### Repositories: Domain與資料層的中介，負責資料處理、操作等

  - 以aggregate root / entity為單位
  - 預設方法: `GetAsync`, `FindAsync`, `InsertAsync`, `UpdateAsync`, `DeleteAsync`
    - Bulk: `InsertManyAsync`, `UpdateManyAsync`, `DeleteManyAsync`
    - 唯讀: `GetListAsync`, `GetPagedListAsync`, `GetCountAsync` (介面`IReadOnlyRepository`)
    - Join: `WithDetails`, `WithDetailsAsync`
  - 提供Soft / Hard Delete
  - 提供Ensure Entities Exist檢查
    - EnsurePropertyLoadedAsync
      - 讓尚未填入的property (如virtual) 從DB填補
      - 很適合insert entity後，填補inserted後的關聯欄位

        ```C#
        _ = await Repository.InsertAsync(entity);

        await Repository.EnsurePropertyLoadedAsync(entity, x => x.RelatedObject);
        entity.RelatedObject  // --> 不會是Null了;
        ```

  - 提供Enabling / Disabling the Change Tracking (大量Query時再考慮關閉):
    - `using (_myRpository.DisableTracking())` 或 `[DisableEntityChangeTracking]`

    ```C#
    using System;
    using System.Threading.Tasks;
    using Volo.Abp.Application.Services;
    using Volo.Abp.Domain.Repositories;

    namespace Demo
    {
        // 以下為Repo使用範例
        public class PersonAppService : ApplicationService
        {
            private readonly IRepository<Person, Guid> _personRepository;

            public PersonAppService(IRepository<Person, Guid> personRepository)
            {
                _personRepository = personRepository;
            }

            public async Task CreateAsync(CreatePersonDto input)
            {
                var person = new Person(input.Name);

                await _personRepository.InsertAsync(person);
            }

            public async Task<int> GetCountAsync(string filter)
            {
                return await _personRepository.CountAsync(p => p.Name.Contains(filter));
            }
        }
    }
    ```

  - Query / Linq語法支援: `GetQueryableAsync`取得IQueryable物件

    ```C#
    using System;
    using System.Linq;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    using Volo.Abp.Application.Services;
    using Volo.Abp.Domain.Repositories;

    namespace Demo
    {
        public class PersonAppService : ApplicationService
        {
            private readonly IRepository<Person, Guid> _personRepository;

            public PersonAppService(IRepository<Person, Guid> personRepository)
            {
                _personRepository = personRepository;
            }

            // LINQ query syntax
            public async Task<List<PersonDto>> GetListAsync1(string filter)
            {
                //Obtain the IQueryable<Person>
                IQueryable<Person> queryable = await _personRepository.GetQueryableAsync();

                //Create a query
                var query = from person in queryable
                    where person.Name == filter
                    orderby person.Name
                    select person;

                //Execute the query to get list of people
                var people = query.ToList();

                //Convert to DTO and return to the client
                return people.Select(p => new PersonDto {Name = p.Name}).ToList();
            }

            // LINQ method syntax
            public async Task<List<PersonDto>> GetListAsync2(string filter)
            {
                //Obtain the IQueryable<Person>
                IQueryable<Person> queryable = await _personRepository.GetQueryableAsync();

                //Execute a query
                var people = queryable
                    .Where(p => p.Name.Contains(filter))
                    .OrderBy(p => p.Name)
                    .ToList();

                //Convert to DTO and return to the client
                return people.Select(p => new PersonDto {Name = p.Name}).ToList();
            }
        }
    }
    ```

  - Custom Repository: 自行繼承Interface / Class (如EfCoreRepository)

    ```C#
    public interface IPersonRepository : IRepository<Person, Guid>
    {
        Task<Person> FindByNameAsync(string name);
    }

    public class PersonRepository : EfCoreRepository<MyDbContext, Person, Guid>, IPersonRepository
    {
        public PersonRepository(IDbContextProvider<TestAppDbContext> dbContextProvider) 
            : base(dbContextProvider)
        {

        }

        public async Task<Person> FindByNameAsync(string name)
        {
            var dbContext = await GetDbContextAsync();
            return await dbContext.Set<Person>()
                .Where(p => p.Name == name)
                .FirstOrDefaultAsync();
        }
    }
    ```

  #### Specification: 基於Domain物件的特例
  
  - 優點: 有命名、可重用、可合併、可單獨測試
  - 缺點: C# Expression可取代邏輯(後來才出的)、不適用無商業邏輯或報表相關的情境
  - 符合條件邏輯寫在`ToExpression`

    ```C#
    using System;
    using System.Linq.Expressions;
    using Volo.Abp.Specifications;

    namespace MyProject
    {
        public class Age18PlusCustomerSpecification : Specification<Customer>
        {
            public override Expression<Func<Customer, bool>> ToExpression()
            {
                return c => c.Age >= 18;
            }
        }
    }
    ```

    - 用法: 做為批次篩選使用

        ```C#
        using System;
        using System.Collections.Generic;
        using System.Linq;
        using System.Threading.Tasks;
        using Volo.Abp.DependencyInjection;
        using Volo.Abp.Domain.Repositories;
        using Volo.Abp.Domain.Services;

        namespace MyProject
        {
            public class CustomerManager : DomainService, ITransientDependency
            {
                private readonly IRepository<Customer, Guid> _customerRepository;

                public CustomerManager(IRepository<Customer, Guid> customerRepository)
                {
                    _customerRepository = customerRepository;
                }

                public async Task<List<Customer>> GetCustomersCanBuyAlcohol()
                {
                    var queryable = await _customerRepository.GetQueryableAsync();
                    // 這邊直接將Expression放到Where裡面做篩選
                    // 提示: 下面ToExpression語法可省略不用寫
                    var query = queryable.Where(
                        new Age18PlusCustomerSpecification().ToExpression()
                    );
                    
                    return await AsyncExecuter.ToListAsync(query);
                }
            }
        }
        ```

    - 可多重Specification篩選: `And`, `Or`, `Not`, `AndNot`

        ```C#
        using System;
        using System.Threading.Tasks;
        using Volo.Abp.DependencyInjection;
        using Volo.Abp.Domain.Repositories;
        using Volo.Abp.Domain.Services;
        using Volo.Abp.Specifications;

        namespace MyProject
        {
            public class CustomerManager : DomainService, ITransientDependency
            {
                private readonly IRepository<Customer, Guid> _customerRepository;

                public CustomerManager(IRepository<Customer, Guid> customerRepository)
                {
                    _customerRepository = customerRepository;
                }

                public async Task<int> GetAdultPremiumCustomerCountAsync()
                {
                    // 這邊做多重語法
                    return await _customerRepository.CountAsync(
                        new Age18PlusCustomerSpecification()
                        .And(new PremiumCustomerSpecification()).ToExpression()
                    );
                }
            }
        }
        ```

    - 新增Specification合併多個: 需要重用的情形使用

        ```C#
        using Volo.Abp.Specifications;

        namespace MyProject
        {
            public class AdultPremiumCustomerSpecification : AndSpecification<Customer>
            {
                // 用Constructor Overload來合併
                public AdultPremiumCustomerSpecification() 
                    : base(new Age18PlusCustomerSpecification(),
                        new PremiumCustomerSpecification())
                {
                }
            }
        }
        ```

  - 提供`IsSatisfiedBy`供單一實體驗證符合條件
  
    ```C#
    using System;
    using System.Threading.Tasks;
    using Volo.Abp.DependencyInjection;

    namespace MyProject
    {
        public class CustomerService : ITransientDependency
        {
            public async Task BuyAlcohol(Customer customer)
            {
                if (!new Age18PlusCustomerSpecification().IsSatisfiedBy(customer))
                {
                    throw new Exception(
                        "This customer doesn't satisfy the Age specification!"
                    );
                }
                
                //TODO...
            }
        }
    }
    ```

  #### Domain Service: 商業邏輯存在的地方

  - 用途: 處理資料的邏輯(比如不可重複、特定規則等等) → 與AggregateRoot不同的地方是，其只針對本身Property進行操作，而如到DB查詢重複、數量限制等等，還是需要Domain service進行
  - 預設DI週期為Transient
  - 可使用預設Injected Properties，如`ILogger`, `IGuidGenerator`
  - 命名suffix推薦: Manager / Service

    ```C#
    public class IssueManager : DomainService
    {
        private readonly IRepository<Issue, Guid> _issueRepository;

        public IssueManager(IRepository<Issue, Guid> issueRepository)
        {
            _issueRepository = issueRepository;
        }
        
        public async Task AssignAsync(Issue issue, AppUser user)
        {
            var currentIssueCount = await _issueRepository
                .CountAsync(i => i.AssignedUserId == user.Id);
            
            //Implementing a core business validation
            if (currentIssueCount >= 3)
            {
                throw new IssueAssignmentException(user.UserName);
            }

            issue.AssignedUserId = user.Id;
        }    
    }
    ```

  - Referenced by "Application Services"
  
    ```C#
    using System;
    using System.Threading.Tasks;
    using MyProject.Users;
    using Volo.Abp.Application.Services;
    using Volo.Abp.Domain.Repositories;

    namespace MyProject.Issues
    {
        public class IssueAppService : ApplicationService, IIssueAppService
        {
            private readonly IssueManager _issueManager;
            private readonly IRepository<AppUser, Guid> _userRepository;
            private readonly IRepository<Issue, Guid> _issueRepository;

            public IssueAppService(
                IssueManager issueManager,
                IRepository<AppUser, Guid> userRepository,
                IRepository<Issue, Guid> issueRepository)
            {
                _issueManager = issueManager;
                _userRepository = userRepository;
                _issueRepository = issueRepository;
            }

            public async Task AssignAsync(Guid id, Guid userId)
            {
                var issue = await _issueRepository.GetAsync(id);
                var user = await _userRepository.GetAsync(userId);

                await _issueManager.AssignAsync(issue, user);
                await _issueRepository.UpdateAsync(issue);
            }
        }
    }
    ```

  #### Application Service

  - 與Domain Service比較:

    ||Application|Domain|
    |--|--|--|
    |內容|跟使用者互動有關的行為(如Web)|核心邏輯、獨立領域知識|
    |返回型別|Data Transfer Object (DTO)|Domain Objects (Entities, Value Object)|
    |相依來源|Presentation Layer / Client Applications|Application Service / 其他Domain Services|

    ```C#
    public class BookAppService : ApplicationService, IBookAppService
    {
        private readonly IRepository<Book, Guid> _bookRepository;

        public BookAppService(IRepository<Book, Guid> bookRepository)
        {
            _bookRepository = bookRepository;
        }

        public async Task CreateAsync(CreateBookDto input)
        {
            var book = new Book(
                GuidGenerator.Create(),
                input.Name,
                input.Type,
                input.Price
            );

            await _bookRepository.InsertAsync(book);
        }
    }
    ```

### Pattern與架構套裝

#### Event

- 提供Event供註冊與取用

```C#
// 實作Event內容
public class EmailService(SmtpClient client)
    : ILocalEventHandler<MyEmailEvent>
{
    // ILocalEventHandler實作
    public Task HandleEventAsync(MyEmailEvent eventData)
    {
        return RunTheEvent(eventData.EmailModel);
    }

    public async Task RunTheEvent(EmailMessage emailMessage)
    {
        // ...
        await client.SendAsync(emailMessage);
    }
}

// Event輸入參數
public class MyEmailEvent
{
    public EmailMessage EmailModel { get; set; }
}

// 呼叫Event範例
public class MyApp(ILocalEventBus eventBus)
{
    public async Task DoSomething(EmailMessage message)
    {
        // ...
        await eventBus.PublishAsync(new RecipeLockedEvent()
        {
            EmailModel = message
        });
    }
}
```

## 重點心得

### Aggregate Root / Entity

1. Entity / Aggregate Root著重在record "ID"的操作主導性，作為CRUD的根本依據 (而不是傳統的各種Where條件)
2. Domain邏輯越靠近Entity / Aggregate Root越好，不過不能包含資料庫Repository、Collection等邏輯在裡面
3. Aggregate Root底下的各種Entity / Owned Entity，需以主ID做關聯、並且留意Create / Update Date等相關參數需要與否
4. Aggregate Root留意Concurrency Stamp做資料覆寫保護
5. Aggregate Root注意Entity的關注點分離(比如CRUD內的互相影響)，保留其擴充性與彈性
   - 舉例: 一次只更新一個Related Entity而不是一整組
   - 舉例: 部分Property / Related Entity改動要不要涉及Concurrency Stamp的檢查保護

### Domain Service

1. 需外部相依的Service、複雜的邏輯等，需從Entity那邊獨立過來
2. 小缺點: Entity的Auto mapper會無法自動mapping
3. 注意: 跟Collection / Repository有關的東西，要放在App Service而不是這裡

### Application Service

1. 注意Domain邏輯的操作性 (比如A Property存在就不能Delete entity，那就要注意先操作A Property再做Delete)
2. Application Service間可以互相呼叫，比如可作為部分App Service的擴充 (但注意一定要用DTO做溝通)
3. DTO的共用性須注意，權衡要選擇"減少重複維護"/"分離讀寫或類似功能"的時間成本考量