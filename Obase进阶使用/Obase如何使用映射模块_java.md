在Obase中,有四条映射管道分别提供了若干的事件,这些事件会在特定的时机被触发并抛出事件数据,通过这些事件数据可以实现对特定过程中某些节点的Obase对象的查看和修改.

这四条管道分别是:

- “保存”管道,提供了如开始保存getIBeginSaving()事件,开始保存映射单元getIBeginSavingUnit()事件,执行Sql语句前getIPreExecuteCommand()事件,执行Sql语句后getIPostExecuteCommand()事件等保存相关的事件.
- "删除"管道,提供了如开始删除getIBeginDeleting()事件,开始一组BeginDeletingGroup()删除事件,执行Sql语句前PreExecuteCommand()事件,执行Sql语句后PostExecuteCommand()事件等删除相关的事件.
- "查询"管道,提供了如开始查询getIBeginQuery()事件,结束查询getIEndQuery事件,执行Sql语句前getIPreExecuteCommand()事件,执行Sql语句后getIPostExecuteCommand()事件等查询相关的事件.
- "就地修改"管道,提供了如开始就地修改getIBeginDirectlyChanging()事件,结束就地修改getIEndDirectlyChanging()事件,执行Sql语句前getIPreExecuteCommand()事件,执行Sql语句后getIPostExecuteCommand()事件等就地修改相关的事件.

这些事件方法返回的对象都可以用addListener方法为其订阅事件并在订阅事件的代码中对事件数据对象进行处理,addListener方法为接受EventListener<T extends EventObject>类型的参数,可以使用Lambda表达式来进行匿名实现.

Obase提供了IMappingModule接口,实现此接口的方法即可为传入的四条管道注册事件处理器来处理事件,此外还在上下文上提供了registerModule用于向某个上下文注册映射模块.

详细的内容可以查看源代码,接下来使用两个例子来讲解如何使用这些扩展管道.

## 查看执行的Sql语句

首先,我们定义一个QueryTestModule用于显示Sql语句的模块:

```
/**
 * 用于显示Sql语句的模块
 */
public class QueryTestModule implements IMappingModule{

    /**
     * 当前的数据源
     */
    private final EDataSource dataSource;

    /**
     * 构造用于显示Sql语句的模块
     * @param dataSource 当前的数据源
     */
    private QueryTestModule(EDataSource dataSource){

        this.dataSource = dataSource;
    }

    /**
     * 初始化映射模块
     *
     * @param savingPipeline           "保存"管道
     * @param deletingPipeline         "删除"管道
     * @param queryPipeline            "查询"管道
     * @param directlyChangingPipeline "就地修改"管道
     * @param objectContext            对象上下文
     */
    @Override
    public void init(ISavingPipeline savingPipeline, IDeletingPipeline deletingPipeline, IQueryPipeline queryPipeline, IDirectlyChangingPipeline directlyChangingPipeline, ObjectContext objectContext) {
        //查询sql执行前
        queryPipeline.getIQueryPipelinePreExecuteCommand().addListener(eventObject -> {
            //用QuerySqlParameteredView进行处理
            var view = SqlParameterizedView.getSqlParameterizedView((QuerySql) obj, this.dataSource);
            //自行处理SimpleViewString
            System.out.println(view.getSimpleSqlString());
        });
    }
}
```
这里注册了"查询"管道的执行Sql语句前getIPreExecuteCommand()事件,并且在处理程序中用SqlParameterizedView的静态方法来获取SqlParameterizedView对象,此对象上有方法getSqlString()和getSimpleViewString()来分别获取Sql语句和非参数化的Sql语句.

然后在调用上下文的registerModule来注册这个模块的实例即可实现在某个上下文查询时显示查询的Sql语句.

```
//模块仅对当前上下文有效 如果需要都生效 把注册代码放置于构造函数中
context.registerModule(new QueryTestModule(dataSource));
```

## 禁止执行删除语句

首先,我们定义一个DisableDeleteModule禁用删除模块的模块:

```
/**
 * 禁用删除模块
 */
public class DisableDeleteModule implements IMappingModule{

    /**
     * 初始化映射模块
     *
     * @param savingPipeline           "保存"管道
     * @param deletingPipeline         "删除"管道
     * @param queryPipeline            "查询"管道
     * @param directlyChangingPipeline "就地修改"管道
     * @param objectContext            对象上下文
     */
    @Override
    public void init(ISavingPipeline savingPipeline, IDeletingPipeline deletingPipeline, IQueryPipeline queryPipeline, IDirectlyChangingPipeline directlyChangingPipeline, ObjectContext objectContext) {
        //保存(Update/Insert)Sql执行前
        savingPipeline.getSavingPreExecuteCommand().addListener(eventObject -> {
            ChangeSql changeSql = (ChangeSql)eventObject.getCommand();
            if(changeSql.getChangeType().equals(EChangeType.Delete))
                throw new RuntimeException("删除操作已被禁用")
        });
    }
}
```

这里注册了"保存"管道的执行Sql语句前getIPreExecuteCommand()事件,在语句类型为删除时抛出异常.

然后在调用上下文的registerModule来注册这个模块的实例即可实现在某个上下文尝试删除时抛出异常.

```
//模块仅对当前上下文有效 如果需要都生效 把注册代码放置于构造函数中
context.registerModule(new DisableDeleteModule());
```