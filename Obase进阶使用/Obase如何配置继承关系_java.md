继承是面向对象的重要的重要特性,在实际使用中我们通常会在以下两种场景里使用继承:
1. 复用父类的属性和方法
2. 表示从一般到特殊的派生关系

对于第一种情况,这些类和派生类之间并没有逻辑关系,也往往不需要存储于同一张表内一起查询,此时只要按照普通的配置逻辑,按照不同的类配置即可.

对于第二种情况,此时这些类和派生类表示的同一个类型中抽象和具体的关系,那么就往往要存储于同一张表内,且需要在查询父类时同时查询到子类的对象.

对于如何区分这两种情况,这里提出一个判断标准:

**如果继承关系中的基类没有引用其他类或被其他类引用则不需要配置继承关系,否则都需要配置继承关系.**

## 模型内配置继承关系

继承配置是包含在Obase本体中的,因此不需要额外引用软件包,通常情况下,由于引用传递只需要引用对应的数据源提供器即可.

Obase使用具体类型区分器和具体类型标记这两个概念来处理继承关系,区分器需要实现IConcreteTypeDiscriminator接口,在接口实现中处理区分字段的哪个值对应哪个子类;具体类型标记则是一个元组,表示这个类对应哪个字段的哪个值.

以下的内容中,把有派生类的类称之为父类,有父类的类称之为子类,在模型配置中有以下几个方法用于配置对象的继承关系:
1. 配置继承关系方法deriveFrom,此方法仅子类需要调用,用于配置类的父类是哪个类.
2. 配置类的具体类型区分标志方法hasConcreteTypeSign,此方法所有的父类和子类都需要调用,用于配置这个类在表中对应哪个字段的哪个值.
3. 配置具体类型区分器的方法hasConcreteTypeDiscriminator,此方法用于配置父类如何区分子类,需要实现IConcreteTypeDiscriminator接口的实例.

## 实现具体类型区分器

具体类型区分器IConcreteTypeDiscriminator接口的实现方法内,需要返回结构化类型,此接口的实现逻辑一般为根据传入参数typeCode从模型中获取对应的结构化类型.

在6.5.0中,此接口增加了默认实现,因为具体类型区分器的实现逻辑相对固定而且涉及到了一些Obase的内部,所以在增加了默认实现后大部分情况下都不再需要自己实现具体类型区分器了.

## 查询派生类

假设A有派生类A1和A2,A1有派生类A3,那么在配置了继承关系后,查询A类型会查询出所有的A,A1,A2,A3的对象;查询A1类型,会查询出A1,A3对象;查询A2和A3则只会查询出A2和A3的对象.

## 具体示例

考虑一个例子,Bike表示自行车,BikeWheel表示车轮,BikeLight表示车灯.

而后有一个特殊的MyBikeA继承Bike,有一个额外的旗子BikeFlag.一个特殊的MyBikeB继承Bike,有一个额外的车筐BikeBucket. 一个特殊的MyBIkeC继承MyBikeA,是可以共享的.

