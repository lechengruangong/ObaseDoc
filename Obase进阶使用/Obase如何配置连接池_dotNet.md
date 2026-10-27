在通常情况下,使用Obase默认配置的连接池配置就可以较好地完成数据源连接的池化操作,但在某些情况下仍然需要对连接池的某些配置进行调整.

Obase提供了使用依赖注入配置接口实现的方式配置连接池..NET版的连接池是Obase自带的连接池,配置项与Java版(HikariCP)保持一一对应,但受自身能力限制,有四项只保留配置入口、不产生实际效果,详见下表.

## 配置项一览

| 属性 | 含义 | 默认值 | 备注 |
|---|---|---|---|
| `Name` | 连接池名称 | `Obase ConnectionPool-连接池序号` | 为空或空字符串时使用默认值 |
| `MaximumPoolSize` | 连接池最大连接数 | `100` | 小于等于0时使用默认值 |
| `MinimumIdle` | 最小空闲连接数,体现为连接池预热时建立的连接数 | `5` | 小于等于0时使用默认值 |
| `ConnectionTimeout` | 获取连接的超时时间 | `10`秒 | 小于等于`TimeSpan.Zero`时不调整 |
| `IdleTimeout` | 空闲连接的超时时间 | `20`秒 | 小于等于`TimeSpan.Zero`时不调整 |
| `ValidationTimeout` | 连接有效性校验的超时时间,体现为执行校验语句的命令超时时间 | `5`秒 | 小于等于`TimeSpan.Zero`时不调整 |
| `KeepaliveTime` | 连接保活的时间间隔,体现为后台定时检查连接可用性的间隔 | `5`秒 | 小于等于`TimeSpan.Zero`时不调整;不足一秒按一秒计算 |
| `ConnectionTestQuery` | 连接有效性校验使用的语句 | `SELECT 1` | 为空或空字符串时使用默认值 |
| `MaxLifetime` | 连接的最大存活时间 | 不限制 | **.NET版不生效**,自有连接池没有该能力 |
| `LeakDetectionThreshold` | 连接泄漏检测的阈值 | 不启用 | **.NET版不生效**,自有连接池没有该能力 |
| `AutoCommit` | 连接是否自动提交事务 | 不设置 | **.NET版不生效**,DbConnection没有该能力 |
| `ReadOnly` | 连接是否只读 | 不设置 | **.NET版不生效**,DbConnection没有该能力 |

取值约定是所有配置项一致的:字符串为`null`或空字符串、`TimeSpan`小于等于`TimeSpan.Zero`、可空类型为`null`,都表示使用默认值,即不做调整.

> 注意:6.5.6起本接口由2个成员扩展为12个成员,已有的实现类需要补齐新增的10个成员才能编译通过.不需要调整的成员返回上表中的默认值即可.

## 定义一个实现类

首先,定义一个实现IObaseConnectionPoolConfiguration的类.下面的例子只调整了名称与最大连接数,其余成员返回默认值表示不做调整:

```
/// <summary>
///     连接池配置
/// </summary>
public class ConnectionPoolConfiguration : IObaseConnectionPoolConfiguration
{
    /// <summary>连接池的名称 如果为空或空字符串 则使用默认值Obase ConnectionPool-连接池序号</summary>
    public string Name => "WebApi ConnectionPool";

    /// <summary>连接池的最大大小 如果小于等于0 则使用默认值100</summary>
    public int MaximumPoolSize => 50;

    /// <summary>最小空闲连接数 小于等于0时使用默认值5</summary>
    public int MinimumIdle => 0;

    /// <summary>获取连接的超时时间 小于等于Zero时不调整</summary>
    public TimeSpan ConnectionTimeout => TimeSpan.Zero;

    /// <summary>空闲连接的超时时间 小于等于Zero时不调整</summary>
    public TimeSpan IdleTimeout => TimeSpan.Zero;

    /// <summary>连接有效性校验的超时时间 小于等于Zero时不调整</summary>
    public TimeSpan ValidationTimeout => TimeSpan.Zero;

    /// <summary>连接保活的时间间隔 小于等于Zero时不调整</summary>
    public TimeSpan KeepaliveTime => TimeSpan.Zero;

    /// <summary>连接的最大存活时间 .NET版不生效</summary>
    public TimeSpan MaxLifetime => TimeSpan.Zero;

    /// <summary>连接泄漏检测的阈值 .NET版不生效</summary>
    public TimeSpan LeakDetectionThreshold => TimeSpan.Zero;

    /// <summary>连接是否自动提交事务 .NET版不生效</summary>
    public bool? AutoCommit => null;

    /// <summary>连接是否只读 .NET版不生效</summary>
    public bool? ReadOnly => null;

    /// <summary>连接有效性校验使用的语句 为空时使用默认值SELECT 1</summary>
    public string ConnectionTestQuery => null;
}
```

