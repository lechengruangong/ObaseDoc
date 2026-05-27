在Obase中建模过程使用的是管道模式,每条管道都有若干个具体的解析器负责处理具体的逻辑.

共有四条管道处理建模,分别是:

- 类型解析管道,由类型解析器负责处理类型的配置逻辑.管道中解析器是ITypeAnalyzer的接口实现,可以对所有的结构化配置进行修改,实现接口的方法即可在参数中获取要配置的结构化类型配置.
- 类型元素解析管道,由类型元素解析器负责处理类型上的属性配置逻辑.管道中解析器是ITypeMemberAnalyzer的接口实现,可以对所有的类型元素进行修改,实现接口的方法即可在参数中获取要配置的类型元素配置.
- 代理解析管道,由代理解析器负责处理代理类型的逻辑.管道中解析器是IProxyTypeGenerator的接口实现,可以决定是否要创建某个结构化类型的代理类型,实现接口的方法即可返回是否要创建代理类型和如何定义代理类型.
- 补充配置管道,由补充配置器负责在所有模型都配置后处理补充逻辑.管道中的配置器是IComplementConfigurator的接口实现,可以修改已经配置的结构化类型和对应的结构化类型配置,实现接口的方即可获取到结构化类型和对应的结构化类型配置.需要注意的是,此时结构化类已经根据配置生成了,修改方法参数中的配置参数不会直接影响结构化类.

这四条管道分别在建模过程的不同阶段被调用,具体请参考[Obase的对象数据模型建模过程](../Obase基础知识/Obase的对象数据模型建模过程_java.md)文档,其中第7,8,10,12步中使用的管道就是对应的管道.

每条管道都有各自的接口可以实现,并且在对象模型配置建造器上有相应的方法将解析器接口实现注入到管道中用来扩展建模过程.

接下来以一个例子来说明如何扩展建模过程:

## 为所有的映射表增加特定前缀

思考这样的一个场景,我们的数据库表需要使用某个特定的前缀来表示,比如Enc前缀表示某个模块的名字,那么我们可以扩展补充配置管道来实现此需求.

首先,实现一个补充配置器:

```
/**
 * 映射表前缀补充配置器
 */
public class PrefixComplementConfigurator implements IComplementConfigurator {

    /**
     * 下一个补充配置器
     */
    private final IComplementConfigurator next;

    /**
     * 初始化映射表前缀补充配置器
     * @param next 下一个补充配置器
     */
    public PrefixComplementConfigurator(IComplementConfigurator next) {
        this.next = next;
    }

    /**
     * 补充配置管道中的下一个配置器
     *
     * @return 下一个补充配置器
     */
    @Override
    public IComplementConfigurator getNext() {
        return next;
    }

    /**
     * 根据类型配置项中的元数据配置指定的类型
     *
     * @param targetType    要配置的类型
     * @param configuration 包含配置元数据的类型配置项
     */
    @Override
    public void configure(StructuralType targetType, StructuralTypeConfiguration<?> configuration) {
        //TargetTable定义于ObjectType上 此处处理ObjectType
        if(targetType instanceof ObjectType){
            ObjectType objectType = (ObjectType) targetType;
            objectType.setTargetTable("Enc" + objectType.getTargetTable());
        }
    }
}
```

注意这个类的构造函数,考虑到管道中还有其他节点,此处应当接受Next参数作为下一个节点.

接着在上下文配置提供器的createModel方法里加入下的代码即可:

```
/**
 * 使用指定的建模器创建对象数据模型
 *
 * @param modelBuilder 建模器
 */
@Override
protected void createModel(ModelBuilder modelBuilder) {
    //向管道中注册新的节点
    modelBuilder.useComplementConfigurator(PrefixComplementConfigurator::new);

    //以下为模型配置代码
}
```