类定义如下:
```
/**
 * 表示自行车
 */
public class Bike {

    /**
     * 自行车编码
     */
    private String code;

    /**
     * 自行车灯
     */
    private BikeLight light;

    /**
     * 车灯编码
     */
    private String lightCode;

    /**
     * 自行车名称
     */
    private String name;

    /**
     * 1-普通车 2-MyBikeA 3-MyBikeB
     */
    private int type;

    /**
     * 自行车轮
     */
    private List<BikeWheel> wheels;

    /**
     * 获取自行车编码
     *
     * @return 自行车编码
     */
    public String getCode() {
        return this.code;
    }

    /**
     * 设置自行车编码
     *
     * @param code 自行车编码
     */
    public void setCode(String code) {
        this.code = code;
    }

    /**
     * 获取自行车灯
     *
     * @return 自行车灯
     */
    public BikeLight getLight() {
        return this.light;
    }

    /**
     * 设置自行车灯
     *
     * @param light 自行车灯
     */
    public void setLight(BikeLight light) {
        this.light = light;
    }

    /**
     * 获取车灯编码
     *
     * @return 车灯编码
     */
    public String getLightCode() {
        return this.lightCode;
    }

    /**
     * 设置车灯编码
     *
     * @param lightCode 车灯编码
     */
    public void setLightCode(String lightCode) {
        this.lightCode = lightCode;
    }

    /**
     * 获取自行车名称
     *
     * @return 自行车名称
     */
    public String getName() {
        return this.name;
    }

    /**
     * 设置自行车名称
     *
     * @param name 自行车名称
     */
    public void setName(String name) {
        this.name = name;
    }

    /**
     * 获取类型
     * 1-普通车 2-MyBikeA 3-MyBikeB
     *
     * @return 类型
     */
    public int getType() {
        return this.type;
    }

    /**
     * 设置类型
     * 0-普通车 1-MyBikeA 2-MyBikeB
     *
     * @param type 类型
     */
    void setType(int type) {
        this.type = type;
    }

    /**
     * 获取自行车轮
     *
     * @return 自行车轮
     */
    public List<BikeWheel> getWheels() {
        return this.wheels;
    }

    /**
     * 设置自行车轮
     *
     * @param wheels 自行车轮
     */
    public void setWheels(List<BikeWheel> wheels) {
        this.wheels = wheels;
    }
}

/**
 * 车轮
 */
public class BikeWheel {

    /**
     * 车编码
     */
    private String bikeCode;

    /**
     * 车轮编码
     */
    private String code;

    /**
     * 获取车编码
     *
     * @return 车编码
     */
    public String getBikeCode() {
        return this.bikeCode;
    }

    /**
     * 设置车编码
     *
     * @param bikeCode 车编码
     */
    public void setBikeCode(String bikeCode) {
        this.bikeCode = bikeCode;
    }

    /**
     * 获取车轮编码
     *
     * @return 车轮编码
     */
    public String getCode() {
        return this.code;
    }

    /**
     * 设置车轮编码
     *
     * @param code 车轮编码
     */
    public void setCode(String code) {
        this.code = code;
    }
}

/**
 * 车灯
 */
public class BikeLight {
    /**
     * 车灯编码
     */
    private String code;

    /**
     * 亮度
     */
    private int value;

    /**
     * 获取车灯编码
     *
     * @return 车灯编码
     */
    public String getCode() {
        return this.code;
    }

    /**
     * 设置车灯编码
     *
     * @param code 车灯编码
     */
    public void setCode(String code) {
        this.code = code;
    }

    /**
     * 获取亮度
     *
     * @return 亮度
     */
    public int getValue() {
        return this.value;
    }

    /**
     * 设置亮度
     *
     * @param value 亮度
     */
    public void setValue(int value) {
        this.value = value;
    }
}

/**
 * 特殊的我的自行车A 有一个额外的旗子
 */
public class MyBikeA extends Bike {

    /**
     * 旗子
     */
    private BikeFlag flag;

    /**
     * 旗子编码
     */
    private String flagCode;

    /**
     * 获取旗子
     *
     * @return 旗子
     */
    public BikeFlag getFlag() {
        return this.flag;
    }

    /**
     * 设置旗子
     *
     * @param flag 旗子
     */
    public void setFlag(BikeFlag flag) {
        this.flag = flag;
    }


    /**
     * 获取旗子编码
     *
     * @return 旗子编码
     */
    public String getFlagCode() {
        return this.flagCode;
    }


    /**
     * 设置旗子编码
     *
     * @param flagCode 旗子编码
     */
    public void setFlagCode(String flagCode) {
        this.flagCode = flagCode;
    }
}

/**
 * 车旗子
 */
public class BikeFlag {

    /**
     * 车旗子编码
     */
    private String code;

    /**
     * 车旗子文字
     */
    private String value;

    /**
     * 获取车旗子编码
     *
     * @return 车旗子编码
     */
    public String getCode() {
        return this.code;
    }

    /**
     * 设置车旗子编码
     *
     * @param code 车旗子编码
     */
    public void setCode(String code) {
        this.code = code;
    }

    /**
     * 获取车旗子文字
     *
     * @return 车旗子文字
     */
    public String getValue() {
        return this.value;
    }

    /**
     * 设置车旗子文字
     *
     * @param value 车旗子文字
     */
    public void setValue(String value) {
        this.value = value;
    }
}

/**
 * 特殊的我的自行车B 有一个额外的旗子
 */
public class MyBikeB extends Bike {

    /**
     * 车筐
     */
    private BikeBucket bucket;

    /**
     * 车筐编码
     */
    private String bucketCode;

    /**
     * 获取车筐
     *
     * @return 车筐
     */
    public BikeBucket getBucket() {
        return this.bucket;
    }

    /**
     * 设置车筐
     *
     * @param bucket 车筐
     */
    public void setBucket(BikeBucket bucket) {
        this.bucket = bucket;
    }

    /**
     * 获取车筐编码
     *
     * @return 车筐编码
     */
    public String getBucketCode() {
        return this.bucketCode;
    }

    /**
     * 设置车筐编码
     *
     * @param bucketCode 车筐编码
     */
    public void setBucketCode(String bucketCode) {
        this.bucketCode = bucketCode;
    }
}

/**
 * 车筐
 */
public class BikeBucket {

    /**
     * 车筐编码
     */
    private String code;

    /**
     * 车筐大小
     */
    private String sp;

    /**
     * 获取车筐编码
     *
     * @return 车筐编码
     */
    public String getCode() {
        return this.code;
    }

    /**
     * 设置车筐编码
     *
     * @param code 车筐编码
     */
    public void setCode(String code) {
        this.code = code;
    }

    /**
     * 获取车筐大小
     *
     * @return 车筐大小
     */
    public String getSp() {
        return this.sp;
    }

    /**
     * 设置车筐大小
     *
     * @param sp 车筐大小
     */
    public void setSp(String sp) {
        this.sp = sp;
    }
}

/**
 * 特殊的我的自行车C 是可以共享的
 */
public class MyBikeC extends MyBikeA {

    /**
     * 是否可共享
     */
    private boolean canShared;

    /**
     * 获取是否可共享
     *
     * @return 是否可共享
     */
    public boolean getCanShared() {
        return this.canShared;
    }

    /**
     * 设置是否可共享
     *
     * @param canShared 是否可共享
     */
    public void setCanShared(boolean canShared) {
        this.canShared = canShared;
    }
}
```

