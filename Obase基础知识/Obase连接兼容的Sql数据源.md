在Obase中,目前提供了五种数据库语法类型的支持,并且为他们提供了使用官方驱动的提供器包,在连接这些数据库官方实现时,只需要引入对应的包并且继承提供器包内的提供器基类即可,这部分内容可以参考[Obase的配置提供器有哪些可以重写的方法和属性](./Obase的配置提供器有哪些可以重写的方法和属性.md)这篇文档.

但在有些情况下,我们可能不能使用官方的连接驱动或者有其他的需求,这是就不能使用[Obase的配置提供器有哪些可以重写的方法和属性](./Obase的配置提供器有哪些可以重写的方法和属性.md)里所提到的提供器包了,但Obase其实是可以连接这些兼容这五种语法的数据库的,接下来就介绍如何连接.

## dotNet

Obase在Obase.Providers.Sql这一包中提供了语法的兼容,而诸如Obase.Providers.MySql,Obase.Providers.Sqlite等等这些包其实只是提供了对Obase.Providers.Sql内SqlContextConfigProvider的包装类而已.

所以只需要将上下文配置提供者的继承基类改为SqlContextConfigProvider即可实现使用用户配置的驱动连接数据源.

代码如下:

```
 /// <summary>
 ///     使用自定义驱动的示例上下文配置
 /// </summary>
 public class SqlContextConfiguration : SqlContextConfigProvider
 {
     /// <summary>使用指定的建模器创建对象数据模型。</summary>
     /// <param name="modelBuilder">对象数据模型建造器</param>
     protected override void CreateModel(ModelBuilder modelBuilder)
     {
         //在这里实现模型创建逻辑
     }

     /// <summary>由派生类实现，获取特定于数据库服务器（SQL Server、Oracle等）的数据提供程序工厂。</summary>
     protected override DbProviderFactory DbProviderFactory  => MySqlConnectorFactory.Instance;

     /// <summary>由派生类实现，获取数据库连接字符串。</summary>
     protected override string ConnectionString => "这里是数据库连接字符串";

     /// <summary>获取数据源类型。</summary>
     protected override EDataSource SourceType => EDataSource.MySql;
 }
```
在这个类里我们需要重写以下属性:
1. SourceType,一个枚举,可以为以下几种值SqlServer,Oracle,MySql,SqlitePostgreSql,会影响相应执行的Sql语句语法.
2. ConnectionString,数据库的连接字符串.
3. DbProviderFactory,驱动的数据提供程序工厂,此处示例使用的是MySqlConnector的工厂,实际使用时替换为引用的驱动中提供的数据提供程序工厂即可.
当然,还需要实现CreateModel方法来创建对象数据模型.