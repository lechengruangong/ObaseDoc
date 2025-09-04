本篇以一个简单的例子来讲解Obase对象数据模型.

## 概念辨析

我准备定义一个包含班级,学生和教师的简单领域模型,在这个模型中一个班级包含多名学生,一个教师可以在多个不同的班级任教且可能会教授不同的课程,学生还需要一个结构来存储家庭住址.

那么我就在这个领域模型中,应当包含以下的实体:
1. 学生
2. 班级
3. 教师

包含以下的关系:
1. 学生和班级之间的关系,班级需要引用多个学生.
2. 教师和班级之间的关系,且这个关系有独属于关系的属性任教科目.

## 定义类型

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

## 配置模型
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