这些类都存储于Bike表中,使用Type作为区分每个类型的字段.

**目前需要注意的是,子类的所独有的字段不能设置为非空的,因为在插入其他子类时会将这些字段设为空.**

Obase使用具体类型区分器和具体类型标记这两个概念来处理继承关系,区分器需要实现IConcreteTypeDiscriminator接口,在接口实现中处理区分字段(这里就是Type)的哪个值对应哪个子类;具体类型标记则是一个元组,表示这个类对应哪个字段的哪个值.

由于在6.5版本中对继承配置进行了优化,所以接下来的内容区分为6.4和6.5.

## 6.4版本配置

配置时要为父类配置具体类型判别器,父类和子类配置判别类型标识,子类配置派生自哪个类.
```
//定义一个自行车实体配置
EntityTypeConfiguration<Bike> bikeEntity = modelBuilder.entity(Bike.class);
bikeEntity.hasKeyAttribute(p -> p.getCode()).hasKeyIsSelfIncreased(false);
//此处需要配置类型判别器和根据哪个数据源字段的值来判断 不再需要配置自定义的构造器
//具体配置见下方的BikeConcreteTypeDiscriminator
bikeEntity.hasConcreteTypeDiscriminator(new BikeConcreteTypeDiscriminator(), "Type");
//Bike的Type字段是1 这里的值需要根据具体的类型进行调整 
//如果此基础类型是抽象的 此处可以配置一个如-1一类的值抽象的类型不会被创建 所以配置一个特殊值即可
//字段名需要与基类类型的HasConcreteTypeDiscriminator方法的第二个参数相同
bikeEntity.hasConcreteTypeSign("Type", 1);

//定义车灯实体配置
EntityTypeConfiguration<BikeLight> bikeLightEntity = modelBuilder.entity(BikeLight.class);
bikeLightEntity.hasKeyAttribute(p -> p.getCode()).hasKeyIsSelfIncreased(false);

//定义车轮实体配置
EntityTypeConfiguration<BikeWheel> bikeWheelEntity = modelBuilder.entity(BikeWheel.class);
bikeWheelEntity.hasKeyAttribute(p -> p.getCode()).hasKeyIsSelfIncreased(false);

//定义车旗实体配置
EntityTypeConfiguration<BikeFlag> bikeFlagEntity = modelBuilder.entity(BikeFlag.class);
bikeFlagEntity.hasKeyAttribute(p -> p.getCode()).hasKeyIsSelfIncreased(false);

//定义车筐实体配置
EntityTypeConfiguration<BikeBucket> bikeBucketEntity = modelBuilder.entity(BikeBucket.class);
bikeBucketEntity.hasKeyAttribute(p -> p.getCode()).hasKeyIsSelfIncreased(false);

//定义一个特定的我的自行车A
EntityTypeConfiguration<MyBikeA> myBikeAEntity = modelBuilder.entity(MyBikeA.class);
myBikeAEntity.hasKeyAttribute(p -> p.getCode()).hasKeyIsSelfIncreased(false);
//设置继承关系
myBikeAEntity.deriveFrom(Bike.class);
//MyBikeA的Type字段是2 这里的值需要根据具体的类型进行调整
//字段名需要与基类类型的HasConcreteTypeDiscriminator方法的第二个参数相同
myBikeAEntity.hasConcreteTypeSign("Type", 2);
//设置A和C的具体类型区分器
myBikeAEntity.hasConcreteTypeDiscriminator(new MyBikeConcreteTypeDiscriminator(), "Type");
//此处与父类一起保存于Bike
myBikeAEntity.toTable("Bike");

//定义一个特定的我的自行车B
EntityTypeConfiguration<MyBikeB> myBikeBEntity = modelBuilder.entity(MyBikeB.class);
myBikeBEntity.hasKeyAttribute(p -> p.getCode()).hasKeyIsSelfIncreased(false);
//设置继承关系
myBikeBEntity.deriveFrom(Bike.class);
//MyBikeB的Type字段是3 这里的值需要根据具体的类型进行调整
//字段名需要与基类类型的HasConcreteTypeDiscriminator方法的第二个参数相同
myBikeBEntity.hasConcreteTypeSign("Type", 3);
//此处与父类一起保存于Bike
myBikeBEntity.toTable("Bike");

//定义一个特定的我的自行车C
EntityTypeConfiguration<MyBikeC> myBikeCEntity = modelBuilder.entity(MyBikeC.class);
myBikeCEntity.hasKeyAttribute(p -> p.getCode()).hasKeyIsSelfIncreased(false);
//设置继承关系
myBikeCEntity.deriveFrom(MyBikeA.class);
//MyBikeB的Type字段是4 这里的值需要根据具体的类型进行调整
//字段名需要与基类类型的HasConcreteTypeDiscriminator方法的第二个参数相同
myBikeCEntity.hasConcreteTypeSign("Type", 4);
//此处与父类一起保存于Bike
myBikeCEntity.toTable("Bike");

//定义车灯的关联
AssociationConfiguratorBuilder bikeAssLight = modelBuilder.association();
//关联端 关联映射
AssociationEndConfigurationGeneric<Bike> bikeEnd1 = bikeAssLight.associationEnd(Bike.class);
//启用延迟加载
bikeEnd1.associationReference(p -> p.getLight()).hasEnableLazyLoading(true);
bikeEnd1.hasMapping("Code", "Code");
bikeAssLight.associationEnd(BikeLight.class).hasMapping("Code", "LightCode");
bikeAssLight.toTable("Bike");

//定义车轮的关联
AssociationConfiguratorBuilder bikeAssWheel = modelBuilder.association();
//关联端 关联映射
AssociationEndConfigurationGeneric<Bike> bikeEnd2 = bikeAssWheel.associationEnd(Bike.class);
//启用延迟加载
bikeEnd2.associationReference(p -> p.getWheels()).hasEnableLazyLoading(true);
bikeEnd2.hasMapping("Code", "BikeCode");
bikeAssWheel.associationEnd(BikeWheel.class).hasMapping("Code", "Code");
bikeAssWheel.toTable("BikeWheel");

//定义车旗的关联
AssociationConfiguratorBuilder mybikeAssFlag = modelBuilder.association();
//关联端 关联映射
AssociationEndConfigurationGeneric<MyBikeA> myBikeEnd1 = mybikeAssFlag.associationEnd(MyBikeA.class);
myBikeEnd1.associationReference(p -> p.getFlag()).hasEnableLazyLoading(true);
//启用延迟加载
myBikeEnd1.hasMapping("Code", "Code");
mybikeAssFlag.associationEnd(BikeFlag.class).hasMapping("Code", "FlagCode");
mybikeAssFlag.toTable("Bike");

//定义车筐的关联
AssociationConfiguratorBuilder mybikeAssBucket = modelBuilder.association();
//关联端 关联映射
AssociationEndConfigurationGeneric<MyBikeB> myBikeEnd2 = mybikeAssBucket.associationEnd(MyBikeB.class);
myBikeEnd2.associationReference(p -> p.getBucket()).hasEnableLazyLoading(true);
//启用延迟加载
myBikeEnd2.hasMapping("Code", "Code");
mybikeAssBucket.associationEnd(BikeBucket.class).hasMapping("Code", "BucketCode");
mybikeAssBucket.toTable("Bike");
```
具体类型选择器代码如下
```
/**
 * 自行车类型选择器
 */
public class BikeConcreteTypeDiscriminator implements IConcreteTypeDiscriminator {

    /**
     * 根据类型代码选择一个具体类型
     *
     * @param typeCode 类型代码
     * @return 具体的结构化类型
     */
    @Override
    public StructuralType discriminate(Object typeCode) {
        //这里的类型代码typeCode就是获取到的用于判别类型的值
        //这里我们规定1是Bike 2是MyBikeA 3是myBikeB 4是MyBikeC

        //从模型里取具体的类型 此处获取模型的参数是此配置属于的上下文类型
        StructuralType bikeType = GlobalModelCache.getInstance().getModel(SampleContext.class).getStructuralType(Bike.class);
        StructuralType myBikeAType = GlobalModelCache.getInstance().getModel(SampleContext.class).getStructuralType(MyBikeA.class);
        StructuralType myBikeBType = GlobalModelCache.getInstance().getModel(SampleContext.class).getStructuralType(MyBikeB.class);
        StructuralType myBikeCType = GlobalModelCache.getInstance().getModel(SampleContext.class).getStructuralType(MyBikeC.class);

        //处理参数
        if (typeCode == null)
            throw new IllegalArgumentException("未能获取类型判别参数.");

        if (Objects.equals(typeCode.toString(), "1"))
            return bikeType;

        if (Objects.equals(typeCode.toString(), "2"))
            return myBikeAType;

        if (Objects.equals(typeCode.toString(), "3"))
            return myBikeBType;

        if (Objects.equals(typeCode.toString(), "4"))
            return myBikeCType;

        throw new IllegalArgumentException("未知的类型判别参数值" + typeCode + ".");
    }
}

/**
 * 我的自行车类型选择器
 */
public class MyBikeConcreteTypeDiscriminator implements IConcreteTypeDiscriminator {

    /**
     * 根据类型代码选择一个具体类型
     *
     * @param typeCode 类型代码
     * @return 具体的结构化类型
     */
    @Override
    public StructuralType discriminate(Object typeCode) {
        //这里的类型代码typeCode就是获取到的用于判别类型的值
        //这里我们规定2是MyBikeA 4是MyBikeC

        //从模型里取具体的类型 此处获取模型的参数是此配置属于的上下文类型
        StructuralType myBikeAType = GlobalModelCache.getInstance().getModel(SampleContext.class).getStructuralType(MyBikeA.class);
        StructuralType myBikeCType = GlobalModelCache.getInstance().getModel(SampleContext.class).getStructuralType(MyBikeC.class);

        //处理参数
        if (typeCode == null)
            throw new IllegalArgumentException("未能获取类型判别参数.");

        if (Objects.equals(typeCode.toString(), "2"))
            return myBikeAType;

        if (Objects.equals(typeCode.toString(), "4"))
            return myBikeCType;

        throw new IllegalArgumentException("未知的类型判别参数值" + typeCode + ".");
    }
}

```
主要的配置思路为使用deriveFrom方法来配置当前类派生自那个类,使用hasConcreteTypeSign方法来配置这个类在表里对应哪个字段的哪个值,对有子类的类使用hasConcreteTypeDiscriminator方法配置具体类型区分器,以及实现IConcreteTypeDiscriminator接口.

