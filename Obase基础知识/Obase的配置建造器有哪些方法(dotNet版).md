Obase使用对象数据模型用于解释领域模型,并根据对象数据模型提供对象变更侦测,属性映射,关系映射,关联导航等功能.故在使用Obase时需要将领域模型通过配置方法注册为对象数据模型

调用方需要在配置提供器的必须重写方法protected override void CreateModel(ModelBuilder modelBuilder)中配置Obase的对象数据模型,此方法的参数模型建造器提供了一系列的方法用于配置对象数据模型.

对象数据模型的配置可以分为两大类:结构化类型配置和类型元素配置.其中结构化类型配置是关于类和类间关系的配置,而类型元素配置则是类属性的配置,那么自然的,一个类型元素配置必然属于某个结构化配置.

结构化类型配置有三个实现类:实体型配置,关联型配置,复杂类型配置.类型元素配置有三个实现类:属性配置,关联引用配置,关联端配置.其中关联引用配置是实体型配置独有的,关联端配置是关联型配置独有的.

除结构化类型配置和类型元素配置外,还有一些杂项的配置,如构造函数的配置,并发策略配置等.

本篇主要介绍配置API

## 结构化类型配置

所有的结构化类型配置都是从重写的配置方法参数ModelBuilder发起的,首先介绍一组批量注册方法:


```
public void RegisterType(Assembly assembly);
public void RegisterType(params Type[] types);
public ModelBuilder Ignore<T>();
```

RegisterType将某个程序集注册的方法RegisterType,将某个程序集传入后会将所有符合推断的类按照推断注册至对象数据模型.

RegisterType的另外一个重载可以将指定的类型按照推断注册至对象数据模型.

Ignore是将某个类型添加在至忽略列表中,被忽略的类型不会参与RegisterType的注册,并且如果调用此方法时类型已注册,也会从已注册类型中删除.

### 共通配置

通过ModelBuilder的以下方法可以注册结构化类型:


```
public EntityTypeConfiguration<TEntity> Entity<TEntity>();
public AssociationTypeConfiguration<TAssociation> Association<TAssociation>();
public ComplexTypeConfiguration<TComplex> Complex<TComplex>();
```

Entity方法会返回一个实体型配置,每次调用时返回的均**是同一个对象.**

Association方法会返回一个显式关联型配置(此外还有隐式关联型配置,详见关联型配置部分),每次调用时返回的均是**同一个对象.**

Complex方法会返回一个复杂类型型配置,每次调用时返回的均**是同一个对象.**

这三种配置都具有以下几组配置方法:

- **属性配制方法**


```
public AttributeConfiguration<TStructural, TConfiguration> Attribute(string name, Type dataType);
public AttributeConfiguration<TStructural, TConfiguration> Attribute(string name);
public AttributeConfiguration<TStructural, TConfiguration> Attribute<TResult>(Expression<Func<TStructural, TResult>> expression);
public AttributeConfiguration<TStructural, TConfiguration> Attribute<TResult>(Expression<Func<TStructural, TResult>> expression,Type dataType);
```

这组属性配置方法都会返回一个属性配置(详见类型元素配置部分)每次调用时返回的均**是同一个对象.**

这些配置方法分别是(取值器,设值器,映射字段等详见类型元素配置部分):

1. 手动配置方法,指定属性的名称和映射类型配置属性,此方法仅会创建配置而不会配置取值器,设值器和映射字段.

2. 指定名称配制方法,指定属性的名称配置属性.会自动侦测属性的类型配置取值器,设值器和映射字段.

3. 使用表达式配置方法,会根据表达式所指向的属性访问器配置名称,属性的类型配置取值器,设值器和映射字段.

4. 使用表达式和指定映射类型的配置方法,会根据表达式所指向的属性访问器配置名称,指定的类型配置取值器,设值器和映射字段.

这里的映射类型指的是Obase基元类型,详见[Obase基元类型](./Obase基元类型与数据库类型映射.md)

*一般情况下使用较多的是第一个配置方法,因为大多数的属性都可以会被反射建模自动侦测并配置,只有在需要自定义取值器设值器映射属性的时候才会使用这些配置方法.*

- **构造函数配置方法**


