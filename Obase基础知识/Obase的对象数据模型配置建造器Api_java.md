Obase使用对象数据模型用于解释领域模型,并根据对象数据模型提供对象变更侦测,属性映射,关系映射,关联导航等功能.故在使用Obase时需要将领域模型通过配置方法注册为对象数据模型

调用方需要在配置提供器的必须重写方法protected void createModel(ModelBuilder modelBuilder)中配置Obase的对象数据模型,此方法的参数模型建造器提供了一系列的方法用于配置对象数据模型.

对象数据模型的配置可以分为两大类:结构化类型配置,类型元素和序列化实体类型配置.其中结构化类型配置是关于类和类间关系的配置,而类型元素配置则是类属性的配置,那么自然的,一个类型元素配置必然属于某个结构化配置.至于序列化实体配置,是用于指导如何将某个属性序列化后进行存储的.

结构化类型配置有三个实现类:实体型配置,关联型配置,复杂类型配置.类型元素配置有三个实现类:属性配置,关联引用配置,关联端配置.其中关联引用配置是实体型配置独有的,关联端配置是关联型配置独有的.

除结构化类型配置,类型元素配置和序列化实体外,还有一些杂项的配置,如配置建模管道,配置是否进行完整性检查等.

以下介绍配置API:

## 结构化类型配置

所有的结构化类型配置都是从重写的配置方法参数ModelBuilder发起的,首先介绍一组批量注册方法:


```
public void registerType(String assembly);
public void registerType(String assembly, IAssemblyAnalyzer analyzer);
public void registerType(Class<?>... types);
public void registerType(IAssemblyAnalyzer analyzer, Class<?>... types);
public ModelBuilder ignore(Class<?> ignored);
```

registerType将某个程序集注册的方法,将某个程序集(即软件包全名)传入后会将所有符合推断的类按照推断注册至对象数据模型.

registerType的另外一个重载可以将指定的类型按照推断注册至对象数据模型.

带IAssemblyAnalyzer的重载可以使用自定义的程序集解析器来决定程序集中哪些类型参与注册,不传入时使用Obase默认的程序集解析器.

ignore是将某个类型添加在至忽略列表中,被忽略的类型不会参与registerType的注册,并且如果调用此方法时类型已注册,也会从已注册类型中删除.

### 共通配置

通过ModelBuilder的以下方法可以注册结构化类型:


```
public <TEntity> EntityTypeConfiguration<TEntity> entity(Class<TEntity> entityClass);
public <TAssociation> AssociationTypeConfiguration<TAssociation> association(Class<TAssociation> associationType);
public <TComplex> ComplexTypeConfiguration<TComplex> complex(Class<TComplex> complexClass);
```

entity方法会返回一个实体型配置,每次调用时返回的均**是同一个对象.**

association方法会返回一个显式关联型配置(此外还有隐式关联型配置,详见关联型配置部分),每次调用时返回的均是**同一个对象.**

complex方法会返回一个复杂类型型配置,每次调用时返回的均**是同一个对象.**

这三种配置都具有以下几组配置方法:

- **属性配制方法**


```
public AttributeConfigurationGeneric<TStructural, TConfiguration> attribute(String name, Class<?> dataType);
public AttributeConfigurationGeneric<TStructural, TConfiguration> attribute(String name);
public <TResult> AttributeConfigurationGeneric<TStructural, TConfiguration> attribute(SerializedFunction<TStructural, TResult> get);
public <TResult> AttributeConfigurationGeneric<TStructural, TConfiguration> attribute(SerializedFunction<TStructural, TResult> get, Class<?> dataType);
```

这组属性配置方法都会返回一个属性配置(详见类型元素配置部分)每次调用时返回的均**是同一个对象.**

这些配置方法分别是(取值器,设值器,映射字段等详见类型元素配置部分):

1. 手动配置方法,指定属性的名称和映射类型配置属性,此方法仅会创建配置而不会配置取值器,设值器和映射字段.

2. 指定名称配制方法,指定属性的名称配置属性.会自动侦测属性的类型配置取值器,设值器和映射字段.

3. 使用表达式配置方法,会根据表达式所指向的属性访问器配置名称,属性的类型配置取值器,设值器和映射字段.

4. 使用表达式和指定映射类型的配置方法,会根据表达式所指向的属性访问器配置名称,指定的类型配置取值器,设值器和映射字段.

这里的映射类型指的是Obase基元类型,详见[Obase基元类型](./Obase的基元类型与默认数据库类型映射_java.md)

*一般情况下使用较多的是后两个配置方法,因为大多数的属性都可以会被反射建模自动侦测并配置,只有在需要自定义取值器设值器映射属性的时候才会使用这些配置方法.*

- **构造函数配置方法**


