在Obase中,根据不同的平台特性提供了不同的事务处理.
## dotNet
在dotNet平台,Obase提供了如下的事务处理模式:

1. 自动事务

   Obase的自动事务,指的是Obase自动附加的事务.在Obase中,每次SaveChanges时会把此次保存和上次保存之间的所有上下文管理的对象修改操作包含在一个事务块内.

   如果是首次SaveChanges则是从构造上下文开始的所有上下文管理的对象修改(此处的上下文管理的对象指的是附加到上下文的新对象和由上下文查询得到的旧对象).
   
   以下代码为自动事务的示例: 
   ```
   //构造一个上下文
   var context = new Context();
   //查询对象
   var list = context.CreateSet<DomainClass>().ToList();
   //将前三个的主键修改 其中第二个主键修改为空
   list[0].Id = 11;
   list[1].Id = null;
   list[2].Id = 13;
   //在保存时 会由数据源抛出异常 此时这三个对象的修改都处于同一事务块内 所以这三个的修改都不会保存
   context.SaveChanges();
   ```
2. 手动事务

   Obase的手动事务,指的是调用Obase的手动事务方法自己控制事务.

   Obase的手动事务方法遵循ADO.NET的try-Begin-Commit-Catch-RollBack-Finally-Release模式.

   以下代码为手动事务的示例:
   ```
   //构造一个上下文
   var context = new Context();
   //查询对象
   var list = context.CreateSet<DomainClass>().ToList();
   try
   {
      //手动开启事务
      context.BeginTransaction();

      //修改前三个的LongNumber
      list[0].LongNumber = 14;
      list[1].LongNumber = 15;
      list[2].LongNumber = 16;
   
      //保存之前的修改
      context.SaveChanges();
      //调用模拟的外部方法 
      //假定此方法会抛异常 那么就会在下面的Catch中导致从BeginTransaction开始的事务块回滚
      //假定此方法不抛异常 那么就会就会进入Commit将从BeginTransaction开始的事务块提交
      OuterMethod();
      
      //提交修改
      context.Commit();
   }
   catch (Exception)
   {
      //发生异常 回滚
      context.RollbackTransaction();
   }
   finally
   {
      //最后释放资源
      context.Release();
   }
   ```

3. 已有连接事务
   
   Obase的已有连接事务,指的是由第三方提供连接来构造上下文配置提供者并由第三方来管理连接的事务的模式.

   此种事务模式通常用于与旧系统的事务进行对接,如旧系统中已存在了一个业务场景,原有逻辑为有一个事务先执行A语句再执行B语句最后一起提交,此时新的C部分由Obase来负责,需要将旧的逻辑改为先执行A再执行新逻辑C最后执行B再提交.

   此时可以使用已有连接事务来处理,令Obase也使用旧系统提供的连接来处理事务.

   以下代码为旧系统的事务示例:

   ```
   //假设此时已经有了一个连接 且是打开的
   var connection = GetConnection();
   //此处开始是原有的代码 使用try代码块包裹的事务
   try
   {
      //原有的开启了一个事务
      var transaction = connection.BeginTransaction();
      //原有的执行逻辑A
      //此处做了一些数据库操作XXX
   
      //原有的执行逻辑B
      //此处做了一些数据库操作XXX
   
      //原有的提交事务
      transaction.Commit();
   
      //原有的其它代码XXX
   }
   catch(Exception ex)
   {
      //原有的回滚事务
      transaction.Rollback();
   }
   finally
   {
       //原有的关闭连接
       connection.Close();
       connection.Dispose();
   }
   ```
   此时引入Obase并且像保持之前的事务处理逻辑,需要使用已有连接事务,首先需要定义一个新的上下文配置提供者(此处以MySql为例):
   ```
   /// <summary>
   ///     使用已存在的MySql连接上下文配置
   /// </summary>
   public class MySqlExistingConnectionContextConfiger : SqlContextConfigurator
   {
      /// <summary>
      ///     使用已存在的MySql连接上下文配置。
      /// </summary>
      /// <param name="sqlExecutor">Sql语句执行器。</param>
      /// <param name="enableStructMapping">指示是否启用存储架构映射</param>
      public MySqlExistingConnectionContextConfiger(ISqlExecutor sqlExecutor, bool enableStructMapping = false) : base(sqlExecutor, enableStructMapping)
      {
      }
   
       /// <summary>
       ///     使用指定的建模器创建对象数据模型。
       /// </summary>
       /// <param name="modelBuilder">对象数据模型建造器</param>
       protected override void CreateModel(ModelBuilder modelBuilder)
       {
           //模型注册相关的代码
       }
   }
   ```
   这个上下文配置提供者继承的配置提供器是SqlContextConfigurator,构造函数需要传入一个特定的SQL执行器来接收已有的连接作为参数,这样就可以使用之前的事务代码来控制事务了,对应的上下文如下:
   ```
   /// <summary>
   ///     使用已存在的MySql连接上下文
   /// </summary>
   public class MySqlExistingConnectionContext : ObjectContext
   {
      /// <summary>
      ///     构造ObjectContext对象
      /// </summary>
      /// <param name="providerFactory">用于创建数据提供程序类实例的工厂</param>
      /// <param name="connection">连接</param>
      public MySqlExistingConnectionContext(DbProviderFactory providerFactory, DbConnection connection,DbTransaction transaction = null) : base(new MySqlExistingConnectionContextConfiger(new ExistingConnectionSqlExecutor(providerFactory, connection, EDataSource.MySql, transaction)))
      {
      }
   }
   ```
   这种使用已有连接的ExistingConnectionSqlExecutor需要这样几个参数:数据提供器工厂,数据源连接,数据源类型和数据源连接已开启的事务.

   使用这些参数构造的ExistingConnectionSqlExecutor会将已有的连接和事务作为Obase的Sql执行者使用,这样就可以复用原有的事务处理了.

   接下来只要修改旧逻辑即可:
   ```
   //假设此时已经有了一个连接 且是打开的
   var connection = GetConnection();
   //此处开始是原有的代码 使用try代码块包裹的事务
   try
   {
      //原有的开启了一个事务
      var transaction = connection.BeginTransaction();
      //原有的执行逻辑A
      //此处做了一些数据库操作XXX
   
      //构造一个上下文
      var exContext = new MySqlExistingConnectionContext(MySqlClientFactory.Instance, connection, transaction);
   
      //执行一些Obase的逻辑
      //Obase的保存
      exContext.SaveChanges();
   
      //原有的执行逻辑B
      //此处做了一些数据库操作XXX
   
      //原有的提交事务
      transaction.Commit();
   
      //原有的其它代码XXX
   }
   catch(Exception ex)
   {
      //原有的回滚事务
      transaction.Rollback();
   }
   finally
   {
       //原有的关闭连接
       connection.Close();
       connection.Dispose();
   }
   ```
   实际上就是在原有的逻辑A和B之间使用现有的连接和事务构造上下文,之后再增加Obase的逻辑即可.
## java

java版待重写