```
public TConfiguration HasConstructor(Func<TStructural> construct);
public StructuralTypeConfiguration HasConstructor(IInstanceConstructor constructor);
public ParameterConfiguration<TStructural, TConfiguration> HasConstructor(ConstructorInfo constructorInfo);
```

这组方法中,前两个方法会直接返回结构化配置本身,第三个则返回的是构造函数参数配置,每次调用这些方法时,会**覆盖**之前的构造函数配置.

这些配置方法分别是:

1. 将无参的委托构造函数配置为当前结构化对象配置的构造方法的配制方法.

2. 使用IInstanceConstructor接口的实例作为当前结构化配置的构造方法的配制方法.

3. 使用反射获取的构造函数信息作为当前结构化配置的构造方法的配制方法.

*第一个方法适用于无参的构造情形,第二个则是由配置方实现构造逻辑的情形,实现IInstanceConstructor接口自行处理诸如参数等构造逻辑.第三种是使用已经在被配置的类内定义的反持久化构造函数来作为构造方法,需要在返回的ParameterConfiguration上调用Map函数指定每个参数都是对应哪个属性,最后再调用End函数结束配置.*

*一般情况下,如果类内有定义无参的构造函数,反射建模时会自动进行配置.如果没有无参的构造函数或要使用特定的构造函数进行反持久化,则需要配置此项*.

- **类型扩展配置方法**


```
public TExtensionConfiguration HasExtension<TExtensionConfiguration>() where TExtensionConfiguration : TypeExtensionConfiguration, new();
```

这个方法的泛型参数必须是TypeExtensionConfiguration的继承类且必须要有无参的构造函数.

这个方法会将TypeExtensionConfiguration的MakeExtension()方法返回的TypeExtension继承类存储于相应的结构化类型中,以供进行某些扩展性的处理.

可以在映射模块,配置管道等扩展性机制中获取这些TypeExtension类型扩展进行某些判断或者处理携带的信息等操作.

- **基类型配置方法**


```
public void DeriveFrom<TDerived>();
```

此方法用于配置此结构化类型的基类,基类也需要被配置为对象数据模型中的结构化类型.

当结构化类型被配置基类后,会获得基类所有的属性,关联引用这些类型元素的配置.

- **功能性配置方法**


```
public StructuralTypeConfiguration HasNewInstanceConstructor(ConstructorInfo constructorInfo);
public StructuralTypeConfiguration HasNewInstanceConstructor<T>(Func<T, TStructural> construct);
```

这组配置方法用于配置新实例对象,新实例对象是由上下文的Create<>方法创建的已附加至附加到上下文且可以进行延迟加载的对象.

此类对象可以用于按需加载等场景.

这些方法分别是使用反射获取的构造函数作为当前结构化配置的新实例构造方法的配制方法和使用委托作为当前结构化配置的新实例构造方法的配制方法.


```
public void Ignore<TResult>(Expression<Func<TStructural, TResult>> expression);
```

这个方法用于指示建模器在反射建模时忽略此属性,被忽略的属性不会参与反射建模和推断.

对于实体型配置和显式关联型配置还额外具有以下的配置方法:

- **对象通知配置方法**


```
public TConfiguration HasNoticeAttributes(List<string> noticeAttributes);
public TConfiguration HasNoticeAttributes();
public TConfiguration HasNotifyCreation(bool notifyCreation);
public TConfiguration HasNotifyDeletion(bool notifyDeletion);
public TConfiguration HasNotifyUpdate(bool notifyUpdate)
```

这组方法用来配置对象通知,对象通知会在对象被创建,修改,删除时通过上下文中重写的GetMessageQueue方法获取的IMessageQueue实例发送消息.

这些方法分别是:

1. 指示哪些属性和值要包含在通知中.

2. 指示将所有属性及值包含在通知中,注意此方法会覆盖有参方法设值的属性.

3. 指示是否在创建时发送消息.

4. 指示是否在删除时发送消息.

5. 指示是否在修改时发送消息.

- **并发策略配置方法**


```
public TConfiguration HasConcurrentConflictHandlingStrategy(eConcurrentConflictHandlingStrategy strategy);
```

并发策略适用于对象创建和修改时出现并发的情况.

