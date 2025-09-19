
## dotNet

在dotNet版本中,可以使用扩展方法和结构化类型扩展来配置逻辑删除.

首先,需要在引用Obase本体及数据源提供器之外,引用逻辑删除插件.

通常情况下,由于引用传递只需要引用对应的数据源提供器和逻辑删除插件即可.

配置文件类似如下形式:

```
<PackageReference Include="Obase.LogicDeletion" Version="6.1.0" />
<PackageReference Include="Obase.Providers.MySql" Version="6.1.0" />
```

在引用了逻辑删除插件之后,模型配置中增加了一种新的扩展:LogicDeletionExtensionConfiguration<TEntity>,使用如下代码即可配置在结构化类型的扩展中:


```
//此处省略了一些配置结构化类型的代码 鉴定已经将类Domain配置为实体型
//创建逻辑删除扩展
var logicDeletionExt = logicDeletion.HasExtension<LogicDeletionExtensionConfiguration<Domain>>();
//当类中有定义逻辑删除字段时 指定为逻辑删除标记
logicDeletionExt.HasDeletionMark(p => p.Deleted);
//当类中没有定义逻辑删除字段时 需要指定逻辑删除映射字段
logicDeletionExt.HasDeletionField("Deleted");
```

此处可以分为两种配置,一种是类内有定义逻辑删除的字段,一种是没有定义逻辑删除的字段.

当然,对于Sql数据源,这两种情况都需要在映射表内存在逻辑删除的字段.

第5行就是类内有定义逻辑删除字段的情况,此种情况需要指定哪个属性是逻辑删除字段.

第7行时类内没有定义逻辑删除字段的情况,此时仅需要配置数据源中哪个字段是逻辑删除即可.

这两行配置根据需要选择其中一个即可.

在配置了逻辑删除之后,如果要使用逻辑删除还需要再查询或保存对象前,对上下文启用逻辑删除,代码如下:


```
//省略一些构造上下文的代码 此处假定上下文已经构造出来了
context.EnableLogicDeletion();
```

启用逻辑删除后,此上下文对象会在查询时默认过滤掉逻辑删除字段为true的记录并且在保存时为逻辑删除字段赋值.

如果你希望这个上下文一直启用逻辑删除,可以在构造函数里调用启用方法.

此外,上下文还增加了以下几个方法用于逻辑删除和取消逻辑删除.


```
//逻辑标记移除对象
context.CreateSet<Domain>().RemoveLogically(entity);
//逻辑标记恢复对象
context.CreateSet<Domain>().RecoveryLogically(entity);
//直接逻辑删除符合条件的对象
context.CreateSet<Domain>().DeleteLogically(p=>p.Id > 50);
//直接逻辑恢复符合条件的对象
context.CreateSet<Domain>().RecoveryLogically(p=>p.Id > 50);
```

需要注意的是,逻辑删除只生效于查询基点,即在A类上配置类逻辑删除,使用CreateSet^A时会将逻辑删除的A对象过滤掉.但在B中引用了A,查询B时使用Include方法一并查询A时,B引用的A不会处理逻辑删除.

## java

java版待重写