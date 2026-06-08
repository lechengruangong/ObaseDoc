Obase支持多种对象查询操作,这些查询方法在Sql数据源中被映射为特定的关系运算,用户在查询时可以组合这些查询操作来实现查询需求.

以下内容中的SerializedPredicate<T>,SerializedFunction<T, R>等类似的参数均为Functional Interface,可以使用Lambda表达式来作为参数.

关于这些Lambda表达的使用, 可以参考Lambda表达式的限制一节的内容.

## 查询运算

接下来介绍这些查询操作和对应的关系运算.

这里需要注意的是,在java中,所有的查询表达式里作为条件的变量都必须为本地变量,即比如我们要查询Id > 0的数据,那么在写表达式时应当写作set.filter(p -> p.getId() > 0)或者定义一个本地变量int con = 0然后写作set.filter(p -> p.getId() > con).

而不能将传入的其他对象的方法作为查询条件,这部分内容在下一节筛选条件拼合器中详细解释.

- All方法,判断对象集内对象是否全部满足条件.

映射为获取满足条件取反的数据个数操作后返回个数是否等于0的运算.

```
allMatch(final SerializedPredicate<T> predicate);
```

- Any方法,判断对象集内是否存在满足条件的对象.

映射为获取满足条件的数据个数操作后返回个数是否大于0的运算.

```
anyMatch(final SerializedPredicate<T> predicate);
anyMatch();
```

- Average方法,获取对象集中对象某个属性的平均数.

映射为取目标字段平均数运算.

```
avg(final SerializedDoubleResult<T> doubleResult);
avg(final SerializedFloatResult<T> floatResult);
avg(final SerializedBigDecimalResult<T> bigDecimalResult);
avgInt(final SerializedIntResult<T> intResult);
```

- Contains方法,判断对象集中对象的某个属性是否被包含于某个集合中或某个属性是否包含某个值.

映射为获取满足条件的数据个数操作后返回个数是否大于0的运算.

