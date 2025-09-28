Obase默认情况下不会处理类之间的继承关系,只是将他们作为不同的类型加以处理,

当需要父类与子类都存储于同一张表内且需要分页处理的场景时,就要进行特殊的配置.

## dotNet
设想如下的场景,Bike表示自行车,BikeWheel表示车轮,BikeLight表示车灯.

而后有一个特殊的MyBikeA继承Bike,有一个额外的旗子BikeFlag.一个特殊的MyBikeB继承Bike,有一个额外的车筐BikeBucket. 一个特殊的MyBIkeC继承MyBikeA,是可以共享的.

类定义如下:
```
/// <summary>
///     表示自行车
/// </summary>
public class Bike
{
    /// <summary>
    ///     自行车编码
    /// </summary>
    private string _code;

    /// <summary>
    ///     自行车灯
    /// </summary>
    private BikeLight _light;

    /// <summary>
    ///     车灯编码
    /// </summary>
    private string _lightCode;

    /// <summary>
    ///     自行车名称
    /// </summary>
    private string _name;

    /// <summary>
    ///     1-普通车 2-MyBikeA 3-MyBikeB
    /// </summary>
    private int _type;

    /// <summary>
    ///     自行车轮
    /// </summary>
    private List<BikeWheel> _wheels;

    /// <summary>
    ///     自行车编码
    /// </summary>
    public string Code
    {
        get => _code;
        set => _code = value;
    }

    /// <summary>
    ///     自行车名称
    /// </summary>
    public string Name
    {
        get => _name;
        set => _name = value;
    }

    /// <summary>
    ///     自行车灯
    /// </summary>
    public virtual BikeLight Light
    {
        get => _light;
        set => _light = value;
    }

    /// <summary>
    ///     自行车轮
    /// </summary>
    public virtual List<BikeWheel> Wheels
    {
        get => _wheels;
        set => _wheels = value;
    }

    /// <summary>
    ///     车灯编码
    /// </summary>
    public string LightCode
    {
        get => _lightCode;
        set => _lightCode = value;
    }

    /// <summary>
    ///     1-普通车 2-MyBikeA 3-MyBikeB
    /// </summary>
    public int Type
    {
        get => _type;
        protected internal set => _type = value;
    }
}

/// <summary>
///     车轮
/// </summary>
public class BikeWheel
{
    /// <summary>
    ///     车编码
    /// </summary>
    private string _bikeCode;

    /// <summary>
    ///     车轮编码
    /// </summary>
    private string _code;

    /// <summary>
    ///     车轮编码
    /// </summary>
    public string Code
    {
        get => _code;
        set => _code = value;
    }

    /// <summary>
    ///     车编码
    /// </summary>
    public string BikeCode
    {
        get => _bikeCode;
        set => _bikeCode = value;
    }
}

/// <summary>
///     车灯
/// </summary>
public class BikeLight
{
    /// <summary>
    ///     车灯编码
    /// </summary>
    private string _code;

    /// <summary>
    ///     亮度
    /// </summary>
    private int _value;

    /// <summary>
    ///     车灯编码
    /// </summary>
    public string Code
    {
        get => _code;
        set => _code = value;
    }

    /// <summary>
    ///     亮度
    /// </summary>
    public int Value
    {
        get => _value;
        set => _value = value;
    }
}

/// <summary>
///     特殊的我的自行车A 有一个额外的旗子
/// </summary>
public class MyBikeA : Bike
{
    /// <summary>
    ///     旗子
    /// </summary>
    private BikeFlag _flag;

    /// <summary>
    ///     旗子编码
    /// </summary>
    private string _flagCode;

    /// <summary>
    ///     旗子编码
    /// </summary>
    public string FlagCode
    {
        get => _flagCode;
        set => _flagCode = value;
    }

    /// <summary>
    ///     旗子
    /// </summary>
    public virtual BikeFlag Flag
    {
        get => _flag;
        set => _flag = value;
    }
}

/// <summary>
///     车旗子
/// </summary>
public class BikeFlag
{
    /// <summary>
    ///     车旗子编码
    /// </summary>
    private string _code;

    /// <summary>
    ///     车旗子文字
    /// </summary>
    private string _value;

    /// <summary>
    ///     车旗子编码
    /// </summary>
    public string Code
    {
        get => _code;
        set => _code = value;
    }

    /// <summary>
    ///     车旗子文字
    /// </summary>
    public string Value
    {
        get => _value;
        set => _value = value;
    }
}

/// <summary>
///     特殊的我的自行车B 有一个额外的车筐
/// </summary>
public class MyBikeB : Bike
{
    /// <summary>
    ///     车筐
    /// </summary>
    private BikeBucket _bucket;

    /// <summary>
    ///     车筐编码
    /// </summary>
    private string _bucketCode;

    /// <summary>
    ///     车筐
    /// </summary>
    public virtual BikeBucket Bucket
    {
        get => _bucket;
        set => _bucket = value;
    }

    /// <summary>
    ///     车筐编码
    /// </summary>
    public string BucketCode
    {
        get => _bucketCode;
        set => _bucketCode = value;
    }
}

/// <summary>
///     车筐
/// </summary>
public class BikeBucket
{
    /// <summary>
    ///     车筐编码
    /// </summary>
    private string _code;

    /// <summary>
    ///     车筐大小
    /// </summary>
    private string _sp;

    /// <summary>
    ///     车筐编码
    /// </summary>
    public string Code
    {
        get => _code;
        set => _code = value;
    }

    /// <summary>
    ///     车筐大小
    /// </summary>
    public string Sp
    {
        get => _sp;
        set => _sp = value;
    }
}

/// <summary>
///      特殊的我的自行车C 是可以共享的
/// </summary>
public class MyBikeC : MyBikeA
{
    /// <summary>
    ///     是否可共享
    /// </summary>
    private bool _canShared;


    /// <summary>
    ///     是否可共享
    /// </summary>
    public bool CanShared
    {
        get => _canShared;
        set => _canShared = value;
    }
}
```
配置时要为父类和子类配置判别类型标识和具体类型判别器.

