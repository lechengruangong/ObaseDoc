配置对象数据模型时,如果配置有误,一般需要等到构造上下文时才会通过完整性检查发现,而构造上下文需要配置数据源和连接字符串等与数据源相关的内容.有些场景下,我们希望在不进行任何数据源操作的前提下直接验证注册的模型,例如在单元测试中验证模型配置,或者在应用启动时提前发现模型配置的错误.

为此,Obase提供了一个抽象类OdmValidator,继承此抽象类并实现CreateModel方法注册模型之后,调用Validate方法即可直接建造一次对象数据模型并对其进行完整性检查,整个过程不会构造上下文,也不会连接或者访问数据源.

## 定义验证器

OdmValidator位于Obase.Core.Odm命名空间,其中包含Validate方法和抽象方法CreateModel(ModelBuilder modelBuilder),Validate方法的返回值为ValidationResult,此返回值的内容见后文.

继承OdmValidator并实现CreateModel方法,将模型注册代码放入此方法即可:

```
/// <summary>
///     班级模型的ODM验证器
/// </summary>
public class ClassOdmValidator : OdmValidator
{
    /// <summary>
    ///     使用指定的建模器创建对象数据模型
    /// </summary>
    /// <param name="modelBuilder">对象数据模型建造器</param>
    protected override void CreateModel(ModelBuilder modelBuilder)
    {
        //配置班级
        var classEntity = modelBuilder.Entity<Class>();
        classEntity.HasKeyAttribute(p => p.Id).HasKeyIsSelfIncreased(true);
        classEntity.ToTable("Class");

        //配置学生
        var studentEntity = modelBuilder.Entity<Student>();
        studentEntity.HasKeyAttribute(p => p.Id).HasKeyIsSelfIncreased(true);
        studentEntity.ToTable("Student");
        modelBuilder.Complex<Address>();

        //配置教师
        var teacherEntity = modelBuilder.Entity<Teacher>();
        teacherEntity.HasKeyAttribute(p => p.Id).HasKeyIsSelfIncreased(true);
        teacherEntity.ToTable("Teacher");

        //配置班级和学生的关系
        var classStudent = modelBuilder.Association();
        classStudent.AssociationEnd<Class>().HasMapping("Id", "ClassId");
        classStudent.AssociationEnd<Student>().HasMapping("Id", "Id");
        classStudent.ToTable("Student");

        //配置任课关系
        var teaching = modelBuilder.Association<Teaching>();
        teaching.AssociationEnd(p => p.Class).HasMapping("Id", "ClassId");
        teaching.AssociationEnd(p => p.Teacher).HasMapping("Id", "TeacherId");
        teaching.ToTable("Teaching");
    }
}
```

此处的注册代码与上下文配置提供器的CreateModel方法内的注册代码相同,关于模型注册可以参考[Obase如何配置对象数据模型](../Obase入门/Obase如何配置对象数据模型_dotNet.md),关于配置提供器可以参考[Obase如何定义和管理对象上下文配置提供器](../Obase入门/Obase如何定义和管理对象上下文配置提供器_dotNet.md).

推荐将模型注册代码提取为一个公共的注册类或者注册方法,由验证器和配置提供器共同调用,这样可以保证验证的模型与实际使用的模型是一致的,避免两处配置出现不同步的情况.

## 进行验证

定义好验证器之后,构造验证器并调用Validate方法即可进行验证:

```
//构造验证器
var validator = new ClassOdmValidator();
//进行验证
var result = validator.Validate();

//查看验证是否通过
if (result.IsValid)
    Console.WriteLine("模型验证通过");
else
    //验证未通过时Message内是完整性检查的错误信息
    Console.WriteLine(result.Message);
```

Validate方法每次调用时都会新建一个模型建造器并重新建造模型,所以可以反复调用,上一次的验证结果不会影响下一次验证,因此也可以直接在单元测试中这样断言:

```
[Test]
public void ModelShouldBeValid()
{
    //进行验证 验证未通过时把错误信息作为断言消息输出
    var result = new ClassOdmValidator().Validate();
    Assert.That(result.IsValid, Is.True, result.Message);
}
```

需要注意的是,验证器建造的模型只在本次验证中使用,不会成为任何上下文的模型;而且由于验证器不提供上下文,建造模型时上下文的类型为空,所以与上下文类型相关的配置(例如多租户的租户标识)在验证器中不会生效,这类配置需要在真实的上下文中验证.

