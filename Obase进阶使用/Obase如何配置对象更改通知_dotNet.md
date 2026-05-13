有些时候我们需要在创建了新对象,修改某个对象或者删除了某个对象并保存之后向其他的系统或者中间件发送通知来实现内部协作.

为了应对这个需求,Obase提供了对象变更通知的功能.

## 配置方法
对象更改通知是包含在Obase本体中的,因此不需要额外引用软件包,通常情况下,由于引用传递只需要引用对应的数据源提供器即可.

在模型配置中有以下几个方法用于配置对象变更通知,这些方法可以由实体型配置对象或者关联型配置对象调用,以下代码中所有的noticeEntityConfig就是要在变更后进行通知的实体型或者关联型配置对象:
```
//配置要进行通知的属性方法 参数为属性的名称 当发生特定的行为时 这些属性的值会包含在通知消息内
noticeEntityConfig.HasNoticeAttributes(new List<string> { "Description", "Background" });
//无参的要进行通知的属性方法则表示通知所有的属性 注意此方法会覆盖有参的方法配置的属性
noticeEntityConfig.HasNoticeAttributes();
//指示是否在对象被创建时进行通知
noticeEntityConfig.HasNotifyCreation(true);
//指示是否在对象被删除时进行通知
noticeEntityConfig.HasNotifyDeletion(true);
//指示是否在对象被修改时进行通知
noticeEntityConfig.HasNotifyUpdate(true);
```

这些配置方法主要指定了以下几个配置内容,分别是是否在创建时通知,是否在删除时通知,是否在修改时通知和哪些字段要包含在通知中.

其中的创建时进行通知是在新对象在上下文中保存时发出通知,修改时进行通知是旧对象被修改后在上下文中保存或者使用就地修改方法(NewAttribute和IncreaseAttribute)时发出通知,删除时进行通知时在旧对象被删除后再上下文中保存或者使用就地删除方法(Delete)时发出通知.

## 依赖注入

Obase使用依赖注入来实现对具体通知逻辑的解耦,所以在配置了要通知的时机和内容后,还要向Obase注入IChangeNoticeSender更改通知发送器的具体实现.

```
//为Obase注入消息发送器
var oBuilder = ObaseDependencyInjection.CreateBuilder<DataContext>();
oBuilder.AddSingleton<IChangeNoticeSender, ChangeNoticeSender>();
oBuilder.Build();
```

此处的DataContext就是要注入的上下文类型,ChangeNoticeSender则是IChangeNoticeSender的具体实现类,并且需要保证这段依赖注入代码仅运行一次

## 启用通知

考虑到不一定是所有场景中都需要发送变更通知,需要对上下文启用对象通知才会进行通知,调用方法如下:

```
//省略一些构造上下文的代码 此处假定上下文已经构造出来了
context.EnableChangeNotice();
```

如果需要某个上下文默认启用更改通知,可以在上下文的构造方法里调用此方法.

启用更改通知后在配置对应的对象新建,修改或者删除并保存时,就会根据配置调用IChangeNoticeSender的具体实现发送修改通知,IChangeNoticeSender中方法Send的参数ChangeNotice就是具体的通知内容,根据不同的变更类型分别有两个具体的实现类ObjectChangeNotice和DirectlyChangingNotice.

通知对象内部包含了对象变更行为,对象的属性及其取值,对象标识等信息,根据具体的需要取值处理即可.

接下来我们以一个具体的场景为例来介绍这个功能.

## 具体示例

考虑一个如下的场景,我们网站上某个页面有个广告,每当用户点击这个广告时,要记录下点击广告的用户ID,广告的ID和点击时间.

这些记录被保存到数据库时,还要同时向一个其他系统的中间件发送消息用于统计.

示例使用.net9版本的ASP.NET进行,数据源为MySql,那么引用如下的包(具体版本号换成你需要的):

```
<PackageReference Include="Obase.Providers.MySql" Version="x.x.x" />
```

首先,定义实体类,这里定义了一个广告记录类,表示用户点击广告时的广告记录.

