Obase提供了一个简易的依赖注入容器,用于支持Obase的某些功能,如[对象更改通知](../Obase进阶使用/Obase如何配置对象更改通知.md),[多租户](../Obase进阶使用/Obase如何配置多租户.md)中就都使用了依赖注入来注入某些接口.

当然,我们也可以使用Obase的依赖注入容器来实现基础设施层(Obase通常于基础设施层引入)的依赖注入,本篇就介绍如何使用Obase的依赖注入.

## dotNet

### 依赖注入的服务注册

Obase的依赖注入中,服务的注册和使用是分离的,通过ObaseDependencyInjection的静态方法CreateBuilder<>()获取依赖注入建造器,并调用建造器的AddSingleton和AddTransient方法注册单例和多例的服务,最后调用建造器的Build方法建造依赖注入容器.
需要注意的是,ObaseDependencyInjection的静态方法CreateBuilder<>()方法的类型参数必须为继承ObjectContext的类型,即上下文类型,因为Obase的依赖注入容器是根据上下文类型加以区分的.

这里的Build方法在整个生命周期中应只调用一次,所以需要将Obase依赖注入注册的代码放置于主入口或其他只会执行一次的部分中.

#### 一般使用

在一般情况下,即所有注入的服务均由Obase负责构造和管理的情况下,此时只要考虑在注册时将构造服务的方式注册即可,看一个如下的示例:

```
/// <summary>
///     服务A 构造函数无参数
/// </summary>
public class ServiceA
{
    /// <summary>
    ///     创建时间
    /// </summary>
    public DateTime Now { get; } = DateTime.Now;
}

/// <summary>
///     服务B 构造函数有参数服务A
/// </summary>
public class ServiceB
{
    /// <summary>
    ///     服务A
    /// </summary>
    public ServiceA ServiceA { get; }

    /// <summary>
    ///     初始化服务B
    /// </summary>
    /// <param name="serviceA">服务A</param>
    public ServiceB(ServiceA serviceA)
    {
        ServiceA = serviceA;
    }
}

/// <summary>
///     服务C 构造函数有参数DateTime
/// </summary>
public class ServiceC
{
    /// <summary>
    ///     创建时间
    /// </summary>
    public DateTime CreateTime { get; }

    /// <summary>
    ///     初始化服务C
    /// </summary>
    /// <param name="createTime"></param>
    public ServiceC(DateTime createTime)
    {
        CreateTime = createTime;
    }
}
```

服务A,构造时无参数;服务B,构造时需要参数服务A;服务C,构造时需要传入DateTime.分析一下可知,服务A在依赖注入注册时无需指定构造方式,而服务B需要服务A作为构造函数参数,Obase的依赖注入可以将已注册的服务类型作为其他服务类型的构造参数,故在依赖注入注册时也无需指定构造方式,服务C则需要使用委托来确定构造方式.

所以注册代码如下:

```
//注册Obase的服务 这里假设已经定义了一个SampleContext上下文
 ObaseDependencyInjection.CreateBuilder<SampleContext>()
     .AddTransient<ServiceA>().AddTransient<ServiceB>()
     .AddTransient<ServiceC>(_ => new ServiceC(DateTime.Now))
     .Build();
```

此处需要注意的是ServiceC的注册代码,委托有一个参数是当前的服务容器,你可以通过此参数的GetService方法获取已在当前容器注册的服务实例,不过此处没有使用到这个参数.

#### 与其他IOC容器一同使用

与其他IOC容器一同使用时大体与一般情况相同,此处仅介绍如果需要向Obase依赖注入中注册由其他IOC容器管理的服务时要如何配置.

比如在ASP.NET Core的框架下,我有一个服务需要使用IConfiguration来读取配置文件,此服务是由Obase的依赖注入管理的,但构造此服务的IConfiguration则是由Asp.net Core管理的,故此时需要使用委托的方式来指定构造方法.以下为服务类的定义:

```
/// <summary>
///     Value服务
/// </summary>
public class ValueService
{
    /// <summary>
    ///     值
    /// </summary>
    public string Value { get; }

    /// <summary>
    ///     初始化Value服务
    /// </summary>
    /// <param name="configuration">配置</param>
    public ValueService(IConfiguration configuration)
    {
        Value = configuration["AllowedHosts"];
    }
}
```

此时Obase依赖注入的注册代码则必须在ASP.NET Core的启动类中,因为要借助WebApplication的GetService来获取IConfiguration的实例:

```
//获取WebApplication的建造器
var builder = WebApplication.CreateBuilder(args);

//此处省略一些代码

//建造WebApplication
var app = builder.Build();

//注册ValueService服务 假设此处已有SampleContext上下文
ObaseDependencyInjection.CreateBuilder<SampleContext>()
    .AddSingleton<ValueService>(_ => new ValueService(app.Services.GetService<IConfiguration>())).Build();
```
此处的app即为ASP.NET Core的启动类中的由WebApplicationBuilder.Build获取的WebApplication,在ValueService的注册代码委托中,从WebApplication的Service里获取IConfiguration的实例.

### 依赖注入的使用

在Obase.Core.Common命名空间下的Utils类提供了两个依赖注入相关的方法GetDependencyInjectionService和GetDependencyInjectionServiceOrNull,这两个方法的区别是GetDependencyInjectionService在遇到无法找到服务时会抛出异常,GetDependencyInjectionServiceOrNull则只会返回空.

```
//ServiceA ServiceB ServiceC已注册 但Program未注册 所以获取Program返回null
var serviceA = Utils.GetDependencyInjectionService<ServiceA>(typeof(SampleContext));
var serviceB = Utils.GetDependencyInjectionService<ServiceB>(typeof(SampleContext));
var serviceC = Utils.GetDependencyInjectionService<ServiceC>(typeof(SampleContext));

var program = Utils.GetDependencyInjectionServiceOrNull<Program>(typeof(SampleContext));
```

此处使用的注册代码为上一节一般情况下的注册代码.

Obase的依赖注入还有一些其他方法,可以参考[Obase的依赖注入方法有哪些(dotNet版)](../Obase基础知识/Obase的依赖注入方法有哪些(dotNet版).md)这篇文档.