要调整生效的配置项时,把对应属性的返回值改成需要的值即可,例如:

```
/// <summary>最小空闲连接数 预热时建立10个连接</summary>
public int MinimumIdle => 10;

/// <summary>获取连接的超时时间 30秒</summary>
public TimeSpan ConnectionTimeout => TimeSpan.FromSeconds(30);

/// <summary>空闲连接的超时时间 10分钟</summary>
public TimeSpan IdleTimeout => TimeSpan.FromMinutes(10);

/// <summary>连接有效性校验的超时时间 3秒</summary>
public TimeSpan ValidationTimeout => TimeSpan.FromSeconds(3);

/// <summary>连接保活的时间间隔 30秒</summary>
public TimeSpan KeepaliveTime => TimeSpan.FromSeconds(30);

/// <summary>连接有效性校验使用的语句</summary>
public string ConnectionTestQuery => "SELECT 1";
```

如果实现类很多成员都返回默认值,也可以选择继承一个基类或者用抽象类集中提供默认值,再在子类里重写需要调整的成员,以减少重复代码.

## 进行依赖注入

然后,进行依赖注入:
```
//构造Obase的依赖注入容器
var obuilder = ObaseDependencyInjection.CreateBuilder<DataContext>();
//注入Obase连接池配置
obuilder.AddSingleton<IObaseConnectionPoolConfiguration, ConnectionPoolConfiguration>();
//建造Obase的依赖注入容器
obuilder.Build();
```
之后就可以使用[Obase如何查看连接池信息(dotNet版)](./Obase如何查看连接池信息_dotNet.md)中介绍的方法查看连接池的信息.

## 在ASP.NET里管理连接池配置

如果我们想在ASP.NET里管理连接池配置并且从配置中读取相应的值,那么我们需要对连接池配置做如下的修改:
```
/// <summary>
///     连接池配置
/// </summary>
public class ConnectionPoolConfiguration : IObaseConnectionPoolConfiguration
{
    /// <summary>
    ///     配置
    /// </summary>
    private readonly IConfiguration _configuration;

    /// <summary>
    ///     初始化连接池配置
    /// </summary>
    /// <param name="configuration">配置</param>
    public ConnectionPoolConfiguration(IConfiguration configuration)
    {
        _configuration = configuration;
    }

    /// <summary>连接池的名称 如果为空或空字符串 则使用默认值Obase ConnectionPool-连接池序号</summary>
    public string Name => _configuration.GetValue<string>("PoolName");

    /// <summary>连接池的最大大小 如果小于等于0 则使用默认值100</summary>
    public int MaximumPoolSize => _configuration.GetValue<int>("PoolSize");

    /// <summary>最小空闲连接数 小于等于0时使用默认值5</summary>
    public int MinimumIdle => _configuration.GetValue<int>("PoolMinimumIdle");

    /// <summary>获取连接的超时时间 小于等于Zero时不调整</summary>
    public TimeSpan ConnectionTimeout => TimeSpan.FromMilliseconds(_configuration.GetValue<int>("PoolConnectionTimeout"));

    /// <summary>空闲连接的超时时间 小于等于Zero时不调整</summary>
    public TimeSpan IdleTimeout => TimeSpan.Zero;

    /// <summary>连接有效性校验的超时时间 小于等于Zero时不调整</summary>
    public TimeSpan ValidationTimeout => TimeSpan.Zero;

    /// <summary>连接保活的时间间隔 小于等于Zero时不调整</summary>
    public TimeSpan KeepaliveTime => TimeSpan.Zero;

    /// <summary>连接的最大存活时间 .NET版不生效</summary>
    public TimeSpan MaxLifetime => TimeSpan.Zero;

    /// <summary>连接泄漏检测的阈值 .NET版不生效</summary>
    public TimeSpan LeakDetectionThreshold => TimeSpan.Zero;

    /// <summary>连接是否自动提交事务 .NET版不生效</summary>
    public bool? AutoCommit => null;

    /// <summary>连接是否只读 .NET版不生效</summary>
    public bool? ReadOnly => null;

    /// <summary>连接有效性校验使用的语句 为空时使用默认值SELECT 1</summary>
    public string ConnectionTestQuery => null;
}
```
改为从IConfiguration里获取相应的配置,然后在启动文件中做如下的修改:
```
var builder = WebApplication.CreateBuilder(args);

//省略一些ASP.NET的依赖注入

//在ASP.NET里注入数据上下文
builder.Services.AddTransient<ObaseConfiguration>();
builder.Services.AddTransient<DataContext>();
//在ASP.NET里注入连接池配置
builder.Services.AddSingleton<ConnectionPoolConfiguration>();

var app = builder.Build();

//获取ASP.NET管理的连接池配置
var poolConfig = app.Services.GetService<ConnectionPoolConfiguration>();

//构造Obase的依赖注入容器
var obuilder = ObaseDependencyInjection.CreateBuilder<DataContext>();
//注入Obase连接池配置
obuilder.AddSingleton<IObaseConnectionPoolConfiguration, ConnectionPoolConfiguration>( _ => poolConfig);
//建造Obase的依赖注入容器
obuilder.Build();

//省略一些ASP.NET 的服务处理

app.Run();
```

