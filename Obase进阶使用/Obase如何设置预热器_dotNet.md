Obase在默认的配置中,模型和连接池这些资源均是懒加载的,即在首次访问时进行加载,之后都使用首次加载时创建的对象.对于一般的网站,不是时刻都处于高并发的的状态下,大多是平时有一些零散的请求,在某些特殊时刻才会处于高并发的状态先.在这种情境下懒加载可以节省资源开销,并按需处理这些资源,但有些时候我们需要在应用启动时就将模型和连接池加载以保证访问速度,于是我们引入了一个新的机制:预热器.

预热器会尝试访问上下文的模型和对应的连接池,这样就可以将模型和连接池预热生成,不再占用首次访问的性能.

接下来以使用.net9版本的ASP.NET进行,数据源为MySql为示例来演示如何使用预热器.

首先,我们要定义一个继承自SqlContextPreHeater的类:
```
/// <summary>
///     Obase预热器 继承预热器抽象类
/// </summary>
public class ObasePreHeater : SqlContextPreHeater
{
}
```

只需要继承SqlContextPreHeater即可,之后在启动类里做如下配置:

```
//省略一些向ASP.NET依赖注入上下文的代码
//获取注入ASP.NET的Obase
var app = builder.Build();
var context = app.Services.GetService<DataContext>();
//执行预热器逻辑
result.PreHeat(context);
```

此处省略的向ASP.NET依赖注入上下文的代码是在[Obase的上下文管理(dotNet版)](../Obase入门/Obase的上下文管理_dotNet.md)中介绍的将上下文以多例模式注入ASP.NET服务的代码.

当然,如果仍需要查看连接池或者预热器的输出,可以使用Obase的依赖注入框架注入日志工厂,那么启动类的代码可以写作如下的逻辑:

```
//省略一些向ASP.NET依赖注入上下文和日志工厂的代码
var app = builder.Build();
//构造Obase的依赖注入容器
var obuilder = ObaseDependencyInjection.CreateBuilder<DataContext>();
//注入当前使用的日志工厂 这里用控制台作为示例
obuilder.AddSingleton<ILoggerFactory, LoggerFactory>(_ => LoggerFactory.Create(p => p.AddConsole()));
//建造Obase的依赖注入容器
obuilder.Build();
//获取ASP.NET容器管理的上下文
var context = app.Services.GetService<DataContext>();
//构造预热器
var result = new ObasePreHeater();
//执行预热器逻辑
result.PreHeat(context);
```
此时会使用注入的日志工厂来输出日志,会在日志中观察到类似于"XXX Has Initialized"的输出.
