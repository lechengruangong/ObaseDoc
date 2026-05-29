在Obase中,有四条映射管道分别提供了若干的事件,这些事件会在特定的时机被触发并抛出事件数据,通过这些事件数据可以实现对特定过程中某些节点的Obase对象的查看和修改.

这四条管道分别是:

- “保存”管道,提供了如开始保存BeginSaving事件,开始保存映射单元BeginSavingUnit事件,执行Sql语句前PreExecuteCommand事件,执行Sql语句后PostExecuteCommand事件等保存相关的事件.
- "删除"管道,提供了如开始删除BeginDeleting事件,开始一组BeginDeletingGroup删除事件,执行Sql语句前PreExecuteCommand事件,执行Sql语句后PostExecuteCommand事件等删除相关的事件.
- "查询"管道,提供了如开始查询BeginQuery事件,结束查询EndQuery事件,执行Sql语句前PreExecuteCommand事件,执行Sql语句后PostExecuteCommand事件等查询相关的事件.
- "就地修改"管道,提供了如开始就地修改BeginDirectlyChanging事件,结束就地修改EndDirectlyChanging事件,执行Sql语句前PreExecuteCommand事件,执行Sql语句后PostExecuteCommand事件等就地修改相关的事件.

Obase提供了IMappingModule接口,实现此接口的方法即可为传入的四条管道注册事件处理器来处理事件,此外还在上下文上提供了RegisterModule用于向某个上下文注册映射模块.

详细的内容可以查看源代码,接下来使用两个例子来讲解如何使用这些扩展管道.

## 查看执行的Sql语句

首先,我们定义一个QueryTestModule用于显示Sql语句的模块:

```
/// <summary>
///     用于显示Sql语句的模块
/// </summary>
public class QueryTestModule : IMappingModule
{
    /// <summary>
    ///     当前的数据源
    /// </summary>
    private readonly EDataSource _dataSource;

    /// <summary>
    ///     构造用于显示Sql语句的模块
    /// </summary>
    public QueryTestModule(EDataSource dataSource)
    {
        _dataSource = dataSource;
    }

    /// <summary>
    ///     初始化映射模块。
    /// </summary>
    /// <param name="savingPipeline">"保存"管道。</param>
    /// <param name="deletingPipeline">"删除"管道。</param>
    /// <param name="queryPipeline">"查询"管道。</param>
    /// <param name="directlyChangingPipeline">"就地修改"管道。</param>
    /// <param name="objectContext">对象上下文</param>
    public void Init(ISavingPipeline savingPipeline, IDeletingPipeline deletingPipeline,
        IQueryPipeline queryPipeline,
        IDirectlyChangingPipeline directlyChangingPipeline, ObjectContext objectContext)
    {
        queryPipeline.PreExecuteCommand += (sender, args) =>
        {
            //用SqlParameterizedView进行处理
            var view = SqlParameterizedView.GetSqlParameterizedView((QuerySql)args.Context.Command,
                _dataSource);
            //自行处理SimpleViewString
            Console.WriteLine(view.SimpleViewString);
        };
    }
}
```
这里注册了"查询"管道的执行Sql语句前PreExecuteCommand事件,并且在处理程序中用SqlParameterizedView类的静态方法来获取SqlParameterizedView对象,此对象上有属性访问器SqlString和SimpleViewString来分别获取Sql语句和非参数化的Sql语句.

然后在调用上下文的RegisterModule来注册这个模块的实例即可实现在某个上下文查询时显示查询的Sql语句.

```
//模块仅对当前上下文有效 如果需要都生效 把注册代码放置于构造函数中
context.RegisterModule(new QueryTestModule(dataSource));
```

## 禁止执行删除语句

首先,我们定义一个DisableDeleteModule禁用删除模块的模块:

```
/// <summary>
///     禁用删除模块
/// </summary>
public class DisableDeleteModule : IMappingModule
{
    /// <summary>
    ///     初始化映射模块。
    /// </summary>
    /// <param name="savingPipeline">"保存"管道。</param>
    /// <param name="deletingPipeline">"删除"管道。</param>
    /// <param name="queryPipeline">"查询"管道。</param>
    /// <param name="directlyChangingPipeline">"就地修改"管道。</param>
    /// <param name="objectContext">对象上下文</param>
    public void Init(ISavingPipeline savingPipeline, IDeletingPipeline deletingPipeline, IQueryPipeline queryPipeline,
        IDirectlyChangingPipeline directlyChangingPipeline, ObjectContext objectContext)
    {
        savingPipeline.PreExecuteCommand += (sender, args) =>
        {
            var changeSql = (ChangeSql)args.Command;
            if(changeSql.ChangeType == EChangeType.Delete)
                throw new InvalidOperationException("删除操作已被禁用");
        };
    }
}
```

这里注册了"保存"管道的执行Sql语句前PreExecuteCommand事件,在语句类型为删除时抛出异常.

然后在调用上下文的RegisterModule来注册这个模块的实例即可实现在某个上下文尝试删除时抛出异常.

```
//模块仅对当前上下文有效 如果需要都生效 把注册代码放置于构造函数中
context.RegisterModule(new DisableDeleteModule());
```