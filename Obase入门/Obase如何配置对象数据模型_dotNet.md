## 全面考虑的思路

以下所有内容中的,检查是否符合推断中的推断都指的是[Obase的配置有哪些默认值(dotNet版)](./Obase的配置有哪些默认值(dotNet版).md)里的推断.

1. **注册程序集**.在重写的配置方法中,将领域模型的Assembly调用modelBuilder.RegisterType方法注册.这样做可以将整个领域模型中符合推断的类型都注册为Obase的对象模型类型,之后碰到配置的类型如果符合推断就无需再次进行配置了.

2. **添加忽略类型**.领域模型内可能存在一些不需要Obase管理的类,如某些接口的默认实现类,不需要独立存储的类等,将这些类型调用modelBuilder.Ignore方法忽略掉.

3. **检查实体型配置**.挑选一个要配置为实体型类检查.依次检查以下几项:
    1. 是否符合推断?不符合,则要调用Entity方法将其注册为实体型配置.符合,暂时不调用Entity方法创建实体配置项如果以下几个都符合推断,则不需要创建,否则仍需要创建实体配置.
    2. 是否符合主键推断?是否符合主键自增的推断?不符合,则需要在实体型配置调用HasKeyAttribute和HasKeyIsSelfIncreased.符合则不需要调用.
    3. 是否符合构造函数推断?不符合则需要实体型配置调用HasConstructor方法配置.符合则不需要调用.
    4. 是否符合映射表推断?不符合则需要实体型配置调用ToTable方法配置,符合则不需要调用.
    5. (可选)是否需要配置并发冲突相关配置?是否需要配置对象变更通知相关配置?如果需要,则在实体型配置上调用相应的方法.

4. **检查实体型的属性配置**.继续对第三步的类属性进行检查,以确定属性和关联引用.
    1. 是否是一个属性?是否符合属性的推断?不符合,则需要实体型配置调用Attribute方法配置,符合则不需要.
    2. (可选)是否需要配置属性的并发冲突策略?是否需要配置精度,最大长度,是否可空?如果需要,则在属性配置上调用相应的方法.
    3. 是否是一个关联引用?如果是隐式关联的关联引用(即关联引用的元素类型是另外一个实体型),转入**步骤5**.如果是显式关联的关联引用(即关联引用的元素类型是显式关联型),转入**步骤6.**

5. **检查隐式关联型的配置**.按照以下步骤检查此隐式关联:
    1. 是否是一个标准的两方或多方隐式关联(即引用是否是关系的另外一方或者用元组Tuple组成的多方)?如果不是,则需要调用modelBuilder.Association方法启动一个新的隐式关联建造器进行配置.否则,继续检查以下几项,如果都符合推断,则不需要配置.
    2. 检查此关联型的映射表是否符合推断?如果不符合,调用隐式关联建造器ToTable方法配置;否则不需要配置.
    3. 检查关联的每一方,是否都符合推断?如果不是,则需要调用隐式关联建造器的AssociationEnd方法声明关联端,并配置相应的映射,取值器,设值器等.否则不需要配置.
    4. 检查检查关联的每一方的关联端,端上针对此关联型的关联引用是否符合推断?不符合则需要配置相应的取值器,设值器,是否延迟加载等.否则不需要配置.
    5. (可选)检查关联的每一方的关联端,是否都是用默认的延迟加载,是否聚合,是否默认附加至上下文?如果不符合,则调用相应的方法配置.否则不需要配置.

6. **检查显式关联型的配置**.按照以下步骤检查此显式关联:
    1. 此显式关联是否符合推断?如果不是,则需要调用modelBuilder.Association<>方法配置显式关联型,否则,继续检查以下几项,如果都符合推断,则不需要配置.
    2. 检查此关联型的映射表是否符合推断?如果不符合,调用显式关联型配置的ToTable方法配置;否则不需要配置.
    3. 检查关联的每一方,是否都符合推断?如果不是,则需要调用隐式关联建造器的AssociationEnd方法声明关联端,并配置相应的映射,取值器,设值器等.否则不需要配置.
    4. 检查检查关联的每一方的关联端,端上针对此关联型的关联引用是否符合推断?不符合则需要配置相应的取值器,设值器,是否延迟加载等.否则不需要配置.

7. **重复以上的步骤3至步骤6**,直到所有的类型都被配置完成.

## 简明思路

Obase的配置实质上只关心这些内容:
- 哪些类型是实体型,哪些类型是关联型,这些类型如何映射.
- 这些实体型之间的关系是如何映射的.
- 这些实体型和关联型上的属性是如何映射的.

