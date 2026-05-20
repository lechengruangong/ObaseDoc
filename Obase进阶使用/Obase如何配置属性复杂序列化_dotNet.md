在文章[Obase如何配置属性的序列化器](./Obase如何配置属性简单序列化_dotNet.md)中,我们介绍了如何配置属性的序列化器,解决了在持久化/反持久化过程中有些属性需要进行序列化的问题.

但仅对属性配置序列化器时,如果遇到较为复杂的情况(如某些对象属性要特殊存储,对象间存在互相引用等)时,就需要定义一个继承自TextSerializer的特定序列化器,并且在实现的方法中针对不同的类型书写具体的序列化方案.

针对这种情况,Obase在6.5.0中新增了一个新的序列化对象模型(Serialization Object Data Model,之后简称为SOdm)作为之前一直使用的对象数据模型(Object Data Model,之后简称为Odm)的新组成部分,SOdm用于指导Obase在持久化/反持久化如何对持久化/反持久化对象的属性进行序列化.

SOdm使用特定的数据传输对象(Data Transfer Object,以后简称为DTO)来存储要序列化的对象,同时为要序列化的对象定义属性,构造函数,引用三种配置来指导如何序列化和反序列化.为了解决循环引用的问题,SOdm会在序列化过程中为每个对象分配唯一的临时ID,并存储在DTO中的引用字段里,同时在反序列化时用具体对象替换.

下面来看一下如何进行配置.

## dotNet

假设我们有一个Service服务类,上面引用了Identity身份和Component组件类,这两个都需要序列化为Json字符串进行存储.

以下是Service的定义:
```
/// <summary>
///     模拟某种服务
/// </summary>
public class Service
{
    /// <summary>
    ///     主键
    /// </summary>
    public string Code
    {
        get => _code;
        set => _code = value;
    }
    
    /// <summary>
    ///     身份
    /// </summary>
    public Identity Identity
    {
        get => _identity;
        set => _identity = value;
    }
    
    /// <summary>
    ///     组件
    /// </summary>
    public List<Component> Components { get; set; }
}

```
以下是Identity的定义:
```
/// <summary>
///     某种身份
/// </summary>
public class Identity
{
    /// <summary>
    ///     创建时间
    /// </summary>
    private DateTime _createTime;

    /// <summary>
    ///     身份标识
    /// </summary>
    private Guid _id;

    /// <summary>
    ///     查询时间
    /// </summary>
    private DateTime _queryTime;

    /// <summary>
    ///     角色
    /// </summary>
    private string _role;

    /// <summary>
    ///     初始化某种身份
    /// </summary>
    /// <param name="id">身份标识</param>
    /// <param name="createTime">创建时间</param>
    /// <param name="role">角色</param>
    public Identity(Guid id, DateTime createTime, string role)
    {
        _id = id;
        _createTime = createTime;
        _role = role;
        _queryTime = DateTime.Now;
    }

    /// <summary>
    ///     反序列化函数
    /// </summary>
    /// <param name="id">身份标识</param>
    /// <param name="createTime">创建时间</param>
    /// <param name="role">角色</param>
    /// <param name="queryTime">查询时间</param>
    protected internal Identity(Guid id, DateTime createTime, string role, DateTime queryTime)
    {
        _id = id;
        _createTime = createTime;
        _role = role;
        _queryTime = queryTime;
    }

    /// <summary>
    ///     身份标识
    /// </summary>
    public Guid Id
    {
        get => _id;
        protected internal set => _id = value;
    }

    /// <summary>
    ///     创建时间
    /// </summary>
    public DateTime CreateTime
    {
        get => _createTime;
        protected internal set => _createTime = value;
    }

    /// <summary>
    ///     角色
    /// </summary>
    public string Role
    {
        get => _role;
        protected internal set => _role = value;
    }

    /// <summary>
    ///     查询时间
    /// </summary>
    public DateTime QueryTime
    {
        get => _queryTime;
        protected internal set => _queryTime = value;
    }

    /// <summary>
    ///     版本
    /// </summary>
    public int Version { get; set; }

    /// <summary>
    ///     返回字符串
    /// </summary>
    /// <returns></returns>
    public override string ToString()
    {
        return
            $"{nameof(_createTime)}: {_createTime}, {nameof(_id)}: {_id}, {nameof(_queryTime)}: {_queryTime}, {nameof(_role)}: {_role}";
    }
}


```
以下是Component的定义
```
/// <summary>
///     组件类
/// </summary>
public class Component
{
    /// <summary>
    ///     引用的组件集合
    /// </summary>
    public List<IComponent> Components { get; set; }

    /// <summary>
    ///     组件名称
    /// </summary>
    public string Name { get; set; }

    /// <summary>
    ///     返回字符串
    /// </summary>
    /// <returns></returns>
    public override string ToString()
    {
        return $"{nameof(Name)}: {Name}";
    }
}
```

