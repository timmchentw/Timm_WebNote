# Design Pattern

- [Design Pattern](#design-pattern)
  - [Factory](#factory)
    - [相依性注入](#相依性注入)
    - [簡單工廠](#簡單工廠)
    - [抽象工廠](#抽象工廠)
    - [靜態工廠方法（Static Factory Method）](#靜態工廠方法static-factory-method)
  - [策略模式](#策略模式)
  - [Template 範本模式](#template-範本模式)
  - [Singleton 單例模式](#singleton-單例模式)
  - [Adapter 適配器模式](#adapter-適配器模式)
  - [Prototype 原型模式](#prototype-原型模式)
  - [Chain of Responsibility 責任鏈模式](#chain-of-responsibility-責任鏈模式)


## Factory

### 相依性注入

- 將相依方法、參數以Constructor注入其中，Class內不負責相依項目的實例化
- 搭配Interface會降低更多的耦合度
- 優點: 低耦合，符合LSP，且減化掉實例化的邏輯
- 缺點: 程式較不直觀，且需要注意注入的Class的實例狀況 (如Singleton instance需要注意各個引用的class的共用狀況)

    ```C#
    public class DIClass
    {
        private readonly AppSettings _appSettings;
        private readonly IService1 _serivce1;
        private readonly IService2 _service2;

        public DIClass(AppSettings appSettings, IService1 serivce1, IService2 service2)
        {
            _appSettings = appSettings;
            _service1 = service1;
            _service2 = service2;
        }

        public void Method()
        {
            _service1.GetData();
        }
    }
    ```

### 簡單工廠

- 用於取代高耦合的New instance，適合工廠變動性不大的情境
- 可搭配Enum建立不同的實體，符合LSP
- 優點: 簡單暴力，可避免高階方法直接依賴低階方法
- 缺點: 靜態方法尤其限制性，當參數多、需要抽換工廠時較不夠用

    ```C#
    // SimpleFactory.cs
    public class SimpleFactory
    {
        public static IService CreateService()
        {
            return new MyService();
        }

        public static IService CreateService(ServiceType type)
        {
            switch (type)
            {
                case ServiceType.Service1:
                    return new MyService1();
                case ServiceType.Service2:
                    return new MyService2();
            }
        }
    }

    public enum ServiceType
    {
        Service1,
        Service2,
    }
    ```

    ```C#
    // Program.cs
    public static Main(string[] args)
    {
        IService myService = SimpleFactory.CreateService();
        //myService...
    }
    ```

### 抽象工廠

- 適合需擴增工廠的情境、或工廠有需要input parameters的情況

    ```C#
    // AbstractFactory.cs
    public class AbstractFactory : IAbstractFactory
    {
        public AbstractFactory(/*...*/)
        {
            //...
        }
    
        public static IService CreateService()
        {
            return new MyService(/*...*/);
        }
    }
    ```

    ```C#
    // Program.cs
    public static Main(string[] args)
    {
        IAbstractFactory abstractFactory = new AbstractFactory(/*...*/);  // 方便示意，可以用DI時建議替換掉寫法
        IService myService = abstractFactory.CreateService();
        //myService...
    }
    ```

### 靜態工廠方法（Static Factory Method）

- 使用static function封裝物件建立邏輯，提供有意義的方法名稱來建立實例
- 與簡單工廠的差異: 靜態工廠方法通常定義在"產品class本身內"，而簡單工廠是獨立的工廠類別
- 優點
  1. 方法名稱更有意義，比起 constructor 更能表達建立意圖
  2. 可以控制實例的建立（如快取、單例等
  3. 可以返回subclass或interface
  4. 封裝 constructor，提供更簡潔的API
- 缺點
  1. 靜態方法無法被覆寫
  2. 如果只有靜態方法，class無法被繼承
- 使用情境: 當建構邏輯複雜、或需要提供特定建立方式時

    ```C#
    // Color.cs
    public class Color
    {
        public int R { get; }
        public int G { get; }
        public int B { get; }

        private Color(int r, int g, int b)
        {
            R = r;
            G = g;
            B = b;
        }
    
        public static Color Red()
        {
            return new Color(255, 0, 0);
        }

        public static Color Green()
        {
            return new Color(0, 255, 0);
        }

        public static Color Blue()
        {
            return new Color(0, 0, 255);
        }

        public static Color FromRgb(int r, int g, int b)
        {
            return new Color(r, g, b);
        }
    }
    ```

    ```C#
    // Program.cs
    public static void Main(string[] args)
    {
        Color red = Color.Red();
        Color customColor = Color.FromRgb(128, 64, 32);
        
        Console.WriteLine($"Red: R={red.R}, G={red.G}, B={red.B}");
        Console.WriteLine($"Custom: R={customColor.R}, G={customColor.G}, B={customColor.B}");
    }
    ```

## 策略模式

- 適合常拓展的方法，撰寫新的Class繼承特定Strategy介面，使用時進行抽換
- 優點: 好維護、易抽換、符合SRP
- 缺點: 會產生很多的Class

    ```C#
    public interface IStrategy
    {
        string GetData();
    }

    public class StrategyApi : IStrategy
    {
        public string GetData()
        {

        }
    }

    public class StrategyDb : IStrategy
    {
        public string GetData();
        {

        }
    }

    ```

    ```C#
    static void Main(string[] args)
    {
        IStrategy str1 = new StrategyApi();
        IStrategy str2 = new StrategyDb();

        if (args.Length > 0)
        {
            str1.GetData();
        }
        else
        {
            str2.GetData();
        }
    }
    ```

- 適合搭配工廠模式抽換

    ```C#
    public class SimpleFactory
    {
        public static IStrategy CreateService(ServiceType type)
        {
            switch (type)
            {
                case ServiceType.Service1:
                    return new StrategyApi();
                case ServiceType.Service2:
                    return new StrategyDb();
            }
        }
    }
    ```

    ```C#
    static void Main(string[] args)
    {
        if (args.Length > 0)
        {
            SimpleFactory.CreateService(ServiceType.Service1).GetData();
        }
        else
        {
            SimpleFactory.CreateService(ServiceType.Service2).GetData();
        }
    }
    ```

## Template 範本模式

- 以Abstract class進行實作的模式
- 優點
  1. 可共用一些方法，減少重複的程式碼
  2. 可在Abstract class定義好固定流程，再由繼承class實做特定細節流程，求同存異
- 缺點
  1. 對實體Class進行高耦合繼承，須留意相依性，尤其需要**非常注意盡量避免override virtual method**，否則繼承階層一多起來，底層Class改動virtual方法將影響整個繼承鍊(極高風險產生難解的BUG)
  2. Constructor會互相影響，底層Class新增Parameter時會需要修改所有繼承的Class
- 建議
  1. 少用Virtual，盡量只使用Abstract method進行Override，避免繼承鍊的破壞
  2. 明確區分private & protected，不必要給繼承class使用的method盡量使用private

## Singleton 單例模式

- 只new一個實體，日後直接共用該實體
- 優點
  1. 資源效率最大化，可節省記憶體
  2. 降低耦合度，只在特定地方new instance
- 缺點
  1. 需特別注意共用的情況，由其該class有property時，多個reference改變其值會有不可預測的風險
  2. 如共用db connection，須注意transaction lock的狀況
  
    ```C#
    public class SingletonClass
    {
        private readonly _myService = new MyService();

        public MyService GetService()
        {
            return _myService;
        }
    }
    ```

## Adapter 適配器模式

- 優點: 在不改變現有Class的情況下，製造出符合Interface要求的內容
- 缺點: 增加程式碼複雜度，可能會降低系統效能（多一層包裝）
- 使用情境: 當現有class的interface間不匹配時，或需要讓不相容的class能夠協同工作

    ```C#
    // 程式端需求的介面
    interface ITarget
    {
        public void Query();
    }

    // 轉換前的物件
    class OriginalClass
    {
        public void SpecificQuery()
        {
            System.Console.WriteLine("run specificRequest in Adaptee class");
        }
    }

    // 轉換器模式 → 符合介面
    class ObjectAdapter : ITarget
    {
        private OriginalClass _adaptee;

        public ObjectAdapter(OriginalClass adaptee)
        {
            // 注入原本的類別
            _adaptee = adaptee;
        }
        public void Query()
        {
            _adaptee.SpecificQuery();
        }
    }

    // 實際跑起來的樣子
    public class Program
    {
        static void Main(string[] args)
        {
            OriginalClass adaptee = new OriginalClass();
            ITarget target = new ObjectAdapter(adaptee);
            target.Query();
        }
    }
    ```

## Prototype 原型模式

- 取代constructor，使用clone方式建立現有class的instance

- 優點
    1. 避免複雜的初始化過程，直接複製現有物件
    2. 可以在運行時動態添加或移除產品
    3. 減少sub class的數量
- 缺點
    1. 處理recurring物件很困難
    2. shallow/deep copy需要謹慎處理
- 使用情境
    1. 創建物件的代價比較大（如需要大量計算或資料庫操作）
    2. 需要創建大量相似物件時
    3. 物件的初始化狀態種類有限且預先定義

    ```C#
    // Product.cs
    public class Product : ICloneable
    {
        public string Name { get; set; }
        public decimal Price { get; set; }
        public string Category { get; set; }

        public Product(string name, decimal price, string category)
        {
            Name = name;
            Price = price;
            Category = category;
        }

        // Shallow Copy
        public object Clone()
        {
            return this.MemberwiseClone();
        }

        // Deep Copy (如果有reference type需要deep copy)
        public Product DeepClone()
        {
            return new Product(this.Name, this.Price, this.Category);
        }
    }
    ```

    ```C#
    // Program.cs
    public static void Main(string[] args)
    {
        // 建立原型物件
        Product originalProduct = new Product("Laptop", 30000, "Electronics");
        
        // 使用 Clone 複製物件
        Product clonedProduct = (Product)originalProduct.Clone();
        clonedProduct.Name = "Gaming Laptop";
        clonedProduct.Price = 50000;
        
        Console.WriteLine($"Original: {originalProduct.Name} - ${originalProduct.Price}");
        Console.WriteLine($"Cloned: {clonedProduct.Name} - ${clonedProduct.Price}");
        
        // Output:
        // Original: Laptop - $30000
        // Cloned: Gaming Laptop - $50000
    }
    ```

## Chain of Responsibility 責任鏈模式

- 將流程以鏈狀handler接續(一個ref一個)，直到被處理為止
- 優點
  1. 降低request與response的耦合度
  2. 增強靈活性，可動態新增或修改handler
  3. 符合單一職責原則(SRP)，每個處理者只負責自己的處理邏輯
  4. 符合開放封閉原則(OCP)，新增處理者不需修改現有程式碼
- 缺點
  1. Request可能不被處理（沒有handler接手）
  2. 較難debug，需要追蹤整條鏈
  3. 效能問題，請求可能需要遍歷整條鏈
- 使用情境
  1. 多個物件可以處理同一request，但具體由哪個物件處理在運行時決定
  2. 需要動態指定處理請求的物件集合
  3. 審批流程、事件處理、日誌記錄等場景

    ```C#
    // Handler.cs
    public abstract class ApprovalHandler
    {
        protected ApprovalHandler _nextHandler;

        public void SetNext(ApprovalHandler handler) => _nextHandler = handler;

        public abstract string ProcessRequest(decimal amount);
    }
    ```

    ```C#
    // ConcreteHandlers.cs
    public class ManagerHandler : ApprovalHandler
    {
        public override string ProcessRequest(decimal amount)
        {
            if (amount <= 10000)
                return $"Manager approved ${amount}";
            
            return _nextHandler?.ProcessRequest(amount) ?? "Request rejected";
        }
    }

    public class DirectorHandler : ApprovalHandler
    {
        public override string ProcessRequest(decimal amount)
        {
            if (amount <= 50000)
                return $"Director approved ${amount}";
            
            return _nextHandler?.ProcessRequest(amount) ?? "Request rejected";
        }
    }

    public class CEOHandler : ApprovalHandler
    {
        public override string ProcessRequest(decimal amount)
        {
            if (amount <= 100000)
                return $"CEO approved ${amount}";
            
            return "Request rejected - exceeds limit";
        }
    }
    ```

    ```C#
    // Program.cs
    public static void Main(string[] args)
    {
        // 建立處理鏈
        var manager = new ManagerHandler();
        var director = new DirectorHandler();
        var ceo = new CEOHandler();

        manager.SetNext(director);
        director.SetNext(ceo);

        // 發送請求
        Console.WriteLine(manager.ProcessRequest(5000));    // Manager approved $5000
        Console.WriteLine(manager.ProcessRequest(25000));   // Director approved $25000
        Console.WriteLine(manager.ProcessRequest(80000));   // CEO approved $80000
        Console.WriteLine(manager.ProcessRequest(150000));  // Request rejected - exceeds limit
    }
    ```

- 補充: 可用動態instance進行chain接續，順序通常由class name決定 (也可以用int property自行設定排序)

    ```C#
    // 使用 Reflection 動態建立責任鏈
    public static void Main(string[] args)
    {
        // 取得所有繼承 ApprovalHandler 的類型
        var handlerTypes = Assembly.GetExecutingAssembly()
            .GetTypes()
            .Where(t => t.IsSubclassOf(typeof(ApprovalHandler)) && !t.IsAbstract)
            .ToList();

        // 建立所有 handler 實例並依照 class name 排序
        var handlers = handlerTypes
            .OrderBy(t => t.Name)
            .Select(t => (ApprovalHandler)Activator.CreateInstance(t))
            .ToList();

        // 自動設定責任鏈
        for (int i = 0; i < handlers.Count - 1; i++)
        {
            handlers[i].SetNext(handlers[i + 1]);
        }

        // 使用責任鏈的第一個 handler
        if (handlers.Any())
        {
            var firstHandler = handlers.First();
            Console.WriteLine(firstHandler.ProcessRequest(5000));
            Console.WriteLine(firstHandler.ProcessRequest(25000));
            Console.WriteLine(firstHandler.ProcessRequest(80000));
        }
    }
    ```