```
public TConfiguration hasConstructor(FunctionWithNoArg<TStructural> construct);
public StructuralTypeConfiguration<TStructural> hasConstructor(IInstanceConstructor constructor);
public <T> ParameterConfiguration<TStructural, TConfiguration> hasConstructor(FunctionWithOneArg<T, TStructural> construct, Class<T> t1Class);
public <T1, T2> ParameterConfiguration<TStructural, TConfiguration> hasConstructor(FunctionWithTwoArgs<T1, T2, TStructural> construct, Class<T1> t1Class, Class<T2> t2Class);
public <T1, T2, T3> ParameterConfiguration<TStructural, TConfiguration> hasConstructor(FunctionWithThreeArgs<T1, T2, T3, TStructural> construct, Class<T1> t1Class, Class<T2> t2Class, Class<T3> t3Class);
public <T1, T2, T3, T4> ParameterConfiguration<TStructural, TConfiguration> hasConstructor(FunctionWithFourArgs<T1, T2, T3, T4, TStructural> construct, Class<T1> t1Class, Class<T2> t2Class, Class<T3> t3Class, Class<T4> t4Class);
public ParameterConfiguration<TStructural, TConfiguration> hasConstructor(Constructor<TStructural> constructorInfo);
```

这组方法中,前两个方法会直接返回结构化配置本身,其余方法则返回的是构造函数参数配置,每次调用这些方法时,会**覆盖**之前的构造函数配置.

与hasConstructor对应的还有一组hasNewInstanceConstructor方法,参数形式相同,用于配置创建新实例(即由Obase内部创建对象时)所使用的构造函数.

这些配置方法分别是:

1. 将无参的委托构造函数配置为当前结构化对象配置的构造方法的配制方法.

2. 使用IInstanceConstructor接口的实例作为当前结构化配置的构造方法的配制方法.

3. 使用带1个到4个参数的委托配置构造方法,委托的参数即构造函数的参数,由于Java的类型擦除,还需要在委托之后依次传入各参数的Class,配置完成后需要在返回的ParameterConfiguration上指定每个参数对应哪个元素.

4. 使用反射获取的构造函数信息作为当前结构化配置的构造方法的配制方法.

*第一个方法适用于无参的构造情形,第二个则是由配置方实现构造逻辑的情形,实现IInstanceConstructor接口自行处理诸如参数等构造逻辑.第三种是使用带参数的委托配置构造方法,第四种是使用已经在被配置的类内定义的反持久化构造函数来作为构造方法,这两种都需要在返回的ParameterConfiguration上调用map函数指定每个参数都是对应哪个属性,最后再调用end函数结束配置.*

*一般情况下,如果类内有定义无参的构造函数,反射建模时会自动进行配置.如果没有无参的构造函数或要使用特定的构造函数进行反持久化,则需要配置此项*.

- **类型扩展配置方法**


```
public <TExtensionConfiguration extends TypeExtensionConfiguration> TExtensionConfiguration hasExtension(Class<TExtensionConfiguration> extensionConfigurationType) ;
```

这个方法的泛型参数必须是TypeExtensionConfiguration的继承类且必须要有无参的构造函数.

这个方法会将TypeExtensionConfiguration的makeExtension()方法返回的TypeExtension继承类存储于相应的结构化类型中,以供进行某些扩展性的处理.

可以在映射模块,配置管道等扩展性机制中获取这些TypeExtension类型扩展进行某些判断或者处理携带的信息等操作.

- **基类型配置方法**


```
public TConfiguration deriveFrom(Class<?> type);
```

此方法用于配置此结构化类型的基类,基类也需要被配置为对象数据模型中的结构化类型.

当结构化类型被配置基类后,会获得基类所有的属性,关联引用这些类型元素的配置.

- **功能性配置方法**


```
public StructuralTypeConfiguration<TStructural> hasNewInstanceConstructor(Constructor<TStructural> constructorInfo);
public <T> StructuralTypeConfiguration<TStructural> hasNewInstanceConstructor(FunctionWithOneArg<T, TStructural> construct, Class<T> t1Class);
```

这组配置方法用于配置新实例对象,新实例对象是由上下文的Create方法创建的已附加至附加到上下文且可以进行延迟加载的对象.

此类对象可以用于按需加载等场景.

这些方法分别是使用反射获取的构造函数作为当前结构化配置的新实例构造方法的配制方法和使用委托作为当前结构化配置的新实例构造方法的配制方法.


```
public <TResult> void ignore(SerializedFunction<TStructural, TResult> get);
```

这个方法用于指示建模器在反射建模时忽略此属性,被忽略的属性不会参与反射建模和推断.

对于实体型配置和显式关联型配置还额外具有以下的配置方法:

- **对象通知配置方法**


```
public TConfiguration hasNoticeAttributes(List<String> noticeAttributes);
public TConfiguration hasNoticeAttributes();
public TConfiguration hasNotifyCreation(boolean notifyCreation);
public TConfiguration hasNotifyDeletion(boolean notifyDeletion);
public TConfiguration hasNotifyUpdate(boolean notifyUpdate)
```

这组方法用来配置对象通知,对象通知会在对象被创建,修改,删除时通过依赖注入获取的IChangeNoticeSender实例发送消息.

其中hasNotifyCreation,hasNotifyDeletion,hasNotifyUpdate三个开关的默认值均为false,即默认不发送通知.

这些方法分别是:

1. 指示哪些属性和值要包含在通知中.

2. 指示将所有属性及值包含在通知中,注意此方法会覆盖有参方法设值的属性.

3. 指示是否在创建时发送消息.

4. 指示是否在删除时发送消息.

5. 指示是否在修改时发送消息.

- **并发策略配置方法**


```
public TConfiguration hasConcurrentConflictHandlingStrategy(EConcurrentConflictHandlingStrategy strategy);
```

