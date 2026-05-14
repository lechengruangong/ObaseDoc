有些时候我们需要在创建了新对象,修改某个对象或者删除了某个对象并保存之后向其他的系统或者中间件发送通知来实现内部协作.

为了应对这个需求,Obase提供了对象变更通知的功能.

## 模型内配置对象更改通知
对象更改通知是包含在Obase本体中的,因此不需要额外引用软件包,通常情况下,由于引用传递只需要引用对应的数据源提供器即可.

在模型配置中有以下几个方法用于配置对象变更通知,这些方法可以由实体型配置对象或者关联型配置对象调用,以下代码中所有的noticeEntityConfig就是要在变更后进行通知的实体型或者关联型配置对象:
```
//属性名称集合
List<String> attrs = new ArrayList<>();
attrs.add("Description");
attrs.add("Background");
//配置要进行通知的属性方法 参数为属性的名称 当发生特定的行为时 这些属性的值会包含在通知消息内
noticeEntityConfig.hasNoticeAttributes(attrs);
//无参的要进行通知的属性方法则表示通知所有的属性 注意此方法会覆盖有参的方法配置的属性
noticeEntityConfig.hasNoticeAttributes();
//指示是否在对象被创建时进行通知
noticeEntityConfig.hasNotifyCreation(true);
//指示是否在对象被删除时进行通知
noticeEntityConfig.hasNotifyDeletion(true);
//指示是否在对象被修改时进行通知
noticeEntityConfig.hasNotifyUpdate(true);
```

这些配置方法主要指定了以下几个配置内容,分别是是否在创建时通知,是否在删除时通知,是否在修改时通知和哪些字段要包含在通知中.

其中的创建时进行通知是在新对象在上下文中保存时发出通知,修改时进行通知是旧对象被修改后在上下文中保存或者使用就地修改方法(newAttribute和increaseAttribute)时发出通知,删除时进行通知时在旧对象被删除后再上下文中保存或者使用就地删除方法(delete)时发出通知.

## 依赖注入对象更改通知发送器

Obase使用依赖注入来实现对具体通知逻辑的解耦,所以在配置了要通知的时机和内容后,还要向Obase注入IChangeNoticeSender更改通知发送器的具体实现.

```
//为Obase注入消息发送器
ServiceContainerBuilder oBuilder = ObaseDependencyInjection.createBuilder(DataContext.class);
oBuilder.addSingleton(IChangeNoticeSender.class, ChangeNoticeSender.class);
oBuilder.build();
```

此处的DataContext就是要注入的上下文类型,ChangeNoticeSender则是IChangeNoticeSender的具体实现类,并且需要保证这段依赖注入代码仅运行一次

## 启用对象更改通知

考虑到不一定是所有场景中都需要发送变更通知,需要对上下文启用对象通知才会进行通知,调用方法如下:

```
//省略一些构造上下文的代码 此处假定上下文已经构造出来了
ChangeNoticeExtensions.enableChangeNotice(context);
```

如果需要某个上下文默认启用更改通知,可以在上下文的构造方法里调用此方法.

启用更改通知后在配置对应的对象新建,修改或者删除并保存时,就会根据配置调用IChangeNoticeSender的具体实现发送修改通知,IChangeNoticeSender中方法Send的参数ChangeNotice就是具体的通知内容,根据不同的变更类型分别有两个具体的实现类ObjectChangeNotice和DirectlyChangingNotice.

通知对象内部包含了对象变更行为,对象的属性及其取值,对象标识等信息,根据具体的需要取值处理即可.

接下来我们以一个具体的场景为例来介绍这个功能.

## 具体示例

考虑一个如下的场景,我们网站上某个页面有个广告,每当用户点击这个广告时,要记录下点击广告的用户ID,广告的ID和点击时间.

这些记录被保存到数据库时,还要同时向一个其他系统的中间件发送消息用于统计.

示例使用SpringBoot 2.7.18进行,数据源为MySql,那么引用如下的包(具体版本号换成你需要的):
```
<dependency>
    <groupId>io.obase</groupId>
    <artifactId>providers.mysql</artifactId>
    <version>x.x.x</version>
</dependency>
```
首先,定义实体类,这里定义了一个广告记录类,表示用户点击广告时的广告记录.

```
/**
 * 广告记录
 */
public class AdvRecord {

    /**
     * 用户ID
     */
    private String userId;

    /**
     * 广告ID
     */
    private String advId;

    /**
     * 点击时间
     */
    private LocalDateTime clickTime;

    /**
     * 获取用户ID
     * @return 用户ID
     */
    public String getUserId() {
        return userId;
    }

    /**
     * 设置用户ID
     * @param userId 用户ID
     */
    public void setUserId(String userId) {
        this.userId = userId;
    }

    /**
     * 获取广告ID
     * @return 广告ID
     */
    public String getAdvId() {
        return advId;
    }

    /**
     * 设置广告ID
     * @param advId 广告ID
     */
    public void setAdvId(String advId) {
        this.advId = advId;
    }

    /**
     * 获取点击时间
     * @return 点击时间
     */
    public LocalDateTime getClickTime() {
        return clickTime;
    }

    /**
     * 设置点击时间
     * @param clickTime 点击时间
     */
    public void setClickTime(LocalDateTime clickTime) {
        this.clickTime = clickTime;
    }
}
```
然后进行配置.

