## dotNet

在dotNet版本中,可以使用扩展方法和结构化类型方法来配置对象更改通知.

### 配置方法

对象更改通知是包含在Obase本体中的.

通常情况下,由于引用传递只需要引用对应的数据源提供器即可.
在模型配置中对以下几处进行配置即可

```
//此处省略了一些配置结构化类型的代码 鉴定已经将类Domain配置为实体型
//配置更改通知
//配置要进行通知的属性 这些属性即此实体型的属性 当发生特定的行为时 这些属性的值会包含在通知消息内
noticeEntityConfig.HasNoticeAttributes(new List<string> { "Description", "Background" });
//无参的方法则表示通知所有的属性 注意此方法会覆盖有参的方法
noticeEntityConfig.HasNoticeAttributes();
//指示是否在对象被创建时进行通知
noticeEntityConfig.HasNotifyCreation(true);
//指示是否在对象被删除时进行通知
noticeEntityConfig.HasNotifyDeletion(true);
//指示是否在对象被修改时进行通知
noticeEntityConfig.HasNotifyUpdate(true);
```

此处配置主要指定了以下几个配置,分别是是否在创建时通知,是否在删除时通知,是否在修改时通知和哪些字段要包含在通知中.

其中的创建时进行通知是在新对象在上下文中保存时发出通知,修改时进行通知是旧对象被修改后在上下文中保存或者使用就地修改方法(NewAttribute和IncreaseAttribute)时发出通知,删除时进行通知时在旧对象被删除后再上下文中保存或者使用就地删除方法(Delete)是发出通知.

在配置了对象更改通知之后,如果要使用还需要在查询或保存对象前,对上下文启用对象通知,代码如下:


```
//省略一些构造上下文的代码 此处假定上下文已经构造出来了
context.EnableChangeNotice();
```

启用更改通知后,此上下文对象会在保存时针对配置进行通知.如果需要某个上下文默认启用更改通知,可以在构造方法里调用此方法.

此外,还需要自行实现IChangeNoticeSender并在应用层将IChangeNoticeSender作为单例注入至Obase的依赖注入框架中,具体的实现见下方示例部分.

### 示例

示例使用.net6版本的ASP.NET CORE进行,数据源为MySql.

首先,定义要进行测试的实体类,这里定义了一个类,表示广告记录,当用户点击广告时要保存在数据库里同时还要向其他的第三方服务发送点击的信息用于统计.


```
/// <summary>
///     广告记录
/// </summary>
public class AdvRecord
{
    /// <summary>
    ///     用户ID
    /// </summary>
    public string? UserId { get; set; }

    /// <summary>
    ///     广告ID
    /// </summary>
    public string? AdvId { get; set; }

    /// <summary>
    ///     点击时间
    /// </summary>
    public DateTime ClickTime { get; set; }
}
```

然后进行配置.


```
//配置一个实体型
var record = modelBuilder.Entity<AdvRecord>();
record.HasKeyAttribute(p => p.UserId).HasKeyAttribute(p => p.AdvId).HasKeyIsSelfIncreased(false);
//通知所有属性
record.HasNoticeAttributes();
//在创建时通知
record.HasNotifyCreation(true);
```

再定义一个更改消息发送器ChangeNoticeSender:


```
/// <summary>
///     更改消息发送器
/// </summary>
public class ChangeNoticeSender : IChangeNoticeSender
{
    /// <summary>发送变更通知</summary>
    /// <param name="notice">变更通知</param>
    public void Send(ChangeNotice notice)
    {
        //可能将消息处理后发往Hbase/Redis之类的第三方服务
        //此处用日志作为模拟
        //此处收到的ChangeNotice是抽象类 有两个实现类 可以根据ChangeNotice的Type来进行区分
        //ObjectChange类型的对应是ObjectChangeNotice
        //DirectlyChanging类型对应的是DirectlyChangingNotice
        var logger = ServiceLocator.LoggerFactory;
        logger?.CreateLogger<ChangeNoticeSender>().LogWarning(JsonConvert.SerializeObject(notice));
    }
}
```

这里面使用了ServiceLocator类的LoggerFactory属性来获取查询字符串,这里其实是因为.net的日志组件是依赖注入的,所以需要我们自己构造一个静态类的属性来获取当前的日志工厂.

ServiceLocator的代码如下,其中的Instance即为依赖注入容器,是在项目的启动文件里赋值的,具体参考依赖注入部分的代码.


```
/// <summary>
///     当前所有注入的服务
/// </summary>
public static class ServiceLocator
{
    /// <summary>
    ///     服务实例
    /// </summary>
    public static IServiceProvider? Instance { get; set; }

    /// <summary>
    ///     日志工厂
    /// </summary>
    public static ILoggerFactory? LoggerFactory => Instance?.GetService<ILoggerFactory>();
}
```

最后进行依赖注入,对于.net项目,可以和WebApplication的builder一起在启动文件里进行注入.代码如下:


```
 var builder = WebApplication.CreateBuilder(args);

 builder.Services.AddControllers();

 //在ASP.NET里注入数据上下文
 builder.Services.AddTransient<ObaseConfiguration>();
 builder.Services.AddTransient<DataContext>();
 //此处省略若干日志的配置
 //指定启动端口
 builder.WebHost.UseUrls("http://*:5000");

 //为Obase注入消息发送器
 var oBuilder = ObaseDependencyInjection.CreateBuilder<DataContext>();
 oBuilder.AddSingleton<IChangeNoticeSender, ChangeNoticeSender>();
 oBuilder.Build();

 var app = builder.Build();

 //用一个静态类保存当前注入的服务
 ServiceLocator.Instance = app.Services;

 app.UseRouting();
 app.UseEndpoints(endpoints => { endpoints.MapControllers(); });

 app.Run();
```

这里的第13行到第15行就是将IChangeNoticeSender注入Obase中的代码,注意创建依赖注入建造器时需要指定上下文的类型,且更改通知需要的注入类型是IChangeNoticeSender,不能只将实现类的类型注入.

## Java

JAVA版待重写.