并发策略适用于对象创建和修改时出现并发的情况.

Obase将并发冲突分为三种:重复创建,版本冲突,更新幻影.

重复创建,即尝试创建主键相同的对象.

版本冲突,在配置了版本键的情况下,修改对象时版本键已被其他线程/进程修改.

更新幻影,修改对象时对象已被其他线程/进程删除.

此方法用于配置发生这三种并发冲突时如何处理,参数值和相应的处理方式如下:

1. EConcurrentConflictHandlingStrategy.Ignore:忽略策略,当发生并发时,不做任何处理.

2. EConcurrentConflictHandlingStrategy.ThrowException:抛出异常策略,当发生并发异常,会抛出特定的异常.默认的处理策略即为抛出异常 故使用此种策略时可以不配置

3. EConcurrentConflictHandlingStrategy.Overwrite:强制覆盖策略 当发生并发时 用当前对象覆盖原有对象

4. EConcurrentConflictHandlingStrategy.Combine:版本合并策略 当发生异常时 将当前对象与旧对象的属性进行合并.如何进行合并详见类型元素配置中的属性配置.

5. EConcurrentConflictHandlingStrategy.Reconstruct:重建对象策略 当发生更新幻影冲突时 将对象作为新对象重新写入数据源以重建此对象.注意此策略不能用于重复创建和版本冲突,遇到这两种冲突时会抛出异常.


```
public <TResult> TConfiguration hasVersionAttribute(SerializedFunction<TObject, TResult> get);
```

此方法用于配置版本键,版本键用于检测修改时的并发冲突,要想处理版本冲突并发,必须配置版本键,版本键可以配置多个.

可以使用会发生并发冲突的属性,或者使用时间戳标识最后的修改时间来作为版本键,能区分对象最后被谁修改的属性都可以作为版本键.

- **杂项配置方法**


```
public TConfiguration toTable(String table);
```

此方法用于配置该结构化类型的映射表.

- **继承相关配置方法**


```
public StructuralTypeConfiguration<TStructural> hasConcreteTypeDiscriminator(String typeAttributeName)
public StructuralTypeConfiguration<TStructural> hasConcreteTypeDiscriminator(String typeAttributeName, IConcreteTypeDiscriminator concreteTypeDiscriminator);
```

此方法用于 设置此类型的具体类型判别规范,用来用于判断此类型的要如何创建具体的类型.

第一个参数为数据源中用于判断的字段名.第二个参数为类型判别器,需要配置方实现,如果没有特殊的逻辑仅为哪个值对应哪个类型,则不需要配置使用默认的内部判别器即可(使用第一个方法).

判别器需要配置在所有需要将子类混合查出的类型上,如A有继承类B和C,B有继承类D,则需要A和B都配置相应的类型判别器.

A的判别器处理A/B/C/D四种类型,B的判别器处理B/D两种类型.


```
public StructuralTypeConfiguration<TStructural> hasConcreteTypeSign(Object value);
```

此方法用于设置此类型的判别字段和判别字段的值.

参数为此类型的判别字段的值,与类型一一对应.如果此类型是抽象的,此值可以配置一个不会被使用的值.

此值不能为空,且类型必须是String,Integer,int,long中的一种(即String/Integer/Long,Java版不接受short),否则会在配置时抛出异常.此外基类上必须已经使用hasConcreteTypeDiscriminator配置过判别字段,否则也会抛出异常.此方法配置的判别字段名称会沿用基类上配置的名称,不需要重复指定.

### 实体型配置

实体型配置是从ModelBuilder的entity方法返回值得到的.

实体型配置有以下特定的配置方法:


```
public EntityTypeConfiguration<TEntity> hasKeyAttribute(String attrName);
public <TAttribute> EntityTypeConfiguration<TEntity> hasKeyAttribute(SerializedFunction<TEntity, TAttribute> get);
```

此方法用于配置实体型的主键,主键是区分对象唯一性的标识,可以配置多个.

每调用一次本方法会追加一个标识属性,重复配置同一属性只保留一次.按名称配置时会先按名称(忽略大小写)解析属性,名称无法解析为属性时会立即抛出异常;按表达式配置时,表达式必须能解析为get方法的成员访问表达式,否则会抛出异常.


```
public EntityTypeConfiguration<TEntity> hasKeyIsSelfIncreased(boolean keyIsSelfIncreased);
```

此方法用于配置实体型的主键是否自增,当主键由数据库生成时,需要将此项配置为true.

自增有以下硬性约束,违反时会在完整性检查时报错:

1. 联合主键不能配置为自增(实体型的主键多于一个时配置自增会报"主键是联合主键,不能配置为自增.").

2. 自增主键的类型必须是short,int,long之一,基本类型与包装类型(Short/Integer/Long)都接受.

3. 主键必须有取值器;如果没有设值器,则必须出现在构造函数的参数中.

此外,如果只配置了一个主键且没有显式调用过hasKeyIsSelfIncreased,当主键类型为int或long(基本类型)时,Obase会自动将其推断为自增.

### 关联型配置

Obase将关联型分为两种,区别在于关联型类内是否定义了除关联端(详见类型元素关联端配置)以外的其他独属于关联的属性.存在则称为显式关联型,否则称为隐式关联型.

