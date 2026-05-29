有些时候我们的系统中可能存在多个不同的用户,他们购买我们的服务支撑自己的业务,这些业务对象需要根据用户进行区分.

为了应对这个需求,Obase提供了多租户的功能.

## 模型内配置多租户

首先,需要在引用Obase本体及数据源提供器之外,引用多租户插件.通常情况下,由于引用传递只需要引用对应的数据源提供器和多租户插件即可.

引入的配置类似如下形式,版本改为你需要的版本即可:

```
<dependency>
    <groupId>io.obase</groupId>
    <artifactId>providers.mysql</artifactId>
    <version>x.x.x</version>
</dependency>
<dependency>
    <groupId>io.obase</groupId>
    <artifactId>multi.tenant</artifactId>
    <version>x.x.x</version>
</dependency>
```
在引用了多租户件之后,模型配置中增加了一种新的扩展配置MultiTenantExtensionConfiguration<TEntity>,使用hasExtension方法即可配置多租户,此方法可以由实体型配置对象或者关联型配置对象调用.

hasExtension方法会返回MultiTenantExtensionConfiguration<TEntity>的实例,MultiTenantExtensionConfiguration有hasTenantIdMark和hasTenantIdField方法用于配置多租户的标记属性和数据源字段,以下代码中的multiTenant就是要在配置多的实体型或者关联型配置对象:

```
//此处的Domain就是要配置多租户的类型
//创建多租户扩展
var multiTenantExt = multiTenant.hasExtension<MultiTenantExtensionConfiguration<Domain>>();
//当类中有定义多租户属性时 指定某个属性为多租户标记
multiTenantExt.hasTenantIdMark(p => p.getTenantId());
//当类中没有定义多租户属性时或者数据源的字段与属性名称不一致时 需要指定多租户映射字段
multiTenantExt.hasTenantIdField("TenantId");
//设置全局的多租户ID 如果系统中需要一些属于全局的 可以被每个租户都查询的 需要配置此项
multiTenantExt.hasGlobalTenantId("00000000");
//在配置了全局多租户ID后 如果需要每个租户都查询到全局的 需要配置此项为true
multiTenantExt.hasLoadingGlobal(true);
```
此处可以分为两种配置,一种是类内有定义多租户的属性,一种是没有定义多租户的属性.

当然,对于Sql数据源,这两种情况都需要在映射表内存在多租户的字段.

第5行就是类内有定义多租户属性的情况,此种情况需要指定哪个属性是多租户属性,如果数据源的字段与属性名称相同,则只需要配置hasTenantIdMark

第7行是类内没有定义多租户字段或者数据源的字段与属性名称不一致的情况,此时需要配置数据源中哪个字段是多租户的字段.

此外还有全局租户相关的配置,用于系统中存在需要被所有租户获取的对象的情况.

## 依赖注入租户ID读取器

Obase使用依赖注入来实现对获取租户ID逻辑的解耦,所以在配置了多租户后,还要向Obase注入ITenantIdReader租户Id读取器的具体实现.

```
//为Obase注入租户ID读取器
ServiceContainerBuilder oBuilder = ObaseDependencyInjection.createBuilder(DataContext.class);
oBuilder.addSingleton(ITenantIdReader.class, TenantIdReader.class);
oBuilder.build();
```
此处的DataContext就是要注入的上下文类型,TenantIdReader则是ITenantIdReader的具体实现类,并且需要保证这段依赖注入代码仅运行一次

## 启用多租户

考虑到不一定是所有场景中都需要多租户,需要对上下文启用多租户才会根据多租户字段进行筛选,调用方法如下:

```
//省略一些构造上下文的代码 此处假定上下文已经构造出来了
MultiTenantExtensions.enableMultiTenant(context);
```

启用多租户后,此上下文对象会在查询时会根据多租户字段过滤记录并且在保存时为多租户字段赋值.

如果你希望这个上下文一直启用多租户,可以在构造函数里调用启用方法.

接下来我们以一个具体的场景为例来介绍这个功能.

## 具体示例

考虑一个如下的场景,某个系统内存在多个推广员,他们会引导新用户访问一个聚合页并留下记录,这些推广员只需要查询自己推广的用户即可,此时可以将推广员视作系统的租户,他们的各种对象应当都是隔离的,此时就可以使用多租户.

示例使用SpringBoot 2.7.18进行,数据源为MySql,那么引用如下的包(具体版本号换成你需要的):

```
<dependency>
    <groupId>io.obase</groupId>
    <artifactId>providers.mysql</artifactId>
    <version>x.x.x</version>
</dependency>
<dependency>
    <groupId>io.obase</groupId>
    <artifactId>multi.tenant</artifactId>
    <version>x.x.x</version>
</dependency>
```
首先,定义实体类,这里定义了一个聚合页访问记录类,表示用户访问聚合页时留下的记录.