## 验证结果

Validate方法的返回值ValidationResult包含IsValid和Message两个属性,IsValid表示验证是否通过,Message是验证结果信息,验证通过和验证未通过时Message的内容是不同的.

### 验证通过

验证通过时IsValid为true,Message内是此对象数据模型的完整映射关系视图,内容与[Obase如何查看对象数据模型的映射关系](./Obase如何查看对象数据模型的映射关系_dotNet.md)中介绍的GetFullObjectDataModelMappingView方法返回的相同,可以直接输出用于查看模型的映射关系.

以上文的班级模型为例(示例中的类型均位于Demo.Domain命名空间下),验证通过后得到的Message类似于以下内容(此处仅节选班级的部分,实际输出会包含模型中所有实体型的映射关系):

```
本模型共包含3个实体型.

实体型Demo.Domain.Class的映射表为Class.
实体型Demo.Domain.Class共有1个主键.
1. 自增主键Id,映射类型System.Int32,映射字段Id.
实体型Demo.Domain.Class共有2个属性.
1. 简单属性Id,映射类型System.Int32,映射字段Id.
2. 简单属性Name,映射类型System.String,映射字段Name.
实体型Demo.Domain.Class共有2个关联引用.
1. 关联引用Students,对应关联型为ImplicitAssociation_Class_Student<>1,映射表为Student.
在映射表Student中,共有关联端2个.
关联端Demo.Domain.Class的映射为:
主键Id映射为ClassId
关联端Demo.Domain.Student的映射为:
主键Id映射为Id
...
```

### 验证未通过

验证未通过时IsValid为false,Message内是按照类型分组的完整性检查错误信息,每一组信息中包含了类型名称(类型的全名),此类型下的错误个数以及具体的错误信息.

以忘记为课程类型配置主键的验证器为例:

```
/// <summary>
///     一个没有为课程配置主键的验证器
/// </summary>
public class InvalidClassValidator : OdmValidator
{
    /// <summary>
    ///     使用指定的建模器创建对象数据模型
    /// </summary>
    /// <param name="modelBuilder">对象数据模型建造器</param>
    protected override void CreateModel(ModelBuilder modelBuilder)
    {
        //配置课程时忘记配置主键
        modelBuilder.Entity<Course>().ToTable("Course");
    }
}
```

此时得到的Message类似于以下内容:

```
类型Demo.Domain.Course存在1个完整性检查错误:
1. [ODM]实体Course未配置主键,请为实体指定主键属性.
```

错误信息前的[ODM]表示此错误是对象数据模型(Object Data Model)的错误,[SODM]表示此错误是序列化对象数据模型(Serialization Object Data Model)的错误,这两个前缀也同样定义在IntegrityCheckFailException的OdmMessagePrefix和SodmMessagePrefix常量中.关于序列化对象数据模型可以参考[Obase如何配置属性复杂序列化](./Obase如何配置属性复杂序列化_dotNet.md).

建造模型时会先对序列化对象数据模型进行完整性检查,如果序列化对象数据模型的配置有误,则得到的Message内只会包含带[SODM]前缀的错误信息,此时不会再进行对象数据模型的完整性检查.序列化实体相关的错误信息类似于以下内容:

```
类型XXX存在1个完整性检查错误:
1. [SODM]XXX的构造器应有5个参数,实际只配置了1个.
```

以上两种错误信息的具体处理方式可以参考[Obase的对象数据模型完整性检查与错误速查](../Obase基础知识/Obase的对象数据模型完整性检查与错误速查_dotNet.md).

## 注意事项

- 验证器建造模型时不会执行结构映射,所以不会在数据源中建表,需要在数据源中建表时请参考[Obase如何配置结构映射](./Obase如何配置结构映射_dotNet.md).
- Validate方法只会捕获完整性检查未通过的异常IntegrityCheckFailException,如果注册代码中存在其他问题(例如配置的类型无法反射构造等),这些异常会直接抛出.
- 不要在验证器的CreateModel方法中调用HasIntegrityCheck(false)关闭完整性检查,验证器内部已经为此模型建造器启用了完整性检查,如果在注册代码中关闭,验证将失去意义.
