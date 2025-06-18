## dotNet

在dotNet版本中,可以使用扩展方法和结构化类型扩展来配置多租户.

### 配置方法

首先,需要在引用Obase本体及数据源提供器之外,引用多租户插件.

通常情况下,由于引用传递只需要引用对应的数据源提供器和多租户插件即可.配置文件类似如下形式:


```
<PackageReference Include="Obase.MultiTenant" Version="6.3.0" />
<PackageReference Include="Obase.Providers.MySql" Version="6.3.0" />
```

在引用了多租户插件之后,模型配置中增加了一种新的扩展:MultiTenantExtensionConfiguration<TEntity>,使用如下代码即可配置在结构化类型的扩展中:


```
//此处省略了一些配置结构化类型的代码 鉴定已经将类Domain配置为实体型
//创建逻辑删除扩展
var multiTenantEx = multiTenant.HasExtension<MultiTenantExtensionConfiguration<Domain>>();
//当类中有定义逻辑删除字段时 指定为逻辑删除标记
multiTenantEx.HasTenantIdMark(p => p.MultiTenantId);
//当类中没有定义逻辑删除字段时 需要指定逻辑删除映射字段
multiTenantEx.HasTenantIdField("MultiTenantId");
//为此类型配置全局多租户ID
multiTenantExtX.HasGlobalTenantId("0000");
//是否启用全局租户ID查询
multiTenantExtX.HasLoadingGlobal(true);
```

此处可以分为两种配置,一种是类内有定义多租户的字段,一种是没有定义多租户的字段.

当然,对于Sql数据源,这两种情况都需要在映射表内存在多租户的字段.

第5行就是类内有定义多租户字段的情况,此种情况需要指定哪个属性是多租户字段.

第7行时类内没有定义多租户字段的情况,此时仅需要配置数据源中哪个字段是多租户即可.

这两行配置根据需要选择其中一个即可.

第9行是配置全局的租户ID方法,此方法用于在某些情况下,查询时不止要查询属于当前租户的对象同时也要查询属于全局的(如平台)对象,此处即可设置全局的租户ID.

调用此方法会同时将是否查询时同时查询全局租户ID设置为true;

第10行配置是否查询时同时查询全局租户ID,true即是查询,false是不查询.

在配置了多租户之后,如果要使用多租户还需要再查询或保存对象前,对上下文启用多租户,代码如下:


```
//省略一些构造上下文的代码 此处假定上下文已经构造出来了
context.EnableMultiTenant();
```

启用多租户后,此上下文对象会在查询时默认过滤多租户字段为获取到的值的记录并且在保存时为多租户字段赋值.

此外,还需要自行实现ITenantIdReader并在应用层将ITenantIdReader作为单例注入至Obase的依赖注入框架中,具体的实现见下方示例部分.

### 示例

示例使用.net6版本的ASP.NET CORE进行,数据源为MySql.

首先,定义要进行测试的实体类,这里定义了两个类,Record为没有在类内定义多租户字段的,RecordX为在类内定义多租户字段的:


```
/// <summary>
///     简单记录
/// </summary>
public class Record
{
    /// <summary>
    ///     主键
    /// </summary>
    public int Id { get; set; }

    /// <summary>
    ///     内容
    /// </summary>
    public string? Content { get; set; }
}

/// <summary>
///     有租户ID的简单记录
/// </summary>
public class RecordX
{
    /// <summary>
    ///     主键
    /// </summary>
    public int Id { get; set; }

    /// <summary>
    ///     多租户ID
    /// </summary>
    public string? MultiTenantId { get; set; }

    /// <summary>
    ///     内容
    /// </summary>
    public string? Content { get; set; }
}
```

然后进行配置.


```
var record = modelBuilder.Entity<Record>();
record.HasKeyAttribute(p => p.Id).HasKeyIsSelfIncreased(true);
//创建多租户扩展
var multiTenantExt = record.HasExtension<MultiTenantExtensionConfiguration<Record>>();
//当类中未定义多租户字段时 需要指定字段设置字段和类型
multiTenantExt.HasTenantIdField("MultiTenantId", typeof(string));

var recordX = modelBuilder.Entity<RecordX>();
recordX.HasKeyAttribute(p => p.Id).HasKeyIsSelfIncreased(true);
//创建多租户扩展
var multiTenantExtX = recordX.HasExtension<MultiTenantExtensionConfiguration<RecordX>>();
//当类中未定义多租户字段时 需要指定字段设置字段和类型
multiTenantExtX.HasTenantIdMark(p=>p.MultiTenantId);
```

再定义一个多租户ID读取器TenantReader:


```
/// <summary>
///     租户ID读取器
/// </summary>
public class TenantReader : ITenantIdReader
{
    /// <summary>获取租户ID</summary>
    /// <returns></returns>
    public object GetTenantId()
    {
        //因为IHttpContextAccessor是ASP.NET 管理的 所以这里使用了ServiceLocator保存了依赖注入容器
        //用于获取IHttpContextAccessor
        var accessor = ServiceLocator.HttpContextAccessor;
        if (accessor is { HttpContext: not null })
            //此处演示从查询字符串里读取
            //也可以从头里读取令牌处理成用户ID之类的 在类中定义的多租户属性为string 故此处进行了转换
            return accessor.HttpContext.Request.Query["tenantId"].ToString();

        return string.Empty;
    }
}
```

这里面使用了ServiceLocator类的HttpContextAccessor属性来获取查询字符串,这里其实是因为.net的HttpContext是依赖注入的,所以需要我们自己构造一个类似于HttpContext.Current的属性来获取当前的Http上下文.

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
    ///     当前上下文访问者
    /// </summary>
    public static IHttpContextAccessor? HttpContextAccessor => Instance?.GetService<IHttpContextAccessor>();
}
```

最后进行依赖注入,对于.net项目,可以和WebApplication的builder一起在启动文件里进行注入.代码如下:


```
 var builder = WebApplication.CreateBuilder(args);

 builder.Services.AddControllers();

 //在ASP.NET里注入数据上下文
 builder.Services.AddTransient<ObaseConfiguration>();
 builder.Services.AddTransient<DataContext>();
 //注入Http上下文访问器
 builder.Services.AddHttpContextAccessor();
 //为Obase注入多租户读取器
 var oBuilder = ObaseDependencyInjection.CreateBuilder<DataContext>();
 oBuilder.AddSingleton<ITenantIdReader, TenantReader>();
 oBuilder.Build();

 var app = builder.Build();

 //用一个静态类保存当前注入的服务
 ServiceLocator.Instance = app.Services;

 app.UseRouting();
 app.UseEndpoints(endpoints => { endpoints.MapControllers(); });

 app.Run();
```

这里的第10行到第12行就是将ITenantIdReader注入Obase中的代码,注意创建依赖注入建造器时需要指定上下文的类型,且多租户需要的注入类型是ITenantIdReader,不能只将实现类的类型注入.

## Java

java版待重写