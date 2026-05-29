在文章[Obase如何配置属性的序列化器](./Obase如何配置属性简单序列化_java.md)中,我们介绍了如何配置属性的序列化器,解决了在持久化/反持久化过程中有些属性需要进行序列化的问题.

但仅对属性配置序列化器时,如果遇到较为复杂的情况(如某些对象属性要特殊存储,对象间存在互相引用等)时,就需要定义一个继承自TextSerializer的特定序列化器,并且在实现的方法中针对不同的类型书写具体的序列化方案.

针对这种情况,Obase在6.5.0中新增了一个新的序列化对象模型(Serialization Object Data Model,之后简称为SOdm)作为之前一直使用的对象数据模型(Object Data Model,之后简称为Odm)的新组成部分,SOdm用于指导Obase在持久化/反持久化如何对持久化/反持久化对象的属性进行序列化.

SOdm使用特定的数据传输对象(Data Transfer Object,以后简称为DTO)来存储要序列化的对象,同时为要序列化的对象定义属性,构造函数,引用三种配置来指导如何序列化和反序列化.为了解决循环引用的问题,SOdm会在序列化过程中为每个对象分配唯一的临时ID,并存储在DTO中的引用字段里,同时在反序列化时用具体对象替换.

## 持久化模型配置

持久化模型主要的配置方法为模型建造器上的serializationEntity方法,此方法会返回序列化实体配置,此序列化实体配置上有以下方法:
```
//配置序列化实体的属性方法
entityConfiguration.attribute(p -> p.getRole());
//配置序列化实体的构造器方法
entityConfiguration.hasConstructor(constructor).hasParameter(p -> p.getRole(), String.class, true);
//配置序列化实体的引用方法
entityConfiguration.reference(p -> p.getRef());
//忽略某个属性
entityConfiguration.ignore("Attr");
```
这里的配置需要注意的是,一个序列化对象的配置分为三种配置:属性,引用和构造函数.

其中属性是一定需要存储于DTO中的,引用则是一定不需要存储于DTO中的(只会存储序列化过程中分配的唯一临时ID),而构造函数的参数则是可由配置者自行配置是否需要存储于DTO的.

对于需要存储于DTO的属性或者构造函数参数,则只支持将Obase基元类型进行存储;不需要存储的构造函数参数,则是构造对象时用来注入某些外部对象用的.

和Odm相似,对于大部分的属性和引用Obase都可以自动侦测并进行配置,实际上需要配置的只有反序列化的构造函数其他的大多可以交给Obase进行自动配置.

如果有某些属性不需要被序列化,可以使用ignore方法进行忽略.

如果在序列化过程中,碰到没有配置的对象会跳过此对象,在反序列化时自然也就没有相应的对象被构造了.

此外,在属性配置上还需要配置使用序列化模型,方法如下:
```
//启用序列化模型
attrConfiguration.useSerializationModel(true);
```

## 实现序列化器

此处内容与文章[Obase如何配置属性的序列化器](./Obase如何配置属性简单序列化_java.md)中相同.

## 具体示例

假设我们有一个Service服务类,上面引用了Identity身份和Component组件类,这两个都需要序列化为Json字符串进行存储.

以下是Service的定义:

```
/**
 * 模拟某种服务
 */
public class Service {

    /**
     * 主键
     */
    private String code;

    /**
     * 身份
     */
    private Identity identity;

    /**
     * 组件
     */
    private List<Component> components;

    /**
     * 获取主键
     *
     * @return 主键
     */
    public String getCode() {
        return this.code;
    }

    /**
     * 设置主键
     *
     * @param code 主键
     */
    public void setCode(String code) {
        this.code = code;
    }

    /**
     * 获取身份
     *
     * @return 身份
     */
    public Identity getIdentity() {
        return this.identity;
    }

    /**
     * 设置身份
     *
     * @param identity 身份
     */
    public void setIdentity(Identity identity) {
        this.identity = identity;
    }

    /**
     * 获取组件
     *
     * @return 组件
     */
    public List<Component> getComponents() {
        return this.components;
    }

    /**
     * 设置组件
     *
     * @param components 组件
     */
    public void setComponents(List<Component> components) {
        this.components = components;
    }
}
```
以下是Identity的定义:
```
/**
 * 某种身份
 */
public class Identity {

    /**
     * 创建时间
     */
    private LocalDateTime createTime;

    /**
     * 身份标识
     */
    private UUID id;

    /**
     * 查询时间
     */
    private LocalDateTime queryTime;

    /**
     * 角色
     */
    private String role;

    /**
     * 版本
     */
    private int version;


    /**
     * 初始化某种身份
     *
     * @param id         身份标识
     * @param createTime 创建时间
     * @param role       角色
     */
    public Identity(UUID id, LocalDateTime createTime, String role) {
        this.id = id;
        this.createTime = createTime;
        this.role = role;
        this.queryTime = LocalDateTime.now();
    }

    /**
     * 反序列化函数
     *
     * @param id         身份标识
     * @param createTime 创建时间
     * @param role       角色
     * @param queryTime  查询时间
     */
    protected Identity(UUID id, LocalDateTime createTime, String role, LocalDateTime queryTime) {
        this.id = id;
        this.createTime = createTime;
        this.role = role;
        this.queryTime = queryTime;
    }

    /**
     * 获取创建时间
     *
     * @return 创建时间
     */
    public LocalDateTime getCreateTime() {
        return this.createTime;
    }

    /**
     * 设置创建时间
     *
     * @param createTime 创建时间
     */
    void setCreateTime(LocalDateTime createTime) {
        this.createTime = createTime;
    }

    /**
     * 获取身份标识
     *
     * @return 身份标识
     */
    public UUID getId() {
        return this.id;
    }

    /**
     * 设置身份标识
     *
     * @param id 身份标识
     */
    void setId(UUID id) {
        this.id = id;
    }

    /**
     * 获取查询时间
     *
     * @return 查询时间
     */
    public LocalDateTime getQueryTime() {
        return this.queryTime;
    }

    /**
     * 设置查询时间
     *
     * @param queryTime 查询时间
     */
    void setQueryTime(LocalDateTime queryTime) {
        this.queryTime = queryTime;
    }

    /**
     * 获取角色
     *
     * @return 角色
     */
    public String getRole() {
        return this.role;
    }

    /**
     * 设置角色
     *
     * @param role 角色
     */
    void setRole(String role) {
        this.role = role;
    }

    /**
     * 获取版本
     *
     * @return 版本
     */
    public int getVersion() {
        return this.version;
    }

    /**
     * 设置版本
     *
     * @param version 版本
     */
    public void setVersion(int version) {
        this.version = version;
    }

    /**
     * 转换为字符串表示形式
     *
     * @return 字符串表示形式
     */
    @Override
    public String toString() {
        return "Identity{" +
                "createTime=" + this.createTime +
                ", id=" + this.id +
                ", queryTime=" + this.queryTime +
                ", role='" + this.role + '\'' +
                '}';
    }
}
```
以下是Component的定义
```
/**
 * 组件类
 */
public class Component{

    /**
     * 组件名称
     */
    private String name;


    /**
     * 组件
     */
    private Component[] components;

    /**
     * 组件名称
     *
     * @return 组件名称
     */
    public String getName() {
        return this.name;
    }

    /**
     * 组件名称
     *
     * @param name 组件名称
     */
    public void setName(String name) {
        this.name = name;
    }

    /**
     * 组件
     *
     * @return 组件
     */
    public Component[] getComponents() {
        return this.components;
    }

    /**
     * 组件
     *
     * @param components 组件
     */
    public void setComponents(Component[] components) {
        this.components = components;
    }

    /**
     * 转换为字符串表示形式
     *
     * @return 字符串表示形式
     */
    @Override
    public String toString() {
        return "Component{" +
                "name='" + this.name + '\'' +
                '}';
    }
}

```
那么我们需要做如下的配置:
```
//配置序列化模型Identity
var idEntityConfiguration = modelBuilder.serializationEntity(Identity.class);
//Identity的属性 无需配置 自动侦测
//为了测试配置 故写一个属性配置
idEntityConfiguration.attribute("Role");
//Identity的构造函数需要配置
Constructor<Identity> identityConstructor;
try {
    identityConstructor = Identity.class.getDeclaredConstructor(UUID.class, LocalDateTime.class, String.class, LocalDateTime.class);
} catch (NoSuchMethodException e) {
    throw new RuntimeException("无法获取Identity的构造字段", e);
}
Field createTimeField;
try {
    createTimeField = Identity.class.getDeclaredField("createTime");
} catch (NoSuchFieldException e) {
    throw new RuntimeException("无法获取Identity的createTime字段", e);
}
var idConstructor = idEntityConfiguration.hasConstructor(identityConstructor);
//配置第一个参数 需要存储 从ID里取出Id属性的值存储
idConstructor.hasParameter((Identity p) -> p.getId(), UUID.class, true)
        //配置第二个参数 需要存储 从字段_createTime里取出CreateTime属性的值存储
        .hasParameter(createTimeField, LocalDateTime.class, true)
        //配置第三个参数 需要存储 从Role里取出Role属性的值存储
        .hasParameter((Identity p) -> p.getRole(), String.class, true)
        //配置第四个参数 不需要存储 直接传入当前时间 注意这个委托的参数会传空
        .hasParameter((Identity p) -> LocalDateTime.now(), LocalDateTime.class, false);

//Identity没有引用 无需配置
//忽略版本和次版本
idEntityConfiguration.ignore("Version");

//配置序列化模型Component
modelBuilder.serializationEntity(Component.class);
//Component的属性 无需配置 自动侦测
//Component有无参的反序列化构造函数 无需配置 自动侦测
//Component有引用 无需配置 自动侦测
```
最后,需要为Service进行配置,并在相应的属性配置上启用复杂的序列化,代码如下:
```
//注册服务为实体型
EntityTypeConfiguration<Service> serviceEntity = modelBuilder.entity(Service.class);
serviceEntity.hasKeyAttribute(p -> p.getCode()).hasKeyIsSelfIncreased(false);
serviceEntity.toTable("Service");

//设置Identity属性 长度设置的长一些 保证存储的下
serviceEntity.attribute(p -> p.getIdentity(), String.class).hasMaxCharNumber(1000)
        //需要有模型的序列化和设置序列化器
        .useSerializer(new JsonSerializer(), Identity.class).useSerializationModel(true);
     
//设置Components属性 长度设置的长一些 保证存储的下
serviceEntity.attribute("Components", String.class).hasValueGetter((Service p) -> p.getComponents())
        .hasValueSetter((Service p, List<Component> value) -> p.setComponents(value))
        .hasMaxCharNumber(1000)
        //需要有模型的序列化和设置序列化器
        .useSerializer(new JsonSerializer(), List.class).useSerializationModel(true);

```
这里的配置和之前文章[Obase如何配置属性的序列化器](./Obase如何配置属性简单序列化_java.md)中配置类似,只是新增了useSerializationModel这个配置. JsonSerializer的定义也是在文章[Obase如何配置属性的序列化器](./Obase如何配置属性简单序列化_java.md)中给出的.

当然,如果没有为某个属性启用useSerializationModel(true),依然是使用之前文章[Obase如何配置属性的序列化器](./Obase如何配置属性简单序列化_java.md)中的逻辑,直接将原始对象传入由用户定义的序列化器中,启用则是将特定的DTO对象传入.
