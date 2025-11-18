## Command and Query Responsibility Segregation (CQRS)

* 將存取功能分離成Command與Query，進行讀寫分離
* 優點: 職責分離、測試與多工開發方便
* 範例: 將有讀寫功能的Repository，拆分成CommandHandler & QueryHandler

## Mediator

* 服務中介層，可讓高層級端不須注入特定型別的Service，僅需呼叫這個中介層，達到完全解耦 
  * 使用情境: 如兩個不能互相相依的projects
  * 範例: Product get對cache handler的鬆散依賴
* 為中介者模式實作，一般會搭配CQRS模式
  * SQRS: Command and Query Responsibility Segregation = 寫入與讀取職責分離
    * 範例: GetProductQueryHandler 專門處理查詢、AddProductCommandHandler 專門處理新增、UpdateProductCommandHandler 專門處理更新
* 與Event差異: Mediator重視來回一對一同步溝通，Event則否
* 套件: MediatR
* 程式碼範例

```C#
using MediatR;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Caching.Memory;

// ===== 1. 定義Query物件 (查詢請求) =====
public record GetProductQuery(int ProductId) : IRequest<ProductDto>;  // 實作IRequest<回傳型別>

public record ProductDto(int Id, string Name, decimal Price);

// ===== 2. 定義Query Handler邏輯 (查詢處理器) =====
public class GetProductQueryHandler : IRequestHandler<GetProductQuery, ProductDto>
{
    private readonly IMemoryCache _cache;
    private readonly IProductRepository _repository;

    public GetProductQueryHandler(IMemoryCache cache, IProductRepository repository)
    {
        _cache = cache;
        _repository = repository;
    }

    // 處理邏輯: 先查Cache，沒有再查DB
    public async Task<ProductDto> Handle(GetProductQuery request, CancellationToken cancellationToken)
    {
        var cacheKey = $"Product_{request.ProductId}";
        
        // 檢查Cache
        if (_cache.TryGetValue(cacheKey, out ProductDto cachedProduct))
            return cachedProduct;

        // 查詢DB
        var product = await _repository.GetByIdAsync(request.ProductId, cancellationToken);
        
        if (product != null)
        {
            // 存入Cache
            _cache.Set(cacheKey, product, TimeSpan.FromMinutes(10));
        }

        return product;
    }
}

// ===== 3. 使用範例 (Controller) =====
public class ProductController : ControllerBase
{
    private readonly IMediator _mediator;  // 注入Mediator (不需知道Handler細節)

    public ProductController(IMediator mediator)
    {
        _mediator = mediator;
    }

    [HttpGet("{id}")]
    public async Task<ActionResult<ProductDto>> GetProduct(int id)
    {
        // 透過Mediator發送Query，自動找到對應Handler執行
        var product = await _mediator.Send(new GetProductQuery(id));
        
        return product != null ? Ok(product) : NotFound();
    }
}

// ===== 4. DI 註冊 =====
builder.Services.AddMediatR(cfg => cfg.RegisterServicesFromAssembly(Assembly.GetExecutingAssembly()));
```

### 相關連結

