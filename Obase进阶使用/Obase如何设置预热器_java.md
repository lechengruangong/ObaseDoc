Obase在默认的配置中,模型和连接池这些资源均是懒加载的,即在首次访问时进行加载,之后都使用首次加载时创建的对象.对于一般的网站,不是时刻都处于高并发的的状态下,大多是平时有一些零散的请求,在某些特殊时刻才会处于高并发的状态先.在这种情境下懒加载可以节省资源开销,并按需处理这些资源,但有些时候我们需要在应用启动时就将模型和连接池加载以保证访问速度,于是我们引入了一个新的机制:预热器.

预热器会尝试访问上下文的模型和对应的连接池,这样就可以将模型和连接池预热生成,不再占用首次访问的性能.

接下来以使用SpringBoot 2.7.18进行,数据源为MySql为示例来演示如何使用预热器.

首先,我们要定义一个继承自SqlContextPreHeater的类:
```
/**
 * Obase预热器 继承预热器抽象类
 */
public class ObasePreHeater extends SqlContextPreHeater {
}
```

只需要继承SqlContextPreHeater即可,之后在启动类里做如下配置:

```
//省略一些向SpringBoot依赖注入上下文工厂的的标注代码和类定义
ConfigurableApplicationContext applicationContext = SpringApplication.run(NoticeDemoApplication.class, args);
//获取SpringBoot管理的上下文工厂
DataContextFactory factory = applicationContext.getBean(DataContextFactory.class);
//生产上下文
DataContext dataContext = factory.generateContext();
//构造预热器
ObasePreHeater preHeater = new ObasePreHeater();
//预热
preHeater.preHeat(dataContext);
```
此处省略的向SpringBoot依赖注入上下文工厂的的标注代码和类定义是在[Obase的上下文管理(java版)](../Obase入门/Obase的上下文管理_java.md)中介绍的上下文工厂注入SpringBoot的代码.

预热器会默认使用slf4j日志输出预热结果,在日志中可以观察到类似于"XXX Has Initialized"的输出.
