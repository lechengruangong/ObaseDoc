有些时候我们需要在创建了新对象,修改某个对象或者删除了某个对象并保存之后,向其他的系统或者中间件发送通知来实现内部协作.为了应对这个需求,Obase提供了对象变更通知的功能.

## 配置方法
对象更改通知是包含在Obase本体中的,因此不需要额外引用软件包,通常情况下,由于引用传递只需要引用对应的数据源提供器即可.

在模型配置中增加以下几处配置,以下代码中所有的noticeEntityConfig就是要在变更后进行通知的实体型配置对象:
```C#
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

Obase使用依赖注入来实现对具体通知逻辑的解耦,所以在配置了要通知的时机和内容后,还要向Obase注入IChangeNoticeSender更改通知发送器的具体实现.

```C#
//为Obase注入消息发送器
var oBuilder = ObaseDependencyInjection.CreateBuilder<DataContext>();
oBuilder.AddSingleton<IChangeNoticeSender, ChangeNoticeSender>();
oBuilder.Build();
```

此处的DataContext就是要注入的上下文类型,ChangeNoticeSender就是IChangeNoticeSender的具体实现类,并且需要保证这段依赖注入代码仅运行一次

在配置了对象更改通知之后,如果要使用还需要在查询或保存对象前对上下文启用对象通知,以下代码中的context就是对象上下文:

```C#
//省略一些构造上下文的代码 此处假定上下文已经构造出来了
context.EnableChangeNotice();
```

启用更改通知后,此上下文对象会在保存时针对配置进行通知.如果需要某个上下文默认启用更改通知,可以在构造方法里调用此方法.

此时在配置对应的对象新建,修改或者删除并保存时,就会调用ChangeNoticeSender发送修改通知,IChangeNoticeSender中方法Send的参数ChangeNotice就是具体的通知内容,根据不同的变更类型分别有两个具体的实现类ObjectChangeNotice和DirectlyChangingNotice.

通知对象内部包含了对象变更行为,对象的属性及其取值,对象标识等信息,根据具体的需要取值处理即可.

接下来我们以一个具体的场景为例来介绍这个功能.

## 示例

考虑一个如下的场景,我们网站上某个页面有个广告,每当用户点击这个广告时,要记录下点击广告的用户ID,广告的ID和点击时间.

这些记录被保存到数据库时,还要同时向一个其他系统的中间件发送消息用于统计.

示例使用.net6版本的ASP.NET CORE进行,数据源为MySql,那么引用如下的包:

```xml
<PackageReference Include="Obase.Providers.MySql" Version="x.x.x" />
```

### 定义实体

首先,定义实体类,这里定义了一个广告记录类,表示用户点击广告时的广告记录.

```C#
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

```C#
//配置一个实体型
var record = modelBuilder.Entity<AdvRecord>();
record.HasKeyAttribute(p => p.UserId).HasKeyAttribute(p => p.AdvId).HasKeyIsSelfIncreased(false);
//通知所有属性
record.HasNoticeAttributes();
//在创建时通知
record.HasNotifyCreation(true);
```

再定义一个更改消息发送器ChangeNoticeSender:

```C#
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
        //实际的逻辑可能是将消息处理后发往Hbase/Redis之类的第三方服务
        //此处仅在Console里显示一下
        Console.WriteLine(notice);
    }
}
```

实际使用时,ChangeNoticeSender的Send内改为自己的具体逻辑即可.

```

最后进行依赖注入,对于asp.net项目,可以和WebApplication的builder一起在启动文件里进行注入.代码如下:


```C#
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

 //用一个静态类保存当前注入的服务
 ServiceLocator.Instance = app.Services;

 app.UseRouting();
 app.UseEndpoints(endpoints => { endpoints.MapControllers(); });

 app.Run();
```

这里的第13行到第15行就是将IChangeNoticeSender注入Obase中的代码,注意创建依赖注入建造器时需要指定上下文的类型,且更改通知需要的注入类型是IChangeNoticeSender,不能只将实现类的类型注入.