此处主要的修改为Obase依赖注入方法改为使用委托作为参数的注入方法,并在获取到WebApplication后从app.Services中获取ConnectionPoolConfiguration服务作为注入Obase的实例.

## 与预热器一同使用

如果要和预热器一同使用,那么需要注意的是要在预热逻辑执行之前向Obase注入连接池的配置,示例如下:
```
var builder = WebApplication.CreateBuilder(args);

//省略一些ASP.NET的依赖注入

//在ASP.NET里注入数据上下文
builder.Services.AddTransient<ObaseConfiguration>();
builder.Services.AddTransient<DataContext>();
//在ASP.NET里注入连接池配置
builder.Services.AddSingleton<ConnectionPoolConfiguration>();

var app = builder.Build();

//获取ASP.NET管理的连接池配置
var poolConfig = app.Services.GetService<ConnectionPoolConfiguration>();

//构造Obase的依赖注入容器
var obuilder = ObaseDependencyInjection.CreateBuilder<DataContext>();
//注入Obase连接池配置
obuilder.AddSingleton<IObaseConnectionPoolConfiguration, ConnectionPoolConfiguration>( _ => poolConfig);
//建造Obase的依赖注入容器
obuilder.Build();

//获取ASP.NET容器管理的上下文
var context = app.Services.GetService<DataContext>();
//构造预热器
var result = new ObasePreHeater();
//执行预热器逻辑
result.PreHeat(context);
//省略一些ASP.NET 的服务处理

app.Run();
```
否则预热器在预热连接池时,由于还没有注入配置,会导致使用默认的配置构造连接池.

## 注意事项

- 连接池按连接字符串区分,同一个连接字符串只会建造一个连接池,所以同一个连接字符串下的所有上下文类型共用第一个建造连接池时读到的配置.
- 配置在连接池建造时读取一次,建造完成后再修改配置不会影响已有的连接池.
- `MinimumIdle`在.NET版中体现为预热时建立的连接数,该值过大而数据库连接数有限时,预热阶段可能因为拿不到连接而失败,应根据实际承载能力设置.
- `KeepaliveTime`在.NET版中体现为后台定时检查连接可用性的间隔,以秒为单位,不足一秒按一秒计算.
- `MaxLifetime`、`LeakDetectionThreshold`、`AutoCommit`、`ReadOnly`这四项在.NET版中虽然保留在接口上,但自有连接池与`DbConnection`都没有对应能力,配置后不产生任何效果.需要限制连接存活时间时,可以改用数据库端或连接字符串自身的相关设置.
