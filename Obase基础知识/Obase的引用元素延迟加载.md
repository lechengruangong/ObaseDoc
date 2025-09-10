Obase中的引用预算分为关联引用和关联端两种,理论上存在四种形式的引用元素:
1. 实体型上的隐式关联型关联引用,引用的是实体型.
2. 实体型上的显式关联型关联引用,引用的是显式关联型.
3. 隐式关联型上的关联端,引用的是实体型.
4. 显式关联型上的关联端,引用的是实体型.
不过由于隐式关联型是隐含的,所以实际上只有三种形式的引用元素,即1,2,4这三种.

大多情况下,我们在进行查询时就已经知道要和当前查询的对象一起加载哪些关联对象,故可以使用Include方法来进行加载,此外Obase还提供了延迟加载,可以在访问相应的引用元素时加载关联对象.

延迟加载的配置方法位于引用元素上且启用了延迟加载的属性必须为virtual的,如果将某个引用元素配置为启用延迟加载但所对应的属性不是virtual的,会在完整性检查时抛出相应的异常.

## dotNet

此处使用与[Obase对象数据模型配置示例](./Obase对象数据模型配置示例.md)中类似的类定义来做演示:

```
/// <summary>
///     班级
/// </summary>
public class Class
{
    /// <summary>
    ///     班级ID
    /// </summary>
    public int Id { get; protected internal set; }

    /// <summary>
    ///     班级名称
    /// </summary>
    public string Name { get; set; }

    /// <summary>
    ///     学生列表
    /// </summary>
    public virtual List<Student> Students { get; protected internal set; } = [];

    /// <summary>
    ///     任课教师列表
    /// </summary>
    public virtual List<Teaching> Teachings { get; protected internal set; } = [];
}

/// <summary>
///     学生
/// </summary>
public class Student
{
    /// <summary>
    ///     班级ID
    /// </summary>
    public int Id { get; protected internal set; }

    /// <summary>
    ///     学生姓名
    /// </summary>
    public string Name { get; set; }

    /// <summary>
    ///     年龄
    /// </summary>
    public int Age { get; set; }
}

/// <summary>
///     教师ID
/// </summary>
public class Teacher
{
    /// <summary>
    ///     教师ID
    /// </summary>
    public int Id { get; protected internal set; }

    /// <summary>
    ///     教师姓名
    /// </summary>
    public string Name { get; set; }

    /// <summary>
    ///     年龄
    /// </summary>
    public int Age { get; set; }
    
    /// <summary>
    ///     班级ID
    /// </summary>
    public int ClassId { get; protected internal set; }
}

/// <summary>
///     任课关系
/// </summary>
public class Teaching
{
    /// <summary>
    ///     授课科目
    /// </summary>
    public string Course { get; set; }

    /// <summary>
    ///     班级ID
    /// </summary>
    public int ClassId { get; protected internal set; }

    /// <summary>
    ///     教师ID
    /// </summary>
    public int TeacherId { get; protected internal set; }

    /// <summary>
    ///     班级
    /// </summary>
    public virtual Class Class { get; protected internal set; }

    /// <summary>
    ///     教师
    /// </summary>
    public virtual Teacher Teacher { get; protected internal set; }
}
```

此处的实体型Class有两个关联引用,分别是隐式关联型的关联引用Students和显式关联型的关联引用Teachings;显式关联型Teaching有两个关联端Class和Teacher.

注意这些引用元素都需要定义为virtual的.

以下是配置部分:

```
//配置班级
var classEntity = modelBuilder.Entity<Class>();
//班级主键
classEntity.HasKeyAttribute(p => p.Id).HasKeyIsSelfIncreased(true);
//班级存储在哪张表
classEntity.ToTable("Class");

//配置学生
var studentEntity = modelBuilder.Entity<Student>();
//学生主键
studentEntity.HasKeyAttribute(p => p.Id).HasKeyIsSelfIncreased(true);
//学生存储在哪张表
studentEntity.ToTable("Student");
//学生的家庭住址
modelBuilder.Complex<Address>();

//配置教师
var teacherEntity = modelBuilder.Entity<Teacher>();
//教师主键
teacherEntity.HasKeyAttribute(p => p.Id).HasKeyIsSelfIncreased(true);
//教师存储在哪张表
teacherEntity.ToTable("Teacher");

//配置班级和学生的关系
var classStudent = modelBuilder.Association();
//班级关联端
var classEnd = classStudent.AssociationEnd<Class>();
//在Student这张表里班级的主键ID存储于ClassId这个数据库字段里
classEnd.HasMapping("Id", "ClassId");
//为班级这一端的学生引用启用延迟加载
classEnd.AssociationReference(p => p.Students).HasEnableLazyLoading(true);
//学生关联端
var studentEnd = classStudent.AssociationEnd<Student>();
//在Student这张表里学生的主键ID存储于Id这个数据库字段里
studentEnd.HasMapping("Id", "Id");
//班级和学生的关系存储在哪张表
classStudent.ToTable("Student");

//配置任课关系
var teaching = modelBuilder.Association<Teaching>();
//班级关联端
var classTeachingEnd = teaching.AssociationEnd(p =>p.Class);
//在Teaching这张表里班级的主键ID存储于ClassId这个数据库字段里
classTeachingEnd.HasMapping("Id", "ClassId");
//为任课关系的班级这一端启用延迟加载
classTeachingEnd.HasEnableLazyLoading(true);
//为班级这一端的任课引用启用延迟加载
classTeachingEnd.AssociationReference(p => p.Teachings).HasEnableLazyLoading(true);
//教师关联端
var teacherTeachingEnd = teaching.AssociationEnd(p => p.Teacher);
//在Teaching这张表里教师的主键ID存储于TeacherId这个数据库字段里
teacherTeachingEnd.HasMapping("Id", "TeacherId");
//为任课关系的教师这一端启用延迟加载
teacherTeachingEnd.HasEnableLazyLoading(true);
//任课关系存储在哪张表
teaching.ToTable("Teaching");
```

注意这些配置中,需要在引出关联引用和关联端的配置后使用HasEnableLazyLoading(bool)来配置关联引用延迟加载的启用和关闭(默认关闭).

启用了延迟加载后,即可在访问引用元素对应的属性时查询数据源加载对象,类似于如下的代码:

```
//查询班级 此处假设数据库内已有相关数据
var cla = context.CreateSet<Class>().First();
//查询班级的任课教师
var teachings = cla.Teachings;
//延迟加载 不为空
Debug.Assert(teachings != null);
//查询班级的学生
var students = cla.Students;
//延迟加载 不为空
Debug.Assert(students != null);
```

需要注意的是,谨慎的将延迟加载和循环结合使用,比如一个列表中有10个对象,这些对象的某个属性是启用了延迟加载的引用元素对应的属性,那么在访问这些属性时就会访问10次数据源来加载对象,造成性能下降.