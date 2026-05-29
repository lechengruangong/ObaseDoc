在通常情况下,使用Obase默认配置的连接池配置就可以较好地完成数据源连接的池化操作,但在某些情况下仍然需要对连接池的某些配置进行调整.

Obase提供了使用依赖注入配置接口实现的方式配置连接池.

首先,定义一个实现IObaseConnectionPoolConfiguration的类.

```
/// <summary>
///     连接池配置
/// </summary>
public class ConnectionPoolConfiguration : IObaseConnectionPoolConfiguration
{
    /// <summary>连接池的名称 如果为空或空字符串 则使用默认值Obase ConnectionPool</summary>
    public string Name => "WebApi ConnectionPool";

    /// <summary>连接池的最大大小 如果小于等于0 则使用默认值100</summary>
    public int MaximumPoolSize => 50;
}
```
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

    /// <summary>连接池的名称 如果为空或空字符串 则使用默认值Obase ConnectionPool</summary>
    public string Name => _configuration.GetValue<string>("PoolName");

    /// <summary>连接池的最大大小 如果小于等于0 则使用默认值100</summary>
    public int MaximumPoolSize => _configuration.GetValue<int>("PoolSize");
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