```
//配置一个实体型
EntityTypeConfiguration<AdvRecord> record = modelBuilder.entity(AdvRecord.class);
record.hasKeyAttribute(p -> p.getUserId()).hasKeyAttribute(p -> p.getAdvId()).hasKeyIsSelfIncreased(false);
record.toTable("AdvRecord");
//通知所有属性
record.hasNoticeAttributes();
//在创建时通知
record.hasNotifyCreation(true);
```

再定义一个更改消息发送器ChangeNoticeSender:

```
/**
 * 更改消息发送器
 */
public class ChangeNoticeSender implements IChangeNoticeSender {
    /**
     * 发送变更通知
     *
     * @param notice 变更通知
     */
    @Override
    public void send(ChangeNotice notice) {
        //此处收到的ChangeNotice是抽象类 有两个实现类 可以根据ChangeNotice的Type来进行区分
        //ObjectChange类型的对应是ObjectChangeNotice
        //DirectlyChanging类型对应的是DirectlyChangingNotice
        //这里可以从notice里获取属性和属性值等信息
        //实际的逻辑可能是将消息处理后发往Hbase/Redis之类的第三方服务
        //此处仅在Console里显示一下
        System.out.println(notice);
    }
}
```
实际使用时,ChangeNoticeSender的Send内改为自己的具体逻辑即可.

接下来进行依赖注入,对于SpringBoot项目,可以在启动文件里进行注入.代码如下:
```
//为Obase注入消息发送器
ServiceContainerBuilder oBuilder = ObaseDependencyInjection.createBuilder(DataContext.class);
oBuilder.addSingleton(IChangeNoticeSender.class, ChangeNoticeSender.class);
oBuilder.build();
```
这里的第2行到第4行就是将IChangeNoticeSender注入Obase中的代码,注意创建依赖注入建造器时需要指定上下文的类型,且更改通知需要的注入类型是IChangeNoticeSender,不能只将实现类的类型注入.

如果IChangeNoticeSender的具体实现需要用到由SpringBoot管理的某些服务,比如需要用Redis存储,那么需要修改一下ChangeNoticeSender,在构造函数中注入Redis服务:

```
/**
 * 更改消息发送器
 */
public class ChangeNoticeSender implements IChangeNoticeSender {

    /**
     * Redis服务
     */
    private final RedisService redisService;

    /**
     * 初始化更改消息发送器
     * @param redisService Redis服务
     */
    public ChangeNoticeSender(RedisService redisService) {
        this.redisService = redisService;
    }

    /**
     * 发送变更通知
     *
     * @param notice 变更通知
     */
    @Override
    public void send(ChangeNotice notice) {
    //此处收到的ChangeNotice是抽象类 有两个实现类 可以根据ChangeNotice的Type来进行区分
        //ObjectChange类型的对应是ObjectChangeNotice
        //DirectlyChanging类型对应的是DirectlyChangingNotice
        //这里可以从notice里获取属性和属性值等信息
        //此处的RedisService是在SrpingBoot中注册Bean
        if(this.redisService != null)
            this.redisService.Send(notice.toString());
    }
}
```
启动文件也要做如下的修改:
```
//获取配置上下文 用于之后获取SpringBoot的Bean
ConfigurableApplicationContext context = SpringApplication.run(NoticeDemoApplication.class, args);
//为Obase注入消息发送器
ServiceContainerBuilder oBuilder = ObaseDependencyInjection.createBuilder(DataContext.class);
oBuilder.addSingleton(IChangeNoticeSender.class, ChangeNoticeSender.class, p -> new ChangeNoticeSender(context.getBean(RedisService.class)));
oBuilder.build();
```
此处主要的修改为Obase依赖注入方法改为使用委托作为参数的注入方法,并在获取到ConfigurableApplicationContext后从SpringBoot的IOC容器中获取具体服务作为构造ChangeNoticeSender的参数.

最后只要和平常一样调用保存新对象的逻辑即可,唯一需要注意的是为上下文启用对象通知:
```
//此处省略从SpringBoot的依赖注入中获取context的代码
//如果有需要 可以在对象上下文的构造函数里调用enableChangeNotice方法 这样所有构造出来的上下文就都是启用了对象通知的

//启用对象通知
ChangeNoticeExtensions.enableChangeNotice(context);
//新建对象
AdvRecord advRecord = new AdvRecord();
advRecord.setUserId("1");
advRecord.setAdvId("2");
advRecord.setClickTime(LocalDateTime.now());
//附加并保存对象
context.attach(advRecord);
context.saveChanges();
```