**目前需要注意的是,子类的所独有的字段不能设置为非空的,因为在插入其他子类时会将这些字段设为空.**

```
//定义一个自行车实体配置
var bikeEntity = modelBuilder.Entity<Bike>();
bikeEntity.HasKeyAttribute(p => p.Code).HasKeyIsSelfIncreased(false);
//此处需要配置类型判别器和根据哪个数据源字段的值来判断 不再需要配置自定义的构造器
//具体配置见下方的BikeConcreteTypeDiscriminator
bikeEntity.HasConcreteTypeDiscriminator(new BikeConcreteTypeDiscriminator(), "Type");
//Bike的Type字段是1 这里的类型需要根据具体的类型进行调整 
//如果此基础类型是抽象的 此处可以配置一个如-1一类的值抽象的类型不会被创建 所以配置一个特殊值即可
//字段名需要与基类类型的HasConcreteTypeDiscriminator方法的第二个参数相同
bikeEntity.HasConcreteTypeSign("Type", 1);

//定义车灯实体配置
var bikeLightEntity = modelBuilder.Entity<BikeLight>();
bikeLightEntity.HasKeyAttribute(p => p.Code).HasKeyIsSelfIncreased(false);

//定义车轮实体配置
var bikeWheelEntity = modelBuilder.Entity<BikeWheel>();
bikeWheelEntity.HasKeyAttribute(p => p.Code).HasKeyIsSelfIncreased(false);

//定义车旗实体配置
var bikeFlagEntity = modelBuilder.Entity<BikeFlag>();
bikeFlagEntity.HasKeyAttribute(p => p.Code).HasKeyIsSelfIncreased(false);

//定义车筐实体配置
var bikeBucketEntity = modelBuilder.Entity<BikeBucket>();
bikeBucketEntity.HasKeyAttribute(p => p.Code).HasKeyIsSelfIncreased(false);

//定义一个特定的我的自行车A
var myBikeAEntity = modelBuilder.Entity<MyBikeA>();
myBikeAEntity.HasKeyAttribute(p => p.Code).HasKeyIsSelfIncreased(false);
//设置继承关系
myBikeAEntity.DeriveFrom<Bike>();
//MyBikeA的Type字段是2 这里的类型需要根据具体的类型进行调整
//字段名需要与基类类型的HasConcreteTypeDiscriminator方法的第二个参数相同
myBikeAEntity.HasConcreteTypeSign("Type", 2);
//设置A和C的具体类型区分器
myBikeAEntity.HasConcreteTypeDiscriminator(new MyBikeConcreteTypeDiscriminator(), "Type");
//此处与父类一起保存于Bike
myBikeAEntity.ToTable("Bike");

//定义一个特定的我的自行车B
var myBikeBEntity = modelBuilder.Entity<MyBikeB>();
myBikeBEntity.HasKeyAttribute(p => p.Code).HasKeyIsSelfIncreased(false);
//设置继承关系
myBikeBEntity.DeriveFrom<Bike>();
//MyBikeB的Type字段是3 这里的类型需要根据具体的类型进行调整
//字段名需要与基类类型的HasConcreteTypeDiscriminator方法的第二个参数相同
myBikeBEntity.HasConcreteTypeSign("Type", 3);
//此处与父类一起保存于Bike
myBikeBEntity.ToTable("Bike");

//定义一个特定的我的自行车C
var myBikeCEntity = modelBuilder.Entity<MyBikeC>();
myBikeCEntity.HasKeyAttribute(p => p.Code).HasKeyIsSelfIncreased(false);
//设置继承关系
myBikeCEntity.DeriveFrom<MyBikeA>();
//MyBikeB的Type字段是4 这里的类型需要根据具体的类型进行调整
//字段名需要与基类类型的HasConcreteTypeDiscriminator方法的第二个参数相同
myBikeCEntity.HasConcreteTypeSign("Type", 4);
//此处与父类一起保存于Bike
myBikeCEntity.ToTable("Bike");

//定义车灯的关联
var bikeAssLight = modelBuilder.Association();
//关联端 关联映射
var bikeEnd1 = bikeAssLight.AssociationEnd<Bike>();
//启用延迟加载
bikeEnd1.AssociationReference(p => p.Light).HasEnableLazyLoading(true);
bikeEnd1.HasMapping("Code", "Code");
bikeAssLight.AssociationEnd<BikeLight>().HasMapping("Code", "LightCode");
bikeAssLight.ToTable("Bike");

//定义车轮的关联
var bikeAssWheel = modelBuilder.Association();
//关联端 关联映射
var bikeEnd2 = bikeAssWheel.AssociationEnd<Bike>();
//启用延迟加载
bikeEnd2.AssociationReference(p => p.Wheels).HasEnableLazyLoading(true);
bikeEnd2.HasMapping("Code", "BikeCode");
bikeAssWheel.AssociationEnd<BikeWheel>().HasMapping("Code", "Code");
bikeAssWheel.ToTable("BikeWheel");

//定义车旗的关联
var mybikeAssFlag = modelBuilder.Association();
//关联端 关联映射
var myBikeEnd1 = mybikeAssFlag.AssociationEnd<MyBikeA>();
myBikeEnd1.AssociationReference(p => p.Flag).HasEnableLazyLoading(true);
//启用延迟加载
myBikeEnd1.HasMapping("Code", "Code");
mybikeAssFlag.AssociationEnd<BikeFlag>().HasMapping("Code", "FlagCode");
mybikeAssFlag.ToTable("Bike");

//定义车筐的关联
var mybikeAssBucket = modelBuilder.Association();
//关联端 关联映射
var myBikeEnd2 = mybikeAssBucket.AssociationEnd<MyBikeB>();
myBikeEnd2.AssociationReference(p => p.Bucket).HasEnableLazyLoading(true);
//启用延迟加载
myBikeEnd2.HasMapping("Code", "Code");
mybikeAssBucket.AssociationEnd<BikeBucket>().HasMapping("Code", "BucketCode");
mybikeAssBucket.ToTable("Bike");

```
具体类型选择器代码如下
```
/// <summary>
///     自行车类型选择器
/// </summary>
public class BikeConcreteTypeDiscriminator : IConcreteTypeDiscriminator
{
    /// <summary>根据类型代码选择一个具体类型。</summary>
    /// <param name="typeCode">类型代码</param>
    public StructuralType Discriminate(object typeCode)
    {
        //这里的类型代码typeCode就是获取到的用于判别类型的值
        //这里我们规定1是Bike 2是MyBikeA 3是myBikeB 4是MyBikeC

        //从模型里取具体的类型 此处获取模型的参数是此配置属于的上下文类型
        var bikeType = GlobalModelCache.Current.GetModel(typeof(SampleContext)).GetStructuralType(typeof(Bike));
        var myBikeAType = GlobalModelCache.Current.GetModel(typeof(SampleContext)).GetStructuralType(typeof(MyBikeA));
        var myBikeBType = GlobalModelCache.Current.GetModel(typeof(SampleContext)).GetStructuralType(typeof(MyBikeB));
        var myBikeCType = GlobalModelCache.Current.GetModel(typeof(SampleContext)).GetStructuralType(typeof(MyBikeC));

        //处理参数
        if (typeCode == null)
            throw new ArgumentException("未能获取类型判别参数.");

        if (typeCode.ToString() == "1")
            return bikeType;

        if (typeCode.ToString() == "2")
            return myBikeAType;

        if (typeCode.ToString() == "3")
            return myBikeBType;

        if (typeCode.ToString() == "4")
            return myBikeCType;

        throw new ArgumentException($"未知的类型判别参数值{typeCode}.");
    }
}

/// <summary>
///     我的自行车类型选择器
/// </summary>
public class MyBikeConcreteTypeDiscriminator : IConcreteTypeDiscriminator
{
    /// <summary>根据类型代码选择一个具体类型。</summary>
    /// <param name="typeCode">类型代码</param>
    public StructuralType Discriminate(object typeCode)
    {
        //这里的类型代码typeCode就是获取到的用于判别类型的值
        //这里我们规定 2是MyBikeA 4是myBikeC

        //从模型里取具体的类型 此处获取模型的参数是此配置属于的上下文类型
        var myBikeAType = GlobalModelCache.Current.GetModel(typeof(SampleContext)).GetStructuralType(typeof(MyBikeA));
        var myBikeCType = GlobalModelCache.Current.GetModel(typeof(SampleContext)).GetStructuralType(typeof(MyBikeC));

        //处理参数
        if (typeCode == null)
            throw new ArgumentException("未能获取类型判别参数.");

        if (typeCode.ToString() == "2")
            return myBikeAType;

        if (typeCode.ToString() == "4")
            return myBikeCType;

        throw new ArgumentException($"未知的类型判别参数值{typeCode}.");
    }
}
```
值得注意的是,有些时候虽然存在继承关系,但并不需要进行继承配置,比如只是复用一些属性.这里提供一个继承关系配置的判断标准:

**如果继承关系中的基类没有引用其他类或被其他类引用则不需要配置急关系,否则都需要配置继承关系.**

设想一个这样的场景,A引用了B,但实际上使用的是B的子类B1,所以就没有配置B为实体型也没有配置B1继承自B,那么在使用A.Include("B")这样的查询时,就会出现"包含路径不合法,类型B没有注册."的错误.

## Java

Java版待重写