所以只要配置这些基础的配置就可以构建对象数据模型了,那么就有如下的简明配置思路.

1. 将要配置的对象系统中所有的由Obase管理存储的类型分类为实体型(表示一个实体,有主键的类型)和显式关联型(表示实体之间的关系,且有属于关系的属性)这两个类别.
2. 将所有的实体型都调用modelBuilder的Entity<>()注册为实体型,并且配置主键和映射表,代码类似于以下三行```var cfg = modelBuilder.Entity<XXX>(); cfg.HasKeyAttribute(p => p.XXX); cfg.ToTable("XXX");```.
3. 将所有的显式关联型都调用modelBuilder的Association<>()注册为显式关联型,并配置关联型的关联端及端映射和关联的映射表,代码类似于以下四行```var cdg = modelBulder.Association<XXX>();cfg.AssociationEnd(p=>p.XXX).HasMapping("XXX","XXX");cfg.AssociationEnd(p=>p.YYY).HasMapping("YYY","YYY");cfg.ToTable("XXX"); ```一个关联至少有两个实体型参与,所以此处至少需要两个关联端.
4. 检查所有的实体型,这些实体型除了基元类型的属性以外,是否还引用了其他的实体型,如果有这些实体型的引用代表存在隐含的关联,需要将这些隐含的关联注册为隐式关联型,并配置关联型的关联端及端映射和关联的映射表,代码类似于以下四行```var cfg = modelBuilder.Association();cfg.AssociationEnd<XXX>().HasMapping("XXX","XXX"); cfg.AssociationEnd<YYY>().HasMapping("YYY","YYY");cfg.ToTable("XXX");```一个关联至少有两个实体型参与,所以此处至少需要两个关联端.
5. 反复使用以上的代码,将所有的实体型和关联型都注册,需要注意的是如果A和B有隐式关联,只需要注册一次即可除非A和B存在多个不同的关系.
6. 检查所有的实体型和关联型上的普通属性和构造器,这些属性和构造器是否需要特殊的处理,如果需要为其配置相应的取值器设值器映射等配置.检查所有的实体型上的其他实体型或关联型的引用属性,这些属性是否需要特殊的处理,如果需要为其配置相应的取值器设值器等配置.
7. 最后,如果有需要特殊配置的,如继承配置,对象更改通知,两个类之间有多个关系之类的,参照Obase的进阶使用中相关的内容进行配置即可.

## 配置示例

我准备定义一个包含班级,学生和教师的简单领域模型,在这个模型中一个班级包含多名学生,一个教师可以在多个不同的班级任教且可能会教授不同的课程,学生还需要一个结构来存储家庭住址.

那么我就在这个领域模型中,应当包含以下的实体:
1. 学生
2. 班级
3. 教师

包含以下的关系:
1. 学生和班级之间的关系,班级需要引用多个学生.
2. 教师和班级之间的关系,且这个关系有独属于关系的属性任教科目.

### 定义类型

辨析清楚概念后,就可以做出如下的定义:

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
    public List<Student> Students { get; protected internal set; } = [];

    /// <summary>
    ///     任课教师列表
    /// </summary>
    public List<Teaching> Teachings { get; protected internal set; } = [];
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

    /// <summary>
    ///     家庭住址
    /// </summary>
    public Address Address { get; set; }
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
    public Class Class { get; protected internal set; }

    /// <summary>
    ///     教师
    /// </summary>
    public Teacher Teacher { get; protected internal set; }
}

/// <summary>
///     家庭住址
/// </summary>
public struct Address
{
    /// <summary>
    ///     省份
    /// </summary>
    public string Province { get; set; }

    /// <summary>
    ///     城市
    /// </summary>
    public string City { get; set; }

    /// <summary>
    ///     区县
    /// </summary>
    public string District { get; set; }