故关联型的配置也有两种,一种是显式关联型配置,一种是隐式关联型配置.

- **显式关联型**

显式关联型配置是从ModelBuilder的association(Class<TAssociation>)方法返回值获取的,参数即要注册为显式关联型的类型,每次调用时返回的均**是同一个对象.**

显式关联型有以下特定的配置方法:


```
public AssociationTypeConfiguration<TAssociation> hasVisible(boolean visible);
```

此方法用于设置关联型是显式还是隐式的,此方法的默认值为true.即使用association(Class< TAssociation >)方法获取的配置默认为显式关联配置.

一个关联型是否为显式关联会影响关联引用的设值和取值(关联引用详见类型元素配置中关联引用配置),一般而言显式关联型对应的关联引用的元素类型应该是显式关联型,而隐式关联型的关联引用应该是关联的另外一端.

当然,在某些情况下,显式关联型的关联引用也可以是关联的对端.不过一般而言不需要更改此选项的配置.


```
public AssociationEndConfiguration<TAssociation> associationEnd(String name, Class<?> entityType) ;
public <TResult> AssociationEndConfigurationGeneric<TAssociation, TResult> associationEnd(SerializedFunction<TAssociation, TResult> get);
public AssociationEndConfiguration<TAssociation> associationEnd(String name);
```

这组方法用于配置关联端,每次调用时返回的均**是同一个对象.**

这些方法分别为:

1. 使用指定的名称和关联端类型创建关联端配置,此方法仅会创建配置而不会配置取值器,设值器.

2. 使用表达式创建关联端配置,此方法会创建配置且配置取值器,设值器.

3. 使用指定的名称创建关联端配置,此方法会创建配置且配置取值器,设值器.

- **隐式关联型**

隐式关联型配置是从ModelBuilder的association()方法返回值获取的,每次调用返回的都**是一个新的配置对象.**

此方法返回的对象是AssociationConfiguratorBuilder隐式关联建造器,建造器没有之前共通配置部分的方法,而是有以下配制方法:


```
public <TEntity> AssociationEndConfigurationGeneric<TEntity> associationEnd(Class<TEntity> endClass)
```

此方法用于配置隐式关联型的关联端,每次调用返回的都**是一个新的配置对象.**

隐式关联型会根据注册的关联端动态创建类型,至少需要两个关联端才可以创建,所以每个隐式关联建造器需要调用两次associationEnd方法来注册关联端.

隐式关联型还有以下限制:

1. 关联端标签由各关联端实体型的完全限定名排序后串联生成,因此同一组类型(顺序无关)只能建立一条隐式关联,第二个指向同一组类型的关联引用不会再自动建立关联型.

2. 只有两个关联端的隐式关联会被置为不可见(visible为false),即不作为独立的关联型参与查询,其所有关联端的延迟加载也会被强制关闭.

3. 如果配置了伴随端,且根据伴随端推断出的映射表与toTable配置的表不一致,会在建模时抛出异常.

需要注意的是,隐式关联型不能定义属性.如果概念上的关联存在独属于关系的属性,应当改为使用显式关联型(即定义关联类并用association(Class<TAssociation>)注册),或参考[Obase如何配置隐式关联的显式化](../Obase进阶使用/Obase如何配置隐式关联的显式化_java.md).


```
public AssociationConfiguratorBuilder toTable(String table);
```

此方法用于配置隐式关联的映射表.

### 复杂类型配置

复杂类型配置是从ModelBuilder的complex方法返回值得到的.

复杂类型没有特定于复杂类型的配置.

## 类型元素配置

类型元素配置是使用结构化类型的相应的配置方法获取的,如结构化类型的attribute方法获取的属性配置等.

以下提及的属性访问器以及Property类型均为使用ObaseIntrospector的静态方法getObaseBeanProperties获取到的属性,此方法内部使用的是Java内省机制获取的Getter和Setter.

### 共通配置

对于属性配置,关联端配置和关联引用配置,都具有以下的共通配置方法:

- **类型元素扩展配置方法**


```
public ElementExtensionConfiguration hasExtension(Class<? extends ElementExtensionConfiguration> configType);
```

这个方法的泛型参数必须是ElementExtensionConfiguration的继承类且必须要有无参的构造函数.

这个方法会将ElementExtensionConfiguration的makeExtension()方法返回的ElementExtension继承类存储于相应的类型元素类型中,以供进行某些扩展性的处理.

可以在映射模块,配置管道等扩展性机制中获取这些ElementExtension类型扩展进行某些判断或者处理携带的信息等操作.


```
public TConfiguration hasValueGetter(IValueGetter valueGetter);
public TConfiguration hasValueGetter(FunctionWithOneArg<TStructural, TProperty> getValue);
public TConfiguration hasValueGetter(Method method);
public TConfiguration hasValueGetter(Property property);
public TConfiguration hasValueGetter(Field field);
public TConfiguration hasValueGetter(String memberName);
public <TProperty> TConfiguration hasValueGetterMultiple(FunctionWithOneArg<TStructural, Iterable<TProperty>> getValue);
```

这是一组设置类型元素取值器的方法,取值器的**取值指的是从对象中获取值**,在探测对象是否修改,关联是否建立等场景中,Obase会调用取值器来获取元素的值.

