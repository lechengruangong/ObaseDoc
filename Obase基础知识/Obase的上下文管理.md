Obase的上下文指的是继承了ObjectContext的类实例,在Obase中上下文是一种轻量的资源,用于保存在此上下文中附加(调用Attach方法)的新对象和查询出来的旧对象的状态,并在保存更改时分析和追踪这些对象的改动最终形成最终要进行修改和保存的对象图.

所以根据上下文的设计意图,Obase的上下文应当随需要进行构造而不是在应用程序域中使用单例的上下文.

相应的,在Obase中较重的资源则是Obase对象数据模型和数据源连接池,默认情况下Obase对象数据模型会在首次初始化上下文时建造并保存至以上下文类型为键的缓存中,而数据源连接池则会在首次进行数据源操作时建造并保存至以数据源连接字符串为键的缓存中.
如果使用场景需要关注首次调用性能,可以使用[预热器](../Obase进阶使用/Obase如何设置预热器.md)来进行对象数据模型和数据源连接池的预热.

## dotNet

### 一般情况下

一般情况下,即不使用其他IOC框架管理上下文的情况下,应当以实现某个场景或者功能为粒度来使用上下文.

比如一个桌面程序,注册和登陆这两个功能所调用的方法就应当使用不同的上下文,代码类似于:

```


 /// <summary>
 ///     注册方法
 /// </summary>
 /// <param name="reg">注册信息</param>
 public void Registe(RegisteObject reg)
 {
     //构造上下文
     var context = new SampleContext();

     //注册操作

     //保存
     context.SaveChanges();
 }
 
  /// <summary>
 ///     登陆方法
 /// </summary>
 /// <param name="login">登陆信息</param>
 public void Login(LoginObject login)
 {
     //构造上下文
     var context = new SampleContext();

     //登陆验证

 }
```

总则就是在不同的场景或者功能中使用不同的上下文.

### 使用IOC框架的情况下

在使用IOC框架时,就应当将上下文在框架中注册为多例的,比如在Asp.net core中就可以进行如下的注册:

```
//获取WebApplication建造器
var builder = WebApplication.CreateBuilder(args);

//注册Obase上下文服务
builder.Services.AddTransient<SampleContextConfigProvider>();
builder.Services.AddTransient<SampleContext>();

//其他的代码
```

对应的类定义如下:

```
/// <summary>
///     示例上下文
/// </summary>
public class SampleContext : ObjectContext
{
   /// <summary>构造ObjectContext对象</summary>
   /// <param name="provider">对象上下文配置提供者</param>
   public SampleContext(SampleContextConfigProvider provider) : base(provider)
   {
   }
}

/// <summary>
///     示例上下文配置提供者
/// </summary>
public class SampleContextConfigProvider : MySqlContextConfigProvider
{
   /// <summary>
   ///     连接字符串
   /// </summary>
   private readonly string _connectionString;

   /// <summary>初始化对象上下文配置提供者</summary>
   /// <param name="configuration">配置</param>
   public SampleContextConfigProvider(IConfiguration configuration)
   {
       _connectionString = configuration.GetConnectionString("DefaultConnection");
   }

   /// <summary>使用指定的建模器创建对象数据模型。</summary>
   /// <param name="modelBuilder">对象数据模型建造器</param>
   protected override void CreateModel(ModelBuilder modelBuilder)
   {
       //此处增加对象数据模型的配置代码
   }

   /// <summary>由派生类实现，获取数据库连接字符串。</summary>
   protected override string ConnectionString => _connectionString;
}
```

这里的示例上下文配置提供者构造参数使用了IConfiguration来获取配置.