```
/// <summary>
///     广告记录
/// </summary>
public class AdvRecord
{
    /// <summary>
    ///     用户ID
    /// </summary>
    public string UserId { get; set; }

    /// <summary>
    ///     广告ID
    /// </summary>
    public string AdvId { get; set; }

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
record.ToTable(nameof(AdvRecord));
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
        //此处收到的ChangeNotice是抽象类 有两个实现类 可以根据ChangeNotice的Type来进行区分
        //ObjectChange类型的对应是ObjectChangeNotice
        //DirectlyChanging类型对应的是DirectlyChangingNotice
        //这里可以从notice里获取属性和属性值等信息
        //实际的逻辑可能是将消息处理后发往Hbase/Redis之类的第三方服务
        //此处仅在Console里显示一下
        Console.WriteLine(notice);
    }
}
```

实际使用时,ChangeNoticeSender的Send内改为自己的具体逻辑即可.

接下来进行依赖注入,对于asp.net项目,可以和WebApplication的builder一起在启动文件里进行注入.代码如下:

```
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

//在ASP.NET里注入数据上下文
builder.Services.AddTransient<ObaseConfiguration>();
builder.Services.AddTransient<DataContext>();
//此处省略若干其他的配置

//为Obase注入消息发送器
var oBuilder = ObaseDependencyInjection.CreateBuilder<DataContext>();
oBuilder.AddSingleton<IChangeNoticeSender, ChangeNoticeSender>();
oBuilder.Build();

var app = builder.Build();

app.UseRouting();
app.UseEndpoints(endpoints => { endpoints.MapControllers(); });

app.Run();
```

这里的第13行到第15行就是将IChangeNoticeSender注入Obase中的代码,注意创建依赖注入建造器时需要指定上下文的类型,且更改通知需要的注入类型是IChangeNoticeSender,不能只将实现类的类型注入.

如果IChangeNoticeSender的具体实现需要用到由ASP.Net管理的某些服务,比如需要用日志服务记录一下,那么需要修改一下ChangeNoticeSender,在构造函数中注入日志工厂:

```
/// <summary>
///     更改消息发送器
/// </summary>
public class ChangeNoticeSender : IChangeNoticeSender
{
    /// <summary>
    ///     日志工厂
    /// </summary>
    private readonly ILoggerFactory _logger;

    /// <summary>
    ///     初始化更改消息发送器
    /// </summary>
    /// <param name="logger">日志工厂</param>
    public ChangeNoticeSender(ILoggerFactory logger)
    {
        _logger = logger;
    }

    /// <summary>发送变更通知</summary>
    /// <param name="notice">变更通知</param>
    public void Send(ChangeNotice notice)
    {
        //此处收到的ChangeNotice是抽象类 有两个实现类 可以根据ChangeNotice的Type来进行区分
        //ObjectChange类型的对应是ObjectChangeNotice
        //DirectlyChanging类型对应的是DirectlyChangingNotice
        //这里可以从notice里获取属性和属性值等信息
        //此处用日志作为模拟
        _logger?.CreateLogger<ChangeNoticeSender>().LogWarning(JsonConvert.SerializeObject(notice));
    }
}
```
启动文件也要做如下的修改:
```
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

//在ASP.NET里注入数据上下文
builder.Services.AddTransient<ObaseConfiguration>();
builder.Services.AddTransient<DataContext>();
//此处省略若干其他的配置

var app = builder.Build();

//为Obase注入消息发送器
var oBuilder = ObaseDependencyInjection.CreateBuilder<DataContext>();
oBuilder.AddSingleton<IChangeNoticeSender, ChangeNoticeSender>(_ => new ChangeNoticeSender(app.Services.GetService<ILoggerFactory>()));
oBuilder.Build();

app.UseRouting();
app.UseEndpoints(endpoints => { endpoints.MapControllers(); });

app.Run();
```
此处主要的修改为Obase依赖注入方法改为使用委托作为参数的注入方法,并在获取到WebApplication后从app.Services中获取具体服务作为构造ChangeNoticeSender的参数.

最后只要和平常一样调用保存新对象的逻辑即可,唯一需要注意的是为上下文启用对象通知:

```

//此处省略从ASP.NET的依赖注入中获取context的代码
//如果有需要 可以在对象上下文的构造函数里调用EnableChangeNotice方法 这样所有构造出来的上下文就都是启用了对象通知的

//启用对象通知
context.EnableChangeNotice();
//新建对象
var record = new AdvRecord()
{
    UserId = userId,
    AdvId = advId,
    ClickTime = DateTime.Now
};
//附加并保存对象
context.Attach(record);
context.SaveChanges();
```