这些方法分别是:

1. 将IValueGetter接口的实现类设置为取值器.

2. 将调用传入委托作为取值方法的取值器设置为取值器.

3. 将调用传入方法作为取值方法的取值器设置为取值器.

4. 将调用传入属性访问器的Set方法作为取值方法的取值器设置为取值器.

5. 将调用传入字段反射取值方法作为取值方法的取值器设置为取值器.

6. 传入成员名称,按名称自动侦测取值器.

7. 将集合取值委托(FunctionWithOneArg<TStructural, Iterable<TProperty>>)作为取值方法的取值器设置为取值器,用于显式声明取到的是集合.此重载只能用于具多重性的类型元素,对单一性元素调用会抛出异常.


```
public TConfiguration hasValueSetter(IValueSetter valueSetter);
public <TValue> TConfiguration hasValueSetter(ActionWithTwoArg<TStructural, TValue> setValue, EValueSettingMode mode);
public TConfiguration hasValueSetter(Method method, EValueSettingMode mode);
public TConfiguration hasValueSetter(Property property);
public TConfiguration hasValueSetter(Field field);
public TConfiguration hasValueSetter(String memberName);
public <TProperty extends Iterable<TElement>, TElement> TConfiguration hasValueSetter(ActionWithTwoArg<TStructural, TProperty> propertyExp, Function<Iterable<TElement>, TProperty> valueCreator);
public <TProperty> TConfiguration hasValueSetter(ActionWithTwoArg<TStructural, TProperty> propertyExp);
```

这是一组设置类型元素设值器的方法,取值器的**设**值**指的是向对象中设置值**,在构造对象等场景中,Obase会调用设值器来为元素设置值.

这些方法分别是:

1. 将IValueSetter接口的实现类设置为设值器.

2. 将调用传入委托作为设值方法和指定的设值模式(模式包括追加和赋值两种)的设值器设置为设值器.

3. 将调用传入方法作为设值方法和指定的设值模式(模式包括追加和赋值两种)的设值器设置为设值器.

4. 将调用传入属性访问器的Set方法作为设值方法和赋值设值模式的设值器设置为设值器.

5. 将调用传入字段反射设值方作为设值方法和赋值设值模式的设值器设置为设值器.

6. 传入成员名称,按名称自动侦测设值器.由于无法确定设值模式,此重载只支持按字段名构造设值器,如果按名称只找到方法,会抛出"暂不支持用Method构造设值器"的异常.

7. 传入属性访问委托,或者传入属性访问委托与"由值序列项构造属性值"的函数,用于集合属性的设值.

### 属性配置

属性配置是从结构化类型的attribute方法得到的,有以下特定的配制方法:


```
public AttributeConfiguration<TStructural> hasCombinationHandler(EAttributeCombinationHandlingStrategy strategy);
```

此方法用于在该结构化类型配置为EConcurrentConflictHandlingStrategy.Combine版本合并策略时,此属性的合并方式,共有三种:

1. EAttributeCombinationHandlingStrategy.Ignore:忽略 即使用旧对象(即原有对象)的值

2. EAttributeCombinationHandlingStrategy.Overwrite:覆盖 即使用新对象(即当前)的值 此种策略为默认的策略 可以不配置

3. EAttributeCombinationHandlingStrategy.Accumulate:累加 即将当前版本中属性值的增量累加到对方版本 只支持数值型的属性


```
public AttributeConfiguration<TStructural> hasMappingConnectionChar(char value);
```

此方法用于配置映射连接符,映射连接符是指示复杂类型是如何存储于映射表中的,如使用复杂类型的属性字段为A,复杂类型有两个属性B和C,那么在映射表中字段名称会被设置为"A映射连接符B"和"A映射连接符C".

默认的映射连接符为空字符.


```
public AttributeConfiguration<TStructural> hasMaxCharNumber(int maxCharNumber);
```

此方法用于配置属性的最大字符数,仅对映射类型为String和UUID类型的有效.会影响结构映射自动建表,如果设置为0,字段长度会被设置为40.如果长度超过255,会被设置为Text字段(SqlServer的阈值是500,Sqlite则忽略长度直接映射为TEXT).

默认值为0;


```
public AttributeConfiguration<TStructural> hasPrecision(byte precision);
```

此方法用于配置属性的精度,参数表示小数点后的位数,最大可以为28,仅对映射类型decimal有效.会影响结构映射自动建表.

默认值为0.


```
public AttributeConfiguration<TStructural> hasNullable(boolean value);
```

此方法用于配置属性是否可空,对于主键此配置是无效的,主键必然不为空.会影响结构映射自动建表.

默认值为true,可以为空.


```
public AttributeConfiguration<TStructural> toField(String field);
```

此方法用于设置映射字段.

```
public <TProperty> AttributeConfiguration<TStructural> hasValueSetter(ActionWithTwoArg<TStructural, TProperty> setValue);
```

这是属性配置特有的设值器重载,等价于基类上的hasValueSetter(setValue, EValueSettingMode.Assignment),即固定使用赋值模式,只传入设值委托.

```
public AttributeConfiguration<TStructural> useSerializer(ITextSerializer serializer, Class<?> valueType);
public AttributeConfiguration<TStructural> useSerializer(TextSerializer serializer, Class<?> valueType);
```