Obase将并发冲突分为三种:重复创建,版本冲突,更新幻影.

重复创建,即尝试创建主键相同的对象.

版本冲突,在配置了版本键的情况下,修改对象时版本键已被其他线程/进程修改.

更新幻影,修改对象时对象已被其他线程/进程删除.

此方法用于配置发生这三种并发冲突时如何处理,参数值和相应的处理方式如下:

1. eConcurrentConflictHandlingStrategy.Ignore:忽略策略,当发生并发时,不做任何处理.

2. eConcurrentConflictHandlingStrategy.ThrowException:抛出异常策略,当发生并发异常,会抛出特定的异常.默认的处理策略即为抛出异常 故使用此种策略时可以不配置

3. eConcurrentConflictHandlingStrategy.Overwrite:强制覆盖策略 当发生并发时 用当前对象覆盖原有对象

4. eConcurrentConflictHandlingStrategy.Combine:版本合并策略 当发生异常时 将当前对象与旧对象的属性进行合并.如何进行合并详见类型元素配置中的属性配置.


```
public TConfiguration HasVersionAttribute<TAttribute>(Expression<Func<TObject, TAttribute>> expression);
```

此方法用于配置版本键,版本键用于检测修改时的并发冲突,要想处理版本冲突并发,必须配置版本键,版本键可以配置多个.

可以使用会发生并发冲突的属性,或者使用时间戳标识最后的修改时间来作为版本键,能区分对象最后被谁修改的属性都可以作为版本键.

- **杂项配置方法**


```
public TConfiguration ToTable(string table);
```

此方法用于配置该结构化类型的映射表.

- **继承相关配置方法(6.3+可用)**


```
HasConcreteTypeDiscriminator(IConcreteTypeDiscriminator concreteTypeDiscriminator, string typeAttributeName);
```

此方法用于 设置此类型的具体类型判别规范,用来用于判断此类型的要如何创建具体的类型.

第一个参数为类型判别器,需要配置方实现.第二个参数为数据源中用于判断的字段名.

判别器需要配置在所有需要将子类混合查出的类型上,如A有继承类B和C,B有继承类D,则需要A和B都配置相应的类型判别器.

A的判别器处理ABCD四种类型,B的判别器处理BD两种类型.


```
HasConcreteTypeSign(string typeName, object value);
```

此方法用于设置此类型的判别字段和判别字段的值.

第一个参数为数据源中用于判断的字段名,第二个参数为此类型的判别字段的值,与类型一一对应.如果此类型是抽象的,此值可以配置一个不会被使用的值.

继承配置的详情可见 obase#K-4如何配置继承关系的示例 中6.3新增的部分.

### 实体型配置

实体型配置是从ModelBuilder的Entity方法返回值得到的.

实体型配置有以下特定的配置方法:


```
public EntityTypeConfiguration<TEntity> HasKeyAttribute<TAttribute>(Expression<Func<TEntity, TAttribute>> expression);
```

此方法用于配置实体型的主键,主键是区分对象唯一性的标识,可以配置多个.


```
public EntityTypeConfiguration<TEntity> HasKeyIsSelfIncreased(bool keyIsSelfIncreased);
```

此方法用于配置实体型的主键是否自增,当主键由数据库生成时,需要将此项配置为true.

### 关联型配置

Obase将关联型分为两种,区别在于关联型类内是否定义了除关联端(详见类型元素关联端配置)以外的其他独属于关联的属性.存在则称为显式关联型,否则称为隐式关联型.

故关联型的配置也有两种,一种是显式关联型配置,一种是隐式关联型配置.

- **显式关联型**

显式关联型配置是从ModelBuilder的Association<TAssociation>()方法返回值获取的,泛型参数即要注册为显式关联型的类型,每次调用时返回的均**是同一个对象.**

显式关联型有以下特定的配置方法:


```
public AssociationTypeConfiguration<TAssociation> HasVisible(bool visible);
```

此方法用于设置关联型是显式还是隐式的,此方法的默认值为true.即使用Association<TAssociation>()方法获取的配置默认为显式关联配置.