## 6.5版本配置

在6.5.0版本中,配置具体类型判别器的方法进行了优化,加入了没有配置具体类型判别器时使用默认的具体类型判别器的逻辑,故这个例子的配置部分可以简化为如下的样子:

```
//定义一个自行车实体配置
EntityTypeConfiguration<Bike> bikeEntity = modelBuilder.entity(Bike.class);
bikeEntity.hasKeyAttribute(p -> p.getCode()).hasKeyIsSelfIncreased(false);
//此处需要配置类型判别器和根据哪个数据源字段的值来判断 不再需要配置自定义的构造器
//如果此处的具体类型判别器没有特殊逻辑 可以只传入判别字段名 使用Obase内置的判别器
bikeEntity.hasConcreteTypeDiscriminator("Type");
//Bike的Type字段是1 这里的类型需要根据具体的类型进行调整
//如果此基础类型是抽象的 此处可以配置一个如-1一类的值抽象的类型不会被创建 所以配置一个特殊值即可
bikeEntity.hasConcreteTypeSign(1);

//定义车灯实体配置
EntityTypeConfiguration<BikeLight> bikeLightEntity = modelBuilder.entity(BikeLight.class);
bikeLightEntity.hasKeyAttribute(p -> p.getCode()).hasKeyIsSelfIncreased(false);

//定义车轮实体配置
EntityTypeConfiguration<BikeWheel> bikeWheelEntity = modelBuilder.entity(BikeWheel.class);
bikeWheelEntity.hasKeyAttribute(p -> p.getCode()).hasKeyIsSelfIncreased(false);

//定义车旗实体配置
EntityTypeConfiguration<BikeFlag> bikeFlagEntity = modelBuilder.entity(BikeFlag.class);
bikeFlagEntity.hasKeyAttribute(p -> p.getCode()).hasKeyIsSelfIncreased(false);

//定义车筐实体配置
EntityTypeConfiguration<BikeBucket> bikeBucketEntity = modelBuilder.entity(BikeBucket.class);
bikeBucketEntity.hasKeyAttribute(p -> p.getCode()).hasKeyIsSelfIncreased(false);

//定义一个特定的我的自行车A
EntityTypeConfiguration<MyBikeA> myBikeAEntity = modelBuilder.entity(MyBikeA.class);
myBikeAEntity.hasKeyAttribute(p -> p.getCode()).hasKeyIsSelfIncreased(false);
//设置继承关系
myBikeAEntity.deriveFrom(Bike.class);
//MyBikeA的Type字段是2 这里的类型需要根据具体的类型进行调整
myBikeAEntity.hasConcreteTypeSign(2);
//设置A和C的具体类型区分器  使用Obase内置的判别器
myBikeAEntity.hasConcreteTypeDiscriminator("Type");
//此处与父类一起保存于Bike
myBikeAEntity.toTable("Bike");

//定义一个特定的我的自行车B
EntityTypeConfiguration<MyBikeB> myBikeBEntity = modelBuilder.entity(MyBikeB.class);
myBikeBEntity.hasKeyAttribute(p -> p.getCode()).hasKeyIsSelfIncreased(false);
//设置继承关系
myBikeBEntity.deriveFrom(Bike.class);
//MyBikeB的Type字段是3 这里的类型需要根据具体的类型进行调整
myBikeBEntity.hasConcreteTypeSign(3);
//此处与父类一起保存于Bike
myBikeBEntity.toTable("Bike");

//定义一个特定的我的自行车C
EntityTypeConfiguration<MyBikeC> myBikeCEntity = modelBuilder.entity(MyBikeC.class);
myBikeCEntity.hasKeyAttribute(p -> p.getCode()).hasKeyIsSelfIncreased(false);
//设置继承关系
myBikeCEntity.deriveFrom(MyBikeA.class);
//MyBikeB的Type字段是4 这里的类型需要根据具体的类型进行调整
myBikeCEntity.hasConcreteTypeSign(4);
//此处与父类一起保存于Bike
myBikeCEntity.toTable("Bike");

//定义车灯的关联
AssociationConfiguratorBuilder bikeAssLight = modelBuilder.association();
//关联端 关联映射
AssociationEndConfigurationGeneric<Bike> bikeEnd1 = bikeAssLight.associationEnd(Bike.class);
//启用延迟加载
bikeEnd1.associationReference(p -> p.getLight()).hasEnableLazyLoading(true);
bikeEnd1.hasMapping("Code", "Code");
bikeAssLight.associationEnd(BikeLight.class).hasMapping("Code", "LightCode");
bikeAssLight.toTable("Bike");

//定义车轮的关联
AssociationConfiguratorBuilder bikeAssWheel = modelBuilder.association();
//关联端 关联映射
AssociationEndConfigurationGeneric<Bike> bikeEnd2 = bikeAssWheel.associationEnd(Bike.class);
//启用延迟加载
bikeEnd2.associationReference(p -> p.getWheels()).hasEnableLazyLoading(true);
bikeEnd2.hasMapping("Code", "BikeCode");
bikeAssWheel.associationEnd(BikeWheel.class).hasMapping("Code", "Code");
bikeAssWheel.toTable("BikeWheel");

//定义车旗的关联
AssociationConfiguratorBuilder myBikeAssFlag = modelBuilder.association();
//关联端 关联映射
AssociationEndConfigurationGeneric<MyBikeA> myBikeEnd1 = myBikeAssFlag.associationEnd(MyBikeA.class);
myBikeEnd1.associationReference(p -> p.getFlag()).hasEnableLazyLoading(true);
//启用延迟加载
myBikeEnd1.hasMapping("Code", "Code");
myBikeAssFlag.associationEnd(BikeFlag.class).hasMapping("Code", "FlagCode");
myBikeAssFlag.toTable("Bike");

//定义车筐的关联
AssociationConfiguratorBuilder myBikeAssBucket = modelBuilder.association();
//关联端 关联映射
AssociationEndConfigurationGeneric<MyBikeB> myBikeEnd2 = myBikeAssBucket.associationEnd(MyBikeB.class);
myBikeEnd2.associationReference(p -> p.getBucket()).hasEnableLazyLoading(true);
//启用延迟加载
myBikeEnd2.hasMapping("Code", "Code");
myBikeAssBucket.associationEnd(BikeBucket.class).hasMapping("Code", "BucketCode");
myBikeAssBucket.toTable("Bike");
```
对于通常情况下,不再需要自行实现具体类型判别器了,接口仍然保留,如果有特殊的需求也可以自己实现,以上的例子中使用的是默认的判别器,所以原来的接口实现类都不再需要了.