设置了序列化的属性会在保存时先进行序列化再进行存储,在构造对象时则先进行反序列化再设置值.

第一个方法是使用自定义的的序列化方案对当前属性进行处理,需要实现接口定义的相关方法,第一个参数即实现类,第二个参数是此属性的原始类型.

第二个方法是使用预制的序列化方案基类对当前属性进行处理,需要实现从字符串转换为对象的方法和对象转换为字符串的方法,第一个参数即实现类,第二个参数是此属性的原始类型.

```
public AttributeConfiguration<TStructural> useSerializationModel(boolean use);
```

此方法用于设置是否使用序列化模型来处理此属性的序列化,具体的序列化逻辑由序列化实体型配置决定.

启用后,属性的取值器与设值器会被包装为序列化模型的取值器/设值器,因此启用前必须已经配置了取值器(或设值器)与序列化器,否则在取值/设值时会抛出异常.

详细内容参见下面的序列化元素配置部分.

### 关联端配置

关联型上对于参与关联各端的引用称为关联端,关联端配置是从关联型配置的associationEnd方法得到的.

关联端配置有以下特定的配置方法:


```
public AssociationEndConfiguration<TAssociation> asCompanion(boolean asCompanion) ;
```

设置此端是否为伴随端,设置当前端为伴随端会将之前设置的伴随端改设不作为伴随端.

伴随端会影响此关联的映射表,在未使用toTable方法设置映射表时,会将伴随端的映射表设置为关联的映射表.

默认值为false,不是伴随端.


```
public TConfiguration hasEnableLazyLoading(boolean enableLazyLoading);
```

在启用了延迟加载后,访问此端的属性访问器Get就会触发延迟加载,尝试从数据源加载对象.

默认为false,不启用延迟加载.


```
public AssociationEndConfiguration<TAssociation> hasDefaultAsNew(boolean defaultAsNew);
```

设置此关联端是否默认作为新对象附加至上下文.

关联端是否默认创建新对象配置控制如果某一个对象被创建出来后,未附加至上下文,但作为其他已附加对象的引用对象时,是否作为新对象附加至上下文.

默认为不作为,因为对象往往是由应用层创建的,是否需要附加由应用层决定即可.

但如果某个对象的关联是无法通过此对象外部进行创建 如只能在构造函数内一起创建时,外部无法获取这个被一起创建的对象进行附加操作,此时将此值设置为true.

默认值false,不默认作为新对象附加至上下文.


```
public AssociationEndConfiguration<TAssociation> isAggregated(boolean isAggregated);
```

设置此关联端是否为聚合的.

设置为true表示聚合的端,对象在被聚合的关系解除或者关系的另外一端被删除是会被一并删除的.如A和B的关系,将B设置为聚合的,删除A时和解除A和B关系时(如把A中的B置空或者替换,如果是一对多的从集合中移除) B也会被删除.

默认值false,不是聚合的.


```
public AssociationEndConfiguration<TAssociation> hasMapping(String keyAttribute, String targetField);
```

此方法用于设置关联端在关联表内主键的映射.

第一个参数固定为当前关联端的实体型主键中的一个,第二个参数为此主键在关联表内的字段名称.完整性检查要求实体型的每一个主键属性都应有且仅有一个映射,配置不合规会报"标识属性XXX应有且仅有1个映射".

如果实体型存在多个主键,可以多次调用,每次调用会添加一个主键的映射.


```
public AssociationReferenceConfiguration<TEntity> associationReference(String name, boolean isMultiple);
public AssociationReferenceConfiguration<TEntity> associationReference(SerializedFunction<TEntity, TAssociation> get);
```

这组方法只在关联端上提供,实体型配置上没有associationReference方法.实际可用的重载取决于关联端是显式关联端还是隐式关联端:

1. 显式关联端(AssociationEndConfigurationGeneric)上可用的重载为:按名称和多重性配置的associationReference(String name, boolean isMultiple),以及按表达式配置的重载associationReference(SerializedFunction<TEntity, TAssociation>).其中按名称配置的重载不带被引对象类型参数,无法指定关联引用指向的关联型.

2. 隐式关联端(AssociationEndConfigurationGeneric,位于implicitAssociationConfigor包下)上额外提供了带被引对象类型参数的方法associationReference(String name, boolean isMultiple),可显式指定关联引用指向的关联型,这也是隐式关联显式化时要用到的写法.

这组方法用于配置此端实体型的关联引用,每次调用得到的都**是同一个对象.**

这组方法分别为:

1. 使用指定名称和多重性(一对一为false,一对多为true)来配置关联引用,会配置取值器和设置器.

2. 使用表达式来配置关联引用,此方法会创建配置,且会配置取值器和设置器.

需要注意的是,关联引用的左端与右端会在配置过程中自动推断:调用associationReference时会把左端名自动设为本关联端名,右端名在补充配置阶段根据关联端推断,一般不需要手工调用hasLeftEnd/hasRightEnd(这两个方法在AssociationReferenceConfiguration上是包级私有方法,仅由构建器内部调用).

### 关联引用配置

实体型上对于关联对象的引用称为关联引用,对于一般的显式关联,关联引用对象的元素类型是显式关联型;对于隐式关联,关联引用对象的元素类型是关联另外的端.关联引用配置是从关联端配置的associationReference方法得到的.