一个关联型是否为显式关联会影响关联引用的设值和取值(关联引用详见类型元素配置中关联引用配置),一般而言显式关联型对应的关联引用的元素类型应该是显式关联型,而隐式关联型的关联引用应该是关联的另外一端.

当然,在某些情况下,显式关联型的关联引用也可以是关联的对端.不过一般而言不需要更改此选项的配置.


```
public AssociationEndConfiguration<TAssociation> AssociationEnd(string name, Type entityType);
public AssociationEndConfiguration<TAssociation,TResult> AssociationEnd<TResult>(Expression<Func<TAssociation, TResult>> expression) where TResult : class;
public AssociationEndConfiguration<TAssociation> AssociationEnd<TEnd>(string name) where TEnd : class;
```

这组方法用于配置关联端,每次调用时返回的均**是同一个对象.**

这些方法分别为:

1. 使用指定的名称和关联端类型创建关联端配置,此方法仅会创建配置而不会配置取值器,设值器.

2. 使用表达式创建关联端配置,此方法会创建配置且配置取值器,设值器.

3. 使用指定的名称创建关联端配置,此方法会创建配置且配置取值器,设值器.

- **隐式关联型**

隐式关联型配置是从ModelBuilder的Association()方法返回值获取的,每次调用返回的都**是一个新的配置对象.**

此方法返回的对象是AssociationConfiguratorBuilder隐式关联建造器,建造器没有之前共通配置部分的方法,而是有以下配制方法:


```
public AssociationEndConfiguration<TEntity> AssociationEnd<TEntity>() where TEntity : class;
public AssociationEndConfiguration AssociationEnd(Type endType);
```

这组方法用于配置隐式关联型的关联端,每次调用返回的都**是一个新的配置对象.**

隐式关联型会根据注册的关联端动态创建类型,至少需要两个关联端才可以创建,所以每个隐式关联建造器需要调用两次AssociationEnd方法来注册关联端.


```
public AssociationConfiguratorBuilder ToTable(string table);
```

此方法用于配置隐式关联的映射表.

### 复杂类型配置

实体型配置是从ModelBuilder的Complex方法返回值得到的.

复杂类型没有特定于复杂类型的配置.

## 类型元素配置

类型元素配置是使用结构化类型的相应的配置方法获取的,如结构化类型的Attribute方法获取的属性配置等.

### 共通配置

对于属性配置,关联端配置和关联引用配置,都具有以下的共通配置方法:

- **类型元素扩展配置方法**


```
public TExtensionConfiguration HasExtension<TExtensionConfiguration>() where TExtensionConfiguration : ElementExtensionConfiguration, new();
```

这个方法的泛型参数必须是ElementExtensionConfiguration的继承类且必须要有无参的构造函数.

这个方法会将ElementExtensionConfiguration的MakeExtension()方法返回的ElementExtension继承类存储于相应的类型元素类型中,以供进行某些扩展性的处理.

可以在映射模块,配置管道等扩展性机制中获取这些ElementExtension类型扩展进行某些判断或者处理携带的信息等操作.


```
public TConfiguration HasValueGetter(IValueGetter valueGetter);
public TConfiguration HasValueGetter<TProperty>(Func<TStructural, TProperty> getValue);
public TConfiguration HasValueGetter(MethodInfo method);
public TConfiguration HasValueGetter(PropertyInfo property);
public TConfiguration HasValueGetter(FieldInfo field);
```

这是一组设置类型元素取值器的方法,取值器的**取值指的是从对象中获取值**,在探测对象是否修改,关联是否建立等场景中,Obase会调用取值器来获取元素的值.

这些方法分别是:

1. 将IValueGetter接口的实现类设置为取值器.

2. 将调用传入委托作为取值方法的取值器设置为取值器.

3. 将调用传入方法作为取值方法的取值器设置为取值器.

4. 将调用传入属性访问器的Set方法作为取值方法的取值器设置为取值器.

5. 将调用传入字段反射取值方法作为取值方法的取值器设置为取值器.


```
public TConfiguration HasValueSetter(IValueSetter valueSetter);
public TConfiguration HasValueSetter<TValue>(Action<TStructural, TValue> setValue, eValueSettingMode mode);
public TConfiguration HasValueSetter(MethodInfo method, eValueSettingMode mode);
public TConfiguration HasValueSetter(PropertyInfo property);
public TConfiguration HasValueSetter(FieldInfo field);
```

