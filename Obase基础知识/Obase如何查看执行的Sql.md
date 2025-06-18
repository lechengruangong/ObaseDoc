在6.1.7中,Obase提供了一个新的工具类QuerySqlParameteredView,用于查看Obase执行的Sql语句

此工具类提供了GetSqlParameterizedView方法和SimpleViewString,Parameters属性访问器用于查看Sql,具体的使用方法如下

## .net版

定义一个如下的映射模块,此模块中的datasource即为当前所使用的数据源类型,支持Oracle,MySql,Sqlite,Sqlserver,PostgreSql.

```
 /// <summary>
 ///     查询测试用的模块
 /// </summary>
 public class QueryTestModule : IMappingModule
 {
     /// <summary>
     ///     当前的数据源
     /// </summary>
     private readonly eDataSource _dataSource;

     /// <summary>
     ///     构造测试用的模块
     /// </summary>
     /// <param name="dataSource"></param>
     public QueryTestModule(eDataSource dataSource)
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
        //保存管道的PreExecuteCommand事件会将创建和修改所使用的SQL在事件数据中返回
        savingPipeline.PreExecuteCommand += (sender, args) =>
        {
            //用QuerySqlParameteredView进行处理 注意此处的参数为ChangeSql
            var view = SqlParameterizedView.GetSqlParameterizedView((ChangeSql)args.Command,
                _dataSource);
            //此处即可获取到view 此View内保存了执行的SQL字符串
        };
        
        //删除管道的PreExecuteCommand事件会将标记删除时所使用的SQL在事件数据中返回
        deletingPipeline.PreExecuteCommand += (sender, args) =>
        {
            //用QuerySqlParameteredView进行处理 注意此处的参数为ChangeSql
            var view = SqlParameterizedView.GetSqlParameterizedView((ChangeSql)args.Command,
                _dataSource);
            //此处即可获取到view 此View内保存了执行的SQL字符串
        };
        
        //查询管道的PreExecuteCommand事件会将执行查询所使用的SQL在事件数据中返回
        queryPipeline.PreExecuteCommand += (sender, args) =>
        {
            //用QuerySqlParameteredView进行处理 注意此处的参数为QuerySql
            var view = SqlParameterizedView.GetSqlParameterizedView((QuerySql)args.Context.Command,
                _dataSource);
             //此处即可获取到view 此View内保存了执行的SQL字符串
        };
        
        //就地修改管道的PreExecuteCommand事件会将就地删除 就地修改时所使用的SQL在事件数据中返回
        directlyChangingPipeline.PreExecuteCommand += (sender, args) =>
        {
            //用QuerySqlParameteredView进行处理 注意此处的参数为ChangeSql
            var view = SqlParameterizedView.GetSqlParameterizedView((ChangeSql)args.Command,
                _dataSource);
            //此处即可获取到view 此View内保存了执行的SQL字符串
        };
        
     }
 }
```

然后为上下文启用此模块,即可在执行语句时看到Obase生成的Sql语句,模块中savingPipeline的事件是在保存时触发,queryPipeline是在查询时触发deletingPipeline,是标记删除时触发,directlyChangingPipeline是就地修改时触发.

```
context.RegisterModule(new QueryTestModule(dataSource));
```
此时只需要将获取到View的内返回的SQL字符串输出即可获取到Obase执行的SQL语句了.

如果需要所有的上下文都启用此模块,可以将这些代码在构造函数内调用.

## Java版

Java版待重写