配置对象数据模型时,如果配置有误,一般需要等到构造上下文时才会通过完整性检查发现,而构造上下文需要配置数据源和连接字符串等与数据源相关的内容.有些场景下,我们希望在不进行任何数据源操作的前提下直接验证注册的模型,例如在单元测试中验证模型配置,或者在应用启动时提前发现模型配置的错误.

为此,Obase提供了一个抽象类OdmValidator,继承此抽象类并实现createModel方法注册模型之后,调用validate方法即可直接建造一次对象数据模型并对其进行完整性检查,整个过程不会构造上下文,也不会连接或者访问数据源.

## 定义验证器

OdmValidator位于io.obase.core.odm包,其中包含validate方法和抽象方法createModel(ModelBuilder modelBuilder),validate方法的返回值为ValidationResult,此返回值的内容见后文.

继承OdmValidator并实现createModel方法,将模型注册代码放入此方法即可:

```
/**
 * 班级模型的ODM验证器
 */
public class ClassOdmValidator extends OdmValidator {

    /**
     * 使用指定的建模器创建对象数据模型
     *
     * @param modelBuilder 对象数据模型建造器
     */
    @Override
    protected void createModel(ModelBuilder modelBuilder) {
        //配置班级
        EntityTypeConfiguration<Class> classEntity = modelBuilder.entity(Class.class);
        classEntity.hasKeyAttribute(p -> p.getId()).hasKeyIsSelfIncreased(true);
        classEntity.toTable("Class");

        //配置学生
        EntityTypeConfiguration<Student> studentEntity = modelBuilder.entity(Student.class);
        studentEntity.hasKeyAttribute(p -> p.getId()).hasKeyIsSelfIncreased(true);
        studentEntity.toTable("Student");
        modelBuilder.complex(Address.class);

        //配置教师
        EntityTypeConfiguration<Teacher> teacherEntity = modelBuilder.entity(Teacher.class);
        teacherEntity.hasKeyAttribute(p -> p.getId()).hasKeyIsSelfIncreased(true);
        teacherEntity.toTable("Teacher");

        //配置班级和学生的关系
        AssociationConfiguratorBuilder classStudent = modelBuilder.association();
        classStudent.associationEnd(Class.class).hasMapping("Id", "ClassId");
        classStudent.associationEnd(Student.class).hasMapping("Id", "Id");
        classStudent.toTable("Student");

        //配置任课关系
        AssociationEndConfigurationGeneric<Teaching> teaching = modelBuilder.association(Teaching.class);
        teaching.associationEnd(p -> p.getClazz()).hasMapping("Id", "ClassId");
        teaching.associationEnd(p -> p.getTeacher()).hasMapping("Id", "TeacherId");
        teaching.toTable("Teaching");
    }
}
```

此处的注册代码与上下文配置提供器的createModel方法内的注册代码相同,关于模型注册可以参考[Obase如何配置对象数据模型](../Obase入门/Obase如何配置对象数据模型_java.md),关于配置提供器可以参考[Obase如何定义和管理对象上下文配置提供器](../Obase入门/Obase如何定义和管理对象上下文配置提供器_java.md).

推荐将模型注册代码提取为一个公共的注册类或者注册方法,由验证器和配置提供器共同调用,这样可以保证验证的模型与实际使用的模型是一致的,避免两处配置出现不同步的情况.

## 进行验证

定义好验证器之后,构造验证器并调用validate方法即可进行验证:

```
//构造验证器
ClassOdmValidator validator = new ClassOdmValidator();
//进行验证
OdmValidator.ValidationResult result = validator.validate();

//查看验证是否通过
if (result.getIsValid())
    System.out.println("模型验证通过");
else
    //验证未通过时getMessage返回的是完整性检查的错误信息
    System.out.println(result.getMessage());
```

validate方法每次调用时都会新建一个模型建造器并重新建造模型,所以可以反复调用,上一次的验证结果不会影响下一次验证,因此也可以直接在单元测试中这样断言:

```
@Test
public void modelShouldBeValid() {
    //进行验证 验证未通过时把错误信息作为断言消息输出
    var result = new ClassOdmValidator().validate();
    assertTrue(result.getIsValid(), result.getMessage());
}
```