* [MediatR介紹與CQRS簡介](https://hackmd.io/@spyua/rJZuyK3L_)
* [MSDN](https://learn.microsoft.com/en-us/azure/architecture/patterns/cqrs)

## Event Driven Architecture (TDA)

* 優點: 低耦合關係，使得不同Component能夠跨域溝通
* 與Mediator的差異:
  * Event屬射後不理 (fire-and-forget)
    * Mediator則是不在意跟哪個詳細型別做互動，但通常會有回傳值
  * 非同步、不在意結果
  * 可一對多
* Event
  * 為一個簡單物件，負責傳遞Event資訊，與Handler可為不同概念 (通常較廣)
    * 比如User Added, Product deleted...等
* Event Handler
  * 與Event綁定，負責處理事件內容邏輯 (通常較細節)
    * 比如操作Cache、寄送通知...等
    * 單一Event可給多個Event Handler非同步觸發
* Event Bus
  * 為Event處理的中介層，負責做觸發與接收
  * Subscribe
    * 用於訂閱事件 (綁定Event & Handler)
  * Publish
    * 用於觸發事件 (執行Event handler)
  * 使用`ConcurrentDictionary`跨執行緒紀錄事件
  * 程式碼範例

```C#
// ===== 1. 定義Event (事件類別) =====
public record ProductDeletedEvent(int ProductId);  // 產品刪除事件

// ===== 2. 定義Event Handler (事件處理器) =====
public interface IEventHandler<in TEvent>
{
    Task HandleAsync(TEvent evt);
}

// 實作Handler: 清除Cache
public class ClearCacheHandler : IEventHandler<ProductDeletedEvent>
{
    private readonly IMemoryCache _cache;

    public ClearCacheHandler(IMemoryCache cache) => _cache = cache;

    public Task HandleAsync(ProductDeletedEvent evt)
    {
        _cache.Remove(evt.ProductId);
        return Task.CompletedTask;
    }
}

// 實作Handler: 發送通知
public class SendNotificationHandler : IEventHandler<ProductDeletedEvent>
{
    public Task HandleAsync(ProductDeletedEvent evt)
    {
        // 發送Email或其他通知...
        Console.WriteLine($"Product {evt.ProductId} deleted notification sent");
        return Task.CompletedTask;
    }
}

// ===== 3. Event Bus (事件中介) =====
public class EventBus
{
    // Key: Event類型, Value: Handler清單
    private readonly ConcurrentDictionary<Type, ConcurrentDictionary<string, object>> _handlers = new();

    // 訂閱: 註冊Handler到特定Event
    public string Subscribe<TEvent>(IEventHandler<TEvent> handler)
    {
        var eventType = typeof(TEvent);
        var handlers = _handlers.GetOrAdd(eventType, _ => new());
        
        var token = Guid.NewGuid().ToString();
        handlers[token] = handler;
        return token;
    }

    // 發布: 觸發所有訂閱該Event的Handlers
    public async Task PublishAsync<TEvent>(TEvent evt)
    {
        var eventType = typeof(TEvent);
        
        if (!_handlers.TryGetValue(eventType, out var handlers))
            return;

        var tasks = handlers.Values
            .Cast<IEventHandler<TEvent>>()
            .Select(h => h.HandleAsync(evt));

        await Task.WhenAll(tasks);
    }

    // 取消訂閱
    public void Unsubscribe<TEvent>(string token)
    {
        if (_handlers.TryGetValue(typeof(TEvent), out var handlers))
            handlers.TryRemove(token, out _);
    }
}

// ===== 4. 使用範例 =====
public class ProductService
{
    private readonly EventBus _eventBus;

    public ProductService(EventBus eventBus)
    {
        _eventBus = eventBus;
        
        // 訂閱事件
        _eventBus.Subscribe(new ClearCacheHandler(/* cache */));
        _eventBus.Subscribe(new SendNotificationHandler());
    }

    public async Task DeleteProductAsync(int productId)
    {
        // 刪除產品邏輯...
        
        // 發布事件 (自動觸發所有Handler)
        await _eventBus.PublishAsync(new ProductDeletedEvent(productId));
    }
}
```

  ### 相關連結
  * [Event Driven Architecture (TDA)](https://www.linkedin.com/pulse/event-driven-architectureeda-pattern-mukesh-kumar--snudf)


## Queue

### System.Threading.Channels

* 類似於RabbitMQ，是Thread當中的Queue，如下範例提供寫入與取出的用法

```C#
using System;
using System.Threading.Channels;
using System.Threading.Tasks;

class Program
{
    static async Task Main(string[] args)
    {
        // 創建一個無界通道
        var channel = Channel.CreateUnbounded<int>();

        // 啟動生產者任務
        var producer = Task.Run(async () =>
        {
            for (int i = 0; i < 10; i++)
            {
                await channel.Writer.WriteAsync(i);
                Console.WriteLine($"Produced: {i}");
                await Task.Delay(100); // 模擬生產延遲
            }
            channel.Writer.Complete();
        });

        // 啟動消費者任務
        var consumer = Task.Run(async () =>
        {
            await foreach (var item in channel.Reader.ReadAllAsync())
            {
                Console.WriteLine($"Consumed: {item}");
                await Task.Delay(150); // 模擬消費延遲
            }
        });

        // 等待生產者和消費者完成
        await Task.WhenAll(producer, consumer);
    }
}

```