可以支持字符串属性包含(如set.filter(p->p.getName().contains("123")))和集合包含某个属性(如set.filter(p->list.contains(p.getId()))

此外字符串属性还支持startWith和endWith,如set.filter(p->p.getName().startWith("123"))

- Count方法,获取对象集中符合条件的对象个数.

映射为获取满足条件的数据个数运算.

```
count(final SerializedPredicate<T> predicate);
count();
```

- Distinct方法,获取对象集中不重复的对象.

映射为去除查询结果中的重复值运算.

```
distinct();
```

- ElementAt方法,获取对象集中位于某个索引的对象,从0开始.

映射为提取传入的索引值+1并跳过索引值条数据的运算.

```
elementAt(int index);
```

- Filter运算,指示查询时的筛选条件.

映射为获取满足条件的数据运算.

```
filter(final SerializedPredicate<T> filterExpression);
```

- FirstOrDefault方法,获取对象集中符合条件的第一个对象,不存在对象时会返回空.

映射为获取满足条件的首个数据运算.

```
findFirst(final SerializedPredicate<T> predicate);
findFirst();
```
- FindLast方法,获取对象集中符合条件的最后一个对象,不存在对象时会返回空.

映射为获取满足条件的按照排序键倒序排序的首个数据运算.

```
findLast(final SerializedPredicate<T> predicate);
findLast();
```

- FlatMap运算,将对象集中对象平展投影至关联对象.

如果目标为字段,映射为取指定字段运算.如果目标为关联对象,映射为联表查询运算.

```
flatMap(final SerializedFunction<T, Iterable<R>> mapExpression, Class<R> targetClass);
flatMap(final SerializedFunction<T, Iterable<TCollect>> getCollect, final FunctionWithTwoArgs<T, TCollect, TResult> getResult, Class<TResult> resultType);
```

- Group方法,获取对象集中将对象分组投影后的结果.

映射为分组运算.

```
groupBy(final SerializedFunction<T, TKey> getKey);
groupBy(final SerializedFunction<T, TKey> getKey, final SerializedFunction<T, TElement> getElement);
groupBy(final SerializedFunction<T, TKey> getKey, final FunctionWithTwoArgs<TKey, IAggregation<T>, TResult> getResult, Class<?> resultType);
```

- Include方法,指示查询时要一并加载的关联对象.

映射为联表查询运算.

```
include(final SerializedFunction<T, R> includeExpression);
include(String includeExpression);
```

第一个重载参数可以表示为set.Include(p=>p.B.Cs),即表示查询A时加载B,C集合.

请注意此方法只支持引用元素(关联引用或关联端)成员访问表达式,如果遇到需要加载一对多时,使用第二个重载方法.

第二个重载参数可以表示为set.Include("B.Cs.D").

- Limit运算,指示查询时提取多少个对象.

映射为提取指定条数数据的运算.

```
limit(int maxSize);
```
- Map运算,将对象集中的对象投影至某个结果.

如果目标为字段,映射为取指定字段运算.如果目标为关联对象,映射为联表查询运算.

```
map(final SerializedFunction<T, R> mapExpression, Class<?> targetClass);
```

- Max运算,获取对象集中对象某个属性的最大值.

映射为取目标字段的最大值运算.

```
max(final SerializedFunction<T, R> get);
```

- Min运算,获取对象集中对象某个属性的最小值.

映射为取目标字段的最小值运算.

```
min(final SerializedFunction<T, R> get);
```

- Reverse运算,指示查询结果是否逆序返回.

映射为按照排序键倒序排序运算.

```
reverse();
```

- Single运算,获取对象集中符合条件的某个单一元素,如果不存在符合条件的元素会返回空.

映射为获取满足条件的数据后,判断数据是否只有一条的运算.

- Skip运算,指示查询时跳过多少个对象.

映射为跳过指定条数数据的运算.

```
skip(int n);
```

- Sum运算,获取对象集中对象某个属性的和.

映射为取目标字段的和运算.

```
sum(final SerializedLongResult<T> longResult);
sum(final SerializedDoubleResult<T> doubleResult);
sum(final SerializedFloatResult<T> floatResult);
sum(final SerializedBigDecimalResult<T> bigDecimalResult);
```
- Sort运算,指示查询时的排序.

映射为按目标字段排序运算.

```
sorted(final SerializedFunction<T, R> get);
sortedDesc(final SerializedFunction<T, R> get);
thenSorted(final SerializedFunction<T, R> get);
thenSortedDesc(final SerializedFunction<T, R> get);
```

这些方法中,Sorted是覆盖之前条件的正序排序,SortedByDescending是覆盖之前排序条件的倒序排序,ThenSorted是保留之前排序的正序排序,ThenSortedDesc是保留之前排序的倒序排序.

- 获取结果方法

这些方法是用来执行终结操作,将结果转换为指定类型的方法.

```
toArray();
toList();
toHashMap(SerializedFunction<T, TKey> getKey, SerializedFunction<T, TResult> getResult);
toHashMapWithIterableResult(SerializedFunction<T, TKey> getKey, SerializedFunction<T, TResult> getResult);
```

第一个方法为转换为数组;第二个方法为转换为列表;第三个方法为转换为HashMap,相同键的结果保留一个,通常用于Group操作之后;第四个方法为转换为HashMap并将相同键的结果放入Iterable中,通常用于Group操作之后.

## 筛选条件拼合器

日常使用中,最常用的查询操作就是筛选(Filter)操作,为了动态的拼接筛选条件需要能直接构造筛选条件.

Obase提供了PredicateCombiner类用于拼接筛选条件.

首先是一组新的and和or的重载,现在and和or都有三个重载,分别是

- public PredicateCombiner<T> and(final SerializedPredicate<T> filterExpression) throws Exception 接受一个普通的谓词逻辑表达式

- public PredicateCombiner<T> and(LambdaExpression second) throws Exception 接受一个解析完成的Lambda表达式

- public <R> PredicateCombiner<T> and(final SerializedFunction<T, R> getExpression, ePredicateType predicateType, Object value) throws Exception  接受成员表达式,谓词运算符,参数值

其中第一个重载即接受普通的p->p.getCode() == parameter 这种的表达式,其中parameter必须为本地变量.

第二个重载可以接受predicateCombiner.getWrapper()的eq相等,ne不相等,lt小于,gt大于等数个方法的发回执或者接受另外一个predicateCombiner.getLambdaExpression()的返回值.

第三个重载的参数分别为成员表达式p->p.getCode() ,谓词运算符(等于,不等于,大于,小于等等)和参数值,此处的第三个参数值参数可以不是普通变量.

然后还提供了一组equal,notEqual,lessThan等方法,这些方法均有两个重载

- public <R> PredicateCombiner<T> equal(final SerializedFunction<T, R> getExpression, Object value) throws Exception 接受成员表达式和参数值

- public <R> PredicateCombiner<T> equal(final SerializedFunction<T, R> getExpression, Object value, eCombineType combineType) throws Exception 接受成员表达式,参数值和拼合类型

第一个重载可以接受成员表达式p->p.getCode(),第二个参数为要运算的参数值并且以and的方式拼合条件

第二个重载可以接受成员表达式p->p.getCode(),第二个参数为要运算的参数值并且以第三个参数指定的方式拼合条件

这些方法的返回值都是拼合器自身以方便链式调用,这些方法当然可以混用,最终通过拼合器的.getLambdaExpression()获取最终的返回值.

接下来我们用一个示例来介绍这些新方法


```
/**
 * 表示客户
 */
public class Client {

    /**
     *表示年龄
     */
    private int age;

    /**
     * 客户ID
     */
    private long clientId;

    /**
     * 创建时间
     */
    private LocalDateTime createdTime = LocalDateTime.now();

    /**
     * 客户姓名
     */
    private String name;

    /**
     * 客户手机号
     */
    private String phoneNumber;

    /**
     * 性别
     */
    private eSex sex;

    /**
     * 反持久化构造函数
     * @param clientId 客户ID
     */
    protected Client(long clientId)
    {
        this.clientId = clientId;
    }

    /**
     * 使用姓名和手机号初始化客户
     * @param name 客户姓名
     * @param phoneNumber 客户手机号
     */
    public Client(String name,String phoneNumber){
        this.name = name;
        this.phoneNumber = phoneNumber;
    }

    /**
     * 获取客户姓名
     * @return 客户姓名
     */
    public String getName() {
        return name;
    }

    /**
     * 设置客户姓名
     * @param name 客户姓名
     */
    public void setName(String name) {
        this.name = name;
    }

    /**
     * 获取创建时间
     * @return 创建时间
     */
    public LocalDateTime getCreatedTime() {
        return createdTime;
    }

    /**
     * 反持久化用设置创建时间
     * @param createdTime 创建时间
     */
    void setCreatedTime(LocalDateTime createdTime) {
        this.createdTime = createdTime;
    }

    /**
     * 获取年龄
     * @return 年龄
     */
    public int getAge() {
        return age;
    }

    /**
     * 设置年龄
     * @param age 年龄
     */
    public void setAge(int age) {
        this.age = age;
    }

    /**
     * 获取客户ID
     * @return 客户ID
     */
    public long getClientId() {
        return clientId;
    }

    /**
     * 反持久化设置客户ID
     * @param clientId 客户ID
     */
    void setClientId(long clientId) {
        this.clientId = clientId;
    }

    /**
     * 获取客户手机号
     * @return 客户手机号
     */
    public String getPhoneNumber() {
        return phoneNumber;
    }

    /**
     * 设置客户手机号
     * @param phoneNumber 客户手机号
     */
    public void setPhoneNumber(String phoneNumber) {
        this.phoneNumber = phoneNumber;
    }

    /**
     * 获取性别
     * @return 性别
     */
    public eSex getSex() {
        return sex;
    }

    /**
     * 设置性别
     * @param sex 性别
     */
    public void setSex(eSex sex) {
        this.sex = sex;
    }

    @Override
    public String toString() {
        return "Client{" +
                "age=" + age +
                ", clientId=" + clientId +
                ", createdTime=" + createdTime +
                ", name='" + name + '\'' +
                ", phoneNumber='" + phoneNumber + '\'' +
                ", sex=" + sex +
                '}';
    }
}

```

这是一个简单的域类,假设我们有一个接口来分页的查询此域类对象,并使用以下的Dto作为参数.


```
/**
 * 通用的Api请求分页参数
 */
@Data
@NoArgsConstructor
@ToString
public class ClientRequest implements Serializable {

      /**
     * 客户名称
     */
    private String name;

    /**
     * 客户手机号
     */
    private String phoneNumber;

    /**
     * 性别
     */
    private eSex sex;

    /**
     *表示年龄
     */
    private int age;

    /**
     * 创建时间
     */
    @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private LocalDateTime createdTime;

    /**
     * 页大小
     */
    private int pageSize;

    /**
     * 从1开始的页索引
     */
    private int pageIndex;

    /**
     * 获取具体跳过的数量
     * @return 具体跳过的数量
     */
    public int skip(){
        if(pageSize <= 0)
            pageSize = 0;
        if(pageIndex < 1)
            pageIndex = 1;

        return pageSize * (pageIndex - 1);
    }

    /**
     * 获取具体提取的数量
     * @return 具体提取的数量
     */
    public int limit(){
        if(pageSize <= 0)
            pageSize = 10;

        return pageSize;
    }
}
```

接口的代码如下:


```
//获取上下文
SampleObjectContext sampleObjectContext = this.context;
//构造条件拼合器
PredicateCombiner<Client> predicateCombiner = new PredicateCombiner<>();
//使用predicateCombiner.getWrapper()构造传入的拼合条件 此处也可以传入其他的表达式拼合的getLambdaExpression()
predicateCombiner.and(predicateCombiner.getWrapper().eq(Client::getSex, clientRequest.getSex()));
//如果存在关键字
if(clientRequest.getName() != null && !clientRequest.getName().isEmpty()){
    //使用指定ePredicateType进行拼合 需要成员表达式和值
    predicateCombiner.and(Client::getName, ePredicateType.Contains, clientRequest.getName());
}
//使用其他拼合方法 如equal lessThan greaterThanOrEqual 需要成员表达式和值 默认为与逻辑运算 也可以在第三个参数指定逻辑运算
predicateCombiner.equal(Client::getPhoneNumber,clientRequest.getPhoneNumber())
        .lessThan(Client::getCreatedTime,clientRequest.getCreatedTime())
        .greaterThanOrEqual(Client::getAge,clientRequest.getAge());
//查询总数
long count = sampleObjectContext.CreateSet(Client.class).count(predicateCombiner.getLambdaExpression());
if(count == 0)
    return ApiPagingResponse.success(clientRequest.getPageSize(),clientRequest.getPageIndex(),0);
//查询对象
List<Client> clients = sampleObjectContext.CreateSet(Client.class).filter(predicateCombiner.getLambdaExpression()).skip(clientRequest.skip()).limit(clientRequest.limit()).toList();
//组装结果
ClientDto[] result = clients.stream().map(p->new ClientDto(p.getAge(),p.getClientId(),
        p.getCreatedTime(),p.getName(),p.getPhoneNumber(),p.getSex())).toArray(ClientDto[]::new);
return ApiPagingResponse.success(result,clientRequest.getPageSize(),clientRequest.getPageIndex(),count);
```

这里的第6行,第10行和第13行分别使用了新的拼合方法:

第6行使用的方法需要调用predicateCombiner.getWrapper()的方法进行拼合,predicateCombiner.getWrapper()提供了eq相等运算,ne不相等运算,lt小于运算,gt大于运算,le小于等于运算,ge大于等于运算,cs字符串包含运算,sw字符串以XX开头运算,ew字符串以XX结尾运算.

第10行使用的方法需要自己指定运算符,ePredicateType包括的运算类型与predicateCombiner.getWrapper()提供的方法相同.

第13行的方法是指定运算类型的方法,如equal表示相等运算,此方法的第三个参数如果不传默认为与逻辑运算.

这些方法内部都有相应的检查,比如只支持Obase基元类型参与运算,某些类型不支持某些运算等.

不过由于条件拼合器默认是平展的,即各个条件之间都是在同一优先级,形如A and B or C.

那么如果要拼合复合条件,比如(A or B) and(C or D)这种时,条件拼合器无法满足.

拼合器有两个静态方法用于构造复杂的表达式树.

```
PredicateCombiner<JavaBean> sub1 = new PredicateCombiner<>();
sub1.or(combiner.getWrapper().eq(JavaBean::getIntNumber,1)).or(combiner.getWrapper().eq(JavaBean::getIntNumber,20));
PredicateCombiner<JavaBean> sub2 = new PredicateCombiner<>();
sub2.or(combiner.getWrapper().eq(JavaBean::getString,"1号字符串")).or(combiner.getWrapper().eq(JavaBean::getString,"19号字符串"));

List<JavaBean> list = context.CreateSet(JavaBean.class).filter(PredicateCombiner.and(sub1.getLambdaExpression(),sub2.getLambdaExpression())).toList();
```

这里我们使用了两个条件拼合器,sub1和sub2.

他们各自的条件是intNumber=1 or intNumber=20和string="1号字符串" or string = "19号字符串".

在使用PredicateCombiner.and()这一静态方法后,拼合为条件((intNumber=1 or intNumber=20) and (string="1号字符串" or string = "19号字符串"))

那么获得的结果就是intNumber=1 且 string="1号字符串"的对象.

当然,对于较为简单的情况,你可以直接在拼合条件时增加括号来指定优先级.

## Lambda表达式的限制

Obase使用的Lambda表达式主要有两个限制,分别是值要使用本地变量和查询类型必须为Obase注册的类型.

值要使用本地变量指的是在使用具体的某个值作为表达式的一部分时,存储这个值的变量必须为当前域的本地变量,代码如下:

```
String keyWord = dto.getKeyWord();
set.filter(p->p.getName() == keyWord);
```

这里的dto为假定的数据传输对象,我们想要根据Name和dto的KeyWord相等作为筛选条件,这里就需要先将dto的KeyWord存储于本地变量keyWord中.

如果直接将dto.getKeyWord()加入表达式,就会触发查询类型必须为Obase注册的类型的限制.

查询类型必须为Obase注册的类型指的是如果将没在Obase注册的类型传入表达式时,会抛出一个类似于如下的异常:

```
org.jinq.rebased.org.objectweb.asm.tree.analysis.AnalyzerException: Unknown method org/example/Examlpe:getCode()Ljava/lang/String; encountered
    at ch.epfl.labos.iu.orm.queryll2.symbolic.BasicSymbolicInterpreter.naryOperation(BasicSymbolicInterpreter.java:367)
    at org.jinq.rebased.org.objectweb.asm.tree.analysis.Frame.executeInvokeInsn(Frame.java:684)
    at org.jinq.rebased.org.objectweb.asm.tree.analysis.Frame.execute(Frame.java:609)
    at ch.epfl.labos.iu.orm.queryll2.path.CodePath.calculateReturnValueAndConditions(CodePath.java:148)
    at ch.epfl.labos.iu.orm.queryll2.path.TransformationClassAnalyzer.analyzeMethod(TransformationClassAnalyzer.java:505)
    at ch.epfl.labos.iu.orm.queryll2.path.TransformationClassAnalyzer.analyzeLambdaMethod(TransformationClassAnalyzer.java:471)
```

这个异常可以按照如下的顺序进行排查:

按照以下步骤进行排查

1. 检查异常中的 org/example/Examlpe:getCode()Ljava/lang/String 这一部分,此部分为方法签名的JNI表示形式,冒号前部分即为类名,检查此类型是否为在Obase中注册的类型.
    - 是注册类型,进入步骤2.
    - 不是注册类型,不要直接将非Obase模型类型传入表达式中,如果是Dto对象,需要先将其中包含的数据取出放置于本地变量中才能正确解析.

2. 检查抛出异常的代码行之前是否有正确的调用配置类的构造函数,建模过程是在配置类基类的构造函数内完成的,只有执行完配置类的构造函数才能在解析器中注册.
    - 有正确的执行配置类构造函数,检查此异常中的 org/example/Examlpe:getCode()Ljava/lang/String 冒号后半部分的方法签名,是不是定义了其他类似的内省方法导致此方法被内省机制识别为非Getter/Setter.如为一个boolean字段同时定义了get开头的和is开头的方法.
    - 没有正确执行,无论是PredicateCombiner还是ObjectSet的方法执行前都需要正确的完成建模过程,在调用之前先执行配置类的构造函数.

3. 特别的,在SpringBoot环境下需要特别进行如下检查:
    - 当前上下文是否是使用依赖注入获取的?如果是,继续检查.不是,同步骤2检查构造函数是否正确执行.
    - 依赖注入获取的上下文在没有被访问前并没有被真正构造,需要访问一次才会走构造函数,所以需要在调用前访问一次才能正确完成建模.

不过既然注册至解析器方法是Obase控制的,为什么不在把所有传入的表达式内的类型都在解析器里注册?这样不就不会报这个异常了吗?

这些限制主要有以下两个考量:

1. 将非Obase对象数据模型类型注册至解析器中会制造大量无意义的冗余数据.

2. 表达式解析中使用了反序列化的方式来处理,为了防止触发反序列化漏洞,解析器只允许被注册的类型才能进行解析,Obase作为解析器的调用方当然也只能允许指定的类型才能传入表达式进行解析,目前这个指定类型就是当前数据模型中的类型.如果允许任意类型都进入解析器中进行解析,反序列化的漏洞触发相当于交给了输入方,这是非常危险的.而现在的只允许对象数据模型的类型进行解析,可以由配置者进行甄别,以防止触发反序列化漏洞.

并且传入Dto对象也被注册的话,就会获得错误的解析结果,而不将传入的对象注册则会抛出此异常,防止了解析错误.例如如下的表达式

```
set.filter(p->p.getName() == dto.getKeyWord());
```

实际上想要表达的是查找A的Name与Dto中KeyWord的值相等的对象,但如果Dto对象也被注册,就会解析为A.Name == Dto.KeyWord,这显然是错误的.

当然,你可以使用条件拼合器来直接拼接条件里的值,条件拼合器里是直接获取值作为表达式的一部分而不是将传入参数作为表达式的一部分,所以不会触发这两个限制.