这是一组设置类型元素设值器的方法,取值器的**设**值**指的是向对象中设置值**,在构造对象等场景中,Obase会调用设值器来为元素设置值.

这些方法分别是:

1. 将IValueSetter接口的实现类设置为设值器.

2. 将调用传入委托作为设值方法和指定的设值模式(模式包括追加和赋值两种)的设值器设置为设值器.

3. 将调用传入方法作为设值方法和指定的设值模式(模式包括追加和赋值两种)的设值器设置为设值器.

4. 将调用传入属性访问器的Set方法作为设值方法和赋值设值模式的设值器设置为设值器.

5. 将调用传入字段反射设值方作为设值方法和赋值设值模式的设值器设置为设值器.

### 属性配置

属性配置是从结构化类型的Attribute方法得到的,有以下特定的配制方法:


```
public AttributeConfiguration<TStructural> HasCombinationHandler(eAttributeCombinationHandlingStrategy strategy);
```

此方法用于在该结构化类型配置为eConcurrentConflictHandlingStrategy.Combine版本合并策略时,此属性的合并方式,共有三种:

1. eAttributeCombinationHandlingStrategy.Ignore:忽略 即使用旧对象(即原有对象)的值

2. eAttributeCombinationHandlingStrategy.Overwrite:覆盖 即使用新对象(即当前)的值 此种策略为默认的策略 可以不配置

3. eAttributeCombinationHandlingStrategy.Accumulate:累加 即将当前版本中属性值的增量累加到对方版本 只支持数值型的属性


```
public AttributeConfiguration<TStructural> HasMappingConnectionChar(char value);
```

此方法用于配置映射连接符,映射连接符是指示复杂类型是如何存储于映射表中的,如使用复杂类型的属性字段为A,复杂类型有两个属性B和C,那么在映射表中字段名称会被设置为"A映射连接符B"和"A映射连接符C".

默认的映射连接符为空字符.


```
public AttributeConfiguration<TStructural> HasMaxcharNumber(ushort maxcharNumber);
```

此方法用于配置属性的最大字符数,仅对映射类型为字符串类型的有效.会影响结构映射自动建表,如果设置为0,字段长度会被设置为255.如果超过255,会被设置为Text字段.

默认值为0;


```
public AttributeConfiguration<TStructural> HasPrecision(byte precision);
```

此方法用于配置属性的精度,参数表示小数点后的位数,最大可以为28,仅对映射类型decimal有效.会影响结构映射自动建表.

默认值为0.


```
public AttributeConfiguration<TStructural> HasNullable(bool value);
```

此方法用于配置属性是否可空,对于主键此配置是无效的,主键必然不为空.会影响结构映射自动建表.

默认值为true,可以为空.


```
public AttributeConfiguration<TStructural> ToField(string field);
```

此方法用于设置映射字段.

### 关联端配置

关联型上对于参与关联各端的引用称为关联端,关联端配置是从关联型配置的AssociationEnd方法得到的.

关联端配置有以下特定的配置方法:


```
public AssociationEndConfiguration AsCompanion(bool value);
```

设置此端是否为伴随端,设置当前端为伴随端会将之前设置的伴随端改设不作为伴随端.

伴随端会影响此关联的映射表,在未使用ToTable方法设置映射表时,会将伴随端的映射表设置为关联的映射表.

默认值为false,不是伴随端.


```
public AssociationEndConfiguration HasEnableLazyLoading(bool enableLazyLoading);
```

是否启用延迟加载,要想启用延迟加载,此关联端的定义必须是virtual的.在启用了延迟加载后,访问此端的属性访问器就会触发延迟加载,尝试从数据源加载对象.

默认为false,不启用延迟加载.


```
public AssociationEndConfiguration HasDefaultAsNew(bool defaultAsNew);
```

设置此关联端是否默认作为新对象附加至上下文.

关联端是否默认创建新对象配置控制如果某一个对象被创建出来后,未附加至上下文,但作为其他已附加对象的引用对象时,是否作为新对象附加至上下文.

默认为不作为,因为对象往往是由应用层创建的,是否需要附加由应用层决定即可.

