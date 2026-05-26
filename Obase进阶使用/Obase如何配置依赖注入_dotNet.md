Obase提供了一个小型的依赖注入框架,用于对某些需要用户提供具体实现逻辑的场景的解耦,在[对象更改通知](./Obase如何配置对象更改通知_dotNet.md),[多租户](./Obase如何配置多租户_dotNet.md),[配置连接池](./Obase如何配置连接池_dotNet.md)等场景中均有使用.

虽然大部分情况下,引用Obase的应用层都往往已经引用了其他的依赖注入框架,使用Obase的依赖注入也只是要实现Obase某些功能要求的,但如果应用层没有引用依赖注入框架当然也可以使用Obase的依赖注入.

Obase的依赖注入管理的服务分为两种,一种是单例,一种是多例.单例的服务只会在首次被获取时创建,此后获取到的都是首次创建的同一个对象;多例的服务则在每次获取时创建,每次获取到的都是新对象.

由于Obase的依赖注入一开始就是设计来为上下文服务的,所以注册服务时需要先根据上下文的类型创建配置建造器,再调用配置建造器的方法来注册服务,获取服务时也需要提供上下文类型才能获取服务.

## 依赖注入方法

依赖注入是包含在Obase本体中的,因此不需要额外引用软件包,通常情况下,由于引用传递只需要引用对应的数据源提供器即可.

依赖注入主要使用的有以下几个方法:
```
//创建某个对象上下文的依赖注入建造器
var builder = ObaseDependencyInjection.CreateBuilder<SampleContext>();
//注册一个单例的服务
builder.AddSingleton<ServiceA>();
//注册一个多例的服务
builder.AddTransient<ServiceB>();
//建造依赖注入容器
builder.Build();
```

需要注意的是这些代码应当只被调用一次,重复调用Build方法会抛出异常.

Obase的依赖注入还有一些其他方法,可以参考[Obase的依赖注入Api](../Obase基础知识/Obase的依赖注入建造器Api_dotNet.md)这篇文档.

## 依赖注入的使用

在Obase.Core.Common命名空间下的Utils类提供了两个依赖注入相关的方法GetDependencyInjectionService和GetDependencyInjectionServiceOrNull,这两个方法的区别是GetDependencyInjectionService在遇到无法找到服务时会抛出异常,GetDependencyInjectionServiceOrNull则只会返回空.

```
//ServiceA ServiceB 已注册 但Program未注册 所以获取Program返回null
var serviceA = Utils.GetDependencyInjectionService<ServiceA>(typeof(SampleContext));
var serviceB = Utils.GetDependencyInjectionService<ServiceB>(typeof(SampleContext));

var program = Utils.GetDependencyInjectionServiceOrNull<Program>(typeof(SampleContext));
```

这里获取到的服务会根据注册时是单例还是多例的有不同的行为.

## 具体示例

假设我们有三个服务,都需要注册为多例的,其中服务A可以直接无参构造,服务B则需要依赖于服务A构造,服务C则需要一个DateTime参数来构造.

那么定义如下:

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

最后,要使用哪个服务使用Utils获取即可:
```
//获取ServiceA
var serviceA = Utils.GetDependencyInjectionService<ServiceA>(typeof(SampleContext));
```