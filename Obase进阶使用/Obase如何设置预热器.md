Obase在默认的配置中,模型和连接池这些资源均是懒加载的,即在首次访问时进行加载,之后都使用首次加载时创建的对象.对于一般的网站,不是时刻都处于高并发的的状态下,大多是平时有一些零散的请求,在某些特殊时刻才会处于高并发的状态先.在这种情境下懒加载可以节省资源开销,并按需处理这些资源,但有些时候我们需要在应用启动时就将模型和连接池加载以保证访问速度,于是我们引入了一个新的机制:预热器.
预热器自6.0.1版本后可用.

这里简单的介绍一下预热器在WebApplication框架下的使用.

## 与Asp.netCore结合使用

```
/// <summary>
///     继承预热器抽象类
/// </summary>
public class ObasePreHeater : SqlContextPreHeater
{
    /// <summary>
    ///     Obase预热器
    /// </summary>
    public ObasePreHeater() : base()
    {

    }
}
```

只需要继承SqlContextPreHeater即可,但需要在Program或者StartUp类里做如下配置:

```
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();
//注入Obase
builder.Services.AddTransient<ContextConfiguration>();
builder.Services.AddTransient<SampleObjectContext>();
var app = builder.Build();
//获取注入的Obase
var context = app.Services.GetService<SampleObjectContext>();
//执行预热器逻辑
result.PreHeat(context);
```

当然,如果仍需要查看连接池或者预热器的输出,可以使用Obase的依赖注入框架注入日志工厂,那么预热器代码可以写作如下的逻辑:

```c#
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();
//注入Obase
builder.Services.AddTransient<ContextConfiguration>();
builder.Services.AddTransient<SampleObjectContext>();
var app = builder.Build();
//构造Obase的依赖注入容器
var obuilder = ObaseDependencyInjection.CreateBuilder<SampleObjectContext>();
//注入当前使用的日志工厂 这里用控制台作为示例
obuilder.AddSingleton<ILoggerFactory, LoggerFactory>(_ => LoggerFactory.Create(p=> p.AddConsole()));
//建造Obase的依赖注入容器
obuilder.Build();
//获取ASP.NET容器管理的上下文
var context = app.Services.GetService<SampleObjectContext>();
var result = new ObasePreHeater();
//执行预热器逻辑
result.PreHeat(context);
```
此时会使用注入的日志工厂来输出日志.


## 与SpringBoot结合使用

待重写