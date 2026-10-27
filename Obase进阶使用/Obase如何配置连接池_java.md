在通常情况下,使用Obase默认配置的连接池配置就可以较好地完成数据源连接的池化操作,但在某些情况下仍然需要对连接池的某些配置进行调整.

Obase提供了使用依赖注入配置接口实现的方式配置连接池.Java版的连接池由HikariCP提供,因此可配置的内容以HikariCP提供的能力为准.

## 配置项一览

`IObaseConnectionPoolConfiguration`中只有`name()`与`getMaximumPoolSize()`需要实现,其余配置项都提供了默认实现,已有的实现类无需改动即可继续使用;需要调整某项配置时,重写对应的方法即可.

| 方法 | 含义 | 默认值 | 备注 |
|---|---|---|---|
| `name()` | 连接池名称 | `Obase ConnectionPool` | 为空或空字符串时使用默认值 |
| `getMaximumPoolSize()` | 连接池最大连接数 | `10` | 小于等于0时使用默认值 |
| `getMinimumIdle()` | 最小空闲连接数 | HikariCP默认值(与最大连接数相同) | 小于等于0时不调整;不应大于最大连接数 |
| `getConnectionTimeout()` | 获取连接的超时时间(毫秒) | `30000` | 小于等于0时不调整;不应小于250 |
| `getIdleTimeout()` | 空闲连接的超时时间(毫秒) | `600000` | 小于等于0时不调整;不应小于10000 |
| `getValidationTimeout()` | 连接有效性校验的超时时间(毫秒) | `5000` | 小于等于0时不调整;不应小于250,且应小于获取连接的超时时间 |
| `getKeepaliveTime()` | 连接保活的时间间隔(毫秒) | 不启用 | 小于等于0时不启用;启用时不应小于30000 |
| `getMaxLifetime()` | 连接的最大存活时间(毫秒) | `1800000` | 小于等于0时不调整;不应小于30000 |
| `getLeakDetectionThreshold()` | 连接泄漏检测的阈值(毫秒) | 不启用 | 小于等于0时不启用;启用时不应小于2000 |
| `getAutoCommit()` | 连接是否自动提交事务 | 不设置 | 为`null`时不设置,沿用驱动或连接字符串的默认值 |
| `getReadOnly()` | 连接是否只读 | 不设置 | 为`null`时不设置,沿用驱动或连接字符串的默认值 |
| `getConnectionTestQuery()` | 连接有效性校验使用的语句 | 由驱动提供校验方式 | 为`null`或空字符串时使用默认值,驱动不支持JDBC4的`isValid`时才需要指定 |

取值约定是所有配置项一致的:字符串为`null`或空字符串、数值小于等于0、包装类型`Boolean`为`null`,都表示使用默认值,即不做调整.

时间类配置项的单位都是毫秒.取值小于HikariCP要求的范围时,HikariCP会按它自己的规则调整并输出警告日志,此时实际生效的不是配置的值.

## 定义一个实现类

首先,定义一个实现IObaseConnectionPoolConfiguration的类.下面的例子只调整了名称与最大连接数:

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
     * 连接池的最大大小 如果小于等于0 则使用默认值10
     *
     * @return 连接池的最大大小
     */
    @Override
    public int getMaximumPoolSize() {
        return 1;
    }
}
```

如果要调整更多的配置项,重写对应的方法即可,没有重写的方法保持默认行为:

```
/**
 * 连接池配置
 */
public class ConnectionPoolConfiguration implements IObaseConnectionPoolConfiguration {

    @Override
    public String name() {
        return "SpringBoot Connection Pool";
    }

    @Override
    public int getMaximumPoolSize() {
        return 50;
    }

    /**
     * 最小空闲连接数 保持10个空闲连接
     */
    @Override
    public int getMinimumIdle() {
        return 10;
    }

    /**
     * 获取连接的超时时间 30秒
     */
    @Override
    public long getConnectionTimeout() {
        return 30000L;
    }

    /**
     * 空闲连接的超时时间 10分钟
     */
    @Override
    public long getIdleTimeout() {
        return 600000L;
    }

    /**
     * 连接的最大存活时间 30分钟
     */
    @Override
    public long getMaxLifetime() {
        return 1800000L;
    }

    /**
     * 连接泄漏检测 借出超过60秒未归还时输出警告日志
     */
    @Override
    public long getLeakDetectionThreshold() {
        return 60000L;
    }
}
```

## 进行依赖注入

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

## 在SpringBoot里管理连接池配置

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
     * 最小空闲连接数
     */
    @Value("${poolMinimumIdle}")
    private int minimumIdle;

    /**
     * 获取连接的超时时间(毫秒)
     */
    @Value("${poolConnectionTimeout}")
    private long connectionTimeout;

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
     * 连接池的最大大小 如果小于等于0 则使用默认值10
     *
     * @return 连接池的最大大小
     */
    @Override
    public int getMaximumPoolSize() {
        return size;
    }

    /**
     * 连接池的最小空闲连接数 如果小于等于0 则使用默认值
     *
     * @return 连接池的最小空闲连接数
     */
    @Override
    public int getMinimumIdle() {
        return minimumIdle;
    }

    /**
     * 获取连接的超时时间(毫秒) 如果小于等于0 则使用默认值30000
     *
     * @return 获取连接的超时时间(毫秒)
     */
    @Override
    public long getConnectionTimeout() {
        return connectionTimeout;
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

## 与预热器一同使用

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

此处省略的向SpringBoot依赖注入上下文工厂的的标注代码和类定义是在[Obase如何定义和管理对象上下文(java版)](../Obase入门/Obase如何定义和管理对象上下文_java.md)中介绍的上下文工厂注入SpringBoot的代码.

## 注意事项

- 连接池按连接字符串区分,同一个连接字符串只会建造一个连接池,所以同一个连接字符串下的所有上下文类型共用第一个建造连接池时读到的配置.
- 配置在连接池建造时读取一次,建造完成后再修改配置不会影响已有的连接池.
- 时间类配置项的单位是毫秒,且受HikariCP的下限约束;配置了小于下限的值时以HikariCP调整后的值为准,应查看日志确认实际生效的值.
- `getConnectionTestQuery()`通常不需要设置,只有在驱动不支持JDBC4的`isValid`时才需要指定,例如Oracle的`SELECT 1 FROM DUAL`.