关联引用配置有以下特定的配置方法:


```
public TConfiguration hasEnableLazyLoading(boolean enableLazyLoading);
```

是否启用延迟加载,要想启用延迟加载,要延迟加载的引用元素(属性或关联引用)的定义必须有对应的Getter和Setter.在启用了延迟加载后,访问此端属性的Getter方法就会触发延迟加载,尝试从数据源加载对象.

需要注意的是,关联引用的延迟加载可能会导致性能损失,对一个由多个对象组成的集合,每次访问集合中的对象上配置了延迟加载的属性都会触发延迟加载,即进行了集合大小次的延迟加载.

默认为false,不启用延迟加载.

## 序列化元素配置

类型元素配置是使用建模器的serializationEntity方法得到的.

### 建模器方法

```
public <TEntity> SerializationEntityConfigurationGeneric<TEntity> serializationEntity(Class<TEntity> entityClass);
```

此方法就是用于获得序列化实体型配置的方法,此后的配置需要由此方法返回的对象引出,每次调用时返回的均**是同一个对象.**.

### 序列化实体型配置方法

序列化实体有以下几组配置方法:

```
public SerializationAttributeConfiguration<T> attribute(String name, Class<?> valueType);
public SerializationAttributeConfiguration<T> attribute(String name);
public <TResult> SerializationAttributeConfiguration<T> attribute(SerializedFunction<T, TResult> expression)
```

这组方法都会返回一个序列化属性配置,每次调用时返回的均**是同一个对象.**

这些方法分别是:

1. 手动配置序列化属性方法,指定序列化属性的名称和类型配置属性,此方法仅会创建配置而不会配置取值器,设值器.此方法会校验传入的类型必须是Obase基元类型,否则抛出异常.
2. 指定名称配制方法,指定属性的名称配置属性.会自动侦测属性的类型配置取值器,设值器.
3. 使用表达式配置方法,会根据表达式所指向的属性访问器配置名称,属性的类型,配置取值器,设值器.

*一般情况下使用较多的是第三个配置方法,因为大多数的属性都可以会被反射建模自动侦测并配置,只有在需要自定义取值器设值器映射属性的时候才会使用这些配置方法.*

```
public SerializationConstructorConfiguration<T> hasConstructor(Constructor<T> constructor);
```

此方法会返回一个序列化构造函数配置,每次调用时返回的**是一个新对象.**

此方法需要类的构造函数作为参数.

```
public SerializationReferenceConfiguration<T> reference(String name, boolean isMultiple);
public SerializationReferenceConfiguration<T> reference(String name);
public <TResult> SerializationReferenceConfiguration<T> reference(SerializedFunction<T, TResult> expression);
```
这组方法都会返回一个序列化引用配置,每次调用时返回的均**是同一个对象.**

这些方法分别是:

1. 手动配置序列化引用方法,指定序列化引用的名称和是否为多值的,此方法仅会创建配置而不会配置取值器,设值器.
2. 指定名称配制方法,指定引用的名称配置引用.会自动侦测引用的类型配置取值器,设值器.
3. 使用表达式配置方法,会根据表达式所指向的属性访问器配置名称,引用的否为多值,配置取值器,设值器.

注意取值器与设值器是序列化属性配置和序列化引用配置共有的配置(它们都继承自序列化类型元素配置),所以序列化引用上也可以使用hasValueGetter和hasValueSetter,下面序列化引用配置方法部分列出的即为此类共有方法.

```
public SerializationEntityConfigurationGeneric<T> ignore(String name);
public <TResult> SerializationEntityConfigurationGeneric<T> ignore(SerializedFunction<T, TResult> expression);
```

这组方法用于指定不处理哪些属性,每次调用都会返回当前的序列化实体配置.

### 序列化属性配置方法

```
public SerializationTypeElementConfiguration<TStructural> hasValueGetter(Field field);
public <TProperty> SerializationTypeElementConfiguration<TStructural> hasValueGetter(FunctionWithOneArg<TStructural, TProperty> getValue);
public SerializationTypeElementConfiguration<TStructural> hasValueGetter(IValueGetter getter);
```

这组方法用于配置序列化属性的取值器,分别为:

1. 使用字段配置的取值器方法.
2. 使用委托配置的取值器方法.
3. 使用取值器接口实现配置的取值器方法.

```
public SerializationTypeElementConfiguration<TStructural> hasValueSetter(Field field);
public <TValue> SerializationTypeElementConfiguration<TStructural> hasValueSetter(ActionWithTwoArg<TStructural, TValue> setValue, EValueSettingMode mode);
public SerializationTypeElementConfiguration<TStructural> hasValueSetter(IValueSetter valueSetter);
```

这组方法用于配置序列化属性的设值器,分别为:

1. 使用字段配置的设值器方法.
2. 使用委托和设值模式配置的设值器方法.
3. 使用设值器接口实现配置的设值器方法.

### 序列化构造函数配置方法

