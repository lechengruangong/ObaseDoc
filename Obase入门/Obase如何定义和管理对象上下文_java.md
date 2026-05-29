Obase的上下文指的是继承了ObjectContext的类实例,在Obase中上下文是一种轻量的资源,用于保存在此上下文中附加(调用attach方法)的新对象和查询出来的旧对象的状态,并在保存更改时分析和追踪这些对象的改动最终形成最终要进行修改和保存的对象图.

所以根据上下文的设计意图,Obase的上下文应当随需要进行构造而不是在应用程序域中使用单例的上下文.

相应的,在Obase中较重的资源则是Obase对象数据模型和数据源连接池,默认情况下Obase对象数据模型会在首次初始化上下文时建造并保存至以上下文类型为键的缓存中,而数据源连接池则会在首次进行数据源操作时建造并保存至以数据源连接字符串为键的缓存中.

### 一般情况下

一般情况下,即不使用其他IOC框架管理上下文的情况下,应当以实现某个场景或者功能为粒度来使用上下文.

比如一个桌面程序,注册和登陆这两个功能所调用的方法就应当使用不同的上下文,代码类似于:

```
 /**
 * 注册方法
 * @param reg 注册信息
 */
public void registry(RegisteObject reg)
{
    //构造上下文
    SampleContext context = new SampleContext(new Configuration());

    //注册操作

    //保存
    context.saveChanges();
}
 
 /**
 * 登陆方法
 * @param reg 登陆信息
 */
 public void login(LoginObject login)
 {
     //构造上下文
     SampleContext context = new SampleContext(new Configuration());

     //登陆验证

 }
```

总则就是在不同的场景或者功能中使用不同的上下文.

### 使用IOC框架的情况下

在使用IOC框架时,就应当将上下文在框架中注册为多例的,但现在较为流行的SpringBoot则默认将服务注册为单例的且注册为多例的服务较为繁琐,所以介绍一个将使用工厂来创建上下文的类注册为SpringBoot服务的例子.

首先,定义一个上下文配置提供器,并在构造函数内注入JDBC的连接字符串,用户名和密码.

```
/**
 * 上下文配置提供者
 */
public class Configuration extends MySqlContextConfigProvider {

    /**
     * JDBC连接字符串
     */
    private final String connectionString;

    /**
     * JDBC用户名
     */
    private final String connectionUserName;

    /**
     * JDBC密码
     */
    private final String connectionPassWord;

    /**
     * 初始化上下文配置提供者
     * @param connectionString JDBC连接字符串
     * @param connectionUserName JDBC用户名
     * @param connectionPassWord JDBC密码
     */
    public Configuration(String connectionString, String connectionUserName, String connectionPassWord) {
        this.connectionString = connectionString;
        this.connectionUserName = connectionUserName;
        this.connectionPassWord = connectionPassWord;
    }

    /**
     * 由派生类实现，获取数据库连接字符串
     *
     * @return 数据库连接字符串
     */
    @Override
    protected String getConnectionString() {
        return connectionString;
    }

    /**
     * 由派生类实现 获取数据库用户名
     *
     * @return 数据库用户名
     */
    @Override
    protected String getConnectionUserName() {
        return connectionUserName;
    }

    /**
     * 由派生类实现 获取数据库密码
     *
     * @return 数据库密码
     */
    @Override
    protected String getConnectionPassWord() {
        return connectionPassWord;
    }

    /**
     * 使用指定的建模器创建对象数据模型
     *
     * @param modelBuilder 建模器
     */
    @Override
    protected void createModel(ModelBuilder modelBuilder) {
        //注册模型代码
    }
}

```

当然,你也可以使用其他方法来注入JDBC的连接字符串,用户名和密码.

然后我们定义一个上下文:
```
/**
 * 对象上下文
 */
public class DataContext extends ObjectContext {

    /**
     * 初始化对象上下文
     * @param provider 对象上下文配置提供器
     */
    public DataContext(ContextConfigProvider provider) {
        super(provider);
    }
}
```

然后定义一个对象上下文工厂,并标注为SpringBoot的组件.

```
/**
 * 对象上下文工厂
 */
@Component
public class DataContextFactory {

    /**
     * 从配置文件中获取JDBC连接字符串
     */
    @Value("${spring.datasource.url}")
    private String connectionString;

    /**
     * 从配置文件中获取JDBC用户名
     */
    @Value("${spring.datasource.username}")
    private String connectionUserName;

    /**
     * 从配置文件中获取JDBC密码
     */
    @Value("${spring.datasource.password}")
    private String connectionPassWord;

    /**
     * 生产一个对象上下文
     * @return 对象上下文
     */
    public DataContext generateContext(){
        //注入配置文件中获取JDBC信息
        Configuration configuration = new Configuration(connectionString, connectionUserName, connectionPassWord);
        return new DataContext(configuration);
    }
}
```

之后在SpringBoot的控制器里,注入对象上下文工厂并在方法里使用生产的对象上下文即可,唯一需要注意的是每次生产的对象上下文都是新对象.