需要注意的是,验证器建造的模型只在本次验证中使用,不会成为任何上下文的模型;而且由于验证器不提供上下文,建造模型时上下文的类型为空,所以与上下文类型相关的配置(例如多租户的租户标识)在验证器中不会生效,这类配置需要在真实的上下文中验证.

## 验证结果

validate方法的返回值ValidationResult包含getIsValid和getMessage两个方法,getIsValid表示验证是否通过,getMessage返回验证结果信息,验证通过和验证未通过时信息的内容是不同的.

### 验证通过

验证通过时getIsValid返回true,getMessage返回的是此对象数据模型的完整映射关系视图,内容与[Obase如何查看对象数据模型的映射关系](./Obase如何查看对象数据模型的映射关系_java.md)中介绍的getFullObjectDataModelMappingView方法返回的相同,可以直接输出用于查看模型的映射关系.

以上文的班级模型为例(示例中的类均位于valdemo.domain包下),验证通过后得到的信息类似于以下内容(此处仅节选班级的部分,实际输出会包含模型中所有实体型的映射关系):

```
本模型共包含3个实体型.

实体型valdemo.domain.Class的映射表为Class.
实体型valdemo.domain.Class共有1个主键.
1. 自增主键Id,映射类型int,映射字段Id.
实体型valdemo.domain.Class共有2个属性.
1. 简单属性Id,映射类型int,映射字段Id.
2. 简单属性Name,映射类型class java.lang.String,映射字段Name.
实体型valdemo.domain.Class共有2个关联引用.
1. 关联引用Students,对应关联型为io.obase.proxy.module.ImplicitAssociation_Class_Student_1,映射表为Student
在映射表Student中,共有关联端2个.
关联端valdemo.domain.Student的映射为:
主键Id映射为Id
关联端valdemo.domain.Class的映射为:
主键Id映射为ClassId
...
```

### 验证未通过

验证未通过时getIsValid返回false,getMessage返回的是按照类型分组的完整性检查错误信息,每一组信息中包含了类型名称(类型的全名),此类型下的错误个数以及具体的错误信息.

以忘记为课程类型配置主键的验证器为例:

```
/**
 * 一个没有为课程配置主键的验证器
 */
public class InvalidClassValidator extends OdmValidator {

    /**
     * 使用指定的建模器创建对象数据模型
     *
     * @param modelBuilder 对象数据模型建造器
     */
    @Override
    protected void createModel(ModelBuilder modelBuilder) {
        //配置课程时忘记配置主键
        modelBuilder.entity(Course.class).toTable("Course");
    }
}
```

此时得到的信息类似于以下内容:

```
类型valdemo.domain.Course存在1个完整性检查错误:
1. [ODM]实体Course未配置主键,请为实体指定主键属性.
```

错误信息前的[ODM]表示此错误是对象数据模型(Object Data Model)的错误,[SODM]表示此错误是序列化对象数据模型(Serialization Object Data Model)的错误,这两个前缀也同样定义在IntegrityCheckFailException的ODM_MESSAGE_PREFIX和SODM_MESSAGE_PREFIX常量中.关于序列化对象数据模型可以参考[Obase如何配置属性复杂序列化](./Obase如何配置属性复杂序列化_java.md).

建造模型时会先对序列化对象数据模型进行完整性检查,如果序列化对象数据模型的配置有误,则得到的信息内只会包含带[SODM]前缀的错误信息,此时不会再进行对象数据模型的完整性检查.序列化实体相关的错误信息类似于以下内容:

```
类型XXX存在1个完整性检查错误:
1. [SODM]XXX的构造器应有5个参数,实际只配置了1个.
```

以上两种错误信息的具体处理方式可以参考[Obase的对象数据模型完整性检查与错误速查](../Obase基础知识/Obase的对象数据模型完整性检查与错误速查_java.md).

## 注意事项

- 验证器建造模型时不会执行结构映射,所以不会在数据源中建表,需要在数据源中建表时请参考[Obase如何配置结构映射](./Obase如何配置结构映射_java.md).
- validate方法只会捕获完整性检查未通过的异常IntegrityCheckFailException,如果注册代码中存在其他问题(例如配置的类型无法反射构造等),这些异常会直接抛出.
- 不要在验证器的createModel方法中调用hasIntegrityCheck(false)关闭完整性检查,验证器内部已经为此模型建造器启用了完整性检查,如果在注册代码中关闭,验证将失去意义.
