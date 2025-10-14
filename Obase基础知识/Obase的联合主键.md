在领域模型中,一个实体型有可能有不止一个主键,此种情况称之为联合主键的实体型.

对于联合主键的实体型,Obase需要在配置时指定所有的主键同时如果此实体型参与了关联,那么相应的映射也要为所有主键都配置映射.

我们以一个教师和教师通行证为例来讲解配置.

## dotNet

```
/// <summary>
///     教师
/// </summary>
public class Teacher
{
    /// <summary>
    ///     教师姓名
    /// </summary>
    private string _name;

    /// <summary>
    ///     所拥有的的通行证
    /// </summary>
    private List<PassPaper> _passPaperList;

    /// <summary>
    ///     教师ID
    /// </summary>
    private long _teacherId;

    /// <summary>
    ///     教师ID
    /// </summary>
    public long TeacherId
    {
        get => _teacherId;
        set => _teacherId = value;
    }

    /// <summary>
    ///     教师名称
    /// </summary>
    public string Name
    {
        get => _name;
        set => _name = value;
    }

    /// <summary>
    ///     所拥有的的通行证
    /// </summary>
    public List<PassPaper> PassPaperList
    {
        get => _passPaperList ?? [];
        set => _passPaperList = value;
    }

    /// <summary>
    ///     转换为字符串表示形式
    /// </summary>
    /// <returns></returns>
    public override string ToString()
    {
        return $"Teacher:{{TeacherId-{_teacherId},Name-\"{_name}\",SchoolId-{_schoolId}}}";
    }
}

/// <summary>
///     教师的通行证
/// </summary>
public class PassPaper
{
    /// <summary>
    ///     教师ID
    /// </summary>
    private readonly long _teacherId;

    /// <summary>
    ///     通行证类型
    /// </summary>
    private readonly EPassPaperType _type;

    /// <summary>
    ///     备注
    /// </summary>
    private string _memo;

    /// <summary>
    ///     所属的教师
    /// </summary>
    private Teacher _teacher;

    /// <summary>
    ///     构造函数
    /// </summary>
    /// <param name="teacherId">教师ID</param>
    /// <param name="type">通行证类型</param>
    public PassPaper(long teacherId, EPassPaperType type)
    {
        _teacherId = teacherId;
        _type = type;
    }

    /// <summary>
    ///     教师ID
    /// </summary>
    public long TeacherId => _teacherId;

    /// <summary>
    ///     通行证类型
    /// </summary>
    public EPassPaperType Type => _type;

    /// <summary>
    ///     备注
    /// </summary>
    public string Memo
    {
        get => _memo;
        set => _memo = value;
    }

    /// <summary>
    ///     所属的教师
    /// </summary>
    public virtual Teacher Teacher
    {
        get => _teacher;
        protected internal set => _teacher = value;
    }
}

```
以上为教师和教师通行证的类定义,其中教师的通行证主键是由教师ID和通行证类型同时作为主键的.

接下来只要做如下的配置即可:

```
//教师的配置
var teacherCfg = modelBuilder.Entity<Teacher>();
//自增主键
teacherCfg.HasKeyAttribute(p => p.TeacherId).HasKeyIsSelfIncreased(true);
//映射表
teacherCfg.ToTable("Teacher");

//通行证 不符合推断 是个联合主键 且 没有无参构造函数
var passPaperCfg = modelBuilder.Entity<PassPaper>();
//联合主键
passPaperCfg.HasKeyAttribute(p => p.TeacherId).HasKeyAttribute(p => p.Type).HasKeyIsSelfIncreased(false);
//构造函数
passPaperCfg.HasConstructor(typeof(PassPaper).GetConstructor(new[] { typeof(long), typeof(EPassPaperType) }))
     .Map(p => p.TeacherId).Map(p => p.Type).End();
//映射表
passPaperCfg.ToTable("PassPaper");

//教师->通行证关联
var teacherPassPaper = modelBuilder.Association();
//teacher端 配置主键TeacherId的映射
teacherPassPaper.AssociationEnd<Teacher>().HasMapping("TeacherId","TeacherId");
//PassPaper端  配置主键TeacherId 和 Type的映射
teacherPassPaper.AssociationEnd<PassPaper>().HasMapping("TeacherId", "TeacherId").HasMapping("Type", "Type");
//映射表
teacherPassPaper.ToTable("PassPaper");
```

与普通的实体型和关联型配置没有较大的区别,只是配置实体型时需要将每个主键都进行配置,配置关联端映射时也需要将所有主键都进行配置.