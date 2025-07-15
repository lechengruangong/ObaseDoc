当多个进程或线程并发的对同一个数据源进行操作时,就有可能出现并发冲突.

Obase将并发冲突归类为以下三中:
1. 重复创建,即尝试创建主键相同的对象.
2. 版本冲突,在配置了版本键的情况下,修改对象时版本键已被其他线程/进程修改.
3. 更新幻影,修改对象时对象已被其他线程/进程删除.

这三种并发冲突Obase使用5种策略来进行处理,这些策略需要配置在结构化类型(实体型或关联型)上.

这5种策略分别是:
1. 忽略策略,当发生并发时不做任何处理,本策略可以处理所有类型的并发冲突.
2. 抛出异常策略,当发生并发异常,会抛出特定的异常告知调用方处理,本策略可以处理所有类型的并发冲突.
3. 强制覆盖策略,当发生并发时,用当前对象覆盖原有对象,本策略可以处理重复创建和版本冲突两种并发情况.
4. 重建对象策略,当发生异常时,将当前对象做为新对象进行创建,本策略可以处理更新幻影这种并发情况.
5. 版本合并策略,当发生异常时,将当前对象与旧对象的属性进行合并,本策略可以处理重复创建和版本冲突这两种并发情况.

三种并发冲突中,重复创建和更新幻影Obase会根据数据源返回的信息进行判断,版本冲突则一般表现为数据错误,不能由数据源返回的信息进行判断,考虑一个如下的典型的版本冲突场景:
> SQL数据源中有两张表,其中A表有一个属性sum记录B表中的行数总和,此时有两个线程T1和T2分别查询了B表的行数,并且要插入一条新的B表记录.
> T1和T2获取到的行数都是b,并且都将b+1的值更新至A表的sum字段,但此时B表其实插入了两条数据,此时A表的sum应该是b+2.

所以此时需要一个额外的配置来标记A表的版本,此配置即为版本键配置.

版本键可以配置多个,可以使用会发生并发冲突的属性或者使用时间戳标识最后的修改时间来作为版本键,能区分对象最后被谁修改的属性都可以作为版本键.

对于上面的例子就可以建一个LastModifyTime作为版本键或者直接将Sum作为版本键.

## dotNet

首先,配置版本键,如果确定一个结构化类型(实体型或关联型)不会发生版本冲突,也可以不配置版本键.
```
//省略掉了定义实体型的代码 此处的entityConfig即为一个实体型配置
entityConfig.HasVersionAttribute(p => p.VersionKey);
```
使用如上方法即可配置版本键,每次调用HasVersionAttribute就会增加一个版本键.

然后配置此结构化类型的并发冲突处理策略.
```
//省略掉了定义实体型的代码 此处的entityConfig即为一个实体型配置
entityConfig.HasConcurrentConflictHandlingStrategy(EConcurrentConflictHandlingStrategy);
```
EConcurrentConflictHandlingStrategy枚举包含以下值:
```
/// <summary>
///     忽略。
/// </summary>
Ignore = 0,

/// <summary>
///     引发异常。
/// </summary>
ThrowException = 1,

/// <summary>
///     强制覆盖。
/// </summary>
Overwrite = 2,

/// <summary>
///     版本合并。
/// </summary>
Combine = 3,

/// <summary>
///     重建对象。
/// </summary>
Reconstruct = 4
```
这些策略中,ThrowException是默认的策略,如果需要设置为此种策略,可以不进行配置.指定为Combine策略时,要额外指定当前结构化类型的属性要如何合并,这些属性合并策略配置方法定义在属性上.

```
//省略掉定义属性的代码 此处的attr即为一个当前结构化类型的属性
attr.HasCombinationHandler(EAttributeCombinationHandlingStrategy);
```
这里的EAttributeCombinationHandlingStrategy有以下值:
```
/// <summary>
///     覆盖——强制覆盖对方版本的值。
/// </summary>
Overwrite = 0,

/// <summary>
///     忽略——忽略当前属性，即承认冲突对方版本的值。
/// </summary>
Ignore = 1,

/// <summary>
///     累加——将当前版本中属性值的增量累加到对方版本。
/// </summary>
Accumulate = 2
```
由于在进行版本合并的处理时,大多数的属性都不需要处理,所以默认的属性合并策略是Ignore,在这种属性合并策略下会以数据源内的数据为准.

如果需要用新值直接覆盖旧值,就使用Overwrite属性合并策略.

大多数情况下,是需要进行合并,要使用Accumulate属性合并策略,不过需要注意的是Accumulate策略只能用在数值类型的属性上.

综上,如果要将一个实体类型的合并策略设置为版本合并,且将Value属性作为版本键和使用累加属性合并策略的例子如下:
```
 //配置实体型
 var accumulateCombineConfig = modelBuilder.Entity<AccumulateCombineKeyValue>();
 //配置键属性
 accumulateCombineConfig.HasKeyAttribute(p => p.Id).HasKeyIsSelfIncreased(false);
 //配置并发处理策略为 版本合并
 //版本合并策略可以处理重复创建和版本冲突 这两种并发情况
 //版本合并策略 当发生异常时 将当前对象与旧对象的属性进行合并
 accumulateCombineConfig.HasConcurrentConflictHandlingStrategy(EConcurrentConflictHandlingStrategy.Combine);
 //配置版本键 用于检测修改时的并发冲突 要想处理版本冲突并发 必须配置版本键
 //版本键可以配置多个
 //可以使用会发生并发冲突的属性 或者使用 时间戳标识最后的修改时间 来作为版本键
 //能区分对象最后被谁修改的属性都可以作为版本键
 accumulateCombineConfig.HasVersionAttribute(p => p.VersionKey);
 //配置映射表
 accumulateCombineConfig.ToTable("KeyValues");
 //配置要进行合并的属性的合并策略
 accumulateCombineConfig.Attribute(p => p.Value)
     //设置为累加 即将当前版本中属性值的增量累加到对方版本 只支持数值型的属性
     .HasCombinationHandler(EAttributeCombinationHandlingStrategy.Accumulate);
```
## Java

java版待重写