    /// <summary>
    ///     详细地址
    /// </summary>
    public string Detail { get; set; }
}
```
定义了班级,学生,教师,家庭住址,并且班级引用了任课关系和学生,都是一对多的.定义的内容与概念辨析中一致,只在原有的概念上对一些属性进行了封装.

### 配置模型
接下来就可以进行配置了,首先先对这个简单的领域模型进行分类:

实体型包括班级,学生,教师;关联型包括任课关系以及班级和学生之间隐含的关系;家庭住址是一个复杂类型,作为学生的复杂属性类型.

然后先配置实体型:
首先考虑班级,班级需要定义为实体型,然后班级有一个主键ID,这个ID是自增的,最后班级在数据库里存储到Class这张表里.

班级的其他属性都是Obase的基元类型,也没有什么特别需要配置的地方,跳过.

班级还有两个引用其他实体的属性,这些放到关系部分配置,那么配置代码就是这样的:

```
//配置班级
var classEntity = modelBuilder.Entity<Class>();
classEntity.HasKeyAttribute(p => p.Id).HasKeyIsSelfIncreased(true);
classEntity.ToTable("Class");
```
然后考虑学生,学生需要定义为实体型,学生的主键是ID,自增的,最后学生仔数据库存储到Student表.

学生的其他属性里有一个家庭住址,需要配置为复杂类型和复杂属性,其他的是基元类型可以跳过.

学生也没有引用其他的实体型,不需要考虑配置关系,那么配置代码就是这样的:

```
//配置学生
var studentEntity = modelBuilder.Entity<Student>();
//学生主键
studentEntity.HasKeyAttribute(p => p.Id).HasKeyIsSelfIncreased(true);
//学生存储在哪张表
studentEntity.ToTable("Student");
//学生的家庭住址
modelBuilder.Complex<Address>();
```
再考虑教师,教师需要定义为实体型,主键是ID,自增的,最后教师要存储到Teacher表.

教师其他属性都是Obase基元类型,也没有引用其他的实体型,所以配置代码就是这样的:

```
//配置教师
var teacherEntity = modelBuilder.Entity<Teacher>();
//教师主键
teacherEntity.HasKeyAttribute(p => p.Id).HasKeyIsSelfIncreased(true);
//教师存储在哪张表
teacherEntity.ToTable("Teacher");
```
接下来配置关联型:

我们现在有两个关系要配置,首先是班级和学生之间的关系,这个关系没有独属于关系的属性,显然是个隐式关联.

对于隐式关联,只要配置关联端,关联端的映射,关联存储于哪张表即可.

对于这个关系,有两个端参与其中,分别是班级和学生;班级包含多个学生,那么只需要在学生表内存储班级的主键就可以表示这个关系.

所以这个关系应当存储于Student表,在这张表里学生的主键Id就存储于Id这个数据库字段中,班级的主键Id则存储于ClassId这个数据库字段中,综上配置代码如下:
```
//配置班级和学生的关系
var classStudent = modelBuilder.Association();
//班级关联端
var classEnd = classStudent.AssociationEnd<Class>();
//在Student这张表里班级的主键ID存储于ClassId这个数据库字段里
classEnd.HasMapping("Id", "ClassId");
//学生关联端
var studentEnd = classStudent.AssociationEnd<Student>();
//在Student这张表里学生的主键ID存储于Id这个数据库字段里
studentEnd.HasMapping("Id", "Id");
//班级和学生的关系存储在哪张表
classStudent.ToTable("Student");
```

然后是任课这个关系,这个关系定义显式关联型Teaching,上面各自引用了班级和教师.

对于显式关联,需要配置关联端,关联端的映射,关联存储于哪张表.此外的属性是Obase基元类型,所以不需要配置.

对于这个关系,显然是存储于Teaching这张表里的.有两个端参与其中,分别是班级和教师,那么在Teaching这张表里分别存储班级和教师的主键ID即可.

所以这个关系应当存储于Teaching表,在这张表里班级的主键Id就存储于ClassId这个数据库字段中,教师的主键Id则存储于TeacherId这个数据库字段中,综上配置代码如下:

```
//配置任课关系
var teaching = modelBuilder.Association<Teaching>();
//班级关联端
var classTeachingEnd = teaching.AssociationEnd(p =>p.Class);
//在Teaching这张表里班级的主键ID存储于ClassId这个数据库字段里
classTeachingEnd.HasMapping("Id", "ClassId");
//教师关联端
var teacherTeachingEnd = teaching.AssociationEnd(p => p.Teacher);
//在Teaching这张表里教师的主键ID存储于TeacherId这个数据库字段里
teacherTeachingEnd.HasMapping("Id", "TeacherId");
//任课关系存储在哪张表
teaching.ToTable("Teaching");
```

最后,将所有的配置综合起来如下:
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
//教师关联端
var teacherTeachingEnd = teaching.AssociationEnd(p => p.Teacher);
//在Teaching这张表里教师的主键ID存储于TeacherId这个数据库字段里
teacherTeachingEnd.HasMapping("Id", "TeacherId");
//任课关系存储在哪张表
teaching.ToTable("Teaching");
```
至此我们就完成了对这个简单领域的Obase配置.