```
public SerializationConstructorConfiguration<TStructural> hasParameter(Field field, Class<?> valueType, boolean needStorage);
public <TProperty> SerializationConstructorConfiguration<TStructural> hasParameter(FunctionWithOneArg<TStructural, TProperty> getValue, Class<?> valueType, boolean needStorage);
public SerializationConstructorConfiguration<TStructural> hasParameter(IValueGetter valueGetter, Class<?> valueType, boolean needStorage);
public SerializationConstructorConfiguration<TStructural> hasParameter(Field field, Class<?> valueType, boolean needStorage, FunctionWithOneArg<Object, Object> valueConvert);
public <TProperty> SerializationConstructorConfiguration<TStructural> hasParameter(FunctionWithOneArg<TStructural, TProperty> getValue, Class<?> valueType, boolean needStorage, FunctionWithOneArg<Object, Object> valueConvert);
public SerializationConstructorConfiguration<TStructural> hasParameter(IValueGetter valueGetter, Class<?> valueType, boolean needStorage, FunctionWithOneArg<Object, Object> valueConvert);
```
这组方法用于配置序列化构造函数的参数,每次调用会配置一个参数,调用时要与使用的构造函数参数一一对应.

这些方法分别是:
1. 使用参数对应的取值字段,存储的类型,是否需要存储来配置序列化构造器参数方法.
2. 使用参数的取值委托,存储的类型,是否需要存储来配置序列化构造器参数方法.
3. 使用取值器的实现,存储的类型,是否需要存储来配置序列化构造器参数方法.

其中前三个方法不带valueConvert参数,后三个方法多一个值转换委托,用于在存储前把取值器取得的值转换为其他类型,不需要转换时可以不传.

这些方法的参数中,第二个参数为取得的值类型,如果设置needStorage为true,则在序列化时会检查取值器取得的值是否是此类型的,同时会校验此类型必须是Obase的基元类型,否则抛出异常.第三个参数是是否需要存储,如果是true,则取值器会在序列化时被调用将取得的值进行存储,此时传入的取值器的参数为当前要序列化的对象;如果是false 则取值器会在反序列化被调用,取得的值用于构造函数,此时传入的取值器的参数为null.

配置参数时必须与构造函数参数一一对应,配置的参数个数超过构造函数的真实参数个数会抛出异常;如果当前配置的值类型与构造函数对应位置的参数类型不匹配且没有提供valueConvert,同样会抛出异常.

### 序列化引用配置方法

```
public SerializationTypeElementConfiguration<TStructural> hasValueGetter(Field field);
public <TProperty> SerializationTypeElementConfiguration<TStructural> hasValueGetter(FunctionWithOneArg<TStructural, TProperty> getValue);
public SerializationTypeElementConfiguration<TStructural> hasValueGetter(IValueGetter getter);
```

这组方法用于配置序列化引用的取值器,分别为:

1. 使用字段配置的取值器方法.
2. 使用委托配置的取值器方法.
3. 使用取值器接口实现配置的取值器方法.

```
public SerializationTypeElementConfiguration<TStructural> hasValueSetter(Field field);
public <TValue> SerializationTypeElementConfiguration<TStructural> hasValueSetter(ActionWithTwoArg<TStructural, TValue> setValue, EValueSettingMode mode);
public SerializationTypeElementConfiguration<TStructural> hasValueSetter(IValueSetter valueSetter);
```

这组方法用于配置序列化引用的设值器,分别为:

1. 使用字段配置的设值器方法.
2. 使用委托和设值模式配置的设值器方法.
3. 使用设值器接口实现配置的设值器方法.

## 其他配置

### 建模器方法

建模器ModelBuilder还有以下几个用于配置管道相关的配置方法.

配置管道是一种扩展机制,可以由调用方在管道中加入步骤来处理某些特定的逻辑.

Obase的配置管道分为四条,分别是类型解析管道,类型元素解析管道,代理类型创建管道和补充配置管道.

不同的配置管道会在建模过程中的不同步骤中被调用.

类型解析管道将在反射建模的最开始时被调用,分析已有的结构化配置并对结构化类型进行配置.

类型元素解析管道将在反射建模创建结构化配置之后被调用,对结构化类型的类型元素进行配置.

代理类型创建管将在反射建模配置结束后调用,对代理类型进行配置.

补充配置管道反射建模最后被调用,进行补充配置.


```
public TypeAnalyticPipelineBuilder useTypeAnalyzer(FunctionWithOneArg<ITypeAnalyzer, ITypeAnalyzer> middlewareDelegate);
public TypeMemberpublic TypeMemberAnalyticPipelineBuilder useMemberAnalyzer(FunctionWithOneArg<ITypeMemberAnalyzer, ITypeMemberAnalyzer> middlewareDelegate);
public ProxyTypeGenerationPipelineBuilder useProxyTypeGenerator(FunctionWithOneArg<IProxyTypeGenerator, IProxyTypeGenerator> middlewareDelegate);
public ComplementConfigurationPipelineBuilder useComplementConfigurator(FunctionWithOneArg<IComplementConfigurator, IComplementConfigurator> middlewareDelegate);
```

这组方法即用于配置管道的方法,每个都接受一个委托,此委托用于创建所需的解析器.

每次调用这些方法都会在管道中新增一个子步骤,每个管道的结构为默认解析器->第一次调用加入的解析器->第二个......->最后加入的.

```
public void hasIntegrityCheck(boolean integrityCheck)
```

此方法用于设置是否进行完整性检查,默认为进行,如果设置为false则不进行完整性检查.