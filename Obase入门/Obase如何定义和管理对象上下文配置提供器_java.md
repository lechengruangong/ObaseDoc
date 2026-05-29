Obase使用配置提供器来为特定的上下文提供配置,某个特定的上下文(Context)需要配套特定的上下文配置提供器(Provider),在构造上下文时需要将配置提供器作为构造函数的参数传入.

默认情况下,一般都会选择对应的提供程序包进行重写,这些提供程序包默认实现了配置类的一些方法和属性,但仍有一些方法需要再基础设施层自行实现,在这里统一介绍一下.

目前提供适用于SQL数据源的配置类基类分别如下:

- io.obase:providers.mysql包的MySqlContextConfigProvider

- io.obase:providers.sqlite包的SqliteContextConfigProvider

- io.obase:providers.sqlserver的SqlServerContextConfigProvider

- io.obase:providers.oracle的OracleContextConfigProvider

- io.obase:providers.postgresql的PostgreSqlContextConfigProvider

### 必须实现的方法

-  protected String getConnectionString();
   此属性用于返回连接字符串.

-  protected String getConnectionUserName();
   此属性用于返回用户名.

-  protected String getConnectionPassWord();
   此属性用于返回密码.

- protected void createModel(ModelBuilder modelBuilder);
  此方法是模型注册方法,所有的模型注册代码需要在此方法内调用.此方法仅在首次构造对象数据模型时被调用,且全局只存在一个对象数据模型.

### 可选实现的方法

- protected boolean getWhetherCreateSet();
  此属性指定是否在上下文中自动为ObjectSet<>类型的属性访问器赋值,目前大多使用CreateSet<>方法创建ObjectSet<>,此属性较少使用.此外需注意启用此属性会增大构造上下文时的性能开销.

- protected boolean getEnableStructMapping();
  此属性指定是否使用模型结构映射,对于Sql数据源,启用此属性时会尝试在数据库内建表.启用此属性会在模型首次构建后进行建表,性能开销较大,推荐仅在需要建表时使用一次.详细的介绍可以参考[Obase的结构映射](../Obase进阶使用/Obase如何配置结构映射_java.md)这篇文档.

### 管理对象上下文配置提供器

对象上下文配置提供器的管理要与对象上下文的管理相结合,请阅读[Obase如何定义和管理对象上下文](./Obase如何定义和管理对象上下文_java.md)中相关的内容.