那么我们需要做如下的配置:
```
//配置序列化模型Identity
var idEntityConfiguration = modelBuilder.SerializationEntity<Identity>();
//Identity的属性 无需配置 自动侦测
//为了测试配置 故写一个属性配置
idEntityConfiguration.Attribute(p => p.Role);
//Identity的构造函数需要配置
var idConstructor = idEntityConfiguration.HasConstructor(typeof(Identity).GetConstructor(
    BindingFlags.Instance | BindingFlags.NonPublic, null,
    [typeof(Guid), typeof(DateTime), typeof(string), typeof(DateTime)], null));
//配置第一个参数 需要存储 从ID里取出Id属性的值存储
idConstructor.HasParameter(p => p.Id, typeof(Guid), true)
    //配置第二个参数 需要存储 从字段_createTime里取出CreateTime属性的值存储
    .HasParameter(typeof(Identity).GetField("_createTime", BindingFlags.Instance | BindingFlags.NonPublic),
        typeof(DateTime), true)
    //配置第三个参数 需要存储 从Role里取出Role属性的值存储
    .HasParameter(p => p.Role, typeof(string), true)
    //配置第四个参数 不需要存储 直接传入当前时间 注意这个委托的参数会传空
    .HasParameter(_ => DateTime.Now, typeof(DateTime), false);
//Identity没有引用 无需配置
//忽略版本
idEntityConfiguration.Ignore(p => p.SubVersion);

//配置序列化模型Component
modelBuilder.SerializationEntity<Component>();
//Component的属性 无需配置 自动侦测
//Component有无参的反序列化构造函数 无需配置 自动侦测
//Component有引用 无需配置 自动侦测

```
这里的配置需要注意的是,一个序列化对象的配置分为三种配置:属性,引用和构造函数.

其中属性是一定需要存储于DTO中的,引用则是一定不需要存储于DTO中的(只会存储序列化过程中分配的唯一临时ID),而构造函数的参数则是可由配置者自行配置是否需要存储于DTO的.

对于需要存储于DTO的属性或者构造函数参数,则只支持将Obase基元类型进行存储;不需要存储的构造函数参数,则是构造对象时用来注入某些外部对象用的.

和Odm相似,对于大部分的属性和引用Obase都可以自动侦测并进行配置,实际上需要配置的只有反序列化的构造函数其他的大多可以交给Obase进行自动配置,就像Component的配置一样.

如果有某些属性不需要被序列化,可以使用Ignore方法进行忽略.

如果在序列化过程中,碰到没有配置的对象会跳过此对象,在反序列化时自然也就没有相应的对象被构造了.

最后,需要为Service进行配置,并在相应的属性配置上启用复杂的序列化,代码如下:
```
//注册服务为实体型
var serviceEntity = modelBuilder.Entity<Service>();
serviceEntity.HasKeyAttribute(p => p.Code).HasKeyIsSelfIncreased(false);
serviceEntity.ToTable("Service");

//设置Identity属性 长度设置的长一些 保证存储的下
serviceEntity.Attribute(p => p.Identity, typeof(string)).HasMaxcharNumber(1000)
     //需要有模型的序列化和设置序列化器
     .UseSerializer(new JsonSerializer(), typeof(Identity)).UseSerializationModel(true);
     
//设置Components属性 长度设置的长一些 保证存储的下
serviceEntity.Attribute(p => p.Components, typeof(string)).HasMaxcharNumber(1000)
     //需要有模型的序列化和设置序列化器
     .UseSerializer(new JsonSerializer(), typeof(List<Component>)).UseSerializationModel(true);

```
这里的配置和之前文章[Obase如何配置属性的序列化器](./Obase如何配置属性简单序列化_dotNet.md)中配置类似,只是新增了UseSerializationModel这个配置. JsonSerializer的定义也是在文章[Obase如何配置属性的序列化器](./Obase如何配置属性简单序列化_dotNet.md)中给出.

当然,如果没有为某个属性启用UseSerializationModel(true),依然是使用之前文章[Obase如何配置属性的序列化器](./Obase如何配置属性简单序列化_dotNet.md)中的逻辑,直接将原始对象传入由用户定义的序列化器中,启用则是将特定的DTO对象传入.

附加,保存和查询部分和之前的均相同,此处不再赘述.

## java