```
/**
 * 路由页面访问记录
 */
public class RoutePageVisitRecord {

    /**
     * 访问记录ID
     */
    private int recordId;

    /**
     * 访问者ID
     */
    private int visitorId;

    /**
     * 访问时间
     */
    private LocalDateTime visitTime;

    /**
     * 获取访问记录ID
     * @return 访问记录ID
     */
    public int getRecordId() {
        return recordId;
    }

    /**
     * 设置访问记录ID
     * @param recordId 访问记录ID
     */
    public void setRecordId(int recordId) {
        this.recordId = recordId;
    }

    /**
     * 获取访问者ID
     * @return 访问者ID
     */
    public int getVisitorId() {
        return visitorId;
    }

    /**
     * 设置访问者ID
     * @param visitorId 访问者ID
     */
    public void setVisitorId(int visitorId) {
        this.visitorId = visitorId;
    }

    /**
     * 获取访问时间
     * @return 访问时间
     */
    public LocalDateTime getVisitTime() {
        return visitTime;
    }

    /**
     * 设置访问时间
     * @param visitTime 访问时间
     */
    public void setVisitTime(LocalDateTime visitTime) {
        this.visitTime = visitTime;
    }
}
```
当然实际使用时还会有一些其他的属性,这里仅作示例,接下来进行配置:
```
//配置一个实体型
EntityTypeConfiguration<RoutePageVisitRecord> routeRecord = modelBuilder.entity(RoutePageVisitRecord.class);
routeRecord.hasKeyAttribute(p -> p.getRecordId()).hasKeyIsSelfIncreased(true);
record.toTable("RoutePageVisitRecord");
//配置多租户
MultiTenantExtensionConfiguration<RoutePageVisitRecord> multiTenantExtension = routeRecord.hasExtension(MultiTenantExtensionConfiguration.class);
//没在类内定义多租户字段 这里直接配置逻辑删除字段名称即可
multiTenantExtension.hasTenantIdField("TenantId", String.class);
```
这里的逻辑不涉及全局的租户也没有在类内定义租户ID,所以只需要配置自己的多租户字段名.

再定义一个更改消息发送器TenantIdReader:

```
/**
 * 租户ID读取器
 */
public class TenantIdReader implements ITenantIdReader {
    /**
     * 获取租户ID
     *
     * @return 租户ID
     */
    @Override
    public Object getTenantId() {
        return "USER123";
    }
}
```
实际使用时,TenantIdReader的GetTenantId内改为自己的具体逻辑即可.

接下来进行依赖注入,对于SpringBoot项目,可以在启动文件里进行注入.代码如下:
```
//为Obase注入租户ID读取器
ServiceContainerBuilder oBuilder = ObaseDependencyInjection.createBuilder(DataContext.class);
oBuilder.addSingleton(ITenantIdReader.class, TenantIdReader.class);
oBuilder.build();
```
这里的第2行到第4行就是将ITenantIdReader注入Obase中的代码,注意创建依赖注入建造器时需要指定上下文的类型,且多租户需要的注入类型是ITenantIdReader,不能只将实现类的类型注入.

如果ITenantIdReader的具体实现需要用到由SpringBoot管理的某些服务,比如需要用RequestContextHolder获取某个固定的参数或者头信息,那么先定义一个Bean来获取Http请求:
```
/**
 * Http请求参数提取器
 */
@Component
public class HttpParamExtractor {

    /**
     * 提取请求参数的值
     * @param name 参数名称
     * @return 参数值
     */
    public String getParameter(String name) {
        HttpServletRequest request = getCurrentRequest();
        return request != null ? request.getParameter(name) : null;
    }

    /**
     * 提取请求头的值
     * @param name 头的键
     * @return 对应的值
     */
    public String getHeader(String name) {
        HttpServletRequest request = getCurrentRequest();
        return request != null ? request.getHeader(name) : null;
    }

    /**
     * 调用RequestContextHolder获取请求
     * @return 请求
     */
    private HttpServletRequest getCurrentRequest() {
        ServletRequestAttributes attrs =
                (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        return attrs != null ? attrs.getRequest() : null;
    }
}
```
这里定义了一个Http请求参数提取器并使用注解添加为SpringBoot的Bean,然后再修改一下TenantIdReader,在构造函数中注入一个我们定义的HttpParamExtractor:
```
/**
 * 租户ID读取器
 */
public class TenantIdReader implements ITenantIdReader {

    /**
     * Http请求提取器
     */
    private HttpParamExtractor extractor;

    /**
     * 初始化租户ID读取器
     * @param extractor Http请求提取器
     */
    public TenantIdReader(HttpParamExtractor extractor) {
        this.extractor = extractor;
    }

    /**
     * 获取租户ID
     *
     * @return 租户ID
     */
    @Override
    public Object getTenantId() {
        //没有注入提取器 返回null
        if(this.extractor == null)
            return null;
        //提取参数TenantId
        return this.extractor.getParameter("TenantId");
    }
}
```
启动文件也要做如下的修改:
```
//获取配置上下文 用于之后获取SpringBoot的Bean
ConfigurableApplicationContext context = SpringApplication.run(NoticeDemoApplication.class, args);
//为Obase注入租户ID读取器
ServiceContainerBuilder oBuilder = ObaseDependencyInjection.createBuilder(DataContext.class);
oBuilder.addSingleton(ITenantIdReader.class, TenantIdReader.class, p -> new TenantIdReader(context.getBean(HttpParamExtractor.class)));
oBuilder.build();
```
此处主要的修改为Obase依赖注入方法改为使用委托作为参数的注入方法,并在获取到ConfigurableApplicationContext后从SpringBoot的IOC容器中获取具体服务作为构造TenantIdReader的参数.

最后只要和平常一样调用保存新对象和查询的逻辑即可,唯一需要注意的是为上下文启用对象通知:
```

//此处省略从SpringBoot的依赖注入中获取context的代码
//如果有需要 可以在对象上下文的构造函数里调用enableMultiTenant方法 这样所有构造出来的上下文就都是启用了对象通知的

//启用多租户
MultiTenantExtensions.enableMultiTenant(context);
//新建对象
RoutePageVisitRecord record = new RoutePageVisitRecord();
record.setVisitorId(1);
record.setVisitTime(LocalDateTime.now());
//附加并保存对象
context.attach(record);
context.saveChanges();
```