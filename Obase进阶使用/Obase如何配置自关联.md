一般情况下,关系是存在于两个不同的类之间的,但有些时候关系的两个参与者是相同的,这种情况就称之为自关联.

自然的,自关联也分为两种,隐式自关联和显式自关联.接下来考虑一个如下的场景,Area(区域)表示行政区划,如省,市,区.Area上有两个关联,分别是当前区域的父级和子级区域以及当前区域的友好区域(如友好城市).

那么类定义如下:

## dotNet

```
/// <summary>
///     表示一个区域
/// </summary>
public class Area
{
    /// <summary>
    ///     区域代码
    /// </summary>
    private string _code;

    /// <summary>
    ///     友好区域
    /// </summary>
    private List<FriendlyArea> _friendlyAreas;

    /// <summary>
    ///     名字
    /// </summary>
    private string _name;

    /// <summary>
    ///     父级区域
    /// </summary>
    private Area _parentArea;

    /// <summary>
    ///     父级区域代码
    /// </summary>
    private string _parentCode;

    /// <summary>
    ///     子区域
    /// </summary>
    private List<Area> _subAreas;

    /// <summary>
    ///     区域代码
    /// </summary>
    public string Code
    {
        get => _code;
        set => _code = value;
    }

    /// <summary>
    ///     父级区域代码
    /// </summary>
    public string ParentCode
    {
        get => _parentCode;
        set => _parentCode = value;
    }

    /// <summary>
    ///     父级区域
    /// </summary>
    public Area ParentArea
    {
        get => _parentArea;
        set => _parentArea = value;
    }

    /// <summary>
    ///     子区域
    /// </summary>
    public List<Area> SubAreas
    {
        get => _subAreas;
        set => _subAreas = value;
    }

    /// <summary>
    ///     友好区域
    /// </summary>
    public List<FriendlyArea> FriendlyAreas
    {
        get => _friendlyAreas;
        set => _friendlyAreas = value;
    }

    /// <summary>
    ///     名字
    /// </summary>
    public string Name
    {
        get => _name;
        set => _name = value;
    }
}

/// <summary>
///     友好区域
/// </summary>
public class FriendlyArea
{
    /// <summary>
    ///     区域
    /// </summary>
    private Area _area;

    /// <summary>
    ///     区域代码
    /// </summary>
    private string _areaCode;

    /// <summary>
    ///     友好区域
    /// </summary>
    private Area _friend;

    /// <summary>
    ///     友好区域代码
    /// </summary>
    private string _friendlyAreaCode;

    /// <summary>
    ///     友好关系开始时间
    /// </summary>
    private DateTime _startTime;

    /// <summary>
    ///     区域代码
    /// </summary>
    public string AreaCode
    {
        get => _areaCode;
        set => _areaCode = value;
    }

    /// <summary>
    ///     区域
    /// </summary>
    public virtual Area Area
    {
        get => _area;
        set => _area = value;
    }

    /// <summary>
    ///     友好区域代码
    /// </summary>
    public string FriendlyAreaCode
    {
        get => _friendlyAreaCode;
        set => _friendlyAreaCode = value;
    }

    /// <summary>
    ///     友好区域
    /// </summary>
    public virtual Area Friend
    {
        get => _friend;
        set => _friend = value;
    }

    /// <summary>
    ///     友好关系开始时间
    /// </summary>
    public DateTime StartTime
    {
        get => _startTime;
        set => _startTime = value;
    }
}
```

对于这两个关系,可以辨析出区域和区域之间的父级子级的自关联和友好区域自关联,分别是隐式自关联和显示自关联.那么相应的配置如下:

```
//注册区域
var areaEntity = modelBuilder.Entity<Area>();
areaEntity.HasKeyAttribute(p => p.Code).HasKeyIsSelfIncreased(false);
areaEntity.ToTable("Area");

//区域的自关联
var areaArea = modelBuilder.Association();
//配置第一个Area端
var areaEnd1 = areaArea.AssociationEnd<Area>();
//配置映射
areaEnd1.HasMapping("Code", "Code");
//配置关联引用和关联引用的延迟加载
areaEnd1.AssociationReference(p => p.ParentArea);
//配置第二个Area端
var areaEnd2 = areaArea.AssociationEnd<Area>();
//配置映射
areaEnd2.HasMapping("Code", "ParentCode");
//配置关联引用和关联引用的延迟加载
areaEnd2.AssociationReference(p => p.SubAreas);
//映射表
areaArea.ToTable("Area");

//区域的显式自关联 友好区域
var friendlyArea = modelBuilder.Association<FriendlyArea>();
//配置第一个Area端
var friendlyAreaEnd1 = friendlyArea.AssociationEnd(p => p.Area);
//配置延迟加载 映射
friendlyAreaEnd1.HasMapping("Code", "AreaCode");
//配置关联引用  映射
friendlyAreaEnd1.AssociationReference(p => p.FriendlyAreas);
//配置第二个Area端 配置映射 是否启用延迟加载
friendlyArea.AssociationEnd(p => p.Friend).HasMapping("Code", "FriendlyAreaCode");
//映射表
friendlyArea.ToTable("FriendlyArea");
```

这里与普通的配置不同的地方是,在设置关联端的映射之后,需要指定此关联端上的关联引用,注意配置两个端的具体配置顺序即可.