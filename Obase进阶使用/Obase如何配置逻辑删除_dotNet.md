有些场景下,我们对于某些对象不能真实的从数据源内删除,而是需要使用某个字段标识状态,在场景中某些状态的对象不参与,这种模式一般称之为软删除.

为应对此需求,Obase框架提供了逻辑删除功能.

## 配置方法

首先,需要在引用Obase本体及数据源提供器之外,引用逻辑删除插件.通常情况下,由于引用传递只需要引用对应的数据源提供器和逻辑删除插件即可.

引入的配置类似如下形式,版本改为你需要的版本即可:

```
<PackageReference Include="Obase.LogicDeletion" Version="x.x.x" />
<PackageReference Include="Obase.Providers.MySql" Version="x.x.x" />
```

在引用了逻辑删除插件之后,模型配置中增加了一种新的扩展配置LogicDeletionExtensionConfiguration<TEntity>,使用HasExtension方法即可配置逻辑删除,此方法可以由实体型配置对象或者关联型配置对象调用.

HasExtension方法会返回LogicDeletionExtensionConfiguration<TEntity>的实例,LogicDeletionExtensionConfiguration有HasDeletionMark和HasDeletionField方法用于配置逻辑删除的标记属性和数据源字段,以下代码中的logicDeletion就是要在配置逻辑删除的实体型或者关联型配置对象:

```
//此处的Domain就是要配置罗删除的类型
//创建逻辑删除扩展
var logicDeletionExt = logicDeletion.HasExtension<LogicDeletionExtensionConfiguration<Domain>>();
//当类中有定义逻辑删除属性时 指定某个属性为逻辑删除标记
logicDeletionExt.HasDeletionMark(p => p.Deleted);
//当类中没有定义逻辑删除字段时或者数据源的字段与属性名称不一致时 需要指定逻辑删除映射字段
logicDeletionExt.HasDeletionField("Deleted");
```

此处可以分为两种配置,一种是类内有定义逻辑删除的属性,一种是没有定义逻辑删除的属性.

当然,对于Sql数据源,这两种情况都需要在映射表内存在逻辑删除的字段.

第5行就是类内有定义逻辑删除属性的情况,此种情况需要指定哪个属性是逻辑删除属性,如果数据源的字段与属性名称相同,则只需要配置HasDeletionMark.

第7行是类内没有定义逻辑删除字段或者数据源的字段与属性名称不一致的情况,此时需要配置数据源中哪个字段是逻辑删除的字段.

这两行配置根据需要选择.

## 启用逻辑删除

考虑到不一定是所有场景中都需要逻辑删除,需要对上下文启用逻辑删除才会根据逻辑删除字段进行筛选,调用方法如下:

```
//省略一些构造上下文的代码 此处假定上下文已经构造出来了
context.EnableLogicDeletion();
```

启用逻辑删除后,此上下文对象会在查询时默认过滤掉逻辑删除字段为true的记录并且在保存时为逻辑删除字段赋值.

如果你希望这个上下文一直启用逻辑删除,可以在构造函数里调用启用方法.

## 逻辑删除相关方法

此外,上下文还增加了以下几个方法用于逻辑删除和取消逻辑删除,这些方法主要用于没有在类内定义逻辑删除属性的情况,类内定义了属性的情况直接操作相应的属性即可.

```
//这里的Domain就是配置为逻辑删除的类 entity是配置为逻辑删除类的对象
//逻辑标记移除对象
context.CreateSet<Domain>().RemoveLogically(entity);
//逻辑标记恢复对象
context.CreateSet<Domain>().RecoveryLogically(entity);
//直接逻辑删除符合条件的对象
context.CreateSet<Domain>().DeleteLogically(p => p.Id > 50);
//直接逻辑恢复符合条件的对象
context.CreateSet<Domain>().RecoveryLogically(p => p.Id > 50);
```

需要注意的是,逻辑删除只生效于查询基点,即在A类上配置类逻辑删除,使用CreateSet^A时会将逻辑删除的A对象过滤掉.但在B中引用了A,查询B时使用Include方法一并查询A时,B引用的A不会处理逻辑删除.

## 具体示例

考虑一个如下的场景,某个用户的主页被其他用户访问时,会留下访问记录.用户可以删除自己访问其他用户主页的记录,但后台分析用户访问记录时仍然需要这些数据,所以不能真实的在数据库中删除这些记录,此时就可以使用逻辑删除功能.

示例使用.net9版本的ASP.NET进行,数据源为MySql,那么引用如下的包(具体版本号换成你需要的):

```
<PackageReference Include="Obase.LogicDeletion" Version="x.x.x" />
<PackageReference Include="Obase.Providers.MySql" Version="x.x.x" />
```

首先,定义实体类,这里定义了一个访问记录类,表示用户访问主页时留下的记录.

```
/// <summary>
///     访问记录
/// </summary>
public class VisitRecord
{
    /// <summary>
    ///     主页拥有者ID
    /// </summary>
    public int OwnerId { get; set; }

    /// <summary>
    ///     访问者ID
    /// </summary>
    public int Visitor { get; set; }

    /// <summary>
    ///     访问时间
    /// </summary>
    public DateTime VisitTime { get; set; }
}
```
当然实际使用时还会有一些其他的属性,这里仅作示例,接下来进行配置:
```
//配置一个实体型
var visitRecord = modelBuilder.Entity<VisitRecord>();
visitRecord.HasKeyAttribute(p => p.OwnerId).HasKeyAttribute(p => p.VistorId).HasKeyIsSelfIncreased(false);
visitRecord.ToTable(nameof(VisitRecord));
//配置逻辑删除
var logicDeletionExtension = visitRecord.HasExtension<LogicDeletionExtensionConfiguration<VisitRecord>>();
//没在类内定义逻辑删除字段 这里直接配置逻辑删除字段名称即可
logicDeletionExtension.HasDeletionField("Deleted");
```

最后只要和平常一样查询即可,要进行逻辑删除时使用之前介绍的逻辑删除相关方法,唯一需要注意的是为上下文启用逻辑删除:

```
//此处省略从ASP.NET的依赖注入中获取context的代码
//如果有需要 可以在对象上下文的构造函数里调用EnableChangeNotice方法 这样所有构造出来的上下文就都是启用了对象通知的

//启用逻辑删除
context.EnableChangeNotice();
//查询对象
var record = _context.CreateSet<VisitRecord>().FirstOrDefault(p => p.OwnerId == 1 && p.Visitor == 2);
//逻辑删除此对象
context.CreateSet<VisitRecord>().RemoveLogically(record);
context.SaveChanges();
```
此后在启用了逻辑删除的上下文中查询VisitRecord就无法查询到OwnerId为1且VisitorId为2的对象了.