但如果某个对象的关联是无法通过此对象外部进行创建 如只能在构造函数内一起创建时,外部无法获取这个被一起创建的对象进行附加操作,此时将此值设置为true.

默认值false,不默认作为新对象附加至上下文.


```
public AssociationEndConfiguration IsAggregated(bool isAggregated);
```

设置此关联端是否为聚合的.

设置为true表示聚合的端,对象在被聚合的关系解除或者关系的另外一端被删除是会被一并删除的.如A和B的关系,将B设置为聚合的,删除A时和解除A和B关系时(如把A中的B置空或者替换,如果是一对多的从集合中移除) B也会被删除.

默认值false,不是聚合的.


```
public AssociationEndConfiguration HasMapping(string keyAttribute, string targetField);
```

此方法用于设置关联端在关联表内主键的映射.

第一个参数固定为当前关联端的实体型主键中的一个,第二个参数为此主键在关联表内的字段名称.

如果实体型存在多个主键,可以多次调用,每次调用会添加一个主键的映射.


```
public AssociationReferenceConfiguration<TEntity> AssociationReference<TReferred>(string name, bool isMultiple);
public AssociationReferenceConfiguration<TEntity> AssociationReference<TAssociation>(Expression<Func<TEntity, TAssociation>> expression) where TAssociation : class;
```

这组方法用于配置此端实体型的关联引用,每次调用得到的都**是同一个对象.**

这组方法分别为:

1. 使用指定名称和多重性(一对一为false,一对多为true)来配置关联引用,会配置取值器和设置器.

2. 使用表达式来配置关联引用,此方法会创建配置,且会配置取值器和设置器.

### 关联引用配置

实体型上对于关联对象的引用称为关联引用,对于一般的显式关联,关联引用对象的元素类型是显式关联型;对于隐式关联,关联引用对象的元素类型是关联另外的端.关联引用配置是从关联端配置的AssociationReference方法得到的.

关联引用配置有以下特定的配置方法:


```
public AssociationEndConfiguration HasEnableLazyLoading(bool enableLazyLoading);
```

是否启用延迟加载,要想启用延迟加载,此关联端的定义必须是virtual的.在启用了延迟加载后,访问此端的属性访问器就会触发延迟加载,尝试从数据源加载对象.

需要注意的是,关联引用的延迟加载可能会导致性能损失,对一个由多个对象组成的集合,每次访问集合中的对象上配置了延迟加载的属性都会触发延迟加载,即进行了集合大小次的延迟加载.

默认为false,不启用延迟加载.

## 其他配置

#### 建模器方法

建模器ModelBuilder还有以下几个用于配置管道相关的配置方法.

配置管道是一种扩展机制,可以由调用方在管道中加入步骤来处理某些特定的逻辑.

Obase的配置管道分为四条,分别是类型解析管道,类型元素解析管道,代理类型创建管道和补充配置管道.

不同的配置管道会在建模过程中的不同步骤中被调用.

类型解析管道将在反射建模的最开始时被调用,分析已有的结构化配置并对结构化类型进行配置.

类型元素解析管道将在反射建模创建结构化配置之后被调用,对结构化类型的类型元素进行配置.

代理类型创建管将在反射建模配置结束后调用,对代理类型进行配置.

补充配置管道反射建模最后被调用,进行补充配置.


```
public TypeAnalyticPipelineBuilder Use(Func<ITypeAnalyzer, ITypeAnalyzer> middlewareDelegate);
public TypeMemberAnalyticPipelineBuilder Use(Func<ITypeMemberAnalyzer, ITypeMemberAnalyzer> middlewareDelegate);
public ProxyTypeGenerationPipelineBuilder Use(Func<IProxyTypeGenerator, IProxyTypeGenerator> middlewareDelegate);
public ComplementConfigurationPipelineBuilder Use(Func<IComplementConfigurator, IComplementConfigurator> middlewareDelegate);
```

这组方法即用于配置管道的方法,每个都接受一个委托,此委托用于创建所需的解析器.

每次调用这些方法都会在管道中新增一个子步骤,每个管道的结构为默认解析器->第一次调用加入的解析器->第二个......->最后加入的.