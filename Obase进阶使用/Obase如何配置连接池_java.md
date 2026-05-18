在通常情况下,使用Obase默认配置的连接池配置就可以较好地完成数据源连接的池化操作,但在某些情况下仍然需要对连接池的某些配置进行调整.

Obase提供了使用依赖注入配置接口实现的方式配置连接池.

首先,定义一个实现IObaseConnectionPoolConfiguration的类.

```
/**
 * 连接池配置
 */
public class ConnectionPoolConfiguration implements IObaseConnectionPoolConfiguration {
    /**
     * 连接池的名称 如果为空或空字符串 则使用默认值Obase ConnectionPool
     *
     * @return 连接池的名称
     */
    @Override
    public String name() {
        return "SpringBoot Connection Pool";
    }

    /**
     * 连接池的最大大小 如果小于等于0 则使用默认值-1
     *
     * @return 连接池的最大大小
     */
    @Override
    public int getMaximumPoolSize() {
        return 1;
    }
}
```
然后,进行依赖注入:
```
//构造Obase的依赖注入容器
ServiceContainerBuilder builder = ObaseDependencyInjection.createBuilder(DataContext.class);
//注入Obase连接池配置
builder.addSingleton(IObaseConnectionPoolConfiguration.class, ConnectionPoolConfiguration.class);
//建造Obase的依赖注入容器
builder.build();
```
之后就可以使用[Obase如何查看连接池信息(java版)](./Obase如何查看连接池信息_java.md)中介绍的方法查看连接池的信息.

如果我们想在SpringBoot里管理连接池配置并且从配置中读取相应的值,那么我们需要对连接池配置做如下的修改:

```
/**
 * 连接池配置
 */
@Component
public class ConnectionPoolConfiguration implements IObaseConnectionPoolConfiguration {

    /**
     * 连接池名称
     */
    @Value("${poolName}")
    private String name;

    /**
     * 连接池大小
     */
    @Value("${poolSize}")
    private int size;

    /**
     * 连接池的名称 如果为空或空字符串 则使用默认值Obase ConnectionPool
     *
     * @return 连接池的名称
     */
    @Override
    public String name() {
        return name;
    }

    /**
     * 连接池的最大大小 如果小于等于0 则使用默认值-1
     *
     * @return 连接池的最大大小
     */
    @Override
    public int getMaximumPoolSize() {
        return size;
    }
}
```
用Component标注将ConnectionPoolConfiguration注册为SpringBoot的Bean,并且从配置中读取相应的值.

之后在启动类里做如下配置:
```
//获取配置上下文 用于之后获取SpringBoot的Bean
ConfigurableApplicationContext applicationContext = SpringApplication.run(NoticeDemoApplication.class, args);
//构造Obase的依赖注入容器
ServiceContainerBuilder builder = ObaseDependencyInjection.createBuilder(DataContext.class);
//注入Obase连接池配置
builder.addSingleton(IObaseConnectionPoolConfiguration.class, ConnectionPoolConfiguration.class, p -> applicationContext.getBean(ConnectionPoolConfiguration.class));
//建造Obase的依赖注入容器
builder.build();
```
此处主要的修改为Obase依赖注入方法改为使用委托作为参数的注入方法,并在获取到ConfigurableApplicationContext后从SpringBoot的IOC容器中获取ConnectionPoolConfiguration服务作为注入Obase的实例.

如果要和预热器一同使用,那么需要注意的是要在预热逻辑执行之前向Obase注入连接池的配置,示例如下:
```
//省略一些向SpringBoot依赖注入上下文工厂的的标注代码和类定义
ConfigurableApplicationContext applicationContext = SpringApplication.run(NoticeDemoApplication.class, args);
//构造Obase的依赖注入容器
ServiceContainerBuilder builder = ObaseDependencyInjection.createBuilder(DataContext.class);
//注入Obase连接池配置
builder.addSingleton(IObaseConnectionPoolConfiguration.class, ConnectionPoolConfiguration.class, p -> applicationContext.getBean(ConnectionPoolConfiguration.class));
//建造Obase的依赖注入容器
builder.build();

//获取工厂
DataContextFactory factory = applicationContext.getBean(DataContextFactory.class);
//生产上下文
DataContext dataContext = factory.generateContext();
//构造预热器
ObasePreHeater preHeater = new ObasePreHeater();
//预热
preHeater.preHeat(dataContext);
```
否则预热器在预热连接池时,由于还没有注入配置,会导致使用默认的配置构造连接池.

此处省略的向SpringBoot依赖注入上下文工厂的的标注代码和类定义是在[Obase的上下文管理(java版)](../Obase入门/Obase的上下文管理_java.md)中介绍的上下文工厂注入SpringBoot的代码.